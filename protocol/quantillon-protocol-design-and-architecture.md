# Quantillon Protocol: Design & Architecture

Quantillon should be understood as a **protocol pattern** before it is understood as a specific asset. The architecture is designed to transform deep USD-based DeFi liquidity into a user-facing **local-currency market** through protocol-native FX hedging.

## The Protocol Pattern

At the highest level, Quantillon has four layers:

| Layer | Role | Why It Exists |
| --- | --- | --- |
| **USD Liquidity Base** | Uses deep USD-denominated liquidity and yield sources | The deepest DeFi markets are still mostly dollar-based |
| **FX Hedging Layer** | Offsets the currency leg between USD and the target market | Users should not be forced to hold unwanted USD exposure |
| **Local-Currency Asset Layer** | Delivers market-specific exposure to end users | The product should match the user's target currency |
| **Governance + Wrappers** | Coordinates incentives, parameters, wrappers, and expansion | The architecture must scale beyond one deployment |

This is the core reason Quantillon is protocol-first. The asset visible to the user changes by market, while the economic logic underneath remains reusable.

## Why Start From USD Liquidity

Quantillon does not try to rebuild the deepest DeFi markets from zero. It starts from the reality that:

* USD liquidity is deeper than most other on-chain currency markets
* much of DeFi's best collateral and yield sits on top of that liquidity
* the real problem for non-dollar users is not liquidity access alone, but **liquidity access without forced USD exposure**

The protocol therefore treats USD liquidity as infrastructure, not as the desired end state.

## The FX Hedging Layer

The defining feature of Quantillon is that the FX leg is handled inside the protocol architecture.

This matters because the underlying problem is not "how do I buy a non-dollar stablecoin?" It is:

> how do I access DeFi liquidity and yield while still ending up exposed to the currency I actually care about?

Quantillon answers that with a hedging layer that sits between the USD base and the user-facing local-currency asset.

## The Local-Currency Asset Layer

The output of the system is a market-specific asset that tracks the target currency exposure of the deployment.

That means Quantillon can support a family of deployments that share one architecture but serve different target markets. The local-currency asset is therefore a **deployment layer**, not the full identity of the protocol.

## Governance Above the Deployment Layer

QTI governs the protocol above any one market. It exists to coordinate:

* deployment policy
* incentives
* risk parameters
* long-term protocol evolution

This separation is important because a protocol that can expand beyond one currency should not bind governance to a single deployment narrative.

## QEURO as the First Deployment

QEURO is the first production implementation of the Quantillon architecture.

It demonstrates the model on EUR first:

* USD liquidity and yield serve as the base layer
* the hedging layer neutralizes the EUR/USD leg
* the user-facing asset delivers EUR exposure
* wrappers such as stQEURO extend the first deployment into yield-bearing formats

The token documentation and smart contract component pages under this section should therefore be read as **implementation-specific docs for the first deployment**, not as the whole scope of Quantillon.

## Why EUR First

The euro is a strong first deployment market because it combines:

* meaningful non-dollar demand
* a recognizable DeFi access problem
* deep FX infrastructure for the hedge side

That makes QEURO a credible proof point. It does not mean the protocol is locked to EUR forever.

## Reusable Expansion Model

The same architecture can be reused for future deployments when the economics are compelling. Illustrative examples include CHF, JPY, or GBP, but deployment decisions should be driven by governance and market conditions rather than brand ambition.

In that sense, Quantillon’s architecture is meant to be:

* **modular**
* **reusable**
* **observable**
* **governable across multiple markets**

> **Read the rest of the documentation with this framing in mind: Quantillon is the protocol architecture, and QEURO is the first market built on top of it.**
