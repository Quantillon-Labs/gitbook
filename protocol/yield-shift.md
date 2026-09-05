# YieldShift

## YieldShift: Dynamic Yield Distribution System

### 📋 Overview

YieldShift is the dynamic yield-allocation engine of the Quantillon protocol. It balances the split of yield between the user side (stQEURO holders) and the hedger side according to pool conditions, and keeps the **hedger-side yield ledger** from which the hedger claims.

> **How yield flows in the live deployment**: the yield earned by the external staking vault (Morpho) is harvested by `QuantillonVault.harvestAndDistributeVaultYield`, which credits the stakers' share **directly** into the stQEURO series (exchange rate) and the treasury's share to the treasury. YieldShift receives yield only from governance-authorised sources through `addYield` and tracks the hedger's claimable share. No yield source is authorised on the live deployment today, and the hedger funding carve-out is currently 0 bps — see [stQEURO Token](quantillon-protocols-tokens/stqeuro-token.md) for the harvest split.

Live version: **1.0.5** · address `0xdcd66568F8623bDa3387287c31F14b43e49665b1` (Base mainnet).

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
| `IstQEUROFactory` | `stQEUROFactory` | Resolves the stQEURO series per `vaultId` |
| `address` | `treasury` | Recovery address |
| `TimeProvider` | `TIME_PROVIDER` | Centralized time management |

***

### 🔐 Roles & Permissions

| Role | Responsibilities |
|------|-----------------|
| `DEFAULT_ADMIN_ROLE` | Role management, recovery |
| `GOVERNANCE_ROLE` | Yield model, dependencies, yield-source authorisation, source→vault bindings, forced distribution update |
| `YIELD_MANAGER_ROLE` | `updateYieldAllocation` (allocation hook; held by the governance Safe) |
| `EMERGENCY_ROLE` | Pause/resume distribution, emergency distribution |
| `UPGRADER_ROLE` | Upgrade execution (timelock-gated) |

***

### ⚙️ Yield Shift Parameters

#### Main Parameters

| Parameter | Type | Description | Live Value |
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
uint256 public constant MAX_HISTORY_LENGTH = 1000;     // Max stored snapshots
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

```solidity
// Recompute the shift when the update interval has elapsed (anyone)
function checkAndUpdateYieldDistribution() external;

// Force a recompute (GOVERNANCE_ROLE)
function forceUpdateYieldDistribution() external;

// Preview
function calculateOptimalYieldShift() external view
    returns (uint256 optimalShift, uint256 currentDeviation);
```

***

### 🕐 TWAP System (Time-Weighted Average)

TWAP smooths pool metrics over 24 hours to prevent manipulation.

#### Pool Snapshots

```solidity
struct PoolSnapshot {
    uint128 userPoolSize;      // User pool size
    uint128 hedgerPoolSize;    // Hedger pool size
    uint64 timestamp;          // Capture moment
}

PoolSnapshot[] public userPoolHistory;
PoolSnapshot[] public hedgerPoolHistory;
uint256 public constant MAX_HISTORY_LENGTH = 1000;

struct YieldShiftSnapshot { uint128 yieldShift; uint64 timestamp; }
YieldShiftSnapshot[] public yieldShiftHistory;
function poolHistoryCount() external view returns (uint256);
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
mapping(address => uint256) public lastDepositTime;
function updateLastDepositTime(address user) external;  // called by the pools on deposit
```

#### Eligible Pool Metrics

```solidity
function _getEligiblePoolMetrics() internal view returns (
    uint256 eligibleUserPoolSize,
    uint256 eligibleHedgerPoolSize
);
```

This function returns only "eligible" pool sizes — deposits that have passed the holding period.

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

function isYieldSourceAuthorized(address source, bytes32 yieldType) external view returns (bool);

