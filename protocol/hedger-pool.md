# HedgerPool

## HedgerPool: EUR/USD Hedging & Collateral Management

### 📋 Overview

The HedgerPool is the contract responsible for managing EUR/USD hedging positions in the Quantillon protocol. It allows the designated hedger to provide delta-neutral coverage, maintaining QEURO peg stability while generating revenue.

> **Single hedger model**: the protocol runs a **single designated hedger** — the `singleHedger` address set by governance. In the current phase that hedger is Quantillon Labs' hedging engine, which neutralizes the EUR/USD exposure on Hyperliquid (see [Oracle Architecture](oracle-architecture.md) for why mint/redeem pricing follows the hedge venue). Hedger USDC is pooled in `QuantillonVault` for unified liquidity.

***

### 🏗️ Contract Architecture

**Inheritance**

```solidity
contract HedgerPool is 
    Initializable,
    ReentrancyGuardUpgradeable,
    AccessControlUpgradeable,
    PausableUpgradeable,
    SecureUpgradeable
```

`SecureUpgradeable` routes upgrades through the 12-hour OZ TimelockController (see [Quantillon DAO](../quantillon-dao.md)). Live version: **1.0.8** (since 2 September 2026).

**External Dependencies**

| Contract | Role |
|----------|------|
| `IERC20 usdc` | Collateral token |
| `IOracle oracle` | EUR/USD price — the `OracleRouter` (Hyperliquid market mid, Chainlink fallback) |
| `IYieldShift yieldShift` | Hedger-side yield ledger |
| `IQuantillonVault vault` | Mint/redeem synchronization and pooled hedger USDC |
| `address feeCollector` | Receives position fees (currently 0) |
| `address treasury` | Receives recovered tokens |
| `TimeProvider TIME_PROVIDER` | Time management (testability) |

***

### 🔐 Roles & Permissions

| Role | Responsibilities |
|------|-----------------|
| `DEFAULT_ADMIN_ROLE` | General administration, role assignment, token recovery, `feeCollector` change |
| `GOVERNANCE_ROLE` | `configureRiskAndFees`, `configureDependencies`, `setSingleHedger` |
| `EMERGENCY_ROLE` | `emergencyClosePosition`, `pause` / `unpause` |
| `UPGRADER_ROLE` | Upgrade execution (timelock-gated) |

There is **no hedger role**: hedger-only functions check `msg.sender == singleHedger` and revert with `NotAuthorized` otherwise.

**`onlyVault` Modifier**

```solidity
modifier onlyVault() {
    if (msg.sender != address(vault)) revert HedgerPoolErrorLibrary.OnlyVault();
    _;
}
```

The mint/redeem synchronization hooks can only be called by the Vault.

***

### 📊 Single Hedger Model

**Configuration**

```solidity
address public singleHedger;  // Designated hedger address

function setSingleHedger(address hedger) external;  // GOVERNANCE_ROLE
```

**Assignment Flow**

```
1. Governance calls setSingleHedger(hedgerAddress)
2. Reverts with HedgerHasActivePosition while the current position is open
3. The singleHedger address is rotated (SingleHedgerRotationApplied)
4. Only the new address can open positions and claim rewards
```

***

### 📈 Position Structure

**HedgePosition**

```solidity
struct HedgePosition {
    address hedger;           // Hedger address
    uint96 positionSize;      // Position size (total exposure)
    uint96 filledVolume;      // Volume filled by user mints
    uint96 margin;            // USDC margin deposited
    uint96 entryPrice;        // Average entry price (EUR/USD)
    uint32 entryTime;         // Opening timestamp
    uint32 lastUpdateTime;    // Last update
    int128 unrealizedPnL;     // Unrealized P&L
    int128 realizedPnL;       // Cumulative realized P&L from closed portions
    uint16 leverage;          // Leverage used
    bool isActive;            // Position active
    uint128 qeuroBacked;      // Exact QEURO amount backed by this position (18 decimals)
    uint64 openBlock;         // Block number when opened (min hold period)
}
```

**HedgerRewardState**

```solidity
struct HedgerRewardState {
    uint128 pendingRewards;    // Pending rewards
    uint64 lastRewardClaim;    // Last claim
}
```

