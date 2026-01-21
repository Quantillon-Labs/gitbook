# AaveVault

## AaveVault: Aave v3 Integration for Yield Generation

### 📋 Overview

The AaveVault is the contract that manages integration with the Aave v3 protocol. It deploys protocol USDC to Aave for yield generation via lending, then distributes revenue to the YieldShift system.

***

### 🏗️ Contract Architecture

**Inheritance**

```solidity
contract AaveVault is 
    Initializable,
    ReentrancyGuardUpgradeable,
    AccessControlUpgradeable,
    PausableUpgradeable,
    SecureUpgradeable
```

**Aave Dependencies**

| Interface | Variable | Role |
|-----------|----------|------|
| `IERC20` | `usdc` | Underlying USDC token |
| `IERC20` | `aUSDC` | Aave deposit token |
| `IPool` | `aavePool` | Main Aave pool |
| `IPoolAddressesProvider` | `aaveProvider` | Aave address registry |
| `IRewardsController` | `rewardsController` | Aave rewards (if available) |
| `IYieldShift` | `yieldShift` | Yield distribution |

***

### 🔐 Roles & Permissions

| Role | Responsibilities |
|------|-----------------|
| `DEFAULT_ADMIN_ROLE` | General administration |
| `GOVERNANCE_ROLE` | Parameters (exposure, fees, thresholds) |
| `VAULT_MANAGER_ROLE` | Deploy/withdraw operations |
| `EMERGENCY_ROLE` | Emergency mode, pause, emergency withdrawal |

***

### ⚙️ Configuration Parameters

#### Limits and Thresholds

| Parameter | Type | Description | Default Value |
|-----------|------|-------------|---------------|
| `maxAaveExposure` | `uint256` | Max Aave exposure (USDC 6 dec) | 50,000,000e6 (50M) |
| `harvestThreshold` | `uint256` | Min yield for harvest (USDC) | 1,000e6 (1K) |
| `rebalanceThreshold` | `uint256` | Threshold to trigger rebalance (BPS) | 500 (5%) |
| `utilizationLimit` | `uint256` | Aave utilization limit (BPS) | 9500 (95%) |
| `emergencyExitThreshold` | `uint256` | Emergency exit threshold (BPS) | 110 (1.1%) |

#### Fees

| Parameter | Type | Description | Default Value |
|-----------|------|-------------|---------------|
| `yieldFee` | `uint256` | Fee on harvested yield (BPS) | 1000 (10%) |

***

### 💰 Deployment Operations

#### Deploy to Aave

```solidity
function deployToAave(uint256 amount) 
    external onlyRole(VAULT_MANAGER_ROLE) whenNotPaused nonReentrant;
```

**Flow**

```
1. Verify emergencyMode == false
2. Verify current + amount <= maxAaveExposure
3. Verify _isAaveHealthy() == true
4. Approve USDC for aavePool
5. aavePool.supply(usdc, amount, address(this), 0)
6. principalDeposited += amount
7. Emit DeployedToAave event
```

**Validations**

| Check | Error if Failed |
|-------|-----------------|
| `!emergencyMode` | `EmergencyModeActive` |
| `newExposure <= maxAaveExposure` | `MaxExposureExceeded` |
| `_isAaveHealthy()` | `AaveNotHealthy` |

#### Withdraw from Aave

```solidity
function withdrawFromAave(uint256 amount) 
    external onlyRole(VAULT_MANAGER_ROLE) whenNotPaused nonReentrant;
```

**Flow**

```
1. Verify amount <= aUSDC.balanceOf(this)
2. aavePool.withdraw(usdc, amount, address(this))
3. Adjust principalDeposited (proportionally)
4. Emit WithdrawnFromAave event
```

***

### 🏥 Health Monitoring

#### _isAaveHealthy Function

```solidity
function _isAaveHealthy() internal view returns (bool);
```

**Checks Performed**

```
┌──────────────────────────────────────────────────────────┐
│                    AAVE HEALTH CHECKS                     │
├──────────────────────────────────────────────────────────┤
│ 1. Utilization Rate                                       │
│    └── currentUtilization <= utilizationLimit (95%)       │
│                                                           │
│ 2. Pool Liquidity                                         │
│    └── availableLiquidity > 0                             │
│                                                           │
│ 3. Reserve Status                                         │
│    └── Reserve is active and not frozen                   │
│                                                           │
│ 4. aToken Balance                                         │
│    └── aUSDC balance reflects deposits correctly          │
└──────────────────────────────────────────────────────────┘
```

**When to Use**

