# Core Mechanisms

## Mechanisms

Understanding how Quantillon Protocol operates is essential for both users and developers. This guide describes the mechanisms used by the first deployment, QEURO, and shows how the broader protocol turns USD liquidity into local-currency exposure through FX hedging, overcollateralization, and Yield Shift incentives.

> **📋 Scope**: This page is the conceptual overview of the contracts deployed on Base mainnet (public launch planned for Q4 2026). Contract-level detail (signatures, structs, constants, events) lives on the per-contract pages linked from each section - those pages are the reference; this one summarises them.

***

### 🏗️ Protocol Architecture Overview

Quantillon Protocol represents a reusable pattern for local-currency DeFi markets, combining the capital efficiency of overcollateralized systems with the liquidity advantages of forex markets. In the current deployment, these mechanisms are expressed through QEURO as the first EUR deployment.

#### Core Design Principles

* **🔒 Over-collateralization**: Minting requires the protocol collateralization ratio to stay above the governance-set minting floor - currently **102.5%** (lowered from 105% on 2 September 2026; hard minimum 101%); 101% is the critical threshold that triggers liquidation mode
* **⚖️ Delta-neutral hedging**: FX risk managed by a single designated hedger (Quantillon Labs' hedging engine, executing on Hyperliquid) in the current phase
* **📈 Dynamic yield distribution**: YieldShift mechanism for market-responsive incentive alignment
* **🌊 Liquidity inheritance**: Leverages existing USDC liquidity depth
* **🏛️ Progressive decentralization**: Parameters managed today by a 2-of-3 Safe with a 12h upgrade timelock; QTI community governance planned via a future activation upgrade

***

### 💶 QEURO Stablecoin Mechanics

#### Minting Process

The QEURO minting mechanism is designed for simplicity and capital efficiency:

**📥 Step-by-Step Minting**

1. **USDC Deposit**: Users deposit USDC to the QuantillonVault
2. **Oracle Price Check**: the protocol's EUR/USD oracle provides the real-time exchange rate (the hedge venue's market mid, Hyperliquid, with Chainlink as fallback; see [Oracle Architecture](oracle-architecture.md))
3. **Collateral Verification**: Protocol verifies that the collateralization ratio stays at or above the minting floor (currently 102.5%)
4. **QEURO Issuance**: Users receive QEURO at the current oracle price; the minting fee is currently 0 (governance-settable, capped at 5%)
5. **Yield Deployment**: USDC collateral can be deployed to external staking vaults (currently Morpho/MetaMorpho - see [External Staking Vaults](external-staking-vaults.md)) for yield generation

> **Note**: USDC is the sole collateral accepted by the protocol.

```
Minting Transaction Example:
User deposits: 1,000 USDC
EUR/USD rate: 1.10
Minting fee: currently 0 (governance-settable, max 5%)
Net collateral: 1,000 USDC
QEURO received: 1,000 ÷ 1.10 = 909.09 QEURO
```

**⚡ Key Features**

* **Zero slippage**: Minting at oracle rates, no DEX impact (a `minQeuroOut` guard protects against an oracle move between quote and execution)
* **Instant settlement**: Single-block transaction finality
* **Open access**: Any address can deposit/redeem via the Vault
* **Rate limiting**: Global mint and burn cap of 10M QEURO per 300-block window (~10 minutes on Base) against large-scale manipulation

#### Redemption Process

Redemption operates as the inverse of minting, ensuring users can always exit at fair value:

**📤 Step-by-Step Redemption**

1. **QEURO Submission**: Users submit QEURO for redemption via Vault
2. **Oracle Verification**: Current EUR/USD rate determines USDC value
3. **Collateral Release**: Equivalent USDC released from the vault (withdrawn from the external staking vault if deployed)
4. **Fee Deduction**: redemption fee applied - currently 0 (governance-settable, capped at 5%)
5. **USDC Transfer**: Net USDC transferred to user wallet

```
Redemption Transaction Example:
User redeems: 900 QEURO
EUR/USD rate: 1.08
USD value: 900 × 1.08 = 972 USDC
Redemption fee: currently 0 (governance-settable, max 5%)
Net USDC received: 972 USDC
```

When the protocol collateralization ratio is at or below 101%, redemption switches to **liquidation mode** (pro-rata on the remaining collateral) - see [Liquidation Mode](liquidation-mode.md).

***

### 🎭 Dual-Pool Architecture

