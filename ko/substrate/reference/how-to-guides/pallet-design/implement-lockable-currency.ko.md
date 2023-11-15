---
title: 잠금 가능한 통화 구현
description:
keywords:
  - 팔렛 디자인
  - 통화
  - 중간
  - 런타임
---

이 가이드는 사용자가 스테이킹과 투표를 위해 자금을 동결 할 수 있는 팔렛을 작성하는 방법을 보여줍니다.
[`LockableCurrency`](https://paritytech.github.io/substrate/master/frame_support/traits/trait.LockableCurrency.html) 트레이트는 교환 가능한 자원을 담보로 하여 책임 추적을 강제하는 경제 시스템의 맥락에서 유용합니다.
잠긴 자금을 관리하기 위해 Substrate [스테이킹 팔렛](https://paritytech.github.io/substrate/master/pallet_staking/index.html)을 사용할 수 있습니다.

이 가이드에서는 우리 자신의 커스텀 팔렛에서 `set_lock`, `extend_lock` 및 `remove_lock` 메서드를 구현할 것입니다.

## 시작하기 전에

이 가이드를 따르기 위해서는 이미 팔렛이 런타임에 통합되어 있어야 합니다.
이 가이드는 체인의 계정 및 잔액을 처리하기 위해 [Balances 팔렛](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/balances)을 포함하는 런타임을 사용한다고 가정합니다.
이 가이드를 따르기 위해 [노드 템플릿](https://github.com/substrate-developer-hub/substrate-node-template)의 템플릿 팔렛을 사용할 수 있습니다.

## 필요한 종속성 선언하기

[`LockableCurrency`](https://paritytech.github.io/substrate/master/frame_support/traits/trait.LockableCurrency.html)의 메서드들은 `frame_support`에서 몇 가지 트레이트를 가져와야 합니다.

1. 팔렛의 상단 섹션에 다음 트레이트들이 가져와져 있는지 확인하세요:

   ```rust
   use frame_support::{
   	dispatch::DispatchResult,
   	traits::{Currency, LockIdentifier, LockableCurrency, WithdrawReasons},
   };
   ```

1. `LockIdentifier` 상수를 선언하세요.

   `LockableCurrency`를 사용하기 위해서는 `LockIdentifier`를 선언해야 합니다.
   `[u8; 8]` 타입을 필요로 하기 때문에, 이 식별자는 여덟 글자로 이루어져야 합니다.

   ```rust
   const EXAMPLE_ID: LockIdentifier = *b"example ";
   ```

   이후에 포함할 메서드들을 선언하기 위해 이 식별자가 필요합니다.

## 커스텀 팔렛 타입 선언하기

커스텀 구성 타입을 정의하면 팔렛이 구현하려는 메서드들을 상속할 수 있습니다.

1. 잠금 가능한 통화 타입을 정의하세요. `StakeCurrency`라고 부르겠습니다:

   ```rust
   	type StakeCurrency: LockableCurrency<Self::AccountId, Moment = Self::BlockNumber>;
   ```

1. [`Currency`](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/currency/trait.Currency.html) 트레이트에서 [`Balance`](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/currency/trait.Currency.html#associatedtype.Balance)에 해당하는 타입을 만족시키기 위한 타입을 정의하세요:

   ```rust
   	type BalanceOf<T> =
   		<<T as Config>::StakeCurrency as Currency<<T as frame_system::Config>::AccountId>>::Balance;
   ```

   이렇게 하면 팔렛이 구현할 메서드들의 `amount` 필드를 처리할 타입이 있게 됩니다.

1. 런타임에 대한 새로운 타입을 지정하세요:

   ```rust
   	impl pallet_template::Config for Runtime {
   		type RuntimeEvent = RuntimeEvent;
   		type StakeCurrency = Balances; // <- 이 줄을 추가하세요
   	}
   ```

   `Balances`를 전달하면 팔렛의 `LockableCurrency` 메서드들이 블록체인의 계정 잔액을 처리하는 팔렛과 동일한 `Balance`에 대한 이해를 갖게 됩니다.

## 필요한 메서드를 사용하여 실행 가능한 함수 생성하기

필요한 메서드들은 다음과 같습니다:

- `fn lock_capital`: 호출자로부터 지정된 양의 토큰을 잠급니다.
- `fn extend_lock`: 동결 기간을 연장합니다.
- `fn unlock_all`: 모든 동결된 토큰을 동결해제합니다.

1. `lock_capital`을 작성하고, `T::StakeCurrency::set_lock`을 사용하세요:

   ```rust
   	#[pallet::weight(10_000 + T::DbWeight::get().writes(1))]
   	pub(super) fn lock_capital(
   		origin: OriginFor<T>,
   		#[pallet::compact] amount: BalanceOf<T>
   	) -> DispatchResultWithPostInfo {

   		let user = ensure_signed(origin)?;

   		T::StakeCurrency::set_lock(
   			EXAMPLE_ID,
   			&user,
   			amount,
   			WithdrawReasons::all(),
   		);

   		Self::deposit_event(Event::Locked(user, amount));
   		Ok(().into())
   	}
   ```

1. `extend_lock`을 작성하고, `T::StakeCurrency::extend_lock`을 사용하세요:

   ```rust
   	#[pallet::weight(1_000)]
   	pub(super) fn extend_lock(
   		origin: OriginFor<T>,
   		#[pallet::compact] amount: BalanceOf<T>,
   	) -> DispatchResultWithPostInfo {
   		let user = ensure_signed(origin)?;

   		T::StakeCurrency::extend_lock(
   			EXAMPLE_ID,
   			&user,
   			amount,
   			WithdrawReasons::all(),
   		);

   		Self::deposit_event(Event::ExtendedLock(user, amount));
   		Ok(().into())
   	}
   ```

1. `unlock_all`을 작성하고, `T::StakeCurrency::remove_lock`을 사용하세요:

   ```rust
   	#[pallet::weight(1_000)]
   	pub(super) fn unlock_all(
   		origin: OriginFor<T>,
   	) -> DispatchResultWithPostInfo {
   		let user = ensure_signed(origin)?;

   		T::StakeCurrency::remove_lock(EXAMPLE_ID, &user);

   		Self::deposit_event(Event::Unlocked(user));
   		Ok(().into())
   	}
   ```

1. 세 가지 디스패처 함수에 대한 이벤트를 작성하세요:

   ```rust
   	#[pallet::event]
   	#[pallet::generate_deposit(pub(super) fn deposit_event)]
   	pub enum Event<T: Config> {
   		Locked(T::AccountId, BalanceOf<T>),
   		Unlocked(T::AccountId),
   		LockExtended(T::AccountId, BalanceOf<T>),
   	}
   ```

## 예시

- [`lockable-currency`](https://github.com/substrate-developer-hub/substrate-how-to-guides/blob/main/example-code/template-node/pallets/lockable-currency/src/lib.rs) 예시 팔렛

## 관련 자료

- [Currency 트레이트](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/currency/trait.Currency.html)
- [LockableCurrency](https://paritytech.github.io/substrate/master/frame_support/traits/trait.LockableCurrency.html)
- [LockIdentifier](https://paritytech.github.io/substrate/master/frame_support/traits/type.LockIdentifier.html)