# UserPool

## UserPool: QEURO Deposits and Staking Management

### 📋 Overview

The UserPool is the central contract that manages USDC user deposits, QEURO staking operations, and reward distribution. It's the main entry point for Quantillon protocol users.

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

**External Dependencies**

| Contract | Variable | Role |
|----------|----------|------|
| `IQEUROToken` | `qeuro` | QEURO token for mint/burn |
| `IERC20` | `usdc` | USDC token for deposits |
| `IQuantillonVault` | `vault` | Main vault for QEURO operations |
| `IOracle` | `oracle` | Real-time EUR/USD price |
| `IYieldShift` | `yieldShift` | Yield distribution |
| `TimeProvider` | `TIME_PROVIDER` | Centralized time management |
| `address` | `treasury` | ETH recovery address |

***

### 🔐 Roles & Permissions

| Role | Responsibilities |
|------|-----------------|
| `DEFAULT_ADMIN_ROLE` | General administration |
| `GOVERNANCE_ROLE` | Parameter modification (APY, fees, cooldown) |
| `EMERGENCY_ROLE` | Pause, emergency unstake, recovery |

***

### ⚙️ Configuration Parameters

#### Staking Parameters

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `stakingAPY` | `uint256` | Staking APY in basis points | 500 = 5% |
| `depositAPY` | `uint256` | Base APY on deposits in basis points | 200 = 2% |
| `minStakeAmount` | `uint256` | Minimum amount to stake (in QEURO) | 100e18 = 100 QEURO |
| `unstakingCooldown` | `uint256` | Cooldown period before unstake (in seconds) | 604800 = 7 days |

**Configuration Functions**

```solidity
function updateStakingParameters(
    uint256 _stakingAPY,
    uint256 _depositAPY,
    uint256 _minStakeAmount,
    uint256 _unstakingCooldown
) external onlyRole(GOVERNANCE_ROLE);
```

#### Performance Fee

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `performanceFee` | `uint256` | Fee on distributed yield (basis points) | 1000 = 10% |

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

#### Deposit (USDC → QEURO)

```solidity
function deposit(uint256 usdcAmount) 
    external whenNotPaused nonReentrant;
```

**Flow**

```
1. User approves USDC for UserPool
2. Calls deposit(amount)
3. USDC transferred to Vault
4. Vault mints QEURO to user via oracle price
5. History updated
```

#### Withdrawal (QEURO → USDC)

```solidity
function withdraw(uint256 qeuroAmount) 
    external whenNotPaused nonReentrant;
```

**Flow**

```
1. User calls withdraw(amount)
2. Vault burns QEURO
3. Vault returns USDC to user
4. History updated
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
| `stakingAPY` | Annual rate in BPS (500 = 5%) |
| `timeElapsed` | Time since last calculation |

**Reward Update**

```solidity
function _updatePendingRewards(address user) internal;
```

This function is called automatically on each interaction (stake, unstake, claim).

#### Stake

```solidity
function stake(uint256 amount) external whenNotPaused nonReentrant;
```

**Validations**

1. `amount >= minStakeAmount` (minimum amount)
2. User has sufficient QEURO balance
3. Contract not paused

**Effects**

1. Updates pending rewards
2. Transfers QEURO from user to contract
3. Increases `stakedAmount` and `totalStakes`
4. Records `lastStakeTime`

#### ⏱️ Cooldown Mechanism (Unstaking)

The cooldown system prevents rapid stake/unstake cycles and protects against manipulation.

**Unstaking Flow**

```
┌─────────────────────────────────────────────────────────┐
│                    UNSTAKING FLOW                        │
├─────────────────────────────────────────────────────────┤
│  Step 1: requestUnstake(amount)                         │
│  ├── Set unstakeAmount                                  │
│  ├── Set unstakeRequestTime = now                       │
│  └── Emit UnstakeRequested event                        │
├─────────────────────────────────────────────────────────┤
│  Step 2: Wait for cooldown period                       │
│  └── unstakingCooldown (e.g., 7 days)                   │
├─────────────────────────────────────────────────────────┤
│  Step 3: unstake()                                      │
│  ├── Verify: now >= unstakeRequestTime + cooldown       │
│  ├── Transfer QEURO back to user                        │
│  ├── Clear unstakeAmount and unstakeRequestTime         │
│  └── Emit Unstaked event                                │
└─────────────────────────────────────────────────────────┘
```

**Functions**

```solidity
// Step 1: Request unstake
function requestUnstake(uint256 amount) external whenNotPaused nonReentrant;

