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
- **Never use em-dashes (U+2014) anywhere in this repo.** Prefer a comma, semicolon or
  parentheses; use a standard hyphen `-` only when a dash is really needed. Applies to prose,
  table cells, headings and code-block comments alike. Existing en-dashes in numeric ranges
  (0.80-1.40, 3-14 days) are left untouched.
- Commit messages follow conventional commits (`docs:`, `fix:`, `feat:`).

## Protocol Domain Knowledge

When editing documentation, the following are established facts about the protocol
(as of 2026-09-04, verified against live Base mainnet state via cast; the smart-contracts repo
is the source of truth for on-chain behavior, the live chain beats any .md file):

**Three-token ecosystem:**
- **QEURO** - Euro-pegged stablecoin, USDC-collateralized. **No fixed tokenomic supply cap** -
  supply is bounded by hedging capacity: minting requires the protocol CR to stay above the
  governance-set minting floor, **currently 102.5 %** (lowered from 105 % on 2026-09-02; hard
  minimum 101 %); liquidation/critical mode engages at 101 % (pro-rata redemption inside
  QuantillonVault). Token-level guardrails (not supply policy): administrative supply ceiling 100M
  (governance-raisable) and a **global** mint/burn rate limit of 10M QEURO per 300-block window
  (~10 min on Base). Mint/redeem fees are 0 (governance-settable, max 5 %). Minting also enforces a
  2 % price-deviation guard against the oracle cache.
- **stQEURO** - Yield-bearing staked QEURO (ERC-4626 vault, one series per external vault via
  `stQEUROFactory`; live series `stQEUROMORPHO1`, vaultId 2); exchange rate rises as vault yield
  is credited by `QuantillonVault.harvestAndDistributeVaultYield` - no rebasing, no claim.
- **QTI** - Governance token with vote-escrow mechanics (veQTI), 100M supply cap.
  **Currently dormant**: no mint path is wired, supply is 0; lock/vote/propose activate with a
  future upgrade. Never describe QTI governance, QTI incentives or QTI vesting in the present tense.

**Smart contract stack** (documented here, implemented in the smart-contracts repo):
- Solidity 0.8.24, OpenZeppelin UUPS upgradeable proxies, Foundry framework.
- **Contracts deployed on Base Mainnet (chain 8453) since June 2026; the public launch (the
  application open to users) is planned for Q4 2026 - docs say "deployed", never "live/open
  to users" for the product**, governed by a 2-of-3 Gnosis Safe
  (0x1d7fF432…e6cd). Upgrades of the core contracts (QuantillonVault, QEUROToken, QTIToken,
  UserPool, HedgerPool, YieldShift, stQEUROFactory, stQEUROToken) go through the OZ
  TimelockController (0x7Ade8f3B…8342, 12 h). FeeCollector, OracleRouter, ChainlinkOracle,
  HyperliquidEurUsdOracle, LighterEurUsdOracle and SlippageStorage are upgraded directly by the
  Safe (no timelock). Operational roles are delegated: the independent hedging watchdog wallet
  holds a pause-only `EMERGENCY_ROLE` on QuantillonVault; keeper wallets hold
  `VAULT_OPERATOR_ROLE` / `YIELD_DISTRIBUTOR_ROLE`; the oracle publisher (and the operational
  deployer key) hold `WRITER_ROLE` on SlippageStorage; the OracleRouter holds
  ORACLE_MANAGER + EMERGENCY on the market oracle by deploy design. There is no on-chain
  "guardian" role: the guardian concept = `EMERGENCY_ROLE` (+ `PAUSER_ROLE` on QEURO).
  HedgerPool has no `HEDGER_ROLE`: the single hedger is the `singleHedger` address (Quantillon
  Labs' hedging engine). YieldShift does have a `YIELD_MANAGER_ROLE` (held by the Safe);
  stQEUROToken does not.
- **Disclosure rule (D-6):** describe roles and controls; never name operator wallet addresses
  (watchdog, keepers, publisher, deployer/hedger/treasury EOA). Contract addresses are fine.
- Live contracts and versions (2026-09-04): QuantillonVault 1.1.11 (0x833E5Ba5…4a07),
  QEUROToken 1.0.6, QTIToken 1.0.2, UserPool 1.0.3, HedgerPool 1.0.8, FeeCollector 1.0.2
  (split 60/25/15 treasury/dev/community), YieldShift 1.0.5, stQEUROFactory 1.0.1,
  stQEUROToken 1.0.3, OracleRouter 1.1.1, ChainlinkOracle 1.0.4, HyperliquidEurUsdOracle 1.0.2,
  SlippageStorage 1.0.2, LighterEurUsdOracle 1.0.1 (inert, no router role), StorkOracle
  (parked, unused), TimeProvider (plain contract, no proxy), MetaMorphoStakingVaultAdapter
  0xb2f253Cd…8EA3 → MetaMorpho USDC vault 0xBEEFE94c…83b2. The inventory with full addresses is
  `protocol/smart-contract-components.md` §1.
- **Oracle**: `OracleRouter` slot 1 (MARKET, ex-STORK) = `HyperliquidEurUsdOracle` (active):
  the Hyperliquid `xyz:EUR` perp mid is published on-chain into `SlippageStorage` (source 1,
  minUpdateInterval 20 s) by Quantillon's slippage-monitor publisher; staleness 900 s (hard cap
  1 h), bounds 0.80–1.40, 5 % deviation breaker, `isValid=false` = hard stop. Slot 0 =
  `ChainlinkOracle` = one-tx fallback (`switchOracle(0)` by the Safe) and USDC/USD source (2 h
  EUR/USD / 25 h USDC/USD staleness, 5 % deviation, ±2 % USDC tolerance, Base L2
  sequencer-uptime check with 1 h grace, dev mode = two-step 48 h, disabled). An independent
  watchdog freezes mint+redeem (vault pause) on stale/circuit-broken oracle, HL-vs-Chainlink
  basis > 100 bps, or unhealthy hedging; it lifts only its own pauses.