The dual-pool system creates natural peg stability through aligned economic incentives.

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
* Redeem anytime at oracle-determined rates
* QTI governance is dormant today; parameters are set by the 2-of-3 governance Safe (see [Quantillon DAO](../quantillon-dao.md))

**Technical Parameters (UserPool, live values)**

| Parameter           | Description              | Live value     |
| ------------------- | ------------------------ | -------------- |
| `stakingAPY`        | APY for staked positions | 8% (800 bps)   |
| `depositAPY`        | APY on deposits          | 4% (400 bps)   |
| `minStakeAmount`    | Minimum stake amount     | 100 QEURO      |
| `unstakingCooldown` | Cooldown before unstake  | 7 days         |
| `performanceFee`    | Fee on yield             | 0              |

The UserPool is an optional batch deposit/stake contract; the dApp's primary flows use `QuantillonVault` and the stQEURO vaults directly. Reference: [UserPool](user-pool.md).

#### 🛡️ Hedger Pool (HedgerPool Contract)

**Single Hedger Model**

> **Important**: The protocol runs a **single designated hedger** model. The hedger is the `singleHedger` address set by governance via `setSingleHedger()`; there is no hedger role. In the current phase the hedger is Quantillon Labs' hedging engine, which neutralizes the EUR/USD exposure on Hyperliquid.

**Hedger Function**

The designated hedger provides delta-neutral EUR/USD hedging:

* **Position Opening**: Deposits USDC margin to open a hedge position (`enterHedgePosition(usdcAmount, leverage)`)
* **Leverage**: Governance-set `maxLeverage` - currently 20×
* **P\&L Tracking**: Real-time unrealized and realized P\&L calculation
* **Yield Earning**: Receives yield allocation via YieldShift

**Position Management**

Each position tracks its size, filled volume (backed by user mints), margin, entry price, P\&L, leverage and the QEURO it backs. The full `HedgePosition` struct, the position lifecycle and the vault-synchronization hooks are documented on the [HedgerPool](hedger-pool.md) page.

**Compensation Structure**

```
Hedger Revenue Sources:
├── EUR/USD interest rate differential (currently 3.50% EUR / 4.50% USD, governance-set)
├── Hedger funding carve-out on harvested vault yield
│   (governance-set annual rate, capped at 50% of each harvest -
│    currently 0 bps with no recipient configured)
└── YieldShift allocation layer (base 50%, up to 90% shift
    between the user and hedger yield pools)
```

**Risk Management (live values)**

| Parameter        | Description               | Live value |
| ---------------- | ------------------------- | ---------- |
| `minMarginRatio` | Minimum margin ratio      | 250 bps (2.5%) since 2 September 2026 - contract floor |
| `maxLeverage`    | Maximum leverage allowed  | 20×        |
| `entryFee` / `exitFee` / `marginFee` | Position fees | 0 (governance-settable) |

**Position Health**

Margin cannot be withdrawn below the minimum margin ratio (health gate on `removeMargin`). There is no per-position keeper liquidation: the protocol-level [Liquidation Mode](liquidation-mode.md) is the only liquidation mechanism. The emergency role can force-close a position with `emergencyClosePosition(hedger, positionId)` - see [HedgerPool](hedger-pool.md).

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

#### Collateralization Requirements

**Minimum Ratios**

| Actor        | Minimum Ratio                 | Threshold  |
| ------------ | ----------------------------- | ---------------------- |
| **Protocol** | Governance-set minting floor - currently 102.5% (105% at launch; hard minimum 101%) | ≤ 101% (critical) triggers liquidation mode |
| **Hedger**   | `minMarginRatio` - 250 bps (2.5%) | Margin cannot be withdrawn below it; no per-position liquidation |

**Accepted Collateral**

| Asset                  | Status  | Notes              |
| ---------------------- | ------- | ------------------ |
| **USDC**               | ✅ Live  | Sole collateral    |

#### Collateral Management

USDC collateral is deployed to the registered external staking vault (MetaMorpho USDC, `vaultId` 2) by the vault-operator role and withdrawn automatically to serve redemptions; the protocol collateralization ratio reflects the external vault's current value. See [External Staking Vaults](external-staking-vaults.md).

***

### ⚖️ The Yield Shift Mechanism

The YieldShift represents Quantillon's most innovative feature - a dynamic system that rebalances yield distribution based on pool conditions.

