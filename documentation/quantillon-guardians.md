---
description: The Guardian to take rapid decisions in a crisis
---

# Quantillon Guardians

## 🔎 TL;DR <a href="#tl-dr" id="tl-dr"></a>

* Like in any protocol, it may be necessary to rapidly pause some functionalities, to change some parameters or references in the protocol in order to react rapidly to unforeseen events.
* The protocol has a multisig on each of the chains on which it is deployed which has the rights to perform such changes.

## 💡 Need for a Guardian <a href="#need-for-a-guardian" id="need-for-a-guardian"></a>

The Guardian role is a privileged role granted to some addresses on every chain where the protocol is deployed. All the privileges of the guardian are also available to the governor addresses (e.g. the `Timelock` addresses) of the Quantillon Protocol.

This guardian role is held on every chain by a 2/3 Gnosis Multisig controlled by the same Quantillon Labs team members as those in the emergency multisig TBD.

If a guardian role with specific rights is needed, it is because there could be some situations in the protocol which are time sensitive. In case there is a flaw in one of the contracts where an attacker can systematically make a profit, we should be able to pause the necessary parts of the contract without having to wait for the end of a governance vote for its execution.

Note however that governance can revoke the guardian ability to any address at any time.

## 🔘 Responsibilities <a href="#responsibilities" id="responsibilities"></a>

In clear terms, the guardian role enables to:

* Pause and unpause some contracts functionalities
* Update rapidly some parameters like users minting and burning fees, or the debt ceiling for an asset in the borrowing module
* The guardian is reponsible for the planned emissions of the QTI token.

Quantillonsmart contracts have been designed to limit a guardian's ability to impact the protocol in a way that is harmful to the system. For instance, a guardian cannot modify references to an oracle contract and hence manipulate prices at its advantage.
