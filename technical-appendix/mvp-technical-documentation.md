---
description: >-
  Comprehensive technical specification for Quantillon Protocol MVP
  implementation on Base mainnet
---

# MVP Technical Documentation

## MVP Technical Documentation

### Overview

This document outlines the technical architecture, smart contract specifications, and implementation roadmap for the Quantillon Protocol MVP. The MVP introduces Quantillon's revolutionary **three-token ecosystem**: QEURO (euro-pegged stablecoin), stQEURO (yield-bearing auto-compounding wrapper), and QTI (governance token with vote-escrow mechanics).

> **Three-Token MVP Scope**: Complete ecosystem implementation with QEURO minting/redemption, stQEURO auto-compounding mechanics, QTI governance foundations, and aQEURO vault with Aave v3 integration on Base mainnet.

***

### 🏗️ Global Technical Architecture

#### Revolutionary Three-Token Ecosystem

Quantillon Protocol operates through a sophisticated three-token architecture designed to create interconnected value flows where each token reinforces the utility and adoption of the others:

| Token          | Primary Function       | Key Innovation                                       | MVP Implementation                    |
| -------------- | ---------------------- | ---------------------------------------------------- | ------------------------------------- |
| **💶 QEURO**   | Euro-pegged stablecoin | Dual-pool architecture with embedded hedging         | Full mint/redeem + basic staking      |
| **📈 stQEURO** | Yield-bearing wrapper  | Auto-compounding without token inflation             | Complete auto-compounding system      |
| **🏛️ QTI**    | Governance backbone    | Vote-escrow mechanics + progressive decentralization | Basic governance + token distribution |

#### Recommended Tech Stack

| Component                  | Technology        | Version/Notes                               |
| -------------------------- | ----------------- | ------------------------------------------- |
| **Smart Contracts**        | Solidity          | 0.8.x with OpenZeppelin upgradeable proxies |
| **Development Framework**  | Foundry           | For development, testing, and deployment    |
| **Blockchain**             | Base Mainnet      | Production deployment (L2 efficiency)       |
| **Price Oracles**          | Chainlink         | EUR/USD and USDC/USD price feeds            |
| **Yield Integration**      | Aave v3           | USDC collateral deployment on Base          |
| **Frontend**               | React + Next.js   | Complete three-token interface              |
| **Backend/Infrastructure** | Minimal off-chain | Real-time yield calculation and analytics   |

#### Core Protocol Architecture

```
                    ┌─────────────────┐
                    │ Chainlink       │
                    │ Oracles         │
                    │ (EUR/USD,       │
                    │  USDC/USD)      │
                    └──────┬─────┬────┘
                           │     │
            ┌──────────────▼─────▼──────────────┐
            │         QEURO Contract           │
            │      (Euro Stablecoin)           │
            └──────┬───────────────────┬───────┘
                   │                   │
        ┌──────────▼───────────┐   ┌───▼───────────────┐
        │   stQEURO Contract   │   │   VaultEngine     │
        │ (Auto-Compounding)   │   │ (Collateral Mgmt) │
        └──────┬───────────────┘   └───┬───────────────┘
               │                       │
        ┌──────▼───────────┐   ┌───────▼───────────────┐
        │   QTI Contract   │   │   Aave v3 Pool        │
        │ (Governance)     │   │ (Yield Generation)    │
        └──────┬───────────┘   └───────────────────────┘
               │
        ┌──────▼───────────┐
        │  YieldController │
        │ (Distribution)   │
        └──────────────────┘

Ecosystem Flow:
1. Users deposit USDC → Mint QEURO (overcollateralized 101%+)
2. Users stake QEURO → Receive stQEURO (auto-compounding)
3. Collateral deployed to Aave v3 → Generates yield
4. YieldController distributes yield to stQEURO holders
5. QTI holders govern protocol parameters and treasury
6. Hedgers provide EUR/USD hedging (future full implementation)
```

**Component Responsibilities:**

