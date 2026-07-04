# Tokenomics & Governance

### The $QTI Token: Governance, Distribution, and Incentives

#### 5.1 Three-Token Ecosystem Architecture

Quantillon's first deployment operates through a three-token system designed to create sustainable value flows and optimal capital efficiency for the EUR market, while keeping governance at the broader protocol layer.

**The $QTI Token: Governance & Value Accrual**

The $QTI token serves as the governance backbone of the Quantillon Protocol, featuring advanced vote-escrow (veQTI) mechanics and progressive decentralization. **QTI is currently dormant**: the contract is deployed with a 100,000,000 QTI supply cap, but no tokens have been minted (supply is 0) and governance functions are inactive until a future activation upgrade. The distribution below is the planned strategy, not an on-chain state:

**Strategic Token Allocation:**

| Category                  | Allocation | Amount         | Lock Period | Vesting Schedule            |
| ------------------------- | ---------- | -------------- | ----------- | --------------------------- |
| **Community & Ecosystem** | 50%        | 50,000,000 QTI | Variable    | 48-month algorithmic curve  |
| **Team & Founders**       | 15%        | 15,000,000 QTI | 12 months   | 36 months linear            |
| **Investors (SAFT/BSA)**  | 10%        | 10,000,000 QTI | 6 months    | 24-36 months tiered         |
| **DAO Treasury**          | 13%        | 13,000,000 QTI | Immediate   | Governance-controlled       |
| **Strategic Partners**    | 5%         | 5,000,000 QTI  | 6 months    | 18 months performance-based |
| **Advisors**              | 2%         | 2,000,000 QTI  | 6 months    | 18 months milestone-driven  |
| **Liquidity Provision**   | 5%         | 5,000,000 QTI  | Immediate   | Market-responsive release   |

**Vote-Escrow (veQTI) System**

QTI holders can lock their tokens for periods ranging from 7 days to 365 days (1 year), receiving voting power multipliers up to 4x base weight. This system ensures long-term alignment and prevents governance attacks while enabling meaningful decentralized decision-making.

**Governance parameters (as coded, inactive while QTI is dormant):**

* **Proposal threshold:** 100,000 QTI
* **Quorum:** 1,000,000 QTI
* **Voting period:** 3 days minimum, 14 days maximum
* **Execution delay:** 2 days

A multi-layer proposal model (higher thresholds for constitutional changes than for operational decisions) is **aspirational — subject to governance design, not implemented in the deployed contracts**. Until QTI activation, the protocol is governed by a 2-of-3 Gnosis Safe with a 12-hour upgrade timelock (see [Quantillon DAO](quantillon-dao.md)).

**stQEURO: Yield-Bearing Euro Infrastructure**

stQEURO represents the first deployment's yield-bearing wrapper, automatically compounding returns from QEURO collateral deployment. Unlike traditional staking mechanisms, stQEURO maintains constant token quantity in user wallets while increasing intrinsic value over time through the formula:

**stQEURO Value = 1 stQEURO = (1 + Cumulative Yield Rate) QEURO**

Key benefits include:

* **Automatic Compounding:** No manual reinvestment required
* **Instant Liquidity:** No lock periods or withdrawal delays
* **DeFi Composability:** Full integration across protocols while earning yield
* **Tax Efficiency:** No rebase events creating potential taxable income

### Yield Mechanics and the "Yield Shift"

Quantillon introduces an innovative mechanism called the **Yield Shift**, which serves as the protocol's internal rebalancing engine. It functions by redistributing yield between Users and Hedgers based on market conditions, supply/demand imbalances, and peg deviation pressures.

Collateral deployed in external staking vaults (currently Morpho USDC lending on Base) generates a baseline APY. When that yield is harvested:

1. **Hedger funding is paid first** — a governance-set annual rate, capped at 50% of each harvest, compensating the EUR/USD hedge.
2. **The residual is split between stQEURO stakers and the treasury** according to the staked share of QEURO.
3. On the yield-pool allocation layer, the Yield Shift adjusts the user/hedger split (base 50%, up to 90% to users):
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

Governance mechanisms are engineered to be progressive. In the current phase, protocol changes require multi-signature validation by a 2-of-3 Gnosis Safe, with core-contract upgrades gated by a 12-hour timelock, to ensure operational security. Over time, power will transition toward full DAO control, contingent on metrics like TVL, QTI token dispersion, and governance participation rates.

> **Quantillon's tokenomics combine strong economic incentives with a governance architecture inspired by proven DeFi protocols such as Curve, Aave, and MakerDAO. QEURO and stQEURO instantiate the model for the EUR market first, while QTI remains the governance layer for protocol risk, incentives, and future deployment policy.**
