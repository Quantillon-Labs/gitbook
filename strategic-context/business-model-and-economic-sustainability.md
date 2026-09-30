# Business Model & Economic Sustainability

Quantillon's current Base deployment combines QEURO issuance, stQEURO staking and a designated hedge operator. The Safe governs the protocol today; QTI is deployed but unissued and token governance is dormant.

## Current economic flows

* **Mint and redemption fees:** configurable, currently zero. Execution spreads and buffers still affect the quote; a zero fee does not make a round trip costless.
* **External-vault yield:** the harvest-time staked QEURO fraction determines gross staker allocation. The unstaked allocation goes to treasury. Any configured hedger haircut comes from gross staker yield; it is currently zero. See [Yield Distribution](../protocol/yield-distribution.md).
* **Hedger position fees:** entry, exit and margin fees are currently zero. The separate vault and HedgerPool fee-routing settings determine what portion of collected fees reaches the hedger reward reserve. They do not impose a 20% charge on reward claims.
* **FeeCollector:** distributes the fees it receives between treasury, development and community. It does not apply its ratios to the full Morpho harvest.
* **Hedge economics:** position interest, reward-reserve funding, venue funding, execution costs and hedging losses affect the operator. Hedger collateral does not earn a base share of Morpho yield.

For the dated settings and the distinction between the two fee-routing parameters, see [Production Deployment Status](../protocol/deployment-status.md).

## Sustainability and costs

Strategy returns, service operation, gas, hedge execution and available capital all affect sustainability. YieldShift is a separate authorized-source ledger; it does not dynamically rebalance the current Morpho harvest. Neither the hedge nor external strategy eliminates loss or liquidity risk.

No revenue projections are published. Future integrations, institutional services and cross-chain features are possibilities, not deployed revenue streams or established partnerships. The current deployment does not establish a cross-chain bridge-fee business.

## Directional targets

These are internal adoption targets, not forecasts or current TVL:

| Target period | TVL target |
| --- | --- |
| Q4 2026 public launch | $1M |
| Q2 2027 | $10M |
| Q1 2028 | $10M to $100M |

Expansion remains subject to security readiness, hedge capacity, liquidity and governance decisions. See [Roadmap](../roadmap-and-adoption-strategy.md).
