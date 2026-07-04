# QEURO Token

## QEURO Tokenomics: First EUR Deployment Architecture

### 📋 Executive Summary

The Quantillon Euro (QEURO) is the first market-specific asset built on the Quantillon Protocol. It combines euro-denominated exposure with USD liquidity depth, protocol-native FX hedging, and DeFi yield infrastructure. QEURO is therefore the first deployment of the broader architecture, not the full definition of Quantillon.

Our stablecoin architecture incorporates advanced mechanisms including overcollateralized minting/redemption via the QuantillonVault, delta-neutral hedging, and external staking vault yield generation (currently Morpho/MetaMorpho) that delivers superior capital efficiency while maintaining regulatory compliance. The design prioritizes user experience, institutional adoption, and sustainable yield generation across diverse market conditions.

***

### Core Token Specifications

**Technical Architecture**

| Parameter         | Value                       | Rationale                      |
| ----------------- | --------------------------- | ------------------------------ |
| **Name**          | Quantillon Euro             | Clear utility identification   |
| **Symbol**        | QEURO                       | Recognizable euro denomination |
| **Standard**      | ERC-20 (UUPS Upgradeable)   | Future-proof with security     |
| **Network**       | Base L2 (Primary)           | L2 efficiency and lower costs  |
| **Peg Target**    | 1 QEURO = 1 EUR             | Direct euro denomination       |
| **Decimals**      | 18                          | Full ERC-20 compatibility      |
| **Max Supply**    | 100,000,000 QEURO           | Governance-adjustable cap      |
| **Contract Type** | OpenZeppelin + Custom Logic | Battle-tested + innovation     |

**Implemented Features**

* **Oracle Integration**: Hyperliquid EUR/USD market mid (active source) with Chainlink as fallback and USDC/USD validation, all behind circuit breakers — see [Oracle Architecture](../oracle-architecture.md)
* **Slippage-Free Operations**: Mint/redeem at oracle rates; fees are currently 0 (governance-settable, capped at 5%)
* **Emergency Controls**: Pausable with time-locked upgrades via UUPS pattern
* **Compliance System**: Blacklist/whitelist functionality for regulatory compliance
* **Rate Limiting**: Protection against large-scale manipulation attacks
* **Minting Killswitch**: Emergency minting halt capability

> **🚧 Roadmap Features**: Cross-chain bridging (Base, Arbitrum, Optimism) and multi-collateral support (ETH, WBTC) are planned for future phases.

***

### Stablecoin Mechanics & Architecture

**Overcollateralization Model**

**📊 Collateral Framework (MVP)**

| Collateral Type | Status | Minimum Ratio | Accepted Assets |
| --------------- | ------ | ------------- | --------------- |
| **Primary**     | ✅ Live | 105%+ (minting floor; 101% is the critical/liquidation threshold) | USDC |

> **Note**: Multi-collateral support (ETH, WBTC, governance-approved assets) is planned for future protocol upgrades.

**🔒 Security Mechanisms**

* **Real-time Monitoring**: Oracle staleness checks (Hyperliquid feed: 15-minute default; Chainlink fallback: 2 hours EUR/USD, 25 hours USDC/USD)
* **Price Deviation Protection**: 5% maximum price change tolerance
* **Circuit Breakers**: Automatic pause on oracle failures or extreme conditions
* **Multi-Sig Controls**: Time-locked upgrades for critical parameter changes
* **Compliance Controls**: Blacklist/whitelist address management

**Minting & Redemption Process**

**📥 Minting Workflow (MVP)**

```
User USDC → QuantillonVault → Oracle Price Check → QEURO Mint → External Vault Deployment
```

1. **USDC Deposit**: Users deposit USDC to the QuantillonVault
2. **Oracle Price Check**: Real-time EUR/USD rate verification via the oracle router (Hyperliquid mid, Chainlink fallback)
3. **Collateral Lock**: Protocol enforces the 105% minimum collateralization ratio for minting
4. **QEURO Issuance**: Vault mints QEURO at the oracle rate; the minting fee is currently 0 (governance-settable, max 5%)
5. **Yield Deployment**: Collateral can be deployed to external staking vaults (currently Morpho/MetaMorpho)

> **Important**: Minting occurs via the QuantillonVault contract which holds the MINTER_ROLE. Users interact through the Vault interface, not directly with the QEURO token contract.