// Authorise or revoke a source for one yield type (GOVERNANCE_ROLE)
function setYieldSourceAuthorization(address source, bytes32 yieldType, bool authorized) external;
```

**Example configuration** (no source is authorised on the live deployment today):

| Source | Type | Description |
|--------|------|-------------|
| `QuantillonVault` | VAULT_YIELD | Hedger share of harvested external-vault yield |
| `QuantillonVault` | PROTOCOL_FEES | Mint/redeem fees (currently 0) |
| `HedgerPool` | HEDGING_FEES | Hedger operation fees (currently 0) |

#### Source → Vault Binding

Since the multi-vault upgrade (v1.0.5) every yield source is bound to a `vaultId`, so yield can only be attributed to the stQEURO series it came from:

```solidity
mapping(address => uint256) public sourceToVaultId;
bool public enforceSourceVaultBinding;   // live: true

function setSourceVaultBinding(address source, uint256 vaultId) external;      // GOVERNANCE_ROLE, vaultId != 0
function clearSourceVaultBinding(address source) external;                     // GOVERNANCE_ROLE
function setSourceVaultBindingEnforcement(bool enabled) external;              // GOVERNANCE_ROLE
```

#### Receiving Yield

```solidity
function addYield(uint256 vaultId, uint256 yieldAmount, bytes32 source) external;
```

Reverts unless the caller is an authorised source of that `source` type, `vaultId != 0`, and — while enforcement is on — `vaultId` equals the caller's bound vault. The USDC is pulled from the caller (exact amount checked) and split into `userYieldPool` / `hedgerYieldPool` according to `currentYieldShift`; emits `YieldAdded`.

#### Source Tracking

```solidity
mapping(bytes32 => uint256) public yieldSources;
bytes32[] public yieldSourceNames;
function getYieldSources() external view
    returns (uint256 aaveYield, uint256 protocolFees, uint256 interestDifferential, uint256 otherSources);
```

***

### 💰 State Variables

```solidity
uint256 public currentYieldShift;      // Current shift (BPS)
uint256 public lastUpdateTime;         // Last update
uint256 public totalYieldGenerated;    // Total yield received
uint256 public totalYieldDistributed;  // Total distributed
uint256 public userYieldPool;          // Pending user pool
uint256 public hedgerYieldPool;        // Pending hedger pool
address public treasury;               // Treasury address

mapping(address => uint256) public userPendingYield;
mapping(address => uint256) public hedgerPendingYield;
mapping(address => uint256) public userLastClaim;
mapping(address => uint256) public hedgerLastClaim;
```

***

### 🔄 Distribution Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    YIELD DISTRIBUTION FLOW                   │
├─────────────────────────────────────────────────────────────┤
│  User side (live path, outside YieldShift)                   │
│  QuantillonVault.harvestAndDistributeVaultYield(vaultId)     │
│     ├── hedger funding carve-out first (currently 0 bps)     │
│     ├── stakers' share credited to the stQEURO series        │
│     │   (exchange rate rises — no claim needed)              │
│     └── treasury share to the treasury                        │
│                                                              │
│  Hedger side (YieldShift ledger)                             │
│  1. authorised source calls addYield(vaultId, amount, type)  │
│     ├── userYieldPool   += amount × currentYieldShift        │
│     └── hedgerYieldPool += amount × (1 - currentYieldShift)  │
│  2. checkAndUpdateYieldDistribution() refreshes the shift    │
│  3. HedgerPool.claimHedgingRewards() → claimHedgerYield()    │
│     └── hedger's pending yield paid in USDC                  │
└─────────────────────────────────────────────────────────────┘
```

```solidity
function claimHedgerYield(address hedger) external returns (uint256 yieldAmount);
// called through HedgerPool.claimHedgingRewards(); emits HedgerYieldClaimed
```

***

### 📊 View Functions

#### Yield Shift Status

```solidity
function currentYieldShift() external view returns (uint256);
function baseYieldShift() external view returns (uint256);
function maxYieldShift() external view returns (uint256);
function adjustmentSpeed() external view returns (uint256);
function targetPoolRatio() external view returns (uint256);
```

