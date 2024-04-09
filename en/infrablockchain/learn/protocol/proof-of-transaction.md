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

The core idea behind **Proof-of-Transaction (PoT)**, the proprietary consensus mechanism of **_InfraBlockchain_**, is encapsulated in the concept of **Transaction-as-a-Vote(TaaV)**. The metadata of transactions in **_InfraBlockchain_** may optionally include votes for candidates already participating or eligible to participate in the blockchain consensus process. Transaction messages containing votes for block producer candidates are signed with the private key of the blockchain account that initiated the transaction, providing cryptographic proof for each transaction vote. **_InfraBlockchain_** votes possess the following characteristics:

- **System Token Identifier**: The weight of a vote depends on whether the transaction fee was paid with a specific [System Token](../protocol/system-token.md). Thus, it includes an `id` that identifies this token.

- **Vote Target**: The entity to be voted for must have a [blockchain account](../substrate/learn/basic/accounts-addresses-keys.md) and be of type InfraRelayChain's blockchain account, typically an `AccountId32`.

- **Transaction Weight**: The weight of a transaction vote is determined based on the transaction fee.

```rust
// For example,
pub struct PoTVote {
    system_token_id: u32,
    account_id: infra_relay::AccountId,
    vote_weight: u128
}
```

### Transaction Metadata

Transaction-as-a-Vote (TaaV) includes votes in the metadata of transactions. In Substrate, the `SignedExtension` trait can be used to add transaction metadata. Here's an example using this trait:

```rust
pub struct ChargeSystemToken<Account, Balance> {
    #[codec(compact)]
    tip: Balance,
    system_token_id: Option<u32>,
    vote_candidate: Option<Account>
}

impl SignedExtension for ChargeSystemToken {
    ...
    pub fn post_dispatch(..) {
        // Handling votes
    }
}
```

## Validator Pool

**_InfraBlockchain_** is designed to operate nodes honestly in any scenario, as it is an enterprise blockchain for institutions and public entities. Moreover, it serves as a public blockchain where any entity can be elected as a block producer. To achieve these ideal properties, **_InfraBlockchain_** has two pools of block producers (validators):

- **Proof-of-Transaction Node Pool**: Manages nodes elected by PoT.

- **Seed Trust Node Pool**: Manages nodes operated by entities like financial institutions or government organizations that operate honest nodes in any situation.

![Validator Pool](/media/images/docs/infrablockchain/learn//protocol/validator-pool.png)

The initial configuration of **_InfraBlockchain_** validators consists of Seed Trust validators, forming a permissioned blockchain. As the network stabilizes, it can transition to a public blockchain where anyone can participate as a block producer using the PoT consensus mechanism.

## Aggregated Proof-of-Transaction(PoT)

**_InfraBlockchain_** is a multi-chain architecture where multiple parachain blocks are executed in parallel, centered around the **_InfraRelayChain_**. InfraRelayChain validators verify each parachain block and collect the votes included in that block. This process is referred to as **Aggregated Proof-of-Transaction**.

For each block, votes for candidates for the block producer (validator) of the **_InfraRelayChain_** are optionally included, and during the verification process, these votes are collected and stored as a state in the **_InfraRelayChain_**. After a certain period, an aggregation of votes is performed based on the collected votes, and the candidate with the most votes is elected as the block producer.

```rust
#[pallet::storage]
#[pallet::unbounded]
pub type PotValidatorPool<T: Config> = StorageValue<_, VotingStatus<T>, ValueQuery>;

#[derive(Encode, Decode, Clone, PartialEq, RuntimeDebug, TypeInfo)]
#[scale_info(skip_type_params(T))]
pub struct VotingStatus<T: Config> {
	pub status: Vec<(T::InfraVoteAccountId, T::InfraVotePoints)>,
}
```

## Time-Weighted PoT Votes

To give more weight to recent transaction votes, a block time weight is multiplied. The weight increases at a rate of 2x per year. The reference time is the genesis block of the **_InfraRelayChain_**(block 0), and the weight is multiplied based on the **_InfraRelayChain_** block time. The precise calculation of the transaction vote weight considering block time is as follows:

Example,

```
- Number of InfraRelayChain blocks equivalent to a year (6 seconds per block) = 5,256,000

- Transaction vote weight considering block time = 2^(current InfraRelayChain block / Number of InfraRelay Chain blocks equivalent to a year)
```

## Next Steps

- [How to Vote with TaaV](../../tutorials/basic/how-to-vote-with-taav.md)
- [System Token](./system-token.md)
- [ParasInclusion](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/master/infrablockchain/runtime/parachains/src/inclusion/mod.rs)
