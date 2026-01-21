# Technical Appendices

## Technical Reference Documentation

This section provides a comprehensive technical reference for the Quantillon protocol smart contracts, including access roles, key functions, events, and custom errors.

***

## 1. Contract Architecture

### Overview

```
                    ┌─────────────────────┐
                    │   ChainlinkOracle   │
                    │   (EUR/USD, USDC)   │
                    └──────────┬──────────┘
                               │
            ┌──────────────────┼──────────────────┐
            │                  │                  │
   ┌────────▼────────┐ ┌───────▼───────┐ ┌───────▼───────┐
   │   QEUROToken    │ │QuantillonVault│ │   HedgerPool  │
   │ (ERC20 Stable)  │ │(Collateral Mgr)│ │(Hedger Margin)│
   └────────┬────────┘ └───────┬───────┘ └───────┬───────┘
            │                  │                  │
   ┌────────▼────────┐ ┌───────▼───────┐ ┌───────▼───────┐
   │  stQEUROToken   │ │   AaveVault   │ │  YieldShift   │
   │ (Yield Bearing) │ │(Aave v3 Yield)│ │(Distribution) │
   └────────┬────────┘ └───────────────┘ └───────────────┘
            │
   ┌────────▼────────┐
   │    QTIToken     │
   │  (Governance)   │
   └─────────────────┘
```

### Shared Libraries

| Library | Function |
|---------|----------|
| `CommonErrorLibrary` | Shared custom errors |
| `CommonValidationLibrary` | Input validations |
| `AccessControlLibrary` | Access control helpers |
| `VaultMath` | Mathematical calculations |
| `TreasuryRecoveryLibrary` | Token recovery |
| `TimeProvider` | Centralized time management |

***

## 2. Roles and Permissions

### QEUROToken

| Role | Hash Value | Permissions |
|------|------------|-------------|
| `DEFAULT_ADMIN_ROLE` | `0x00` | Role management, killswitch |
| `MINTER_ROLE` | `keccak256("MINTER_ROLE")` | Mint QEURO |
| `BURNER_ROLE` | `keccak256("BURNER_ROLE")` | Burn QEURO |
| `PAUSER_ROLE` | `keccak256("PAUSER_ROLE")` | Pause/unpause |
| `COMPLIANCE_ROLE` | `keccak256("COMPLIANCE_ROLE")` | Blacklist/whitelist |

### stQEUROToken

| Role | Hash Value | Permissions |
|------|------------|-------------|
| `DEFAULT_ADMIN_ROLE` | `0x00` | Role management |
| `GOVERNANCE_ROLE` | `keccak256("GOVERNANCE_ROLE")` | Parameters, treasury |
| `YIELD_MANAGER_ROLE` | `keccak256("YIELD_MANAGER_ROLE")` | Yield distribution |
| `EMERGENCY_ROLE` | `keccak256("EMERGENCY_ROLE")` | Pause, emergency withdraw |

### QTIToken

| Role | Hash Value | Permissions |
|------|------------|-------------|
| `DEFAULT_ADMIN_ROLE` | `0x00` | Role management |
| `GOVERNANCE_ROLE` | `keccak256("GOVERNANCE_ROLE")` | Governance parameters |
| `EMERGENCY_ROLE` | `keccak256("EMERGENCY_ROLE")` | Pause, cancel proposals |

### QuantillonVault

| Role | Hash Value | Permissions |
|------|------------|-------------|
| `DEFAULT_ADMIN_ROLE` | `0x00` | Role management |
| `GOVERNANCE_ROLE` | `keccak256("GOVERNANCE_ROLE")` | Vault parameters |
| `EMERGENCY_ROLE` | `keccak256("EMERGENCY_ROLE")` | Pause, recovery |
| `VAULT_OPERATOR_ROLE` | `keccak256("VAULT_OPERATOR_ROLE")` | Vault operations |

### HedgerPool

| Role | Hash Value | Permissions |
|------|------------|-------------|
| `DEFAULT_ADMIN_ROLE` | `0x00` | Role management |
| `GOVERNANCE_ROLE` | `keccak256("GOVERNANCE_ROLE")` | Hedging parameters |
| `EMERGENCY_ROLE` | `keccak256("EMERGENCY_ROLE")` | Emergency closure |
| `HEDGER_ROLE` | `keccak256("HEDGER_ROLE")` | Hedger operations |

