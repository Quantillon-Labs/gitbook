# Core Mechanisms

## Mechanisms

Understanding how Quantillon Protocol operates is essential for both users and developers. This comprehensive guide breaks down the core mechanisms that power QEURO—from the dual-pool architecture and overcollateralization model to the innovative Yield Shift system that maintains peg stability while generating sustainable returns.

> **📋 MVP Status**: This documentation reflects the current MVP implementation. Features marked with 🚧 are planned for future phases.

***

### 🏗️ Protocol Architecture Overview

Quantillon Protocol represents a paradigm shift in stablecoin design, combining the capital efficiency of overcollateralized systems with the liquidity advantages of forex markets. At its core, the protocol operates through interconnected mechanisms that ensure peg stability, yield generation, and capital efficiency.

#### Core Design Principles

* **🔒 Over-collateralization**: Minimum 101% backing ensures systemic stability
* **⚖️ Delta-neutral hedging**: FX risk managed by designated hedger (MVP: single hedger)
* **📈 Dynamic yield distribution**: YieldShift mechanism for market-responsive incentive alignment
* **🌊 Liquidity inheritance**: Leverages existing USDC liquidity depth
* **🏛️ Decentralized governance**: Community-controlled parameter management via QTI

***

### 💶 QEURO Stablecoin Mechanics

#### Minting Process (MVP Implementation)

The QEURO minting mechanism is designed for simplicity and capital efficiency:

**📥 Step-by-Step Minting**

1. **USDC Deposit**: Users deposit USDC to the QuantillonVault
2. **Oracle Price Check**: Chainlink oracles provide real-time EUR/USD exchange rates
3. **Collateral Verification**: Protocol verifies 101%+ collateralization ratio
4. **QEURO Issuance**: Users receive QEURO at current oracle price minus 0.1% minting fee
5. **Yield Deployment**: USDC is deployed to Aave v3 for yield generation

> **Note**: The MVP only accepts USDC as collateral. Multi-collateral support (ETH, WBTC) is planned for future phases.

```
Minting Transaction Example:
User deposits: 1,000 USDC
EUR/USD rate: 1.10
Minting fee: 0.1% = 1 USDC
Net collateral: 999 USDC
QEURO received: 999 ÷ 1.10 = 908.18 QEURO
```

**⚡ Key Features**

* **Zero slippage**: Minting at oracle rates, no DEX impact
* **Instant settlement**: Single-block transaction finality
* **Open access**: Any address can deposit/redeem via the Vault
* **Rate limiting**: Protection against large-scale manipulation

#### Redemption Process

Redemption operates as the inverse of minting, ensuring users can always exit at fair value:

**📤 Step-by-Step Redemption**

1. **QEURO Submission**: Users submit QEURO for redemption via Vault
2. **Oracle Verification**: Current EUR/USD rate determines USDC value
3. **Collateral Release**: Equivalent USDC withdrawn from Aave
4. **Fee Deduction**: 0.1% redemption fee applied
5. **USDC Transfer**: Net USDC transferred to user wallet

```
Redemption Transaction Example:
User redeems: 900 QEURO
EUR/USD rate: 1.08
USD value: 900 × 1.08 = 972 USDC
Redemption fee: 0.1% = 0.97 USDC
Net USDC received: 971.03 USDC
```

***

### 🎭 Dual-Pool Architecture

The innovative dual-pool system creates natural peg stability through aligned economic incentives.

#### 👥 Users Pool (UserPool Contract)

Users are participants who mint/hold QEURO for euro exposure and yield generation.

**User Motivations**

* **🇪🇺 Native Euro Exposure**: Euro-denominated value without EUR/USD volatility risk
* **📈 Yield Generation**: Earn returns through stQEURO staking
* **🔗 DeFi Access**: Participate in euro-denominated DeFi strategies
* **💼 Treasury Management**: Corporate and institutional euro liquidity

**User Mechanics**

* Deposit USDC via Vault to mint QEURO
* Stake QEURO to stQEURO to earn auto-compounding yield
* Participate in governance through QTI holdings
* Redeem anytime at oracle-determined rates

**Technical Parameters**

