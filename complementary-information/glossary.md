# Glossary

## Glossary of Key Terms

> 📘 This glossary provides definitions for key concepts and technical terms used throughout the Quantillon Protocol documentation.

***

### ACPR (Prudential Supervision and Resolution Authority)

The French authority responsible for prudential supervision and resolution, which engages in regulatory dialogue with Quantillon.

***

### APY (Annual Percentage Yield)

The annualized percentage return, measuring the real yield earned on an investment over one year.

***

### aQEURO / mQEURO / bQEURO / eQEURO

Early naming concepts for risk-segmented QEURO vault variants, each representing a different collateral strategy:

* **aQEURO**: Backed by USDC deposited on a lending market.
* **mQEURO**: Backed by MakerDAO vaults.
* **bQEURO**: Backed by tokenized Treasury Bills (T-Bills).
* **eQEURO**: Based on synthetic yield models.

In the live deployment, vault variants are exposed as per-vault **stQEURO series** instead (e.g. `stQEUROMORPHO1`, backed by a Morpho USDC vault); the other variants remain roadmap concepts.

***

### Cantillon Effect

An economic concept describing how monetary expansion initially benefits capital allocators closest to the money source, then gradually spreads, diluting value for others.\
Quantillon aims to **reverse** this effect to benefit European savers.

***

### CASP (Crypto-Asset Service Providers)

Entities providing crypto-asset services, as defined in the **MiCA regulation**.

***

### CeDeFi (Centralized Decentralized Finance)

A hybrid approach combining centralized and decentralized systems, often integrating DeFi technology into centralized frameworks.

***

### Chainlink

A decentralized oracle network used by Quantillon as the **fallback** EUR/USD source and for USDC/USD collateral validation. The active EUR/USD source is the market mid of the active hedge venue (currently Hyperliquid) - see the Oracle Architecture page.

***

### Overcollateralization

A system where the value of pledged collateral exceeds the value of the issued loan or stablecoin.\
**Minting QEURO requires the protocol collateralization ratio to stay above the governance-set minting floor - currently 102.5% (since 2 September 2026; 105% at launch); 101% is the critical threshold that triggers liquidation mode.**

***

### Composability

The ability of DeFi protocols to integrate and interact seamlessly, enabling the creation of complex and composable financial products.

***

### DAO (Decentralized Autonomous Organization)

An organization governed by smart contracts on a blockchain, with no centralized control.\
Quantillon aims for **full DAO governance** over time.

***

### DeFi (Decentralized Finance)

A financial ecosystem built on blockchain technology, operating without traditional intermediaries.

***

### Delta-Neutral Hedging

A risk-mitigation strategy aiming to make a position **neutral to price changes** in the underlying asset (e.g., EUR/USD exposure in Quantillon).

***

### EUR/USD

The Euro/US Dollar currency pair. A key factor in volatility and currency risk for the QEURO deployment and for EUR users exposed to USD-based DeFi.

***

### Forex (Foreign Exchange Market)

The global market for currency trading. Quantillon leverages its depth and liquidity.

***

### Hedgers

Participants who provide USDC to hedge against EUR/USD volatility, in exchange for compensation via the Yield Shift mechanism. In the current phase a single designated hedger (Quantillon Labs' hedging engine, executing on Hyperliquid) fills this role - see the HedgerPool page.

***

### Hyperliquid

A decentralized perpetual-futures exchange. Currently the protocol's **active venue** for EUR/USD hedge execution, whose market mid is the active on-chain EUR/USD pricing source - see the Oracle Architecture page.

***

### KPI (Key Performance Indicator)

Metrics used to assess the protocol’s growth and performance.\
Examples: **TVL**, **swap volume**, **user base**.

***

### Liquidity by Design

Quantillon's strategy of leveraging **existing liquidity** (USDC and Forex) instead of building new liquidity from scratch - reducing costs and slippage.

***

### Margin Rebalancing

The operational policy, in force since September 2026, under which Quantillon Labs' hedging engine keeps the collateral of the two legs of the EUR/USD hedge - the HedgerPool position on Base and the Hyperliquid perpetual - near a 2.5% equity-to-notional target through bounded, monitored USDC transfers. The on-chain HedgerPool minimum margin ratio is 250 bps (2.5%) and the minting floor is 102.5% - see the HedgerPool page.

***

### Livret A

A low-yield regulated savings product in France, often cited as an example of how European capital is passively “parked”.

***

### MiCA (Markets in Crypto-Assets Regulation)

The EU regulation establishing a legal framework for crypto-assets and their service providers.

***

### Recital 22

A clause in MiCA that exempts protocols which are **fully decentralized** and **not controlled by a legal entity** from some regulatory obligations.

***

### QEURO

Quantillon’s euro-pegged stablecoin, overcollateralized and backed primarily by USDC.

***

### $QTI Token

Quantillon’s native governance token. **Dormant** (deployed with a 100M supply cap, live supply 0, no mint path). Intended for protocol governance once activated; until then the protocol is governed by a 2-of-3 Safe with a 12-hour upgrade timelock.

***

### Quantillon Rewards (QP)

An off-chain loyalty program operated by Quantillon Labs that records points ("QP") for QEURO depositors and stakers. QP have no monetary value and confer no right to any token or allocation. The terms are published (Rewards Program Terms); the program is not yet open in the application.

***

### RWAs (Real World Assets)

Tokenized representations of real-world financial instruments (e.g., Treasury Bills). Not used by the protocol today; an RWA-backed venue could be onboarded as an external staking vault in a future phase, subject to governance.

***

### Stablecoin

A cryptocurrency pegged to a stable asset (like fiat or commodities), designed to reduce volatility.

***

### TVL (Total Value Locked)

The total value of assets locked within the protocol.\
A key indicator of Quantillon’s adoption and capitalization.

***

### USDC

A USD-pegged stablecoin used as **primary collateral** in the Quantillon Protocol.

***

### Users

Participants who mint or stake QEURO to gain euro exposure and earn yield.

***

### Vaults

Risk-segmented architecture modules where users deposit collateral across various DeFi backends (currently a Morpho USDC vault; other backends possible in future phases).

***

### veQTI (Vote-Escrowed QTI)

A staking model where $QTI tokens are **locked** for a fixed time in exchange for:

* Increased voting power
* Long-term alignment incentives

***

### Yield Shift

Quantillon’s unique mechanism for **dynamically redistributing yield** from collateral between Users and Hedgers to maintain the stablecoin’s peg.

***

> 💡 Want to dive deeper into any of these terms? Use the GitBook search or navigate through the sidebar for more detailed sections.
