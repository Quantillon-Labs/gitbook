# Mechanisms

## Mechanisms

Understanding how Quantillon Protocol operates is essential for both users and developers. This comprehensive guide breaks down the core mechanisms that power QEURO—from the dual-pool architecture and overcollateralization model to the innovative Yield Shift system that maintains peg stability while generating sustainable returns.

***

### 🏗️ Protocol Architecture Overview

Quantillon Protocol represents a paradigm shift in stablecoin design, combining the capital efficiency of overcollateralized systems with the liquidity advantages of forex markets. At its core, the protocol operates through interconnected mechanisms that ensure peg stability, yield generation, and capital efficiency.

#### Core Design Principles

* **🔒 Over-collateralization**: Minimum 101% backing ensures systemic stability
* **⚖️ Delta-neutral hedging**: FX risk is distributed across willing participants
* **📈 Dynamic yield distribution**: Market-responsive incentive alignment
* **🌊 Liquidity inheritance**: Leverages existing USDC and forex market depth
* **🏛️ Decentralized governance**: Community-controlled parameter management

***

### 💶 QEURO Stablecoin Mechanics

#### Minting Process

The QEURO minting mechanism is designed for simplicity and capital efficiency:

**📥 Step-by-Step Minting**

1. **Asset Deposit**: Users deposit accepted ERC-20 tokens (USDC, ETH, WBTC, etc.)
2. **Automatic Routing**: Protocol routes deposits through integrated DEX for optimal pricing
3. **USDC Conversion**: All deposits are automatically converted to USDC collateral
4. **Oracle Price Check**: Chainlink oracles provide real-time EUR/USD exchange rates
5. **QEURO Issuance**: Users receive QEURO at current oracle price minus 0.1% minting fee
6. **Collateral Lock**: USDC is deposited into yield-generating vaults (Aave, Maker, etc.)

```
Example Minting Transaction:
User deposits: 1,000 USDC
EUR/USD rate: 1.10
Minting fee: 0.1% = 1 USDC
Net collateral: 999 USDC
QEURO received: 999 ÷ 1.10 = 908.18 QEURO
```

**⚡ Key Features**

* **Zero slippage** minting at oracle rates
* **Instant settlement** within one block
* **Permissionless access** 24/7 availability
* **Multi-asset support** for user convenience

#### Redemption Process

Redemption operates as the inverse of minting, ensuring users can always exit at fair value:

**📤 Step-by-Step Redemption**

1. **QEURO Burn**: Users submit QEURO for redemption
2. **Oracle Verification**: Current EUR/USD rate determines USD value
3. **Collateral Release**: Equivalent USDC withdrawn from vaults
4. **Fee Deduction**: 0.1% redemption fee applied
5. **USDC Transfer**: Net USDC transferred to user wallet

```
Example Redemption Transaction:
User redeems: 900 QEURO
EUR/USD rate: 1.08
USD value: 900 × 1.08 = 972 USDC
Redemption fee: 0.1% = 0.97 USDC
Net USDC received: 971.03 USDC
```

***

### 🎭 Dual-Pool Architecture

The innovative dual-pool system is what sets Quantillon apart from traditional stablecoins. It creates natural peg stability through aligned economic incentives rather than relying solely on arbitrage.

#### 👥 Users Pool

Users are participants who mint and hold QEURO for euro exposure and yield generation.

**User Motivations**

* **🇪🇺 Native Euro Exposure**: Eliminate EUR/USD volatility risk
* **📈 Yield Generation**: Earn returns through staking mechanisms
* **🔗 DeFi Access**: Participate in euro-denominated DeFi strategies
* **💼 Treasury Management**: Corporate and institutional euro liquidity

**User Mechanics**

* Mint QEURO against accepted collateral
* Stake QEURO to earn variable APY from Yield Shift
* Participate in governance through staked positions
* Redeem anytime at oracle-determined rates

#### 🛡️ Hedgers Pool

