---
description: >-
  Comprehensive technical specification for Quantillon Protocol MVP
  implementation on Base mainnet
---

# MVP Technical Documentation

## MVP Technical Documentation

### Overview

This document outlines the technical architecture, smart contract specifications, and implementation roadmap for the Quantillon Protocol MVP. The MVP focuses on core stablecoin functionality with QEURO minting/redemption, yield-generating collateral management, and foundational hedging infrastructure.

> **MVP Scope**: Single vault implementation (aQEURO) with USDC collateral, Aave v3 integration, and basic frontend for mint/redeem operations on Base mainnet.

***

### 🏗️ Global Technical Architecture

#### Recommended Tech Stack

| Component                  | Technology        | Version/Notes                               |
| -------------------------- | ----------------- | ------------------------------------------- |
| **Smart Contracts**        | Solidity          | 0.8.x with OpenZeppelin upgradeable proxies |
| **Development Framework**  | Foundry           | For development, testing, and deployment    |
| **Blockchain**             | Base Mainnet      | Production deployment (no testnet required) |
| **Price Oracles**          | Chainlink         | EUR/USD and USDC/USD price feeds            |
| **Yield Integration**      | Aave v3           | USDC collateral deployment on Base          |
| **Frontend**               | React + Next.js   | Lightweight MVP user interface              |
| **Backend/Infrastructure** | Minimal off-chain | Analytics and monitoring only               |

#### Core On-Chain Components

```
                    ┌─────────────────┐
                    │ Chainlink       │
                    │ Oracles         │
                    │ (EUR/USD,       │
                    │  USDC/USD)      │
                    └──────┬─────┬────┘
                           │     │
            ┌──────────────▼─────▼──────────────┐
            │         QEURO Contract            │
            │      (ERC20 Stablecoin)           │
            └──────────────┬────────────────-───┘
                           │
            ┌──────────────▼───────────────────┐
            │         VaultEngine              │
            │    (Collateral Management)       │
            └──────┬───────────────────┬───────┘
                   │                   │
        ┌──────────▼───────────┐   ┌───▼───────────────┐
        │   YieldController    │   │   Aave v3 Pool    │
        │ (Yield Distribution) │   │ (Yield Generation)│
        └──────┬───────────────┘   └───────────────────┘
               │
        ┌──────▼───────────┐
        │   HedgerPool     │
        │ (Margin & Risk)  │
        └──────────────────┘

Data Flow:
1. Chainlink Oracles → Price feeds to QEURO & YieldController
2. QEURO Contract → Mint/redeem operations via VaultEngine
3. VaultEngine → Deploys USDC to Aave v3 for yield generation
4. VaultEngine → Sends yield data to YieldController
5. YieldController → Distributes yield between Users & Hedgers
6. YieldController → Manages HedgerPool incentives
```

**Component Responsibilities:**

* **QEURO Contract**: ERC20 stablecoin with mint/redeem logic and collateral ratio enforcement
* **VaultEngine**: Manages USDC collateral deposits and Aave v3 integration for yield generation
* **YieldController**: Distributes yield between Users and Hedgers according to Yield Shift mechanism
* **HedgerPool**: Manages hedger collateral, margin requirements, and liquidation states (MVP placeholder)
* **Governance**: Future $QTI DAO contracts (not included in MVP)

#### Frontend Architecture

**User Application (MVP Scope)**

* Mint QEURO against USDC collateral
* Redeem USDC by burning QEURO
* Real-time display of collateral ratio, vault TVL, and staking APR
* Oracle price feeds integration for EUR/USD rates

**Future Extensions**

* Hedger dashboard for position management and margin monitoring
* DAO governance interface for proposals and voting
* Advanced analytics and portfolio management tools

***

### 📜 Smart Contract Specifications

#### QEURO (ERC20 Stablecoin)

**Primary Role**: Euro-pegged stablecoin issuance and redemption with collateral backing

**Core Functions:**

```solidity
interface IQEURO {
    function mint(address to, uint256 amount) external;
    function redeem(address to, uint256 amount) external;
    function collateralRatio() external view returns (uint256);
    function liquidate(address user) external;
}
```

**Implementation Details:**

* **Minting**: Issues QEURO tokens when protocol collateralization ≥ 101%
* **Redemption**: Burns QEURO and releases equivalent USDC from vault
* **Collateral Monitoring**: Real-time tracking of protocol-wide collateral ratio
* **Liquidation**: Automated liquidation trigger when collateral falls below threshold