***

### 💰 Position Management

#### Open a Position

```solidity
function enterHedgePosition(uint256 usdcAmount, uint256 leverage)
    external whenNotPaused nonReentrant returns (uint256 positionId);
// caller must be singleHedger
```

**Validations**

1. Caller is `singleHedger`
2. `usdcAmount >= minMarginAmount` (currently 0)
3. `leverage <= coreParams.maxLeverage` (currently 20)
4. The hedger has no active position

**Flow**

```
1. Hedger deposits USDC (margin); the entry fee (currently 0) is deducted
2. positionSize = net margin × leverage; entryPrice = current oracle price
3. Position marked active (openBlock recorded)
4. USDC transferred to the Vault for unified liquidity (addHedgerDeposit)
```

#### Close a Position

```solidity
function exitHedgePosition(uint256 positionId)
    external whenNotPaused nonReentrant returns (int256 pnl);
// caller must be singleHedger
```

**Validations**

1. Active position owned by the caller
2. `minPositionHoldBlocks` elapsed since `openBlock` (currently 0 blocks) — otherwise `MinHoldPeriodNotElapsed`
3. Closure must not leave the protocol under-collateralized — otherwise `PositionClosureRestricted`

**Flow**

```
1. Final P&L calculation
2. Return margin ± P&L to hedger (exit fee currently 0)
3. Position marked inactive
4. Residual rewards claimable
```

#### Add Margin

```solidity
function addMargin(uint256 positionId, uint256 amount)
    external whenNotPaused nonReentrant;
```

Allows the hedger to improve the margin ratio without closing the position (margin fee currently 0).

#### Remove Margin

```solidity
function removeMargin(uint256 positionId, uint256 amount)
    external whenNotPaused nonReentrant;
```

**Condition**: `removeMargin` reverts with `InsufficientMargin` if the effective margin ratio (margin ± unrealized P&L over the filled notional, at a fresh oracle price) would fall below `minMarginRatio`. This health gate is the only per-position margin enforcement: there is no keeper liquidation of hedger positions.

***

### 📊 P&L Calculation

#### Unrealized P&L

```solidity
// Simplified: P&L based on EUR/USD price variation
unrealizedPnL = filledVolume × (currentPrice - entryPrice) / entryPrice
```

| EUR/USD Movement | Hedger Impact |
|------------------|---------------|
| EUR ↑ vs USD | Negative P&L |
| EUR ↓ vs USD | Positive P&L |

#### Realized P&L

P&L is realized during QEURO redemptions:

```solidity
function recordUserRedeem(uint256 usdcAmount, uint256 redeemPrice, uint256 qeuroAmount)
    external onlyVault whenNotPaused;
```

**Redemption Flow**

```
1. User redeems QEURO via Vault
2. Vault calls recordUserRedeem
3. HedgerPool calculates portion of filled volume to close
4. Proportional P&L is realized
5. filledVolume and qeuroBacked decrease
```

**Formula**

```solidity
// Closed portion
closedPortion = qeuroAmount / totalQeuroBacked

// Realized P&L on this portion  
realizedPnLPortion = closedPortion × (redemptionPrice - entryPrice) × filledVolume
```

***

### 🔄 Vault Synchronization

The HedgerPool is synchronized with Vault mint/redeem operations.

#### During a Mint

```solidity
function recordUserMint(uint256 usdcAmount, uint256 fillPrice, uint256 qeuroAmount)
    external onlyVault whenNotPaused;
```

**Actions**

1. Increases `filledVolume` of the active position
2. Updates `entryPrice` (weighted average)
3. Increases `qeuroBacked`

#### During a Redeem

```solidity
function recordUserRedeem(uint256 usdcAmount, uint256 redeemPrice, uint256 qeuroAmount)
    external onlyVault whenNotPaused;
```

**Actions**

1. Decreases `filledVolume` proportionally
2. Realizes P&L on the closed portion
3. Decreases `qeuroBacked`

#### During a Liquidation

```solidity
function recordLiquidationRedeem(uint256 qeuroAmount, uint256 totalQeuroSupply)
    external onlyVault whenNotPaused;
```

