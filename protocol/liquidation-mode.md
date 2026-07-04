# Liquidation Mode

## Liquidation Mode: Protocol Solvency Protection

### 📋 Overview

Quantillon protects protocol solvency with a **protocol-level liquidation mode** built directly into `QuantillonVault`. Unlike traditional DeFi liquidations — where keepers liquidate individual positions for a bonus — Quantillon uses a **pro-rata redemption model** that fairly distributes any collateral shortfall among all QEURO holders who choose to exit.

> **There is no standalone `LiquidationSystem` contract and no liquidator role.** Liquidation mode is a state of the protocol as a whole, evaluated inside `QuantillonVault` from the global collateralization ratio.

***

### 🎯 Collateralization Thresholds

```solidity
uint256 public constant MIN_COLLATERALIZATION_RATIO_FOR_MINTING = 105e18; // 105%
uint256 public constant CRITICAL_COLLATERALIZATION_RATIO_BPS   = 10100;   // 101%
```

| Protocol State | Collateralization Ratio | Effect |
|----------------|------------------------|--------|
| **Healthy** | > 105% | Normal minting and redemption |
| **Restricted** | 101% – 105% | Minting blocked (CR below the 105% minting floor); redemption normal |
| **Liquidation mode** | ≤ 101% | Pro-rata liquidation redemption activated |

Both thresholds are governance-configurable within hard floors: the minting ratio can never be set below 101%, and the critical ratio never below 100%.

***

### ⚙️ How Liquidation Mode Works

When the protocol collateralization ratio falls to or below the critical ratio (101%), `QuantillonVault` switches redemptions to **pro-rata mode**:

```
usdcPayout = (qeuroAmount / totalQEUROSupply) × totalVaultUSDC
```

* Every redeeming holder receives their **proportional share of the remaining collateral**, at the actual backing ratio rather than the oracle EUR/USD price.
* No holder can front-run others to exit at par and leave the shortfall to the rest — losses (or residual premium) are shared proportionally.
* The standard `redemptionFee` still applies to liquidation-mode redemptions (**currently 0**; governance-settable up to 5%).
* Redemptions emit `LiquidationRedeemed(user, qeuroAmount, usdcPayout, collateralizationRatioBps, isPremium)` for transparency.

#### Effect on hedgers

In liquidation mode, hedger positions' **effective margin is treated as zero** — the hedgers, who are short EUR against the users, absorb losses first through their margin before holders share any residual shortfall. See [HedgerPool](hedger-pool.md).

#### Live state checks

```solidity
vault.getProtocolCollateralizationRatio(); // current global CR (non-view: refreshes oracle cache)
vault.canMint();                           // false when CR < 105%
vault.shouldTriggerLiquidationLive();      // true when CR <= criticalCollateralizationRatio
```

***

### 🔄 Recovery

Liquidation mode is not terminal. The protocol exits it automatically as soon as the collateralization ratio rises back above the critical threshold — through EUR/USD price movement, hedger margin top-ups, or new collateral. Minting resumes once CR is back above the 105% minting floor.

***

### 🧭 Design Rationale

1. **Fairness** — pro-rata sharing removes the race-to-exit dynamic of first-come-first-served redemptions under stress.
2. **Simplicity** — no keeper infrastructure, auction mechanics, or liquidation bonuses; the vault itself enforces the payout math.
3. **Solvency-first** — the 105% minting floor keeps new issuance from diluting collateral quality, while the 101% critical band gives hedgers room to recapitalize before holders are impacted.
4. **Continuous operation** — users can always redeem, in any protocol state; only the payout formula changes.

***

### 🔗 Related Pages

* [Core Mechanisms](mechanisms.md) — mint/redeem flows and collateral accounting
* [HedgerPool](hedger-pool.md) — hedger margin rules and the single-hedger model
* [Oracle Architecture](oracle-architecture.md) — the EUR/USD price feeding the collateralization ratio