* **QEURO Contract**: ERC20 stablecoin with mint/redeem logic, collateral ratio enforcement, and hedger integration
* **stQEURO Contract**: Yield-bearing wrapper with automatic compounding and instant liquidity
* **QTI Contract**: Governance token with vote-escrow mechanics and progressive decentralization
* **VaultEngine**: Manages USDC collateral deposits and Aave v3 integration for yield generation
* **YieldController**: Distributes yield between stQEURO holders and hedgers using Yield Shift mechanism
* **HedgerPool**: Delta-neutral EUR/USD hedging infrastructure (simplified MVP implementation)

***

### 📜 Smart Contract Specifications

#### QEURO (Euro Stablecoin)

**Primary Role**: Euro-pegged stablecoin with overcollateralized USDC backing and embedded hedging

**Core Functions:**

```solidity
interface IQEURO {
    function mint(address to, uint256 usdcAmount) external;
    function redeem(address to, uint256 qeuroAmount) external;
    function stake(uint256 amount) external returns (uint256 stQEURO);
    function collateralRatio() external view returns (uint256);
    function liquidate(address user) external;
    function getEURUSDRate() external view returns (uint256);
}
```

**Implementation Details:**

* **Minting**: Issues QEURO tokens when protocol collateralization ≥ 101% using real-time EUR/USD rates
* **Redemption**: Burns QEURO and releases equivalent USDC from vault (0.1% fee)
* **Staking Integration**: Direct staking to stQEURO with automatic yield allocation
* **Oracle Integration**: Chainlink EUR/USD and USDC/USD price feeds with fallback mechanisms
* **Fee Structure**: 0.1% mint/redeem fees supporting protocol sustainability

#### stQEURO (Yield-Bearing Euro Token)

**Primary Role**: Auto-compounding yield-bearing wrapper for QEURO with instant liquidity

**Core Functions:**

```solidity
interface IstQEURO {
    function stake(uint256 qeuroAmount) external returns (uint256 stQEURO);
    function unstake(uint256 stQEURO) external returns (uint256 qeuroAmount);
    function getExchangeRate() external view returns (uint256);
    function getTotalYield() external view returns (uint256);
    function compound() external; // Automatic via YieldController
}
```

**Auto-Compounding Mechanism:**

* **Value Appreciation**: stQEURO token quantity remains constant, but each token increases in QEURO value
* **Exchange Rate Formula**: `stQEURO Rate = 1 + (Cumulative Yield ÷ Total Days × Days Held)`
* **Yield Sources**: Aave USDC lending returns (4-12% APY) minus protocol fees and hedging costs
* **Distribution**: 87-89% of yield auto-compounded to stQEURO holders

**Example Timeline:**

* **Day 0**: Stake 1,000 QEURO → Receive 1,000 stQEURO (Rate: 1.000)
* **Day 365**: Still hold 1,000 stQEURO (Rate: 1.053 with 5.3% APY)
* **Unstaking**: 1,000 stQEURO → 1,053 QEURO

#### QTI (Governance Token)

**Primary Role**: Protocol governance with vote-escrow mechanics and progressive decentralization

**Core Functions:**

```solidity
interface IQTI {
    function lock(uint256 amount, uint256 duration) external returns (uint256 veQTI);
    function vote(uint256 proposalId, bool support, uint256 weight) external;
    function claimRewards() external returns (uint256);
    function createProposal(bytes calldata data) external returns (uint256);
}
```

**Governance Features:**

* **Vote-Escrow (veQTI)**: Lock QTI for voting power and yield sharing
* **Progressive Decentralization**: Gradual transition from Labs control to DAO governance
* **Parameter Control**: Collateral ratios, yield distribution, fee structures
* **Treasury Management**: Protocol revenue allocation and strategic decisions

**Token Distribution:**

* **60%**: Community (users, liquidity providers, ecosystem growth)
* **20%**: Team (long-term vesting with performance milestones)
* **20%**: Backers/Investors (strategic funding with lock-up periods)

#### VaultEngine (Enhanced Collateral Management)

