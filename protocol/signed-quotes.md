# Signed quotes: migration in development

Quantillon is developing transaction-specific signed quotes. This page describes the proposed migration and candidate implementation. It does not announce production activation. The deployed architecture remains documented in [Deployment Status](deployment-status.md), [Oracle Architecture](oracle-architecture.md) and [Execution Pricing](execution-pricing.md).

## What changes for a user

The application first completes any required token approval. It then requests an executable quote for the chosen amount and wallet. The user reviews the exact output and expiry before submitting it on Base. A changed amount requires a new quote. An expired quote cannot execute. Before expiry, execution still depends on current collateralization, capacity, reference-price bounds and pause controls.

The quote incorporates the relevant side of the Hyperliquid order book, trade size, protocol fees and an execution buffer. This builds on existing directional depth pricing; size-aware execution is already part of the current protocol.

## What stays separate

| Responsibility | Purpose |
| --- | --- |
| Execution quote | Defines the amount received for one operation |
| Reference valuation | Values liabilities, collateral and hedger accounting |
| Hedge capacity | Limits newly admitted exposure against reconciled coverage and available margin |

A large trade's execution spread must not change the valuation of every outstanding QEURO. Signed quotes remain subject to protocol accounting, collateralization, independent-reference checks and onchain capacity limits.

Quotes issued at the same time share a risk budget. The system must retain reservations until canonical settlement, expiry or invalidation is established. These reservations do not reserve Hyperliquid's public liquidity.

## Costs and execution risk

Once all consumers migrate, Quantillon can stop publishing an unchanged stream of periodic market snapshots while there is no activity. Required data can travel with user or operational transactions. This removes the idle publisher's recurring gas expense, but does not make settlement, governance or emergency operations free.

Minting on Base and hedging on Hyperliquid remain separate operations. A successful mint is not proof of a completed hedge. Market movement, partial fills, outages and failed hedges remain possible. Buffers, exposure caps, reconciliation and emergency procedures reduce risk without eliminating it.

Signed quotes also depend on operated signing services. If valid quotes cannot be produced, normal signed minting is unavailable. Emergency redemption requires its own governed mode and valid reference data; there is no unconditional redemption guarantee.

## Scope of the migration

The change covers minting, redemption, optional UserPool accounting, mint-and-stake routes, harvested-yield conversion, hedger margin operations and liquidation. Rewards accounting, protocol dashboards and the independent watchdog also need to distinguish stale cached observations from live execution readiness.

Deployment proceeds through development and preproduction before governed production activation. Documentation and contract deployment records will be updated to the verified state at activation. Historical records remain dated records of the earlier architecture.

An intent or multi-hedger RFQ model is a possible later evolution. Neither is required to remove idle periodic publication, and neither provides automatic atomicity across Base and Hyperliquid.