**📤 Redemption Workflow**

```
QEURO Burn → Oracle Verification → Collateral Release → USDC Transfer
```

**Key Benefits**:

* **Zero Slippage**: Oracle-based pricing eliminates DEX impact
* **24/7 Operations**: No banking hours or geographic restrictions
* **Instant Settlement**: Single-block transaction finality
* **Open Access**: Any address can deposit/redeem via the Vault

**Yield Generation Strategy**

**💰 Collateral Deployment**

USDC collateral can be deployed to external staking vaults — currently a MetaMorpho (Morpho) USDC vault via a dedicated adapter — to generate yield. Deployment is managed by governance, and emergency withdrawal is available for crisis situations. See [External Staking Vaults](../aave-vault.md) for details.

**Revenue Distribution Model (Harvest + YieldShift)**

```
External Vault Yield (Variable APY), harvested by QuantillonVault
├── 1. Hedger funding paid first
│      (governance-set annual rate, capped at 50% of each harvest)
└── 2. Residual split between stQEURO stakers and the treasury
       according to the staked share
```

> **Note**: On top of the harvest split, the YieldShift mechanism governs the user/hedger allocation layer of the yield pools based on pool utilization ratios. The base shift is 50% to users, with a maximum of 90%.

***

### Dual-Pool Architecture

**👥 User Pool Mechanics**

**Primary Functions**:

* **QEURO Minting**: Deposit USDC via Vault to mint QEURO
* **Yield Earning**: Stake QEURO to stQEURO to receive auto-compounding yields
* **Governance Participation**: Vote on protocol parameters via QTI
* **Unstaking Cooldown**: Configurable cooldown period for unstaking

**Participation Requirements**:

* **Minimum Stake**: Configurable via governance (minStakeAmount)
* **Collateral Ratio**: Minting requires 105%+ protocol collateralization (101% is the critical threshold)
* **Holding Period**: 7-day minimum for yield claims (anti-manipulation)

**🛡️ Hedger Pool Mechanics**

**Current Implementation: Single Hedger Model**

> **Important**: The MVP implements a single designated hedger model for simplified operations. Multi-hedger support is planned for future phases.

**Hedger Functions**:

* **FX Risk Management**: Delta-neutral EUR/USD exposure via margin positions
* **USDC Provision**: Deposits USDC margin for protocol collateralization
* **Yield Optimization**: Earns compensation from yield shift allocation
* **Peg Maintenance**: Economic incentives to maintain EUR stability

**Compensation Structure**:

```
Base Compensation: EUR/USD Interest Rate Differential
+ Variable Yield Shift: Based on pool utilization ratios
= Total Compensation (configurable via governance)
```

**Risk Management**:

* **Margin Requirements**: Configurable minimum margin ratio (minMarginRatio)
* **Leverage Limits**: Maximum leverage configurable by governance (maxLeverage)
* **Auto-Liquidation**: Triggered when position becomes unhealthy
* **Entry/Exit Fees**: Configurable fees for position management

***

### Yield Shift Mechanism

**⚖️ Dynamic Equilibrium System**

The Yield Shift represents QEURO's most innovative feature—automatically rebalancing incentives between Users and Hedgers based on real-time market conditions.

**📊 Technical Parameters (Code Values)**

| Parameter | Value | Description |
|-----------|-------|-------------|
| **Base Yield Shift** | 50% (5000 bps) | Default allocation to users |
| **Max Yield Shift** | 90% (9000 bps) | Maximum allocation to users |
| **Adjustment Speed** | 100 bps | Rate of shift adjustment |
| **Target Pool Ratio** | 100% (10000 bps) | Optimal user/hedger ratio |
| **TWAP Period** | 24 hours | Time-weighted average window |
| **Min Holding Period** | 7 days | Required for yield claims |

**🎯 How It Works**

| Pool Condition | Yield Shift Direction | User Impact | Hedger Impact |
| -------------- | --------------------- | ----------- | ------------- |
| **High User Pool** | Shift toward hedgers | Lower yields | Higher compensation |
| **Balanced Pools** | Neutral | Standard yields | Standard rates |
| **High Hedger Pool** | Shift toward users | Higher yields | Lower compensation |

**⏱️ Response Mechanism**:

