# Roadmap & Adoption Strategy

> **Status (September 2026): the Quantillon Protocol contracts are deployed on Base mainnet; the public launch is planned for Q4 2026.** Core contracts were deployed in June 2026 and have since been through AI-driven security review with on-chain remediation, oracle go-live and operational hardening - mint/redeem, multi-vault staking with external-vault yield distribution, and autonomous hedging with an independent safety watchdog are all in place on-chain. The application is not yet open to users: **public launch is planned for Q4 2026**, with QTI governance activation to follow.

### Where We Are

The original multi-phase plan (foundation → architecture → MVP → testnet → mainnet) has been executed; security review and remediation continue as ongoing work. The table below records the journey so far and what remains:

| Milestone | Status |
| --- | --- |
| 🏗️ Team assembly, whitepaper, technical architecture (2025) | ✅ Completed |
| 🛠️ MVP build, internal alpha, public testnet (Base Sepolia) | ✅ Completed |
| 🔒 AI-driven security review + whitehat reports, remediated on-chain | 🔄 Ongoing |
| 🔍 Audit by a professional security firm | ⏳ Not yet performed |
| 🚀 **Core contracts deployed on Base (chain 8453)** | ✅ **Deployed June 2026** |
| 📡 Oracle go-live - Hyperliquid EUR/USD via on-chain publisher (2026-06-25) | ✅ Completed |
| 🏦 Multi-vault staking - MetaMorpho external vault live (`vaultId` 2, `stQEUROMORPHO1`) | ✅ Completed |
| ⚖️ Autonomous hedging engine + independent watchdog | ✅ Operating |
| 🎨 Frontend restyle + public protocol dashboard (July 2026) | ✅ Completed |
| 🔀 Alternative hedge-venue evaluation (Lighter, July 2026) | ✅ Closed - Hyperliquid confirmed as the sole venue (September 2026) |
| 🧮 QuantillonVault 1.1.11 - loss-aware external-vault collateral accounting (17 August 2026) | ✅ Live |
| 📦 Eight-contract maintenance bundle (after 26 August 2026) | ✅ Live |
| ⚖️ HedgerPool 1.0.8 - margin policy targeting 2.5% and 102.5% minting floor (2 September 2026) | ✅ Live |
| 🎁 Quantillon Rewards - terms published (4 September 2026) | ⏳ Program not yet open |
| 🌐 **Public launch - application open to users** | ⏳ **Planned Q4 2026** |
| 🗳️ QTI governance activation (token is deployed but dormant - supply 0) | ⏳ Pending |
| 🏛️ Quantillon Foundation establishment | ⏳ Planned |

***

### Current Focus (H2 2026)

* 🌐 **Public launch preparation (Q4 2026)** - final readiness work before the application opens to users
* 📈 **Liquidity & TVL growth** - preparing QEURO supply and stQEURO staking to scale from launch
* 🧱 **Operational hardening** - monitoring, watchdog coverage, oracle resilience, keeper automation
* ⚖️ **Hedge capital efficiency** - the margin policy targeting 2.5%, live since September 2026 (see [HedgerPool](protocol/hedger-pool.md#operational-margin-policy-september-2026))
* 🎁 **Quantillon Rewards** - the off-chain points program is deployed and gated; it will open in the application at a later date (no date committed)
* 🏦 **Additional external vaults** - onboarding further yield venues through the staking-vault adapter pattern
* 🗳️ **QTI activation preparation** - the activation upgrade that mints the supply cap and enables lock/vote/propose
* 🏛️ **Foundation setup** - establishing the Quantillon Foundation as the regulatory interface (jurisdiction to be finalized)

***

### Future Phases (Aspirational)

> **Aspirational - not implemented, no timeline.** The items below are directional ideas, **subject to governance decisions, market depth, hedger participation, and protocol economics** - not commitments. None of them is in the deployed code, and no date is attached to any of them.

#### Multi-Vault Expansion & Institutional Integration

* 🏛️ Additional external-vault stQEURO series (further Morpho vaults, Aave, RWA-backed venues)
* 🤝 Strategic partnerships with CeDeFi brokers and institutional fintech platforms
* 💳 Fiat on/off-ramp integrations with regulated custody partners
* 📊 Institutional reporting features and treasury-management tooling
* 📱 Mobile-first retail experience

#### Advanced DeFi Integration & Full Decentralization

* 🌐 Cross-chain expansion beyond Base - bridging, staking and governance on other networks (subject to governance)
* 🪙 Additional collateral types beyond USDC (e.g. ETH, WBTC, governance-approved assets)
* 🔥 Token-economic mechanisms for QTI after activation (buybacks, burns, insurance fund, multi-layer proposal model)
* 🏦 DAO-controlled treasury with full fee-switch implementation
* 🗳️ Full protocol decentralization - parameter control transferred to veQTI holders
* 🔗 Broad DeFi integrations with stQEURO as collateral
* 🌍 Additional local-currency deployments of the Quantillon architecture beyond EUR

**🎯 Directional Targets** (targets, not current figures):

| Metric | Q4 2026 (launch) | Q2 2027 | Q1 2028 |
| --- | --- | --- | --- |
| **📊 TVL** | $1M | $10M | $10M to $100M |
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

**🔒 Security First** - AI-driven code review and whitehat reports with on-chain remediation, an independent watchdog service, and staged rollouts. A professional audit-firm review has not been carried out to date; see [Risks & Mitigation](risk-management-and-sustainability/risks-and-mitigation-strategies.md).

**👥 Community-Driven Development** - early and continuous community engagement to build a sustainable ecosystem.

**🏛️ Regulatory Compliance** - proactive approach to European regulatory requirements, particularly MiCA (see [Macro, Regulatory & Legal](strategic-context/macro-regulatory-and-legal.md)).

**🔗 Strategic Partnerships** - building key relationships with DeFi protocols, institutions, and service providers.

> **Quantillon is not merely launching a product - it is proving a reusable protocol pattern. The roadmap emphasizes responsible growth, composability, and transparency: QEURO is the first EUR deployment, while future local-currency expansion remains subject to governance, liquidity, hedger participation, and economics.**
