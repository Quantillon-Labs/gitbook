# Quantillon Protocol GitBook

Central documentation hub for the Quantillon Protocol - a Euro-native DeFi infrastructure project providing comprehensive knowledge base for developers, users, regulators, and investors.

## Purpose

This is a **documentation-only repository** (no smart contracts or application code). It contains:

- Protocol whitepaper and technical specifications
- Three-token ecosystem documentation (QEURO, stQEURO, QTI)
- Regulatory and compliance analysis (MiCA framework)
- Business model and economic sustainability
- Governance structure and tokenomics
- MVP technical specifications and roadmap

## Protocol Overview

**Quantillon Protocol** addresses the gap in Euro-native DeFi with:

- **QEURO**: Euro-pegged stablecoin (101%+ USDC overcollateralized)
- **stQEURO**: Yield-bearing auto-compounding wrapper with instant liquidity
- **QTI**: Governance token with vote-escrow mechanics (veQTI)
- **Dual-Pool Architecture**: Separates user deposits from hedging operations
- **Native EUR/USD Hedging**: Integrated FX risk management

## Repository Structure

```
gitbook/
├── SUMMARY.md                     # Table of contents (GitBook navigation)
├── introduction/
│   └── whitepaper.md             # Why Quantillon: macro context
├── protocol/                      # Core technical docs
│   ├── quantillon-protocol-design-and-architecture.md
│   ├── mechanisms.md             # QEURO mechanics, dual-pool, Yield Shift
│   ├── appendices-techniques.md  # Smart contracts, oracles, formulas
│   └── quantillon-protocols-tokens/
│       ├── qeuro-token.md
│       ├── stqeuro-token.md
│       └── qti-token.md
├── strategic-context/             # Business & regulatory
│   ├── macro-and-regulatory-context.md
│   ├── market-and-competitive-analysis.md
│   ├── business-model-and-economic-sustainability.md
│   └── regulatory-and-legal.md   # MiCA compliance, Recital 22
├── risk-management-and-sustainability/
├── complementary-information/     # FAQ, glossary, legal pages
├── mvp-technical-documentation.md # Development phases 1-7
├── roadmap-and-launch-plan.md    # Q2 2025 - Q4 2026
├── tokenomics-and-governance.md
└── team-and-operational-structure.md
```

## Key Technical Details

**Smart Contract Stack** (documented, not in this repo):
- Solidity 0.8.x with OpenZeppelin upgradeable proxies
- Foundry development framework
- Base Mainnet deployment (L2)
- Chainlink oracle feeds

**Governance Parameters**:
- Proposal threshold: 100k QTI
- Quorum: 20% voting power
- Voting period: 4 days
- Decision thresholds: 60-85% based on proposal type

**Token Distribution** (100M QTI total):
- Community & Ecosystem: 50%
- Team & Founders: 15%
- Investors: 13%
- DAO Treasury: 10%
- Strategic Partners: 5%
- Advisors: 2%
- Liquidity: 5%

## Development Status

Currently in **Phase 3 (MVP Build)** of 7-phase roadmap:
- Phase 1-2: Complete (Architecture & Whitepaper)
- Phase 3: In Progress (Core contracts, internal alpha, 95%+ test coverage)
- Phases 4-7: External audits → Public beta → Mainnet → Advanced features

## Regulatory Framework

Protocol designed for **MiCA Recital 22 exemption** (fully decentralized):
- No intermediaries or central issuer
- Structured through three entities:
  - **Quantillon Protocol**: On-chain governance
  - **Quantillon Labs**: Development entity
  - **Quantillon Foundation**: Swiss-style non-profit, regulatory interface
- Early engagement with French ACPR

## Conventions

- Markdown-based, GitBook-compatible structure
- Navigation defined in `SUMMARY.md`
- Living documentation (continuously updated)
- CI/CD: Telegram notifications on push to main/dev