| Parameter           | Description              | Default        |
| ------------------- | ------------------------ | -------------- |
| `stakingAPY`        | APY for staked positions | Governance-set |
| `minStakeAmount`    | Minimum stake amount     | Configurable   |
| `unstakingCooldown` | Cooldown before unstake  | Configurable   |
| `performanceFee`    | Fee on yield             | Governance-set |

#### 🛡️ Hedger Pool (HedgerPool Contract)

**MVP: Single Hedger Model**

> **Important**: The MVP implements a **single designated hedger** model for simplified operations. The hedger is assigned via `setSingleHedger()` by governance.

```solidity
// Single hedger address
address public singleHedger;

// Hedger role for operations
bytes32 public constant HEDGER_ROLE = keccak256("HEDGER_ROLE");
```

**Hedger Function**

The designated hedger provides delta-neutral EUR/USD hedging:

* **Position Opening**: Depositer USDC margin to open hedge positions
* **Leverage**: Configurable via `maxLeverage` parameter
* **P\&L Tracking**: Real-time unrealized and realized P\&L calculation
* **Yield Earning**: Receives yield allocation via YieldShift

**Position Management**

```solidity
struct HedgePosition {
    address hedger;           // Hedger address
    uint96 positionSize;      // Position size
    uint96 filledVolume;      // Volume filled by user mints
    uint96 margin;            // Margin deposited
    uint96 entryPrice;        // Entry price (EUR/USD)
    uint64 entryTime;         // Position open time
    uint64 lastUpdateTime;    // Last update timestamp
    int128 unrealizedPnL;     // Current unrealized P&L
    int128 realizedPnL;       // Realized P&L
    uint8 leverage;           // Position leverage
    bool isActive;            // Position status
    uint128 qeuroBacked;      // QEURO backed by this position
}
```

**Compensation Structure**

```
Hedger Revenue Sources:
├── EUR/USD Interest Rate Differential
├── YieldShift allocation (10-50% based on pool ratios)
├── Entry/Exit fees from position management
└── Margin fees (if applicable)
```

**Risk Management**

| Parameter        | Description               | Location   |
| ---------------- | ------------------------- | ---------- |
| `minMarginRatio` | Minimum margin ratio      | CoreParams |
| `maxLeverage`    | Maximum leverage allowed  | CoreParams |
| `entryFee`       | Fee for opening positions | CoreParams |
| `exitFee`        | Fee for closing positions | CoreParams |

**Position Health & Liquidation**

```solidity
// Check if position is healthy
function _isPositionHealthyForFill(HedgePosition memory pos) internal view returns (bool);

// Emergency close by governance
function emergencyClosePosition(address hedger) external onlyRole(EMERGENCY_ROLE);
```

#### 🔄 Pool Interaction Dynamics

```
Users ←→ QuantillonVault ←→ Hedger
  ↓           ↓              ↓
 Euro       USDC           USD
Exposure   Collateral    Risk Mgmt
  ↓           ↓              ↓
stQEURO ←→ YieldShift ←→ Rewards
```

* **Users** get euro exposure via QEURO/stQEURO
* **Hedger** manages EUR/USD risk for yield compensation
* **Protocol** maintains stability through YieldShift incentives

***

### 📊 Overcollateralization Model

Quantillon uses overcollateralization to mitigate forex market volatility.

#### Collateralization Requirements (MVP)

**Minimum Ratios**

| Actor        | Minimum Ratio                 | Liquidation Threshold  |
| ------------ | ----------------------------- | ---------------------- |
| **Protocol** | 101%                          | < 101% triggers alerts |
| **Hedger**   | Configurable (minMarginRatio) | < minMarginRatio       |

**Accepted Collateral (MVP)**

| Asset                  | Status  | Notes              |
| ---------------------- | ------- | ------------------ |
| **USDC**               | ✅ Live  | Primary collateral |
| 🚧 ETH                 | Planned | Phase 2            |
| 🚧 WBTC                | Planned | Phase 2            |
| 🚧 Governance-approved | Planned | Phase 3            |

#### Collateral Management

**Vault Deployment (MVP)**

