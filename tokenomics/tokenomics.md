# Tokenomics

## QTI Tokenomics

### Executive Summary

Quantillon is a decentralized protocol for issuing Euro-backed stablecoins (QEURO) backed by USDC reserves, featuring community governance through the $QTI token. The $QTI token aligns the interests of users, hedgers, developers, and investors by orchestrating issuance, governance, and value capture within the protocol. Its design prioritizes sustainability, security, and progressive adoption of decentralized governance.

### Token Specifications

| Parameter          | Value                                    |
| ------------------ | ---------------------------------------- |
| **Name**           | Quantillon Governance Token              |
| **Symbol**         | QTI                                      |
| **Token Standard** | ERC-20 (Upgradeable via UUPS Proxy)      |
| **Blockchain**     | Ethereum Layer2 : Base                   |
| **Total Supply**   | 100,000,000 QTI (fixed cap)              |
| **Decimals**       | 18                                       |
| **Divisibility**   | Fully divisible (10^18 units)            |
| **Smart Contract** | OpenZeppelin-based, upgradable & audited |

### Distribution & Vesting

#### Initial Allocation

| Category                    | Allocation (%) | QTI Amount | Vesting Details                                                              |
| --------------------------- | -------------- | ---------- | ---------------------------------------------------------------------------- |
| **Community & Ecosystem**   | 50%            | 50,000,000 | Incentives, retroactive rewards, liquidity mining, contributor airdrops (4y) |
| **Team & Founders**         | 15%            | 15,000,000 | 1-year cliff, 3-year linear vesting                                          |
| **Investors (SAFT/BSA)**    | 13%            | 13,000,000 | Custom vesting per round, typically 6-12M lock, 2-3y vesting                 |
| **DAO Treasury Reserve**    | 10%            | 10,000,000 | DAO-controlled multisig, subject to vote                                     |
| **Strategic Partners**      | 5%             | 5,000,000  | Exchanges, auditors, launchpads, integrations (6M lock, 1.5y vest)           |
| **Advisors**                | 2%             | 2,000,000  | 6M cliff, 18-month linear vesting                                            |
| **Liquidity Bootstrapping** | 5%             | 5,000,000  | Seeded into DEX pools (QTI/ETH, QTI/QEURO)                                   |

#### Rationale

* **Large community & ecosystem allocation** (50%) ensures maximum user and builder engagement
* **Reduced team allocation** (15%) demonstrates commitment to community-first approach
* **Strategic partnerships** enable key integrations and institutional adoption
* **DAO Treasury** provides long-term sustainability and community control

### Utility & Economics

#### 🔧 Token Utility

* **DAO Governance** - Vote on protocol upgrades, treasury allocation, and yield rates
* **veQTI System** - Lock QTI up to 4 years to get veQTI for enhanced voting power
* **Protocol Rewards** - Earn from QEURO minting fees, liquidations, and yield
* **Vault Backstopping** - Optional module for systemic debt protection

#### 💸 Value Accrual Mechanisms

* **Protocol Fee Collection**: Percentage fees on QEURO mint/redeem, yield-bearing vaults, and DEX swaps
* **Fee Sharing**: 50% of collected fees redistributed to staked QTI holders
* **Buybacks**: Treasury can repurchase QTI from the market during surplus periods
* **Burns**: Optional via governance (penalty-based burns or deflationary initiatives)

### Economic Incentives & Mechanisms

#### 🚰 Liquidity Mining

Targeting QEURO pairs on major DEXs:

* Curve Finance
* Uniswap v4
* Aerodrome (Base)

Rewards distributed in QTI for LPs based on duration and impermanent loss risk.

#### 📈 User Incentives

* **Gas rebates** (up to 50%) for QEURO minters or DEX swappers
* **Airdrop campaigns** for on-chain participation and protocol usage

#### 🛠 Developer Grants

1–5% per year of total supply allocated via DAO for:

* dApps integrating QEURO
* Auditing & infrastructure tools
* Oracle & analytics integrations

#### 🔄 Sustainability Measures

* **Emission Decay**: Ecosystem rewards reduce \~15% year-over-year
* **Locked Staking**: Boost rewards for longer lockups (veQTI model optional)