### YieldShift

| Role | Hash Value | Permissions |
|------|------------|-------------|
| `DEFAULT_ADMIN_ROLE` | `0x00` | Role management |
| `GOVERNANCE_ROLE` | `keccak256("GOVERNANCE_ROLE")` | Yield shift parameters |
| `YIELD_MANAGER_ROLE` | `keccak256("YIELD_MANAGER_ROLE")` | Yield distribution |
| `EMERGENCY_ROLE` | `keccak256("EMERGENCY_ROLE")` | Pause distribution |

### AaveVault

| Role | Hash Value | Permissions |
|------|------------|-------------|
| `DEFAULT_ADMIN_ROLE` | `0x00` | Role management |
| `GOVERNANCE_ROLE` | `keccak256("GOVERNANCE_ROLE")` | Aave parameters |
| `VAULT_MANAGER_ROLE` | `keccak256("VAULT_MANAGER_ROLE")` | Deploy/withdraw Aave |
| `EMERGENCY_ROLE` | `keccak256("EMERGENCY_ROLE")` | Emergency withdrawal |

### ChainlinkOracle

| Role | Hash Value | Permissions |
|------|------------|-------------|
| `DEFAULT_ADMIN_ROLE` | `0x00` | Role management, recovery |
| `ORACLE_MANAGER_ROLE` | `keccak256("ORACLE_MANAGER_ROLE")` | Price bounds, feeds |
| `EMERGENCY_ROLE` | `keccak256("EMERGENCY_ROLE")` | Circuit breaker |
| `UPGRADER_ROLE` | `keccak256("UPGRADER_ROLE")` | Contract upgrades |

***

## 3. Important Constants

### QEUROToken

```solidity
uint256 public constant DEFAULT_MAX_SUPPLY = 1_000_000_000e18;  // 1 billion
uint256 public constant MAX_RATE_LIMIT = type(uint96).max;
uint256 public constant RATE_LIMIT_RESET_PERIOD = 1 hours;
uint256 public constant MINT_FEE_RATE = 10;  // 0.10% in basis points
```

### QTIToken

```solidity
uint256 public constant TOTAL_SUPPLY_CAP = 100_000_000e18;  // 100 million
uint256 public constant MAX_LOCK_TIME = 4 * 365 days;  // 4 years
uint256 public constant MIN_LOCK_TIME = 7 days;  // 1 week
uint256 public constant WEEK = 7 days;
uint256 public constant MAX_VE_QTI_MULTIPLIER = 4;  // 4x max voting power
```

### YieldShift

```solidity
uint256 public constant MIN_HOLDING_PERIOD = 7 days;
uint256 public constant TWAP_PERIOD = 24 hours;
uint256 public constant MAX_TIME_ELAPSED = 365 days;
uint256 public constant MAX_HISTORY_LENGTH = 1000;

// Defaults (configurable)
uint256 baseYieldShift = 5000;   // 50%
uint256 maxYieldShift = 9000;    // 90%
uint256 adjustmentSpeed = 100;    // 1%
uint256 targetPoolRatio = 10000;  // 100%
```

### ChainlinkOracle

```solidity
uint256 public constant MAX_PRICE_STALENESS = 3600;  // 1 hour
uint256 public constant MAX_PRICE_DEVIATION = 500;   // 5%
uint256 public constant BASIS_POINTS = 10000;
uint256 public constant MAX_TIMESTAMP_DRIFT = 900;   // 15 minutes

// Defaults (configurable)
uint256 minEurUsdPrice = 0.80e18;   // 0.80 USD/EUR
uint256 maxEurUsdPrice = 1.40e18;   // 1.40 USD/EUR
uint256 usdcToleranceBps = 200;     // 2%
```

### AaveVault

```solidity
// Defaults (configurable)
uint256 maxAaveExposure = 50_000_000e6;   // 50M USDC
uint256 harvestThreshold = 1000e6;         // 1000 USDC
uint256 yieldFee = 1000;                   // 10%
uint256 rebalanceThreshold = 500;          // 5%
uint256 utilizationLimit = 9500;           // 95%
```