> **How yield flows in the live deployment**: yield is generated by the external staking vault (Morpho) and harvested by `QuantillonVault`. On each harvest, a **hedger funding carve-out** is taken first (governance-set annual rate, capped at 50% of the harvest - currently 0 bps with no recipient configured), and the **residual is split between stQEURO stakers and the treasury** according to the staked share. The YieldShift parameters (base 50%, max 90%) govern the user/hedger allocation layer of the yield pools.

#### Technical Parameters

Base shift 50%, maximum shift 90%, adjustment speed 1%, target pool ratio 100%, 7-day holding period, 24-hour TWAP window. The parameter table and the contract API are on the [YieldShift](yield-shift.md) page.

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

```
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

Deposits must be held for 7 days (`MIN_HOLDING_PERIOD`) before they count toward yield allocation. This prevents:

* Flash deposit attacks
* Yield farming manipulation
* Short-term speculation

***

### 🔮 Oracle & Pricing Infrastructure

Accurate, tamper-resistant pricing is critical for all protocol mechanisms. Pricing is served through an **`OracleRouter`** with two interchangeable EUR/USD sources - see **[Oracle Architecture](oracle-architecture.md)** for the full design.

#### Active source: the hedge venue's EUR/USD market mid (Hyperliquid)

QEURO mint/redeem is priced off the **EUR/USD perpetual mid of the hedge venue, Hyperliquid** - the venue where the protocol's EUR/USD hedge is executed - published on-chain and read through the `HyperliquidEurUsdOracle`, so the on-chain valuation stays aligned with the hedge. Chainlink spot is the governance fallback.

#### Fallback source: ChainlinkOracle

The **ChainlinkOracle** (Chainlink EUR/USD spot) is wired as a one-transaction governance fallback (`switchOracle(0)`) and provides **USDC/USD** collateral validation. Both sources enforce the same validation discipline below.

**Price Feeds**

| Feed     | Source | Purpose               | Max Staleness |
| -------- | ------ | --------------------- | ------------- |
| EUR/USD (active) | Hyperliquid market mid via `HyperliquidEurUsdOracle` | QEURO peg pricing | 15 minutes (900 s; governance-settable up to 1 hour) |
| EUR/USD (fallback) | Chainlink via `ChainlinkOracle` | Governance fallback | 2 hours |
| USDC/USD | Chainlink via `ChainlinkOracle` | Collateral validation | 25 hours (daily heartbeat) |

**Validation discipline (both oracles)**

* Price bounds 0.80–1.40 USD/EUR (governance-configurable)
* 5% maximum deviation between consecutive valid prices (circuit breaker)
* USDC/USD tolerance ±2% around $1.00
* On any rejection the oracle returns the last valid price with `isValid = false`, which the vault treats as a hard stop (mint/redeem revert)
* The Chainlink path additionally checks the Base L2 sequencer-uptime feed (1-hour grace period after a restart)

Constants and functions: [ChainlinkOracle](chainlink-oracle.md) (fallback contract reference) and [Oracle Architecture](oracle-architecture.md) (router, market oracle, publisher, watchdog).

**Emergency Functions**

`triggerCircuitBreaker()` / `resetCircuitBreaker()` (emergency role) force and clear the last-valid-price mode on an oracle; `switchOracle(0)` (oracle-manager role) reverts pricing to Chainlink in one transaction. An independent watchdog freezes mint/redeem (vault pause) if the active price is stale, circuit-broken or diverges from Chainlink.

***

### 🛡️ Risk Management

#### Protocol-Level Controls

**Emergency Hierarchy**

```
Level 1: Rate Limiting (global mint/burn cap of 10M QEURO per 300-block window)
    ↓
Level 2: Minting Killswitch (stop new mints only - PAUSER_ROLE on QEUROToken)
    ↓
Level 3: Circuit Breaker (oracle returns isValid = false → mint/redeem revert;
         governance can switch to the Chainlink fallback)
    ↓
Level 4: Full Pause (all vault operations halted - EMERGENCY_ROLE,
         also triggered autonomously by the independent watchdog)