// Step 2: Finalize unstake (after cooldown)
function unstake() external whenNotPaused nonReentrant;
```

**Possible Errors**

| Error | Cause |
|-------|-------|
| `CooldownNotComplete` | Cooldown not yet finished |
| `NoUnstakeRequest` | No unstake request in progress |
| `InsufficientStakedAmount` | Requested amount > staked |

**Cooldown Configuration**

```solidity
// Configurable value via governance
uint256 public unstakingCooldown;  // In seconds

// Example: 7 days = 604800 seconds
// Can be set to 0 to disable cooldown
```

#### Reward Claims

```solidity
function claimStakingRewards() external whenNotPaused nonReentrant;
```

Allows the user to claim their `pendingRewards` in QEURO.

***

### 📦 Batch Operations

To optimize gas on multiple operations:

#### Batch Reward Claim

```solidity
function batchRewardClaim(address[] calldata users) 
    external onlyRole(GOVERNANCE_ROLE) whenNotPaused nonReentrant;
```

**Characteristics**

| Parameter | Value | Description |
|-----------|-------|-------------|
| `MAX_REWARD_BATCH_SIZE` | 50 | Maximum users per batch |

**Use Cases**

- Automated reward distribution
- Governance maintenance operations
- Gas optimization for multiple claims

***

### 💸 Performance Fee

The `performanceFee` is deducted from yield distributed to users.

**Distribution Flow**

```
Gross Yield (from YieldShift)
    │
    ├── Performance Fee (10%) → Treasury
    │
    └── Net Yield → Users (via accumulatedYieldPerShare)
```

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
    uint128 qeuroBalance,
    uint128 stakedAmount,
    uint128 pendingRewards,
    uint128 unstakeAmount,
    uint96 depositHistory,
    uint64 lastStakeTime,
    uint64 unstakeRequestTime
);
```

#### Pool Metrics

```solidity
function getPoolTotals() external view returns (
    uint256 totalStakes,
    uint256 totalUsers
);

function getPoolMetrics() external view returns (
    uint256 stakingAPY,
    uint256 depositAPY,
    uint256 minStakeAmount,
    uint256 unstakingCooldown,
    uint256 performanceFee,
    uint256 totalStakes,
    uint256 totalYieldDistributed
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
function getPoolAnalytics() external view returns (...);
function getPoolConfiguration() external view returns (...);
function calculateProjectedRewards(address user, uint256 duration) 
    external view returns (uint256);
```

***

### 🛡️ Security & Emergency Controls

#### Emergency Unstake

```solidity
function emergencyUnstake() external onlyRole(EMERGENCY_ROLE);
```

Forces unstake for all users in case of emergency.

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
uint256 public constant MAX_BATCH_SIZE = 100;        // Max batch deposits
uint256 public constant MAX_REWARD_BATCH_SIZE = 50;  // Max batch claims
uint256 public constant BLOCKS_PER_DAY = 7200;       // ~12 sec blocks
uint256 public constant MAX_REWARD_PERIOD = 365 days;
```

***

### 📋 Events

```solidity
event Deposited(address indexed user, uint256 usdcAmount, uint256 qeuroReceived);
event Withdrawn(address indexed user, uint256 qeuroAmount, uint256 usdcReceived);
event Staked(address indexed user, uint256 amount);
event UnstakeRequested(address indexed user, uint256 amount, uint256 timestamp);
event Unstaked(address indexed user, uint256 amount);
event RewardsClaimed(address indexed user, uint256 amount);
event YieldDistributed(uint256 amount, uint256 newAccumulatedYieldPerShare);
event ParametersUpdated(uint256 stakingAPY, uint256 depositAPY, uint256 minStakeAmount, uint256 cooldown);
event PerformanceFeeUpdated(uint256 oldFee, uint256 newFee);
```

***

> **Summary**: The UserPool is the main user interaction contract, managing USDC deposits, QEURO staking with configurable APY, and reward distribution. The cooldown mechanism protects against manipulation, while batch operations optimize gas costs.
