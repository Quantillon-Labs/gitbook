# ChainlinkOracle

## ChainlinkOracle: EUR/USD Price Infrastructure

### 📋 Overview

The ChainlinkOracle is the contract that manages integration with Chainlink price feeds to obtain real-time EUR/USD and USDC/USD prices. It includes robust security mechanisms like circuit breakers, data freshness validation, and timestamp manipulation protection.

> **Role in the current architecture:** the ChainlinkOracle is now the **fallback** EUR/USD source (slot 0) behind the [OracleRouter](oracle-architecture.md). QEURO mint/redeem is priced by default off the **Hyperliquid** EUR/USD market mid (the hedge venue) via the `HyperliquidEurUsdOracle`. The ChainlinkOracle still provides **USDC/USD validation** for the protocol and acts as a **one-transaction fallback** (`switchOracle(0)`). The page below documents the ChainlinkOracle contract itself; see **[Oracle Architecture](oracle-architecture.md)** for how the two sources fit together.

***

### 🏗️ Contract Architecture

**Inheritance**

```solidity
contract ChainlinkOracle is 
    IChainlinkOracle,
    Initializable,
    AccessControlUpgradeable,
    PausableUpgradeable,
    UUPSUpgradeable
```

**Chainlink Price Feeds**

| Feed | Variable | Description |
|------|----------|-------------|
| EUR/USD | `eurUsdPriceFeed` | Euro price in dollars |
| USDC/USD | `usdcUsdPriceFeed` | USDC price (validation) |

***

### 🔐 Roles & Permissions

| Role | Responsibilities |
|------|-----------------|
| `DEFAULT_ADMIN_ROLE` | General administration |
| `ORACLE_MANAGER_ROLE` | Feed configuration, bounds, tolerances |
| `EMERGENCY_ROLE` | Circuit breaker, pause |
| `UPGRADER_ROLE` | Contract upgrades |

***

### ⚙️ Security Constants

```solidity
// Data freshness (EUR/USD; USDC/USD uses 25 hours to match its daily heartbeat)
uint256 public constant MAX_PRICE_STALENESS = 2 hours;
uint256 public constant MAX_USDC_PRICE_STALENESS = 25 hours;

// Price deviation
uint256 public constant MAX_PRICE_DEVIATION = 500;     // 5% max between updates

// Calculation base
uint256 public constant BASIS_POINTS = 10000;

// Timestamp protection
uint256 public constant MAX_TIMESTAMP_DRIFT = 900;     // 15 minutes max drift

// Blocks per hour (for block-based checks)
uint256 public constant BLOCKS_PER_HOUR = 300;         // ~12 sec blocks
```

***

### 📊 Configuration Variables

#### Price Bounds

| Variable | Type | Description | Default Value |
|----------|------|-------------|---------------|
| `minEurUsdPrice` | `uint256` | Min EUR/USD price (18 dec) | 0.80e18 |
| `maxEurUsdPrice` | `uint256` | Max EUR/USD price (18 dec) | 1.40e18 |
| `usdcToleranceBps` | `uint256` | USDC tolerance (BPS) | 200 (2%) |

#### State Variables

| Variable | Type | Description |
|----------|------|-------------|
| `lastValidEurUsdPrice` | `uint256` | Last valid EUR/USD price |
| `lastPriceUpdateTime` | `uint256` | Last update timestamp |
| `lastPriceUpdateBlock` | `uint256` | Last update block |
| `circuitBreakerTriggered` | `bool` | Circuit breaker state |
| `devModeEnabled` | `bool` | Development mode active |

***

### 🔄 Price Retrieval

#### getEurUsdPrice Function

```solidity
function getEurUsdPrice() external view returns (uint256 price);
```

**Validation Flow**

```
┌─────────────────────────────────────────────────────────────┐
│                    PRICE VALIDATION FLOW                     │
├─────────────────────────────────────────────────────────────┤
│  1. Fetch EUR/USD from Chainlink                            │
│     └── latestRoundData()                                   │
│                                                              │
│  2. Validate Timestamp                                       │
│     ├── updatedAt + MAX_STALENESS > block.timestamp         │
│     └── updatedAt <= block.timestamp + MAX_TIMESTAMP_DRIFT  │
│                                                              │
│  3. Validate Price Bounds                                    │
│     └── minEurUsdPrice <= price <= maxEurUsdPrice           │
│                                                              │
│  4. Validate Price Deviation (if !devModeEnabled)           │
│     └── |price - lastValidPrice| <= 5% of lastValidPrice    │
│                                                              │
│  5. Update State                                             │
│     ├── lastValidEurUsdPrice = price                        │
│     ├── lastPriceUpdateTime = block.timestamp               │
│     └── lastPriceUpdateBlock = block.number                 │
│                                                              │
│  6. Return price (18 decimals)                               │
└─────────────────────────────────────────────────────────────┘
```

