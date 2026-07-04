# Business Model & Economic Sustainability

Quantillon's first deployment uses a three-token architecture that generates multiple sustainable revenue streams. QEURO and stQEURO are the EUR implementation of the model, while QTI governs the broader protocol and future deployment policy.

### Primary Revenue Sources

The deployed protocol ships with a complete fee framework, most of which is deliberately set to **zero at launch** to bootstrap adoption. The levers below are live in the contracts and governance-adjustable; collected fees route through the FeeCollector, which splits them **60% treasury / 25% dev fund / 15% community** (governance-adjustable).

#### 1. QEURO Operations (Core Stablecoin Activity)

* **Mint/redeem fees**: currently 0, governance-settable up to a 5% cap
* **Yield management**: the treasury share of harvested external-vault yield (currently Morpho USDC lending) — the portion attributable to unstaked QEURO accrues to the treasury

#### 2. stQEURO Yield Infrastructure

* **Yield fee**: a per-series fee on credited staker yield — currently 0, capped at 20%
* **Enhanced trading volume** through improved user retention
* **Premium yield optimization services** for institutional users

#### 3. Hedging Operations

* **Hedger position fees**: entry/exit/margin fees — currently 0, governance-settable
* **Reward fee split**: 20% of hedger rewards at claim time

#### 4. Cross-Protocol Integration (planned)

* **Cross-chain bridge fees**: 0.1-0.3% of transfers (future multi-chain phases)
* **Premium institutional services**: advanced analytics, custom integrations
* **Partnership revenue sharing**: integrations with CeDeFi platforms

### Financial Projections

> **Illustrative and aspirational**: the projections below assume fee levels that are **not currently active** (fees are 0 at launch) and volumes that depend on adoption. They model what the fee framework could yield if governance activates it — they are not forecasts of current revenue.

#### Conservative Growth Scenario (€50M TVL by Year 1)

**Revenue Breakdown:**

```
Annual Revenue Calculation (illustrative fee assumptions):
├── Mint/Redeem Volume: €500M (10x TVL turnover)
│   └── Fees if activated at 0.1% (cap 5%): €500M × 0.1% = €500K
├── Yield Management: €50M × 7% yield × 10% treasury share = €350K
├── stQEURO Yield Fee if activated at 5% (cap 20%): €20M staked × 5% yield × 5% = €50K
└── Cross-Chain Fees (future phases): €100M volume × 0.2% average = €200K

Total Annual Revenue: €1,100K
Operating Costs: €600K (development, infrastructure, legal, audits)
Net Operating Profit: €500K (45% margin)
```

#### Optimistic Growth Scenario (€500M TVL by Year 2)

**Revenue Breakdown:**

```
Annual Revenue Calculation (illustrative fee assumptions):
├── Mint/Redeem Volume: €5B (10x TVL turnover)
│   └── Fees if activated at 0.1% (cap 5%): €5B × 0.1% = €5M
├── Yield Management: €500M × 7% yield × 10% treasury share = €3.5M
├── stQEURO Yield Fee if activated at 5% (cap 20%): €300M staked × 5% yield × 5% = €750K
└── Cross-Chain Fees (future phases): €1.5B volume × 0.2% average = €3M

Total Annual Revenue: €12.25M
Operating Costs: €2.5M (scaled operations)
Net Operating Profit: €9.75M (80% margin)
```

### Key Performance Indicators

#### Growth Metrics

* **TVL Growth**: Target €100M by Month 12, €1B by Month 36
* **Staking Adoption**: 50%+ of QEURO supply in stQEURO by Year 2
* **Daily Volume**: 2-5% of TVL in trading activity
* **Cross-Chain Distribution**: Base-first today; multi-chain expansion targeted from Year 2

#### Sustainability Metrics

* **Operating Margin**: Maintain >40% across market conditions
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