Hedgers provide the delta-neutral backbone that maintains peg stability while earning compensation for EUR/USD exposure.

**Hedger Function**

Hedgers essentially take **long EUR/USD positions** in exchange for:

* Interest rate differential compensation (typically \~1% annually)
* Variable Yield Shift bonuses based on supply/demand
* Protocol fee sharing when activated

**Hedger Entry Process**

1. **Deposit USDC**: Provide USD-denominated liquidity
2. **Leverage Selection**: Choose exposure level (typically 2-10x)
3. **Margin Requirements**: Maintain minimum collateralization ratio
4. **Yield Participation**: Earn compensation from multiple sources

**Risk Management**

* **Liquidation Threshold**: Automatic liquidation if margin ratio falls below 101%
* **Dynamic Pricing**: Real-time EUR/USD exposure pricing via Chainlink
* **Position Limits**: Maximum exposure limits prevent concentration risk

#### 🔄 Pool Interaction Dynamics

The dual pools create a symbiotic relationship:

```
Users ←→ Protocol ←→ Hedgers
  ↓         ↓         ↓
Euro      USDC      USD
Exposure  Backing   Risk
```

* **Users** get euro exposure without FX risk
* **Hedgers** assume FX risk for yield compensation
* **Protocol** maintains stability through balanced incentives

***

### 📊 Overcollateralization Model

Quantillon uses overcollateralization—meaning that the value of assets held in reserves is greater than the pegged value—to mitigate the inherent volatility of the underlying forex markets.

#### Collateralization Requirements

**Minimum Ratios**

* **Users**: 101% minimum collateralization
* **Hedgers**: 101% margin requirement
* **Protocol**: Dynamic buffer based on volatility

**Liquidation Mechanisms**

**User Liquidations:**

* Triggered when collateral value falls below 101% of QEURO debt
* Automatic execution via Chainlink price feeds
* 5% liquidation penalty to maintain protocol health
* Liquidated collateral sold to cover debt and penalties

**Hedger Liquidations:**

* Triggered when margin ratio falls below 101%
* EUR/USD movement against hedger positions
* Automated liquidation bot network ensures rapid execution
* Liquidation penalty redistributed to remaining hedgers

#### Collateral Management

**Accepted Collateral Types**

* **USDC**: Primary collateral for direct backing
* **ETH**: Blue-chip cryptocurrency with high liquidity
* **WBTC**: Wrapped Bitcoin for portfolio diversification
* **Additional assets**: Approved through governance voting

**Vault Deployment Strategy**

| Vault Type | Collateral Backend  | Risk Profile | Target APY |
| ---------- | ------------------- | ------------ | ---------- |
| **aQEURO** | Aave USDC lending   | Low          | 3-7%       |
| **mQEURO** | MakerDAO DSR/PSM    | Very Low     | 2-5%       |
| **bQEURO** | Tokenized T-Bills   | Low          | 4-6%       |
| **eQEURO** | Advanced strategies | Medium       | 5-12%      |

***

### ⚖️ The Yield Shift Mechanism

The Yield Shift represents Quantillon's most innovative feature—a dynamic system that automatically rebalances yield distribution based on market conditions.

#### Mathematical Foundation

The Yield Shift operates on a continuous rebalancing formula:

**Base Variables**

* **R** = Total yield from collateral deployment
* **H** = Hedger pool utilization ratio (0-100%)
* **U** = User pool demand ratio
* **α** = Governance-defined sensitivity parameter

**Distribution Formula**

```
Hedger Yield = min(Interest_Rate_Differential + (Yield_Shift × R), Max_Hedger_Yield)
User Yield = R - Hedger_Yield - Protocol_Fee(10%)
Yield_Shift = α × (Target_H - Current_H) / Target_H
```

Where:

* **Target\_H** = Optimal hedger pool ratio (typically 30-50%)
* **Current\_H** = Actual hedger participation level
* **α** = Sensitivity coefficient (0.1 - 2.0)

#### Dynamic Rebalancing Examples

