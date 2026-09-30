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

Use `protocol/deployment-status.md` for dated production settings and `protocol/smart-contract-components.md` for addresses and versions. The 30 September 2026 review uses finalized Base block 51,985,907. Source code and live chain state take precedence over prose. Historical settings must not be presented as current defaults.

- Public launch remains a Q4 2026 target. Contracts are deployed on Base only. QTI supply is zero and governance is dormant; Rewards is deployed but closed with no opening date.
- Core DEFAULT_ADMIN_ROLE belongs to the 12-hour TimelockController. The 2-of-3 Safe retains operational roles and peripheral administration. Do not describe core role revocation as an instant Safe call.
- Hedge venue is Hyperliquid only. Maximum configured HedgerPool leverage is 40x. Distinguish in-venue order margin preparation from transfers between venues. Public margin policy stays qualitative; do not publish operated buffers, transfer caps, bridge routes or environment variable names.
- Oracle reference valuation and directional execution quotes are different. Zero mint/redeem fees do not mean slippage-free execution. Oracle, pause, capacity and liquidity guards can block transactions.
- Morpho harvests use the harvest-time staked fraction. Unstaked yield goes to treasury; hedger receives only a configured haircut (verified zero). The legacy yieldFee is ignored by vault 1.5.0. Vesting does not create time-weighted ownership. YieldShift and optional UserPool accounting are separate paths.
- No professional security audit has been performed. Only internal, AI-assisted review is confirmed. Do not name AI vendors, claim external/whitehat review, promise an audit report, or portray remediation as a completed milestone. Security contact: team@quantillon.money.
- Never publish operator wallet addresses, private audit records, transaction JSON or Safe import payloads. Contract addresses are public.
- No current TVL, volume or user-count claim without dated evidence. Directional TVL targets: Q4 2026 $1M; Q2 2027 $10M; Q1 2028 $10M to $100M. These are targets, not forecasts. Do not publish revenue or user-count projections.
- QTI distribution is planned: community/ecosystem 40%, treasury/liquidity 30%, team/advisors 20%, investors 10%. Supply remains zero. Multi-collateral, cross-chain, insurance and buyback ideas belong in the roadmap's aspirational section.
- Quantillon Labs is the French operating company; the Foundation is planned and its jurisdiction is undecided. Do not claim a confirmed MiCA exemption or regulatory approval. Legal classification is not established by a code review.
- Terms of Service v1.0 are published, effective 24 September 2026. Rewards terms are separate; login does not automatically enroll a user. Private dapp rewards and legal design notes must not be linked or published.
- Canonical community links: discord.gg/uk8T9GqdE5, x.com/QuantillonLabs, t.me/QuantillonLabs.