- There is no `AaveVault` or `LiquidationSystem` contract (historical design names) -
  liquidation-mode redemption lives in `QuantillonVault`; Aave/Morpho exposure goes through the
  staking vault adapters (MetaMorpho live as vaultId 2). There is no vault-level "emergency
  withdraw" from the external vault; USDC leaves it on redemption or when governance
  deactivates the vault.
- Reference pages are regenerated from the deployed ABIs (`quantillon-dapp/src/lib/contracts/
  abis/abi-only/*.abi.json`); signatures, structs, events and errors come from there and from
  the source, never from memory. Owner page per fact (STR-6): mechanisms.md and the token pages
  are conceptual and link to the owner page; never state a number in two places.

**Hedging (CTO decisions):**
- **Hedge venue = Hyperliquid (`xyz:EUR` perp) only.** A venue-switchable Lighter path was built
  and its oracle deployed in July 2026; on **2026-09-01 Lighter was ruled out for good**. No page
  may present a Lighter cutover as supported, planned or pending. Exactly one historical note
  exists, on `protocol/oracle-architecture.md` ("The Market Slot"); do not add others.
- **Margin policy (on-chain since 2026-09-02):** HedgerPool minMarginRatio 250 bps (2.5 %,
  contract floor `DEFAULT_MIN_MARGIN_RATIO_BPS` = 250), mint floor 102.5 %. Quantillon Labs'
  hedging engine rebalances margin between the HedgerPool pocket (Base, short EUR) and the
  Hyperliquid pocket (long EUR) toward a 2.5 % equity/notional target in 25 bps steps: EUR down →
  bounded Quantillon→Hyperliquid transfers, one at a time, blocked if the projected CR would
  come too close to the mint floor; EUR up → manual fresh-USDC top-up + alert. As of 2026-09-05
  the automated transfer path runs in observation (dry-run) - describe the policy on public
  pages, not an automated execution. **Public depth (D-4): qualitative only - target, step
  size, direction asymmetry, CR guard, the 250 bps contract floor. Never publish the operated
  floor/buffers, USDC caps, daily limits, the bridge route or env variable names.** Canonical
  doc: `quantillon-dapp/docs/hedging-engine-margin-rebalancing.md`. Owner section:
  `protocol/hedger-pool.md` "Operational margin policy (September 2026)"; other pages carry a
  two-sentence summary and a link.
- Hedger funding carve-out on harvests: `fundingRateAnnualBps` = 0, no recipient set (max 50 %).
  Multi-hedger is not promised anywhere: "single designated hedger in the current phase".

**Key live parameters (verified 2026-09-04):** HedgerPool coreParams = min margin 250 bps,
max leverage 20× (setter cap 20; `MAX_LEVERAGE` constant is type(uint16).max), entry/exit/margin
fees 0, eur/usd interest 350/450 bps, rewardFeeSplit 20 %, minMarginAmount 0,
minPositionHoldBlocks 0, single-hedger model; UserPool stakingAPY 8 % / depositAPY 4 %,
100 QEURO min stake, 7-day cooldown, performanceFee 0 (UserPool is an optional batch contract;
the dApp uses the vault and stQEURO directly); stQEURO yieldFee 0 (max 20 %); FeeCollector
60/25/15; YieldShift base 50 % / max 90 %, adjustmentSpeed 100, targetPoolRatio 10000, 7-day
holding, 24 h TWAP, MAX_HISTORY_LENGTH 1000, no yield source authorised live; QEURO whitelist
mode off, killswitch off. **Usage is near zero** (QEURO supply 1 wei, no hedger margin, no
stakes) - **rule (D-12): never quote TVL, volume or user counts as current; every projection
or target is labelled illustrative / target, not a current figure.**
**Canonical TVL targets (the only trajectory any page may state, always as a target):**
Q4 2026 (public launch) $1M; Q2 2027 $10M; Q1 2028 $10M to $100M. Owner pages:
`roadmap-and-adoption-strategy.md` (Directional Targets table),
`strategic-context/market-landscape-and-competitive-analysis.md` (SOM) and the business-model
KPI list - the three must stay identical. **No revenue, burn-rate, operating-margin or
user-count projections are published anywhere** (the €50M/€500M revenue scenarios and the
€20M-TVL surplus claim were removed 2026-09-08); do not reintroduce them.

