---
title: InfraBlockchain Reward System
description: This document covers the overall content related to validator rewards.
keywords:
  - system token
  - validator incentive
---

## Before you begin

Before you begin, Make sure you have the following:

- [System Token](./system-token.md)

## Overview

**_InfraBlockchain_** uses a legal currency-based _system token_ as transaction fees. The transaction fees paid are rewarded to validators.

In this document, we will explore how transaction fees are collected and distributed to validators.

## Using Transaction Fees

**_InfraBlockchain_** requires transaction fees to be paid in order to generate transactions. These fees can be paid using the legal currency-based system token.

The fees can be paid directly by the user generating the transaction or by another user on their behalf. However, in the latter case, the electronic signature of the user paying the fee must also be included in the transaction.

The fees for transactions on the chain are collected in an account called the **bucket account**.

```rust
pub struct CreditToBucket;
impl HandleCredit<AccountId, Assets> for CreditToBucket {
    fn handle_credit(credit: CreditOf<AccountId, Assets>) {
        let dest = FeeTreasuryId::get().into_account_truncating();
        let _ = <Assets as Balanced<AccountId>>::resolve(&dest, credit);
    }
}

impl pallet_system_token_payment::Config for Runtime {
    type RuntimeEvent = RuntimeEvent;
    type Assets = Assets;
    type OnChargeSystemToken = TransactionFeeCharger<
        pallet_assets::BalanceToAssetBalance<Balances, Runtime, ConvertInto>,
        CreditToBucket,
    >;
    type FeeTableProvider = ();
    type VotingHandler = Pot;
    type PalletId = FeeTreasuryId;
}
```

System Token collected in the bucket account are teleported to the Sovereign account of the chain where System Token was originally issued at regular intervals (typically every 10 blocks).

```rust
#[pallet::hooks]
impl<T: Config> Hooks<BlockNumberFor<T>> for Pallet<T>
where
    u32: From<<T as frame_system::Config>::BlockNumber>,
    <<T as frame_system::Config>::RuntimeOrigin as OriginTrait>::AccountId:
        From<AccountIdOf<T>>,
    [u8; 32]: From<<T as frame_system::Config>::AccountId>,
    u128: From<<T as pallet_assets::Config>::Balance>,
{
    fn on_initialize(n: T::BlockNumber) -> Weight {
        if n % T::Period::get() == Zero::zero() {
            Self::do_aggregate_system_token();
            T::DbWeight::get().reads(3)
        } else {
            T::DbWeight::get().reads(0)
        }
    }
}
```

## Distribution to Validators

Through the above process, the collected transaction fees are calculated on a session basis and distributed as a share to the validators of that session.

```rust
/* snip */
if let Some(vote_result) = commitments.vote_result {
    /* snip */
    (
        /* snip */
        T::RewardInterface::aggregate_reward(
            session_index,
            vote.clone().system_token_id.para_id,
            system_token_id.clone(),
            weight,
        );
    };
}
/* snip */
```

```rust
fn aggregate_reward(
    session_index: SessionIndex,
    para_id: ParaId,
    system_token_id: SystemTokenId,
    amount: VoteWeight,
) {
    let amount: u128 = amount.into();

    if let Some(mut rewards) = RewardsByParaId::<T>::get(session_index, para_id.clone()) {
        for reward in rewards.iter_mut() {
            if reward.system_token_id == system_token_id {
                reward.amount += amount;
            }
        }
        RewardsByParaId::<T>::insert(session_index, para_id.clone(), rewards.clone());
    } else {
        let rewards = vec![ValidatorReward::new(system_token_id, amount)];
        RewardsByParaId::<T>::insert(session_index, para_id.clone(), rewards);
    }

    if let Some(mut rewards) = TotalSessionRewards::<T>::get(session_index) {
        for reward in rewards.iter_mut() {
            if reward.system_token_id == system_token_id {
                reward.amount += amount;
            }
        }
        TotalSessionRewards::<T>::insert(session_index, rewards.clone());
    } else {
        let rewards = vec![ValidatorReward::new(system_token_id, amount)];
        TotalSessionRewards::<T>::insert(session_index, rewards);
    }
}
```

The types and quantities of tokens confirmed as rewards are stored in the `ValidatorReward` storage, and validators can receive rewards based on the types and quantities of System Token stored in the `ValidatorReward` storage.

## Next steps

- [Receiving Validator Rewards](../../tutorials/basic/how-to-get-validator-reward.md)
