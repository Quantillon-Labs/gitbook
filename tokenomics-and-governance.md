# Tokenomics & Governance

### The $QTI Token: Governance, Distribution, and Incentives

#### 5.1 Three-Token Ecosystem Architecture

Quantillon's first deployment operates through a three-token system designed to create sustainable value flows and optimal capital efficiency for the EUR market, while keeping governance at the broader protocol layer.

**The $QTI Token: Governance & Value Accrual**

The $QTI token serves as the governance backbone of the Quantillon Protocol, featuring advanced vote-escrow (veQTI) mechanics and progressive decentralization. **QTI is currently dormant**: the contract is deployed with a 100,000,000 QTI supply cap, but no tokens have been minted (supply is 0) and governance functions are inactive until a future activation upgrade. The distribution below is the planned strategy, not an on-chain state:

**Strategic Token Allocation:**

| Category                  | Allocation | Amount         | Lock Period | Vesting Schedule           |
| ------------------------- | ---------- | -------------- | ----------- | -------------------------- |
| **Community & Ecosystem** | 40%        | 40,000,000 QTI | Variable    | 48-month algorithmic curve |
| **Treasury & Liquidity**  | 30%        | 30,000,000 QTI | Immediate   | Governance-controlled      |
| **Team & Advisors**       | 20%        | 20,000,000 QTI | 12 months   | 36 months linear           |
| **Investors (SAFT/BSA)**  | 10%        | 10,000,000 QTI | 6 months    | 24-36 months tiered        |

**Vote-Escrow (veQTI) System**

QTI holders can lock their tokens for periods ranging from 7 days to 365 days (1 year), receiving voting power multipliers up to 4x base weight. This system ensures long-term alignment and prevents governance attacks while enabling meaningful decentralized decision-making.

**Governance parameters (as coded, inactive while QTI is dormant):**

* **Proposal threshold:** 100,000 QTI
* **Quorum:** 1,000,000 QTI
* **Voting period:** 3 days minimum, 14 days maximum
* **Execution delay:** 2 days

A multi-layer proposal model (higher thresholds for constitutional changes than for operational decisions) is **aspirational - subject to governance design, not implemented in the deployed contracts**. Until QTI activation, the protocol is governed by a 2-of-3 Gnosis Safe with a 12-hour upgrade timelock (see [Quantillon DAO](quantillon-dao.md)).

**stQEURO: Yield-Bearing Euro Infrastructure**

stQEURO represents the first deployment's yield-bearing wrapper, automatically compounding returns from QEURO collateral deployment. Unlike traditional staking mechanisms, stQEURO maintains constant token quantity in user wallets while increasing intrinsic value over time through the formula:

**stQEURO Value = 1 stQEURO = (1 + Cumulative Yield Rate) QEURO**

Key benefits include:

* **Automatic Compounding:** No manual reinvestment required
* **Redeemable shares:** No staking-duration lock in the direct ERC-4626 flow; newly credited yield vests, and exits remain subject to contract checks
* **DeFi Composability:** Full integration across protocols while earning yield
* **Tax Efficiency:** No rebase events creating potential taxable income

### Yield Mechanics

QuantillonVault 1.5.0 splits Morpho harvests by the staked fraction of total QEURO at harvest. Stakers receive their allocation after any configured hedger haircut; the unstaked allocation goes to treasury. The legacy per-series staking yield fee is ignored on this path. YieldShift is a separate authorized-source ledger and does not adjust this split.

This is snapshot-weighted allocation. Credited QEURO becomes redeemable as it vests through the share price. See [Yield Distribution](protocol/yield-distribution.md) for the current settings, formula and APY interpretation.

### Incentive Alignment and Protocol Sustainability

Once activated, $QTI is intended to serve as an incentive layer through **liquidity mining programs**, staking multipliers, and governance rewards - time-bound incentives designed to bootstrap adoption without creating long-term inflationary pressures. Today the only incentive program is **Quantillon Rewards**, an off-chain points program for QEURO depositors and stakers whose terms are published ([Rewards Program Terms](complementary-information/rewards-program-terms.md)); the program is not yet open in the application, and it promises no token or allocation.

In the longer term, the protocol aims to activate the **Fee Switch**, diverting a portion of transaction and yield fees to a treasury governed by $QTI holders. This treasury may be used to:

* 🔍 Fund audits and research
* 🛡️ Provide insurance buffers
* 🌉 Invest in ecosystem integrations or cross-chain bridges

Sustainability is further supported by the protocol's lean cost structure. No revenue or burn-rate projections are published: fees are currently 0, and whether the protocol reaches an operating surplus depends on fee levels governance has not set and on volumes that do not exist yet. Any future surplus could be reinvested in growth or redistributed through mechanisms decided by governance.

Governance mechanisms are engineered to be progressive. In the current phase, protocol changes require multi-signature validation by a 2-of-3 Gnosis Safe, with core-contract upgrades gated by a 12-hour timelock, to ensure operational security. Over time, power will transition toward full DAO control, contingent on metrics like TVL, QTI token dispersion, and governance participation rates.

> **Quantillon's tokenomics combine strong economic incentives with a governance architecture inspired by proven DeFi protocols such as Curve, Aave, and MakerDAO. QEURO and stQEURO instantiate the model for the EUR market first, while QTI remains the governance layer for protocol risk, incentives, and future deployment policy.**
