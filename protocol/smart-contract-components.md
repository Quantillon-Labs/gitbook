# Smart Contract Components

## Technical Reference Documentation

This section is the technical inventory of the Quantillon protocol smart contracts on Base mainnet: contract list with live versions and upgrade paths, access roles and who holds them, events, custom errors and external dependencies. Per-contract pages hold the function-level detail.

***

## 1. Contract Architecture

### Overview

```
                    ┌─────────────────────┐        ┌─────────────────┐
                    │    OracleRouter     │◄───────│ SlippageStorage │◄── price publisher
                    │ slot 1: Hyperliquid │        └─────────────────┘
                    │ slot 0: Chainlink   │
                    └──────────┬──────────┘
                               │ IOracle
            ┌──────────────────┼──────────────────┐
            │                  │                  │
   ┌────────▼────────┐ ┌───────▼───────┐ ┌───────▼───────┐
   │   QEUROToken    │ │QuantillonVault│ │   HedgerPool  │
   │ (ERC20 Stable)  │ │(Collateral Mgr)│ │(Hedger Margin)│
   └────────┬────────┘ └───────┬───────┘ └───────┬───────┘
            │                  │                  │
   ┌────────▼────────┐ ┌───────▼───────┐ ┌───────▼───────┐
   │ stQEUROFactory  │ │ Staking Vault │ │  YieldShift   │
   │ → stQEUROToken  │ │Adapter (Morpho)│ │(Distribution) │
   │ (ERC-4626)      │ └───────────────┘ └───────────────┘
   └─────────────────┘
   UserPool (optional batch deposit/stake) · FeeCollector (60/25/15 fee split)
   QTIToken (governance, dormant) · TimeProvider (shared clock)
```

> The OracleRouter's market slot (slot 1) hosts `HyperliquidEurUsdOracle`; `ChainlinkOracle` in slot 0 is the one-transaction fallback and the USDC/USD source. See [Oracle Architecture](oracle-architecture.md).

### Live contracts (Base mainnet, chain 8453)

Versions read on-chain on 4 September 2026 (`version()` on each proxy). "Timelock" = UUPS upgrade through the 12-hour OZ TimelockController (`SecureUpgradeable`); "Safe-direct" = plain UUPS upgraded directly by the 2-of-3 governance Safe.

| Contract | Address | Version | Upgrade path |
|----------|---------|---------|--------------|
| `QuantillonVault` | `0x833E5Ba510a241b21F1C60c987D1c49eB52E4a07` | 1.1.11 | Timelock |
| `QEUROToken` | `0x69aD4e6c49d6275D0e11b5515D98a89f029869AA` | 1.0.6 | Timelock |
| `QTIToken` | `0x246c6F441c0f8Fc6A71Db0F12dB5665D373Df271` | 1.0.2 | Timelock |
| `UserPool` | `0x712bCc77e7aa53C79870A40d044D440Ad2901bF2` | 1.0.3 | Timelock |
| `HedgerPool` | `0xff5D7cE5c7671B2EA805Ee752B4f8eC9Ecf2975A` | 1.0.8 | Timelock |
| `YieldShift` | `0xdcd66568F8623bDa3387287c31F14b43e49665b1` | 1.0.5 | Timelock |
| `stQEUROFactory` | `0x0382B0b9FB6Ff737209C3B31D727BB9d2E2bcb53` | 1.0.1 | Timelock |
| `stQEUROToken` - series `stQEUROMORPHO1` (vaultId 2) | `0x17CD8ed967d17072297CcAe3D379C9e86aeBEb1d` | 1.0.3 | Timelock |
| `FeeCollector` | `0x0A33F72683cfC2303639d5cB9A45D77fF16d9FAD` | 1.0.2 | Safe-direct |
| `OracleRouter` | `0x7ED6aaEd83Db69509A88CAe5C247ef8fA44056E0` | 1.1.1 | Safe-direct |
| `ChainlinkOracle` | `0xaEE3c9c298051ef7242882AbCaE2Fd12d29443E7` | 1.0.4 | Safe-direct |
| `HyperliquidEurUsdOracle` | `0x0B58aBB57775E0fCEDfd4460e00dD9D9610C2C43` | 1.0.2 | Safe-direct |
| `SlippageStorage` | `0x0fde0ff2566be3c24af6d654012dddb4f1da099b` | 1.0.2 | Safe-direct |
| `MetaMorphoStakingVaultAdapter` (vaultId 2) | `0xb2f253Cd74ebfa16894339438B467396De9e8EA3` | - | Replaceable per `vaultId` (governance) |
| `TimeProvider` | `0x520236487CBD0a6958B4EefC7853cd7C3F5C56E7` | - | Deployed directly (no proxy) |

