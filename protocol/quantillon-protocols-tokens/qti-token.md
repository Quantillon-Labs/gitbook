# QTI Token

## QTI Tokenomics: Advanced Economic Design

> **⏸️ Current Status: DORMANT.** The QTI contract is deployed on Base with a 100,000,000 QTI supply cap, but **live supply is 0** - no mint path is wired, and no tokens have been issued or distributed. Lock (veQTI), voting, and proposal functions are inactive until a future activation upgrade. Until then, the protocol is governed by a 2-of-3 Gnosis Safe with a 12-hour upgrade timelock (see [Quantillon DAO](../../quantillon-dao.md)). Everything below describes the coded design and the planned distribution strategy, not a live token.

### 📋 Executive Summary

The Quantillon Governance Token (QTI) represents a sophisticated approach to DeFi governance and value accrual, designed specifically for the Euro stablecoin ecosystem. Built on advanced tokenomic principles including vote-escrow mechanics, dynamic reward systems, and progressive decentralization, QTI creates sustainable incentives for long-term protocol growth while maintaining robust security and community alignment.

Our tokenomic model incorporates cutting-edge mechanisms such as dual-token architecture compatibility, algorithmic emission curves, and multi-layered governance structures that evolve with protocol maturity. The design prioritizes capital efficiency, regulatory compliance, and sustainable yield generation across diverse market conditions.

***

### Core Token Specifications

#### Technical Architecture

| Parameter         | Value                       | Rationale                      |
| ----------------- | --------------------------- | ------------------------------ |
| **Name**          | Quantillon Governance Token | Clear utility identification   |
| **Symbol**        | QTI                         | Memorable, brandable ticker    |
| **Standard**      | ERC-20 (UUPS Upgradeable)   | Future-proof with security     |
| **Network**       | Base L2 (Primary)           | L2 efficiency and lower costs  |
| **Supply cap**    | 100,000,000 QTI (`TOTAL_SUPPLY_CAP`; live supply 0) | Scarcity-driven value model    |
| **Decimals**      | 18                          | Full ERC-20 compatibility      |
| **Contract Type** | OpenZeppelin + Custom Logic | Battle-tested + innovation     |

#### Implemented Features

* **Vote-Escrow (veQTI)**: Lock QTI for enhanced voting power (up to 4x)
* **On-Chain Governance**: Proposal creation, voting, and execution
* **Emergency Controls**: Pausable with time-locked upgrades via UUPS
* **Progressive Decentralization**: Configurable decentralization levels

***

### Token Distribution Architecture

> **Important**: QTI token distribution is managed through governance and operational decisions. The smart contract only defines `TOTAL_SUPPLY_CAP = 100,000,000 QTI`. The allocations below represent the **planned distribution strategy**, not on-chain constraints - and no tokens have been minted yet (supply is 0 while QTI is dormant).

#### Strategic Allocation Framework

| Category                     | Allocation | Amount         | Lock Period | Vesting Schedule | Release Mechanism  |
| ---------------------------- | ---------- | -------------- | ----------- | ---------------- | ------------------ |
| **🌍 Community & Ecosystem** | 40%        | 40,000,000 QTI | Variable    | 48-month curve   | Governance-managed |
| **🏛️ Treasury & Liquidity** | 30%        | 30,000,000 QTI | Immediate   | Governance-gated | Vote-controlled    |
| **👥 Team & Advisors**       | 20%        | 20,000,000 QTI | 12 months   | 36 months linear | Governance-managed |
| **💼 Investors (SAFT/BSA)**  | 10%        | 10,000,000 QTI | 6 months    | 24-36 months     | Governance-managed |

#### Community & Ecosystem Breakdown (40M QTI)

**Phase 1: Bootstrap Incentives (16M QTI - Years 1-2)**

* **Liquidity Mining**: 9M QTI across major DEX pairs
* **User Acquisition**: 4M QTI for onboarding campaigns
* **Developer Grants**: 3M QTI for ecosystem development

**Phase 2: Growth Acceleration (16M QTI - Years 3-4)**

* **Advanced Features**: 7M QTI for new protocol modules
* **Governance Participation**: 5M QTI for voting incentives
* **Cross-Chain Expansion**: 4M QTI for multi-chain deployment

**Phase 3: Maturity & Innovation (8M QTI - Years 5+)**

* **Research & Development**: 3M QTI for protocol evolution
* **Strategic Partnerships**: 3M QTI for institutional integration
* **Community Treasury**: 2M QTI for long-term sustainability

***

### Advanced Utility Mechanisms

#### 🗳️ Governance Architecture

**Vote-Escrow (veQTI) System**

