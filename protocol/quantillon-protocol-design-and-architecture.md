# Quantillon Protocol: Design & Architecture

### QEURO Stablecoin Design: A Euro-Pegged, Overcollateralized Instrument

At the core of the Quantillon Protocol lies the **QEURO**—a euro-pegged stablecoin minted against overcollateralized reserves in USD stablecoins, initially USDC. The choice of overcollateralization offers a high-integrity approach to maintaining the peg and minimizing counterparty risk.

QEURO is minted permissionlessly at the oracle price, allowing any user to deposit accepted ERC20 assets which are then routed through the protocol's smart contracts to be swapped into USDC and locked as collateral. The protocol ensures a minimum **101% collateralization ratio**, with liquidation mechanisms triggered automatically via Chainlink or equivalent high-reliability oracles if the threshold is breached. Redeeming QEURO follows the same logic in reverse, enabling frictionless exit at minimal cost.

The result is a stablecoin that provides euro exposure with USD-based liquidity depth, solving a crucial mismatch in DeFi's monetary architecture.

### The Dual-Pool Model: Users and Hedgers

Quantillon introduces a dual-pool mechanism involving two symbiotic actor groups: **Users** and **Hedgers**.

**👥 Users**

Users are DeFi participants who mint QEURO to access euro-denominated liquidity or stake QEURO in order to earn yield. They deposit volatile or stable crypto assets, which are swapped into USDC and locked as protocol collateral. These users maintain exposure to the euro while benefiting from DeFi-native financial products.

**🛡️ Hedgers**

Hedgers are actors who take the opposite position—providing USDC upfront to hedge against euro exposure. They effectively short the EUR/USD pair by engaging in a leveraged, unidirectional perpetual future. In return, they earn compensation consisting of:

* (i) the EUR/USD interest rate differential (typically \~1% annually)
* (ii) a variable "Yield Shift" bonus extracted from the collateral's DeFi yield.

The protocol enforces a liquidation mechanism for hedgers as well: if their margin ratio drops below 1%, they are liquidated and penalized. This mechanism ensures disciplined FX exposure management and peg resilience.

### Liquidity Architecture: Inheriting Depth from USDC and Forex

Unlike euro-stablecoin predecessors that relied heavily on centralized market makers or illiquid LP incentives, Quantillon's model ensures **"liquidity by design."**

* **📈 On the user side**, all incoming deposits are swapped into USDC—the most liquid dollar-based stablecoin—thus ensuring instant deployability and deep routing.
* **🌍 On the hedger side**, liquidity stems from traditional Forex markets, where EUR/USD is one of the most traded currency pairs globally. This design enables the protocol to scale capital efficiency without relying on shallow DEX pools or bribe-driven gauges.

As a result, QEURO enjoys both upstream (USDC) and downstream (Forex) liquidity, reinforcing slippage-free mint/redeem operations and improved user experience.

### Native Composability and Embedded DEX

Quantillon is designed as a holistically integrated protocol stack. Rather than relying on external DEXs like Curve or Balancer, the protocol features its own embedded exchange mechanism. This ensures control over routing fees, eliminates bribing systems, and increases revenue retention.

Smart contract logic for mint/redeem, hedging, staking, liquidation, and governance is unified under a modular Solidity-based architecture audited and battle-tested through external bug bounty programs. Governance functions, including Yield Shift calibration and vault whitelisting, are performed by holders of the $QTI token.\


### Vault Variants: Modular Architecture for Risk Segmentation

Quantillon's modular design enables multiple vault variants, each corresponding to specific collateral backends and risk profiles. This architecture serves diverse user needs while maintaining unified euro peg mechanics across all variants.

**Vault Portfolio Overview**

| Vault Type | Collateral Backend           | Risk Profile | Target APY | Target Users       |
| ---------- | ---------------------------- | ------------ | ---------- | ------------------ |
| **aQEURO** | Aave USDC lending            | 🟢 Low       | 4-8%       | Retail investors   |
| **mQEURO** | MakerDAO PSM/DSR             | 🟢 Very Low  | 3-6%       | Conservative users |
| **bQEURO** | Tokenized T-Bills & RWAs     | 🟢 Low       | 5-7%       | Institutions       |
| **eQEURO** | Ethena & advanced strategies | 🟡 Medium    | 6-12%      | Sophisticated DeFi |

**Detailed Vault Specifications**

**aQEURO (Aave Integration - Default Configuration)**

