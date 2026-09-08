# UserPool

## UserPool: Batch Deposits and QEURO Staking

### 📋 Overview

The UserPool is an **optional batch deposit/stake contract**: it lets an address mint QEURO from USDC in batches, stake QEURO under a governance-set APY with an unstaking cooldown, and keeps per-user deposit/withdrawal histories. The dApp's primary flows use `QuantillonVault` directly (`mintQEURO`, `mintAndStakeQEURO`, `redeemQEURO`) and the stQEURO ERC-4626 series for yield - see [Core Mechanisms](mechanisms.md) and [stQEURO Token](quantillon-protocols-tokens/stqeuro-token.md).

Live version: **1.0.3** · address `0x712bCc77e7aa53C79870A40d044D440Ad2901bF2` (Base mainnet).

***

### 🏗️ Contract Architecture

**Inheritance**

```solidity
contract UserPool is 
    Initializable,
    ReentrancyGuardUpgradeable,
    AccessControlUpgradeable,
    PausableUpgradeable,
    SecureUpgradeable
```

`SecureUpgradeable` routes upgrades through the 12-hour OZ TimelockController.

**External Dependencies**

| Contract | Variable | Role |
|----------|----------|------|
| `IQEUROToken` | `qeuro` | QEURO token |
| `IERC20` | `usdc` | USDC token for deposits |
| `IQuantillonVault` | `vault` | Mint/redeem of QEURO |
| `IOracle` | `oracle` | EUR/USD price (the `OracleRouter`) |
| `IYieldShift` | `yieldShift` | Yield allocation layer |
| `TimeProvider` | `TIME_PROVIDER` | Centralized time management |
| `address` | `treasury` | Recovery address |

***

### 🔐 Roles & Permissions

| Role | Responsibilities |
|------|-----------------|
| `DEFAULT_ADMIN_ROLE` | Role management, token/ETH recovery |
| `GOVERNANCE_ROLE` | Staking parameters, performance fee, YieldShift wiring |
| `EMERGENCY_ROLE` | Pause/unpause, `emergencyUnstake` |
| `UPGRADER_ROLE` | Upgrade execution (timelock-gated) |

***

### ⚙️ Configuration Parameters

#### Staking Parameters

| Parameter | Type | Description | Live Value |
|-----------|------|-------------|------------|
| `stakingAPY` | `uint256` | Staking APY in basis points | 800 = 8% |
| `depositAPY` | `uint256` | Base APY on deposits in basis points (no setter) | 400 = 4% |
| `minStakeAmount` | `uint256` | Minimum amount to stake (in QEURO) | 100e18 = 100 QEURO |
| `unstakingCooldown` | `uint256` | Cooldown period before unstake (in seconds) | 604800 = 7 days |

**Configuration Functions**

```solidity
function updateStakingParameters(
    uint256 newStakingAPY,
    uint256 newMinStakeAmount,
    uint256 newUnstakingCooldown
) external onlyRole(GOVERNANCE_ROLE);
// emits PoolParameterUpdated(parameter, oldValue, newValue) per changed field
```

#### Performance Fee

| Parameter | Type | Description | Live Value |
|-----------|------|-------------|------------|
| `performanceFee` | `uint256` | Fee on distributed yield (basis points) | 0 (currently; governance-settable) |

> **Note**: Mint and redemption fees are managed by QuantillonVault, not UserPool.

```solidity
function setPerformanceFee(uint256 _performanceFee) 
    external onlyRole(GOVERNANCE_ROLE);
```

***

### 📊 Data Structures

#### UserInfo

```solidity
struct UserInfo {
    uint128 qeuroBalance;        // QEURO balance (18 decimals)
    uint128 stakedAmount;        // Staked amount (18 decimals)
    uint128 pendingRewards;      // Pending rewards
    uint128 unstakeAmount;       // Amount being unstaked
    uint96 depositHistory;       // USDC deposit history (6 decimals)
    uint64 lastStakeTime;        // Last stake timestamp
    uint64 unstakeRequestTime;   // Unstake request timestamp
}
```

> **Optimization**: Fields are packed to reduce gas costs.

#### UserDepositInfo

```solidity
struct UserDepositInfo {
    uint128 usdcAmount;          // USDC deposited (6 decimals)
    uint128 qeuroReceived;       // QEURO received (18 decimals)
    uint64 timestamp;            // Deposit timestamp
    uint32 oracleRatio;          // Oracle ratio (scaled 1e6)
    uint32 blockNumber;          // Block number
}
```

#### UserWithdrawalInfo

```solidity
struct UserWithdrawalInfo {
    uint128 qeuroAmount;         // QEURO withdrawn (18 decimals)
    uint128 usdcReceived;        // USDC received (6 decimals)
    uint64 timestamp;            // Withdrawal timestamp
    uint32 oracleRatio;          // Oracle ratio (scaled 1e6)
    uint32 blockNumber;          // Block number
}
```