**Primary Role**: Multi-vault collateral management with advanced yield optimization

**Core Functions:**

```solidity
interface IVaultEngine {
    function depositCollateral(uint256 amount, uint8 vaultType) external;
    function withdrawCollateral(uint256 amount, uint8 vaultType) external;
    function rebalance(uint8 fromVault, uint8 toVault, uint256 amount) external;
    function getVaultAPY(uint8 vaultType) external view returns (uint256);
    function getTotalTVL() external view returns (uint256);
}
```

**Vault Variants (MVP Foundation):**

* **aQEURO**: Aave v3 USDC integration (primary MVP implementation)
* **mQEURO**: MakerDAO integration (future Phase 2)
* **bQEURO**: Real World Assets integration (future Phase 2)
* **eQEURO**: ETH-based collateral variant (future Phase 3)

#### YieldController (Enhanced Distribution)

**Primary Role**: Sophisticated yield distribution with dynamic optimization

**Advanced Yield Distribution Logic:**

1. Calculate gross Aave yield from aUSDC positions
2. Deduct protocol fees (10% to treasury and development)
3. Calculate hedging costs (EUR/USD spread \~1% + variable Yield Shift)
4. Apply dynamic Yield Shift (target 0.5%) for supply/demand balancing
5. Distribute net yield:
   * **stQEURO Holders**: 87-89% (auto-compounded)
   * **Hedgers**: EUR/USD spread + Yield Shift allocation
   * **QTI Stakers**: Protocol fee sharing based on veQTI weight

```solidity
interface IYieldController {
    function calculateOptimalDistribution() external view returns (
        uint256 stQEUROYield,
        uint256 hedgerYield,
        uint256 qtiYield
    );
    function executeDistribution() external;
    function updateYieldShift(uint256 newShift) external onlyGovernance;
    function getYieldShift() external view returns (uint256);
}
```

***

### 🔗 External Integrations

#### Chainlink Price Oracles

**Required Price Feeds:**

* **EUR/USD**: Primary feed for QEURO peg maintenance and hedging calculations
* **USDC/USD**: Collateral valuation and protocol health monitoring
* **ETH/USD**: Future multi-collateral support and vault variants

**Security Measures:**

* **Dual Oracle Setup**: Primary Chainlink feed with secondary TWAP fallback
* **Circuit Breaker**: Automatic protocol pause during price feed failures
* **Anomaly Detection**: Price deviation alerts (>2% in 5 minutes) with manual intervention
* **Update Frequency**: 12-second maximum latency for price updates

#### Aave v3 Integration (Enhanced)

**Collateral Strategy:**

* Deploy USDC collateral to Aave v3 lending pool on Base
* Receive aUSDC tokens representing yield-bearing positions
* Monitor utilization rates and adjust deployment accordingly

**Risk Management:**

* Real-time Aave protocol health monitoring
* Emergency withdrawal procedures for liquidity crises
* Dynamic rebalancing based on utilization and yield optimization
* Integration with multiple DeFi protocols for diversification (future phases)

**Integration Flow:**

1. User deposits USDC for QEURO minting or stQEURO staking
2. VaultEngine optimally deploys collateral to Aave v3 pool
3. Protocol receives aUSDC representing yield-bearing position
4. YieldController tracks yield generation and executes distribution
5. Automatic rebalancing handles withdrawals and optimization

***

### 💻 Frontend DApp Specification

#### Three-Token Interface (Complete MVP)

**QEURO Panel:**

* **Mint Interface**: USDC → QEURO conversion with real-time EUR/USD rates
* **Redeem Interface**: QEURO → USDC conversion with fee transparency
* **Balance Display**: Real-time QEURO balance and USD/EUR equivalent values
* **Transaction History**: Complete mint/redeem operation tracking

**stQEURO Panel:**

* **Stake Interface**: QEURO → stQEURO with projected APY calculations
* **Unstake Interface**: stQEURO → QEURO with earned yield display
* **Yield Tracking**: Real-time APY, total earned, and compounding visualization
* **Performance Analytics**: Historical yield performance and projections

