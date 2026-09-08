# ChainlinkOracle

## ChainlinkOracle: Contract Reference

### 📋 Overview

The ChainlinkOracle is the contract that integrates the Chainlink EUR/USD and USDC/USD price feeds on Base, with staleness validation, price bounds, a deviation circuit breaker, an L2 sequencer-uptime check and timestamp-manipulation protection.

> **Role in the current architecture:** the ChainlinkOracle is the **fallback** EUR/USD source (OracleRouter slot 0) and the protocol's **USDC/USD** source. The active EUR/USD source is the market slot (slot 1, `HyperliquidEurUsdOracle`), and governance can revert to this contract in one transaction (`switchOracle(0)`). This page documents the contract itself; see **[Oracle Architecture](oracle-architecture.md)** for how the sources fit together.

Live version: **1.0.4** · address `0xaEE3c9c298051ef7242882AbCaE2Fd12d29443E7` (Base mainnet).

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

Plain UUPS proxy: upgrades are executed directly by the governance Safe (`UPGRADER_ROLE`), without the timelock used for the core protocol contracts.

**Chainlink Price Feeds (Base mainnet)**

| Feed | Variable | Address | Description |
|------|----------|---------|-------------|
| EUR/USD | `eurUsdPriceFeed` | `0xc91D87E81faB8f93699ECf7Ee9B44D11e1D53F0F` | Euro price in dollars |
| USDC/USD | `usdcUsdPriceFeed` | `0x7e860098F58bBFC8648a4311b374B1D669a2bc6B` | USDC price (validation) |
| L2 sequencer uptime | `sequencerUptimeFeed` | `0xBCF85224fc0756B9Fa45aA7892530B47e10b6433` | Base sequencer status |

***

### 🔐 Roles & Permissions

| Role | Responsibilities |
|------|-----------------|
| `DEFAULT_ADMIN_ROLE` | Role management, treasury, recovery, dev-mode proposal/apply |
| `ORACLE_MANAGER_ROLE` | Feed configuration, price bounds, USDC tolerance, sequencer feed |
| `EMERGENCY_ROLE` | Circuit breaker trigger/reset, pause/unpause |
| `UPGRADER_ROLE` | Contract upgrades (Safe-direct) |

All four roles are held by the 2-of-3 governance Safe.

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

