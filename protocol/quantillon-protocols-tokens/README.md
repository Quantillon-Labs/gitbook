# QEURO: First Deployment Token Stack

## Quantillon's First Deployment Tokens

### 📋 Overview

This section documents the **first live deployment** of Quantillon.

That distinction matters:

* **Quantillon** is the protocol architecture
* **QEURO and stQEURO** are deployment-specific assets for the EUR market
* **QTI** governs the protocol above any single deployment

So the pages below should be read as **implementation documentation for the EUR launch**, not as the full scope of the protocol thesis.

***

### 🏗️ Token Roles in the First Deployment

**💶** [**QEURO**](qeuro-token.md)\
The first market-specific exposure asset built on Quantillon. QEURO represents EUR exposure through the protocol's USD-liquidity-plus-FX-hedging architecture.

**📈** [**stQEURO**](stqeuro-token.md)\
The yield-bearing wrapper for the first deployment. stQEURO packages the QEURO market into a yield-accruing format while staying tied to the EUR deployment.

**🏛️** [**QTI**](qti-token.md)\
The governance layer for the broader protocol. QTI is not only about QEURO; it coordinates risk, incentives, and future deployment policy across Quantillon.

***

### 🔄 How These Tokens Relate

```text
Protocol Layer (Quantillon)
        ↓
QTI Governance
        ↓
First Deployment (EUR)
        ↓
QEURO Exposure Layer
        ↓
stQEURO Yield Wrapper
```

This is the right mental model:

* **QTI** sits above the deployment
* **QEURO** is the first market-specific asset
* **stQEURO** extends that first asset into a yield-bearing format

***

### 🧭 Why This Framing Matters

Without this framing, it is easy to mistake the current token stack for the entire protocol story. But Quantillon is designed to support additional local-currency deployments in the future.

That means future deployments may:

* reuse governance and risk controls
* reuse wrapper logic
* change the market-specific exposure asset

The QEURO stack is therefore best seen as the **first production template** for the protocol.

***

### 📚 What To Read Here

* Read **QEURO** for the first deployment's market-specific asset design
* Read **stQEURO** for the first deployment's yield wrapper
* Read **QTI** for the governance layer that sits above the market-specific assets

> **In short:** this section explains how Quantillon is implemented for EUR today, while the rest of the documentation explains why the architecture can extend beyond EUR over time.
