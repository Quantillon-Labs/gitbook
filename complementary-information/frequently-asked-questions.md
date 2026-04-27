---
description: FAQ
---

# Frequently Asked Questions

## Quantillon Protocol - Frequently Asked Questions

Welcome to the Quantillon Protocol FAQ. This page is intentionally **protocol-first**: Quantillon is the architecture, and **QEURO** is the first deployment of that architecture.

### 🏛️ Protocol Basics

* [What is Quantillon Protocol?](frequently-asked-questions.md#q-what-is-quantillon-protocol)
* [How does the protocol work?](frequently-asked-questions.md#q-how-does-the-protocol-work)
* [Why not just use USDC or USDT directly?](frequently-asked-questions.md#q-why-not-just-use-usdc-or-usdt-directly)
* [Why is QEURO the first deployment?](frequently-asked-questions.md#q-why-is-qeuro-the-first-deployment)
* [Which currencies can the protocol support?](frequently-asked-questions.md#q-which-currencies-can-the-protocol-support)

### 🎯 Tokenomics & Governance

* [What is the purpose of the $QTI token?](frequently-asked-questions.md#q-what-is-the-purpose-of-the-usdqti-token)
* [How should I think about QEURO and stQEURO?](frequently-asked-questions.md#q-how-should-i-think-about-qeuro-and-stqeuro)
* [How does the "Yield Shift" mechanism work?](frequently-asked-questions.md#q-how-does-the-yield-shift-mechanism-work)

### 🔒 Security & Trust

* [Is Quantillon secure?](frequently-asked-questions.md#q-is-quantillon-secure)

### 💼 Using Quantillon

* [How can I get $QEURO?](frequently-asked-questions.md#q-how-can-i-get-usdqeuro)
* [Can I redeem $QEURO back to USDC?](frequently-asked-questions.md#q-can-i-redeem-usdqeuro-back-to-usdc)

### 🌍 Community & Ecosystem

* [Who controls the protocol?](frequently-asked-questions.md#q-who-controls-the-protocol)
* [How can I join the community?](frequently-asked-questions.md#q-how-can-i-join-the-community)

***

### 🏛️ Protocol Basics

#### **Q: What is Quantillon Protocol?**

**A:** Quantillon is an **FX-hedging DeFi protocol** that lets users access dollar-denominated liquidity and yield without staying exposed to the dollar by default. It does this by combining:

* a USD liquidity base
* a protocol-native FX hedging layer
* a market-specific local-currency asset layer

**QEURO** is the first deployment of this model for EUR exposure.

#### **Q: How does the protocol work?**

**A:** At a high level:

1. The protocol starts from deep USD liquidity and yield sources.
2. A hedging layer absorbs the FX leg between USD and the target market.
3. Users interact with a local-currency asset instead of holding raw USD exposure.
4. Governance coordinates risk controls, incentives, and deployment expansion.

This pattern is reusable across markets, which is why Quantillon is best understood as a protocol rather than a single-asset product.

#### **Q: Why not just use USDC or USDT directly?**

**A:** Because using USD stablecoins directly solves only half the problem. It gives access to DeFi, but it also leaves the user exposed to the dollar. Quantillon exists for users, treasuries, and applications that want:

* access to USD-denominated DeFi depth
* without turning their balance sheet into a USD balance sheet

#### **Q: Why is QEURO the first deployment?**

**A:** EUR is the first market because it is one of the clearest examples of the mismatch Quantillon solves:

* local-currency users are abundant
* DeFi remains mostly USD-denominated
* the EUR/USD pair is deep enough to support a serious hedging architecture

So QEURO is the first proof point, not the ceiling of the protocol.

#### **Q: Which currencies can the protocol support?**

**A:** The architecture is designed to be **currency-agnostic**. QEURO is first, and the same model can in principle support markets such as CHF, JPY, or GBP when governance, hedger participation, and economics support a deployment.

***

### 🎯 Tokenomics & Governance

#### **Q: What is the purpose of the $QTI token?**

**A:** **QTI** is the governance layer for the protocol as a whole. It is not only about QEURO. It is used to coordinate:

* risk parameters
* incentive design
* deployment policy
* long-term protocol evolution

As Quantillon expands beyond its first deployment, QTI remains the governance surface above the individual markets.

#### **Q: How should I think about QEURO and stQEURO?**

**A:** Think of them as the asset stack for the **first deployment**:

* **QEURO** = the EUR exposure layer built on the Quantillon architecture
* **stQEURO** = the yield-bearing wrapper around that first deployment

These are implementation-specific assets. They show how the architecture works in production today.

#### **Q: How does the "Yield Shift" mechanism work?**

**A:** The **Yield Shift** is the rebalancing engine for the first deployment. It dynamically redistributes yield between Users and Hedgers based on market conditions:

**📈 When hedger participation is high:**

* More yield can flow to QEURO and stQEURO users
* The deployment becomes more attractive for local-currency holders

**📉 When hedger supply is low:**

* More yield can be allocated to Hedgers
* The protocol attracts more delta-neutral capital for peg stability

This creates a self-balancing incentive loop where market forces help maintain deployment health without turning Quantillon into a manually managed currency product.

***

### 🔒 Security & Trust

#### **Q: Is Quantillon secure?**

**A:** Quantillon’s security model spans more than contract audits. It includes:

* transparent smart contracts and public observability
* oracle and collateral controls
* liquidation and pause paths
* governance guardrails around protocol parameters

The details evolve by deployment, but the design goal is consistent: keep the protocol legible, observable, and controllable under stress.

***

### 💼 Using Quantillon

#### **Q: How can I get $QEURO?**

**A:** QEURO is the first user-facing market in the Quantillon stack. To use it, start with the app and documentation:

1. Visit [the app](https://app.quantillon.money/)
2. Connect your wallet
3. Follow the current QEURO flow documented for the first deployment

If you are evaluating the protocol itself, read the architecture pages first and then the QEURO deployment docs.

#### **Q: Can I redeem $QEURO back to USDC?**

**A:** Yes. QEURO is built on top of a USD liquidity path, so redemption back through that underlying path remains part of the first deployment model.

***

### 🌍 Community & Ecosystem

#### **Q: Who controls the protocol?**

**A:** Quantillon is intended to move through **progressive decentralization**. Governance lives at the protocol layer, with QTI designed to play an increasing role in parameters, incentives, and future deployments over time.

#### **Q: How can I join the community?**

**A:** You can join us here:

* [Discord](https://discord.gg/uk8T9GqdE5)
* [Telegram](https://t.me/QuantillonLabs)
* [X / Twitter](https://x.com/QuantillonLabs)

These channels are the best place to follow the evolution of QEURO as the first deployment and Quantillon as the broader protocol thesis.
