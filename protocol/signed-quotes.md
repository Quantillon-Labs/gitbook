# Signed Quotes

Transaction-specific signed quotes are **active on Base mainnet since 17 October 2026**. The [confirmed governance execution](https://basescan.org/tx/0xd2945ee2bc1357ec6503e1a374cca9adf841168d4c78739ff6a78a0088aa06f7) activated the migration, and the application reopened at 10:08 UTC. Contract versions and signed bindings were verified at finalized Base block **52,727,379**. See [Deployment Status](deployment-status.md) and [Smart Contract Components](smart-contract-components.md).

## What changes for a user

1. Choose the amount and complete any required token approval.
2. Request an executable quote for that wallet, amount and operation.
3. Review the exact output and expiry, then submit the authorized transaction on Base.
4. Check the transaction receipt before retrying. A network timeout does not prove failure.

A quote lasts **at most 45 seconds from its original market observation**. Wallet interaction and transaction inclusion use that same window. Changing the amount or route requires a new quote; an expired quote cannot execute. A refreshed quote can have a different output and requires another review.

A price preview is informational. Executable quotes bind the wallet, chain, target, exact call, amounts, expiry and nonce. The app submits them through the signed wrapper on QuantillonVault or UserPool. Signing does not bypass current collateralization, reference, capacity, token or pause checks. Even before expiry, a transaction can revert.

## Three separate responsibilities

| Responsibility | Purpose |
| --- | --- |
| Execution quote | Defines the exact output for one operation, using directional book depth, size, fees and an execution buffer |
| Reference valuation | Values liabilities, collateral and hedger accounting, checked against an independent reference |
| Hedge capacity | Limits new exposure against reconciled coverage and available margin |

A large trade's execution spread must not revalue every outstanding QEURO. Quotes issued concurrently share a risk budget. Reservations remain until canonical settlement, expiry or invalidation is established; they do not reserve Hyperliquid's public liquidity.

## Publication and readiness

The periodic market publisher is retired. Signed reference and capacity reports travel with user or operational transactions. This removes the idle publisher's recurring gas expense; settlement, governance and other onchain operations still cost gas.

The watchdog checks whether fresh signed reports can be delivered and accepted, together with hedge health. Read-only simulations can test that path without publishing a new persistent observation. An old cache during inactivity is therefore different from unavailable fresh data.

## Unavailability and emergency redemption

Normal signed settlement requires valid signing services, fresh observations, sufficient admitted capacity and the other protocol checks. A service outage or failed guard can block both minting and normal redemption.

The separate emergency-redemption mechanism has a **non-renewing budget of 0 QEURO** in production and is unavailable with that configuration. It is not an automatic fallback when quotes fail. Enabling it would require governed mode and budget changes, valid independent reference data and the remaining contract and liquidity checks. Switching mode alone does not replenish a budget. [Protocol liquidation mode](liquidation-mode.md), which changes the backing-based payout formula, is a different mechanism.

## Execution risk and scope

Minting on Base and hedging on Hyperliquid remain separate operations. A successful mint is not proof of a completed hedge. Market movement, partial fills, outages and failed hedges remain possible; admission limits and reconciliation reduce risk without eliminating it.

The migration covers mint/redemption, UserPool conversion, mint-and-stake, harvested-yield conversion, hedger reference operations and the related monitoring consumers. It preserves the [yield allocation policy](yield-distribution.md). The technical [production integration guide](https://smartcontracts.quantillon.money/Signed-Quote-Migration.html) describes contract entrypoints and signature bindings.