***

## 4. Main Events

### QEUROToken

```solidity
event Transfer(address indexed from, address indexed to, uint256 amount);
event Approval(address indexed owner, address indexed spender, uint256 amount);
event Mint(address indexed to, uint256 amount);
event Burn(address indexed from, uint256 amount);
event BlacklistUpdated(address indexed account, bool status);
event WhitelistUpdated(address indexed account, bool status);
event WhitelistModeToggled(bool enabled);
event MintingKillswitchSet(bool enabled);
event MaxSupplyUpdated(uint256 newMaxSupply);
event TreasuryUpdated(address indexed newTreasury);
event FeeCollectorUpdated(address indexed newFeeCollector);
```

### stQEUROToken

```solidity
event Staked(address indexed user, uint256 qeuroAmount, uint256 stQEUROAmount);
event Unstaked(address indexed user, uint256 stQEUROAmount, uint256 qeuroAmount);
event YieldDistributed(uint256 yieldAmount, uint256 newExchangeRate);
event ExchangeRateUpdated(uint256 newRate, uint256 timestamp);
event YieldParametersUpdated(uint256 yieldFee, uint256 minThreshold);
event EmergencyWithdrawal(address indexed user, uint256 amount);
```

### QTIToken

```solidity
event Locked(address indexed user, uint256 amount, uint256 unlockTime, uint256 votingPower);
event Unlocked(address indexed user, uint256 amount);
event VotingPowerUpdated(address indexed user, uint256 newVotingPower);
event ProposalCreated(uint256 indexed proposalId, address indexed proposer, string description);
event Voted(uint256 indexed proposalId, address indexed voter, bool support, uint256 weight);
event ProposalExecuted(uint256 indexed proposalId);
event ProposalCancelled(uint256 indexed proposalId);
event GovernanceParametersUpdated(uint256 threshold, uint256 minPeriod, uint256 quorum);
event DecentralizationLevelUpdated(uint256 newLevel);
```

### QuantillonVault

```solidity
event QEUROMinted(address indexed user, uint256 usdcAmount, uint256 qeuroAmount, uint256 fee);
event QEURORedeemed(address indexed user, uint256 qeuroAmount, uint256 usdcAmount, uint256 fee);
event CollateralDeployed(uint256 amount, string destination);
event CollateralWithdrawn(uint256 amount, string source);
event ParametersUpdated(uint256 mintFee, uint256 redemptionFee);
event CollateralizationThresholdsUpdated(uint256 minRatio, uint256 criticalRatio);
event OracleUpdated(address indexed newOracle);
event EmergencyPaused(address indexed triggeredBy);
```

### HedgerPool

```solidity
event HedgePositionOpened(address indexed hedger, uint256 positionId, uint256 margin, uint256 leverage);
event HedgePositionClosed(address indexed hedger, uint256 positionId, int256 pnl);
event MarginAdded(address indexed hedger, uint256 positionId, uint256 amount);
event MarginRemoved(address indexed hedger, uint256 positionId, uint256 amount);
event UserMintRecorded(uint256 qeuroAmount, uint256 usdcVolume, uint256 price);
event UserRedeemRecorded(uint256 qeuroAmount, uint256 usdcVolume, uint256 price);
event HedgingParametersUpdated(uint256 minMarginRatio, uint256 maxLeverage);
event SingleHedgerSet(address indexed hedger);
event EmergencyPositionClosed(address indexed hedger, uint256 positionId);
```

### YieldShift

```solidity
event YieldDistributionUpdated(uint256 newYieldShift, uint256 userAllocation, uint256 hedgerAllocation, uint256 indexed timestamp);
event UserYieldClaimed(address indexed user, uint256 yieldAmount, uint256 timestamp);
event HedgerYieldClaimed(address indexed hedger, uint256 yieldAmount, uint256 timestamp);
event YieldAdded(uint256 yieldAmount, string indexed source, uint256 indexed timestamp);
event YieldShiftParametersUpdated(string indexed parameterType, uint256 baseYieldShift, uint256 maxYieldShift, uint256 adjustmentSpeed);
event YieldSourceAuthorized(address indexed source, bytes32 indexed yieldType);
event YieldSourceRevoked(address indexed source);
event HoldingPeriodProtectionUpdated(uint256 minHoldingPeriod, uint256 baseDiscount, uint256 maxTimeFactor);
```

