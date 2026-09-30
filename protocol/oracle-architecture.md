# Oracle Architecture

## Valuation and execution

OracleRouter provides the EUR/USD reference for QEURO valuation. Slot **1**, the active MARKET slot, reads HyperliquidEurUsdOracle, which consumes the published `xyz:EUR` perpetual mid from SlippageStorage. Slot **0** reads ChainlinkOracle and is a manual governance fallback. ChainlinkOracle also validates USDC/USD.

Normal user execution additionally uses directional order-book quotes from [ExecutionPricing](execution-pricing.md). Using the hedge venue's reference reduces valuation mismatch; it does not eliminate spreads, funding costs, slippage, timing differences or hedge failure risk.

## Data flow

```text
Hyperliquid mid -> operated publisher -> SlippageStorage -> market oracle
Chainlink EUR/USD -------------------------------> independent reference check
market oracle / Chainlink fallback -> OracleRouter -> vault valuation
Hyperliquid depth + hedge admission -> ExecutionPricing -> mint/redeem quote
```

The publisher is an operated service with a restricted writing role. Availability depends on that service and its transaction funding. On-chain checks limit accepted data; they do not prove every accepted observation is economically correct.

## Market-oracle protections

The market oracle checks publication age, EUR/USD bounds, deviation from its last valid price and an **independent Chainlink EUR/USD reference on-chain**. At the [30 September snapshot](deployment-status.md), the market observation limit was 900 seconds (hard maximum one hour), bounds were 0.80 to 1.40 USD/EUR and the consecutive-price deviation limit was 5%.

The independent-reference divergence limit was 2%, with a 3% off-hours band. Its configured age ceiling was 8,100 seconds; the Chainlink reader's own 7,200-second EUR/USD freshness limit also applies. The off-hours band does **not** waive reference freshness: a stale reference can stop valid pricing on weekends or holidays while Hyperliquid continues trading.

Rejected reads return a last-valid value with `isValid = false`; consumers must respect that flag. A cached numerical price is not permission to transact. USDC/USD validation is delegated to ChainlinkOracle. See [ChainlinkOracle](chainlink-oracle.md) for the sequencer guard, feed checks and delayed development-mode controls.

## The market slot

The router interface permits selecting its sources with `switchOracle` and updating their addresses with the appropriate roles. Source selection alone does not provide compatible execution depth or hedge capacity.

A Lighter oracle was deployed during an alternative-venue evaluation; that option was retired on 1 September 2026. Hyperliquid is the sole supported hedge venue.

## Independent watchdog (defence-in-depth)

A separately hosted watchdog observes hedging and oracle health and can pause QuantillonVault. This freezes mint and redeem. Its operating policy is to lift only pauses it created after health recovers. The on-chain EMERGENCY_ROLE itself permits both pause and unpause; self-owned recovery is a service policy, not a separate contract permission.

The Safe retains emergency pause/unpause authority. Granting or revoking a core vault role uses the TimelockController, which holds the core admin role; it is not an immediate direct Safe action. See [Guardians](../quantillon-guardians.md).

## Fallback and recovery

| Action | Effect and limits |
| --- | --- |
| Switch router to Chainlink | Changes valuation source; requires a valid Chainlink feed and does not alone restore minting |
| Trigger oracle circuit breaker | Marks pricing invalid; the last-valid number is not an executable fallback |
| Reset circuit breaker | Resets the breaker subject to contract validation; investigate the cause first |
| Pause vault | Stops guarded operations, including mint and redeem |
| Restore publisher and hedge health | Allows fresh observations and capacity; other guards and pause state still apply |

Degraded redemption may be available under [ExecutionPricing's rules](execution-pricing.md). Neither fallback nor liquidation mode guarantees redemption under all conditions.

## Governance and deployed contracts

The Safe holds the peripheral oracle administration and upgrade permissions; these upgrades do not use the core 12-hour timelock. OracleRouter also has delegated manager/emergency permissions on the market oracle for its forwarding functions. SlippageStorage writes require WRITER_ROLE.

For verified addresses and versions, use [Smart Contract Components](smart-contract-components.md); for the core/peripheral distinction, use [Quantillon DAO](../quantillon-dao.md).