Called when the vault is in liquidation mode (protocol collateralization ratio ≤ 101%). In that mode the hedger's effective margin is treated as 0 and redemptions draw pro-rata on remaining collateral — there is no per-position keeper liquidation. See [Liquidation Mode](liquidation-mode.md).

***

### 💸 Reward Distribution

#### Hedger Revenue Sources

1. **Hedger Funding**: a carve-out taken first out of each external-vault yield harvest (governance-set annual rate, capped at 50% of the harvest), accounted through YieldShift's hedger ledger — **currently 0 bps with no recipient configured**: Quantillon Labs, the sole hedger, receives no funding carve-out today
2. **EUR/USD Rate Differential**: compensation for FX risk (currently 3.50% EUR / 4.50% USD, governance-set)
3. **Position Fees**: entry/exit/margin fees are currently 0 (governance-settable)

> **Reward fee split**: `rewardFeeSplit` (currently 20%, i.e. `2e17` of `1e18`) is the share of protocol fees routed to the local reward reserve. Anyone can top the reserve up with `fundRewardReserve(amount)` (`RewardReserveFunded`).

#### Claim Rewards

```solidity
function claimHedgingRewards()
    external whenNotPaused nonReentrant
    returns (uint256 interestDifferential, uint256 yieldShiftRewards, uint256 totalRewards);
// caller must be singleHedger

// Fallback if the direct USDC transfer failed (e.g. recipient blacklisted by USDC)
function withdrawPendingRewards(address recipient) external nonReentrant;
```

**Constraints**

```solidity
uint256 public constant MAX_REWARD_PERIOD = 365 days;
// Accumulated rewards cannot exceed 1 year of calculation
```

***

### ⚙️ Configurable Parameters

#### CoreParams

```solidity
struct CoreParams {
    uint64 minMarginRatio;     // Minimum margin ratio (BPS)
    uint16 maxLeverage;        // Maximum allowed leverage
    uint16 entryFee;           // Entry fee (BPS)
    uint16 exitFee;            // Exit fee (BPS)
    uint16 marginFee;          // Margin fee (BPS)
    uint16 eurInterestRate;    // EUR interest rate (BPS)
    uint16 usdInterestRate;    // USD interest rate (BPS)
    uint8 reserved;
}
```

**Live Values** (verified on-chain, 4 September 2026)

| Parameter | Live Value | Description |
|-----------|------------|-------------|
| `minMarginRatio` | **250 bps (2.5%)** — live since 2 September 2026 (HedgerPool v1.0.8); was 500 bps at launch. Governance-set, cannot go below the 250 bps contract floor (`DEFAULT_MIN_MARGIN_RATIO_BPS`) | Minimum margin/position ratio |
| `maxLeverage` | 20 | Max 20× leverage (the setter caps it at 20) |
| `entryFee` | 0 | Currently 0 (governance-settable) |
| `exitFee` | 0 | Currently 0 (governance-settable) |
| `marginFee` | 0 | Currently 0 (governance-settable) |
| `eurInterestRate` | 350 (3.50%) | EUR leg interest rate (max 2000) |
| `usdInterestRate` | 450 (4.50%) | USD leg interest rate (max 2000) |
| `minMarginAmount` | 0 | Minimum margin per position (governance-set; initializer default 100 USDC) |
| `minPositionHoldBlocks` | 0 | Minimum blocks before a position can be closed (governance-set; initializer default 5) |
| `rewardFeeSplit` | 20% (`2e17`) | Share of protocol fees routed to the reward reserve (max `1e18`) |

#### Configuration Functions

