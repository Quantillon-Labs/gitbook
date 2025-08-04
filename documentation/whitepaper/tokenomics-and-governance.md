# Tokenomics & Governance

### The $QTI Token: Governance, Distribution, and Incentives

The **$QTI token** serves as the governance backbone of the Quantillon Protocol. Its primary role is to coordinate decisions regarding protocol upgrades, vault whitelisting, fee calibration, and treasury allocation. In contrast to speculative utility tokens with ambiguous roles, $QTI is explicitly designed for long-term protocol stewardship.

The total supply of $QTI is capped, with the initial distribution structured as follows:

**📊 Token Distribution**

* **50%** allocated to the community via liquidity mining, governance rewards, and protocol usage
* **15%** reserved for the core development team
* **35%** allocated to early backers, strategic partners, advisors, liquidity and DAO.

Both the team and backer allocations are subject to a **multiple year linear vesting schedule** with a 12-month cliff. This ensures that early stakeholders are aligned with the long-term health of the protocol.

Governance is executed through a token-weighted voting system using on-chain proposals and Snapshot voting. Over time, governance responsibilities may include activation of the Fee Switch mechanism, treasury investment strategies, and expansion of collateral options.

### Yield Mechanics and the "Yield Shift"

Quantillon introduces an innovative mechanism called the **Yield Shift**, which serves as the protocol's internal rebalancing engine. It functions by redistributing yield between Users and Hedgers based on market conditions, supply/demand imbalances, and peg deviation pressures.

Collateral deployed in DeFi protocols (e.g., Aave) generates a baseline APY. From this yield:

1. A protocol fee of **10%** is applied.
2. Hedgers receive a fixed compensation based on the EUR/USD interest rate spread (typically \~1%).
3. The residual is then distributed variably:
   * **Positive Yield Shift**: More yield incentivizes Hedgers when their supply is insufficient.
   * **Negative Yield Shift**: More yield flows to Users when hedger participation is high.

This creates a dynamic equilibrium. The Yield Shift is not discretionary; it is governed by predefined formulas based on real-time FX rate spreads and User/Hedger supply and demand. Governance can only modify its parameters within capped ranges, preserving systemic integrity.

### Incentive Alignment and Protocol Sustainability

$QTI also serves as an incentive layer through **liquidity mining programs**, staking multipliers, and governance rewards. These incentives are time-bound and designed to bootstrap early adoption without creating long-term inflationary pressures.

In the longer term, the protocol aims to activate the **Fee Switch**, diverting a portion of transaction and yield fees to a treasury governed by $QTI holders. This treasury may be used to:

* 🔍 Fund audits and research
* 🛡️ Provide insurance buffers
* 🌉 Invest in ecosystem integrations or cross-chain bridges

Sustainability is further ensured by the protocol's lean cost structure. With an estimated burn rate of **€400,000 per year** and projected revenues of **€500,000 at 20M TVL**, Quantillon achieves operating surplus early on. This surplus can be reinvested in growth or redistributed through token buybacks or veQTI-style locking mechanisms.

Governance mechanisms are engineered to be progressive. In early phases, protocol changes may require multi-signature validation from the Quantillon Foundation to ensure operational security. Over time, power will transition toward full DAO control, contingent on metrics like TVL, QTI token dispersion, and governance participation rates.

> **Quantillon's tokenomics combine strong economic incentives with a governance architecture inspired by proven DeFi protocols such as Curve, Aave, and MakerDAO, while adapting them to the specific needs of eurozone compliance and FX stability.**