***

### 💰 User Operations

All user entry points take **batches** (arrays of equal length, at most `MAX_BATCH_SIZE = 100` items; `ArrayLengthMismatch`, `BatchSizeTooLarge`, `EmptyArray` otherwise). Each item carries its own slippage guard.

#### Deposit (USDC → QEURO)

```solidity
function deposit(uint256[] calldata usdcAmounts, uint256[] calldata minQeuroOuts)
    external whenNotPaused nonReentrant
    returns (uint256[] memory qeuroMintedAmounts);
```

`minQeuroOuts[i]` is the minimum QEURO accepted for `usdcAmounts[i]`; the call reverts with `ExcessiveSlippage` if the oracle-priced output is lower.

**Flow**

```
1. User approves USDC for UserPool
2. Calls deposit([amounts], [minQeuroOuts])
3. USDC transferred to the Vault, which mints QEURO at the oracle price
4. QEURO forwarded to the user; UserDeposit / UserDepositTracked emitted
5. Deposit history updated
```

#### Withdrawal (QEURO → USDC)

```solidity
function withdraw(uint256[] calldata qeuroAmounts, uint256[] calldata minUsdcOuts)
    external whenNotPaused nonReentrant
    returns (uint256[] memory usdcReceivedAmounts);
```

**Flow**

```
1. User calls withdraw([amounts], [minUsdcOuts])
2. Vault burns QEURO and returns USDC (minUsdcOuts[i] enforced per item)
3. USDC credited to the user's pending balance and settled in the same transaction
4. UserWithdrawal / UserWithdrawalTracked emitted; history updated
```

#### Pending Withdrawals

The USDC of a withdrawal is first credited to `pendingUsdcWithdrawals[user]` and then settled immediately. If the immediate transfer cannot be settled in the same transaction, the amount stays claimable and `WithdrawalPending(user, amount)` is emitted; the user collects it later:

```solidity
mapping(address => uint256) public pendingUsdcWithdrawals;

function claimPendingWithdrawal() external whenNotPaused nonReentrant;
// transfers the full pending balance; emits PendingWithdrawalClaimed(user, amount)
```

***

### 📈 Staking System

#### 🚦 Staking APY Mechanism

The staking APY determines the annual return for users who stake their QEURO.

**Reward Calculation**

```
Rewards = (stakedAmount × stakingAPY × timeElapsed) / (10000 × 365 days)
```

| Variable | Description |
|----------|-------------|
| `stakedAmount` | QEURO amount staked |
| `stakingAPY` | Annual rate in BPS (live: 800 = 8%) |
| `timeElapsed` | Time since last calculation (capped at `MAX_REWARD_PERIOD` = 365 days) |

**Reward Settlement**

```solidity
function _updatePendingRewards(address user, uint256 currentTime) internal;
```

Pending rewards are settled automatically on every stake and unstake interaction and are visible through `getUserInfo(user).pendingRewards`. There is no separate claim function.

#### Stake

```solidity
function stake(uint256[] calldata qeuroAmounts) external whenNotPaused nonReentrant;
```

**Validations**

1. Each `qeuroAmounts[i] >= minStakeAmount`
2. User has sufficient QEURO balance
3. Contract not paused

**Effects**

1. Updates pending rewards
2. Transfers QEURO from user to contract
3. Increases `stakedAmount` and `totalStakes`
4. Records `lastStakeTime`; emits `QEUROStaked`

#### ⏱️ Cooldown Mechanism (Unstaking)

The cooldown system prevents rapid stake/unstake cycles and protects against manipulation.

**Unstaking Flow**

```
┌─────────────────────────────────────────────────────────┐
│                    UNSTAKING FLOW                        │
├─────────────────────────────────────────────────────────┤
│  Step 1: requestUnstake(qeuroAmount)                    │
│  ├── Settle pending rewards                             │
│  ├── Set unstakeAmount                                  │
│  └── Set unstakeRequestTime = now                       │
├─────────────────────────────────────────────────────────┤
│  Step 2: Wait for cooldown period                       │
│  └── unstakingCooldown (live: 7 days)                   │
├─────────────────────────────────────────────────────────┤
│  Step 3: unstake()                                      │
│  ├── Verify: now >= unstakeRequestTime + cooldown       │
│  ├── Transfer QEURO back to user                        │
│  ├── Clear unstakeAmount and unstakeRequestTime         │
│  └── Emit QEUROUnstaked                                 │
└─────────────────────────────────────────────────────────┘
```

**Functions**

```solidity
// Step 1: Request unstake
function requestUnstake(uint256 qeuroAmount) external nonReentrant;

// Step 2: Finalize unstake (after cooldown)
function unstake() external whenNotPaused nonReentrant;
```

**Possible Reverts**

