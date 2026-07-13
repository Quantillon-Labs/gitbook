# Oracle Architecture

## EUR/USD Oracle Architecture

### 📋 Overview

QEURO mint and redeem are priced off the **EUR/USD market price used to execute the protocol's hedge** — the EUR/USD perpetual mid of the active hedge venue (currently the Hyperliquid `EUR` perp) — rather than a generic spot-FX feed. This keeps the on-chain valuation of QEURO aligned with the venue where the EUR/USD exposure is actually neutralized, eliminating the basis between the QEURO liability and its hedge.

Pricing is served through an **`OracleRouter`** that can switch between two sources:

| Slot | Source | Status | What it provides |
|------|--------|--------|------------------|
| 1 | **HyperliquidEurUsdOracle** | **Active** | The Hyperliquid `EUR` market mid (hedge-aligned) |
| 0 | **ChainlinkOracle** | Fallback | Chainlink EUR/USD spot + USDC/USD validation |

The router exposes a single, oracle-agnostic `IOracle` interface to the rest of the protocol, so `QuantillonVault` and other consumers are unaware of which source is active. A one-transaction governance call (`switchOracle`) flips between them.

Slot 1 is the **market** slot: it hosts the oracle of the active hedge venue — currently Hyperliquid, with a sibling `LighterEurUsdOracle` implemented as a switchable alternative (deployment pending the staged rollout) (see the *Venue-Switchable Market Oracle* section below).

***

### 🏗️ Architecture & Data Flow

```
┌──────────────────────────────────────────────────────────────────────┐
│                        EUR/USD PRICE PATH                              │
├──────────────────────────────────────────────────────────────────────┤
│  Hyperliquid market (EUR perpetual mid)                                │
│        │  read off-chain                                               │
│        ▼                                                                │
│  Price Publisher (off-chain service)                                   │
│        │  publishes the mid on-chain (cadence / price-move triggered)  │
│        ▼                                                                │
│  SlippageStorage  ──►  HyperliquidEurUsdOracle                          │
│                          • freshness, price bounds, circuit breaker     │
│                          • USDC/USD delegated to ChainlinkOracle        │
│        ┌─────────────────────────┘                                      │
│        ▼                                                                │
│  OracleRouter (active = Hyperliquid)   ◄── ChainlinkOracle (fallback)   │
│        │  IOracle.getEurUsdPrice()                                      │
│        ▼                                                                │
│  QuantillonVault  (QEURO mint / redeem pricing)                         │
│        +                                                                │
│  Independent watchdog  →  freezes mint/redeem if the price is stale,    │
│                           circuit-broken, or diverges from Chainlink    │
└──────────────────────────────────────────────────────────────────────┘
```

***

### 🔁 Why Hedge-Aligned Pricing

The protocol neutralizes the EUR/USD leg by holding a hedge on a perpetual venue — currently Hyperliquid. If QEURO were minted/redeemed at a *spot* EUR/USD price while the hedge fills at the *venue* price, the small-but-persistent gap (the basis) would leak value and desynchronize the liability from its hedge. Reading the venue mid on-chain removes that gap by construction. Chainlink spot is kept as a safety reference and fallback, not as the primary valuation source.

***

### 🧩 The OracleRouter

The router holds two oracle slots and routes all reads to the currently active one.

```solidity
enum OracleType { CHAINLINK, MARKET }  // slot 0, slot 1 (MARKET was named STORK before router v1.1.0)
function getEurUsdPrice() external returns (uint256 price, bool isValid);
function switchOracle(OracleType newOracle) external;        // ORACLE_MANAGER_ROLE (governance)
function updateOracleAddresses(address chainlink, address slot1) external;
```

- Reads delegate to the active oracle via the generic `IOracle` interface.
- Slot 1 (the market slot) hosts the active venue's oracle — currently the Hyperliquid oracle; switching is a single governance transaction.
- `activeOracle()` and `getOracleAddresses()` expose the current configuration on-chain.

***

### 💧 The Price Publisher (off-chain → on-chain)

