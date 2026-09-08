# Risks & Mitigation Strategies

### Currency Risk: EUR/USD Volatility

Quantillon's first deployment serves EUR users via USD-collateralized instruments, which requires active hedging of EUR/USD exposure. Volatility in this currency pair, especially under divergent monetary policies between the ECB and the Fed, presents a persistent risk to QEURO peg maintenance.

**🛡️ Mitigation:** The protocol's architecture is explicitly delta-neutral. The designated hedger takes the offsetting side of the protocol's EUR/USD exposure in exchange for predictable compensation, aligning its incentives with peg stability. The on-chain margin floor, the protocol-level liquidation mode and real-time FX oracles (the hedge venue's EUR/USD market mid - Hyperliquid - with Chainlink as fallback) enforce discipline. Additionally, the Yield Shift mechanism acts as a buffer: as FX volatility rises, incentives for hedgers increase dynamically.

Since September 2026 the hedge runs on a margin policy targeting 2.5%: the on-chain HedgerPool minimum margin ratio is 2.5% (250 bps) and the minting floor is 102.5%. Quantillon Labs' hedging engine keeps the collateral of both legs of the hedge - the HedgerPool position on Base and the Hyperliquid perpetual - near that target through bounded, monitored transfers that are blocked whenever they would bring the protocol collateralization ratio too close to the minting floor (see [HedgerPool](../protocol/hedger-pool.md#operational-margin-policy-september-2026)).

**Hedge-venue concentration:** the hedge is executed on a single venue (Hyperliquid). This is an **accepted risk**, mitigated operationally: the venue's margin is kept near the 2.5% target with bounded, monitored transfers, an independent watchdog freezes mint/redeem if the hedge or the venue becomes unhealthy, and the protocol can fall back to Chainlink pricing in one transaction. Diversifying venues remains possible by design (the oracle's market slot is venue-agnostic) but is not planned.

### Liquidity Risk: Capital Flight and Redemption Pressure

Periods of market stress may trigger mass redemptions, potentially challenging the protocol's ability to unwind positions or liquidate collateral efficiently.

**🛡️ Mitigation:** Quantillon inherits the deep liquidity of USDC on the user side and leverages the Forex market on the hedger side. Redemption operations are slippage-free, and collateral is deployed on liquid DeFi markets (currently Morpho USDC lending on Base) with instant withdrawal. Furthermore, future vault variants can diversify exposure (e.g., T-Bills, other lending markets), improving redemption resiliency.

### Smart Contract and Oracle Risk

The protocol relies on smart contracts for collateral management, minting, and liquidation. Any vulnerability - whether in protocol contracts or oracle feeds - can undermine systemic integrity.

**🛡️ Mitigation:** **No audit by a professional security firm has been carried out to date.** What the contracts have been through instead: repeated code-review passes with several frontier AI code-analysis models, and findings reported by independent whitehat researchers. Findings are triaged and fixed as they come in, with the remediation shipped on-chain through the normal upgrade path; this is continuous work, not a one-off pass. A bug bounty is planned. Readers should weigh this accordingly: model-driven review and whitehat reports are not a substitute for a professional audit. The protocol adopts a modular architecture, limiting systemic blast radius in case of an exploit. Pricing relies on two independent price paths - the hedge venue's market mid published on-chain by Quantillon's own publisher, and Chainlink as fallback - with circuit breakers for anomalous readings and an independent watchdog that freezes mint/redeem on a stale, circuit-broken or dislocated price.

### Governance Risk and Protocol Capture

As with all DAO-based systems, Quantillon faces the risk of governance capture or low voter participation, especially in early phases.

**🛡️ Mitigation:** Governance powers are progressively decentralized. Initially, a 2-of-3 Gnosis Safe controls privileged roles, with core-contract upgrades gated by a 12-hour timelock to ensure stability. Over time, governance evolves toward QTI token holders (the veQTI system is coded but dormant). A quorum-based system and time-locked execution provide transparency and delay in decision-making.

### Regulatory Risk: Legal Classification and MiCA

Although Quantillon targets the Recital 22 exemption under MiCA, regulatory interpretation can evolve, especially as EU authorities refine crypto oversight.

**🛡️ Mitigation:** The protocol operates under a hybrid compliance architecture. Its decentralized nature is verifiable and publicly auditable. Meanwhile, Quantillon Labs (a French SAS) interfaces with regulators and external auditors, maintaining legal dialogue (notably with the French ACPR); the planned Quantillon Foundation - jurisdiction to be determined - is intended to take over this regulatory-interface role once established. These entities can issue voluntary disclosures or risk assessments without compromising decentralization.

### DeFi Contagion Risk

Quantillon interacts with other DeFi platforms, notably Morpho (the current external yield venue). In the event of failure or depegging on these platforms, collateral could be impaired.

**🛡️ Mitigation:** Collateral can be spread across several governance-registered external vaults (one - the MetaMorpho USDC vault - is live today), and a vault can be deactivated via governance to reduce exposure. The protocol monitors real-time risk parameters, including liquidity ratios and asset volatility. In extreme scenarios, emergency pause mechanisms are available.

### stQEURO-Specific Risk Factors

The introduction of yield-bearing euro infrastructure creates additional risk vectors requiring specialized mitigation strategies.

**Auto-Compounding Mechanism Risk**

**Risk:** Smart contract vulnerabilities in yield calculation or distribution logic could result in incorrect stQEURO valuations or user fund loss.

**🛡️ Mitigation:**&#x20;

* AI-driven code review and whitehat reports covering the protocol contracts, including the ERC-4626 stQEURO series and the harvest/distribution path, with findings remediated on-chain on an ongoing basis; no professional audit-firm review to date
* Independent watchdog and monitoring with automatic pause of mint/redeem
* Emergency pause functionality with governance-controlled restart procedures
* A segregated insurance fund for compounding mechanism failures is **under consideration - not implemented in the deployed protocol**

**Yield Volatility and User Expectation Risk**

**Risk:** Rapid changes in underlying external-vault yields (currently Morpho) could create user dissatisfaction or mass unstaking events if stQEURO returns fall below expectations.

**🛡️ Mitigation:**&#x20;

* 7-day moving average smoothing for APY display to reduce short-term volatility perception
* Transparent communication about yield sources and market dependency
* A protocol yield reserve buffer and APY floor/ceiling mechanisms have been discussed but are **under consideration - not implemented in the deployed protocol**; stQEURO yields are market-driven with no guaranteed floor or ceiling

**Cross-Protocol Composability Risk**

**Risk:** stQEURO's use as collateral in other DeFi protocols could create cascading liquidation events if stQEURO value appreciation calculations fail or if oracle feeds become unreliable.

**🛡️ Mitigation:**&#x20;

* Conservative oracle update mechanisms with multiple data sources and heartbeat monitoring
* Partnership agreements with major DeFi protocols for standardized stQEURO valuation methods
* Graduated rollout of stQEURO integrations with smaller protocols before major platform adoption
* Emergency communication channels for rapid coordination during cross-protocol incidents

> **Quantillon is built on the principle of resilient decentralization. Rather than avoiding risk, it manages it explicitly through incentive design, robust engineering, and layered governance. This makes it uniquely equipped to operate in a volatile, fragmented, and evolving DeFi environment.**
