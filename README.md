# Overview

## Quantillon Protocol

<div align="center"><img src=".gitbook/assets/banner.png" alt="Quantillon Protocol Banner" width="100%"></div>

## Overview

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> FX-hedged local-currency infrastructure for DeFi

### 🧭 Welcome to the Quantillon Protocol Documentation

**Quantillon Protocol** is a currency-agnostic DeFi architecture designed to turn deep USD liquidity and yield into **local-currency exposure**. Instead of asking users to hold raw dollar risk just to access DeFi, Quantillon inserts a protocol-native FX hedging layer between USD liquidity and the user-facing asset.

**QEURO** is the first deployment of that architecture. It proves the model on EUR first, while the protocol itself is designed to support additional markets when governance, hedger participation, and economics justify expansion.

***

### 🎯 What Quantillon Is

Quantillon is built around four reusable layers:

1. **USD liquidity and yield sources** as the economic base layer
2. **A protocol hedging engine** that absorbs the FX leg
3. **A local-currency asset layer** for the market being served
4. **Governance, wrappers, and risk controls** that can be reused across deployments

This documentation is therefore split into two perspectives:

* **Protocol Foundation**: the generic architecture and economic logic
* **QEURO: First Deployment**: the current EUR implementation of that architecture

***

### 💶 QEURO Is the First Proof Point

Quantillon starts with the euro because it is one of the clearest non-dollar markets in DeFi:

* Strong demand for local-currency exposure
* Deep EUR/USD hedging infrastructure
* A clear mismatch between local-currency users and USD-denominated yield

QEURO should be understood as the **first production deployment**, not the full definition of Quantillon.

***

### 🏗️ Documentation Map

#### **Protocol Foundation**

* Why non-dollar users are underserved in DeFi
* The reusable Quantillon architecture
* Core protocol mechanisms and design choices

#### **QEURO: First Deployment**

* QEURO, stQEURO, and QTI documentation
* Deployment-specific smart contract components
* EUR-focused implementation details and current token stack

#### **Governance, Risk, and Launch**

* Strategic context and market analysis
* Risk management and sustainability
* Roadmap, organization, governance, and legal material

***

### 💡 Why This Matters

Most DeFi liquidity still lives in dollar-denominated markets. For users, treasuries, and protocols that think in EUR, CHF, JPY, GBP, or another local currency, that creates a structural problem:

* the best liquidity is in USD
* the best yield is often in USD
* but the desired balance-sheet exposure is not necessarily USD

Quantillon is built to close that gap.

***

### 💬 Community & Resources

* **🐦** [**X (Twitter)**](https://x.com/QuantillonLabs) - Updates and ecosystem news
* **💬** [**Discord**](https://discord.gg/uk8T9GqdE5) - Community and technical discussion
* **📱** [**Telegram**](https://t.me/QuantillonLabs) - Announcements and direct access
* **🔗** [**GitHub**](https://github.com/Quantillon-Labs) - Open-source repositories
* **🌐** [**Website**](https://quantillon.money) - Landing page and QEURO app access

***

**Read this documentation as protocol first, deployment second: Quantillon is the architecture, and QEURO is the first market built on top of it.**