**Scenario 1: Excess Hedger Supply**

* **Situation**: Hedger utilization = 80% (above target 50%)
* **Yield Shift**: Negative → More yield flows to Users
* **Result**: Attracts more QEURO demand, rebalances pools

**Scenario 2: Insufficient Hedgers**

* **Situation**: Hedger utilization = 20% (below target 50%)
* **Yield Shift**: Positive → More yield flows to Hedgers
* **Result**: Attracts delta-neutral capital, strengthens peg

**Scenario 3: Balanced Pools**

* **Situation**: Hedger utilization = 50% (at target)
* **Yield Shift**: Neutral → Standard distribution
* **Result**: Stable yields for both user groups

#### Real-Time Adjustment Mechanics

The Yield Shift updates **continuously** based on:

1. **📊 Pool Utilization**: Real-time hedger/user ratios
2. **💱 FX Volatility**: EUR/USD movement amplifies shifts
3. **🏦 Collateral Performance**: Underlying yield changes
4. **⚖️ Governance Parameters**: Community-set bounds and targets

***

### 🔮 Oracle & Pricing Infrastructure

Accurate, tamper-resistant pricing is critical for all protocol mechanisms. Quantillon employs a multi-layered oracle system for maximum reliability.

#### Primary Oracle Sources

**Chainlink Integration**

* **EUR/USD Price Feeds**: Multiple data aggregators
* **Heartbeat Monitoring**: Maximum 1-hour update intervals
* **Deviation Thresholds**: 0.1% price movement triggers updates
* **Backup Feeds**: Secondary oracle validation

**Price Feed Architecture**

```
Chainlink EUR/USD → Price Aggregator → Protocol Logic
     ↓                    ↓               ↓
  Backup Oracle → Circuit Breaker → Emergency Pause
```

#### Failsafe Mechanisms

**Circuit Breakers**

* **Price Deviation Limits**: >5% movement triggers review
* **Heartbeat Monitoring**: Stale feeds automatically flagged
* **Consensus Requirements**: Multiple oracle confirmation

**Emergency Protocols**

* **Governance Override**: Emergency oracle updates
* **Trading Halts**: Automatic suspension during anomalies
* **Manual Review**: Foundation intervention capabilities

***

### 🛡️ Liquidation & Risk Management

Quantillon's liquidation system ensures protocol solvency while minimizing user losses through efficient, transparent processes.

#### Liquidation Triggers

**Automated Monitoring**

* **Real-time collateral tracking** via Chainlink oracles
* **Health factor calculation** for all positions
* **Liquidation bot network** for rapid execution
* **Public liquidation interface** for community participation

**Liquidation Process Flow**

1. **Health Check**: Continuous monitoring of collateralization ratios
2. **Trigger Event**: Position falls below 101% threshold
3. **Liquidation Call**: Automated or manual liquidation initiated
4. **Collateral Sale**: Assets sold at market rates
5. **Debt Coverage**: QEURO debt burned with proceeds
6. **Penalty Distribution**: 5% penalty split between liquidators and protocol

#### Risk Parameters

**Dynamic Risk Adjustment**

* **Volatility-based margins**: Higher volatility = higher requirements
* **Liquidity considerations**: Illiquid assets require higher collateral
* **Market stress response**: Automatic parameter tightening

**Emergency Measures**

* **Global settlement**: Protocol-wide redemption in extreme scenarios
* **Pause mechanisms**: Halt new positions during market stress
* **Governance intervention**: Community-controlled crisis response

***

### 🏛️ Governance Integration

All protocol mechanisms operate under the oversight of decentralized governance, ensuring community control while maintaining operational efficiency.

#### Governable Parameters

**Economic Variables**

* **Yield Shift sensitivity** (α coefficient)
* **Target pool ratios** for optimal balance
* **Fee structures** (minting, redemption, protocol)
* **Liquidation penalties** and thresholds

**Risk Management**

