---
description: The Guardian to take rapid decisions in a crisis
---

# Quantillon Guardians

## 🔎 TL;DR <a href="#tl-dr" id="tl-dr"></a>

* Like in any protocol, it may be necessary to rapidly pause some functionalities, or to change some parameters or references in the protocol, in order to react rapidly to unforeseen events.
* On Base — the only chain where the protocol is deployed — this guardian capability maps to the contracts' `EMERGENCY_ROLE` (and `PAUSER_ROLE` on QEURO). It is held by the protocol's **2-of-3 Gnosis Safe**; in addition, an **independent watchdog service** holds a pause-only `EMERGENCY_ROLE` on QuantillonVault and freezes mint/redeem automatically when the hedge or the oracle is unhealthy.

## 💡 Need for a Guardian <a href="#need-for-a-guardian" id="need-for-a-guardian"></a>

There is no on-chain "guardian" role: the guardian capability is the contracts' `EMERGENCY_ROLE` (plus `PAUSER_ROLE` on the QEURO token). Today it is exercised by the **2-of-3 Gnosis Safe** (`0x1d7fF432a93d0085Fb69474c7E567f859829e6cd`) controlled by Quantillon Labs team members, which also holds the protocol's other privileged roles — see [Quantillon DAO](quantillon-dao.md).

In addition, an **independent, separately hosted watchdog service** holds a pause-only `EMERGENCY_ROLE` on QuantillonVault: it freezes mint and redeem within seconds if the hedging engine or the EUR/USD oracle is unhealthy, and lifts only pauses it set itself once the situation is healthy again. The Safe can pause or unpause at any time and can revoke the watchdog's role at any time — see [Oracle Architecture](protocol/oracle-architecture.md#independent-watchdog-defence-in-depth).

If a guardian capability with specific rights is needed, it is because some situations in the protocol are time sensitive. If a flaw is found in one of the contracts where an attacker can systematically make a profit, the necessary parts of the contract must be pausable immediately, without waiting for a governance process or an upgrade timelock.

Note that governance can revoke guardian rights from any address at any time.

## 🔘 Responsibilities <a href="#responsibilities" id="responsibilities"></a>

In clear terms, the guardian capability enables to:

* Pause and unpause contract functionalities (full pause, minting killswitch)
* Trigger and reset oracle circuit breakers
* Close a hedger position in an emergency (`emergencyClosePosition`) — a last resort: closing a position that still backs outstanding QEURO removes that backing and lowers the protocol collateralization ratio

Time-sensitive parameter changes — mint and redeem fees, collateralization thresholds, hedging parameters — are not guardian powers: they belong to the governance role, which the same Safe also holds and exercises without a timelock.

Quantillon smart contracts have been designed to limit a guardian's ability to impact the protocol in a way that is harmful to the system: core-contract upgrades always go through the 12-hour timelock, and emergency actions are reversible operations (pause/unpause) rather than value transfers.