| Condition | Revert |
|-------|-------|
| Requested amount > staked amount | `InsufficientBalance` |
| `unstake()` with no request in progress | `InvalidAmount` |
| Cooldown not yet elapsed | `InvalidCondition` |

**Cooldown Configuration**

```solidity
// Configurable value via governance
uint256 public unstakingCooldown;  // In seconds

// Live: 7 days = 604800 seconds
// Can be set to 0 to disable cooldown
```

***

### 💸 Performance Fee

The `performanceFee` (currently 0) is deducted from yield distributed to users.

**Tracking Variables**

```solidity
uint256 public accumulatedYieldPerShare;  // Cumulative yield per share
uint256 public lastYieldDistribution;      // Last distribution timestamp
uint256 public totalYieldDistributed;      // Total yield distributed
```

***

### 📊 View Functions

#### User Information

```solidity
function getUserInfo(address user) external view returns (
    uint256 qeuroBalance,
    uint256 stakedAmount,
    uint256 pendingRewards,
    uint256 depositHistory,
    uint256 lastStakeTime,
    uint256 unstakeAmount,
    uint256 unstakeRequestTime
);
```

#### Pool Metrics

```solidity
function getPoolTotals() external view returns (
    uint256 totalDeposits,
    uint256 totalWithdrawals,
    uint256 totalStakes,
    uint256 totalUsers
);

function getPoolMetrics() external view returns (
    uint256 totalUsers,
    uint256 averageDeposit,
    uint256 stakingRatio,
    uint256 poolTVL
);

function getPoolConfiguration() external view returns (
    uint256 stakingAPY,
    uint256 depositAPY,
    uint256 minStakeAmount,
    uint256 unstakingCooldown,
    uint256 performanceFee
);
```

#### Histories

```solidity
function getUserDepositHistory(address user) external view 
    returns (UserDepositInfo[] memory);

function getUserWithdrawals(address user) external view 
    returns (UserWithdrawalInfo[] memory);
```

#### Analytics

```solidity
// Not a view: refreshes the oracle price before computing the USDC equivalent
function getPoolAnalytics() external returns (
    uint256 currentQeuroSupply,
    uint256 usdcEquivalentAtCurrentRate,
    uint256 totalUsers,
    uint256 totalStakes
);

function calculateProjectedRewards(uint256 qeuroAmount, uint256 duration) 
    external view returns (uint256);
```

***

### 🛡️ Security & Emergency Controls

#### Emergency Unstake

```solidity
function emergencyUnstake(address user, address recipient) external onlyRole(EMERGENCY_ROLE);
```

Forces the unstake of **one user's** staked QEURO to a recipient, bypassing the cooldown. It is a per-user operation, not a global one.

#### Pause

```solidity
function pause() external onlyRole(EMERGENCY_ROLE);
function unpause() external onlyRole(EMERGENCY_ROLE);
```

#### Pool Status

```solidity
function isPoolActive() external view returns (bool);
```

#### Recovery

```solidity
function recoverToken(address token, uint256 amount) 
    external onlyRole(DEFAULT_ADMIN_ROLE);

function recoverETH() external onlyRole(DEFAULT_ADMIN_ROLE);
```

***

### 📏 Constants

```solidity
uint256 public constant MAX_BATCH_SIZE = 100;        // Max items per deposit / withdraw / stake batch
uint256 public constant MAX_REWARD_BATCH_SIZE = 50;  // Reserved; no public batch-claim function uses it
uint256 public constant MAX_REWARD_PERIOD = 365 days;
```

***

### 📋 Events

```solidity
event UserDeposit(address indexed user, uint256 usdcAmount, uint256 qeuroMinted, uint256 timestamp);
event UserWithdrawal(address indexed user, uint256 qeuroBurned, uint256 usdcReceived, uint256 timestamp);
event UserDepositTracked(address indexed user, uint256 usdcAmount, uint256 qeuroReceived, uint256 oracleRatio, uint256 timestamp, uint256 blockNumber);
event UserWithdrawalTracked(address indexed user, uint256 qeuroAmount, uint256 usdcReceived, uint256 oracleRatio, uint256 timestamp, uint256 blockNumber);
event WithdrawalPending(address indexed user, uint256 amount);
event PendingWithdrawalClaimed(address indexed user, uint256 amount);
event QEUROStaked(address indexed user, uint256 qeuroAmount, uint256 timestamp);
event QEUROUnstaked(address indexed user, uint256 qeuroAmount, uint256 timestamp);
event PoolParameterUpdated(string indexed parameter, uint256 oldValue, uint256 newValue);
event ETHRecovered(address indexed to, uint256 indexed amount);
```

***

> **Summary**: The UserPool is an optional batch interface over the vault - batched USDC→QEURO deposits and withdrawals with per-item slippage guards, QEURO staking with a governance-set APY and a 7-day unstaking cooldown, and per-user histories. The dApp's main flows go through `QuantillonVault` and the stQEURO series directly.
