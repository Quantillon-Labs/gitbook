# Protocol operations and responsibilities

Quantillon Protocol uses onchain contracts for QEURO minting, redemption, vault accounting and hedge collateral. The public interface is operated by the French company identified in the [Legal Notice](complementary-information/legal-notice.md).

## Hedging and liquidity

Governance sets the single hedger address. See [HedgerPool](protocol/hedger-pool.md) for the collateral requirements and the risks of relying on one operator.

## Governance and emergency controls

The current governance Safe requires two of three signatures. Core upgrades pass through a 12-hour timelock. The [governance overview](quantillon-dao.md) describes the scope of these controls and the transition planned for QTI governance. [Quantillon Guardians](quantillon-guardians.md) describes emergency powers and delegated roles.

A foundation has been proposed for future oversight. It has not been established.