#### Circuit Breaker

If validation fails:

```solidity
// Circuit breaker triggered
circuitBreakerTriggered = true;

// Return last valid price as fallback
return lastValidEurUsdPrice;
```

***

### ⏱️ Timestamp Validation

#### Manipulation Protection

Miners can slightly manipulate `block.timestamp`. The contract implements two protections:

**1. Timestamp Drift Check**

```solidity
// Oracle timestamp must not be in the future
if (updatedAt > block.timestamp + MAX_TIMESTAMP_DRIFT) {
    revert InvalidTimestamp();
}
```

**2. Block-Based Staleness**

```solidity
// Variable tracking last update block
uint256 public lastPriceUpdateBlock;

// Additional block-based check
function _isBlockBasedStale() internal view returns (bool) {
    return block.number > lastPriceUpdateBlock + BLOCKS_PER_HOUR;
}
```

**Why Both?**

| Method | Advantage | Limitation |
|--------|-----------|------------|
| **Timestamp** | Temporal precision | Manipulable by miners |
| **Block number** | Not manipulable | Varies by chain |

Combining both provides robust protection.

***

### 🧪 Dev Mode

#### What is Dev Mode?

`devModeEnabled` disables certain validations to facilitate testing and development.

```solidity
bool public devModeEnabled;

function setDevMode(bool enabled) external onlyRole(ORACLE_MANAGER_ROLE) {
    devModeEnabled = enabled;
    emit DevModeToggled(enabled, msg.sender);
}
```

**Dev Mode Impact**

| Validation | Normal Mode | Dev Mode |
|------------|-------------|----------|
| Timestamp staleness | ✅ Active | ✅ Active |
| Price bounds | ✅ Active | ✅ Active |
| Price deviation (5%) | ✅ Active | ❌ Disabled |

**When to Use?**

- Local tests (Anvil/Hardhat)
- Testnets with simulated prices
- Edge case debugging

> ⚠️ **WARNING**: Dev Mode should NEVER be enabled in production on mainnet.

***

### 🚨 Circuit Breaker

#### Triggers

The circuit breaker is automatically triggered when:

1. **Price out of bounds**: `price < minEurUsdPrice` or `price > maxEurUsdPrice`
2. **Excessive deviation**: `|price - lastPrice| > 5%` (if devMode off)
3. **Stale data**: EUR/USD not updated for 2 hours (USDC/USD: 25 hours)
4. **Invalid timestamp**: Data in the future

#### Manual Trigger

```solidity
function triggerCircuitBreaker() external onlyRole(EMERGENCY_ROLE);
```

#### Reset

```solidity
function resetCircuitBreaker() external onlyRole(EMERGENCY_ROLE);
```

**Reset Conditions**:
- Oracle must provide fresh data
- Price must be within bounds
- Must be called manually after investigation

***

### 📊 USDC Validation

The contract also validates that USDC stays close to $1.00:

```solidity
// Tolerance: 2% (200 basis points)
uint256 public usdcToleranceBps = 200;

// USDC must be between $0.98 and $1.02
function _validateUsdcPrice() internal view {
    uint256 usdcPrice = _fetchUsdcPrice();
    uint256 deviation = |usdcPrice - 1e18| * BASIS_POINTS / 1e18;
    
    if (deviation > usdcToleranceBps) {
        // USDC is depegged → circuit breaker
        _triggerCircuitBreaker("USDC_DEPEG");
    }
}
```

**Why This Matters?**

- USDC is the protocol's collateral
- A USDC depeg directly affects collateral value
- The protocol must react immediately in case of depeg

***

### 📊 View Functions

#### Oracle Status

```solidity
function getOracleStatus() external view returns (
    uint256 currentPrice,
    uint256 lastUpdateTime,
    uint256 lastUpdateBlock,
    bool circuitBreakerActive,
    bool devModeActive
);
```

#### Health Check

```solidity
function getOracleHealth() external view returns (
    bool isHealthy,
    bool isPriceStale,
    bool isCircuitBreakerActive
);
```

#### Price Info

```solidity
function getPriceInfo() external view returns (
    uint256 eurUsdPrice,
    uint256 usdcUsdPrice,
    uint256 timestamp,
    bool isStale
);
```

***

### ⚙️ Configuration

#### Update Price Bounds

```solidity
function updatePriceBounds(uint256 _minPrice, uint256 _maxPrice) 
    external onlyRole(ORACLE_MANAGER_ROLE);
```

**Validations**:
- `_minPrice > 0`
- `_maxPrice > _minPrice`
- No radical changes (guardrails)

#### Update USDC Tolerance