***

### ⚖️ The Yield Shift Mechanism

The YieldShift represents Quantillon's most innovative feature—a dynamic system that rebalances yield distribution based on pool conditions.

#### Technical Parameters (From Code)

```solidity
uint256 public constant MIN_HOLDING_PERIOD = 7 days;  // Minimum for yield claims
uint256 public constant TWAP_PERIOD = 24 hours;       // Time-weighted average window
uint256 public constant MAX_TIME_ELAPSED = 365 days;

// Configurable parameters
uint256 baseYieldShift = 5000;     // 50% default to users
uint256 maxYieldShift = 9000;      // 90% max to users
uint256 adjustmentSpeed = 100;      // 1% adjustment rate
uint256 targetPoolRatio = 10000;    // 100% target ratio
```

#### Mathematical Foundation

**Distribution Formula**

```
userAllocation = totalYield × (currentYieldShift / 10000)
hedgerAllocation = totalYield - userAllocation

Where:
- currentYieldShift ranges from (10000 - maxYieldShift) to maxYieldShift
- Adjustment based on pool ratios using 24h TWAP
```

**Pool Ratio Calculation**

```solidity
poolRatio = eligibleUserPoolSize × 10000 / eligibleHedgerPoolSize
```

> **Note**: "Eligible" pool sizes exclude recent deposits (< 7 days) to prevent flash deposit manipulation.

#### Dynamic Rebalancing

| Pool Condition              | Current Shift | Direction  | Result               |
| --------------------------- | ------------- | ---------- | -------------------- |
| **User pool > Hedger pool** | High (>50%)   | ↓ Decrease | More yield to hedger |
| **Balanced pools**          | \~50%         | → Stable   | Equal distribution   |
| **Hedger pool > User pool** | Low (<50%)    | ↑ Increase | More yield to users  |

#### Holding Period Protection

```solidity
// Users must hold deposits for 7 days before claiming yield
if (TIME_PROVIDER.currentTime() < lastDepositTime[user] + MIN_HOLDING_PERIOD) {
    revert CommonErrorLibrary.HoldingPeriodNotMet();
}
```

This prevents:

* Flash deposit attacks
* Yield farming manipulation
* Short-term speculation

***

### 🔮 Oracle & Pricing Infrastructure

Accurate, tamper-resistant pricing is critical for all protocol mechanisms.

#### Primary Oracle: ChainlinkOracle Contract

**Price Feeds**

| Feed     | Purpose               | Max Staleness |
| -------- | --------------------- | ------------- |
| EUR/USD  | QEURO peg pricing     | 1 hour        |
| USDC/USD | Collateral validation | 1 hour        |

**Security Parameters**

```solidity
uint256 public constant MAX_PRICE_STALENESS = 3600;   // 1 hour
uint256 public constant MAX_PRICE_DEVIATION = 500;    // 5%
uint256 public constant MAX_TIMESTAMP_DRIFT = 900;    // 15 minutes

// Price bounds (configurable)
uint256 minEurUsdPrice = 0.80e18;   // 0.80 USD/EUR
uint256 maxEurUsdPrice = 1.40e18;   // 1.40 USD/EUR
uint256 usdcToleranceBps = 200;     // 2% USDC tolerance
```

**Circuit Breaker Mechanism**

```
Price Update Flow:
1. Fetch Chainlink data
2. Validate timestamp (< MAX_STALENESS + DRIFT)
3. Validate price bounds (0.80 - 1.40)
4. Check deviation from last price (< 5%)
5. If any check fails → Circuit breaker triggered
6. Protocol uses last valid price as fallback
```

**Emergency Functions**

```solidity
// Manual circuit breaker
function triggerCircuitBreaker() external onlyRole(EMERGENCY_ROLE);

// Reset after incident resolution
function resetCircuitBreaker() external onlyRole(EMERGENCY_ROLE);
```

***

### 🛡️ Risk Management

#### Protocol-Level Controls

**Emergency Hierarchy**

```
Level 1: Rate Limiting (automatic per-address limits)
    ↓
Level 2: Minting Killswitch (stop new mints only)
    ↓
Level 3: Circuit Breaker (oracle fallback mode)
    ↓
Level 4: Full Pause (all operations halted)
```

