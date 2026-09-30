# stQEURO Token

stQEURO is the ERC-4626 share token received when staking QEURO. Each registered external vault has its own series; the current Morpho series is `stQEUROMORPHO1` (vaultId 2). Each series is an ERC-20 fungible token, separate from the other series.

The token holds QEURO, while the protocol deploys USDC backing through the external strategy adapter. Users hold the same number of shares as yield is credited and vests; the redeemable QEURO value per share increases. There is no reward-claim transaction or rebasing.

## Yield source and distribution

QuantillonVault 1.5.0 allocates harvested Morpho yield using the staked fraction of total QEURO at harvest. The unstaked fraction goes to treasury. The hedger receives only the configured percentage haircut on gross staker yield; its collateral does not earn a separate base allocation. The vault ignores the token's legacy `yieldFee` on this credit path. Conversion costs and protocol safety checks remain.

The full formula, current haircut, nightly schedule, vesting period and APY interpretation are maintained on [Yield Distribution](../yield-distribution.md).

## Shares and redeemable value

```text
Position value in QEURO = shares * current QEURO per share
```

Use `convertToAssets(shares)` or `previewRedeem(shares)` for a current quote. `totalAssets()` reports redeemable QEURO backing, excluding unvested yield. The raw QEURO balance used for the harvest allocation includes credited unvested yield, so it can differ from `totalAssets()`.

**Illustrative example:** 1,000 shares redeem for 1,000 QEURO at a rate of 1.000. If vested earnings later raise the rate to 1.010, those shares redeem for 1,010 QEURO. This example is not a forecast of yield or timing.

## Staking and unstaking

1. Resolve the series from `stQEUROFactory.getStQEUROByVaultId(vaultId)`.
2. Approve QEURO and call `deposit(qeuroAssets, receiver)`, or use the protocol's combined mint-and-stake flow.
3. Receive shares at the current exchange rate.
4. Call `redeem(shares, receiver, owner)` to receive currently redeemable QEURO. Converting QEURO back to USDC is a separate protocol redemption with its own liquidity and safety checks.

There is no minimum staking-duration lock in this direct ERC-4626 flow. Newly credited yield vests separately; exiting does not entitle a user to the unvested amount. Normal operations remain subject to pause and contract checks. The governance-authorized emergency withdrawal is not an unrestricted user bypass.

## Technical parameters

| Read / setting | Meaning |
| --- | --- |
| `asset()` | QEURO, the underlying token held by the series |
| `totalAssets()` | Currently redeemable QEURO backing |
| `convertToAssets(shares)` | QEURO value of the requested shares |
| `previewDeposit(assets)` / `previewRedeem(shares)` | Current ERC-4626 quotes |
| `vestingPeriod()` | Release period for new yield; governance-adjustable |
| `yieldFee` | Legacy token setting; ignored by vault 1.5.0 yield credits |

Harvest and direct credit are authorized on QuantillonVault through `YIELD_DISTRIBUTOR_ROLE`. Token governance controls its own parameters and upgrades. Core upgrades use the governance Safe and timelock; see [Smart Contract Components](../smart-contract-components.md).

```solidity
interface IstQEURO {
    function asset() external view returns (address);
    function totalAssets() external view returns (uint256);
    function convertToAssets(uint256 shares) external view returns (uint256);
    function convertToShares(uint256 assets) external view returns (uint256);
    function previewDeposit(uint256 assets) external view returns (uint256);
    function previewRedeem(uint256 shares) external view returns (uint256);
    function deposit(uint256 assets, address receiver) external returns (uint256 shares);
    function redeem(uint256 shares, address receiver, address owner) external returns (uint256 assets);
}
```

## Events and integrations

- `Deposit` and `Withdraw` describe ERC-4626 staking activity.
- `VaultYieldDistributed` on QuantillonVault records the harvested USDC allocation.
- `VaultYieldBreakdown` retains a zero `hedgerBase` field for compatibility in 1.5.0 and records the haircut.
- The token's legacy `YieldParametersUpdated` event does not establish an effective fee on vault 1.5.0 credits.

stQEURO can be integrated as an ERC-20 share token. Applications should use contract previews, handle pause and liquidity conditions, and distinguish vested backing from raw balances. A transfer of shares transfers their participation in the series; harvest allocation does not track a wallet's staking duration.

## Risks and rate presentation

Returns vary with strategy performance, actual deployed backing, harvest execution, conversion costs and governance settings. The underlying provider APY is not a guaranteed net QEURO return. There is no implemented APY floor or ceiling.

External strategy losses, oracle failures, contract defects and liquidity limits can affect the protocol. The contracts have not been audited by a professional security firm; see [Risks and Mitigation Strategies](../../risk-management-and-sustainability/risks-and-mitigation-strategies.md) for the current security posture.