* **Collateral Deployment:** 85% Aave USDC lending, 10% emergency buffer, 5% operations
* **Yield Mechanism:** Direct participation in Aave lending markets with dynamic APY
* **Risk Factors:** Aave protocol risk, USDC depeg risk, smart contract risk
* **Target Users:** Retail DeFi users seeking stable euro yield with proven infrastructure

**mQEURO (MakerDAO Integration)**

* **Collateral Deployment:** MakerDAO Peg Stability Module (PSM) and Dai Savings Rate (DSR)
* **Yield Mechanism:** Conservative yield through DAI ecosystem stability mechanisms
* **Risk Factors:** MakerDAO governance risk, DAI stability risk
* **Target Users:** Ultra-conservative users prioritizing capital preservation

**bQEURO (Real World Assets Focus)**

* **Collateral Deployment:** Tokenized U.S. Treasury Bills, German Bunds, Corporate Bonds
* **Yield Mechanism:** Traditional fixed-income yields through compliant tokenization platforms
* **Risk Factors:** Custody risk, regulatory risk, lower liquidity
* **Target Users:** Institutional investors requiring regulatory clarity and traditional yield sources
* **Minimum Threshold:** €100,000 institutional access requirement

**eQEURO (Advanced DeFi Strategies)**

* **Collateral Deployment:** Ethena USDe, Lido stETH, Rocketpool rETH, recursive strategies
* **Yield Mechanism:** Leveraged yield farming up to 3x through sophisticated DeFi composability
* **Risk Factors:** Higher smart contract risk, leverage risk, strategy complexity
* **Target Users:** Sophisticated DeFi participants seeking maximum yield optimization
* **Advanced Features:** Automated rebalancing, yield optimization bots, governance-adjustable parameters

**Cross-Vault Mechanics**

**Unified Peg Maintenance:** All vault variants maintain the same EUR target price through shared oracle infrastructure and arbitrage mechanisms.

**Risk Isolation:** Individual vault failures are contained and do not affect other variants through modular smart contract architecture.

**Arbitrage Opportunities:** Price differences between variants create natural trading incentives that support overall ecosystem liquidity.

### stQEURO: Yield-Bearing Euro Infrastructure

Beyond serving as a euro-pegged stablecoin, Quantillon introduces stQEURO—a yield-bearing wrapper that automatically compounds returns from QEURO collateral deployment while maintaining full liquidity and composability. This innovation bridges the gap between passive euro exposure and active DeFi yield generation.

#### Auto-Compounding Mechanism

Unlike traditional staking systems requiring manual reward claims, stQEURO automatically increases in intrinsic value over time. The quantity of stQEURO tokens in user wallets remains constant, but each token becomes worth more QEURO as yields accumulate:

**Value Appreciation Formula:**

```
stQEURO Exchange Rate = 1 + (Cumulative Yield ÷ Total Days × Days Held)
```

**Example Timeline:**

* **Day 0:** Stake 1,000 QEURO → Receive 1,000 stQEURO (Rate: 1.000)
* **Day 365:** Still hold 1,000 stQEURO (Rate: 1.053 with 5.3% APY)
* **Unstaking:** 1,000 stQEURO → 1,053 QEURO

#### Yield Source & Distribution

stQEURO's yield originates from the underlying QEURO collateral deployed across battle-tested DeFi protocols:

```
QEURO Collateral Deployment (100%)
├── Aave USDC Lending: 85% (Base APY: 4-12%)
├── Emergency Liquidity Buffer: 10%
└── Protocol Operations Reserve: 5%

Yield Distribution:
├── Protocol Fees: 10% (Treasury + Development)
├── Hedger Compensation: Variable 1-3% (EUR/USD spread + Yield Shift)
└── stQEURO Holders: Remaining 87-89% (Auto-compounded)
```

#### Technical Advantages

**Instant Liquidity:** stQEURO maintains full liquidity without lock periods, enabling immediate unstaking at current exchange rates.

**DeFi Composability:** Users can deploy stQEURO in other protocols (lending, LP positions, collateral) while continuing to earn compounding yield.

**Tax Efficiency:** No rebase events or reward distributions that could trigger taxable income in many jurisdictions.

> **Quantillon's architecture reflects a principled design philosophy: capital efficiency, transparency, resilience, and modularity. It is engineered not only for market fit today, but for adaptability in the rapidly evolving regulatory and macroeconomic context of tomorrow.**