### AaveVault

```solidity
event DeployedToAave(string indexed operationType, uint256 amount, uint256 aTokensReceived, uint256 newBalance);
event WithdrawnFromAave(string indexed operationType, uint256 amountRequested, uint256 amountWithdrawn, uint256 newBalance);
event AaveYieldHarvested(string indexed harvestType, uint256 yieldHarvested, uint256 protocolFee, uint256 netYield);
event AaveRewardsClaimed(address indexed rewardToken, uint256 rewardAmount, address recipient);
event PositionRebalanced(string indexed reason, uint256 oldAllocation, uint256 newAllocation);
event AaveParameterUpdated(string indexed parameter, uint256 oldValue, uint256 newValue);
event EmergencyWithdrawal(string indexed reason, uint256 amountWithdrawn, uint256 timestamp);
event EmergencyModeToggled(string indexed reason, bool enabled);
```

### ChainlinkOracle

```solidity
event PriceUpdated(uint256 eurUsdPrice, uint256 usdcUsdPrice, uint256 indexed timestamp);
event CircuitBreakerTriggered(uint256 attemptedPrice, uint256 lastValidPrice, string indexed reason);
event CircuitBreakerReset(address indexed admin);
event PriceBoundsUpdated(string indexed boundType, uint256 newMinPrice, uint256 newMaxPrice);
event PriceFeedsUpdated(address newEurUsdFeed, address newUsdcUsdFeed);
event TreasuryUpdated(address indexed newTreasury);
event ETHRecovered(address indexed to, uint256 amount);
event DevModeToggled(bool enabled, address indexed caller);
```

***

## 5. Custom Errors (CommonErrorLibrary)

```solidity
error ZeroAddress();
error InvalidAmount();
error InsufficientBalance();
error Unauthorized();
error NotAuthorized();
error AlreadyExists();
error DoesNotExist();
error Paused();
error NotPaused();
error WouldExceedLimit();
error InvalidShiftRange();
error HoldingPeriodNotMet();
error InsufficientYield();
error ConfigValueTooHigh();
error EmergencyModeActive();
error ExcessiveSlippage();
```

***

## 6. Emergency Recovery Functions

All contracts include emergency recovery functions:

```solidity
// Recover ERC20 tokens sent by mistake
function recoverToken(address token, uint256 amount) external onlyRole(DEFAULT_ADMIN_ROLE);

// Recover ETH sent by mistake
function recoverETH() external onlyRole(DEFAULT_ADMIN_ROLE);
```

These functions send recovered funds to the configured `treasury` address.

***

## 7. UUPS Upgrade Pattern

All upgradeable contracts use the UUPS (Universal Upgradeable Proxy Standard) pattern with:

1. **SecureUpgradeable**: Base contract with integrated timelock
2. **Timelock**: Mandatory delay before upgrade execution
3. **UPGRADER_ROLE**: Required permission for upgrades

```solidity
// Upgrade authorization (implemented in each contract)
function _authorizeUpgrade(address newImplementation) 
    internal 
    override 
    onlyRole(UPGRADER_ROLE) 
{
    // Additional validations possible
}
```

***

## 8. External Dependencies

### Chainlink Price Feeds (Base Mainnet)

| Feed | Decimals | Usage |
|------|----------|-------|
| EUR/USD | 8 | Main exchange rate |
| USDC/USD | 8 | USDC stability validation |

### Aave v3 (Base Mainnet)

| Contract | Function |
|----------|----------|
| PoolAddressesProvider | Pool address retrieval |
| Pool | Supply/Withdraw USDC |
| aUSDC | Yield receipt token |
| RewardsController | Claim Aave rewards |

***

## 9. Deployment References

> **Note**: Contract addresses will be published after mainnet deployment.

### Testnets

| Network | Status | Explorer |
|---------|--------|----------|
| Base Sepolia | Active | basescan.org/address/... |

### Mainnet

| Network | Status | Explorer |
|---------|--------|----------|
| Base Mainnet | Planned | basescan.org/address/... |

***

> **Documentation updated**: January 2026  
> **Contract version**: MVP v1.0
