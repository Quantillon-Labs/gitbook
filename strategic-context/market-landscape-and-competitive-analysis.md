# Market Landscape & Competitive Analysis

Quantillon's thesis is that users seeking euro exposure may also want access to USD-based DeFi liquidity. Holding USDC directly leaves EUR/USD exposure with the holder; QEURO instead uses a protocol-operated hedge and brings its own contract, collateral, liquidity and operational risks.

## Comparing designs

| Design | Exposure and dependencies to examine |
| --- | --- |
| Fiat-reserve euro token | Issuer, reserves, redemption eligibility, custody and liquidity |
| Crypto-collateralized euro token | Collateral quality, liquidation rules, oracle design and governance |
| FX-hedged USD collateral, Quantillon's current design | USDC, the external yield vault, hedge venue, margin capacity, publisher, execution costs and privileged roles |

These are design categories, not a ranking of competitors' current liquidity, returns or regulatory status. Such comparisons need dated, product-specific evidence.

## QEURO's current implementation

QEURO is deployed on Base and targets euro exposure using USDC backing and a single designated Hyperliquid hedger. stQEURO holders participate in the staked allocation of realized external-vault yield; holding QEURO alone earns none. The allocation is snapshot-based and the displayed provider APY does not guarantee a holder's return. See [Yield Distribution](../protocol/yield-distribution.md).

Normal mint and redemption use execution quotes, including spreads and buffers. Liquidity in the wider USDC or FX markets does not mean unlimited executable liquidity or immediate redemption inside Quantillon. See [Execution Pricing](../protocol/execution-pricing.md).

## Directional adoption targets

| Period | TVL target, not a forecast or current figure |
| --- | --- |
| Q4 2026 public launch | $1M |
| Q2 2027 | $10M |
| Q1 2028 | $10M to $100M |

No current market-share, addressable-market valuation or guaranteed adoption claim is made here. Expansion depends on the constraints in the [Roadmap](../roadmap-and-adoption-strategy.md) and the [risk disclosures](../risk-management-and-sustainability/risks-and-mitigation-strategies.md).