* **Lock Periods**: 7 days minimum to 365 days (1 year) maximum
* **Voting Power**: Linear multiplier up to 4x base weight at max lock
* **Decay Mechanism**: Gradual reduction until unlock
* **Delegation**: not implemented - there is no delegation function in the deployed contract

**Governance Parameters (as coded, inactive while QTI is dormant)**

| Parameter | Value |
|-----------|-------|
| **Proposal threshold** | 100,000 QTI |
| **Quorum** | 1,000,000 QTI |
| **Voting period** | 3 days minimum, 14 days maximum |
| **Execution delay** | 2 days |

> **Aspirational - subject to governance design, not implemented in the deployed contracts**: a multi-layer proposal model (e.g. higher thresholds and longer timelocks for constitutional changes than for operational decisions) is under consideration for the activation upgrade.

#### 💰 Revenue Generation & Distribution

**Primary Revenue Streams (live framework)**

1. **QEURO Operations**: mint/redeem fees via QuantillonVault - currently 0, governance-settable up to 5%
2. **Yield Fees**: per-series stQEURO yield fee (currently 0, capped at 20%) and the treasury share of harvested external-vault yield
3. **Hedger Position Fees**: entry/exit/margin fees (currently 0, governance-settable) plus a 20% reward fee split on hedger rewards

**Revenue Allocation Model (as coded - FeeCollector)**

```
Collected protocol fees (100%)
├── 60% → Treasury
├── 25% → Dev Fund
└── 15% → Community
(governance-adjustable, must sum to 100%)
```

> **Aspirational - subject to governance design, not implemented in the deployed contracts**: routing a share of protocol revenue to veQTI stakers, an insurance fund, or buyback-and-burn programs would require future governance decisions once QTI is activated.

#### 🔄 Dynamic Reward Systems

> **Aspirational - subject to governance design, not implemented in the deployed contracts.** QTI is dormant with zero supply and no mint path; no emissions of any kind are occurring. The schedule and reward formula below are a design sketch for the future activation, not coded behavior.

**Adaptive Emission Schedule (illustrative)**

* **Year 1**: 15M QTI (15% of total supply)
* **Year 2**: 12M QTI (emission decay: -20%)
* **Year 3**: 9.6M QTI (emission decay: -20%)
* **Year 4+**: Market-responsive emissions based on TVL growth

**Multi-Factor Reward Calculation (illustrative)**

```
User Reward = Base Reward × Lock Multiplier × Participation Bonus × Loyalty Factor

Where:
- Lock Multiplier: 1x to 4x based on veQTI lock period
- Participation Bonus: Up to 2x for active governance
- Loyalty Factor: Up to 1.5x for continuous participation
```

***

### Economic Security Framework

#### 🛡️ Anti-Manipulation Measures

**Whale Protection Systems**

* **Vote-Escrow**: Longer lock periods = more voting power (up to 4x)
* **Time-Weighted Voting**: veQTI decays over time until unlock
* **Governance Thresholds**: Configurable proposal and quorum thresholds
* **Timelock Delays**: Critical changes require waiting periods

> **Note**: Additional whale protection mechanisms (voting caps, sybil resistance) may be implemented via governance proposals.

#### 🔐 Security & Risk Management

**Smart Contract Security**

* **Review Process**: AI-driven code review and whitehat reports, remediated on-chain as findings come in; **no professional audit-firm review to date** - see [Risks & Mitigation](../../risk-management-and-sustainability/risks-and-mitigation-strategies.md)
* **OpenZeppelin Base**: Battle-tested upgradeable contracts
* **Bug Bounty Program**: planned (amounts TBD)
* **Continuous Monitoring**: independent hedging/oracle watchdog with automatic pause and alerting

**Operational Security**

* **Multi-Sig Treasury**: 2-of-3 Gnosis Safe
* **Time-Lock Upgrades**: 12-hour OZ TimelockController on core-contract upgrades
* **Emergency Procedures**: Rapid response for critical threats

***

### Governance Activation

QTI governance activates with a future upgrade that wires a mint path and enables lock, vote and propose. Sequencing and prerequisites are tracked on the [Roadmap](../../roadmap-and-adoption-strategy.md); no dates are committed. No financial projections are published for the token.

***

### Risk Analysis & Mitigation

#### Technical Risks

| Risk Factor                 | Probability | Impact   | Mitigation Strategy                             |
| --------------------------- | ----------- | -------- | ----------------------------------------------- |
| **Smart Contract Exploits** | Medium      | Critical | AI-driven review + whitehat reports, remediated on-chain as they come in; no audit-firm review yet; continuous monitoring |
| **Oracle Manipulation**     | Low         | High     | Hyperliquid mid + Chainlink fallback, circuit breakers |
| **Governance Attacks**      | Low         | High     | Vote-escrow system, time delays, caps           |

