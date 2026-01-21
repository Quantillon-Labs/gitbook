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
| **Standard** | ERC-20 (UUPS Upgradeable) | Auto-compounding functionality |
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

```
QEURO Collateral Deployed to Aave v3
├── Gross Yield Generated (Variable APY: 4-12%)
│
├── Protocol Fee: 10% (to Treasury)
│
└── Net Yield Distribution via YieldShift
    ├── stQEURO Holders: 50-90% (auto-compounded)
    └── Hedgers: 10-50% (yield shift allocation)
```

**📊 Value Appreciation Estimates**

| Market Condition | Aave APY | Estimated Net stQEURO APY | 1 stQEURO Value After 1 Year |
| ---------------- | -------- | ------------------------- | ---------------------------- |
| **Bear Market** | 4% | 2-3% | ~1.025 QEURO |
| **Normal** | 7% | 4-6% | ~1.050 QEURO |
| **Bull Market** | 12% | 8-10% | ~1.090 QEURO |

> **Note**: Actual APY depends on Aave market conditions, YieldShift allocation, and protocol fee deductions. These are estimates, not guarantees.

***

### Technical Parameters

**Contract Configuration**

| Parameter | Description | Governance-Adjustable |
|-----------|-------------|----------------------|
| **exchangeRate** | Current QEURO per stQEURO | Auto-updated |
| **totalUnderlying** | Total QEURO value backing stQEURO | Auto-tracked |
| **yieldFee** | Protocol fee on distributed yield | ✅ Yes |
| **minYieldThreshold** | Minimum yield for distribution | ✅ Yes |
| **maxUpdateFrequency** | Maximum rate of exchange rate updates | ✅ Yes |

**Access Control Roles**

| Role | Permission | Typical Holder |
|------|------------|----------------|
| **GOVERNANCE_ROLE** | Update parameters, manage settings | Governance/Timelock |
| **YIELD_MANAGER_ROLE** | Distribute yield, trigger compounding | YieldShift contract |
| **EMERGENCY_ROLE** | Pause/unpause, emergency withdraw | Emergency multisig |

***

### Staking & Unstaking Operations

**📥 Staking (QEURO → stQEURO)**

```solidity
function stake(uint256 qeuroAmount) external returns (uint256 stQEUROAmount);
```

1. User approves QEURO for stQEURO contract
2. User calls `stake()` with QEURO amount
3. Contract calculates stQEURO based on current exchange rate
4. QEURO transferred to contract, stQEURO minted to user

**📤 Unstaking (stQEURO → QEURO)**

```solidity
function unstake(uint256 stQEUROAmount) external returns (uint256 qeuroAmount);
```

1. User calls `unstake()` with stQEURO amount
2. Contract calculates QEURO based on current exchange rate
3. stQEURO burned, QEURO transferred to user
4. User receives original stake + accumulated yield

**Batch Operations**

* `batchStake(amounts[])` - Stake multiple amounts efficiently
* `batchUnstake(amounts[])` - Unstake multiple amounts efficiently

***

### YieldShift Integration

**Dynamic Yield Distribution**

stQEURO yield is distributed through the YieldShift mechanism which balances incentives between users and hedgers:

```
Yield Distribution Flow:
1. Aave generates yield on USDC collateral
2. AaveVault harvests yield (minus 10% protocol fee)
3. YieldShift receives net yield
4. YieldShift distributes to stQEURO based on current shift ratio
5. stQEURO exchange rate increases automatically
```

**Holding Period Requirement**

> **Important**: The YieldShift mechanism enforces a **7-day minimum holding period** for yield claims. This prevents flash deposit attacks and ensures fair yield distribution.

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
| **Yield Distribution** | Only authorized YieldShift can update |
| **Oracle Integration** | Via YieldShift with Chainlink feeds |
| **Emergency Response** | Pause + emergency withdrawal |

**⚖️ Economic Risk Considerations**

**Yield Variability**

* Yields depend on Aave market conditions (supply/demand)
* YieldShift allocation varies based on pool ratios
* Protocol fees reduce gross yield by 10%

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

1. **Aave Yield**: Primary source from USDC lending
2. **YieldShift Allocation**: Dynamic share of protocol yield
3. **Compound Effect**: Automatic reinvestment without gas costs

**📈 Growth Factors**

* **TVL Growth**: More staked QEURO = more yield generation
* **Aave APY**: Higher market rates = higher stQEURO yields
* **YieldShift Ratio**: Favorable allocation to users = higher returns

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
    // Read current exchange rate
    function exchangeRate() external view returns (uint256);
    
    // Convert amounts
    function getQEUROForStQEURO(uint256 stQEUROAmount) external view returns (uint256);
    function getStQEUROForQEURO(uint256 qeuroAmount) external view returns (uint256);
    
    // Stake/Unstake
    function stake(uint256 qeuroAmount) external returns (uint256 stQEUROAmount);
    function unstake(uint256 stQEUROAmount) external returns (uint256 qeuroAmount);
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

* `Staked(user, qeuroAmount, stQEUROAmount)` - User staked QEURO
* `Unstaked(user, stQEUROAmount, qeuroAmount)` - User unstaked stQEURO
* `YieldDistributed(yieldAmount, newExchangeRate)` - Yield added to exchange rate
* `ParametersUpdated(yieldFee, minThreshold)` - Governance update

**Contract Dependencies**

* `QEURO`: Underlying stablecoin token
* `YieldShift`: Yield distribution controller
* `USDC`: Yield denomination (converted via exchange)
* `Treasury`: Fee recipient

***

### Conclusion: Pioneering Euro Yield Infrastructure

stQEURO represents a fundamental advancement in European DeFi infrastructure, offering a truly euro-native yield-bearing token with instant liquidity and automatic compounding. Through the innovative exchange rate appreciation model, users can earn yield without token rebasing or manual reward claiming.

The integration with YieldShift ensures dynamic, market-responsive yield distribution that balances the interests of all protocol participants. As the Quantillon ecosystem grows, stQEURO will serve as the primary vehicle for passive euro yield generation in DeFi.

***

> **Important Disclaimer**: stQEURO is a yield-bearing token that carries inherent risks including smart contract vulnerabilities, market volatility, and variable yields. Past performance does not guarantee future results. The compounding mechanism may result in tax implications depending on your jurisdiction. This document is for informational purposes only and should not be considered investment advice. Users should conduct thorough research before participating in any DeFi protocol.
