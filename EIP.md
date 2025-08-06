---
title: Fee Bump Transaction for Ethereum.
description: Introduces a mechanism that allows an external party to pay the gas fees for an Ethereum transaction, improving transaction reliability and user experience.
author: Progress Ochuko Eyaadah (@KoxyG)
discussions-to: <URL>
status: Draft
type: Standards Track
category: Core
created: 2025-08-06


## Abstract
This EIP proposes a mechanism that allows an external party to sponsor gas fees for Ethereum transactions, and introduces a fee bumping feature to prioritize transactions during periods of congestion.

## Motivation
As Ethereum's network usage increases, transaction fees can become volatile, making it challenging for users, especially those without ETH, to interact with dApps and the network. The inability to pay gas fees due to surge pricing or fluctuating base fees can block essential transactions, such as escrow releases or contract executions. This proposal addresses this issue by allowing a third party to sponsor the gas fees of a transaction and by introducing a fee bumping mechanism to ensure timely inclusion in the blockchain during periods of high congestion.


## Specification
### Gas Sponsorship

This proposal introduces a mechanism where an external party (the sponsor) can pay the gas fees for a transaction initiated by another account (the sender). The transaction's sender signs the transaction, while the sponsor signs an additional approval to cover the gas fees. The sponsor's account will be charged the gas fees, ensuring that the sender can execute the transaction without holding ETH.

For example:
```solidity
transaction {
    sender: 0x12345...
    sponsor: 0x67890...
    data: <transaction data>
}
```
The transaction will be executed as usual, but the gas fee will be deducted from the sponsor’s account.

## Fee Bumping Mechanism
In the event of network congestion or rising gas prices, this proposal introduces the ability for an external party (the sponsor) to submit a fee bump. The sponsor can increase the gas fee of an already submitted transaction to ensure that it is included in the next block.
- A FeeBumpTransaction wraps an existing transaction, adding an additional gas fee to prioritize its inclusion.
- The sponsor pays for the fee bump, while the inner transaction remains unchanged.
- The fee bump must exceed the original fee rate by a factor of 1.5x to be accepted.

For example:
```Solidity
feeBumpTransaction {
    feeSource: 0xSponsorAccount
    fee: adjustedFee
    innerTx: originalTransaction
}
```

## Validity Rules
The sponsor account must have sufficient balance to cover both the transaction's gas fee and the fee bump.
If the sponsor's balance is insufficient, the transaction will be rejected.
If the inner transaction is valid, but the fee bump transaction fails, the inner transaction will still be processed.
If the inner transaction fails, the fee bump transaction will also fail.

## Fee Calculation and Surge Pricing
The fee bump transaction is considered valid if the outer fee exceeds the original transaction’s fee by at least 1.5x.
Surge pricing considerations are unchanged, but the fee bump mechanism allows for timely transaction inclusion even during high-demand periods.

## Security Considerations
Replay Protection: Each transaction and fee bump will have a unique nonce, preventing replay attacks.
Sponsor Trust: Sponsors must ensure they are aware of the transaction fees they are covering. Misuse of this feature may occur if users or malicious actors trick sponsors into paying for unintended transactions.
Gas Optimization: To prevent excessive gas usage, the fee bump should be as efficient as possible and only applied when necessary to ensure inclusion in a block.


## Rationale
Ethereum transactions can sometimes get delayed due to insufficient gas fees, particularly during periods of high network demand. By introducing gas sponsorship and fee bumping, we can enable third parties (e.g., dApps or service providers) to cover gas fees for users, improving the user experience and ensuring that critical transactions (such as those related to escrow or contract execution) can still be executed. The fee bumping mechanism is particularly important during network congestion, allowing transactions to be prioritized and included in the blockchain without needing to re-submit the transaction.

TBD

## Backwards Compatibility

This proposal introduces new transaction types and changes to the existing transaction format, meaning older Ethereum clients may not be able to process these transactions without updating. However, backward compatibility can be achieved by upgrading Ethereum clients to support the new transaction formats and fee bumping mechanism.


## Test Cases
1. Basic Gas Sponsorship:
- Scenario: A transaction is sent from AccountA with AccountB sponsoring the gas fee. The transaction should be processed, and the gas fee should be deducted from AccountB.
2. Fee Bumping during Network Congestion:
- Scenario: A transaction with insufficient fees is submitted, and a fee bump is sent from AccountB to increase the gas fee and ensure transaction inclusion.

## Reference Implementation
The reference implementation will be available at [GitHub Repo Link], which demonstrates how to implement the gas sponsorship mechanism and fee bumping feature in Ethereum.



## Security Considerations

The main security concern with this proposal is ensuring that sponsors are not tricked into paying for unauthorized transactions. Proper authorization mechanisms and transparency about transaction costs must be implemented to prevent misuse. Additionally, fee bumping should only be used when necessary, and a malicious actor should not be able to manipulate the process to inflate fees unduly.



## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