* **Automatic Adjustments**: Based on 24-hour TWAP calculations
* **Gradual Changes**: Adjustment speed limits sudden shifts
* **Governance Control**: Parameters adjustable via QTI governance

***

### Vault Architecture

**🏗️ Current Implementation**

**External Staking Vault (Live)**

| Parameter | Value | Description |
|-----------|-------|-------------|
| **Backend** | MetaMorpho (Morpho) USDC vault | Primary yield source, accessed via `MetaMorphoStakingVaultAdapter` (vaultId 2) |
| **Staked series** | stQEUROMORPHO1 | Per-vault stQEURO series deployed by `stQEUROFactory` |
| **Risk Profile** | 🟢 Low | Established DeFi protocol |
| **Target APY** | Market-dependent | Variable Morpho lending yield |

> **🚧 Future Vault Variants** (Roadmap): additional external staking vaults (other lending markets, tokenized T-Bills/RWAs, advanced strategies) can be added by governance via the adapter and factory pattern.

***

### Compliance & Security Features

**🔐 Access Control Roles**

| Role | Permission | Typical Holder |
|------|------------|----------------|
| **DEFAULT_ADMIN_ROLE** | Grant/revoke roles, critical admin | Multisig |
| **MINTER_ROLE** | Mint new QEURO | QuantillonVault |
| **BURNER_ROLE** | Burn QEURO | QuantillonVault |
| **PAUSER_ROLE** | Pause/unpause | Emergency multisig |
| **COMPLIANCE_ROLE** | Manage blacklist/whitelist | Compliance team |

***

### 🚦 Rate Limiting System

The rate limiting system protects the protocol against large-scale manipulation attacks by limiting mint/burn volume per address over a given period.

**Technical Implementation**

```
// Rate Limiting (live values)
Mint rate limit: 10,000,000 QEURO per 300-block window (~10 minutes on Base)
```

**Mechanism**

1. **Per-address tracking**: Each address has its own mint/burn limit
2. **Sliding window**: The limit resets after each 300-block window (~10 minutes on Base)
3. **Accumulation**: Operations accumulate within the current window
4. **Blocking**: If cumulative total exceeds the limit, operation fails

**Configuration**

| Parameter | Live Value | Governable |
|-----------|------------|------------|
| Mint rate limit | 10,000,000 QEURO per window | ✅ Yes |
| Rate limit window | 300 seconds | ❌ Constant |

**Use Cases**

- **Anti-whale protection**: Prevents an actor from minting/burning massively
- **Operation smoothing**: Distributes load over time
- **Anomaly detection**: Bypass attempts can be monitored

**Associated Functions**

```solidity
// Check and update mint rate limit
function _checkAndUpdateMintRateLimit(address account, uint256 amount) internal;

// Check and update burn rate limit
function _checkAndUpdateBurnRateLimit(address account, uint256 amount) internal;
```

***

### 🔴 Minting Killswitch

The Minting Killswitch is an emergency mechanism that instantly halts all minting operations on the protocol.

**Implementation**

```solidity
// Killswitch state
bool public mintingKillswitch;

// Enable/disable (admin only)
function setMintingKillswitch(bool enabled) external onlyRole(DEFAULT_ADMIN_ROLE);
```

**When to Use?**

| Situation | Action | Consequence |
|-----------|--------|-------------|
| **Vulnerability detected** | Enable killswitch | No more QEURO can be minted |
| **Oracle compromised** | Enable killswitch | Prevents minting at wrong prices |
| **Attack in progress** | Enable killswitch | Stops exploitation immediately |
| **Situation resolved** | Disable killswitch | Resumes normal operations |

**Impact**

- ✅ **Burns remain possible**: Users can still exit
- ✅ **Transfers remain possible**: QEURO remains transferable
- ✅ **Redemptions remain possible**: Via the Vault (which burns)
- ❌ **Minting blocked**: No new QEURO issuance

**Difference with Pause**

| Function | Mint | Burn | Transfer | Redeem |
|----------|------|------|----------|--------|
| **Killswitch ON** | ❌ | ✅ | ✅ | ✅ |
| **Pause ON** | ❌ | ❌ | ❌ | ❌ |

> **Note**: The killswitch is a less drastic measure than full pause, allowing users to exit the protocol.

***