**Configurable Thresholds (QuantillonVault)**

```solidity
uint256 minCollateralizationRatioForMinting;  // Min ratio to allow mints
uint256 criticalCollateralizationRatio;        // Triggers alerts/restrictions
```

#### Hedger Risk Management

**Position Monitoring**

```solidity
// Check if liquidation should be triggered
function shouldTriggerLiquidation() external view returns (bool);

// Get current liquidation status
function getLiquidationStatus() external view returns (
    bool canLiquidate,
    uint256 shortfall,
    uint256 requiredCollateral
);
```

**Emergency Close**

```solidity
// Force close hedger position in emergency
function emergencyClosePosition(address hedger) external onlyRole(EMERGENCY_ROLE);
```

***

### 🏛️ Governance Integration

All protocol mechanisms operate under decentralized governance via QTI token.

#### Governable Parameters

**Economic Variables**

| Parameter                 | Contract        | Role Required         |
| ------------------------- | --------------- | --------------------- |
| YieldShift base/max/speed | YieldShift      | GOVERNANCE\_ROLE      |
| Mint/redeem fees          | QuantillonVault | GOVERNANCE\_ROLE      |
| Hedger parameters         | HedgerPool      | GOVERNANCE\_ROLE      |
| Oracle bounds             | ChainlinkOracle | ORACLE\_MANAGER\_ROLE |

**Risk Management**

| Parameter                    | Contract        | Role Required        |
| ---------------------------- | --------------- | -------------------- |
| Collateralization thresholds | QuantillonVault | GOVERNANCE\_ROLE     |
| Rate limits                  | QEUROToken      | DEFAULT\_ADMIN\_ROLE |
| Compliance lists             | QEUROToken      | COMPLIANCE\_ROLE     |
| Emergency pause              | All             | EMERGENCY\_ROLE      |

#### QTI Vote-Escrow System

```solidity
// Lock QTI for voting power
function lock(uint256 amount, uint256 duration) external returns (uint256 veQTI);

// Voting power multiplier (up to 4x for max lock)
MAX_LOCK_TIME = 4 years;
MAX_VE_QTI_MULTIPLIER = 4;
```

***

### 🔧 Integration & Composability

#### Protocol Interfaces

**QEURO Token (ERC-20)**

* Full ERC-20 compliance
* Pausable transfers
* Blacklist/whitelist support
* 18 decimals

**stQEURO Token (Yield-Bearing)**

* Exchange rate appreciation model
* Instant stake/unstake
* No rebasing (value appreciation)
* 18 decimals

**Integration Example**

```solidity
// Mint QEURO via Vault
vault.mintQEURO(usdcAmount, recipient);

// Stake QEURO for yield
stQEURO.stake(qeuroAmount);

// Get current exchange rate

```

***

### 📈 Performance Metrics & Monitoring

#### Key Performance Indicators

**Stability Metrics**

* **Peg Maintenance**: QEURO price vs EUR target (< 2% deviation)
* **Collateralization Ratio**: Protocol-wide backing level (> 101%)
* **Oracle Health**: Freshness and accuracy of price feeds

**Efficiency Metrics**

* **Yield Generation**: Aave APY on deployed collateral
* **YieldShift Responsiveness**: Time to rebalance pools
* **Gas Optimization**: Transaction costs for operations

#### Monitoring Functions

```solidity
// QuantillonVault
function getVaultMetrics() external view returns (...);
function getProtocolCollateralizationRatio() external view returns (uint256);

// YieldShift
function getPoolMetrics() external view returns (...);
function getCurrentYieldShift() external view returns (uint256);

// ChainlinkOracle
function getOracleHealth() external view returns (bool, bool, bool);
```

***

> **Quantillon's mechanisms represent a new paradigm in stablecoin design—combining the stability of overcollateralization with the efficiency of delta-neutral hedging and the innovation of dynamic yield distribution. The MVP focuses on core functionality with a single hedger model and Aave v3 integration, with multi-hedger support and additional vault types planned for future phases.**
