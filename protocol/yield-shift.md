# YieldShift

## YieldShift: Dynamic Yield Distribution System

### 📋 Overview

YieldShift is the dynamic yield distribution engine in the Quantillon protocol. It automatically balances revenue distribution between users (stQEURO holders) and hedgers, based on market conditions and pool ratios.

***

### 🏗️ Contract Architecture

**Inheritance**

```solidity
contract YieldShift is 
    Initializable,
    ReentrancyGuardUpgradeable,
    AccessControlUpgradeable,
    PausableUpgradeable,
    SecureUpgradeable
```

**External Dependencies**

| Contract | Variable | Role |
|----------|----------|------|
| `IERC20` | `usdc` | Yield token |
| `IUserPool` | `userPool` | User pool metrics |
| `IHedgerPool` | `hedgerPool` | Hedger pool metrics |
| `IAaveVault` | `aaveVault` | Aave yield source |
| `IstQEURO` | `stQEURO` | User yield distribution |
| `TimeProvider` | `TIME_PROVIDER` | Centralized time management |

***

### 🔐 Roles & Permissions

| Role | Responsibilities |
|------|-----------------|
| `GOVERNANCE_ROLE` | Yield shift parameters |
| `YIELD_MANAGER_ROLE` | Yield distribution, operations |
| `EMERGENCY_ROLE` | Pause, emergencies |

***

### ⚙️ Yield Shift Parameters

#### Main Parameters

| Parameter | Type | Description | Default Value |
|-----------|------|-------------|---------------|
| `baseYieldShift` | `uint256` | Base allocation to users (BPS) | 5000 (50%) |
| `maxYieldShift` | `uint256` | Max allocation to users (BPS) | 9000 (90%) |
| `adjustmentSpeed` | `uint256` | Adjustment speed (BPS) | 100 (1%) |
| `targetPoolRatio` | `uint256` | Optimal user/hedger ratio (BPS) | 10000 (100%) |

**Interpretation**

```
baseYieldShift = 5000 → 50% of yield goes to users by default
maxYieldShift = 9000 → Maximum 90% can go to users
adjustmentSpeed = 100 → Shift changes by 1% max per adjustment
targetPoolRatio = 10000 → Goal is user pool = hedger pool
```

#### Time Constants

```solidity
uint256 public constant MIN_HOLDING_PERIOD = 7 days;   // Min period for yield
uint256 public constant TWAP_PERIOD = 24 hours;        // TWAP window
uint256 public constant MAX_TIME_ELAPSED = 365 days;   // Max calculation period
```

***

### 📊 Distribution Mechanism

#### Current Yield Shift Calculation

```
                         Pool Ratio
                             │
                             ▼
┌─────────────────────────────────────────────────────────┐
│  IF userPoolSize > hedgerPoolSize × targetRatio        │
│  THEN shift toward hedgers (decrease currentYieldShift)│
│                                                         │
│  IF hedgerPoolSize > userPoolSize / targetRatio        │
│  THEN shift toward users (increase currentYieldShift)  │
│                                                         │
│  Constraint: (10000 - maxYieldShift) ≤ shift ≤ max     │
└─────────────────────────────────────────────────────────┘
```

**Distribution Formula**

```solidity
userAllocation = totalYield × currentYieldShift / 10000
hedgerAllocation = totalYield - userAllocation
```

| Pool State | currentYieldShift | Users Receive | Hedgers Receive |
|------------|-------------------|---------------|-----------------|
| Balanced | 50% (5000) | 50% | 50% |
| User Surplus | 30% (3000) | 30% | 70% |
| Hedger Surplus | 80% (8000) | 80% | 20% |

***

### 🕐 TWAP System (Time-Weighted Average Price)

TWAP smooths pool metrics over 24 hours to prevent manipulation.

#### Pool Snapshot

```solidity
struct PoolSnapshot {
    uint256 timestamp;         // Capture moment
    uint256 userPoolSize;      // User pool size
    uint256 hedgerPoolSize;    // Hedger pool size
    uint256 poolRatio;         // Calculated ratio
}
```

#### Snapshot History

