# stQEURO Token

## stQEURO Tokenomics: Yield-Bearing Euro Infrastructure

### 📋 Executive Summary

The Staked Quantillon Euro (stQEURO) represents the next evolution in euro-denominated yield-bearing tokens, designed to automatically compound returns from QEURO collateral deployment while maintaining full liquidity and composability. stQEURO creates a seamless bridge between passive euro exposure and active DeFi yield generation.

Our yield-bearing architecture incorporates cutting-edge mechanisms including automatic compounding functionality via exchange rate appreciation, cross-protocol composability, and integration with the YieldShift system that delivers superior user experience while maintaining security. The design prioritizes capital efficiency, regulatory compliance, and sustainable auto-compounding across diverse market conditions.

***

### Core Token Specifications

**Technical Architecture**

| Parameter | Value | Rationale |
| --------- | ----- | --------- |
| **Name** | Staked Quantillon Euro | Clear staking utility |
| **Symbol** | stQEURO | Intuitive yield-bearing prefix |
| **Standard** | ERC-4626 vault, ERC-20 compatible (UUPS Upgradeable) | Standard vault API + auto-compounding |
| **Network** | Base L2 (Primary) | L2 efficiency and lower costs |
| **Exchange Rate** | Dynamic (yield-adjusted) | Automatic yield accumulation |
| **Decimals** | 18 | Full ERC-20 compatibility |
| **Contract Type** | OpenZeppelin + Custom Logic | Battle-tested + innovation |

**Implemented Features**

* **Automatic Compounding**: Value appreciation via exchange rate mechanism
* **Instant Liquidity**: No lock periods for staking/unstaking
* **YieldShift Integration**: Dynamic yield allocation from protocol yields
* **Oracle Integration**: Real-time yield calculation from underlying sources
* **Emergency Controls**: Pausable with emergency withdrawal capability

> **🚧 Roadmap Features**: Cross-chain staking (Arbitrum, Optimism) is planned for future phases.

***

### Yield-Bearing Mechanics

**🔄 Token Architecture**

**Auto-Compounding Model (Exchange Rate Appreciation)**

Unlike traditional staking where users must manually claim rewards, stQEURO automatically increases in intrinsic value over time. The quantity of stQEURO tokens in user wallets remains constant, but each token becomes worth more QEURO as yields accumulate.

```
stQEURO Value = QEURO Amount × Exchange Rate

Where:
- Exchange Rate starts at 1.0
- Exchange Rate increases as yield is distributed
- No token rebasing or inflation
```

**📊 Yield Calculation**

```
New Exchange Rate = Current Exchange Rate + (Yield Amount / Total stQEURO Supply)
```

**Example Timeline:**

* **Day 0**: Stake 1,000 QEURO → Receive 1,000 stQEURO (Rate: 1.000)
* **Day 365**: Still hold 1,000 stQEURO (Rate: 1.053 with 5.3% APY)
* **Unstaking**: 1,000 stQEURO → 1,053 QEURO

**⚡ Key Benefits**:

* **No Token Inflation**: stQEURO quantity remains fixed in wallets
* **Value Appreciation**: Each stQEURO becomes worth more QEURO over time
* **Composable**: Use stQEURO in other DeFi protocols while value appreciates
* **Tax Efficient**: No rebase events creating potential taxable income

**💰 Yield Source & Distribution**

**Revenue Flow Architecture**

All protocol USDC (hedger collateral + the USDC backing user mints) is pooled into a single curated
yield strategy (Morpho on Base, via a vault adapter). The gross yield is split **in this order**:

```
Gross yield realized from the strategy (variable APY)
│
├── 1. Hedger funding (FIRST) — an absolute, time-prorated funding rate on the hedged
│       notional (governance-set; 0 at launch). Covers the cost of maintaining the hedge.
│
└── 2. Residual (= gross − hedger funding), split by the staking ratio:
        ├── stQEURO holders: residual × (staked QEURO / circulating QEURO)
        │     → credited into stQEURO, raising the exchange rate (auto-compounded)
        └── Treasury: the remainder (yield attributable to unstaked QEURO)
```

> The hedger share is an **absolute funding cost** (reflecting the perp funding rate), not a fixed
> percentage of yield. Only the **staked** fraction of QEURO earns the residual; the unstaked share
> accrues to the treasury. The protocol is only viable when the strategy yield exceeds the hedging
> cost. At launch the funding rate is **0** (Quantillon is the sole hedger and bootstraps at a loss),
> so the full residual flows to stakers and treasury.

**📊 Value Appreciation Estimates**

| Market Condition | Strategy APY | Estimated Net stQEURO APY | 1 stQEURO Value After 1 Year |
| ---------------- | ------------ | ------------------------- | ---------------------------- |
| **Bear Market** | 4% | 2-3% | ~1.025 QEURO |
| **Normal** | 7% | 4-6% | ~1.050 QEURO |
| **Bull Market** | 12% | 8-10% | ~1.090 QEURO |