### 📋 Compliance System (Blacklist/Whitelist)

The QEURO protocol integrates a compliance system to meet regulatory requirements, including MiCA.

**System Architecture**

```
┌─────────────────────────────────────────────────────┐
│                  COMPLIANCE SYSTEM                   │
├─────────────────────────────────────────────────────┤
│  Normal Mode (whitelistEnabled = false)             │
│  ├── Everyone can transfer                          │
│  └── Except blacklisted addresses                   │
├─────────────────────────────────────────────────────┤
│  Whitelist Mode (whitelistEnabled = true)           │
│  ├── Only whitelisted addresses can transfer        │
│  └── Blacklisted remain excluded                    │
└─────────────────────────────────────────────────────┘
```

**Compliance Mappings**

```solidity
mapping(address => bool) public isBlacklisted;   // Blocked addresses
mapping(address => bool) public isWhitelisted;   // Authorized addresses
bool public whitelistEnabled;                     // Whitelist mode active
```

**Management Functions**

| Function | Required Role | Description |
|----------|---------------|-------------|
| `blacklistAddress(address)` | COMPLIANCE_ROLE | Block an address |
| `unblacklistAddress(address)` | COMPLIANCE_ROLE | Unblock an address |
| `whitelistAddress(address)` | COMPLIANCE_ROLE | Authorize an address |
| `unwhitelistAddress(address)` | COMPLIANCE_ROLE | Remove authorization |
| `toggleWhitelistMode()` | COMPLIANCE_ROLE | Enable/disable whitelist mode |

**Batch Operations**

For gas efficiency, batch operations are available:

```solidity
function batchBlacklistAddresses(address[] calldata accounts) external;
function batchUnblacklistAddresses(address[] calldata accounts) external;
function batchWhitelistAddresses(address[] calldata accounts) external;
function batchUnwhitelistAddresses(address[] calldata accounts) external;
```

**Verification Logic**

On each transfer, the `_update` function verifies:

```
1. Sender is not blacklisted
2. Recipient is not blacklisted
3. If whitelistEnabled:
   - Sender must be whitelisted (except for minting)
   - Recipient must be whitelisted (except for burning)
```

**Regulatory Use Cases**

| Scenario | Configuration | Result |
|----------|---------------|--------|
| **Open public** | whitelistEnabled=false | Everyone can transfer except blacklisted |
| **OFAC sanctions** | Blacklist sanctioned addresses | Sanctions compliance |
| **KYC required** | whitelistEnabled=true | Only KYC'd users can use |
| **Fund freeze** | Blacklist specific address | Funds frozen for investigation |

**Emitted Events**

```solidity
event BlacklistUpdated(address indexed account, bool status);
event WhitelistUpdated(address indexed account, bool status);
event WhitelistModeToggled(bool enabled);
```

***

### ⚠️ Emergency Controls

**Emergency Controls Overview**

| Control | Severity | Required Role | Reversible |
|---------|----------|---------------|------------|
| **Rate Limit** | 🟡 Medium | Automatic | ✅ Auto-reset |
| **Killswitch** | 🟠 High | ADMIN | ✅ Yes |
| **Pause** | 🔴 Critical | PAUSER | ✅ Yes |
| **Blacklist** | 🟡 Targeted | COMPLIANCE | ✅ Yes |

**Escalation Procedure**

```
Level 1: Rate Limiting (automatic)
    ↓ If insufficient
Level 2: Minting Killswitch (admin)
    ↓ If insufficient  
Level 3: Targeted Blacklist (compliance)
    ↓ If insufficient
Level 4: Full Pause (pauser/admin)
```

**Token Recovery**

In case of tokens or ETH sent by mistake to the contract:

```solidity
// Recover ERC20 tokens
function recoverToken(address token, uint256 amount) external onlyRole(DEFAULT_ADMIN_ROLE);

// Recover ETH
function recoverETH() external onlyRole(DEFAULT_ADMIN_ROLE);
```

> Recovered funds are sent to the configured `treasury` address.

***

### Economic Sustainability Model

**📈 Revenue Streams**

**Primary Revenue Sources**