#### Pool Metrics

```solidity
function getPoolMetrics() external view returns (
    uint256 userPoolSize,
    uint256 hedgerPoolSize,
    uint256 poolRatio,
    uint256 targetRatio
);
```

#### Yield Statistics

```solidity
function getYieldDistributionBreakdown() external view returns (
    uint256 userYieldPool,
    uint256 hedgerYieldPool,
    uint256 distributionRatio
);

function getYieldPerformanceMetrics() external view returns (
    uint256 totalYieldDistributed,
    uint256 averageUserYield,
    uint256 averageHedgerYield,
    uint256 yieldEfficiency
);
```

#### Historical Data

```solidity
function getHistoricalYieldShift(uint256 period) external view returns (
    uint256 averageShift,
    uint256 maxShift,
    uint256 minShift,
    uint256 volatility
);

function yieldShiftHistory(uint256 index) external view returns (uint128 yieldShift, uint64 timestamp);
function userPoolHistory(uint256 index) external view returns (uint128 userPoolSize, uint128 hedgerPoolSize, uint64 timestamp);
```

***

### ⚙️ Configuration

#### Yield Model

```solidity
struct YieldModelConfig {
    uint256 baseYieldShift;
    uint256 maxYieldShift;
    uint256 adjustmentSpeed;
    uint256 targetPoolRatio;
}

function configureYieldModel(YieldModelConfig calldata cfg) external;  // GOVERNANCE_ROLE
```

**Validations**

- `baseYieldShift` and `maxYieldShift` each a valid shift (≤ 10000)
- `maxYieldShift >= baseYieldShift` (`InvalidShiftRange`)
- `adjustmentSpeed <= 1000` (10%)
- `targetPoolRatio <= 50000` (500%)

#### Dependencies

```solidity
struct YieldDependencyConfig {
    address userPool;
    address hedgerPool;
    address mockAaveVault;     // legacy field, unused in production
    address stQEUROFactory;
    address treasury;
}

function configureDependencies(YieldDependencyConfig calldata cfg) external;  // GOVERNANCE_ROLE
```

***

### 🛡️ Security & Emergency Controls

```solidity
// Halt / resume yield distribution (EMERGENCY_ROLE)
function pauseYieldDistribution() external;
function resumeYieldDistribution() external;

// Manual split in an emergency (EMERGENCY_ROLE)
function emergencyYieldDistribution(uint256 userAmount, uint256 hedgerAmount) external;

// Recovery (DEFAULT_ADMIN_ROLE)
function recoverToken(address token, uint256 amount) external;
function recoverETH() external;
```

***

### 📋 Events

```solidity
event YieldDistributionUpdated(uint256 newYieldShift, uint256 userYieldAllocation, uint256 hedgerYieldAllocation, uint256 indexed timestamp);
event YieldAdded(uint256 yieldAmount, string indexed source, uint256 indexed timestamp);
event HedgerYieldClaimed(address indexed hedger, uint256 yieldAmount, uint256 timestamp);
event SourceVaultBindingUpdated(address indexed source, uint256 indexed vaultId);
event SourceVaultBindingModeUpdated(bool enabled);
```

***

### 📐 Calculation Examples

#### Example 1: Standard Distribution

```
Initial state:
- totalYield = 10,000 USDC
- currentYieldShift = 5000 (50%)

Result:
- userAllocation = 10,000 × 50% = 5,000 USDC → user yield pool
- hedgerAllocation = 10,000 × 50% = 5,000 USDC → hedger yield pool
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
→ Incentivizes more hedging capacity
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

> **Summary**: YieldShift is the allocation layer between the user and hedger sides of the protocol. It dynamically balances incentives via a TWAP-smoothed pool ratio, protects against flash deposits with a 7-day holding period, binds every yield source to its stQEURO series, and keeps the hedger's claimable yield ledger. Staker yield itself is credited directly by the vault into the stQEURO exchange rate.
