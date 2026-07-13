---
description: How Quantillon Protocol is governed today, and how QTI governance will work
cover: .gitbook/assets/banner.png
coverY: 0
---

# Quantillon DAO

## 🔎 TL;DR <a href="#tl-dr" id="tl-dr"></a>

* Quantillon Protocol is live on **Base mainnet (chain 8453)** and is currently governed by a **2-of-3 Gnosis Safe**, with core-contract upgrades routed through a **12-hour OpenZeppelin TimelockController**.
* A full **QTI vote-escrow (veQTI) governance system is implemented in the deployed contracts but dormant**: QTI supply is 0, no mint path is wired, and lock/vote/propose functions are inactive until a future activation upgrade.
* The path is one of **progressive decentralization**: Safe + timelock today, community governance through veQTI once the protocol has matured and QTI is activated.

## 🏛 Governance today: Safe + Timelock <a href="#governance-today" id="governance-today"></a>

### Key contracts

| Contract | Address (Base) | Role |
| -------- | -------------- | ---- |
| **Governance Safe** (2-of-3 Gnosis Safe) | `0x1d7fF432a93d0085Fb69474c7E567f859829e6cd` | Holds all privileged roles across the protocol |
| **Timelock** (OZ TimelockController, 12h delay) | `0x7Ade8f3Bf1FdaF0785efE9Ea5C6339D1aD6B8342` | Gates upgrades of the core contracts |
| **QuantillonVault** | `0x833E5Ba510a241b21F1C60c987D1c49eB52E4a07` | Core mint/redeem and yield-distribution contract under this governance |

### What each layer gates

**Through the 12-hour timelock:**

* Upgrades of the core UUPS proxy contracts (QuantillonVault, token contracts, pools). Any new implementation must be queued in the Timelock and can only be executed after the 12-hour delay — giving all stakeholders a transparent window to review the change and, if they disagree with it, exit the protocol before it takes effect.

**Directly by the 2-of-3 Safe (no timelock):**

* Operational parameters: fee settings (mint/redeem, stQEURO yield fee, hedger fees), collateralization thresholds, hedging parameters, interest rates
* Oracle operations: switching the active EUR/USD source in the OracleRouter (market venue ↔ Chainlink fallback), swapping the market venue oracle itself (Hyperliquid ↔ Lighter, always coupled with the hedge-execution venue), price bounds, circuit-breaker management
* Emergency actions: pause/unpause, minting killswitch, emergency position closure
* Role management and treasury operations

This split is deliberate: irreversible structural changes (upgrades) always carry a delay, while time-sensitive operational and emergency actions can be executed as fast as 2 of the 3 signers can act.

### Scope

Quantillon is deployed on **Base only**. There is no Ethereum-mainnet deployment, no cross-chain governance bridging, and no off-chain voting space at this stage.

## 🗳 Future governance: QTI vote-escrow (dormant) <a href="#future-governance" id="future-governance"></a>

The deployed contracts already contain a complete on-chain governance system built around the [QTI token](protocol/quantillon-protocols-tokens/qti-token.md). It is **inactive today** — QTI supply is 0 and there is no mint path — and will be switched on by a future activation upgrade.

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

A proposer holding at least the threshold submits an on-chain proposal; veQTI holders vote during the voting period; if quorum and majority are reached, the proposal becomes executable after the 2-day delay. Failed or malicious proposals can be cancelled by the emergency role.

### What activation requires

* An upgrade wiring a QTI mint/distribution path (per the [planned distribution strategy](protocol/quantillon-protocols-tokens/qti-token.md#token-distribution-architecture))
* Governance configuration handover: pointing protocol roles at the QTI governance/execution path instead of (or alongside) the Safe

## 🛤 Progressive decentralization <a href="#progressive-decentralization" id="progressive-decentralization"></a>

Quantillon follows a deliberate, staged handover of control:

1. **Bootstrap (now)** — a small, accountable signer set (2-of-3 Safe) operates the protocol with a 12h upgrade timelock as the public safety window. This favors fast incident response while the protocol earns operational track record on mainnet.
2. **Activation** — QTI is minted and distributed, veQTI locking goes live, and on-chain proposals begin governing an expanding set of parameters.
3. **Community control** — privileged roles migrate from the Safe to the governance execution path; the Safe's remit narrows toward emergency response (pause, proposal cancellation) before being phased down as the system proves itself.

The guiding principle: **decentralize authority no faster than the community's demonstrated capacity to exercise it safely** — and never present dormant machinery as live governance.

## 🔐 Security <a href="#security" id="security"></a>

* The Timelock is the standard OpenZeppelin `TimelockController`; the Safe is a standard Gnosis Safe — both are extensively audited, battle-tested building blocks.
* The QTI vote-escrow and proposal contracts are part of the audited protocol codebase and follow the same UUPS upgrade discipline as the rest of the system.
* Emergency controls (pause, killswitch, circuit breakers) are documented in the [Smart Contract Components](protocol/smart-contract-components.md).
