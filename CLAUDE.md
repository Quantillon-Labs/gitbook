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

When editing documentation, the following are established facts about the protocol:

**Three-token ecosystem:**
- **QEURO** — Euro-pegged stablecoin, 101%+ USDC overcollateralized
- **stQEURO** — Yield-bearing auto-compounding wrapper with instant liquidity
- **QTI** — Governance token with vote-escrow mechanics (veQTI), 100M total supply

**Smart contract stack** (documented here, implemented in a separate repo):
- Solidity 0.8.x, OpenZeppelin upgradeable proxies, Foundry framework
- Base Mainnet (L2), Chainlink oracle feeds
- Key contracts: `UserPool`, `HedgerPool`, `YieldShift`, `AaveVault`, `ChainlinkOracle`, `LiquidationSystem`

**Governance parameters:**
- Proposal threshold: 100k QTI | Quorum: 20% | Voting period: 4 days
- Decision thresholds: 60–85% depending on proposal type

**QTI distribution (100M total):**
- Community & Ecosystem 50%, Team & Founders 15%, Investors 10%, DAO Treasury 13%, Liquidity 5%, Strategic Partners 5%, Advisors 2%

**Regulatory positioning:**
- Targeting MiCA Recital 22 exemption (fully decentralized, no central issuer)
- Three-entity structure: Quantillon Protocol (on-chain), Quantillon Labs (dev), Quantillon Foundation (Swiss non-profit, regulatory interface)
- Active engagement with French ACPR

**Development phase** (as of 2026-04-07): Phase 3 of 7 — MVP Build in progress (core contracts, internal alpha, 95%+ test coverage target).
