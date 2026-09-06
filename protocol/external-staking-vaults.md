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

Because all adapters share this interface, the protocol can onboard, migrate, or retire external vaults without changing core contracts — governance simply registers the adapter under a `vaultId` and the runtime routes by that id.

Available adapter implementations:

* **`MetaMorphoStakingVaultAdapter`** — wraps a MetaMorpho (Morpho) vault. **This is the adapter currently live in production.**
* **`MorphoStakingVaultAdapter`** — Morpho markets adapter (symmetric pattern).
* **`AaveStakingVaultAdapter`** — Aave-style adapter (symmetric pattern), used with mock vaults for local development and available should governance onboard an Aave market; no such onboarding is scheduled.

***

### 🚀 Live Deployment (Base Mainnet)

| Component | Value |
| --- | --- |
| Active external vault | MetaMorpho USDC vault (`0xBEEFE94c8aD530842bfE7d8B397938fFc1cb83b2`) |
| Adapter | `MetaMorphoStakingVaultAdapter` — `0xb2f253Cd74ebfa16894339438B467396De9e8EA3` |
| `vaultId` | `2` |
| stQEURO series | `stQEUROMORPHO1` — `0x17CD8ed967d17072297CcAe3D379C9e86aeBEb1d` |

> The vaultId-2 adapter was migrated from a previous address (`0x103aEBD0059AAA3DcCaa9ab0cCb901382Bd48978`) to the current one on 2026-07-01. Migrations like this are possible precisely because of the adapter indirection — stakers' positions and the stQEURO series are unaffected.

***

### 🏭 One stQEURO Series per Vault

`stQEUROFactory` deploys a dedicated stQEURO proxy for every registered external vault and keeps the registry:

* `getStQEUROByVaultId(vaultId)` → the stQEURO token for that vault
* `getVaultById(vaultId)` → the `QuantillonVault` that registered the series (not the external vault)
* `getVaultName(vaultId)` → human-readable label (e.g. `MORPHO1`)
* `QuantillonVault.getVaultExposure(vaultId)` → the adapter address, its active flag, the principal deployed and the current underlying value; the external vault itself is `adapter.metaMorphoVault()`

Each series carries its own exchange rate, so the yield performance of one external vault never dilutes stakers of another. See [stQEURO Token](quantillon-protocols-tokens/stqeuro-token.md) for the exchange-rate mechanics.

***

### 💰 Yield Flow

1. **Deploy** — the vault-operator role (an operational keeper wallet designated by governance) moves idle USDC into an external vault via `QuantillonVault.deployUsdcToVault(vaultId, amount)`.
2. **Accrue** — the external vault (e.g. MetaMorpho) generates money-market yield on that USDC.
3. **Harvest & distribute** — `QuantillonVault.harvestAndDistributeVaultYield(vaultId)` realizes the yield and splits it:
   * a **hedger funding share** first (absolute, time-prorated, at a governance-set annual rate capped at 50% — **currently 0 bps**, no carve-out is taken today),
   * the **residual** is split between the vault's stQEURO stakers and the protocol treasury in proportion to the staked share of circulating QEURO.
4. **Compound** — the staker share raises the stQEURO series' exchange rate; no rebasing, no claim transaction needed.

An off-chain keeper holding the vault's `YIELD_DISTRIBUTOR_ROLE` triggers harvests on a schedule; the distribution math is fully on-chain. See [YieldShift](yield-shift.md) for the user/hedger yield-allocation layer.

***

### 🛡️ Governance & Risk Controls

* Registering or deactivating a vault/adapter (`setStakingVault`) is **governance-gated** (2-of-3 Safe; core-contract upgrades additionally route through a 12h timelock). Deploying USDC is done by the vault-operator role and harvesting by the yield-distributor role — two narrow operational roles on `QuantillonVault` held by keeper wallets that governance can revoke at any time. USDC is withdrawn from the external vault automatically when a redemption needs it; there is no separate emergency-withdrawal function.
* Adapters are deliberately thin pass-throughs — no fixed exposure or rebalance constants live in the adapter; exposure sizing is an operational governance decision per `vaultId`.
* External-vault risk (smart-contract risk of Morpho/Aave, underlying market risk) is isolated per vault for **yield**: each stQEURO series bears only its own vault's performance. Since `QuantillonVault` 1.1.11 the collateral accounting is loss-aware — the protocol collateralization ratio reflects the external vault's current value, so a loss in an external vault lowers the global ratio.
* Onboarding a new external vault follows a runbook: deploy the adapter, register it with the factory (new `vaultId` + stQEURO series), then progressively fund it.

***

### 🔗 Related Pages

* [stQEURO Token](quantillon-protocols-tokens/stqeuro-token.md) — exchange-rate mechanics and staking UX
* [Core Mechanisms](mechanisms.md) — protocol-wide mint/redeem and collateral flows
* [YieldShift](yield-shift.md) — dynamic yield split between user and hedger pools