**QTI Panel:**

* **Governance Dashboard**: Active proposals, voting history, and participation metrics
* **Lock Interface**: QTI → veQTI with duration selection and voting power calculation
* **Rewards Claiming**: Protocol fee sharing and governance participation rewards
* **Treasury Overview**: Protocol revenue, TVL, and treasury composition

**Protocol Dashboard:**

* **Three-Token Overview**: Complete ecosystem metrics and health indicators
* **Collateral Ratio**: Real-time protocol health across all vaults
* **Total Value Locked (TVL)**: Aggregate statistics across QEURO, stQEURO, and vaults
* **Yield Performance**: Current Aave APY, stQEURO returns, and QTI rewards
* **Market Data**: EUR/USD rates, trading volumes, and liquidity metrics

**Technical Implementation:**

```typescript
interface QuantillonProtocol {
  // QEURO Operations
  mintQEURO: (usdcAmount: BigNumber) => Promise<TransactionResponse>;
  redeemUSDC: (qeuroAmount: BigNumber) => Promise<TransactionResponse>;
  
  // stQEURO Operations
  stakeQEURO: (qeuroAmount: BigNumber) => Promise<TransactionResponse>;
  unstakeQEURO: (stQEUROAmount: BigNumber) => Promise<TransactionResponse>;
  
  // QTI Operations
  lockQTI: (amount: BigNumber, duration: number) => Promise<TransactionResponse>;
  vote: (proposalId: string, support: boolean) => Promise<TransactionResponse>;
  
  // Protocol Data
  getProtocolMetrics: () => Promise<ProtocolMetrics>;
  getUserPortfolio: (address: string) => Promise<UserPortfolio>;
}
```

***

### 🛡️ Security & Testing Strategy

#### Comprehensive Testing Framework

**Unit Tests (Foundry)**

* **Three-Token Interactions**: Cross-token functionality and value flows
* **Auto-Compounding Logic**: stQEURO yield calculation and distribution accuracy
* **Governance Mechanisms**: QTI voting, proposals, and parameter changes
* **Collateral Management**: Multi-vault operations and rebalancing
* **Oracle Integration**: Price feed handling and fallback mechanisms
* **Emergency Procedures**: Circuit breakers and protocol pause functionality

**Integration Tests**

* **End-to-End Workflows**: Complete user journeys across all three tokens
* **Aave Integration**: Mock yield generation and withdrawal scenarios
* **Oracle Manipulation**: Price feed attack simulations and recovery
* **Stress Testing**: Mass operations, simultaneous redemptions, and high-load scenarios
* **Multi-Contract Coordination**: Complex interactions between all protocol components

**Security Testing Scenarios:**

```solidity
contract SecurityTests {
    function testReentrancyProtection() external;
    function testFlashLoanAttacks() external;
    function testGovernanceAttacks() external;
    function testYieldManipulation() external;
    function testCrossTokenExploits() external;
    function testOracleManipulation() external;
}
```

#### Advanced Security Measures

**Smart Contract Security:**

* **Multi-Signature Protection**: Critical functions require multiple signatures
* **Time-Locked Operations**: Governance proposals have mandatory delay periods
* **Emergency Procedures**: Protocol-wide pause capability with rapid response
* **Formal Verification**: Mathematical proofs for critical economic mechanisms
* **Bug Bounty Program**: Incentivized community security testing (post-MVP)

**Operational Security:**

* **Progressive Decentralization**: Gradual transition from Labs to DAO control
* **Treasury Protection**: Multi-signature wallets with geographic distribution
* **Upgrade Safety**: Comprehensive testing and community approval for upgrades
* **Incident Response**: Detailed emergency procedures and communication protocols

***

### 🚀 Deployment & Infrastructure

#### Multi-Chain Strategy

**Primary Deployment:**

* **Base Mainnet**: Primary deployment for L2 efficiency and ecosystem integration
* **Ethereum Mainnet**: Future cross-chain bridge and institutional access
* **Additional L2s**: Arbitrum and Optimism expansion (Phase 2)

