# Business Model & Economic Sustainability

Quantillon's first deployment uses a three-token architecture that generates multiple sustainable revenue streams. QEURO and stQEURO are the EUR implementation of the model, while QTI governs the broader protocol and future deployment policy.

### Primary Revenue Sources

The deployed protocol ships with a complete fee framework, most of which is deliberately set to **zero at launch** to bootstrap adoption. The levers below are live in the contracts and governance-adjustable; collected fees route through the FeeCollector, which splits them **60% treasury / 25% dev fund / 15% community** (governance-adjustable).

#### 1. QEURO Operations (Core Stablecoin Activity)

* **Mint/redeem fees**: currently 0, governance-settable up to a 5% cap
* **Yield management**: the treasury share of harvested external-vault yield (currently Morpho USDC lending) - the portion attributable to unstaked QEURO accrues to the treasury

#### 2. stQEURO Yield Infrastructure

* **Yield fee**: a per-series fee on credited staker yield - currently 0, capped at 20%

#### 3. Hedging Operations

* **Hedger position fees**: entry/exit/margin fees - currently 0, governance-settable
* **Reward fee split**: 20% of hedger rewards at claim time

#### 4. Cross-Protocol Integration (planned)

* **Cross-chain bridge fees**: 0.1-0.3% of transfers (future multi-chain phases)
* **Premium institutional services**: advanced analytics, custom integrations
* **Partnership revenue sharing**: integrations with CeDeFi platforms

### Financial Projections

**No revenue projections are published.** All fees are currently 0, usage is at pre-launch levels, and the public launch is planned for Q4 2026; any revenue figure would be an assumption about fee levels governance has not set and volumes that do not exist yet. The fee levers above are what the deployed contracts support, and the TVL targets below are targets, not forecasts.

### Key Performance Indicators

#### Growth Metrics

* **TVL Growth**: Target $1M at the Q4 2026 launch, $10M by Q2 2027, $10M to $100M by Q1 2028
* **Staking Adoption**: 50%+ of QEURO supply in stQEURO within two years of launch
* **Daily Volume**: 2-5% of TVL in trading activity
* **Cross-Chain Distribution**: Base-first today; multi-chain expansion targeted from the second year after launch

#### Sustainability Metrics

* **Operating Margin**: keep the cost base below fee revenue once governance activates fees (no target margin published)
* **Revenue Diversification**: No single source >60% of total revenue
* **User Retention**: >80% of stQEURO holders active after 6 months
* **Protocol Utilization**: Average >80% of collateral deployed in yield strategies

### Capital Efficiency Through Design

Quantillon's model avoids several common inefficiencies in DeFi protocols:

* ❌ No dependence on liquidity mining for peg stability
* ❌ No bribing mechanisms for gauge voting or yield direction
* ❌ Slippage-free mint/redeem operations eliminate arbitrage cost

The dual-pool architecture with dynamic Yield Shift ensures internal market balance, reducing the need for external incentives or unsustainable emissions. In the first deployment, this design is particularly adapted to EUR users who tend to prioritize capital preservation and yield visibility.

### Institutional Fit and Monetization Pathways

Quantillon's infrastructure allows for multiple monetization pathways:

**🏢 Institutional access**

Hedge funds, wealth managers, and corporate treasuries can use QEURO for EUR exposure in DeFi with reduced compliance overhead.

**🔗 CeDeFi integration**

Through partnerships with platforms like Quantfury or Cadmos, QEURO can serve as the euro liquidity backbone in compliant fintech stacks.

**🌉 On/off-ramp monetization**

Future integrations with fiat gateways and custody providers may allow for spread-based revenue models.

The architecture is also designed for future cross-chain deployment, allowing it to tap into additional EVM ecosystems beyond Base for added liquidity depth and composability. Treasury diversification and fixed-income DeFi strategies can further enhance yield capture without increasing risk.

> **In sum, Quantillon is engineered as a low-cost, high-leverage protocol. Its first deployment combines smart contract automation with EUR-denominated asset exposure, while the underlying architecture is designed to remain reusable across future local-currency markets when governance and economics support them.**