#### VaultEngine

**Primary Role**: Collateral management and yield generation through Aave v3 integration

**Core Functions:**

```solidity
interface IVaultEngine {
    function depositCollateral(uint256 amount) external;
    function withdrawCollateral(uint256 amount) external;
    function integrateAave() external;
    function getVaultAPY() external view returns (uint256);
    function rebalance() external;
}
```

**Implementation Details:**

* **Collateral Operations**: Secure USDC deposit/withdrawal with access controls
* **Aave Integration**: Automatic deployment of collateral to generate aUSDC yield
* **APY Tracking**: Real-time monitoring of Aave lending rates
* **Rebalancing**: Handles collateral withdrawals for redemption requests

#### YieldController

**Primary Role**: Yield distribution between Users and Hedgers using dynamic Yield Shift mechanism

**Yield Distribution Logic:**

1. Calculate gross Aave yield from aUSDC positions
2. Deduct hedging costs (estimated \~1% EUR/USD spread)
3. Apply variable Yield Shift (target 0.5%) for supply/demand balancing
4. Distribute net yield:
   * **Users**: Base staking APY + dynamic performance boost
   * **Hedgers**: FX spread compensation + Yield Shift allocation

```solidity
interface IYieldController {
    function calculateYieldDistribution() external view returns (uint256 usersYield, uint256 hedgersYield);
    function updateYieldShift(uint256 newShift) external;
    function distributeYield() external;
}
```

#### HedgerPool (MVP Simplified)

**Primary Role**: Hedger margin management and liquidation infrastructure (placeholder implementation)

**Core Functions:**

```solidity
interface IHedgerPool {
    function registerHedger(address hedger, uint256 margin) external;
    function getMargin(address hedger) external view returns (uint256);
    function liquidateHedger(address hedger) external;
}
```

**MVP Limitations:**

* Simplified margin tracking without live forex integration
* Basic liquidation logic (margin < 1% threshold)
* No automated hedging position management

#### Governance ($QTI) - Future Implementation

**Scope**: DAO-based parameter management and protocol governance (excluded from MVP)

**Future Capabilities:**

* Collateral ratio threshold adjustments
* Yield Shift parameter tuning
* Protocol upgrade approvals
* $QTI token distribution (60% users, 20% team, 20% backers)

***

### 🔗 External Integrations

#### Chainlink Price Oracles

**Required Price Feeds:**

* **EUR/USD**: Primary feed for hedging calculations and peg maintenance
* **USDC/USD**: Collateral valuation and protocol health monitoring

**Security Measures:**

* **Dual Oracle Setup**: Primary Chainlink feed with secondary TWAP fallback
* **Circuit Breaker**: Automatic protocol pause during price feed failures
* **Anomaly Detection**: Price deviation alerts and manual intervention capabilities

**Implementation:**

```solidity
interface IPriceOracle {
    function getEURUSDPrice() external view returns (int256 price, uint256 timestamp);
    function getUSDCUSDPrice() external view returns (int256 price, uint256 timestamp);
    function validatePriceFeeds() external view returns (bool isValid);
}
```

#### Aave v3 Integration

**Collateral Strategy:**

* Deploy USDC collateral to Aave v3 lending pool on Base
* Receive aUSDC tokens representing yield-bearing positions
* Maintain liquidity buffer for redemption requests

**Risk Management:**

* Monitor Aave protocol health and utilization rates
* Implement emergency withdrawal procedures
* Track aUSDC/USDC exchange rate variations

**Integration Flow:**

1. User deposits USDC for QEURO minting
2. VaultEngine deposits USDC to Aave v3 pool
3. Protocol receives aUSDC representing yield-bearing position
4. YieldController tracks and distributes generated yield
5. Rebalancing handles withdrawals for redemption requests

***

### 💻 Frontend DApp Specification

#### User Interface (Phase 1 - MVP)

**Mint Panel:**

* **Input**: USDC amount with wallet balance display
* **Output**: QEURO amount calculated using real-time EUR/USD oracle rates
* **Fee Display**: 0.1% minting fee transparency
* **Transaction Preview**: Gas estimation and confirmation details

**Redeem Panel:**

* **Input**: QEURO amount with current balance
* **Output**: USDC amount after 0.1% redemption fee
* **Availability Check**: Ensure sufficient vault liquidity
* **Processing Time**: Real-time transaction status updates