**Environment Configuration:**

**Development Environment:**

* **Local Testing**: Anvil (Foundry) with complete three-token simulation
* **Contract Development**: Solidity 0.8.x with comprehensive test coverage
* **Integration Testing**: Mock oracles, Aave, and cross-contract interactions

**Production Environment:**

* **Base Mainnet**: Complete three-token ecosystem deployment
* **Cross-Chain Infrastructure**: Future multi-chain synchronization
* **Monitoring**: Real-time protocol health, yield tracking, and governance activity

#### CI/CD Pipeline (Enhanced)

**Automated Deployment:**

```yaml
name: Quantillon Three-Token Protocol Deployment
on:
  push:
    branches: [main, develop]
  
jobs:
  comprehensive-testing:
    runs-on: ubuntu-latest
    steps:
      - name: Three-Token Integration Tests
        run: forge test --match-path "test/integration/*" --gas-report
      
      - name: Cross-Contract Security Analysis
        run: slither . --exclude-dependencies --check-upgradeability
      
      - name: Governance Simulation
        run: forge test --match-path "test/governance/*" --fork-url $BASE_RPC
      
      - name: Deployment with Verification
        run: forge script DeployThreeTokens --broadcast --verify
```

#### Gas Optimization Strategies

**Multi-Token Efficiency:**

* **Batch Operations**: Combined mint/stake operations for gas savings
* **Proxy Patterns**: Upgradeable contracts minimizing deployment costs
* **Storage Optimization**: Packed structs for multi-token user data
* **View Function Optimization**: Efficient read operations across tokens

**Transaction Cost Management:**

* **QEURO Operations**: <60,000 gas for mint/redeem
* **stQEURO Operations**: <80,000 gas for stake/unstake with compounding
* **QTI Operations**: <100,000 gas for voting and governance participation
* **Yield Distribution**: Optimized batch processing with automated triggers

***

### 📋 MVP Development Roadmap

#### Phase 1: Core Three-Token Development (Q4 2025)

**Smart Contract Implementation:**

* \[ ] QEURO ERC20 with mint/redeem and oracle integration
* \[ ] stQEURO auto-compounding wrapper with yield calculation
* \[ ] QTI governance token with vote-escrow mechanics
* \[ ] VaultEngine with Aave v3 integration and multi-vault foundation
* \[ ] YieldController with sophisticated distribution logic
* \[ ] HedgerPool stub for future EUR/USD hedging expansion

**Cross-Token Integration:**

* \[ ] QEURO ↔ stQEURO seamless staking/unstaking
* \[ ] QTI governance parameter control over yield distribution
* \[ ] Unified fee collection and distribution across three tokens
* \[ ] Cross-contract security and access control implementation

**External Integrations:**

* \[ ] Chainlink EUR/USD and USDC/USD oracle connections
* \[ ] Aave v3 lending pool integration with yield optimization
* \[ ] Emergency procedures and circuit breaker implementations
* \[ ] Multi-signature governance setup for progressive decentralization

#### Phase 2: Frontend & User Experience (Q4 2025)

**Complete Three-Token Interface:**

* \[ ] QEURO mint/redeem interface with real-time EUR/USD rates
* \[ ] stQEURO staking interface with yield visualization and auto-compounding
* \[ ] QTI governance dashboard with proposal creation and voting
* \[ ] Unified protocol dashboard with comprehensive analytics
* \[ ] Wallet integration supporting multi-token operations

**Advanced Features:**

* \[ ] Mobile-optimized interface for smartphone-native experience
* \[ ] Portfolio tracking across all three tokens with performance analytics
* \[ ] Educational resources and guided onboarding for new users
* \[ ] Advanced trading features and yield optimization recommendations

#### Phase 3: Security & Compliance (Q1-Q2 2026)

**Comprehensive Security Audit:**

