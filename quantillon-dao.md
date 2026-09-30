---
description: How Quantillon Protocol is governed today, and how QTI governance will work
cover: .gitbook/assets/banner.png
coverY: 0
---

# Quantillon DAO

## 🔎 TL;DR <a href="#tl-dr" id="tl-dr"></a>

* Quantillon Protocol is deployed on **Base mainnet (chain 8453)** - public launch planned for Q4 2026 - and is currently governed by a **2-of-3 Gnosis Safe**, with core-contract upgrades routed through a **12-hour OpenZeppelin TimelockController**.
* A full **QTI vote-escrow (veQTI) governance system is implemented in the deployed contracts but dormant**: QTI supply is 0, no mint path is wired, and lock/vote/propose functions are inactive until a future activation upgrade.
* The path is one of **progressive decentralization**: Safe + timelock today, community governance through veQTI once the protocol has matured and QTI is activated.

## 🏛 Governance today: Safe + Timelock <a href="#governance-today" id="governance-today"></a>

### Key contracts

| Contract | Address (Base) | Role |
| -------- | -------------- | ---- |
| **Governance Safe** (2-of-3 Gnosis Safe) | `0x1d7fF432a93d0085Fb69474c7E567f859829e6cd` | Retains operational governance/emergency roles and peripheral administration; core admin roles belong to the Timelock |
| **Timelock** (OZ TimelockController, 12h delay) | `0x7Ade8f3Bf1FdaF0785efE9Ea5C6339D1aD6B8342` | Holds core DEFAULT_ADMIN_ROLE and gates core upgrades while secure upgrades are enabled |
| **QuantillonVault** | `0x833E5Ba510a241b21F1C60c987D1c49eB52E4a07` | Core mint/redeem and yield-distribution contract under this governance |

Narrow operational roles are delegated to dedicated wallets, revocable by their respective admin authority, through the timelock for core roles: an **independent hedging watchdog** holds `EMERGENCY_ROLE` on QuantillonVault (pause/unpause only), **keeper wallets** designated by governance hold `VAULT_OPERATOR_ROLE` (deploy USDC to the external vault) and `YIELD_DISTRIBUTOR_ROLE` (harvest and distribute yield), and the **oracle publisher** holds `WRITER_ROLE` on `SlippageStorage`. The single hedger of the HedgerPool is Quantillon Labs' hedging engine (a designated address, not a role). The full role inventory is on [Smart Contract Components](protocol/smart-contract-components.md#2-roles-and-permissions).

### What each layer gates

**Through the 12-hour timelock:**

* Upgrades of the core UUPS proxy contracts - **QuantillonVault, QEUROToken, QTIToken, UserPool, HedgerPool, YieldShift, stQEUROFactory and the stQEURO series**. With secure upgrades enabled, a new implementation must be queued and wait for the delay. This gives observers time to review; it does not guarantee an exit if redemptions are paused or otherwise unavailable.
* Core DEFAULT_ADMIN_ROLE actions, including role grants/revocations, supply-cap and rate-limit administration, use the controller.

**Directly by the 2-of-3 Safe (no timelock):**

* Upgrades of the peripheral contracts - **FeeCollector, OracleRouter, ChainlinkOracle, HyperliquidEurUsdOracle and SlippageStorage** - which are plain UUPS proxies upgraded directly by the Safe
* Operational parameters: fee settings (mint/redeem and hedger fees), staking-yield haircut and recipient, vesting parameters (the legacy stQEURO yield fee is ignored by vault 1.5.0), collateralization thresholds (minting floor, currently 102.5%), hedging parameters, interest rates
* Oracle operations: switching the active EUR/USD source in the OracleRouter (Hyperliquid market oracle ↔ Chainlink fallback), price bounds and staleness, circuit-breaker management
* Emergency actions: pause/unpause, minting killswitch, emergency position closure
* Peripheral role management and operations authorized by the Safe's existing roles; core default-admin actions require the controller

Core and peripheral permissions differ. SecureUpgradeable also implements an emergency-disable procedure with a 24-hour delay and two distinct admin approvals; this is separate from ordinary pause/unpause. Read each contract's role and secure-upgrade configuration before preparing an operation.