Governance: Safe (2-of-3) `0x1d7fF432a93d0085Fb69474c7E567f859829e6cd` · Timelock (12 h) `0x7Ade8f3Bf1FdaF0785efE9Ea5C6339D1aD6B8342`. See [Quantillon DAO](../quantillon-dao.md).

### Shared Libraries

| Library | Function |
|---------|----------|
| `CommonErrorLibrary` | Shared custom errors (69) |
| `VaultErrorLibrary`, `HedgerPoolErrorLibrary`, `TokenErrorLibrary` | Contract-specific errors |
| `CommonValidationLibrary`, `HedgerPoolValidationLibrary`, `YieldValidationLibrary`, `PriceValidationLibrary` | Input and price validations |
| `AccessControlLibrary` | Access control helpers |
| `VaultMath`, `HedgerPoolLogicLibrary`, `HedgerPoolRedeemMathLibrary`, `YieldShiftCalculationLibrary`, `StakingYieldLibrary` | Mathematical calculations |
| `TreasuryRecoveryLibrary` | Token recovery |
| `FlashLoanProtectionLibrary` | Same-block balance guards |

`TimeProvider` is a deployed contract (listed above), not a library: every core contract reads `TIME_PROVIDER.currentTime()`.

***

## 2. Roles and Permissions

### Who holds what