* \[ ] Multi-contract security audit by leading firms (OpenZeppelin, ConsenSys)
* \[ ] Cross-token attack vector analysis and mitigation
* \[ ] Formal verification of critical economic mechanisms
* \[ ] Bug bounty program with specialized focus on three-token interactions

**Regulatory Compliance:**

* \[ ] MiCA compliance verification across all three tokens
* \[ ] Legal classification confirmation for QEURO, stQEURO, and QTI
* \[ ] Institutional compliance documentation and procedures
* \[ ] Regulatory reporting framework and transparency measures

#### Phase 4: Mainnet Launch & Ecosystem Growth (Q3 2026)

**Production Deployment:**

* \[ ] Complete three-token ecosystem launch on Base mainnet
* \[ ] Initial liquidity provision and market making across all tokens
* \[ ] DeFi protocol integrations supporting all three tokens
* \[ ] Institutional onboarding and custody provider partnerships

**Community & Governance:**

* \[ ] DAO governance activation with QTI token distribution
* \[ ] Community grant programs and ecosystem development initiatives
* \[ ] Cross-chain expansion planning and implementation
* \[ ] Long-term sustainability and decentralization roadmap

***

### 🎯 Success Metrics & KPIs

#### Three-Token Ecosystem Health

**Stability & Security Metrics:**

* **QEURO Peg Stability**: EUR/QEURO variance <0.5% across all market conditions
* **stQEURO Yield Consistency**: <5% monthly deviation from target APY
* **QTI Governance Participation**: >15% of circulating supply actively voting
* **Protocol Security**: Zero critical vulnerabilities, >99.9% uptime

**Growth & Adoption Metrics:**

* **Total Value Locked (TVL)**: €50M across all three tokens within 12 months
* **User Distribution**: 15,000 QEURO users, 8,000 stQEURO stakers, 3,000 QTI governors
* **Daily Active Users**: 1,000+ daily interactions across ecosystem
* **Cross-Token Utilization**: >60% of users holding multiple tokens

**Economic Performance:**

* **stQEURO APY**: Competitive 5-8% yield with consistent compounding
* **Protocol Revenue**: €200K+ monthly from fees and yield management
* **QTI Token Value**: Sustainable growth through utility and governance demand
* **Yield Distribution Efficiency**: >95% accuracy in automated compounding

#### Technical Performance

**System Reliability:**

* **Multi-Token Uptime**: 99.9% availability across all three contracts
* **Transaction Success Rate**: >99% successful operations
* **Oracle Responsiveness**: <30 second price update latency
* **Cross-Contract Coordination**: Seamless multi-token operations

**User Experience:**

* **Transaction Costs**: <$10 average cost for complex multi-token operations
* **Interface Performance**: <3 second load times for dashboard
* **Mobile Optimization**: 90%+ mobile user satisfaction score
* **Educational Effectiveness**: >80% user onboarding completion rate

***

### 🌟 Innovation Highlights

#### Revolutionary Three-Token Design

**Interconnected Value Creation:**

* **QEURO**: Provides euro exposure with USD liquidity depth
* **stQEURO**: Offers set-and-forget euro yield generation with instant liquidity
* **QTI**: Enables sophisticated governance and protocol value capture

**Technical Innovations:**

* **Auto-Compounding Without Inflation**: stQEURO increases value without token rebasing
* **Embedded DEX**: Native token swapping eliminating external dependencies
* **Yield Shift Mechanism**: Dynamic yield distribution balancing supply and demand
* **Progressive Decentralization**: Gradual transition from centralized launch to DAO governance

**Market Advantages:**

* **First Euro DeFi Ecosystem**: Complete three-token architecture for European market
* **Institutional Ready**: MiCA-compliant design with professional-grade features
* **Capital Efficient**: Optimal collateral utilization across multiple DeFi protocols
* **Community Driven**: Governance-first approach with sustainable tokenomics

***

_This MVP Technical Documentation serves as the comprehensive blueprint for Quantillon Protocol's revolutionary three-token ecosystem. For questions or technical clarification, please refer to the complete protocol whitepaper or contact the development team._