#### Economic Risks

| Risk Factor                | Probability | Impact | Mitigation Strategy                            |
| -------------------------- | ----------- | ------ | ---------------------------------------------- |
| **Token Price Volatility** | High        | Medium | Liquidity incentives (governance-decided after activation) |
| **Regulatory Changes**     | Medium      | High   | Legal compliance, jurisdiction flexibility     |
| **Competitive Pressure**   | High        | Medium | Innovation focus, ecosystem building           |
| **Market Downturns**       | High        | Medium | Treasury diversification, emission flexibility |

#### Operational Risks

| Risk Factor                 | Probability | Impact | Mitigation Strategy                          |
| --------------------------- | ----------- | ------ | -------------------------------------------- |
| **Team Dependency**         | Medium      | High   | Progressive decentralization, documentation  |
| **Key Person Risk**         | Low         | High   | Multi-sig controls, succession planning      |
| **Community Fragmentation** | Low         | Medium | Transparent governance, inclusive processes  |
| **Scalability Issues**      | Medium      | Medium | Base L2 deployment, efficiency improvements |

***

### Regulatory Compliance Framework

#### Legal Structure Optimization

**Foundation Setup (planned)**

* **Quantillon Foundation**: planned future entity for protocol governance - legal form and jurisdiction to be determined
* **Regulatory Classification**: QTI intended as a utility/governance token
* **Operational Flexibility**: Global team coordination

**Compliance Requirements**

**MiCA Regulation Alignment**

* **Stablecoin Reserves**: over-collateralized backing in USDC held on-chain, partly deployed in DeFi money markets
* **Reporting Standards**: Quarterly transparency reports
* **Consumer Protection**: Clear risk disclosures
* **Governance Standards**: Democratic decision-making processes

**Global Regulatory Considerations**

* **US Securities Law**: Utility token qualification via Howey test
* **EU Financial Regulations**: Compliance with 5AMLD/6AMLD
* **Asia-Pacific**: Alignment with Singapore/Hong Kong frameworks
* **Emerging Markets**: Flexible approach for new jurisdictions



***

### Long-term Sustainability Model

#### Ecosystem Development Strategy

**Partnership Network**

* **DeFi Protocols**: Deep integrations with major platforms
* **Traditional Finance**: Bridge to institutional adoption
* **Academic Institutions**: Research collaboration programs
* **Regulatory Bodies**: Proactive engagement and compliance

### Community Engagement Framework

#### Stakeholder Alignment

**Token Holder Benefits**

* **Revenue Sharing**: Direct participation in protocol success
* **Governance Rights**: Meaningful influence over protocol evolution
* **Early Access**: Priority access to new features and products
* **Educational Resources**: Comprehensive learning materials

**Developer Incentives**

* **Technical Grants**: Funding for ecosystem development
* **Revenue Sharing**: Percentage of fees from successful integrations
* **Recognition Programs**: Public acknowledgment and reputation building
* **Career Opportunities**: Direct hiring from contributor community

#### Communication Strategy

**Transparency Initiatives**

* **Monthly Reports**: Detailed progress and financial updates
* **Live AMAs**: Regular community Q\&A sessions
* **Development Blog**: Technical deep-dives and roadmap updates
* **Social Media**: Active engagement across multiple platforms

**Feedback Mechanisms**

* **Community Forums**: Dedicated discussion platforms
* **Proposal System**: Structured improvement suggestions
* **Beta Testing**: Early access programs for new features
* **Advisory Roles**: Community representation in core decisions

***

### Conclusion: Governance Above the First Deployment

The QTI tokenomics represent more than just another governance token: they constitute the economic and governance layer above Quantillon's first EUR deployment and future protocol evolution. Through careful balance of incentives, robust security measures, and progressive decentralization, QTI creates a sustainable foundation for the Quantillon ecosystem.

The approach prioritizes community ownership, regulatory compliance, and continuous innovation while maintaining flexibility to adapt to an evolving DeFi landscape. The result is a tokenomic model that aligns stakeholders around QEURO as the first deployment and Quantillon as the broader protocol.

As the roadmap progresses, QTI should evolve from an initial governance mechanism into the coordination layer for risk, incentives, and deployment policy across Quantillon. This is the beginning of a broader journey toward decentralized FX-hedged infrastructure for local-currency DeFi markets.

***

> **Disclaimer**: This document represents the current tokenomic design and may be subject to modifications based on community governance decisions, regulatory requirements, or technical considerations. All financial projections are estimates and should not be considered investment advice.