### Scope

Quantillon is deployed on **Base only**. There is no Ethereum-mainnet deployment, no cross-chain governance bridging, and no off-chain voting space at this stage.

## 🗳 Future governance: QTI vote-escrow (dormant) <a href="#future-governance" id="future-governance"></a>

The deployed contracts already contain a complete on-chain governance system built around the [QTI token](protocol/quantillon-protocols-tokens/qti-token.md). It is **inactive today** - QTI supply is 0 and there is no mint path - and will be switched on by a future activation upgrade.

### veQTI voting power

* **Lock to vote**: QTI holders lock tokens for a chosen duration to receive veQTI voting power
* **Lock duration**: 7 days minimum, 365 days (1 year) maximum
* **Multiplier**: voting power scales with lock length, up to **4x** at the maximum lock
* **Decay**: veQTI decays gradually as the unlock date approaches

### Proposal lifecycle (as coded)

| Parameter | Value |
| --------- | ----- |
| **Proposal threshold** | 100,000 QTI |
| **Quorum** | 1,000,000 QTI |
| **Voting period** | 3 days minimum, 14 days maximum |
| **Execution delay** | 2 days after a successful vote |

A proposer holding at least the threshold submits an on-chain proposal; veQTI holders vote during the voting period; if quorum and majority are reached, the proposal becomes executable after the 2-day delay. A proposal can be cancelled by its proposer or by the governance role.

### What activation requires

* An upgrade wiring a QTI mint/distribution path (per the [planned distribution strategy](protocol/quantillon-protocols-tokens/qti-token.md#token-distribution-architecture))
* Governance configuration handover: pointing protocol roles at the QTI governance/execution path instead of (or alongside) the Safe

## 🛤 Progressive decentralization <a href="#progressive-decentralization" id="progressive-decentralization"></a>

Quantillon follows a deliberate, staged handover of control:

1. **Bootstrap (now)** - a small, accountable signer set (2-of-3 Safe) operates the protocol with a 12h upgrade timelock as the public safety window. This favors fast incident response while the protocol earns operational track record on mainnet.
2. **Activation** - QTI is minted and distributed, veQTI locking goes live, and on-chain proposals begin governing an expanding set of parameters.
3. **Community control** - privileged roles migrate from the Safe to the governance execution path; the Safe's remit narrows toward emergency response (pause) before being phased down as the system proves itself.

The guiding principle: **decentralize authority no faster than the community's demonstrated capacity to exercise it safely** - and never present dormant machinery as live governance.

## 🔐 Security <a href="#security" id="security"></a>

* The Timelock is the standard OpenZeppelin `TimelockController`; the Safe is a standard Gnosis Safe - both are extensively audited, battle-tested building blocks.
* The QTI vote-escrow and proposal contracts are part of the same reviewed protocol codebase (Internal AI-assisted review; no audit-firm review to date) and follow the same UUPS upgrade discipline as the rest of the system.
* Emergency controls (pause, killswitch, circuit breakers) are documented in the [Smart Contract Components](protocol/smart-contract-components.md).

## 🤖 Automated safety actors <a href="#automated-safety-actors" id="automated-safety-actors"></a>

Since July 2026 an **independent, separately hosted watchdog** can pause QuantillonVault (freezing mint and redeem) on its own when the hedging engine or the EUR/USD oracle is unhealthy - a stale or circuit-broken price, a basis dislocation versus Chainlink, or a hedge that stops responding. It holds a pause-only `EMERGENCY_ROLE` on the vault, lifts only pauses it set itself once the verdict is healthy again, and can be revoked through the core admin timelock. Keeper wallets likewise execute routine operations (deploying USDC to the external vault, harvesting yield) under narrow roles. These narrow service roles do not authorize upgrades or general parameter changes. Harvesting does execute the configured payments to treasury and any haircut recipient. See [Quantillon Guardians](quantillon-guardians.md) and [Oracle Architecture](protocol/oracle-architecture.md#independent-watchdog-defence-in-depth).
