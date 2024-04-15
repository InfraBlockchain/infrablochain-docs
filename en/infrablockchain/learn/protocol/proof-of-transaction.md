---
title: Proof-of-Transaction
description: This document covers the overall content regarding InfraBlockchain's unique consensus mechanism, PoT Proof-of-Transaction).
keywords:
  - Proof-of-Transaction
  - Incentive
  - System Token
---

## Transaction-as-a-Vote(TaaV)

![Transaction as a Vote](/media/images/docs/infrablockchain/learn/protocol/taav.png)

The Proof of Transaction (PoT) is a unique consensus mechanism in InfraBlockchain. Its core concept is Transaction-as-a-Vote. The metadata of transactions in InfraBlockchain can include information about the account of the block producer candidate. The transaction message containing candidate account information is signed with the private key of the entity that generated the transaction, providing its own cryptographic proof. When a transaction is executed, a transaction fee is incurred, and the remainder, excluding the reward for the block producer and the amount refunded to the transaction caller, is returned as a reward to the Infra Relay Chain validator.

## Transaction Metadata

In Substrate, you can freely add verification elements for each chain in addition to the elements necessary to verify transactions. To implement TaaV, we have added metadata for InfraBlockchain transactions, such as the token identifier (`asset_id`) to pay the fee with a system token and information about the Infra Relay Chain validator candidate account.

```rust
pub struct ChargeSystemToken<AssetId, Account> {
#[codec(compact)]
    tip: Balance,
    asset_id: Option<AssetId>,
    candidate: Option<Account>
}
```

- `asset_id`: Represents the system token identifier.
- `candidate`: Represents the account for the voting target.

When a transaction containing the above metadata is received, the voting amount is determined based on the transaction fee. The following is the voting structure:

### Voting

```rust
pub struct Vote<Account, Weight> {
    pub candidate: Account,
    pub amount: Weight,
}
```

- **candidate**: Represents the target to vote for, i.e., the Infra Relay Chain block producer candidate.
- **amount**: Represents the voting amount based on the transaction fee.

### Reward

Represents a reward for a portion of the transaction fee. This reward is distributed to the Infra Relay Chain validator.

```rust
pub struct Reward<DestId, AssetId, Amount> {
    pub origin: RewardOrigin<DestId>,
    pub asset: AssetId,
    pub amount: Amount,
}

```

- `origin`: Represents the location where the reward was generated. The reward generated for each transaction is managed in the Infra Relay Chain `ValidatorManagement` module. When the Infra Relay Chain validator calls the `claim()` transaction, the reward is distributed to the corresponding account from the chain where the reward was generated.
- `asset`: Represents the token identifier that paid the transaction fee.
- `amount`: Represents the amount of the reward.

## Proof of Transaction

For each transaction, there are **voting** and **reward** as proofs. These are aggregated in the `ValidatorManagement` module when verifying a Parachain block or its own block. The following is the `Proof-of-Transaction` data type:

```rust
#[derive(Encode)]
pub struct PoT<Account, DestId, AssetId, Amount, Weight> {
/// Reward amount for each transaction
    pub reward: Reward<DestId, AssetId, Amount>,
    /// Amount of vote for Account based on fee_amount
    pub vote: Option<Vote<Account, Weight>>,
}

```

- `reward`: Generated for each transaction and aggregated in the Infra Relay Chain.
- `vote`: If included in the transaction, it is aggregated and managed in the Infra Relay Chain.

## Validator Pool

InfraBlockchain is an enterprise blockchain for institutions and public institutions, and nodes must operate normally and achieve BFT consensus in any situation. As a public/permissioned hybrid blockchain, any entity can be elected as a block producer. To design these ideal properties, InfraBlockchain has two block producer (validator) pools:

- **Proof-of-Transaction Node Pool**: Manages nodes elected by PoT.
- **Seed Trust Node Pool**: Manages nodes that operate normally in any situation, such as financial institutions and government agencies.

![Validator Pool](/media/images/docs/infrablockchain/learn/protocol/validator-pool.png)

The initial validators of InfraBlockchain are composed of Seed Trust validators, forming a permissioned blockchain. As the network stabilizes, it can transition to a public blockchain where anyone can participate as a block producer using the PoT consensus mechanism.

## Aggregated Proof-of-Transaction (PoT)

InfraBlockchain is a multi-chain architecture where multiple Parachain blocks are executed in parallel, centered around InfraRelayChain. InfraRelayChain validators verify each Parachain block and collect the votes included in the block, which is called Aggregated Proof-of-Transaction.

For each block, the votes for the InfraRelayChain block producer (validator) candidate are optionally included, and these votes are collected during the votes are collected during the verification process and stored in a state of InfraRelayChain. After a certain period, the votes collected are aggregated based on which candidate received the most votes, and the candidate with the most votes is elected as the block producer.

## Block Time Weight

To give more weight to recent transaction votes, a Block Time Weight is multiplied. The weight doubles at a rate of 1 year. The reference time is the genesis block (0th block) of InfraRelayChain, and the weight is multiplied based on the InfraRelayChain block. The exact calculation of the transaction vote weight reflecting the block time is as follows:

For example,

```
The number of InfraRelayChain blocks corresponding to a year (one block every 6 seconds) = 5256000

Block time-weighted transaction vote weight = 2^(current InfraRelayChain block / the number of InfraRelayChain blocks corresponding to a year)
```

## Next Steps

- [How to Vote with TaaV](../../tutorials/basic/how-to-vote-with-taav.md)
- [System Token](./system-token.md)
- [ParasInclusion](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/master/infrablockchain/runtime/parachains/src/inclusion/mod.rs)