// Dev mode two-step delay
uint256 public constant DEV_MODE_DELAY = 48 hours;
```

***

### 📊 Configuration Variables

#### Price Bounds

| Variable | Type | Description | Live Value |
|----------|------|-------------|------------|
| `minEurUsdPrice` | `uint256` | Min EUR/USD price (18 dec) | 0.80e18 |
| `maxEurUsdPrice` | `uint256` | Max EUR/USD price (18 dec) | 1.40e18 |
| `usdcToleranceBps` | `uint256` | USDC tolerance (BPS) | 200 (2%) |
| `sequencerGracePeriod` | `uint256` | Seconds after a sequencer restart before prices are trusted | 3600 (1 hour) |

#### State Variables

| Variable | Type | Description |
|----------|------|-------------|
| `lastValidEurUsdPrice` | `uint256` | Last valid EUR/USD price |
| `lastPriceUpdateTime` | `uint256` | Last update timestamp |
| `lastPriceUpdateBlock` | `uint256` | Last update block |
| `circuitBreakerTriggered` | `bool` | Circuit breaker state (live: `false`) |
| `devModeEnabled` | `bool` | Development mode active (live: `false`) |
| `pendingDevMode` / `devModePendingAt` | `bool` / `uint256` | Two-step dev-mode proposal state |

***

### 🔄 Price Retrieval

#### getEurUsdPrice

```solidity
function getEurUsdPrice() external returns (uint256 price, bool isValid);
```

Not a `view`: a valid read advances `lastValidEurUsdPrice` / `lastPriceUpdateTime` / `lastPriceUpdateBlock` and emits `PriceUpdated`. On any failed check the function returns `(lastValidEurUsdPrice, false)` - it never reverts. **Consumers treat `isValid = false` as a hard stop**: `QuantillonVault` reverts mint and redeem on an invalid price.

**Validation Flow**

```
┌─────────────────────────────────────────────────────────────┐
│                    PRICE VALIDATION FLOW                     │
├─────────────────────────────────────────────────────────────┤
│  0. Circuit breaker active or contract paused?              │
│     └── yes → (lastValidEurUsdPrice, false)                 │
│                                                              │
│  1. L2 sequencer uptime check (Base)                         │
│     ├── sequencer down, or restarted < sequencerGracePeriod │
│     │   ago, or malformed round → (lastValid, false)        │
│                                                              │
│  2. Fetch EUR/USD from Chainlink (latestRoundData)          │
│     └── roundId == answeredInRound, startedAt <= updatedAt  │
│                                                              │
│  3. Validate Timestamp                                       │
│     ├── updatedAt + MAX_PRICE_STALENESS > now                │
│     └── updatedAt <= now + MAX_TIMESTAMP_DRIFT               │
│                                                              │
│  4. Validate Price Bounds                                    │
│     └── minEurUsdPrice <= price <= maxEurUsdPrice           │
│                                                              │
│  5. Validate Price Deviation (if !devModeEnabled)           │
│     └── |price - lastValidPrice| <= 5% of lastValidPrice    │
│                                                              │
│  6. Commit: update lastValid* state, emit PriceUpdated       │
│  7. Return (price, true) - 18 decimals                       │
└─────────────────────────────────────────────────────────────┘
```

#### getUsdcUsdPrice

```solidity
function getUsdcUsdPrice() external view returns (uint256 price, bool isValid);
```

Returns the Chainlink USDC/USD price scaled to 18 decimals. If the feed is stale (older than 25 hours), malformed, or the price is outside `1.00 ± usdcToleranceBps` (0.98–1.02), the function returns `(1e18, false)`: the price defaults to $1.00 and the invalid flag is passed to consumers. A USDC depeg does **not** trip the EUR/USD circuit breaker.

***

### ⏱️ Timestamp Validation

Sequencers can slightly manipulate `block.timestamp`. The contract rejects any feed timestamp in the future beyond `MAX_TIMESTAMP_DRIFT` and any round older than the staleness window:

```solidity
// Oracle timestamp must not be in the future or stale
updatedAt <= now + MAX_TIMESTAMP_DRIFT  &&  updatedAt + MAX_PRICE_STALENESS > now
```

***

### 🛰️ Sequencer Uptime Check

Base is an L2: if the sequencer is down, Chainlink rounds stop updating while their timestamps may still look fresh. The oracle therefore reads the Chainlink **L2 sequencer uptime feed** on every EUR/USD read and refuses to trust prices while the sequencer is reported down and for `sequencerGracePeriod` (1 hour) after it comes back.

```solidity
AggregatorV3Interface public sequencerUptimeFeed;  // address(0) disables the check (L1)
uint256 public sequencerGracePeriod;                // seconds; live 3600

function setSequencerUptimeFeed(address feed, uint256 gracePeriod)
    external onlyRole(ORACLE_MANAGER_ROLE);
// emits SequencerFeedUpdated(feed, gracePeriod)
```

***

### 🧪 Dev Mode

`devModeEnabled` disables the 5% deviation check (staleness and bounds stay active) to facilitate testing on local chains and testnets. Enabling or disabling it is a **two-step, delayed** operation:

```solidity
function proposeDevMode(bool enabled) external onlyRole(DEFAULT_ADMIN_ROLE);
// records pendingDevMode and devModePendingAt = now + DEV_MODE_DELAY (48 hours)
// emits DevModeProposed(pending, activatesAt)

