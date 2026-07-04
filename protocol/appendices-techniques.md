# Technical Appendices

## Technical Reference Documentation

This section provides a comprehensive technical reference for the Quantillon protocol smart contracts, including access roles, key functions, events, and custom errors.

***

## 1. Contract Architecture

### Overview

```
                    ┌─────────────────────┐
                    │    OracleRouter     │
                    │(Hyperliquid + Chainlink)
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
   │  stQEUROToken   │ │ Staking Vault │ │  YieldShift   │
   │ (Yield Bearing) │ │Adapter (Morpho)│ │(Distribution) │
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

### External Staking Vault Adapters

There is no standalone `AaveVault` contract. External yield exposure (currently a MetaMorpho USDC vault) goes through staking vault adapters registered on `QuantillonVault`; deployment, harvest, and emergency withdrawal are gated by the vault's `GOVERNANCE_ROLE`/`EMERGENCY_ROLE`. See [External Staking Vaults](aave-vault.md).

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
uint256 public constant MAX_SUPPLY = 100_000_000e18;  // 100 million QEURO
// Mint rate limit: 10,000,000 QEURO per 300-second window
// Mint/redeem fees: currently 0, governance-settable, capped at 5% (QuantillonVault)
```

### QTIToken

```solidity
uint256 public constant TOTAL_SUPPLY_CAP = 100_000_000e18;  // 100 million
uint256 public constant MAX_LOCK_TIME = 365 days;  // 1 year
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
uint256 public constant MAX_PRICE_STALENESS = 7200;       // 2 hours (EUR/USD)
uint256 public constant MAX_USDC_PRICE_STALENESS = 25 hours;  // USDC/USD daily heartbeat
uint256 public constant MAX_PRICE_DEVIATION = 500;   // 5%
uint256 public constant BASIS_POINTS = 10000;
uint256 public constant MAX_TIMESTAMP_DRIFT = 900;   // 15 minutes

// Defaults (configurable)
uint256 minEurUsdPrice = 0.80e18;   // 0.80 USD/EUR
uint256 maxEurUsdPrice = 1.40e18;   // 1.40 USD/EUR
uint256 usdcToleranceBps = 200;     // 2%
```

### External Staking Vaults (QuantillonVault)

```
// Live configuration
Active adapter: MetaMorphoStakingVaultAdapter (vaultId 2)
Hedger funding: governance-set annual rate, capped at 50% of each harvest
stQEURO yieldFee: currently 0, capped at 20% per series
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

### External Staking Vaults (QuantillonVault)

```solidity
event VaultYieldDistributed(uint256 indexed vaultId, uint256 realizedYield, uint256 hedgerShare, uint256 userShare, uint256 treasuryShare);
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

### Oracle Sources (Base Mainnet)

| Source | Usage |
|--------|-------|
| Hyperliquid EUR/USD market mid (via `SlippageStorage` → `HyperliquidEurUsdOracle`) | Active EUR/USD source (OracleRouter slot 1) |
| Chainlink EUR/USD feed (8 decimals) | Fallback EUR/USD source (OracleRouter slot 0) |
| Chainlink USDC/USD feed (8 decimals) | USDC stability validation |

### Morpho / MetaMorpho (Base Mainnet)

| Contract | Function |
|----------|----------|
| MetaMorpho USDC vault (ERC-4626) | Yield-generating USDC deposit venue |
| `MetaMorphoStakingVaultAdapter` (`0xb2f253Cd74ebfa16894339438B467396De9e8EA3`, vaultId 2) | Adapter between QuantillonVault and the MetaMorpho vault |

***

## 9. Deployment References

### Mainnet

| Network | Status | Key Addresses |
|---------|--------|---------------|
| Base Mainnet (8453) | **Live since June 2026** | QuantillonVault `0x833E5Ba510a241b21F1C60c987D1c49eB52E4a07` · Governance Safe (2-of-3) `0x1d7fF432a93d0085Fb69474c7E567f859829e6cd` · Timelock (12h) `0x7Ade8f3Bf1FdaF0785efE9Ea5C6339D1aD6B8342` |

Oracle contract addresses are listed in [Oracle Architecture](oracle-architecture.md).

### Testnets

Base Sepolia was used for pre-launch testing; the testnet deployment is historical and no longer maintained as a reference environment.

***

> **Documentation updated**: July 2026  
> **Contract version**: QuantillonVault v1.1.1