```solidity
PoolSnapshot[] public poolHistory;
uint256 public constant MAX_HISTORY_LENGTH = 168; // 7 days @ 1h
```

**TWAP Calculation**

```
                    24h Window
          ┌────────────────────────────┐
Timeline: ─○──○──○──○──○──○──○──○──○──○─▶
          s1 s2 s3 s4 s5 s6 s7 s8 s9 s10
          
TWAP = Σ(snapshot_i.poolRatio × duration_i) / total_duration
```

***

### 🛡️ Flash Deposit Protection

#### Holding Period (7 days)

The protocol excludes recent deposits from yield calculations to prevent attacks.

```solidity
// Deposits < 7 days don't count for yield
if (depositTime + MIN_HOLDING_PERIOD > currentTime) {
    // Deposit not eligible
}
```

#### Eligible Pool Metrics

```solidity
function _getEligiblePoolMetrics() internal view returns (
    uint256 eligibleUserPoolSize,
    uint256 eligibleHedgerPoolSize
);
```

This function returns only "eligible" pool sizes - deposits that have passed the holding period.

**Why This Matters**

```
Attack scenario WITHOUT protection:
1. Attacker deposits 10M USDC just before distribution
2. Harvests yield proportional to their deposit
3. Withdraws immediately
4. Profit at expense of long-term depositors

With holding period:
1. Attacker deposits 10M USDC
2. For 7 days: their deposit doesn't count
3. Yield distributed to eligible depositors only
4. Attack not profitable
```

***

### 📡 Authorized Yield Sources

#### Authorization System

```solidity
mapping(address => bool) public authorizedYieldSources;
mapping(address => bytes32) public sourceToYieldType;
```

**Authorized Sources**

| Source | Type | Description |
|--------|------|-------------|
| `AaveVault` | AAVE_YIELD | Yield from USDC lending on Aave |
| `QuantillonVault` | PROTOCOL_FEES | Mint/redeem fees |
| `HedgerPool` | HEDGING_FEES | Hedger operation fees |

#### Source Management

```solidity
// Authorize a new source
function authorizeYieldSource(address source, bytes32 yieldType) 
    external onlyRole(GOVERNANCE_ROLE);

// Remove authorization
function deauthorizeYieldSource(address source) 
    external onlyRole(GOVERNANCE_ROLE);
```

#### Source Tracking

```solidity
mapping(bytes32 => uint256) public yieldSources;
bytes32[] public yieldSourceNames;
```

Allows tracking yield origin:

```solidity
yieldSources["AAVE_YIELD"] = 1_000_000e6;     // 1M USDC from Aave
yieldSources["PROTOCOL_FEES"] = 50_000e6;     // 50K USDC from fees
```

***

### 💰 State Variables

#### Yield Pools

```solidity
uint256 public currentYieldShift;      // Current shift (BPS)
uint256 public lastUpdateTime;         // Last update
uint256 public totalYieldGenerated;    // Total yield generated
uint256 public totalYieldDistributed;  // Total distributed
uint256 public userYieldPool;          // Pending user pool
uint256 public hedgerYieldPool;        // Pending hedger pool
address public treasury;               // Treasury address
```

#### Pending Yield Per User

```solidity
mapping(address => uint256) public userPendingYield;
```

***

### 🔄 Distribution Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    YIELD DISTRIBUTION FLOW                   │
├─────────────────────────────────────────────────────────────┤
│  1. AaveVault.harvestYield()                                │
│     └── Sends yield to YieldShift                           │
│                                                              │
│  2. YieldShift.receiveYield(amount, source)                 │
│     ├── Verify authorized source                            │
│     ├── Calculate currentYieldShift                         │
│     ├── userYieldPool += amount × shift                     │
│     └── hedgerYieldPool += amount × (1 - shift)             │
│                                                              │
│  3. YieldShift.distributeToUsers()                          │
│     └── stQEURO.distributeYield(userYieldPool)              │
│                                                              │
│  4. YieldShift.distributeToHedgers()                        │
│     └── HedgerPool receives hedgerYieldPool                 │
└─────────────────────────────────────────────────────────────┘
```

***

### 📊 View Functions

#### Yield Shift Status

```solidity
function getCurrentYieldShift() external view returns (uint256);