```solidity
function updateUsdcTolerance(uint256 _toleranceBps) 
    external onlyRole(ORACLE_MANAGER_ROLE);
```

#### Update Price Feeds

```solidity
function updatePriceFeeds(address _eurUsdFeed, address _usdcUsdFeed) 
    external onlyRole(ORACLE_MANAGER_ROLE);
```

> ⚠️ Changing price feeds is a critical operation that should go through a timelock.

***

### 🛡️ Security

#### Implemented Protections

| Protection | Description |
|------------|-------------|
| **Staleness Check** | EUR/USD price > 2h = stale (USDC/USD: > 25h) |
| **Timestamp Drift** | No future prices |
| **Block Staleness** | Double-check based on blocks |
| **Price Bounds** | EUR/USD between 0.80 and 1.40 |
| **Deviation Limit** | Max 5% between updates |
| **USDC Validation** | USDC must stay ~$1.00 |
| **Circuit Breaker** | Automatic fallback |
| **Role-Based Access** | Permission separation |

#### Recovery Functions

```solidity
// Recover tokens sent by mistake
function recoverToken(address token, uint256 amount) 
    external onlyRole(DEFAULT_ADMIN_ROLE);

// Recover ETH sent by mistake
function recoverETH() external onlyRole(DEFAULT_ADMIN_ROLE);
```

***

### 📋 Events

```solidity
event PriceUpdated(
    uint256 eurUsdPrice, 
    uint256 usdcUsdPrice, 
    uint256 indexed timestamp
);

event CircuitBreakerTriggered(
    uint256 attemptedPrice, 
    uint256 lastValidPrice, 
    string indexed reason
);

event CircuitBreakerReset(address indexed admin);

event PriceBoundsUpdated(
    string indexed boundType, 
    uint256 newMinPrice, 
    uint256 newMaxPrice
);

event PriceFeedsUpdated(
    address newEurUsdFeed, 
    address newUsdcUsdFeed
);

event DevModeToggled(bool enabled, address indexed caller);

event TreasuryUpdated(address indexed newTreasury);

event ETHRecovered(address indexed to, uint256 amount);
```

***

### 📐 Example Scenarios

#### Scenario 1: Normal Price

```
Chainlink EUR/USD: 1.08000000 (8 decimals)
→ Converted to 18 decimals: 1.080000000000000000
→ Bounds: 0.80 ≤ 1.08 ≤ 1.40 ✅
→ Staleness: 5 minutes < 2 hours ✅
→ Deviation: 0.5% < 5% ✅
→ Return: 1.08e18
```

#### Scenario 2: Stale Price

```
Chainlink EUR/USD: 1.08 (3 hours ago)
→ Staleness: 3 hours > 2 hours ❌
→ Circuit breaker triggered
→ Return: lastValidEurUsdPrice (fallback)
```

#### Scenario 3: Flash Crash

```
Chainlink EUR/USD: 0.70 (sudden crash)
→ Bounds: 0.70 < 0.80 ❌
→ Circuit breaker triggered
→ Return: lastValidEurUsdPrice (1.08e18 fallback)
```

#### Scenario 4: USDC Depeg

```
Chainlink USDC/USD: 0.95
→ Deviation: |0.95 - 1.00| = 5% > 2% ❌
→ Circuit breaker triggered
→ Alert: collateral potentially depreciated
```

***

### 🔗 Protocol Integration

> The EUR/USD path below applies **when the ChainlinkOracle is the selected source in the OracleRouter** (fallback slot 0). In the live configuration, EUR/USD is served by the `HyperliquidEurUsdOracle` (slot 1) and the ChainlinkOracle serves USDC/USD validation — see [Oracle Architecture](oracle-architecture.md).

```
┌─────────────────────────────────────────────────────────────┐
│                 ORACLE INTEGRATION                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Chainlink Data Feeds                                        │
│       │                                                      │
│       ▼                                                      │
│  ChainlinkOracle                                            │
│       │                                                      │
│       ├── EUR/USD price (validated)                          │
│       │       │                                              │
│       │       ├─► QuantillonVault (mint/redeem pricing)     │
│       │       ├─► HedgerPool (P&L calculations)             │
│       │       └─► UserPool (analytics)                       │
│       │                                                      │
│       └── USDC/USD price (validation only)                   │
│               │                                              │
│               └─► Collateral health monitoring              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

***

> **Summary**: The ChainlinkOracle is the protocol's **fallback** EUR/USD source (OracleRouter slot 0) and its USDC/USD validation source, with robust validations. Security mechanisms (staleness — 2h EUR/USD, 25h USDC/USD — bounds, deviation, circuit breaker) protect against manipulation and corrupted data. Dev Mode enables testing but should never be enabled in production.