* **Collateralization requirements** for different asset types
* **Oracle parameters** and backup sources
* **Emergency pause authorities** and conditions
* **Vault whitelisting** for new collateral backends

#### Governance Process

**Proposal Lifecycle**

1. **Community Discussion**: Forum-based proposal development
2. **Formal Submission**: On-chain proposal creation
3. **Voting Period**: 7-day voting window for $QTI holders
4. **Implementation**: Automatic execution if approved
5. **Review Period**: 24-48 hour timelock for critical changes

**Voting Power Distribution**

* **$QTI Token Weight**: Proportional to holdings
* **Staking Multipliers**: Enhanced voting power for committed users
* **Delegation Options**: Community members can delegate votes
* **Quorum Requirements**: Minimum participation for validity

***

### 🔧 Integration & Composability

Quantillon's mechanisms are designed for seamless integration with the broader DeFi ecosystem, enabling composability and innovation.

#### Protocol Interfaces

**Standard ERC-20 Compatibility**

* **QEURO Token**: Full ERC-20 standard compliance
* **DEX Integration**: Native support for Uniswap, Curve, Balancer
* **Lending Protocols**: Collateral for Aave, Compound, etc.
* **Cross-chain Bridges**: LayerZero and other bridge protocols

**Advanced Integration Features**

* **Flash Loan Support**: Atomic arbitrage and liquidation
* **Vault Strategy Plugins**: Custom yield generation modules
* **Cross-protocol Yield**: Compound strategies across platforms
* **Institutional APIs**: Direct integration for qualified entities

#### Developer Resources

**Smart Contract Architecture**

* **Modular Design**: Upgradeable proxy patterns
* **Open Source**: Fully auditable and forkable
* **Documentation**: Comprehensive technical guides
* **Testing Suite**: Battle-tested security frameworks

***

### 📈 Performance Metrics & Monitoring

Understanding protocol health requires continuous monitoring of key mechanisms and their performance.

#### Key Performance Indicators

**Stability Metrics**

* **Peg Maintenance**: QEURO price vs. EUR target
* **Collateralization Ratio**: System-wide backing level
* **Liquidation Frequency**: Risk management effectiveness
* **Yield Shift Responsiveness**: Mechanism adjustment speed

**Efficiency Metrics**

* **Capital Utilization**: Collateral deployment efficiency
* **Yield Generation**: APY across different vault types
* **Gas Optimization**: Transaction cost effectiveness
* **Slippage Minimization**: Trading efficiency measures

#### Public Dashboards

**Real-time Monitoring**

* **Live peg tracking** with historical charts
* **Pool utilization ratios** and Yield Shift status
* **Vault performance** across all backends
* **Liquidation activity** and system health indicators

**Historical Analysis**

* **Performance attribution** across market cycles
* **Mechanism effectiveness** during stress periods
* **User behavior patterns** and adoption metrics
* **Competitive positioning** vs. other stablecoins

***

### 🎯 Mechanism Optimization

Quantillon's mechanisms continuously evolve based on real-world performance and community feedback.

#### Performance Optimization

**Algorithmic Improvements**

* **Machine learning** for Yield Shift optimization
* **Predictive modeling** for liquidation risk
* **Gas efficiency** improvements through batching
* **MEV protection** strategies for user transactions

**Community-Driven Enhancement**

* **Mechanism proposals** from protocol users
* **A/B testing** of parameter modifications
* **Performance incentives** for optimization contributions
* **Research grants** for academic analysis

#### Future Mechanism Development

**Planned Enhancements**

* **Cross-chain expansion** to L2s and alternative chains
* **Additional vault types** for diverse risk profiles
* **Synthetic asset** support beyond EUR
* **Institutional features** for qualified participants

> **Quantillon's mechanisms represent a new paradigm in stablecoin design—combining the stability of overcollateralization with the efficiency of delta-neutral hedging and the innovation of dynamic yield distribution. This creates a self-balancing ecosystem that serves both euro-native users and global DeFi participants.**