```solidity
struct HedgerRiskConfig {
    uint256 minMarginRatio;        // >= DEFAULT_MIN_MARGIN_RATIO_BPS (250)
    uint256 maxLeverage;           // <= 20
    uint256 minPositionHoldBlocks;
    uint256 minMarginAmount;
    uint256 eurInterestRate;       // <= 2000 bps
    uint256 usdInterestRate;       // <= 2000 bps
    uint256 entryFee;
    uint256 exitFee;
    uint256 marginFee;
    uint256 rewardFeeSplit;        // <= MAX_REWARD_FEE_SPLIT (1e18)
}

struct HedgerDependencyConfig {
    address treasury;
    address vault;
    address oracle;
    address yieldShift;
    address feeCollector;          // changing it requires DEFAULT_ADMIN_ROLE
}

// Risk parameters, fees and interest rates — GOVERNANCE_ROLE
function configureRiskAndFees(HedgerRiskConfig calldata cfg) external;

// Contract dependencies — GOVERNANCE_ROLE
function configureDependencies(HedgerDependencyConfig calldata cfg) external;
```

***

### ⚖️ Operational margin policy (September 2026)

Since September 2026 the hedge runs on a **margin policy targeting 2.5%**: the on-chain HedgerPool minimum margin ratio is 2.5% (250 bps, v1.0.8) and the QuantillonVault minting floor is 102.5%. Quantillon Labs' hedging engine keeps the collateral of the two legs of the hedge — the HedgerPool position on Base (short EUR) and the Hyperliquid perpetual (long EUR) — near a 2.5% equity-to-notional target, moving USDC from the HedgerPool to Hyperliquid in bounded steps (25 bps of notional) when EUR/USD falls, and topping the HedgerPool up with fresh USDC when EUR/USD rises. Transfers are bounded in size, executed one at a time, and are blocked whenever they would push the protocol collateralization ratio too close to the minting floor. The 250 bps on-chain minimum is a hard floor that the engine operates above; the independent watchdog freezes mint/redeem if the hedge becomes unhealthy (see [Oracle Architecture](oracle-architecture.md#independent-watchdog-defence-in-depth)).

***

### 🛡️ Security & Emergency Controls

#### Health Checks

```solidity
// Checks if position can support a fill
function _isPositionHealthyForFill(HedgePosition memory pos) 
    internal view returns (bool);

// Margin removal is rejected if the position would become unhealthy
function _validatePositionHealthAfterMarginRemoval(HedgePosition storage pos, uint256 newMargin)
    private;
```

#### Emergency Close

```solidity
function emergencyClosePosition(address hedger, uint256 positionId)
    external nonReentrant;  // EMERGENCY_ROLE
// emits EmergencyPositionClosed(hedger, positionId, marginWithdrawn, outstandingQeuro)
```

Allows the emergency role to close a position if the hedger is non-responsive, the position becomes dangerous for the protocol, or a vulnerability is detected.

> ⚠️ An emergency close while QEURO is still backed by the position (`outstandingQeuro > 0`) removes that backing and reduces the protocol collateralization ratio. It is a last resort, not a routine operation.

#### Pause

```solidity
function pause() external;    // EMERGENCY_ROLE
function unpause() external;  // EMERGENCY_ROLE
```

#### Recovery

```solidity
function recover(address token, uint256 amount) external;  // DEFAULT_ADMIN_ROLE
```

Recovers tokens sent by mistake to the treasury (cannot recover active USDC).

***

### 📏 Constants & Limits

```solidity
// Position limits
uint256 public constant MAX_POSITION_SIZE = type(uint96).max;
uint256 public constant MAX_MARGIN = type(uint96).max;
uint256 public constant MAX_ENTRY_PRICE = type(uint96).max;
uint256 public constant MAX_LEVERAGE = type(uint16).max;   // storage bound; governance can configure at most 20x
uint256 public constant MAX_MARGIN_RATIO = 5000;           // 50% maximum margin ratio
uint256 public constant DEFAULT_MIN_MARGIN_RATIO_BPS = 250; // 2.5% governance floor

// Global limits
uint256 public constant MAX_TOTAL_MARGIN = type(uint128).max;
uint256 public constant MAX_TOTAL_EXPOSURE = type(uint128).max;

// Rewards
uint256 public constant MAX_REWARD_PERIOD = 365 days;
uint256 public constant MAX_REWARD_FEE_SPLIT = 1e18;

// Dust
uint256 public constant QEURO_DUST_THRESHOLD = 1e12;
```

Maximum leverage governance can configure: **20×** (`configureRiskAndFees` reverts above it); live `coreParams.maxLeverage` = 20.

