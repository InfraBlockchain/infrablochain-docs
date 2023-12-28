---
title: Participants in InfraBlockchain
description: Explains the roles of collators and validators.
keywords:
  - Collator
  - Validator
---

Introduction to key participants in the Multi-Chain Infrastructure Blockchain Ecosystem:

- Validators: Validators play a crucial role in the multi-chain infrastructure blockchain ecosystem by generating blocks and establishing shared security. They protect both the Relay Chain and parachains. Validators are responsible for creating blocks, forming shared security to safeguard the Relay Chain, and ensuring the protection of parachains.

- Collators: Collators perform the essential function of generating parachain blocks and delivering them to validators. They play a crucial role in the creation and submission of parachain blocks to validators, contributing to the overall functionality of the multi-chain infrastructure blockchain ecosystem.

![Network Participants](/media/images/docs/infrablockchain/learn/architecture/network-participants.png)

## Validator

Validators verify proofs from collators (para-validators) and protect Relay Chain and parachains through consensus with other validators. Validators play a crucial role in adding new blocks to Relay Chain(block producers), ensuring the inclusion of all parachain blocks.

Validators in the **_InfraBlockchain_** ensure the overall network's security through shared security, enabling the exchange of cross-chain messages using XCM (e.g., token transfers, transaction calls for specific pallets). Validators guarantee that each parachain follows its unique rules and can exchange messages securely among parachains in a trusted environment.

## Para-Validators

### Overview

Para-Validators participate in the **Availability and Validity** protocol's parachain phase, submitting a [candidate receipt](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/822bc6c9706774a98122eb432f412b871a98a4bd/infrablockspace/primitives/src/v6/mod.rs#L521) to the Relay Chain. This process enables block producers to include information about parachain blocks in the Relay Chain through the availability and validity verification process.

### Candidate Receipt

```rust
pub struct CandidateReceipt<H = Hash> {
	pub descriptor: CandidateDescriptor<H>,
	pub commitments_hash: Hash,
}

pub struct CandidateDescriptor<H = Hash> {
	pub para_id: Id,
	pub relay_parent: H,
	pub collator: CollatorId,
	pub persisted_validation_data_hash: Hash,
	pub pov_hash: Hash,
	pub erasure_root: Hash,
	pub signature: CollatorSignature,
	pub para_head: Hash,
	pub validation_code_hash: ValidationCodeHash,
}
```

### Para-Validator Selection

_Para-Validators_ operate as a group and are chosen by the runtime to validate parachain blocks for all parachains connected to the Relay Chain every epoch. The selected Para-Validators participate in validation as one of the randomly chosen **_InfraBlockchain_** [validator](#validator) for each epoch, forming the Para-Validator pool.

### Roles of Para-Validators

Para-Validators verify the validity of the information included in a series of assigned parachain blocks. They receive parachain block candidates and their proof of validity [Proof-of-Validity(PoV)](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/822bc6c9706774a98122eb432f412b871a98a4bd/cumulus/primitives/core/src/lib.rs#L155) from collators.

Para-Validators perform the initial validity check on the block candidate. A candidate that passes the validation with sufficient signatures is considered backable.

## Block Producer

There are validators who participate in the consensus mechanism to produce Relay Chain blocks based on the validity of other validators. These validators, known as block producers, are selected by the consensus engine (e.g., BABE, Aura) and can record at most one backable candidate for each parachain to be included in the Relay Chain. A backable candidate included in the Relay Chain is considered backed in the fork of that Relay Chain.

In Relay Chain blocks, the block producer only includes candidate parachain blocks with parent candidate data ([Candidate Receipt](./network-participants.md#candidate-receipt-예시)) from the previous Relay Chain block. **This ensures that parachains follow a valid chain.** Additionally, the block author includes candidate parachain blocks with [erasure-coding](https://wiki.polkadot.network/docs/learn-parachains-protocol#erasure-codes) chunks (for availability) to facilitate the system's availability and validity checks in the next round.

## Parachain Block Availability

Validators also contribute to the so-called _Availability_ distribution. Even after being backed in the fork of the Relay Chain, a candidate's availability is still pending, and it is not fully included as part of the parachain (temporary inclusion). Information about the candidate's availability is recorded in the next Relay Chain block. Only when sufficient information is available is the candidate considered a complete parachain block or parablock.

## Parachain Block Approval

Validators also participate in the **Approval** process. Even if a parablock is available and considered part of the parachain, it remains in a pending approval state. Since para-validators are a small subset of all validators, there is a potential for dishonest behavior among the majority of para-validators assigned to a specific parachain.

## Rewards

Therefore, to avoid assigning more para-validators, which may potentially degrade the system's throughput, it is necessary to execute auxiliary verification of parablocks. If the verification fails, para-validators receive penalties and are excluded from the validator pool, suppressing misconduct. However, in case of successful performance, the para-validator receives legal fiat-based [System Token](../protocol/system-token.md) rewards, including block rewards (including transaction fees) for their activity.

## Fork Choice

Finally, validators participate in the chain selection process within GRANDPA, ensuring that only available and valid blocks are included in the finalized Relay Chain.

## Collator

Role of Collator
Collators collect all transactions occurring in parachains from users and generate state transition proofs (PoV) for Relay Chain validators. Under normal circumstances, collators collect and execute transactions to create block candidates, proposing them to one or more validators along with PoV for inclusion in the parachain block.

Collators are similar to validators in other blockchains, but since **_InfraBlockchain_** Validators ensure security, collators can build [service-specific](../../service-chains/README.md) blockchains without worrying about security separately. If a parachain block is not valid, it is rejected by the validators. Para-validators assigned to each parachain verify the validity of submitted candidates and then collect and aggregate the validity from other para-validators. This process is called candidate backing. Para-validators receive PoV associated with an arbitrary number of parachain block candidates from untrusted collators. A candidate is considered backable when at least 2/3 of the assigned validators have verified its validity.

Validators must successfully verify the following conditions in the following order:

1. The parachain candidate block should not exceed any parameters of the verification data.
2. The collator's signature must be valid.
3. Verify the parachain candidate block by executing the parachain runtime.

If the candidate block meets the specified criteria to be included in the Relay Chain block, the selected Relay Chain [block producer](#block-producer) selects one backable candidate for each parachain to be included in the Relay Chain block (inclusion). The candidate block is considered backed. Furthermore, having more collators does not necessarily mean better or safer assumptions. Instead, having too many collators can slow down the network. The only malicious behavior a collator can exhibit is transaction censorship. To prevent censorship, a parachain only needs to ensure the presence of a neutral collator, and a majority is not strictly required. **Theoretically, the censorship problem can be solved with just one honest collator**.