- Before each deployment (`deployToAave`)
- During periodic health checks
- Before auto-rebalance

***

### 🔄 Auto-Rebalancing

#### autoRebalance Function

```solidity
function autoRebalance() 
    external onlyRole(VAULT_MANAGER_ROLE) whenNotPaused nonReentrant;
```

**When to Trigger a Rebalance**

```
currentAllocation = principalDeposited / totalAvailableUSDC
optimalAllocation = calculateOptimalAllocation()

IF |currentAllocation - optimalAllocation| > rebalanceThreshold:
    → Execute rebalance
```

**Rebalance Flow**

```
┌───────────────────────────────────────────────────────────┐
│                    REBALANCE FLOW                          │
├───────────────────────────────────────────────────────────┤
│  IF optimalAllocation > currentAllocation:                │
│     → Deploy more to Aave                                  │
│                                                            │
│  IF optimalAllocation < currentAllocation:                │
│     → Withdraw from Aave                                   │
│                                                            │
│  EMIT PositionRebalanced(reason, oldAlloc, newAlloc)      │
└───────────────────────────────────────────────────────────┘
```

#### Optimal Allocation Calculation

```solidity
function calculateOptimalAllocation() 
    public view returns (uint256 optimalPercent, uint256 expectedYield);
```

**Factors Considered**

| Factor | Weight | Description |
|--------|--------|-------------|
| **Aave APY** | Primary | Current lending yield |
| **Utilization** | Safety | Available liquidity |
| **Protocol Needs** | Reserve | Buffer for redemptions |

**Simplified Formula**

```
optimalAllocation = min(
    maxAaveExposure,
    totalUSDC × (1 - liquidityBuffer) × healthFactor
)
```

***

### 🌾 Yield Harvesting

#### harvestYield Function

```solidity
function harvestYield() 
    external onlyRole(VAULT_MANAGER_ROLE) whenNotPaused nonReentrant
    returns (uint256 netYield);
```

**Yield Calculation**

```
currentValue = aUSDC.balanceOf(address(this))
yieldGenerated = currentValue - principalDeposited

IF yieldGenerated >= harvestThreshold:
    protocolFee = yieldGenerated × yieldFee / 10000
    netYield = yieldGenerated - protocolFee
    
    → Withdraw yield from Aave
    → Send fee to treasury
    → Send netYield to YieldShift
```

**Tracking**

```solidity
uint256 public lastHarvestTime;      // Last harvest timestamp
uint256 public totalYieldHarvested;  // Total yield harvested
uint256 public totalFeesCollected;   // Total fees collected
```

#### Claim Aave Rewards

```solidity
function claimAaveRewards() 
    external onlyRole(VAULT_MANAGER_ROLE) nonReentrant
    returns (uint256 rewardsClaimed);
```

If Aave distributes rewards (bonus tokens), this function collects them.

***

### 🚨 Emergency Mode

#### Activation

```solidity
bool public emergencyMode;

function toggleEmergencyMode(bool enabled) 
    external onlyRole(EMERGENCY_ROLE);
```

**Emergency Mode Impact**

| Function | Normal | Emergency Mode |
|----------|--------|----------------|
| `deployToAave` | ✅ Allowed | ❌ Blocked |
| `withdrawFromAave` | ✅ Allowed | ✅ Allowed |
| `harvestYield` | ✅ Allowed | ⚠️ Limited |
| `emergencyWithdrawAll` | ❌ Blocked | ✅ Allowed |

#### Emergency Withdraw All

```solidity
function emergencyWithdrawAll() 
    external onlyRole(EMERGENCY_ROLE) nonReentrant;
```

**Flow**

```
1. Verify emergencyMode == true
2. Withdraw ALL funds from Aave
3. USDC stays in vault (no distribution)
4. Emit EmergencyWithdrawal event
```

**When to Use**

- Vulnerability detected on Aave
- Aave liquidity problem
- Urgent protocol liquidity need
- Security incident

***

### 📊 View Functions

#### Main Metrics

```solidity
function getVaultStatus() external view returns (
    uint256 principalDeposited,
    uint256 currentValue,
    uint256 unharvestedYield,
    uint256 totalYieldHarvested,
    bool emergencyMode,
    bool isHealthy
);
```

#### Aave Metrics

```solidity
function getAaveMetrics() external view returns (
    uint256 currentAPY,
    uint256 utilizationRate,
    uint256 availableLiquidity,
    bool isActive
);
```

#### Risk Metrics