function getYieldShiftParameters() external view returns (
    uint256 baseYieldShift,
    uint256 maxYieldShift,
    uint256 adjustmentSpeed,
    uint256 targetPoolRatio,
    uint256 currentYieldShift
);
```

#### Pool Metrics

```solidity
function getPoolMetrics() external view returns (
    uint256 userPoolSize,
    uint256 hedgerPoolSize,
    uint256 poolRatio,
    uint256 eligibleUserPoolSize,
    uint256 eligibleHedgerPoolSize
);
```

#### Yield Statistics

```solidity
function getYieldStatistics() external view returns (
    uint256 totalYieldGenerated,
    uint256 totalYieldDistributed,
    uint256 userYieldPool,
    uint256 hedgerYieldPool
);
```

#### Historical Data

```solidity
function getHistoricalYieldShift(uint256 periods) external view 
    returns (PoolSnapshot[] memory);

function getPoolHistory() external view 
    returns (PoolSnapshot[] memory);
```

***

### ⚙️ Configuration

#### Update Parameters

```solidity
function updateYieldShiftParameters(
    uint256 _baseYieldShift,
    uint256 _maxYieldShift,
    uint256 _adjustmentSpeed,
    uint256 _targetPoolRatio
) external onlyRole(GOVERNANCE_ROLE);
```

**Validations**

- `baseYieldShift <= 10000` (max 100%)
- `maxYieldShift >= baseYieldShift`
- `maxYieldShift <= 10000`
- `targetPoolRatio > 0`

***

### 🛡️ Security & Emergency Controls

#### Pause

```solidity
function pause() external onlyRole(EMERGENCY_ROLE);
function unpause() external onlyRole(EMERGENCY_ROLE);
```

#### Recovery

```solidity
function recoverToken(address token, uint256 amount) 
    external onlyRole(DEFAULT_ADMIN_ROLE);

function recoverETH() external onlyRole(DEFAULT_ADMIN_ROLE);
```

***

### 📋 Events

```solidity
event YieldReceived(address indexed source, bytes32 indexed yieldType, uint256 amount);
event YieldDistributed(uint256 userAmount, uint256 hedgerAmount);
event YieldShiftUpdated(uint256 oldShift, uint256 newShift);
event ParametersUpdated(uint256 baseShift, uint256 maxShift, uint256 speed, uint256 targetRatio);
event YieldSourceAuthorized(address indexed source, bytes32 indexed yieldType);
event YieldSourceDeauthorized(address indexed source);
event PoolSnapshotTaken(uint256 timestamp, uint256 userPoolSize, uint256 hedgerPoolSize);
```

***

### 📐 Calculation Examples

#### Example 1: Standard Distribution

```
Initial state:
- totalYield = 10,000 USDC
- currentYieldShift = 5000 (50%)

Result:
- userAllocation = 10,000 × 50% = 5,000 USDC → stQEURO
- hedgerAllocation = 10,000 × 50% = 5,000 USDC → HedgerPool
```

#### Example 2: User Imbalance

```
State:
- userPoolSize = 8M USDC
- hedgerPoolSize = 2M USDC
- poolRatio = 4:1 (too many users)
- currentYieldShift adjusted to 3000 (30%)

Result on 10,000 USDC yield:
- userAllocation = 3,000 USDC
- hedgerAllocation = 7,000 USDC
→ Incentivizes more hedgers to participate
```

#### Example 3: Hedger Imbalance

```
State:
- userPoolSize = 2M USDC
- hedgerPoolSize = 8M USDC
- poolRatio = 1:4 (too many hedgers)
- currentYieldShift adjusted to 8000 (80%)

Result on 10,000 USDC yield:
- userAllocation = 8,000 USDC
- hedgerAllocation = 2,000 USDC
→ Incentivizes more users to deposit
```

***

> **Summary**: YieldShift is the nervous system of yield distribution in Quantillon. It dynamically balances incentives between users and hedgers via a TWAP algorithm protected against manipulation, while ensuring that only deposits that have passed the 7-day holding period participate in distribution.