function applyDevMode() external onlyRole(DEFAULT_ADMIN_ROLE);
// reverts before devModePendingAt; emits DevModeToggled(enabled, caller)
```

**Dev Mode Impact**

| Validation | Normal Mode | Dev Mode |
|------------|-------------|----------|
| Timestamp staleness | ✅ Active | ✅ Active |
| Price bounds | ✅ Active | ✅ Active |
| Price deviation (5%) | ✅ Active | ❌ Disabled |

> Live state on Base mainnet: **disabled** (`devModeEnabled = false`). The 48-hour delay makes any change visible on-chain before it takes effect.

***

### 🚨 Circuit Breaker

`circuitBreakerTriggered` latches the oracle into last-valid-price mode: every read returns `(lastValidEurUsdPrice, false)` until the breaker is reset.

**How it is set**

1. **Manually**, by the emergency role: `triggerCircuitBreaker()`.
2. **Automatically** when the contract refreshes its baseline (initialization and `resetCircuitBreaker()`): if the freshly fetched EUR/USD price fails validation (stale, out of bounds, invalid timestamp, deviation > 5%), the breaker is set again and `CircuitBreakerTriggered` is emitted.

A failed validation on an ordinary `getEurUsdPrice()` read does not latch the breaker: it simply returns `(lastValidEurUsdPrice, false)`, which already stops mint/redeem.

```solidity
function triggerCircuitBreaker() external onlyRole(EMERGENCY_ROLE);
function resetCircuitBreaker() external onlyRole(EMERGENCY_ROLE);
```

**Reset conditions**: the reset re-fetches the price; it only clears the breaker if Chainlink provides fresh data within bounds. It must be called manually after investigation.

***

### 📊 View Functions

```solidity
// Health summary
function getOracleHealth() external view
    returns (bool isHealthy, bool eurUsdFresh, bool usdcUsdFresh);

// EUR/USD detail
function getEurUsdDetails() external view
    returns (uint256 currentPrice, uint256 lastValidPrice, uint256 lastUpdate, bool isStale, bool withinBounds);

// Configuration snapshot
function getOracleConfig() external view
    returns (uint256 minPrice, uint256 maxPrice, uint256 maxStaleness, uint256 usdcTolerance, bool circuitBreakerActive);

// USDC/USD
function getUsdcUsdPrice() external view returns (uint256 price, bool isValid);

// Feed wiring
function getPriceFeedAddresses() external view
    returns (address eurUsdFeedAddress, address usdcUsdFeedAddress, uint8 eurUsdDecimals, uint8 usdcUsdDecimals);
function checkPriceFeedConnectivity() external view
    returns (bool eurUsdConnected, bool usdcUsdConnected, uint80 eurUsdLatestRound, uint80 usdcUsdLatestRound);
```

***

### ⚙️ Configuration

```solidity
// Price bounds (18 decimals) - reverts if _minPrice == 0 or _maxPrice <= _minPrice
function updatePriceBounds(uint256 _minPrice, uint256 _maxPrice)
    external onlyRole(ORACLE_MANAGER_ROLE);

// USDC tolerance in basis points
function updateUsdcTolerance(uint256 newToleranceBps)
    external onlyRole(ORACLE_MANAGER_ROLE);

// Chainlink feed addresses
function updatePriceFeeds(address _eurUsdFeed, address _usdcUsdFeed)
    external onlyRole(ORACLE_MANAGER_ROLE);

// L2 sequencer uptime feed and grace period
function setSequencerUptimeFeed(address feed, uint256 gracePeriod)
    external onlyRole(ORACLE_MANAGER_ROLE);

