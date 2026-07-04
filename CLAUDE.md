# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

Documentation-only repository for the Quantillon Protocol. No application code, no build step, no tests. All content is Markdown rendered by GitBook at [docs.quantillon.money](https://docs.quantillon.money).

## Navigation Is Defined in SUMMARY.md

`SUMMARY.md` is the single source of truth for GitBook's left-hand navigation. When adding a new page, it **must** be listed in `SUMMARY.md` or it will not appear in the published docs. The hierarchy in `SUMMARY.md` (indentation = nesting) controls the sidebar structure.

## CI/CD

Pushing to `main` or `dev` triggers `.github/workflows/Telegram Notifications.yml`, which sends a commit notification to a Telegram channel via two GitHub secrets: `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHANNEL_ID`.

## Content Conventions

- All files are GitBook-compatible Markdown (no MDX, no JSX).
- Use relative paths for internal links (e.g., `../protocol/mechanisms.md`).
- Assets go in `.gitbook/assets/`.
- Commit messages follow conventional commits (`docs:`, `fix:`, `feat:`).

## Protocol Domain Knowledge

When editing documentation, the following are established facts about the protocol
(as of 2026-07-04, verified against live Base mainnet state via cast; the smart-contracts repo
is the source of truth for on-chain behavior):

**Three-token ecosystem:**
- **QEURO** — Euro-pegged stablecoin, USDC-collateralized. **No fixed tokenomic supply cap** —
  supply is bounded by hedging capacity: minting requires ≥105% protocol collateralization;
  liquidation/critical mode engages at 101% (pro-rata redemption inside QuantillonVault).
  Token-level safety guardrails (not supply policy): administrative supply ceiling currently
  100M (governance-raisable) and a mint/burn rate limit of 10M QEURO per 300-block window
  (~10 min on Base). Mint/redeem fees are currently 0 (governance-settable, max 5%).
- **stQEURO** — Yield-bearing staked QEURO (ERC-4626 vault, one series per external vault via
  `stQEUROFactory`); exchange rate rises as vault yield is distributed — no rebasing.
- **QTI** — Governance token with vote-escrow mechanics (veQTI), 100M supply cap.
  **Currently dormant**: no mint path is wired, supply is 0; lock/vote/propose activate with a
  future upgrade.

**Smart contract stack** (documented here, implemented in the smart-contracts repo):
- Solidity 0.8.24, OpenZeppelin UUPS upgradeable proxies, Foundry framework.
- **Live on Base Mainnet (chain 8453)**, governed by a 2-of-3 Gnosis Safe; core-contract upgrades
  route through an OZ TimelockController with a 12h delay.
- Key contracts: `QuantillonVault` (mint/redeem + external-vault yield distribution), `QEUROToken`,
  `UserPool`, `HedgerPool` (single-hedger EUR-short model), `YieldShift`, `FeeCollector`,
  `stQEUROFactory`/`stQEUROToken`, `OracleRouter` (active EUR/USD source:
  `HyperliquidEurUsdOracle`, fed on-chain via `SlippageStorage`; `ChainlinkOracle` as fallback and
  the USDC/USD source), external staking adapters (MetaMorpho live as vaultId 2).
- There is no `AaveVault` or `LiquidationSystem` contract — liquidation-mode redemption lives in
  `QuantillonVault`; Aave/Morpho exposure goes through the staking vault adapters.

**Governance parameters** (as coded in QTIToken; inactive while QTI is dormant):
- Proposal threshold: 100k QTI | Quorum: 1M QTI | Voting period: 3–14 days | Execution delay: 2 days
- veQTI locks: 7 days min, 365 days max, up to 4× voting power
- Older "60–85% decision thresholds / 4-year lock" figures were aspirational design, not code

**Key live parameters (verified 2026-07-04):** HedgerPool max leverage 20× (5% min margin,
100 USDC min, fees 0, eur/usd interest 350/450 bps, rewardFeeSplit 20%, single-hedger model);
UserPool stakingAPY 8% / depositAPY 4%, 100 QEURO min stake, 7-day cooldown; stQEURO yieldFee 0
(max 20%); FeeCollector split 60/25/15 treasury/dev/community; YieldShift base 50% / max 90%,
7-day holding, MAX_HISTORY_LENGTH 1000; oracle staleness: Hyperliquid 900 s (1 h hard cap),
Chainlink 2 h EUR/USD / 25 h USDC/USD, 5% deviation breakers, 0.80–1.40 bounds.

**QTI distribution (100M total — planned/aspirational; supply is currently 0):**
- Community & Ecosystem 50%, Team & Founders 15%, Investors 10%, DAO Treasury 13%, Liquidity 5%, Strategic Partners 5%, Advisors 2%

**Regulatory positioning:**
- Targeting MiCA Recital 22 exemption (fully decentralized, no central issuer)
- Three-entity structure: Quantillon Protocol (on-chain), Quantillon Labs (dev — French SAS,
  Mérignac, SIREN 988 682 613), Quantillon Foundation (planned future entity, jurisdiction TBD,
  intended regulatory interface)
- Active engagement with French ACPR

**Development phase** (as of 2026-07-04): live on Base mainnet since June 2026 — core protocol
deployed and operating (mint/redeem, staking with external-vault yield distribution, autonomous
hedging with an independent watchdog); QTI governance activation pending.