A dedicated off-chain service reads the EUR/USD mids of the supported perp venues and publishes them on-chain into `SlippageStorage`, signed by a dedicated publisher wallet that holds **only** a write role (no protocol funds). Every tick it publishes **both** the Hyperliquid and the Lighter mid, each keyed by source (`SOURCE_LIGHTER = 0`, `SOURCE_HYPERLIQUID = 1`) and guarded by a price-integrity check cross-referenced against the other venue's mid.

It publishes when **any** of these fire:
- **Cadence** — at least every N seconds.
- **Price move** — the mid moves more than a small basis-point threshold (keeps the on-chain price fresh during EUR/USD moves).
- **Liquidity/slippage change** — book conditions shift materially.

An on-chain minimum-interval rate limit bounds write frequency, so the published mid stays fresh without unnecessary gas.

> This is a self-operated price source. Its **availability** depends on the publisher running and funded; a failure is **fail-safe** (mint/redeem freezes, or governance falls back to Chainlink) — it never results in a wrong price being used.

***

### 🔐 The HyperliquidEurUsdOracle

Reads the published mid and applies the same valuation-grade safety the protocol's other oracles use, then exposes the standard `IOracle` interface.

| Protection | Behaviour |
|------------|-----------|
| **Staleness** | Reject a published mid older than the freshness window → returns `isValid = false` |
| **Price bounds** | EUR/USD must be within `[minEurUsdPrice, maxEurUsdPrice]` (default 0.80–1.40) |
| **Deviation circuit breaker** | Reject a jump > 5% vs the last valid price; fall back to last valid |
| **Last-valid fallback** | On any rejection, returns the last valid price with `isValid = false` |
| **USDC/USD** | Delegated to the ChainlinkOracle (a USDC feed issue cannot block an EUR/USD read) |

```solidity
function getEurUsdPrice() external returns (uint256 price, bool isValid);
// stale / out-of-bounds / circuit-broken / paused → (lastValidPrice, false)
// a valid read advances the baseline
```

A `false` validity flag is treated by the vault as a **hard stop** (mint/redeem revert) — the protocol never values QEURO at a stale or out-of-band price.

***

### 🔀 Venue-Switchable Market Oracle

The market side of the router is **venue-switchable**. The protocol's EUR/USD hedging and market-mid pricing support two perp venues:

