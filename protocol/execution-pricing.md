# Execution Pricing

## A reference price is not an executable quote

Since 17 October 2026, **SignedExecutionPricing 1.1.0** is the active pricing module. The execution service walks the Hyperliquid order book for the requested amount: asks for minting, bids for redemption. It includes protocol fees and an execution buffer, subject to reference-price bounds. SignedReferenceSource supplies the separate valuation reference.

The resulting quote binds an exact output to a wallet, chain, target, inner call, nonce, epoch and expiry. Zero protocol fees do not mean zero execution cost: execution spread and network gas remain. A mint followed by redemption can return less USDC.

## Capacity and settlement

Executable quotes carry independently authorized reference and hedge-capacity reports. Capacity is based on reconciled hedge observations and cumulative acknowledgments. A fresh report does not erase unhedged exposure. Unresolved buys and sells count toward gross admission rather than cancelling each other merely because net exposure is zero.

The app completes approval before requesting the executable quote, then submits the authorized call through `executeWithQuote`. The maximum lifetime is **45 seconds from the market observation**. A preview does not reserve capacity; issued executable quotes share a bounded risk budget, but cannot reserve public venue liquidity or guarantee execution. Current collateralization, reference, liquidity, capacity, token and pause checks still apply.

If the amount, route or deployment changes, obtain and review a new quote. An expired authorization cannot be reused. Resolve pending transaction receipts before retrying after an ambiguous network response.

## Emergency redemption and liquidation

The legacy pricing module's degraded-redemption fallback is not the normal signed path. The current signed module is in **NORMAL** mode with a **0 QEURO emergency budget**. Quote unavailability does not automatically enable a reference-only redemption.

Emergency redemption would require explicit governed configuration, a non-renewing budget, valid independent reference data and the remaining vault/liquidity checks. It is separate from [liquidation mode](liquidation-mode.md), which changes the payout formula according to remaining backing. Neither mechanism guarantees an unconditional exit.

## Accounting and controls

Execution spread is tracked separately from user backing and protocol fees. The reserve is excluded from collateralization and external-vault deployable principal. SignedExecutionPricing and SignedReferenceSource are non-proxy modules; replacing them requires the applicable governed bindings and reconciliation of outstanding settlements.

Base settlement and subsequent Hyperliquid hedging remain non-atomic. A successful Base transaction is not proof that the venue hedge has filled.

See [Signed Quotes](signed-quotes.md), [Oracle Architecture](oracle-architecture.md), [Deployment Status](deployment-status.md) and [Smart Contract Components](smart-contract-components.md).
