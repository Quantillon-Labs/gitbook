# Core Mechanisms

## Mechanisms

Understanding how Quantillon Protocol operates is essential for both users and developers. This guide describes the mechanisms used by the first deployment, QEURO, and shows how the broader protocol turns USD liquidity into local-currency exposure through FX hedging, overcollateralization, and Yield Shift incentives.

> **📋 MVP Status**: This documentation reflects the current MVP implementation. Features marked with 🚧 are planned for future phases.

***

### 🏗️ Protocol Architecture Overview

Quantillon Protocol represents a reusable pattern for local-currency DeFi markets, combining the capital efficiency of overcollateralized systems with the liquidity advantages of forex markets. In the current MVP, these mechanisms are expressed through QEURO as the first EUR deployment.

#### Core Design Principles

* **🔒 Over-collateralization**: Minting requires a protocol collateralization ratio of at least 105%; 101% is the critical threshold that triggers liquidation mode
* **⚖️ Delta-neutral hedging**: FX risk managed by designated hedger (MVP: single hedger)
* **📈 Dynamic yield distribution**: YieldShift mechanism for market-responsive incentive alignment
* **🌊 Liquidity inheritance**: Leverages existing USDC liquidity depth
* **🏛️ Progressive decentralization**: Parameters managed today by a 2-of-3 Safe with a 12h upgrade timelock; QTI community governance planned via a future activation upgrade

***

### 💶 QEURO Stablecoin Mechanics

#### Minting Process (MVP Implementation)

The QEURO minting mechanism is designed for simplicity and capital efficiency:

**📥 Step-by-Step Minting**

1. **USDC Deposit**: Users deposit USDC to the QuantillonVault
2. **Oracle Price Check**: the protocol's EUR/USD oracle provides the real-time exchange rate (the Hyperliquid market mid by default, with Chainlink as fallback — see [Oracle Architecture](oracle-architecture.md))
3. **Collateral Verification**: Protocol verifies that the collateralization ratio stays at or above the 105% minting floor
4. **QEURO Issuance**: Users receive QEURO at the current oracle price; the minting fee is currently 0 (governance-settable, capped at 5%)
5. **Yield Deployment**: USDC collateral can be deployed to external staking vaults (currently Morpho/MetaMorpho — see [External Staking Vaults](aave-vault.md)) for yield generation

> **Note**: The MVP only accepts USDC as collateral. Multi-collateral support (ETH, WBTC) is planned for future phases.

```
Minting Transaction Example:
User deposits: 1,000 USDC
EUR/USD rate: 1.10
Minting fee: currently 0 (governance-settable, max 5%)
Net collateral: 1,000 USDC
QEURO received: 1,000 ÷ 1.10 = 909.09 QEURO
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
3. **Collateral Release**: Equivalent USDC released from the vault (withdrawn from the external staking vault if deployed)
4. **Fee Deduction**: redemption fee applied — currently 0 (governance-settable, capped at 5%)
5. **USDC Transfer**: Net USDC transferred to user wallet

```
Redemption Transaction Example:
User redeems: 900 QEURO
EUR/USD rate: 1.08
USD value: 900 × 1.08 = 972 USDC
Redemption fee: currently 0 (governance-settable, max 5%)
Net USDC received: 972 USDC
```

***

### 🎭 Dual-Pool Architecture

The innovative dual-pool system creates natural peg stability through aligned economic incentives.

#### 👥 Users Pool (UserPool Contract)

Users are participants who mint/hold QEURO for EUR exposure and yield generation in the first deployment.

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
├── EUR/USD interest rate differential (currently 3.50% EUR / 4.50% USD, governance-set)
├── Hedger funding paid first out of harvested vault yield
│   (governance-set annual rate, capped at 50% of each harvest)
└── YieldShift allocation layer (base 50%, up to 90% shift
    between the user and hedger yield pools)
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
| **Protocol** | 105% (minting floor)          | ≤ 101% (critical) triggers liquidation mode |
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

> **How yield flows in the live deployment**: yield is generated by the external staking vault (Morpho) and harvested by `QuantillonVault`. On each harvest, **hedger funding is paid first** (a governance-set annual rate, capped at 50% of the harvest), and the **residual is split between stQEURO stakers and the treasury** according to the staked share. The YieldShift parameters below (base 50%, max 90%) govern the user/hedger allocation layer of the yield pools.

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

Accurate, tamper-resistant pricing is critical for all protocol mechanisms. Pricing is served through an **`OracleRouter`** with two interchangeable EUR/USD sources — see **[Oracle Architecture](oracle-architecture.md)** for the full design.

#### Active source: Hyperliquid EUR/USD market mid

QEURO mint/redeem is priced off the **Hyperliquid `EUR` perpetual mid** — the venue where the protocol's EUR/USD hedge is executed — published on-chain and read through the `HyperliquidEurUsdOracle`, so the on-chain valuation stays aligned with the hedge.

#### Fallback source: ChainlinkOracle

The **ChainlinkOracle** (Chainlink EUR/USD spot) is wired as a one-transaction governance fallback (`switchOracle(0)`) and provides **USDC/USD** collateral validation. Both sources enforce the same validation discipline below.

**Price Feeds**

| Feed     | Purpose               | Max Staleness |
| -------- | --------------------- | ------------- |
| EUR/USD  | QEURO peg pricing     | 2 hours       |
| USDC/USD | Collateral validation | 25 hours (daily heartbeat) |

**Security Parameters**

```solidity
uint256 public constant MAX_PRICE_STALENESS = 2 hours;       // EUR/USD
uint256 public constant MAX_USDC_PRICE_STALENESS = 25 hours; // USDC/USD
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

Protocol parameters are currently governed by a 2-of-3 Gnosis Safe, with core-contract upgrades routed through a 12-hour timelock (see [Quantillon DAO](../quantillon-dao.md)). QTI vote-escrow governance exists in the contracts but is dormant until a future activation upgrade.

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
MIN_LOCK_TIME = 7 days;
MAX_LOCK_TIME = 365 days;
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
* **Collateralization Ratio**: Protocol-wide backing level (≥ 105% required for minting; 101% critical)
* **Oracle Health**: Freshness and accuracy of price feeds

**Efficiency Metrics**

* **Yield Generation**: External staking vault (Morpho) APY on deployed collateral
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

> **Quantillon's mechanisms represent a new paradigm in stablecoin design—combining the stability of overcollateralization with the efficiency of delta-neutral hedging and the innovation of dynamic yield distribution. The live deployment focuses on core functionality with a single hedger model and external staking vaults (currently Morpho/MetaMorpho), with multi-hedger support and additional vault types planned for future phases.**