#### 🛡 Anti-Dilution Protections

* All non-circulating QTI (e.g., Treasury) excluded from governance quorum
* Optional "vote-escrow" system to weight votes based on lock duration

### Governance Model

#### 🗳 Voting Power Distribution

* **1 QTI = 1 vote** (quadratic + veQTI options under consideration)
* **Delegation enabled** via Snapshot & Tally integration

#### 🏛 Proposal Lifecycle

| Step                   | Requirement                            |
| ---------------------- | -------------------------------------- |
| **Submission**         | 100,000 QTI or via delegate support    |
| **Voting Period**      | 5–7 days                               |
| **Quorum**             | 4% of circulating supply               |
| **Approval Threshold** | 60% yes votes                          |
| **Timelock**           | 2 days (for execution via Gnosis Safe) |
| **Emergency Pause**    | 3-of-5 multisig (rotated quarterly)    |

### Risk Management & Security

#### 🧱 Security Measures

* **Multiple Audits**: All contracts undergo 2+ external audits before mainnet launch
* **Bug Bounty Program**: Up to $250K rewards for critical vulnerabilities

#### 🐳 Whale Protections

* **Voting Cap**: Maximum 2.5% of circulating QTI per address
* **Anti-Sybil Measures**: Proof-of-participation + veQTI requirements

#### 🛑 Circuit Breakers

* **Emergency Pause**: For QEURO mint/redemptions during critical events
* **Slashing Mechanism**: For misbehaving validators (future implementation)

#### 🔐 Treasury Controls

* **Multi-sig Security**: 4/6 core contributors + 1 community representative
* **Progressive Decentralization**: Migration to DAO-controlled Safe + on-chain voting

### Implementation Timeline

| Phase                   | Timeline   | Key Milestones                                  |
| ----------------------- | ---------- | ----------------------------------------------- |
| **Phase 1 – Testnet**   | Q3 2025    | QTI deployment, mock QEURO + governance testing |
| **Phase 2 – Genesis**   | Q4 2025    | LBP launch + airdrop + DAO bootstrapping        |
| **Phase 3 – Growth**    | Q1–Q2 2026 | Full QEURO launch, liquidity mining programs    |
| **Phase 4 – Expansion** | Q3 2026+   | Cross-chain bridges, validator network rollout  |

### Risk Assessment

| Risk                               | Mitigation Strategy                                    |
| ---------------------------------- | ------------------------------------------------------ |
| **Token Price Manipulation**       | Liquidity depth, strategic buybacks, anti-whale voting |
| **Governance Capture**             | Delegation systems + time-locks + community multisig   |
| **Regulatory Scrutiny**            | USDC-backed stablecoin, clear DAO legal separation     |
| **Smart Contract Vulnerabilities** | Multiple audits + bug bounties + upgradability         |
| **Liquidity Drain (Bear Market)**  | Emission decay + flexible reward curves                |

### Financial Projections (2026–2029)

_Conservative case estimates_

| Metric                      | Year 1 (2026) | Year 2 (2027) | Year 3 (2028) | Year 4 (2029) |
| --------------------------- | ------------- | ------------- | ------------- | ------------- |
| **QEURO Supply**            | €20M          | €80M          | €150M         | €250M         |
| **Protocol Revenue**        | €300K         | €2M           | €4.2M         | €7M           |
| **Staking APY (base case)** | 12%           | 9%            | 7%            | 6%            |
| **Treasury Assets**         | €1.5M         | €6M           | €12M          | €20M          |

_Assumptions: 0.5–1% protocol fee with 50% revenue distribution to stakers_

### Legal & Regulatory Framework

#### 🔖 Key Recommendations

* **Non-Security Classification**: Ensure QTI qualifies as utility token through governance and staking functionality
* **Entity Separation**: Maintain clear separation between QEURO issuance entity and governance DAO
* **Regulatory Compliance**: Adherence to MiCA and ESMA stablecoin frameworks



***

> **Note**: This tokenomics design prioritizes long-term sustainability, security, and progressive decentralization while ensuring regulatory compliance and community alignment.
