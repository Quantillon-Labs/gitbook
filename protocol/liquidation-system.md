# Liquidation System

## Liquidation System: Protocol Solvency Protection

### 📋 Overview

Quantillon's liquidation system protects protocol solvency by allowing users to exit even when collateralization is insufficient. Unlike traditional DeFi liquidations, Quantillon uses a **pro-rata distribution model** that fairly distributes losses among all participants.

***

### 🎯 When is Liquidation Triggered?

#### Critical Threshold

```solidity
uint256 public constant CRITICAL_COLLATERALIZATION_RATIO_BPS = 10100; // 101%
```

| Protocol State | Collateralization Ratio | Mode |
|----------------|------------------------|------|
| **Healthy** | > 105% | Normal minting and redemption |
| **Under Watch** | 101% - 105% | Limited minting |
| **Liquidation** | ≤ 101% | Liquidation mode activated |

#### Verification

```solidity
function shouldTriggerLiquidation() public returns (bool shouldLiquidate) {
    uint256 currentRatio = getProtocolCollateralizationRatio();
    return currentRatio < criticalCollateralizationRatio;
}
```

***

### 🔄 How Liquidation Mode Works

#### Automatic Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    LIQUIDATION FLOW                              │
├─────────────────────────────────────────────────────────────────┤
│  1. User calls redeemQEURO(amount, minUsdcOut)                  │
│                                                                  │
│  2. Vault checks Collateralization Ratio                        │
│     └── CR = (totalUSDC × 100) / (totalQEURO × eurUsdPrice)     │
│                                                                  │
│  3. IF CR > 101%:                                                │
│     └── Normal Mode: redemption at oracle price                  │
│                                                                  │
│  4. IF CR ≤ 101%:                                                │
│     └── Liquidation Mode: pro-rata distribution                  │
│         ├── Calculation: payout = (amount / totalSupply) × totalUSDC │
│         ├── Hedger absorbs proportional loss                     │
│         └── User receives their share of actual collateral       │
└─────────────────────────────────────────────────────────────────┘
```

#### Difference: Normal Mode vs Liquidation

| Aspect | Normal Mode | Liquidation Mode |
|--------|-------------|------------------|
| **Price** | Oracle EUR/USD price | Pro-rata of actual collateral |
| **Guarantee** | 1 QEURO = EUR value in USDC | 1 QEURO = share of pool |
| **Hedger** | Normal P&L | Proportional margin loss |
| **Exit** | Guaranteed value | Possible haircut if CR < 100% |

***

### 📐 Calculation Formulas

#### Pro-Rata Distribution

```
userPayout = (qeuroAmount / totalQEUROSupply) × totalVaultUSDC
```

**Example with CR = 95%** (undercollateralized)

```
Protocol state:
- totalQEUROSupply = 1,000,000 QEURO
- totalVaultUSDC = 950,000 USDC (CR = 95%)
- EUR/USD price = 1.00

User wants to redeem 10,000 QEURO:

Normal Mode (if available):
  payout = 10,000 × 1.00 = 10,000 USDC ❌ (not enough collateral)

Liquidation Mode:
  payout = (10,000 / 1,000,000) × 950,000 = 9,500 USDC ✅
  → 5% haircut (reflects protocol undercollateralization)
```

**Example with CR = 102%** (overcollateralized but < 105%)

```
Protocol state:
- totalQEUROSupply = 1,000,000 QEURO
- totalVaultUSDC = 1,020,000 USDC (CR = 102%)
- EUR/USD price = 1.00

User wants to redeem 10,000 QEURO:

Liquidation Mode (CR ≤ 101%, but assuming edge case):
  payout = (10,000 / 1,000,000) × 1,020,000 = 10,200 USDC
  → 2% premium (user receives more than fair value)
```

#### Hedger Loss

```
hedgerLoss = (qeuroAmount / totalQEUROSupply) × hedgerMargin
```

The hedger absorbs a proportional loss on their margin, recorded via:

```solidity
hedgerPool.recordLiquidationRedeem(qeuroAmount, usdcPayout);
```

***

### 📊 View Functions

#### Check Liquidation Status

```solidity
function getLiquidationStatus() external returns (
    bool isInLiquidation,           // True if CR ≤ 101%
    uint256 collateralizationRatioBps,  // CR in basis points
    uint256 totalCollateralUsdc,    // Total USDC in vault
    uint256 totalQeuroSupply        // Total QEURO supply
);
```

**Usage Example**

```solidity
(bool isLiquidation, uint256 crBps, uint256 collateral, uint256 supply) = 
    vault.getLiquidationStatus();

if (isLiquidation) {
    // Display warning to user
    // "Protocol is in liquidation mode - you may receive less than fair value"
}
```

#### Calculate Expected Payout

```solidity
function calculateLiquidationPayout(uint256 qeuroAmount) external returns (
    uint256 usdcPayout,          // USDC amount user will receive
    bool isPremium,              // True if payout > fair value
    uint256 premiumOrDiscountBps // Premium/discount in basis points
);
```

**Usage Example**

```solidity
(uint256 payout, bool isPremium, uint256 diffBps) = 
    vault.calculateLiquidationPayout(10000e18); // 10,000 QEURO

