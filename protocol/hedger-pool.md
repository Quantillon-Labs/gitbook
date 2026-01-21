# HedgerPool

## HedgerPool: EUR/USD Hedging & Collateral Management

### 📋 Overview

The HedgerPool is the contract responsible for managing EUR/USD hedging positions in the Quantillon protocol. It allows the designated hedger to provide delta-neutral coverage, maintaining QEURO peg stability while generating revenue.

> **MVP Implementation**: The MVP uses a **single hedger model** to simplify operations. Multi-hedger support is planned for future phases.

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

**External Dependencies**

| Contract | Role |
|----------|------|
| `IERC20 usdc` | Collateral token |
| `IChainlinkOracle oracle` | Real-time EUR/USD price |
| `IYieldShift yieldShift` | Yield distribution |
| `IQuantillonVault vault` | Mint/redeem synchronization |
| `ITimeProvider TIME_PROVIDER` | Time management (testability) |

***

### 🔐 Roles & Permissions

| Role | Responsibilities |
|------|-----------------|
| `DEFAULT_ADMIN_ROLE` | General administration, role assignment |
| `GOVERNANCE_ROLE` | Pool parameter modification |
| `EMERGENCY_ROLE` | Emergency position closure |
| `HEDGER_ROLE` | Hedging operations (assigned to single hedger) |

**`onlyVault` Modifier**

```solidity
modifier onlyVault() {
    if (msg.sender != address(vault)) {
        revert CommonErrorLibrary.OnlyVault();
    }
    _;
}
```

Certain functions can only be called by the Vault during mint/redeem operations.

***

### 📊 Single Hedger Model

**Configuration**

```solidity
address public singleHedger;  // Designated hedger address

function setSingleHedger(address _hedger) external onlyRole(GOVERNANCE_ROLE);
```

**Assignment Flow**

```
1. Governance calls setSingleHedger(hedgerAddress)
2. Old hedger loses HEDGER_ROLE
3. New hedger receives HEDGER_ROLE
4. Existing position must be closed before change
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
    uint64 entryTime;         // Opening timestamp
    uint64 lastUpdateTime;    // Last update
    int128 unrealizedPnL;     // Unrealized P&L
    int128 realizedPnL;       // Realized P&L
    uint8 leverage;           // Leverage used
    bool isActive;            // Position active
    uint128 qeuroBacked;      // QEURO backed by this position
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
function enterHedgePosition(
    uint96 positionSize,
    uint96 margin,
    uint8 leverage
) external onlyRole(HEDGER_ROLE) whenNotPaused nonReentrant
    returns (uint256 positionId);
```

**Validations**

1. Hedger doesn't already have an active position
2. `positionSize > 0` and `margin > 0`
3. `leverage <= maxLeverage`
4. Sufficient margin ratio: `margin × leverage >= positionSize × minMarginRatio`

**Flow**

```
1. Hedger deposits USDC (margin)
2. Position created with entryPrice = current oracle price
3. Position marked active
4. USDC transferred to Vault for unified liquidity
```

#### Close a Position

```solidity
function exitHedgePosition() 
    external onlyRole(HEDGER_ROLE) whenNotPaused nonReentrant;
```

**Validations**

1. Active position exists
2. `filledVolume == 0` (no more backed QEURO)
3. Position "safe to close" (no debt to users)

**Flow**

```
1. Final P&L calculation
2. Return margin ± P&L to hedger
3. Position marked inactive
4. Residual rewards claimable
```

#### Add Margin

```solidity
function addMargin(uint96 amount) 
    external onlyRole(HEDGER_ROLE) whenNotPaused nonReentrant;
```

Allows the hedger to improve margin ratio without closing the position.

#### Remove Margin

```solidity
function removeMargin(uint96 amount) 
    external onlyRole(HEDGER_ROLE) whenNotPaused nonReentrant;
```

**Condition**: Margin ratio must remain above `minMarginRatio` after withdrawal.

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
function recordUserRedeem(uint256 qeuroAmount, uint256 usdcAmount) 
    external onlyVault;
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
function recordUserMint(uint256 qeuroAmount, uint256 usdcAmount, uint256 price) 
    external onlyVault;
```

**Actions**

1. Increases `filledVolume` of active position
2. Updates `entryPrice` (weighted average)
3. Increases `qeuroBacked`

#### During a Redeem

```solidity
function recordUserRedeem(uint256 qeuroAmount, uint256 usdcAmount) 
    external onlyVault;
```

**Actions**

1. Decreases `filledVolume` proportionally
2. Realizes P&L on closed portion
3. Decreases `qeuroBacked`

#### During a Liquidation

```solidity
function recordLiquidationRedeem(uint256 qeuroAmount, uint256 usdcAmount) 
    external onlyVault;