| Venue | Status | Oracle contract |
|-------|--------|-----------------|
| **Hyperliquid** | **Active** | `HyperliquidEurUsdOracle` (in slot 1 today) |
| **Lighter** ([zkLighter](https://docs.lighter.xyz)) | Supported, not active | `LighterEurUsdOracle` (implemented, not yet deployed) |

- **Both mids are already on-chain.** The publisher writes **both venues'** EUR/USD mids to `SlippageStorage` on every tick, keyed by source (`SOURCE_LIGHTER = 0`, `SOURCE_HYPERLIQUID = 1`), and each tick is guarded by a price-integrity check cross-referenced against the other venue's mid.
- **Sibling oracle, same validation.** `LighterEurUsdOracle` applies the same audited validation logic as the Hyperliquid oracle — 15-minute staleness window, 0.80–1.40 price bounds, 5% deviation circuit breaker, USDC/USD delegated to the ChainlinkOracle — reading the Lighter mid instead.
- **One-transaction switch.** Changing venue is a single governance (2-of-3 Safe) transaction swapping the router's market-oracle address in slot 1. Chainlink remains the manual fallback in slot 0, unchanged.
- **Coupled by design.** The venue where the hedge executes and the venue whose mid prices mint/redeem always move together. The hedging engine refuses to run — and the protocol watchdog freezes mint+redeem — if the execution venue ever differs from the oracle venue (fatal `venue_oracle_mismatch` guard). This eliminates silent cross-venue basis risk.
- **Staged rollout.** Any venue switch follows an operational runbook: testnet certification, live shadow observation, on-chain oracle A/B, then a coupled cutover in a maintenance window with a pre-staged rollback transaction.

Hyperliquid remains the active venue today; Lighter is fully supported alternative infrastructure, not yet active.

***

### ⏱️ Freshness & Staleness

- The publisher keeps the on-chain mid fresh via its cadence plus a price-move trigger; an on-chain rate limit caps write frequency.
- The oracle rejects anything older than its freshness window, which **freezes mint/redeem** rather than ever using a stale price.
- Effective freshness tightens automatically when the market is moving (price-move trigger) and relaxes when it is calm (cadence).

***

### 🛡️ Independent Watchdog (defence-in-depth)

An independent, separately-hosted watchdog continuously polls a read-only health verdict from the hedging engine and can **freeze mint+redeem** (pause the vault) when it detects a problem. Beyond the engine's own liveness checks, the verdict now includes an **EUR/USD oracle cross-check**:

- **Stale / circuit-broken** active oracle → unhealthy → freeze.
- **Basis blow-out** — the active oracle diverges from the Chainlink reference by more than a configured band (e.g. 100 bps) → unhealthy → freeze.

The watchdog is reason-agnostic: any unhealthy verdict triggers a pause, so an oracle dislocation is contained automatically while it is investigated. See [Liquidation Mode](liquidation-mode.md) and the protocol's risk controls for the broader safety hierarchy.

***

### 🔙 Fallback & Recovery

| Action | Effect |
|--------|--------|
| `switchOracle(0)` (governance) | Instantly revert pricing to ChainlinkOracle (spot EUR/USD) |
| `triggerCircuitBreaker()` (emergency) | Force the last-valid price on the active oracle |
| `resetCircuitBreaker()` (emergency) | Re-seed after investigation |
| Vault pause (watchdog/emergency) | Freeze mint+redeem |

Because the ChainlinkOracle remains wired in slot 0, the protocol can always fall back to a third-party spot feed with a single governance transaction.

***

### 🔑 Roles & Governance

The 2-of-3 governance Safe holds **all** administrative roles on each oracle (configuration, bounds, circuit breaker, upgrades) and operates them directly. The router and the deploying key hold no oracle roles. The publisher wallet holds only a narrow write role on `SlippageStorage` and never custodies protocol funds.

***

### 📍 Deployed Contracts (Base Mainnet)

| Contract | Address |
|----------|---------|
| OracleRouter | `0x7ED6aaEd83Db69509A88CAe5C247ef8fA44056E0` |
| HyperliquidEurUsdOracle | `0x0B58aBB57775E0fCEDfd4460e00dD9D9610C2C43` |
| ChainlinkOracle (fallback) | `0xaEE3c9c298051ef7242882AbCaE2Fd12d29443E7` |
| SlippageStorage | `0x0fde0ff2566be3c24af6d654012dddb4f1da099b` |

All contracts are verified on Basescan.

***

### 📐 Example Scenarios

**Normal operation**
```
Hyperliquid EUR mid: 1.1347 → published on-chain (fresh)
Oracle: within bounds ✅, fresh ✅, deviation < 5% ✅
Router (active = Hyperliquid) → QuantillonVault prices QEURO at 1.1347
```

**Publisher interruption**
```
No fresh publish for > freshness window
Oracle → isValid = false → mint/redeem revert (fail-safe)
Recovery: publisher restored, or governance switchOracle(0) → Chainlink
```

**Basis dislocation**
```
Hyperliquid mid diverges > 100 bps from Chainlink spot
Watchdog verdict → unhealthy → vault paused (mint+redeem frozen)
Recovery: investigate; resume or switchOracle(0)
```

***

> **Summary**: QEURO is priced off the active hedge venue's EUR/USD mid (currently Hyperliquid; Lighter is a supported alternative behind a coupled venue switch) via a dual-source `OracleRouter`, with Chainlink retained as a one-transaction fallback. An off-chain publisher streams the mid on-chain; the on-chain oracle enforces freshness, bounds, and a circuit breaker; and an independent watchdog freezes mint/redeem on staleness, circuit-break, or a basis dislocation versus Chainlink. Every failure mode is fail-safe — the protocol freezes or falls back, never values QEURO at a wrong price.
