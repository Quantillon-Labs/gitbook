# Execution Pricing

## A reference price is not an executable quote

The oracle supplies the EUR/USD reference used for valuation and safety checks. **ExecutionPricing** separately quotes normal mint and redemption using published Hyperliquid order-book depth and admitted hedge capacity. A mint uses the buy side (asks); a redemption uses the sell side (bids). Conservative reference-price bounds and a buffer also affect the amount received.

Consequently, a mint followed immediately by a redemption can return less USDC even when the configured mint and redemption fees are zero. Fees, execution spreads and network transaction costs are different costs.

## Capacity and settlement

The publisher submits observations on-chain. The pricing module checks freshness, depth, permitted price impact, source compatibility and outstanding exposure. New minting requires compatible market pricing and sufficient admitted hedge capacity. A newly published book does not erase unhedged exposure: the authorized reporter acknowledges completed hedges separately.

The dapp obtains a quote and passes minimum-output protections to the vault. A preview is not a reservation; a transaction can revert when the quote or capacity changes before execution. Integrations should use the current contract interfaces and repeat previews before submitting.

## Degraded redemption

When executable depth or capacity is unavailable, stale, or incompatible with the active router source, the module can quote a **degraded redemption** from a valid reference price with a configured haircut. For a fresh observation the haircut is at least the maximum permitted book impact, preventing a better fallback quote simply by exhausting depth.

This is not an unconditional exit guarantee. Invalid reference prices, a fresh book that diverges excessively from the reference, vault/token pauses, unavailable USDC and the user's minimum output can still block redemption. Protocol liquidation mode uses its separate proportional-backing formula; see [Liquidation Mode](liquidation-mode.md).

## Accounting and controls

Execution spread is tracked separately from user backing and protocol fees. The reserve is excluded from collateralization and external-vault deployable principal. ExecutionPricing is a **non-proxy contract** with access-controlled configuration and reporting; replacing it requires updating the vault's configured module through the applicable governance controls.

See [Oracle Architecture](oracle-architecture.md) for valuation checks, [Production Deployment Status](deployment-status.md) for the dated production state, and [Smart Contract Components](smart-contract-components.md) for the deployed address.