```

**Configurable Thresholds (QuantillonVault)**

```solidity
uint256 minCollateralizationRatioForMinting;  // currently 102.5% (1.025e20)
uint256 criticalCollateralizationRatio;        // 101% (1.01e20) - liquidation mode
```

**Liquidation-mode checks (QuantillonVault)**

```solidity
function shouldTriggerLiquidation() public view returns (bool shouldLiquidate);
function shouldTriggerLiquidationLive() external returns (bool shouldLiquidate, uint256 collateralizationRatio);
```

See [Liquidation Mode](liquidation-mode.md).

#### Hedger Risk Management

The hedger's margin is bounded by `minMarginRatio` (250 bps) and `maxLeverage` (20×). Since September 2026 the hedge runs on a margin policy targeting 2.5%: the on-chain HedgerPool minimum margin ratio is 2.5% and the minting floor is 102.5%. Quantillon Labs' hedging engine keeps the collateral of the two legs of the hedge - the HedgerPool position on Base and the Hyperliquid perpetual - near a 2.5% equity-to-notional target through bounded, monitored transfers. Details: [HedgerPool - Operational margin policy](hedger-pool.md#operational-margin-policy-september-2026).

**Emergency Close**

```solidity
// Force close a hedger position in an emergency (EMERGENCY_ROLE)
function emergencyClosePosition(address hedger, uint256 positionId) external;
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
| Oracle source switch, price bounds | OracleRouter / oracles | ORACLE\_MANAGER\_ROLE |

**Risk Management**

| Parameter                    | Contract        | Role Required        |
| ---------------------------- | --------------- | -------------------- |
| Collateralization thresholds | QuantillonVault | GOVERNANCE\_ROLE     |
| Rate limits                  | QEUROToken      | DEFAULT\_ADMIN\_ROLE |
| Compliance lists             | QEUROToken      | COMPLIANCE\_ROLE     |
| Minting killswitch           | QEUROToken      | PAUSER\_ROLE         |
| Emergency pause              | All             | EMERGENCY\_ROLE      |

#### QTI Vote-Escrow System

QTI holders will lock tokens for 7 days to 365 days for up to 4× voting power (`lock(amount, lockTime)`); proposal threshold 100k QTI, quorum 1M QTI. The token is deployed but dormant (supply 0) - see [QTI Token](quantillon-protocols-tokens/qti-token.md).

***

### 🔧 Integration & Composability

#### Protocol Interfaces

**QEURO Token (ERC-20)**

* Full ERC-20 compliance
* Pausable transfers
* Blacklist/whitelist support
* 18 decimals

**stQEURO Token (Yield-Bearing, ERC-4626)**

* Exchange rate appreciation model
* Instant deposit/redeem
* No rebasing (value appreciation)
* 18 decimals

**Integration Example**

```solidity
// Mint QEURO via the vault (minQeuroOut = slippage guard)
vault.mintQEURO(usdcAmount, minQeuroOut);

// Stake QEURO into a stQEURO series (ERC-4626)
stQEURO.deposit(qeuroAmount, msg.sender);

// Or mint and stake in one transaction
vault.mintAndStakeQEURO(usdcAmount, minQeuroOut, vaultId, minStQEUROOut);

// Current exchange rate (QEURO per 1 stQEURO)
uint256 rate = stQEURO.convertToAssets(1e18);
```

***

### 📈 Performance Metrics & Monitoring

#### Key Performance Indicators

**Stability Metrics**

* **Peg Maintenance**: QEURO price vs EUR target (< 2% deviation)
* **Collateralization Ratio**: Protocol-wide backing level (above the minting floor - currently 102.5% - for minting; 101% critical)
* **Oracle Health**: Freshness and accuracy of price feeds

**Efficiency Metrics**

* **Yield Generation**: External staking vault (Morpho) APY on deployed collateral
* **YieldShift Responsiveness**: Time to rebalance pools
* **Gas Optimization**: Transaction costs for operations

#### Monitoring Functions

```solidity
// QuantillonVault
function getProtocolCollateralizationRatio() external view returns (uint256 ratio);
function canMint() external view returns (bool);
function getTotalUsdcAvailable() external view returns (uint256);
function getVaultExposure(uint256 vaultId) external view
    returns (address adapter, bool active, uint256 principalTracked, uint256 currentUnderlying);

// YieldShift
function getPoolMetrics() external view
    returns (uint256 userPoolSize, uint256 hedgerPoolSize, uint256 poolRatio, uint256 targetRatio);
function currentYieldShift() external view returns (uint256);

// OracleRouter (delegates to the active oracle)
function getOracleHealth() external returns (bool isHealthy, bool eurUsdFresh, bool usdcUsdFresh);
```

***

> **Quantillon's mechanisms represent a new paradigm in stablecoin design - combining the stability of overcollateralization with the efficiency of delta-neutral hedging and the innovation of dynamic yield distribution. The live deployment runs a single designated hedger and one external staking vault (Morpho/MetaMorpho); additional external vaults can be registered by governance.**