> **Note**: Actual APY depends on the external strategy's market conditions (currently Morpho USDC lending on Base), the hedger funding carve-out, and the per-series yield fee. These are estimates, not guarantees.

***

### Technical Parameters

**Contract Configuration**

| Parameter | Description | Governance-Adjustable |
|-----------|-------------|----------------------|
| **convertToAssets(1e18)** | Current QEURO per stQEURO (exchange rate) | Auto-updated as yield is credited |
| **totalAssets()** | Total QEURO backing this stQEURO series | Auto-tracked |
| **yieldFee** | Fee on credited yield, per stQEURO series (bps, cap 20%) | ✅ Yes |
| **fundingRateAnnualBps** | Hedger funding carve-out before the staker split (on QuantillonVault) | ✅ Yes |
| **hedgerYieldRecipient** | Recipient of the hedger funding share (on QuantillonVault) | ✅ Yes |

**Access Control Roles**

| Role | Permission | Typical Holder |
|------|------------|----------------|
| **GOVERNANCE_ROLE** | Update parameters, manage settings | Governance/Timelock |
| **YIELD_DISTRIBUTOR_ROLE** (on QuantillonVault) | Harvest + distribute yield, credit QEURO into stQEURO | Keeper / Governance Safe |
| **EMERGENCY_ROLE** | Pause/unpause, emergency withdraw | Emergency multisig |

***

### Staking & Unstaking Operations

stQEURO is an **ERC-4626** vault. Each staking vault has its own non-fungible stQEURO series
(e.g. `stQEUROMORPHO1`), resolved from `stQEUROFactory` by `vaultId`.

**📥 Staking (QEURO → stQEURO)**

```solidity
function deposit(uint256 qeuroAssets, address receiver) external returns (uint256 stQEUROShares);
```

1. User approves QEURO for the vault-specific stQEURO contract
2. User calls `deposit()` — or `mintAndStakeQEURO()` on QuantillonVault to mint + stake in one transaction
3. stQEURO shares are minted at the current exchange rate

**📤 Unstaking (stQEURO → QEURO)**

```solidity
function redeem(uint256 stQEUROShares, address receiver, address owner) external returns (uint256 qeuroAssets);
```

1. User calls `redeem()` with their stQEURO shares
2. QEURO is returned at the current exchange rate (original stake + accrued yield)
3. Instant — no lock period

**Preview / conversion helpers**: `convertToAssets`, `convertToShares`, `previewDeposit`, `previewRedeem` (standard ERC-4626).

***

### How Yield Is Credited

stQEURO yield is credited on-chain by the protocol vault's `harvestAndDistributeVaultYield` function,
which harvests the strategy yield, pays the hedger funding share first, and credits the staked-user
share into stQEURO — raising the exchange rate. The **user** side reaches stakers through the exchange
rate (no claim). YieldShift remains the accounting/claim ledger for the **hedger** side of protocol
yield (`claimHedgerYield`).

```
Yield Distribution Flow:
1. The strategy (Morpho/Aave) generates yield on the pooled USDC.
2. harvestAndDistributeVaultYield realizes that yield into the vault.
3. Hedger funding is carved out first; the residual is split staked / unstaked.
4. The staked share is minted into stQEURO → exchange rate rises automatically.
```

> **No claim, no lock**: stQEURO has no reward-claim call and no holding-period lock. Value accrues to
> the exchange rate, so simply holding stQEURO earns yield, and unstaking is always immediate.

***

### Risk Management & Security

**🛡️ Smart Contract Security**

**Security Architecture**

* **OpenZeppelin Base**: Battle-tested upgradeable contracts
* **Reentrancy Protection**: ReentrancyGuardUpgradeable on all external calls
* **Access Control**: Role-based permissions for all sensitive operations
* **Pausable**: Emergency pause capability for crisis situations
* **UUPS Upgradeable**: Secure upgrade pattern with timelock

**🔍 Security Measures**

| Component | Security Measure |
| --------- | ---------------- |
| **Exchange Rate Calculation** | Checked arithmetic, overflow protection |
| **Yield Distribution** | Only QuantillonVault (`YIELD_DISTRIBUTOR_ROLE`) can credit yield |
| **Oracle Integration** | Yield credit prices QEURO via the vault's active oracle (OracleRouter) |
| **Emergency Response** | Pause + emergency withdrawal |

**⚖️ Economic Risk Considerations**

**Yield Variability**

* Yields depend on the external strategy's market conditions (currently Morpho on Base)
* The hedger funding carve-out (`fundingRateAnnualBps`) is taken before the staker residual
* A per-series yield fee (`yieldFee`, capped at 20%) reduces gross staker yield

> **Note**: The documentation previously mentioned "Floor Protection" (2% APY) and "Ceiling Management" (15% APY). These are NOT currently implemented in the smart contracts. Yields are market-driven.

