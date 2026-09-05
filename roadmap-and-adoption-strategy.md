# Roadmap & Adoption Strategy

> **Status (September 2026): the Quantillon Protocol is LIVE on Base mainnet.** Core contracts were deployed in June 2026 and the protocol is operating — mint/redeem, multi-vault staking with external-vault yield distribution, and autonomous hedging with an independent safety watchdog. QTI governance activation is the main remaining milestone.

### Where We Are

The original multi-phase plan (foundation → architecture → MVP → testnet → audits → mainnet) has been executed. The table below records the completed journey and what remains:

| Milestone | Status |
| --- | --- |
| 🏗️ Team assembly, whitepaper, technical architecture (2025) | ✅ Completed |
| 🛠️ MVP build, internal alpha, public testnet (Base Sepolia) | ✅ Completed |
| 🔒 Security audit & remediation (audit fixes deployed on-chain 6 July 2026) | ✅ Completed |
| 🚀 **Mainnet launch on Base (chain 8453)** | ✅ **Live since June 2026** |
| 📡 Oracle go-live — Hyperliquid EUR/USD via on-chain publisher (2026-06-25) | ✅ Completed |
| 🏦 Multi-vault staking — MetaMorpho external vault live (`vaultId` 2, `stQEUROMORPHO1`) | ✅ Completed |
| ⚖️ Autonomous hedging engine + independent watchdog | ✅ Operating |
| 🎨 Frontend restyle + public protocol dashboard (July 2026) | ✅ Completed |
| 🔀 Alternative hedge-venue evaluation (Lighter, July 2026) | ✅ Closed — Hyperliquid confirmed as the sole venue (September 2026) |
| 🧮 QuantillonVault 1.1.11 — loss-aware external-vault collateral accounting (17 August 2026) | ✅ Live |
| 📦 Eight-contract maintenance bundle (after 26 August 2026) | ✅ Live |
| ⚖️ HedgerPool 1.0.8 — 2.5% margin policy and 102.5% minting floor (2 September 2026) | ✅ Live |
| 🎁 Quantillon Rewards — terms published (4 September 2026) | ⏳ Program not yet open |
| 🗳️ QTI governance activation (token is deployed but dormant — supply 0) | ⏳ Pending |
| 🏛️ Quantillon Foundation establishment | ⏳ Planned |

***

### Current Focus (H2 2026)

* 📈 **Liquidity & TVL growth** — scaling QEURO supply and stQEURO staking on the live deployment
* 🧱 **Operational hardening** — monitoring, watchdog coverage, oracle resilience, keeper automation
* ⚖️ **Hedge capital efficiency** — the 2.5% margin policy live since September 2026 (see [HedgerPool](protocol/hedger-pool.md#operational-margin-policy-september-2026))
* 🎁 **Quantillon Rewards** — the off-chain points program is deployed and gated; it will open in the application at a later date (no date committed)
* 🏦 **Additional external vaults** — onboarding further yield venues through the staking-vault adapter pattern
* 🗳️ **QTI activation preparation** — the activation upgrade that mints the supply cap and enables lock/vote/propose
* 🏛️ **Foundation setup** — establishing the Quantillon Foundation as the regulatory interface (jurisdiction to be finalized)

***

### Future Phases (Aspirational)

> **Aspirational — not implemented, no timeline.** The items below are directional ideas, **subject to governance decisions, market depth, hedger participation, and protocol economics** — not commitments. None of them is in the deployed code, and no date is attached to any of them.

#### Multi-Vault Expansion & Institutional Integration

* 🏛️ Additional external-vault stQEURO series (further Morpho vaults, Aave, RWA-backed venues)
* 🤝 Strategic partnerships with CeDeFi brokers and institutional fintech platforms
* 💳 Fiat on/off-ramp integrations with regulated custody partners
* 📊 Institutional reporting features and treasury-management tooling
* 📱 Mobile-first retail experience

#### Advanced DeFi Integration & Full Decentralization

* 🌐 Cross-chain expansion beyond Base — bridging, staking and governance on other networks (subject to governance)
* 🪙 Additional collateral types beyond USDC (e.g. ETH, WBTC, governance-approved assets)
* 🔥 Token-economic mechanisms for QTI after activation (buybacks, burns, insurance fund, multi-layer proposal model)
* 🏦 DAO-controlled treasury with full fee-switch implementation
* 🗳️ Full protocol decentralization — parameter control transferred to veQTI holders
* 🔗 Broad DeFi integrations with stQEURO as collateral
* 🌍 Additional local-currency deployments of the Quantillon architecture beyond EUR

**🎯 Directional Targets** (targets, not current figures):

| Metric | Near term | 12 months | 24 months |
| --- | --- | --- | --- |
| **📊 TVL** | €10M | €100M | €1B+ |
| **👥 User Base** | 10K | 50K | 200K+ |
| **🏦 External Vaults** | 1 (MetaMorpho) | 3 variants | 4+ variants |
| **🗳️ Governance** | Safe + timelock | QTI activated | Progressive decentralization |

***

### Adoption Strategy

Adoption is not only technical but also social and economic. Quantillon's adoption strategy includes:

**🌍 Community Growth**

Incentivized programs for ambassadors, contributors, and early adopters. The first will be Quantillon Rewards, an off-chain points program for QEURO depositors and stakers; its terms are published ([Rewards Program Terms](complementary-information/rewards-program-terms.md)) and the program will open in the application at a later date.

**🏢 B2B Pipelines**

Outreach to wealth managers, DAOs, family offices, and treasuries.

**🛠️ Ecosystem Grants**

For developers building QEURO-integrated products (e.g., wallets, cross-chain bridges, tax tools).

**📚 Educational Content**

Financial literacy campaigns focused on DeFi yields, euro-denominated finance, and MiCA.

***

### Success Factors & Risk Mitigation

**🔒 Security First** — audits, an independent watchdog service, and staged rollouts protect users at every step.

**👥 Community-Driven Development** — early and continuous community engagement to build a sustainable ecosystem.

**🏛️ Regulatory Compliance** — proactive approach to European regulatory requirements, particularly MiCA (see [Macro, Regulatory & Legal](strategic-context/macro-regulatory-and-legal.md)).

**🔗 Strategic Partnerships** — building key relationships with DeFi protocols, institutions, and service providers.

> **Quantillon is not merely launching a product — it is proving a reusable protocol pattern. The roadmap emphasizes responsible growth, composability, and transparency: QEURO is the first EUR deployment, while future local-currency expansion remains subject to governance, liquidity, hedger participation, and economics.**
