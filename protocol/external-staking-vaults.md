# External Staking Vaults

## External Staking Vaults: Yield Generation Through Adapters

### 📋 Overview

The Quantillon Protocol generates yield by deploying part of its USDC collateral into **external yield vaults** (money markets such as Morpho or Aave) through a standardized adapter layer. Each external vault is onboarded under a numeric **`vaultId`** and receives its own dedicated **stQEURO series** deployed by the `stQEUROFactory`.

> **There is no `AaveVault` contract** (the name is historical). Earlier protocol designs described a monolithic Aave-specific vault; the production architecture replaced it with lightweight, vault-agnostic adapters implementing a common `IExternalStakingVault` interface. Aave and Morpho are both supported through this same pattern; only the MetaMorpho adapter is deployed and registered today.

***

### 🏗️ The Adapter Pattern

Every external vault is wrapped by an adapter that exposes an identical, minimal interface to `QuantillonVault`:

| Function | Purpose |
| --- | --- |
| `depositUnderlying(uint256 usdcAmount)` | Deploys USDC from the protocol into the external vault |
| `withdrawUnderlying(uint256 usdcAmount)` | Withdraws USDC back to the protocol |
| `harvestYieldToVault()` | Realizes accrued yield and returns it to `QuantillonVault` |
| `totalUnderlying()` | Reports the current principal + accrued value |

Because all adapters share this interface, the protocol can onboard, migrate, or retire external vaults without changing core contracts - governance simply registers the adapter under a `vaultId` and the runtime routes by that id.

Available adapter implementations:

* **`MetaMorphoStakingVaultAdapter`** - wraps a MetaMorpho (Morpho) vault. **This is the adapter currently deployed and registered on Base mainnet.**
* **`MorphoStakingVaultAdapter`** - Morpho markets adapter (symmetric pattern).
* **`AaveStakingVaultAdapter`** - Aave-style adapter (symmetric pattern), used with mock vaults for local development and available should governance onboard an Aave market; no such onboarding is scheduled.

***

### 🚀 Deployed Contracts (Base Mainnet)

| Component | Value |
| --- | --- |
| Active external vault | MetaMorpho USDC vault (`0xBEEFE94c8aD530842bfE7d8B397938fFc1cb83b2`) |
| Adapter | `MetaMorphoStakingVaultAdapter` - `0x4c9B8b09214d37D5310b8E6768cF28E0dDcEDC30` |
| `vaultId` | `2` |
| stQEURO series | `stQEUROMORPHO1` - `0x17CD8ed967d17072297CcAe3D379C9e86aeBEb1d` |

> The current adapter binding was verified through `getVaultExposure(2)` on 30 September 2026. The staking-token series is unchanged; see [Smart Contract Components](smart-contract-components.md) for the current inventory.

***

### 🏭 One stQEURO Series per Vault

`stQEUROFactory` deploys a dedicated stQEURO proxy for every registered external vault and keeps the registry:

* `getStQEUROByVaultId(vaultId)` → the stQEURO token for that vault
* `getVaultById(vaultId)` → the `QuantillonVault` that registered the series (not the external vault)
* `getVaultName(vaultId)` → human-readable label (e.g. `MORPHO1`)
* `QuantillonVault.getVaultExposure(vaultId)` → the adapter address, its active flag, the principal deployed and the current underlying value; the external vault itself is `adapter.metaMorphoVault()`

Each series carries its own exchange rate. The current allocation release supports one funded strategy and rejects another funded strategy or competing series with outstanding shares; registration alone does not enable simultaneous multi-strategy allocation. See [stQEURO Token](quantillon-protocols-tokens/stqeuro-token.md) for the exchange-rate mechanics.

***

### 💰 Yield Flow

1. **Deploy** - the vault-operator role (an operational keeper wallet designated by governance) moves idle USDC into an external vault via `QuantillonVault.deployUsdcToVault(vaultId, amount)`.
2. **Accrue** - the external vault (e.g. MetaMorpho) generates money-market yield on that USDC.
3. **Harvest & distribute** - `QuantillonVault.harvestAndDistributeVaultYield(vaultId)` allocates the staked fraction of harvested USDC to stakers and the unstaked fraction to treasury. Only a configured haircut on gross staker yield is paid to the hedger.
4. **Credit and vest** - the staker allocation is converted to QEURO and credited to the series. It raises the redeemable exchange rate as yield vests, with no rebasing or claim transaction.

See [Yield Distribution](yield-distribution.md) for the exact snapshot formula, schedule, current haircut, vesting and underlying APY interpretation. The Morpho harvest bypasses YieldShift's separate accounting pools.

***

### 🛡️ Governance & Risk Controls

* Registering or deactivating a vault/adapter (`setStakingVault`) is **governance-gated** (2-of-3 Safe; core-contract upgrades additionally route through a 12h timelock). Deploying USDC is done by the vault-operator role and harvesting by the yield-distributor role - two narrow operational roles on `QuantillonVault` held by keeper wallets that governance can revoke at any time. USDC is withdrawn from the external vault automatically when a redemption needs it; there is no separate emergency-withdrawal function.
* Adapters are deliberately thin pass-throughs - no fixed exposure or rebalance constants live in the adapter; exposure sizing is an operational governance decision per `vaultId`.
* External-vault risk (smart-contract risk of Morpho/Aave, underlying market risk) is isolated per vault for **yield**: each stQEURO series bears only its own vault's performance. Since `QuantillonVault` 1.1.11 the collateral accounting is loss-aware - the protocol collateralization ratio reflects the external vault's current value, so a loss in an external vault lowers the global ratio.
* Onboarding follows a governance runbook. The current single-funded-strategy guard must be respected before any funding; registration does not bypass it.

***

### 🔗 Related Pages

* [stQEURO Token](quantillon-protocols-tokens/stqeuro-token.md) - exchange-rate mechanics and staking UX
* [Core Mechanisms](mechanisms.md) - protocol-wide mint/redeem and collateral flows
* [Yield Distribution](yield-distribution.md) - current Morpho allocation
* [YieldShift](yield-shift.md) - separate authorized-source accounting