* The **2-of-3 governance Safe** holds every `DEFAULT_ADMIN_ROLE`, `GOVERNANCE_ROLE`, `UPGRADER_ROLE`, `ORACLE_MANAGER_ROLE`, `MANAGER_ROLE`, `TREASURY_ROLE` and `EMERGENCY_ROLE` in the stack, plus `PAUSER_ROLE` / `COMPLIANCE_ROLE` on QEURO.
* Narrow operational roles are delegated to dedicated wallets, each revocable by the Safe at any time: an **independent hedging watchdog** holds `EMERGENCY_ROLE` on `QuantillonVault` (pause/unpause only); **keeper wallets** designated by governance hold `VAULT_OPERATOR_ROLE` (deploy USDC to the external vault) and `YIELD_DISTRIBUTOR_ROLE` (harvest and distribute yield) on `QuantillonVault`; the **oracle publisher** holds `WRITER_ROLE` on `SlippageStorage`.
* Inter-contract roles: `QuantillonVault` holds `MINTER_ROLE` / `BURNER_ROLE` on QEURO, `VAULT_FACTORY_ROLE` on `stQEUROFactory`, `VAULT_MANAGER_ROLE` on the staking vault adapter and `FEE_SOURCE_ROLE` on `FeeCollector` (as does `HedgerPool`); `OracleRouter` holds `ORACLE_MANAGER_ROLE` and `EMERGENCY_ROLE` on `HyperliquidEurUsdOracle` for its pass-through admin functions.
* `HedgerPool` has **no hedger role**: hedger-only functions check the `singleHedger` address set by governance (Quantillon Labs' hedging engine in the current phase).

### QEUROToken

| Role | Hash Value | Permissions |
|------|------------|-------------|
| `DEFAULT_ADMIN_ROLE` | `0x00` | Role management, rate limits, supply ceiling (`updateMaxSupply`), treasury, fee collector, recovery |
| `MINTER_ROLE` | `keccak256("MINTER_ROLE")` | Mint QEURO (held by `QuantillonVault`) |
| `BURNER_ROLE` | `keccak256("BURNER_ROLE")` | Burn QEURO (held by `QuantillonVault`) |
| `PAUSER_ROLE` | `keccak256("PAUSER_ROLE")` | Pause/unpause, minting killswitch (`setMintingKillswitch`) |
| `COMPLIANCE_ROLE` | `keccak256("COMPLIANCE_ROLE")` | Blacklist/whitelist, whitelist mode |
| `UPGRADER_ROLE` | `keccak256("UPGRADER_ROLE")` | Upgrades (timelock) |

### stQEUROToken (each series)

| Role | Hash Value | Permissions |
|------|------------|-------------|
| `DEFAULT_ADMIN_ROLE` | `0x00` | Role management, recovery |
| `GOVERNANCE_ROLE` | `keccak256("GOVERNANCE_ROLE")` | `yieldFee` (max 20%), treasury |
| `EMERGENCY_ROLE` | `keccak256("EMERGENCY_ROLE")` | Pause/unpause, `emergencyWithdraw(user)` |
| `UPGRADER_ROLE` | `keccak256("UPGRADER_ROLE")` | Upgrades (timelock) |

Yield is credited to a series by `QuantillonVault.harvestAndDistributeVaultYield` - authorised on the vault (`YIELD_DISTRIBUTOR_ROLE`), not on the token.

### QTIToken

| Role | Hash Value | Permissions |
|------|------------|-------------|
| `DEFAULT_ADMIN_ROLE` | `0x00` | Role management, recovery |
| `GOVERNANCE_ROLE` | `keccak256("GOVERNANCE_ROLE")` | Governance parameters, `cancelProposal` (also the proposer), batch unlock, decentralization level, treasury |
| `EMERGENCY_ROLE` | `keccak256("EMERGENCY_ROLE")` | Pause/unpause |
| `UPGRADER_ROLE` | `keccak256("UPGRADER_ROLE")` | Upgrades (timelock) |

### QuantillonVault

| Role | Hash Value | Permissions |
|------|------------|-------------|
| `DEFAULT_ADMIN_ROLE` | `0x00` | Role management, recovery, dev-mode proposal/apply |
| `GOVERNANCE_ROLE` | `keccak256("GOVERNANCE_ROLE")` | Fees, collateralization thresholds, oracle/pool/collector wiring, staking-vault registration and activation, default vault, redemption priority, hedger funding rate and recipient, reward fee split, fee withdrawal |
| `EMERGENCY_ROLE` | `keccak256("EMERGENCY_ROLE")` | Pause/unpause (Safe and the independent watchdog) |
| `VAULT_OPERATOR_ROLE` | `keccak256("VAULT_OPERATOR_ROLE")` | `deployUsdcToVault` (keeper wallet) |
| `YIELD_DISTRIBUTOR_ROLE` | `keccak256("YIELD_DISTRIBUTOR_ROLE")` | `harvestAndDistributeVaultYield`, `creditVaultYield` (keeper wallet) |
| `UPGRADER_ROLE` | `keccak256("UPGRADER_ROLE")` | Upgrades (timelock) |

### UserPool

| Role | Hash Value | Permissions |
|------|------------|-------------|
| `DEFAULT_ADMIN_ROLE` | `0x00` | Role management, recovery |
| `GOVERNANCE_ROLE` | `keccak256("GOVERNANCE_ROLE")` | Staking parameters, performance fee, YieldShift wiring |
| `EMERGENCY_ROLE` | `keccak256("EMERGENCY_ROLE")` | Pause/unpause, `emergencyUnstake(user, recipient)` |
| `UPGRADER_ROLE` | `keccak256("UPGRADER_ROLE")` | Upgrades (timelock) |

### HedgerPool

| Role | Hash Value | Permissions |
|------|------------|-------------|
| `DEFAULT_ADMIN_ROLE` | `0x00` | Role management, recovery, `feeCollector` change |
| `GOVERNANCE_ROLE` | `keccak256("GOVERNANCE_ROLE")` | `configureRiskAndFees`, `configureDependencies`, `setSingleHedger` |
| `EMERGENCY_ROLE` | `keccak256("EMERGENCY_ROLE")` | `emergencyClosePosition`, pause/unpause |
| `UPGRADER_ROLE` | `keccak256("UPGRADER_ROLE")` | Upgrades (timelock) |
| - | - | Hedger operations: designated `singleHedger` address (no role) |

### YieldShift

| Role | Hash Value | Permissions |
|------|------------|-------------|
| `DEFAULT_ADMIN_ROLE` | `0x00` | Role management, recovery |
| `GOVERNANCE_ROLE` | `keccak256("GOVERNANCE_ROLE")` | Yield model (`configureYieldModel`), dependencies, yield-source authorisation, source→vault bindings, forced distribution update |
| `YIELD_MANAGER_ROLE` | `keccak256("YIELD_MANAGER_ROLE")` | `updateYieldAllocation` (allocation hook; held by the Safe) |
| `EMERGENCY_ROLE` | `keccak256("EMERGENCY_ROLE")` | Pause/resume distribution, `emergencyYieldDistribution` |
| `UPGRADER_ROLE` | `keccak256("UPGRADER_ROLE")` | Upgrades (timelock) |

### stQEUROFactory

| Role | Hash Value | Permissions |
|------|------------|-------------|
| `DEFAULT_ADMIN_ROLE` | `0x00` | Role management |
| `GOVERNANCE_ROLE` | `keccak256("GOVERNANCE_ROLE")` | Token implementation, token admin, oracle, YieldShift, treasury |
| `VAULT_FACTORY_ROLE` | `keccak256("VAULT_FACTORY_ROLE")` | `registerVault` - deploys a new stQEURO series (held by `QuantillonVault`, via `selfRegisterStQEURO`) |
| `UPGRADER_ROLE` | `keccak256("UPGRADER_ROLE")` | Upgrades (timelock) |

### FeeCollector

| Role | Hash Value | Permissions |
|------|------------|-------------|
| `DEFAULT_ADMIN_ROLE` | `0x00` | Role management |
| `GOVERNANCE_ROLE` | `keccak256("GOVERNANCE_ROLE")` | Fee ratios (live 6000/2500/1500 = 60% treasury / 25% dev fund / 15% community), fund addresses, fee-source authorisation, upgrades (Safe-direct) |
| `TREASURY_ROLE` | `keccak256("TREASURY_ROLE")` | `distributeFees(token)` |
| `FEE_SOURCE_ROLE` | `keccak256("FEE_SOURCE_ROLE")` | `collectFees` (held by `QuantillonVault` and `HedgerPool`) |
| `EMERGENCY_ROLE` | `keccak256("EMERGENCY_ROLE")` | Pause/unpause, `emergencyWithdraw(token)` |

### External Staking Vault Adapters

There is no standalone `AaveVault` contract (the name survives only in historical design documents). External yield exposure (currently a MetaMorpho USDC vault) goes through staking vault adapters registered on `QuantillonVault` (`setStakingVault`, `GOVERNANCE_ROLE`). USDC is deployed by `VAULT_OPERATOR_ROLE` (`deployUsdcToVault`), harvested by `YIELD_DISTRIBUTOR_ROLE` (`harvestAndDistributeVaultYield`) and withdrawn automatically to serve redemptions; there is no separate vault-level "emergency withdrawal" function - governance can deactivate a vault and pause the protocol. See [External Staking Vaults](external-staking-vaults.md).

| Role (adapter) | Hash Value | Permissions |
|------|------------|-------------|
| `GOVERNANCE_ROLE` | `keccak256("GOVERNANCE_ROLE")` | Re-point the adapter to another MetaMorpho vault |
| `VAULT_MANAGER_ROLE` | `keccak256("VAULT_MANAGER_ROLE")` | `depositUnderlying`, `withdrawUnderlying`, `harvestYieldToVault` (held by `QuantillonVault`) |

### OracleRouter

| Role | Hash Value | Permissions |
|------|------------|-------------|
| `DEFAULT_ADMIN_ROLE` | `0x00` | Treasury, recovery |
| `ORACLE_MANAGER_ROLE` | `keccak256("ORACLE_MANAGER_ROLE")` | `switchOracle`, `updateOracleAddresses`, pass-through bounds / tolerance / feeds / circuit breaker on the active oracle |
| `EMERGENCY_ROLE` | `keccak256("EMERGENCY_ROLE")` | Pause/unpause |
| `UPGRADER_ROLE` | `keccak256("UPGRADER_ROLE")` | Upgrades (Safe-direct) |

### HyperliquidEurUsdOracle

| Role | Hash Value | Permissions |
|------|------------|-------------|
| `DEFAULT_ADMIN_ROLE` | `0x00` | Treasury, recovery |
| `ORACLE_MANAGER_ROLE` | `keccak256("ORACLE_MANAGER_ROLE")` | Price bounds, USDC tolerance, `maxPriceStaleness` (≤ 1 h), slippage source, USDC source (Safe and `OracleRouter`) |
| `EMERGENCY_ROLE` | `keccak256("EMERGENCY_ROLE")` | Circuit breaker, pause/unpause (Safe and `OracleRouter`) |
| `UPGRADER_ROLE` | `keccak256("UPGRADER_ROLE")` | Upgrades (Safe-direct) |

### ChainlinkOracle

| Role | Hash Value | Permissions |
|------|------------|-------------|
| `DEFAULT_ADMIN_ROLE` | `0x00` | Role management, recovery, dev-mode proposal/apply |
| `ORACLE_MANAGER_ROLE` | `keccak256("ORACLE_MANAGER_ROLE")` | Price bounds, feeds, USDC tolerance, sequencer feed |
| `EMERGENCY_ROLE` | `keccak256("EMERGENCY_ROLE")` | Circuit breaker, pause |
| `UPGRADER_ROLE` | `keccak256("UPGRADER_ROLE")` | Upgrades (Safe-direct) |

### SlippageStorage

| Role | Hash Value | Permissions |
|------|------------|-------------|
| `DEFAULT_ADMIN_ROLE` | `0x00` | Treasury, recovery |
| `WRITER_ROLE` | `keccak256("WRITER_ROLE")` | `updateSlippage` / `updateSlippageBatch` (publisher wallet and the operational deployer key) |
| `MANAGER_ROLE` | `keccak256("MANAGER_ROLE")` | `minUpdateInterval` (live 20 s), deviation threshold, enabled sources, mid-price guards |
| `EMERGENCY_ROLE` | `keccak256("EMERGENCY_ROLE")` | Pause/unpause |
| `UPGRADER_ROLE` | `keccak256("UPGRADER_ROLE")` | Upgrades (Safe-direct) |

### TimeProvider

| Role | Hash Value | Permissions |
|------|------------|-------------|
| `GOVERNANCE_ROLE` | `keccak256("GOVERNANCE_ROLE")` | Time-offset controls (bounded to ±7 days; a test facility) |
| `EMERGENCY_ROLE` | `keccak256("EMERGENCY_ROLE")` | Emergency mode / reset |

***

## 3. Key Constants

Owner pages hold the full constant lists; this table is the cross-reference.

| Contract | Key constants / live values | Reference |
|----------|-----------------------------|-----------|
| `QEUROToken` | No tokenomic supply cap - supply bounded by hedging capacity (governance-set minting CR floor, currently 102.5%). `DEFAULT_MAX_SUPPLY = 100_000_000e18` (administrative ceiling, governance-raisable via `maxSupply`); global mint and burn rate limits 10M QEURO per 300-block window; mint/redeem fees 0 (max 5%, set on `QuantillonVault`) | [QEURO Token](quantillon-protocols-tokens/qeuro-token.md) |
| `QuantillonVault` | Minting floor 102.5% (hard minimum 101%), critical ratio 101%, `MAX_FUNDING_RATE_ANNUAL_BPS = 5000`, mint-time price-deviation guard 2% | [Liquidation Mode](liquidation-mode.md) |
| `HedgerPool` | Min margin 250 bps (contract floor), max leverage 20×, fees 0, interest 350/450 bps, `rewardFeeSplit` 20% | [HedgerPool](hedger-pool.md) |
| `UserPool` | stakingAPY 8%, depositAPY 4%, min stake 100 QEURO, cooldown 7 days, `MAX_BATCH_SIZE = 100` | [UserPool](user-pool.md) |
| `QTIToken` | `TOTAL_SUPPLY_CAP = 100_000_000e18`, `MIN_LOCK_TIME = 7 days`, `MAX_LOCK_TIME = 365 days`, `MAX_VE_QTI_MULTIPLIER = 4`, `PROPOSAL_EXECUTION_DELAY = 2 days` (dormant) | [QTI Token](quantillon-protocols-tokens/qti-token.md) |
| `YieldShift` | `MIN_HOLDING_PERIOD = 7 days`, `TWAP_PERIOD = 24 hours`, `MAX_HISTORY_LENGTH = 1000`; base 50% / max 90% / speed 1% / target ratio 100% | [YieldShift](yield-shift.md) |
| `stQEUROToken` | `yieldFee` 0 (max 20%) per series | [stQEURO Token](quantillon-protocols-tokens/stqeuro-token.md) |
| `FeeCollector` | Split 60/25/15 (treasury / dev fund / community) | this page |
| `ChainlinkOracle` | Staleness 2 h EUR/USD / 25 h USDC/USD, deviation 5%, drift 15 min, bounds 0.80–1.40, USDC tolerance 2%, sequencer grace 1 h | [ChainlinkOracle](chainlink-oracle.md) |
| `HyperliquidEurUsdOracle` | Staleness 900 s (hard cap 1 h), same bounds / deviation / tolerance | [Oracle Architecture](oracle-architecture.md) |
| External staking vaults | Active adapter `MetaMorphoStakingVaultAdapter` (vaultId 2); hedger funding carve-out governance-set, capped at 50% of each harvest - currently 0 bps | [External Staking Vaults](external-staking-vaults.md) |

***

## 4. Main Events

Generated from the deployed ABIs. Upgrade-related events shared by the timelock-gated contracts (`SecureUpgradeAuthorized`, `SecureUpgradesToggled`, `TimelockSet`, `EmergencyDisableProposed`, `EmergencyDisableApproved`) and the OpenZeppelin `Paused` / `Unpaused` / `Upgraded` / role events are omitted.

### QEUROToken

```solidity
event Transfer(address indexed from, address indexed to, uint256 value);
event Approval(address indexed owner, address indexed spender, uint256 value);
event TokensMinted(address indexed to, uint256 indexed amount, address indexed minter);
event TokensBurned(address indexed from, uint256 indexed amount, address indexed burner);
event AddressBlacklisted(address indexed account, string indexed reason);
event AddressUnblacklisted(address indexed account);
event AddressWhitelisted(address indexed account);
event AddressUnwhitelisted(address indexed account);
event WhitelistModeToggled(bool enabled);
event MintingKillswitchToggled(bool enabled, address indexed caller);
event SupplyCapUpdated(uint256 oldCap, uint256 newCap);
event RateLimitsUpdated(string indexed limitType, uint256 mintLimit, uint256 burnLimit);
event RateLimitReset(uint256 indexed blockNumber);
event MinPricePrecisionUpdated(uint256 oldPrecision, uint256 newPrecision);
event TreasuryUpdated(address indexed treasury);
event FeeCollectorUpdated(address indexed oldFeeCollector, address indexed newFeeCollector);
event ETHRecovered(address indexed to, uint256 indexed amount);
```

### stQEUROToken (ERC-4626)

```solidity
event Deposit(address indexed sender, address indexed owner, uint256 assets, uint256 shares);
event Withdraw(address indexed sender, address indexed receiver, address indexed owner, uint256 assets, uint256 shares);
event YieldParametersUpdated(uint256 yieldFee);
event TreasuryUpdated(address indexed oldTreasury, address indexed newTreasury, address indexed caller);
event ResidualSwept(address indexed receiver, uint256 amount);   // rounding residue of the final exit swept to its receiver
event ETHRecovered(address indexed to, uint256 indexed amount);
```

Yield crediting is emitted by the vault (`VaultYieldDistributed`); the series' exchange rate (`convertToAssets`) rises without a token-side event.

### QTIToken

```solidity
event TokensLocked(address indexed user, uint256 indexed amount, uint256 indexed unlockTime, uint256 votingPower);
event TokensUnlocked(address indexed user, uint256 indexed amount, uint256 votingPower);
event VotingPowerUpdated(address indexed user, uint256 oldPower, uint256 newPower);
event ProposalCreated(uint256 indexed proposalId, address indexed proposer, string description);
event Voted(uint256 indexed proposalId, address indexed voter, bool indexed support, uint256 votes);
event ProposalExecuted(uint256 indexed proposalId);
event ProposalCanceled(uint256 indexed proposalId);
event GovernanceParametersUpdated(string indexed parameterType, uint256 proposalThreshold, uint256 minVotingPeriod, uint256 quorumVotes);
event DecentralizationLevelUpdated(uint256 indexed newLevel);
event ETHRecovered(address indexed to, uint256 indexed amount);
```

### QuantillonVault

```solidity
// Mint / redeem
event QEUROminted(address indexed user, uint256 usdcAmount, uint256 qeuroAmount);
event QEURORedeemed(address indexed user, uint256 qeuroAmount, uint256 usdcAmount);
event LiquidationRedeemed(address indexed user, uint256 qeuroAmount, uint256 usdcPayout, uint256 collateralizationRatioBps, bool isPremium);
event PriceCacheUpdated(uint256 oldPrice, uint256 newPrice, uint256 blockNumber);
event PriceDeviationDetected(uint256 currentPrice, uint256 lastValidPrice, uint256 deviationBps, uint256 blockNumber);

// External staking vaults
event StakingVaultConfigured(uint256 indexed vaultId, address indexed adapter, bool active);
event StQEURORegistered(address indexed factory, uint256 indexed vaultId, address indexed stQEUROToken, string vaultName);
event DefaultStakingVaultUpdated(uint256 indexed previousVaultId, uint256 indexed newVaultId);
event RedemptionPriorityUpdated(uint256[] vaultIds);
event UsdcDeployedToExternalVault(uint256 indexed vaultId, uint256 indexed usdcAmount, uint256 principalInVault);
event UsdcWithdrawnFromExternalVault(uint256 indexed vaultId, uint256 indexed usdcAmount, uint256 principalInVault);
event VaultYieldDistributed(uint256 indexed vaultId, uint256 realizedYield, uint256 hedgerShare, uint256 userShare, uint256 treasuryShare);
event FundingRateUpdated(uint256 oldRateBps, uint256 newRateBps);
event HedgerYieldRecipientUpdated(address indexed oldRecipient, address indexed newRecipient);

// Hedger liquidity and fees
event HedgerDepositAdded(address indexed hedgerPool, uint256 usdcAmount, uint256 totalUsdcHeld);
event HedgerDepositWithdrawn(address indexed hedger, uint256 usdcAmount, uint256 totalUsdcHeld);
event HedgerRewardFeeSplitUpdated(uint256 previousSplit, uint256 newSplit);
event ProtocolFeeRouted(string sourceType, uint256 totalFee, uint256 hedgerReserveShare, uint256 collectorShare);

// Configuration
event ParametersUpdated(string indexed parameterType, uint256 mintFee, uint256 redemptionFee);
event CollateralizationThresholdsUpdated(uint256 indexed minCollateralizationRatioForMinting, uint256 indexed criticalCollateralizationRatio, address indexed caller);
event OracleUpdated(address indexed oldOracle, address indexed newOracle);
event DevModeProposed(bool pending, uint256 activatesAt);
event DevModeToggled(bool enabled, address indexed caller);
```

### UserPool

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

### HedgerPool

```solidity
event HedgePositionOpened(address indexed hedger, uint256 indexed positionId, bytes32 packedData);
event HedgePositionClosed(address indexed hedger, uint256 indexed positionId, bytes32 packedData);
event MarginUpdated(address indexed hedger, uint256 indexed positionId, bytes32 packedData);
event SingleHedgerRotationApplied(address indexed previousHedger, address indexed newHedger);
event EmergencyPositionClosed(address indexed hedger, uint256 indexed positionId, uint256 marginWithdrawn, uint256 outstandingQeuro);
event RewardReserveFunded(address indexed funder, uint256 amount);
event ETHRecovered(address indexed to, uint256 indexed amount);
```

### YieldShift

```solidity
event YieldDistributionUpdated(uint256 newYieldShift, uint256 userYieldAllocation, uint256 hedgerYieldAllocation, uint256 indexed timestamp);
event HedgerYieldClaimed(address indexed hedger, uint256 yieldAmount, uint256 timestamp);
event YieldAdded(uint256 yieldAmount, string indexed source, uint256 indexed timestamp);
event SourceVaultBindingUpdated(address indexed source, uint256 indexed vaultId);
event SourceVaultBindingModeUpdated(bool enabled);
```

### stQEUROFactory

```solidity
event VaultRegistered(uint256 indexed vaultId, address indexed vault, address indexed stQEUROToken, string vaultName);
event FactoryConfigUpdated(string indexed key, address oldValue, address newValue);
```

### FeeCollector

```solidity
event FeesCollected(address indexed token, uint256 amount, address indexed source, string indexed sourceType);
event FeesDistributed(address indexed token, uint256 totalAmount, uint256 treasuryAmount, uint256 devFundAmount, uint256 communityAmount);
event FeeRatiosUpdated(uint256 treasuryRatio, uint256 devFundRatio, uint256 communityRatio);
event FundAddressesUpdated(address treasury, address devFund, address communityFund);
```

### OracleRouter

```solidity
event OracleSwitched(uint8 indexed oldOracle, uint8 indexed newOracle, address indexed caller);
event OracleAddressesUpdated(address newChainlinkOracle, address newMarketOracle);
event TreasuryUpdated(address indexed newTreasury);
event ETHRecovered(address indexed to, uint256 amount);
```

### HyperliquidEurUsdOracle

```solidity
event PriceUpdated(uint256 eurUsdPrice, uint256 usdcUsdPrice, uint256 indexed timestamp);
event CircuitBreakerTriggered(uint256 attemptedPrice, uint256 lastValidPrice, string indexed reason);
event CircuitBreakerReset(address indexed admin);
event PriceBoundsUpdated(string indexed boundType, uint256 newMinPrice, uint256 newMaxPrice);
event MaxStalenessUpdated(uint256 oldStaleness, uint256 newStaleness);
event SlippageSourceUpdated(address indexed newSlippageStorage, uint8 newSourceId);
event UsdcSourceUpdated(address indexed newUsdcSource);
event TreasuryUpdated(address indexed newTreasury);
event ETHRecovered(address indexed to, uint256 amount);
```

### SlippageStorage

```solidity
event SlippageUpdated(uint128 midPrice, uint16 worstCaseBps, uint16 spreadBps, uint128 depthEur, uint48 timestamp);
event SlippageSourceUpdated(uint8 indexed sourceId, uint128 midPrice, uint16 worstCaseBps, uint16 spreadBps, uint128 depthEur, uint48 timestamp);
event EnabledSourcesUpdated(uint8 oldMask, uint8 newMask);
event ConfigUpdated(string indexed param, uint256 oldValue, uint256 newValue);
event TreasuryUpdated(address indexed newTreasury);
event ETHRecovered(address indexed to, uint256 amount);
```

### ChainlinkOracle

See [ChainlinkOracle](chainlink-oracle.md#events).

### MetaMorphoStakingVaultAdapter

```solidity
event MetaMorphoVaultUpdated(address indexed oldVault, address indexed newVault);
```

***

## 5. Custom Errors

`CommonErrorLibrary` holds 69 shared errors; the most common reverts are:

```solidity
error ZeroAddress();
error InvalidAddress();
error InvalidAmount();
error InsufficientBalance();
error NotAuthorized();
error NotActive();
error NoChangeDetected();
error ConfigValueTooHigh();
error ConfigValueTooLow();
error AboveLimit();
error WouldExceedLimit();
error ExcessiveSlippage();
error InvalidOraclePrice();
error InsufficientCollateralization();
error HoldingPeriodNotMet();
error InvalidShiftRange();
error InsufficientYield();
error EmergencyModeActive();
```

Contract-specific errors live in `VaultErrorLibrary`, `HedgerPoolErrorLibrary` (listed on the [HedgerPool](hedger-pool.md) page) and `TokenErrorLibrary` (e.g. `RateLimitExceeded`, `MintingDisabled`, `BlacklistedAddress`, `NewCapBelowCurrentSupply`). The full lists are in the public smart-contracts repository (`src/libraries/`).

***

## 6. Emergency Recovery Functions

All contracts include recovery functions for tokens or ETH sent by mistake:

```solidity
// Recover ERC20 tokens sent by mistake (HedgerPool: recover(token, amount))
function recoverToken(address token, uint256 amount) external onlyRole(DEFAULT_ADMIN_ROLE);

// Recover ETH sent by mistake
function recoverETH() external onlyRole(DEFAULT_ADMIN_ROLE);
```

These functions send recovered funds to the configured `treasury` address and cannot recover a contract's own operating token.

***

## 7. UUPS Upgrade Pattern

All contracts are UUPS (Universal Upgradeable Proxy Standard) proxies, but they follow two upgrade paths:

| Path | Contracts | Mechanism |
|------|-----------|-----------|
| **Timelock (12 h)** | `QuantillonVault`, `QEUROToken`, `QTIToken`, `UserPool`, `HedgerPool`, `YieldShift`, `stQEUROFactory`, `stQEUROToken` | `SecureUpgradeable`: the Safe schedules the upgrade on the OZ `TimelockController` (`0x7Ade8f3B…8342`, `minDelay` 43,200 s) and executes it after the delay; `UPGRADER_ROLE` on the contract |
| **Safe-direct** | `FeeCollector`, `OracleRouter`, `ChainlinkOracle`, `HyperliquidEurUsdOracle`, `SlippageStorage` | Plain UUPS: the 2-of-3 Safe calls `upgradeToAndCall` directly (`UPGRADER_ROLE`, or `GOVERNANCE_ROLE` on `FeeCollector`); no timelock |

```solidity
// Upgrade authorization (implemented in each contract)
function _authorizeUpgrade(address newImplementation) internal override onlyRole(UPGRADER_ROLE) {}
```

`SecureUpgradeable` also carries an emergency-disable path for the timelock that requires a quorum of approvals and its own delay (`EMERGENCY_DISABLE_QUORUM`, `EMERGENCY_DISABLE_DELAY`), so the timelock cannot be switched off unilaterally. See [Quantillon DAO](../quantillon-dao.md).

***

## 8. External Dependencies

### Oracle Sources (Base Mainnet)

| Source | Usage |
|--------|-------|
| Hyperliquid EUR/USD market mid (via `SlippageStorage` → `HyperliquidEurUsdOracle`, source `SOURCE_HYPERLIQUID = 1`) | Active EUR/USD source (OracleRouter slot 1) |
| Chainlink EUR/USD feed `0xc91D87E81faB8f93699ECf7Ee9B44D11e1D53F0F` (8 decimals) | Fallback EUR/USD source (OracleRouter slot 0) |
| Chainlink USDC/USD feed `0x7e860098F58bBFC8648a4311b374B1D669a2bc6B` (8 decimals) | USDC stability validation |
| Chainlink Base L2 sequencer uptime feed `0xBCF85224fc0756B9Fa45aA7892530B47e10b6433` | Sequencer check on the Chainlink path (1 h grace) |

### Tokens and Venues (Base Mainnet)

| Contract | Function |
|----------|----------|
| USDC `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` | Collateral token |
| MetaMorpho USDC vault `0xBEEFE94c8aD530842bfE7d8B397938fFc1cb83b2` (ERC-4626) | Yield-generating USDC deposit venue |
| `MetaMorphoStakingVaultAdapter` `0xb2f253Cd74ebfa16894339438B467396De9e8EA3` (vaultId 2) | Adapter between QuantillonVault and the MetaMorpho vault |

The EUR/USD hedge itself is executed off-chain on Hyperliquid by the designated hedger - see [HedgerPool](hedger-pool.md).

***

## 9. Deployment References

Base mainnet (chain 8453) - **contracts deployed since June 2026; public launch planned for Q4 2026**. All addresses are listed in §1; every contract is verified on Basescan. Base Sepolia was used for pre-launch testing; the testnet deployment is historical and no longer maintained as a reference environment.

***

> **Documentation updated**: 4 September 2026 - versions and role holders verified on-chain.