**📊 Risk Monitoring**

Key metrics to monitor:

* **Collateral Health**: Protocol collateralization ratio
* **Yield Stability**: Aave APY fluctuations
* **Liquidity Depth**: stQEURO/QEURO liquidity availability
* **Exchange Rate**: Consistent appreciation pattern

**🚨 Emergency Procedures**

**Crisis Response Protocol**

* **Level 1**: Pause yield distribution
* **Level 2**: Pause staking (new deposits)
* **Level 3**: Emergency withdrawal (users can exit)
* **Level 4**: Protocol pause (all operations halted)

**User Protection Mechanisms**

* **Instant Unstaking**: Always available regardless of protocol state
* **Emergency Withdraw**: Direct withdrawal bypassing normal flow
* **No Lock Periods**: Users can exit at any time

***

### Tokenomic Model & Sustainability

**💰 Value Accrual**

stQEURO value grows through:

1. **Strategy Yield**: Primary source from the vault's external strategy (currently Morpho USDC lending on Base)
2. **Staker Residual**: The realized yield remaining after the hedger funding carve-out, credited pro-rata to the staked fraction
3. **Compound Effect**: Automatic reinvestment without gas costs

**📈 Growth Factors**

* **TVL Growth**: More staked QEURO = more yield generation
* **Strategy APY**: Higher market rates = higher stQEURO yields
* **Funding Rate**: A lower hedger carve-out leaves a larger staker residual

***

### Integration & Composability

**DeFi Compatibility**

stQEURO is designed as a standard ERC-20 token for maximum composability:

* **Lending Protocols**: Use as collateral in other DeFi protocols
* **DEX Liquidity**: Provide liquidity in stQEURO pairs
* **Yield Aggregators**: Compatible with yield optimization platforms
* **Portfolio Tracking**: Standard ERC-20 for wallet integration

**Technical Integration**

```solidity
interface IstQEURO {
    // Underlying asset (QEURO) and total backing
    function asset() external view returns (address);
    function totalAssets() external view returns (uint256);

    // Exchange-rate conversions (ERC-4626)
    function convertToAssets(uint256 shares) external view returns (uint256 qeuroAssets);
    function convertToShares(uint256 assets) external view returns (uint256 stQEUROShares);
    function previewRedeem(uint256 shares) external view returns (uint256 qeuroAssets);

    // Stake / unstake (ERC-4626)
    function deposit(uint256 assets, address receiver) external returns (uint256 shares);
    function redeem(uint256 shares, address receiver, address owner) external returns (uint256 assets);
}
```

***

### Competitive Positioning

**🥊 Market Comparison**

| Protocol | Token | APY Range | Auto-Compound | Euro Focus | Lock Period |
| -------- | ----- | --------- | ------------- | ---------- | ----------- |
| **Sky** | sUSDS | 4-8% | ✅ | ❌ | None |
| **Lido** | stETH | 3-6% | ✅ | ❌ | None |
| **stQEURO** | stQEURO | 4-10% | ✅ | ✅ | None |

**🎯 Unique Value Propositions**

* **Euro-Native**: First major yield-bearing euro token
* **Instant Liquidity**: No lock periods or withdrawal delays
* **YieldShift**: Dynamic optimization between users and hedgers
* **Regulatory Clarity**: MiCA-compliant design approach

***

### Technical Reference

**Key Events**

* `Deposit(sender, owner, assets, shares)` - User staked QEURO (ERC-4626)
* `Withdraw(sender, receiver, owner, assets, shares)` - User unstaked stQEURO (ERC-4626)
* `VaultYieldDistributed(vaultId, realizedYield, hedgerShare, userShare, treasuryShare)` - yield split (QuantillonVault)
* `YieldParametersUpdated(yieldFee)` - per-series stQEURO fee update

**Contract Dependencies**

* `QEURO`: Underlying stablecoin token (the only asset the contract holds)
* `QuantillonVault`: Yield distributor — realizes strategy yield and credits the staker share as QEURO backing
* `Treasury`: Fee recipient

***

### Conclusion: First-Deployment Yield Infrastructure

stQEURO represents the yield-bearing wrapper for Quantillon's first EUR deployment, offering instant liquidity and automatic compounding around QEURO. Through the exchange rate appreciation model, users can earn yield without token rebasing or manual reward claiming.

The integration with YieldShift ensures dynamic, market-responsive yield distribution that balances the interests of all first-deployment participants. As the Quantillon ecosystem grows, stQEURO serves as the first template for how a deployment-specific local-currency asset can be wrapped into a yield-bearing format.

***

> **Important Disclaimer**: stQEURO is a yield-bearing token that carries inherent risks including smart contract vulnerabilities, market volatility, and variable yields. Past performance does not guarantee future results. The compounding mechanism may result in tax implications depending on your jurisdiction. This document is for informational purposes only and should not be considered investment advice. Users should conduct thorough research before participating in any DeFi protocol.