**Protocol Dashboard:**

* **Collateral Ratio**: Live protocol health indicator
* **Total Value Locked (TVL)**: Aggregate vault statistics
* **Current Aave APY**: Real-time yield generation rates
* **User Staking Rewards**: Personalized yield tracking

**Technical Implementation:**

```typescript
interface MintRedeemInterface {
  mintQEURO: (usdcAmount: BigNumber) => Promise<TransactionResponse>;
  redeemUSDC: (qeuroAmount: BigNumber) => Promise<TransactionResponse>;
  getCollateralRatio: () => Promise<BigNumber>;
  getProtocolTVL: () => Promise<BigNumber>;
  getUserRewards: (address: string) => Promise<BigNumber>;
}
```

#### Future Interface Extensions

**Hedger Dashboard (Phase 2):**

* Margin registration and management
* Real-time PnL tracking
* Liquidation risk monitoring
* Incentive rewards claiming

**Governance Interface (Phase 3):**

* DAO proposal creation and voting
* Parameter adjustment proposals
* Treasury management oversight
* Community governance participation

***

### 🛡️ Security & Testing Strategy

#### Comprehensive Testing Framework

**Unit Tests (Foundry)**

* **Mint/Redeem Operations**: Normal and edge case scenarios
* **Collateral Ratio Enforcement**: Threshold compliance testing
* **Liquidation Mechanisms**: Automated liquidation triggers
* **Yield Calculations**: Accurate distribution logic verification
* **Access Controls**: Role-based permission testing

**Integration Tests**

* **Oracle Manipulation**: Price feed attack simulations
* **Aave Integration**: Mock aUSDC yield generation testing
* **Stress Testing**: Mass redemption and oracle downtime scenarios
* **Multi-contract Interactions**: End-to-end workflow validation

**Security Testing Scenarios:**

```solidity
contract SecurityTests {
    function testReentrancyProtection() external;
    function testOracleManipulation() external;
    function testFlashLoanAttacks() external;
    function testCollateralDraining() external;
    function testGovernanceAttacks() external;
}
```

#### Security Best Practices

**Smart Contract Security:**

* **Reentrancy Guards**: OpenZeppelin ReentrancyGuard implementation
* **Circuit Breakers**: Automated pause mechanisms for oracle anomalies
* **Access Control**: Role-based permissions with multi-signature requirements
* **Upgrade Safety**: UUPS proxy pattern with governance-controlled upgrades

**Operational Security:**

* **Multi-signature Wallets**: Critical function protection
* **Time-locked Operations**: Governance proposal execution delays
* **Emergency Procedures**: Protocol pause and recovery mechanisms
* **Regular Audits**: Continuous security assessment and improvement

***

### 🚀 Deployment & Infrastructure

#### Environment Configuration

**Development Environment:**

* **Local Testing**: Anvil (Foundry) for rapid iteration
* **Contract Development**: Solidity 0.8.x with comprehensive test coverage
* **Integration Testing**: Mock oracle and Aave contracts

**Staging Environment:**

* **Testnet Deployment**: Sepolia for frontend integration testing
* **End-to-end Testing**: Complete user workflow validation
* **Performance Testing**: Gas optimization and transaction throughput

**Production Environment:**

* **Base Mainnet**: Primary deployment target
* **No Testnet Requirements**: Direct mainnet deployment strategy
* **Monitoring**: Comprehensive protocol health tracking

#### CI/CD Pipeline

**Automated Deployment:**

```yaml
name: Quantillon Protocol Deployment
on:
  push:
    branches: [main]
  
jobs:
  test-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Smart Contract Testing
        run: forge test --gas-report
      
      - name: Security Analysis
        run: slither . --exclude-dependencies
      
      - name: Deployment
        run: forge script Deploy --broadcast --verify
```

**Infrastructure Components:**

* **GitHub Actions**: Automated build, test, and deployment pipeline
* **Foundry Scripts**: Deterministic deployment and configuration
* **Tenderly/Defender**: Real-time monitoring and alerting system

#### Gas Optimization Strategies

**Storage Optimization:**

* Efficient storage layout design
* Packed struct implementations
* Minimal state variable usage

**Execution Optimization:**

* Batch oracle reads for multiple operations
* Optimized loop structures and conditionals
* Strategic use of view functions for read operations

**Transaction Cost Management:**