```solidity
function getRiskMetrics() external view returns (
    uint256 exposureRatio,       // Current exposure ratio
    uint256 maxExposure,         // Max limit
    uint256 utilizationBuffer,   // Margin before limit
    bool isWithinLimits          // Within limits?
);
```

***

### ⚙️ Configuration

#### Update Parameters

```solidity
function updateAaveParameters(
    uint256 _maxAaveExposure,
    uint256 _harvestThreshold,
    uint256 _yieldFee,
    uint256 _rebalanceThreshold
) external onlyRole(GOVERNANCE_ROLE);
```

#### Update Safety Thresholds

```solidity
function updateSafetyThresholds(
    uint256 _utilizationLimit,
    uint256 _emergencyExitThreshold
) external onlyRole(GOVERNANCE_ROLE);
```

***

### 🛡️ Security

#### Implemented Protections

| Protection | Description |
|------------|-------------|
| **Max Exposure** | Absolute deployment limit |
| **Utilization Check** | Verify available liquidity |
| **Health Monitoring** | Checks before each operation |
| **Emergency Mode** | Quick exit if necessary |
| **Reentrancy Guard** | Reentrancy protection |
| **Role-Based Access** | Permission separation |

#### Recovery

```solidity
function recoverToken(address token, uint256 amount) 
    external onlyRole(DEFAULT_ADMIN_ROLE);

function recoverETH() external onlyRole(DEFAULT_ADMIN_ROLE);
```

> **Note**: Cannot recover active USDC or aUSDC.

***

### 📋 Events

```solidity
event DeployedToAave(
    string indexed operationType, 
    uint256 amount, 
    uint256 aTokensReceived, 
    uint256 newBalance
);

event WithdrawnFromAave(
    string indexed operationType, 
    uint256 amountRequested, 
    uint256 amountWithdrawn, 
    uint256 newBalance
);

event AaveYieldHarvested(
    string indexed harvestType, 
    uint256 yieldHarvested, 
    uint256 protocolFee, 
    uint256 netYield
);

event AaveRewardsClaimed(
    address indexed rewardToken, 
    uint256 rewardAmount, 
    address recipient
);

event PositionRebalanced(
    string indexed reason, 
    uint256 oldAllocation, 
    uint256 newAllocation
);

event AaveParameterUpdated(
    string indexed parameter, 
    uint256 oldValue, 
    uint256 newValue
);

event EmergencyWithdrawal(
    string indexed reason, 
    uint256 amountWithdrawn, 
    uint256 timestamp
);

event EmergencyModeToggled(
    string indexed reason, 
    bool enabled
);
```

***

### 📐 Calculation Examples

#### Example 1: Yield Harvest

```
State:
- principalDeposited = 10,000,000 USDC
- aUSDC balance = 10,500,000 USDC (after interest)
- yieldFee = 1000 (10%)

Calculation:
- yieldGenerated = 10,500,000 - 10,000,000 = 500,000 USDC
- protocolFee = 500,000 × 10% = 50,000 USDC → Treasury
- netYield = 500,000 - 50,000 = 450,000 USDC → YieldShift
```

#### Example 2: Rebalance

```
Initial state:
- totalProtocolUSDC = 20,000,000 USDC
- principalDeposited = 8,000,000 USDC (40%)
- optimalAllocation = 60%
- rebalanceThreshold = 5%

Decision:
- Gap = |40% - 60%| = 20% > 5% threshold
→ Trigger rebalance
→ Deploy 4,000,000 additional USDC to Aave
→ New allocation = 60%
```

***

### 🔗 Protocol Integration

```
┌─────────────────────────────────────────────────────────────┐
│                    PROTOCOL INTEGRATION                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  QuantillonVault                                            │
│       │                                                      │
│       │ USDC for deployment                                  │
│       ▼                                                      │
│  AaveVault                                                  │
│       │                                                      │
│       │ supply/withdraw                                      │
│       ▼                                                      │
│  Aave v3 Protocol                                           │
│       │                                                      │
│       │ Yield (aUSDC appreciation)                          │
│       ▼                                                      │
│  AaveVault.harvestYield()                                   │
│       │                                                      │
│       ├── 10% → Treasury (yieldFee)                         │
│       │                                                      │
│       └── 90% → YieldShift                                  │
│              │                                               │
│              ├── Users (stQEURO)                             │
│              └── Hedgers (HedgerPool)                        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

***

> **Summary**: The AaveVault manages integration with Aave v3 to generate yield on protocol USDC. It includes health monitoring, auto-rebalancing, and emergency mode mechanisms to ensure fund security. Yield is harvested periodically and distributed via YieldShift after deducting protocol fees (10%).