**Governance parameters** (as coded in QTIToken; inactive while QTI is dormant):
- Proposal threshold: 100k QTI | Quorum: 1M QTI | Voting period: 3–14 days | Execution delay: 2 days
- veQTI locks: 7 days min, 365 days max, up to 4× voting power; no delegation function
- `cancelProposal`: proposer or GOVERNANCE_ROLE (not the emergency role)
- Older "60–85% decision thresholds / 4-year lock" figures were aspirational design, not code

**QTI distribution (100M total - planned/aspirational; supply is currently 0):**
- Community & Ecosystem 40%, Treasury & Liquidity 30%, Team & Advisors 20%, Investors (SAFT/BSA) 10%.
  Four buckets only (revised 2026-09-08): no separate Strategic Partners, Advisors, DAO Treasury or
  Liquidity Provision line. Owner tables: `tokenomics-and-governance.md` and
  `protocol/quantillon-protocols-tokens/qti-token.md` - they must stay identical.

**Aspirational content (D-10):** multi-collateral, cross-chain, buybacks/burns, insurance fund,
5-year projections, month-based governance phases live only under the "Aspirational - not
implemented, no timeline" callout on `roadmap-and-adoption-strategy.md`; prune them from token
and component pages.

**Quantillon Rewards (QP points, off-chain):** terms published 2026-09-04
(`complementary-information/rewards-program-terms.md`, v1) and reflected in the privacy policy.
The rewards engine and gateway proxy are deployed on the live host, but the program is closed by
its master switch and the UI is hidden - say **"deployed, program not yet open, no date"**
wherever it is mentioned (terms page status line, roadmap, privacy policy, tokenomics, FAQ,
glossary); never "no proxy" and never a launch date. QP have no monetary value and promise no
token. The design documents in the dapp repo (`docs/rewards-program.md`,
`docs/rewards-program-legal-notes.md`, `GAMIFICATION_SPECS.md`) are local-only and must never be
linked or quoted; only `docs/rewards-program-runbook.md` is committed.

**Security claims (D-9):** **no audit by a professional security firm has ever been carried out.**
What exists: repeated code-review passes with several frontier AI code-analysis models, plus
findings from independent whitehat researchers; findings are remediated on-chain continuously,
along the way, through the normal upgrade path. **Never publish a remediation date or present
remediation as a completed one-off milestone** - it is ongoing work. Describe it as "AI-driven
security review + whitehat reports, remediated on-chain as findings come in", always paired
with the fact that an audit-firm review has not been done; a bug bounty is *planned*. Never
write "independent audit", "audited by", "formal verification", "multiple audits" or
"continuous bug bounty" (calling OpenZeppelin
and Gnosis Safe themselves audited is fine - that is their audit, not ours). Owner page for the
security posture: `risk-management-and-sustainability/risks-and-mitigation-strategies.md`
("Smart Contract and Oracle Risk"); other pages carry one sentence and link to it. Do not name
specific AI models or vendors on public pages. If a security contact is needed, use
team@quantillon.money.

**Regulatory positioning:**
- Targeting the MiCA Recital 22 exemption (fully decentralized, no central issuer) - "targeting",
  never "confirmed"/"acknowledged" unless a written regulator position is cited; the Recital 22
  tables on `strategic-context/macro-regulatory-and-legal.md` are target state vs current state
- Three-entity structure: Quantillon Protocol (on-chain), Quantillon Labs (dev - French SAS,
  2 Avenue de Lognac, 33700 Mérignac, RCS Bordeaux / SIREN 988 682 613, VAT FR87988682613),
  Quantillon Foundation (planned future entity, legal form and jurisdiction TBD - never "Swiss"
  or "Loi 1901" as fact)
- Ongoing dialogue with the French ACPR on the Recital 22 analysis
- Legal pages: Terms of Service carry an interim sentence until published; the legal notice's
  registered office / publication director and the privacy-policy wording items are with counsel

**Community links (canonical, D-14):** discord.gg/uk8T9GqdE5 · x.com/QuantillonLabs ·
t.me/QuantillonLabs (discord.gg/quantillon is dead).

**Timeline** (for the roadmap page): Base mainnet launch June 2026; Hyperliquid oracle live
2026-06-25; 10-proxy upgrade bundle live 2026-07-06 (security fixes; do not publish it as a
remediation date); stQEUROToken 1.0.3 + HedgerPool 1.0.7
mid-July; frontend restyle + public protocol dashboard July 2026; Lighter evaluation closed
2026-09-01; QuantillonVault 1.1.11 (loss-aware external-vault collateral) 2026-08-17;
eight-contract maintenance bundle live after 2026-08-26; HedgerPool 1.0.8 + margin policy +
102.5 % mint floor 2026-09-02; Rewards terms 2026-09-04 (program not open). Pending: public
launch Q4 2026 (application open to users), QTI activation, Foundation, additional external vaults.