// If isPremium = false and diffBps = 500
// → User will receive 5% less than fair value (haircut)
```

***

### 💸 Fees in Liquidation Mode

```solidity
bool public constant TAKES_FEES_DURING_LIQUIDATION = true;
```

By default, redemption fees are applied even in liquidation mode.

| Configuration | Impact |
|---------------|--------|
| `TAKES_FEES_DURING_LIQUIDATION = true` | Normal fee (0.1%) even in liquidation |
| `TAKES_FEES_DURING_LIQUIDATION = false` | No fee in liquidation |

> **Rationale**: Fees allow the protocol to continue generating revenue to cover operational costs even during stress periods.

***

### 🛡️ User Protection

#### Slippage Protection

```solidity
function redeemQEURO(uint256 qeuroAmount, uint256 minUsdcOut) external;
```

The `minUsdcOut` parameter protects against:
- Oracle price changes between submission and execution
- Surprises in liquidation mode

**Recommendation**: In liquidation mode, use a calculated `minUsdcOut`:

```javascript
// Frontend calculation
const status = await vault.getLiquidationStatus();
const expectedPayout = await vault.calculateLiquidationPayout(amount);
const minUsdcOut = expectedPayout.usdcPayout * 0.99; // 1% slippage tolerance
```

#### Monitoring

```solidity
// Event emitted during a liquidation redemption
event LiquidationRedeemed(
    address indexed user,
    uint256 qeuroAmount,
    uint256 usdcPayout,
    uint256 collateralizationRatioBps,
    bool isPremium  // True if user receives more than fair value
);
```

***

### 🔄 Integration with HedgerPool

#### recordLiquidationRedeem

```solidity
// Called by QuantillonVault during a liquidation
function recordLiquidationRedeem(uint256 qeuroAmount, uint256 usdcAmount) 
    external onlyVault;
```

**Impact on Hedger**

1. `filledVolume` decreases proportionally
2. `realizedPnL` updated (usually negative in liquidation)
3. `margin` reduced proportionally
4. If `margin < minMarginRatio`: position may be closed

***

### 📈 Liquidation Scenarios

#### Scenario 1: Sudden EUR/USD Crash

```
Initial state:
- QEURO Supply: 10M
- Vault USDC: 11M (CR = 110%)
- EUR/USD: 1.00

Event: EUR/USD rises to 1.20 (euro strengthens)

New state:
- QEURO EUR value = 10M × 1.20 = 12M USD
- Vault USDC: 11M
- CR = 11M / 12M = 91.67% ❌

→ Liquidation mode activated
→ Users receive ~92 cents per QEURO instead of 1.20
```

#### Scenario 2: Massive Hedger Withdrawal

```
Initial state:
- QEURO Supply: 10M
- User collateral: 10M USDC
- Hedger margin: 1.5M USDC
- CR = 115%

Event: Hedger withdraws margin (position closed)

New state:
- Total USDC: 10M
- CR = 100% ❌

→ Liquidation mode activated
→ Pro-rata distribution activated
```

#### Scenario 3: USDC Depeg

```
If USDC depegs to 0.95:
- ChainlinkOracle triggers circuit breaker
- Protocol pause likely
- Manual liquidation possible via governance
```

***

### ⚙️ Governance Configuration

#### Modify Thresholds

```solidity
function updateCollateralizationThresholds(
    uint256 _minCollateralizationRatioForMinting,
    uint256 _criticalCollateralizationRatio
) external onlyRole(GOVERNANCE_ROLE);
```

**Validations**

```solidity
// Minimum acceptable for minCollateralizationRatioForMinting
uint256 public constant MIN_ALLOWED_COLLATERALIZATION_RATIO = 101e18; // 101%

// Minimum acceptable for criticalCollateralizationRatio
uint256 public constant MIN_ALLOWED_CRITICAL_RATIO = 100e18; // 100%
```

**Event**

```solidity
event CollateralizationThresholdsUpdated(
    uint256 indexed minCollateralizationRatioForMinting,
    uint256 indexed criticalCollateralizationRatio,
    address indexed caller
);
```

***

### 🚨 Emergency Procedure

#### Steps in Case of Liquidation

1. **Automatic monitoring** → Detection of CR ≤ 101%
2. **Notification** → Alerts to stakeholders
3. **Liquidation mode** → Automatically activated
4. **Communication** → Inform users
5. **Analysis** → Identify the cause
6. **Resolution** → Capital injection or other measures

#### Roles Involved

| Role | Possible Action |
|------|-----------------|
| `EMERGENCY_ROLE` | Pause protocol if necessary |
| `GOVERNANCE_ROLE` | Adjust thresholds |
| Hedger | Add margin to improve CR |
| Users | Pro-rata redemption available |

***

### 📋 Summary

| Aspect | Detail |
|--------|--------|
| **Trigger** | CR ≤ 101% |
| **Mechanism** | Pro-rata collateral distribution |
| **Calculation** | `payout = (amount / supply) × totalUSDC` |
| **Hedger** | Absorbs proportional loss |
| **Fees** | Applied by default |
| **Protection** | `minUsdcOut` for slippage |
| **View Functions** | `getLiquidationStatus()`, `calculateLiquidationPayout()` |

***

> **Important**: Liquidation mode is designed as a protection mechanism, not a normal state. The goal is to maintain CR > 105% at all times through the hedging system and YieldShift incentives.
