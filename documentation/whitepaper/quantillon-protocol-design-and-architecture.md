---
description: 'Quantillon Protocol: Design & Architecture'
---

# Quantillon Protocol: Design & Architecture

### QEURO Stablecoin Design: A Euro-Pegged, Overcollateralized Instrument

At the core of the Quantillon Protocol lies the **QEURO**—a euro-pegged stablecoin minted against overcollateralized reserves in USD stablecoins, initially USDC. The choice of overcollateralization offers a high-integrity approach to maintaining the peg and minimizing counterparty risk, avoiding the algorithmic fragility or undercollateralization that plagued predecessors like TerraUSD or IRON.

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

Smart contract logic for mint/redeem, hedging, staking, liquidation, and governance is unified under a modular Solidity-based architecture audited and battle-tested through external bug bounty programs. Governance functions, including Yield Shift calibration and vault whitelisting, are performed by holders of the $QTI token.

### Vault Variants: Modular Architecture for Risk Segmentation

To ensure flexibility and scalability, Quantillon introduces the concept of **vault variants**, each corresponding to a specific collateral backend. This modular approach enables the protocol to serve diverse user needs and risk profiles:

| Vault      | Collateral Backend                                                     | Target Users          |
| ---------- | ---------------------------------------------------------------------- | --------------------- |
| **aQEURO** | Backed by Aave-deposited USDC                                          | Default configuration |
| **mQEURO** | Collateral held in MakerDAO vaults                                     | Conservative users    |
| **bQEURO** | Backed by tokenized T-Bills or short-duration RWAs (Real World Assets) | Institutional clients |
| **eQEURO** | Uses innovative collateral models such as Ethena's synthetic yield     | Advanced users        |

Each variant maintains the same euro peg mechanics but differs in yield source, risk exposure, and potential institutional fit. This flexibility supports composability with a wide range of DeFi platforms and custody arrangements.

> **Quantillon's architecture reflects a principled design philosophy: capital efficiency, transparency, resilience, and modularity. It is engineered not only for market fit today, but for adaptability in the rapidly evolving regulatory and macroeconomic context of tomorrow.**