* **Mint/Redeem**: Minimize state writes in critical user paths
* **Yield Distribution**: Batch processing for efficiency
* **Liquidations**: Gas-optimized emergency procedures

#### Upgradeability Framework

**Proxy Implementation:**

* **UUPS Pattern**: Universal Upgradeable Proxy Standard (EIP-1967)
* **Governance Control**: Future DAO-managed upgrade approvals
* **Security Measures**: Implementation validation and testing requirements

**Upgrade Process:**

1. **Proposal Creation**: Technical specification and impact analysis
2. **Community Review**: Stakeholder feedback and security assessment
3. **Testing Phase**: Comprehensive validation on staging environment
4. **Governance Vote**: DAO approval with required quorum
5. **Implementation**: Controlled upgrade execution with monitoring

***

### 📋 MVP Development Checklist

#### Phase 1: Core Protocol Development

**Smart Contract Implementation:**

* \[ ] QEURO ERC20 with mint/redeem functionality
* \[ ] VaultEngine with Aave v3 integration
* \[ ] YieldController with basic yield distribution
* \[ ] HedgerPool stub contract for future expansion
* \[ ] Oracle integration with Chainlink price feeds

**Integration Development:**

* \[ ] Chainlink EUR/USD and USDC/USD oracle connections
* \[ ] Aave v3 lending pool integration on Base
* \[ ] Circuit breaker implementation for oracle failures
* \[ ] Emergency pause mechanisms

**Security Implementation:**

* \[ ] Comprehensive unit test suite (>95% coverage)
* \[ ] Integration tests with mock external contracts
* \[ ] Reentrancy protection and access controls
* \[ ] Multi-signature wallet setup for critical functions

#### Phase 2: Frontend Development

**User Interface:**

* \[ ] Mint/redeem interface with real-time rate calculations
* \[ ] Protocol dashboard with TVL, collateral ratio, and APY
* \[ ] Wallet integration (MetaMask, WalletConnect, Coinbase Wallet)
* \[ ] Transaction history and user portfolio tracking

**Technical Integration:**

* \[ ] Smart contract interaction layer
* \[ ] Real-time oracle price feed integration
* \[ ] Gas estimation and transaction optimization
* \[ ] Error handling and user feedback systems

#### Phase 3: Testing & Security

**Security Audit:**

* \[ ] Internal security review and testing
* \[ ] External audit by reputable security firm
* \[ ] Bug bounty program launch (post-MVP)
* \[ ] Continuous monitoring setup

**Performance Testing:**

* \[ ] Load testing for high transaction volumes
* \[ ] Gas optimization validation
* \[ ] Oracle failure recovery testing
* \[ ] User experience optimization

#### Phase 4: Deployment

**Production Deployment:**

* \[ ] Base mainnet deployment scripts
* \[ ] Contract verification on Basescan
* \[ ] Frontend deployment and CDN configuration
* \[ ] Monitoring and alerting system activation

**Go-Live Preparation:**

* \[ ] Documentation finalization
* \[ ] User guides and tutorials
* \[ ] Community communication and support
* \[ ] Liquidity bootstrapping and initial market making

***

### 🎯 Success Metrics & KPIs

#### Protocol Health Indicators

**Stability Metrics:**

* **Collateral Ratio**: Maintain >101% at all times
* **Peg Stability**: QEURO/EUR exchange rate variance <0.5%
* **Liquidity Coverage**: Vault liquidity >20% of total QEURO supply

**Growth Metrics:**

* **Total Value Locked (TVL)**: Target €10M within 6 months
* **Daily Active Users**: Measure engagement and adoption
* **Transaction Volume**: Track mint/redeem activity

**Yield Performance:**

* **Aave APY Tracking**: Monitor yield generation efficiency
* **User Staking Returns**: Validate competitive yield distribution
* **Protocol Revenue**: Fee generation and sustainability metrics

#### Technical Performance

**System Reliability:**

* **Uptime**: Target 99.9% protocol availability
* **Transaction Success Rate**: >99% successful operations
* **Oracle Responsiveness**: <30 second price update latency

**Gas Efficiency:**

* **Mint Cost**: <50,000 gas per transaction
* **Redeem Cost**: <40,000 gas per transaction
* **Yield Distribution**: Optimized batch processing costs

***

_This MVP Technical Documentation serves as the foundational blueprint for Quantillon Protocol implementation. For questions or technical clarification, please refer to the broader protocol whitepaper or contact the development team._