1. **Mint/Redeem Fees**: currently 0 (governance-settable, capped at 5%); when collected, routed to the FeeCollector (60% treasury / 25% dev fund / 15% community)
2. **Yield Management**: stQEURO yield fee (currently 0, max 20%) and the treasury share of harvested external-vault yield
3. **Position Fees**: hedger entry/exit/margin fees (currently 0, governance-settable) plus a 20% reward fee split on hedger rewards

> **Note**: Cross-chain bridge fees and additional vault fees are planned for future implementations.

**🎯 Key Performance Indicators**

**Health Metrics**

* **Peg Stability**: Target <2% deviation from EUR
* **Collateral Ratio**: Maintain ≥105% (minting floor) across all conditions; 101% is the critical threshold
* **Yield Consistency**: Dynamic based on external staking vault (Morpho) market conditions
* **Governance Activity**: QTI holder participation

***

### Risk Management Framework

**🛡️ Comprehensive Risk Matrix**

**Technical Risks**

| Risk Factor | Probability | Impact | Mitigation Strategy |
| ----------- | ----------- | ------ | ------------------- |
| **Smart Contract Bug** | Medium | Critical | Multiple audits, formal verification |
| **Oracle Manipulation** | Low | High | Chainlink + circuit breakers, 5% deviation limit |
| **External Vault (Morpho) Risk** | Low | Medium | Emergency withdrawal, governance-controlled deployment |
| **Liquidation Cascade** | Low | High | Circuit breakers, emergency pause |

**Market Risks**

| Risk Factor | Probability | Impact | Mitigation Strategy |
| ----------- | ----------- | ------ | ------------------- |
| **EUR/USD Volatility** | High | Medium | Delta-neutral hedging, yield shift |
| **USDC Depeg** | Low | High | 2% tolerance monitoring, circuit breaker |
| **Interest Rate Changes** | Medium | Low | Dynamic yield adjustment |
| **Regulatory Changes** | Medium | High | MiCA compliance, blacklist capability |

**Emergency Procedures**

**Crisis Response Protocol**

* **Level 1**: Automatic circuit breakers (oracle pause)
* **Level 2**: Emergency role intervention (pause operations)
* **Level 3**: Admin intervention (parameter changes)
* **Level 4**: Protocol pause (complete system halt)

**Recovery Mechanisms**

* **Token Recovery**: Admin can recover accidentally sent tokens
* **ETH Recovery**: Admin can recover accidentally sent ETH
* **Governance Override**: Emergency parameter adjustments via governance

***

### Technical Reference

**Contract Addresses**

> The protocol is live on Base mainnet (chain 8453) since June 2026. QuantillonVault: `0x833E5Ba510a241b21F1C60c987D1c49eB52E4a07`. See [Oracle Architecture](../oracle-architecture.md) for the oracle contract addresses.

**Key Constants**

```solidity
MAX_SUPPLY = 100_000_000e18   // 100 million QEURO (governance-adjustable cap)
// Mint/redeem fees: currently 0, governance-settable, capped at 5%
// Mint rate limit: 10,000,000 QEURO per 300-block window (~10 minutes on Base)
```

**Events**

* `Transfer(from, to, amount)` - Standard ERC20 transfer
* `Mint(to, amount)` - QEURO minted
* `Burn(from, amount)` - QEURO burned
* `BlacklistUpdated(account, status)` - Compliance update
* `MintingKillswitchSet(enabled)` - Emergency control activated

***

#### Conclusion: QEURO as the First Deployment

QEURO represents more than just another stablecoin: it is the first production deployment of Quantillon's FX-hedged local-currency architecture. Through innovative dual-pool mechanics, dynamic yield redistribution via YieldShift, and robust security controls, QEURO creates a sustainable foundation for EUR-denominated decentralized finance.

The live deployment focuses on core functionality with USDC collateral and external staking vault yield generation (currently Morpho/MetaMorpho). Future phases will expand to multi-collateral support, additional vault strategies, and cross-chain deployment.

The first deployment prioritizes user experience, regulatory compliance, and sustainable yield generation while preserving the broader protocol's flexibility. The result is an EUR asset that bridges local-currency balance-sheet needs with the innovation and accessibility of decentralized protocols.

***

> **Risk Disclaimer**: QEURO is a decentralized financial product that carries inherent risks including smart contract vulnerabilities, market volatility, and regulatory changes. This document is for informational purposes only and should not be considered investment advice. Users should conduct their own research and risk assessment before participating in the protocol.