// Treasury for recovered funds
function updateTreasury(address _treasury) external onlyRole(DEFAULT_ADMIN_ROLE);
```

> Changing price feeds is a critical operation. It is executed by the governance Safe (2-of-3); the oracle contracts are not behind the upgrade timelock.

***

### 🛡️ Security

#### Implemented Protections

| Protection | Description |
|------------|-------------|
| **Staleness Check** | EUR/USD price > 2h = stale (USDC/USD: > 25h) |
| **Timestamp Drift** | No future prices (15-minute drift allowance) |
| **Round Integrity** | `roundId == answeredInRound`, `startedAt <= updatedAt`, price > 0 |
| **Sequencer Uptime** | No prices while the Base sequencer is down or within 1 hour of a restart |
| **Price Bounds** | EUR/USD between 0.80 and 1.40 |
| **Deviation Limit** | Max 5% between consecutive valid prices |
| **USDC Validation** | USDC must stay within ±2% of $1.00, else `isValid = false` |
| **Circuit Breaker** | Last-valid-price mode with `isValid = false` |
| **Role-Based Access** | Permission separation, all roles on the governance Safe |

#### Recovery Functions

```solidity
// Recover tokens sent by mistake (to treasury)
function recoverToken(address token, uint256 amount) 
    external onlyRole(DEFAULT_ADMIN_ROLE);

// Recover ETH sent by mistake (to treasury)
function recoverETH() external onlyRole(DEFAULT_ADMIN_ROLE);
```

***

### 📋 Events

```solidity
event PriceUpdated(uint256 eurUsdPrice, uint256 usdcUsdPrice, uint256 indexed timestamp);
event CircuitBreakerTriggered(uint256 attemptedPrice, uint256 lastValidPrice, string indexed reason);
event CircuitBreakerReset(address indexed admin);
event PriceBoundsUpdated(string indexed boundType, uint256 newMinPrice, uint256 newMaxPrice);
event PriceFeedsUpdated(address newEurUsdFeed, address newUsdcUsdFeed);
event SequencerFeedUpdated(address indexed feed, uint256 gracePeriod);
event DevModeProposed(bool pending, uint256 activatesAt);
event DevModeToggled(bool enabled, address indexed caller);
event TreasuryUpdated(address indexed newTreasury);
event ETHRecovered(address indexed to, uint256 amount);
```

***

### 📐 Example Scenarios

#### Scenario 1: Normal Price

```
Chainlink EUR/USD: 1.08000000 (8 decimals)
→ Sequencer up ✅
→ Converted to 18 decimals: 1.080000000000000000
→ Bounds: 0.80 ≤ 1.08 ≤ 1.40 ✅
→ Staleness: 5 minutes < 2 hours ✅
→ Deviation: 0.5% < 5% ✅
→ Return: (1.08e18, true)
```

#### Scenario 2: Stale Price

```
Chainlink EUR/USD: 1.08 (3 hours ago)
→ Staleness: 3 hours > 2 hours ❌
→ Return: (lastValidEurUsdPrice, false) → vault reverts mint/redeem
```

#### Scenario 3: Flash Crash

```
Chainlink EUR/USD: 0.70 (sudden crash)
→ Bounds: 0.70 < 0.80 ❌
→ Return: (lastValidEurUsdPrice, false) → vault reverts mint/redeem
```

#### Scenario 4: USDC Depeg

```
Chainlink USDC/USD: 0.95
→ Deviation: |0.95 - 1.00| = 5% > 2% ❌
→ getUsdcUsdPrice() returns (1.00e18, false)
→ Consumers treat the invalid flag as a stop; the EUR/USD breaker is untouched
```

#### Scenario 5: Sequencer Restart

```
Base sequencer back online 20 minutes ago
→ 20 minutes < sequencerGracePeriod (1 hour) ❌
→ Return: (lastValidEurUsdPrice, false) until the grace period has elapsed
```

***

> **Summary**: The ChainlinkOracle is the protocol's **fallback** EUR/USD source (OracleRouter slot 0) and its USDC/USD source. Every read is validated for sequencer uptime, freshness, bounds and deviation; on any failure it returns the last valid price flagged invalid, which stops mint/redeem rather than ever pricing QEURO wrongly. Dev mode (deviation check off) is a 48-hour two-step change and is disabled on mainnet.
