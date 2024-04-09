---
title: 인프라블록체인 보상 체계
description: 밸리데이터 보상에 대한 전반적인 내용을 다룹니다.
keywords:
  - 시스템 토큰
  - 밸리데이터 인센티브
---

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [시스템 토큰](./system-token.md)

## 개요

***인프라블록체인(InfraBlockchain)*** 은 법정화폐 기반의 _시스템 토큰_ 을 트랜잭션의 수수료로 사용하고 있습니다. 사용된 트랜잭션 수수료는 밸리데이터 에게 보상으로 지급됩니다. 

본 문서에서는 어떠한 방식으로 트랜잭션 수수료가 모여 밸리데이터에게 전달되는지 알아봅니다.

## 트랜잭션 수수료로 사용

***인프라블록체인(InfraBlockchain)*** 은 트랜잭션을 발생시키기 위해서 수수료를 지불해야 합니다. 이러한 수수료는 법정화폐 기반의 시스템 토큰을 사용하여 지불할 수 있습니다. 

수수료는 트랜잭션을 발생시키는 유저가 직접 지불하거나, 혹은 다른 유저로 하여금 수수료를 지불하게 할 수 있습니다. 하지만 후자의 경우 수수료를 지불하는 유저의 전자 서명이 트랜잭션에 포함 되어야 합니다.

체인에서 발생한 트랜잭션에 대한 수수료는 **버켓 계정(bucket account)** 이라고 불리는 계정에 모이게 됩니다.

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

버켓 계정에 모인 시스템 토큰은 일정 주기마다(기본적으로 10 블록) 시스템 토큰이 처음 발행된 체인의 Sovereign 계정으로 텔레포트 됩니다.

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

## 밸리데이터에게 분배

위 과정을 통해서 모인 트랜잭션 수수료는 세션 단위로 계산되어 해당 세션의 밸리데이터의 몫으로 분배 됩니다.

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
		};
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

보상으로 확정된 토큰의 종류 및 수량은 `ValidatorReward` 스토리지에 저장되며 스토리지 조회로 확인할 수 있습니다.

밸리데이터는 `ValidatorReward` 스토리지에 저장된 시스템 토큰의 종류와 수량을 기반으로 보상을 받을 수 있습니다.

## 다음 단계로 넘어가기

- [밸리데이터 보상 받기](../../tutorials/basic/how-to-get-validator-reward.md)


