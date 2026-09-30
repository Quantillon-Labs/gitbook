# Yield Distribution

This page describes QuantillonVault 1.5.0, activated on Base on 30 September 2026. The current funded strategy is the MetaMorpho USDC vault. The allocation is calculated from balances at harvest, not from how long each wallet has staked.

## Where the yield comes from

QEURO is backed by USDC. The protocol deploys part of that USDC through its registered Morpho adapter; only deployed capital earns the strategy's return. Staking QEURO exchanges QEURO for stQEURO shares. It does not deposit a user's QEURO directly into Morpho, and does not itself deploy more USDC.

Hedger collateral supports the FX hedge. Its amount, location and unrealized P&L do not give the hedger a base share of Morpho yield under this allocation policy.

## The nightly split

The keeper is scheduled daily at **03:00 UTC**. Successful execution depends on the strategy's available liquidity and the protocol's safety checks.

For each successful harvest:

1. Snapshot total QEURO supply and the registered staking token's raw QEURO balance before harvesting or minting yield.
2. Allocate harvested USDC to stakers in proportion to that staked share. Allocate the unstaked share to the treasury.
3. Deduct the configured hedger haircut from the gross staker allocation only. Transfer that haircut to the configured recipient in USDC.
4. Convert the remaining staker allocation to QEURO and credit the staking token. Yield becomes redeemable through vesting and increases the QEURO value of each share.

The haircut was **0 bps when checked on 30 September 2026**: the hedger receives no harvested yield at this setting. Governance can change it. There is no additional stQEURO yield fee on this vault's credit path; its legacy stored `yieldFee` is ignored. Conversion costs and safety checks still apply. Mint/redeem fees are separate.

### Exact allocation

Let `Y` be actual harvested USDC, `Q` total QEURO supply, `S` the raw QEURO balance of the selected staking token when it has outstanding shares, and `h` the haircut in basis points. Clamp `S` to `Q`.

```text
Gross staker yield G = floor(Y * S / Q)
Hedger payout H      = floor(G * h / 10,000)
Staker allocation   = G - H
Treasury allocation = Y - G
```

If total QEURO supply is zero or there are no staking shares, `G = 0`. Staking-ratio rounding stays with treasury; haircut rounding stays with stakers. Credited but unvested QEURO counts in `S`, although it is excluded from the currently redeemable share price.

**Illustrative harvest:** with 10 USDC harvested, 1,000 QEURO total supply and 600 QEURO held by the staking token, gross staker yield is 6 USDC and treasury yield is 4 USDC. At zero haircut the hedger gets zero. At a 2% haircut the hedger gets 0.12 USDC and stakers receive the QEURO credit resulting from 5.88 USDC, after conversion costs.

## Share weighting and vesting

Stakers participate pro rata through their stQEURO shares. This remains **snapshot-weighted**: the protocol does not accumulate a wallet's staking duration to divide a harvest. Being staked earlier in the day does not create a separate historical entitlement.

New yield currently vests over **24 hours**. Vesting spreads when credited yield becomes redeemable; it does not turn the allocation into time-weighted staking. Existing principal is not subjected to that yield vesting period. Unstaking returns the currently redeemable QEURO value, excluding unvested yield. If the final shareholder exits, remaining unvested yield goes to treasury. Overlapping credits combine the remaining and new vesting schedules using an amount-weighted end time.

## What the displayed APY means

The dapp shows the provider's **one-month average underlying APY**, before the configured haircut and conversion costs. It is not a fixed or guaranteed net QEURO return, and the dapp does not calculate a new net-APY estimate.

For example, staking 100 QEURO while the provider displays 4.13% does not guarantee 104.13 QEURO after a year. That idealized result requires matching deployed backing, stable conversion assumptions, continuous participation, the assumed compounding and no deductions or execution costs. Actual credit depends on yield realized on deployed USDC, the harvest snapshot, haircut, conversion and vesting. Holding unstaked QEURO does not earn this strategy yield.

## Contract and governance boundaries

- The current release supports **one funded strategy** and rejects competing funded strategies or another staking series with outstanding shares. Multiple registered adapters are not evidence that independent simultaneous allocations are supported.
- `previewVaultYieldDistribution(vaultId)` previews the split; actual liquid proceeds can differ. A successful preview does not guarantee that conversion checks pass.
- A conversion failure rolls back the harvest and transfers atomically. The keeper preserves its journal and guards against duplicate successful harvests.
- `YieldShift` is a separate accounting contract. The current Morpho harvest does not use its dynamic split or hedger-claim ledger.
- Governance sets `hedgerStakingYieldHaircutBps` and `hedgerYieldRecipient`. The old annual funding setting is retired. There is no payment solely because hedger collateral exists.

## Activation record

[The confirmed Base upgrade](https://basescan.org/tx/0xc41087befce4ef0832398af3abfd45a91860eff5f97f2af6bd36df993d3255b3) activated vault 1.5.0 on 30 September 2026 at 09:21:13 UTC, block 51,985,363. Existing staking shares and vesting were preserved. Historical payouts are not recomputed; the first successful harvest after activation uses the new policy for all then-pending yield.

See [stQEURO](quantillon-protocols-tokens/stqeuro-token.md), [External Staking Vaults](external-staking-vaults.md), and the [contract release reference](https://smartcontracts.quantillon.money/Yield-Distribution-1.5.0.html).