```

Similar to standard redeem, but may apply special conditions.

***

### 💸 Reward Distribution

#### Hedger Revenue Sources

1. **YieldShift Allocation**: Share of Aave yield (10-50% depending on ratios)
2. **EUR/USD Rate Differential**: Compensation for FX risk
3. **Position Fees**: Entry/exit fees if configured

#### Claim Rewards

```solidity
function claimHedgingRewards() 
    external onlyRole(HEDGER_ROLE) whenNotPaused nonReentrant;
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
    uint16 minMarginRatio;     // Minimum margin ratio (BPS)
    uint8 maxLeverage;         // Maximum allowed leverage
    uint16 entryFee;           // Entry fee (BPS)
    uint16 exitFee;            // Exit fee (BPS)
    uint16 marginFee;          // Margin fee (BPS)
    uint16 eurInterestRate;    // EUR interest rate (BPS)
    uint16 usdInterestRate;    // USD interest rate (BPS)
}
```

**Defaults**

| Parameter | Default Value | Description |
|-----------|---------------|-------------|
| `minMarginRatio` | 1000 (10%) | Minimum margin/position ratio |
| `maxLeverage` | 10 | Max 10x leverage |
| `entryFee` | 0 | No entry fee |
| `exitFee` | 0 | No exit fee |

#### Configuration Functions

```solidity
// General parameters
function updateHedgingParameters(uint16 minMargin, uint8 maxLev) 
    external onlyRole(GOVERNANCE_ROLE);

// Interest rates
function updateInterestRates(uint16 eurRate, uint16 usdRate) 
    external onlyRole(GOVERNANCE_ROLE);

// Fees
function setHedgingFees(uint16 entry, uint16 exit, uint16 margin) 
    external onlyRole(GOVERNANCE_ROLE);
```

***

### 🛡️ Security & Emergency Controls

#### Health Checks

```solidity
// Checks if position can support a fill
function _isPositionHealthyForFill(HedgePosition memory pos) 
    internal view returns (bool);

// Checks if closure is safe
function _validatePositionClosureSafety(HedgePosition storage pos) 
    internal view;
```

#### Emergency Close

```solidity
function emergencyClosePosition(address hedger) 
    external onlyRole(EMERGENCY_ROLE);
```

Allows the emergency team to close a position if:
- Hedger is non-responsive
- Position becomes dangerous for the protocol
- A vulnerability is detected

#### Pause

```solidity
function pause() external onlyRole(EMERGENCY_ROLE);
function unpause() external onlyRole(EMERGENCY_ROLE);
```

#### Recovery

```solidity
function recover(address token, uint256 amount) 
    external onlyRole(DEFAULT_ADMIN_ROLE);
```

Recovers tokens sent by mistake (cannot recover active USDC).

***

### 📏 Constants & Limits

```solidity
// Position limits
uint96 public constant MAX_POSITION_SIZE = type(uint96).max;
uint96 public constant MAX_MARGIN = type(uint96).max;
uint96 public constant MAX_ENTRY_PRICE = type(uint96).max;
uint8 public constant MAX_LEVERAGE = 10;
uint16 public constant MAX_MARGIN_RATIO = 10000;  // 100%

// Global limits
uint128 public constant MAX_TOTAL_MARGIN = type(uint128).max;
uint128 public constant MAX_TOTAL_EXPOSURE = type(uint128).max;

// Time limits
uint256 public constant MAX_REWARD_PERIOD = 365 days;
```

***

### 📊 View Functions

#### Collateral Metrics

```solidity
// Total effective hedger collateral (margin + P&L)
function getTotalEffectiveHedgerCollateral() 
    external view returns (uint256);

// Is there an active hedger?
function hasActiveHedger() external view returns (bool);
```

#### Position Data

```solidity
// Get a position by ID
mapping(uint256 => HedgePosition) public positions;

// Hedger's active position ID
mapping(address => uint256) public hedgerActivePositionId;
```

***

### 🔗 YieldShift Integration

The HedgerPool integrates with YieldShift for yield distribution:

```solidity
IYieldShift public yieldShift;
```

**Distribution Flow**

```
Aave generates yield
    ↓
YieldShift calculates allocations
    ↓
HedgerPool receives its share (10-50%)
    ↓
Hedger claims via claimHedgingRewards()
```

***

### 📋 Events

```solidity
event PositionOpened(address indexed hedger, uint256 positionId, uint96 size, uint96 margin);
event PositionClosed(address indexed hedger, uint256 positionId, int128 finalPnL);
event MarginAdded(address indexed hedger, uint256 positionId, uint96 amount);
event MarginRemoved(address indexed hedger, uint256 positionId, uint96 amount);
event RewardsClaimed(address indexed hedger, uint256 amount);
event SingleHedgerSet(address indexed oldHedger, address indexed newHedger);
```

***

### ⚠️ Custom Errors

```solidity
error PositionAlreadyExists();
error NoActivePosition();
error InsufficientMargin();
error LeverageTooHigh();
error PositionNotSafeToClose();
error FilledVolumeNotZero();
error InvalidAmount();
```

***

> **Summary**: The HedgerPool is the core of Quantillon's hedging mechanics. In the MVP, a single hedger provides EUR/USD coverage, earning revenue via YieldShift and rate differentials. The real-time P&L system and security controls ensure protocol stability.
