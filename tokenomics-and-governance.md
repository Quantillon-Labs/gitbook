# Tokenomics & Governance

### The $QTI Token: Governance, Distribution, and Incentives

#### 5.1 Three-Token Ecosystem Architecture

Quantillon operates through a sophisticated three-token system designed to create sustainable value flows and optimal capital efficiency across the euro-DeFi ecosystem.

**The $QTI Token: Governance & Value Accrual**

The $QTI token serves as the governance backbone of the Quantillon Protocol, featuring advanced vote-escrow (veQTI) mechanics and progressive decentralization. With a fixed total supply of 100,000,000 QTI tokens, the distribution is strategically structured as follows:

**Strategic Token Allocation:**

| Category                  | Allocation | Amount         | Lock Period | Vesting Schedule            |
| ------------------------- | ---------- | -------------- | ----------- | --------------------------- |
| **Community & Ecosystem** | 50%        | 50,000,000 QTI | Variable    | 48-month algorithmic curve  |
| **Team & Founders**       | 15%        | 15,000,000 QTI | 12 months   | 36 months linear            |
| **Investors (SAFT/BSA)**  | 13%        | 13,000,000 QTI | 6-18 months | 24-36 months tiered         |
| **DAO Treasury**          | 10%        | 10,000,000 QTI | Immediate   | Governance-controlled       |
| **Strategic Partners**    | 5%         | 5,000,000 QTI  | 6 months    | 18 months performance-based |
| **Advisors**              | 2%         | 2,000,000 QTI  | 6 months    | 18 months milestone-driven  |
| **Liquidity Provision**   | 5%         | 5,000,000 QTI  | Immediate   | Market-responsive release   |

**Vote-Escrow (veQTI) System**

QTI holders can lock their tokens for periods ranging from 1 week to 4 years, receiving voting power multipliers up to 4x base weight. This system ensures long-term alignment and prevents governance attacks while enabling meaningful decentralized decision-making.

**Governance operates through three layers:**

* **Constitutional Changes:** 85% threshold, 14-day timelock (protocol parameters, emergency procedures)
* **Operational Decisions:** 60% threshold, 3-day timelock (fee structures, incentive programs)
* **Community Proposals:** Simple majority, 24-hour timelock (grants, marketing initiatives)

**stQEURO: Yield-Bearing Euro Infrastructure**

stQEURO represents the protocol's yield-bearing token, automatically compounding returns from QEURO collateral deployment. Unlike traditional staking mechanisms, stQEURO maintains constant token quantity in user wallets while increasing intrinsic value over time through the formula:

**stQEURO Value = 1 stQEURO = (1 + Cumulative Yield Rate) QEURO**

Key benefits include:

* **Automatic Compounding:** No manual reinvestment required
* **Instant Liquidity:** No lock periods or withdrawal delays
* **DeFi Composability:** Full integration across protocols while earning yield
* **Tax Efficiency:** No rebase events creating potential taxable income

### Yield Mechanics and the "Yield Shift"

Quantillon introduces an innovative mechanism called the **Yield Shift**, which serves as the protocol's internal rebalancing engine. It functions by redistributing yield between Users and Hedgers based on market conditions, supply/demand imbalances, and peg deviation pressures.

Collateral deployed in DeFi protocols (e.g., Aave) generates a baseline APY. From this yield:

1. A protocol fee of **10%** is applied.
2. Hedgers receive a fixed compensation based on the EUR/USD interest rate spread (typically \~1%).
3. The residual is then distributed variably:
   * **Positive Yield Shift**: More yield incentivizes Hedgers when their supply is insufficient.
   * **Negative Yield Shift**: More yield flows to Users when hedgers participation is high.

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
