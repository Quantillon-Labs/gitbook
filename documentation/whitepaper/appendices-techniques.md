---
description: Appendices Techniques
---

# Appendices Techniques

### Smart Contract Architecture

Quantillon's codebase is structured around a modular Solidity framework, designed for upgradability, auditability, and composability. Core contracts include:

**🔧 Core Smart Contracts**

| Contract             | Function                                                                                     |
| -------------------- | -------------------------------------------------------------------------------------------- |
| **MintManager**      | Handles the conversion of ERC20 assets to QEURO at oracle rates, manages collateral routing. |
| **VaultEngine**      | Oversees collateral deposits, DeFi deployment, liquidation triggers, and hedger margins.     |
| **YieldController**  | Applies yield share logic, calculates Yield Shift, and manages protocol fees.                |
| **GovernanceModule** | Administers voting logic, fee toggles, and whitelisting of new vaults or oracle feeds.       |

All contracts are **non-custodial**, with critical functions gated through governance time-locks. Emergency pause mechanisms are included.

### Oracle and Pricing Infrastructure

The protocol relies on **Chainlink** as its primary data oracle, with fallback nodes under Quantillon Labs. Oracle feeds are used for:

**📊 Data Sources**

* **EUR/USD spot rates**
* **USDC price stability checks**
* **On-chain collateral valuation** (Aave, Maker)

**🛡️ Three-tier Logic**

1. **Medianized price feeds** across multiple sources
2. **Heartbeat and deviation checks** to detect anomalies
3. **Fallback escalation** using governance-mandated oracles

This ensures robust resistance to manipulation or downtime.

### Yield Shift Formula

The dynamic yield redistribution follows a parametric curve governed by the ratio of hedger supply to QEURO demand.

**📐 Mathematical Model**

Let:

* **R** = net available yield from collateral
* **H** = % of target hedger pool filled
* **Yh** = yield allocated to hedgers
* **Yu** = yield allocated to users

Then:

* **Yh = min(1%, R × H × α)**
* **Yu = R − Yh − protocol fee (10%)**

Where **α** is a governance-defined coefficient that tunes market incentives. This creates an implicit rebalancing of protocol incentives and peg resilience.

### Value at Risk and Stress Testing

Quantillon incorporates continuous monitoring of its financial health using:

**📊 Risk Analytics**

* **VaR (95%/99%)** models on EUR/USD and TVL drawdowns
* **Stress tests** assuming FX shock (+/- 10%), collateral yield drop to 0%, or Aave liquidity collapse
* **Daily simulations** to validate liquidation thresholds and hedger margin adequacy

Results are logged on-chain and reviewed by both internal risk modules and external auditors. Public dashboards will visualize these metrics post-launch.

### Compliance Interfaces

Though out-of-scope under MiCA, the protocol maintains a compliance-oriented posture:

**✅ Transparency Measures**

* **Public transparency** of all vault mechanics, treasury flows, and governance changes
* **Voluntary disclosures** issued by Quantillon Foundation
* **Audit trails** for smart contracts and FX margin mechanisms

This hybrid approach supports future institutional integrations and legal adaptability across EU jurisdictions.

> **Quantillon's technical backbone is crafted not only for security and functionality, but also for auditability, regulatory interface, and long-term evolvability.**