***

### 📊 View Functions

#### Collateral Metrics

```solidity
// Total effective hedger collateral (margin ± P&L) at a given EUR/USD price
function getTotalEffectiveHedgerCollateral(uint256 price) external view returns (uint256);

// Is there an active hedger?
function hasActiveHedger() external view returns (bool);

// Aggregates
function totalMargin() external view returns (uint256);
function totalExposure() external view returns (uint256);
function totalFilledExposure() external view returns (uint256);
```

#### Position Data & Configuration

```solidity
// Get a position by ID
mapping(uint256 => HedgePosition) public positions;

// Configuration getters
function singleHedger() external view returns (address);
function coreParams() external view returns (CoreParams memory);
function minMarginAmount() external view returns (uint256);
function minPositionHoldBlocks() external view returns (uint256);
function rewardFeeSplit() external view returns (uint256);
function pendingRewardWithdrawals(address hedger) external view returns (uint256);
function version() external pure returns (string memory);  // "1.0.8"
```

The hedger → active-position mapping (`hedgerActivePositionId`) is private; read the position with `positions(1)` while the single-hedger model holds.

***

### 🔗 YieldShift Integration

The HedgerPool integrates with YieldShift for yield distribution:

```solidity
IYieldShift public yieldShift;
```

**Distribution Flow**

```
External staking vault (Morpho) generates yield
    ↓
QuantillonVault harvests; a hedger funding carve-out is taken first
(governance-set annual rate, capped at 50% of each harvest — currently 0 bps)
    ↓
YieldShift tracks the hedger's claimable share
    ↓
Hedger claims via claimHedgingRewards()
```

***

### 📋 Events

```solidity
event HedgePositionOpened(address indexed hedger, uint256 indexed positionId, bytes32 packedData);
event HedgePositionClosed(address indexed hedger, uint256 indexed positionId, bytes32 packedData);
event MarginUpdated(address indexed hedger, uint256 indexed positionId, bytes32 packedData);
event SingleHedgerRotationApplied(address indexed previousHedger, address indexed newHedger);
event EmergencyPositionClosed(address indexed hedger, uint256 indexed positionId, uint256 marginWithdrawn, uint256 outstandingQeuro);
event RewardReserveFunded(address indexed funder, uint256 amount);
event ETHRecovered(address indexed to, uint256 indexed amount);
```

`packedData` packs the position size, margin, price and P&L fields of the event into one word (decoded by the indexer). Upgrade-related events (`SecureUpgradeAuthorized`, `SecureUpgradesToggled`, `TimelockSet`, `EmergencyDisableProposed/Approved`) come from `SecureUpgradeable`.

***

### ⚠️ Custom Errors (`HedgerPoolErrorLibrary`)

```solidity
error InvalidPosition();
error InvalidHedger();
error OnlyVault();
error InsufficientMargin();
error MarginRatioTooLow();
error MarginRatioTooHigh();
error LeverageTooHigh();
error InvalidLeverage();
error HedgerHasActivePosition();
error NoActiveHedgerLiquidity();
error MinHoldPeriodNotElapsed();
error PositionClosureRestricted();
error PositionOwnerMismatch();
error InsufficientHedgerCapacity();
error MarginExceedsMaximum();
error NewMarginExceedsMaximum();
error PositionSizeExceedsMaximum();
error EntryPriceExceedsMaximum();
error LeverageExceedsMaximum();
error TotalMarginExceedsMaximum();
error TotalExposureExceedsMaximum();
error RewardOverflow();
error TimestampOverflow();
error FlashLoanAttackDetected();
```

Generic reverts (`NotAuthorized`, `InvalidAmount`, `ConfigValueTooHigh/TooLow`, `InvalidOraclePrice`, …) come from `CommonErrorLibrary`.

***

> **Summary**: The HedgerPool is the core of Quantillon's hedging mechanics. A single designated hedger — Quantillon Labs' hedging engine in the current phase — provides EUR/USD coverage, earning revenue via the rate differential and YieldShift. The margin health gate, the September 2026 margin policy and the emergency controls keep the hedge and the protocol collateralization aligned.
