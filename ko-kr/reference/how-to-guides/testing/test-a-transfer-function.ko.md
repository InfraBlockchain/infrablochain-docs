---
title: 전송 함수 테스트하기
description:
keywords:
  - 테스트
  - 런타임
  - 초보자
---

각 함수를 테스트하는 것은 제품용 팔레트를 개발하는 중요한 부분입니다.
이 가이드에서는 기본 `transfer` 함수에 대한 테스트 케이스 작성에 대한 모범 사례를 안내합니다.

## `transfer` 함수 개요

전송 함수에는 두 가지 주요 요소가 있습니다: 계정에서 잔액을 빼고 해당 잔액을 다른 계정에 추가하는 것입니다.
여기에서는 이 함수를 개요로 시작하겠습니다:

```rust
#[pallet::weight(10_000)]
pub (super) fn transfer(
  origin: OriginFor<T>,
  to: T::AccountId,
  #[pallet::compact] amount: T::Balance,
  ) -> DispatchResultWithPostInfo {
    let sender = ensure_signed(origin)?;

    Accounts::<T>::mutate(&sender, |bal| {
      *bal = bal.saturating_sub(amount);
      });
      Accounts::<T>::mutate(&to, |bal| {
        *bal = bal.saturating_add(amount);
        });

    Self::deposit_event(Event::<T>::Transferred(sender, to, amount))
    Ok(().into())
}
```

## 송신자의 잔액이 충분한지 확인

먼저, 송신자의 잔액이 충분한지 확인해야 합니다.
별도의 `tests.rs` 파일에서 첫 번째 테스트 케이스를 작성하세요:

```rust
#[test]
fn transfer_works() {
  new_test_ext().execute_with(|| {
    MetaDataStore::<Test>::put(MetaData {
			issuance: 0,
			minter: 1,
			burner: 1,
		});
    // 계정 2에 42개의 코인을 발행합니다.
    assert_ok!(RewardCoin::mint(RuntimeOrigin::signed(1), 2, 42));
    // 계정 2에서 계정 3으로 50개의 코인을 전송합니다.
    asset_noop!(RewardCoin::transfer(RuntimeOrigin::signed(2), 3, 50), Error::<T>::InsufficientBalance);
```

### 오류 처리 구성

일부 오류 확인을 구현하기 위해 `mutate`를 `try_mutate`로 대체하여 `ensure!`를 사용하세요.
이렇게 하면 _bal이 amount보다 크거나 같은지_ 확인하고 그렇지 않으면 오류 메시지를 throw합니다:

```rust
Accounts::<T>::try_mutate(&sender, |bal| {
  ensure!(bal >= amount, Error::<T>::InsufficientBalance);
  *bal = bal.saturating_sub(amount);
  Ok(())
});
```

팔레트 디렉토리에서 `cargo test`를 실행하세요.

## 송신 계정이 최소 잔액 아래로 내려가지 않는지 확인

`transfer_works` 함수 내부에서:

```rust
assert_noop!(RewardCoin::transfer(RuntimeOrigin::signed(2), 3, 50), Error::<Test>::InsufficientBalance);
```

## 두 테스트가 함께 작동하는지 확인

두 가지 확인을 감싸기 위해 `#[transactional]`을 사용하여 래퍼를 생성하세요:

```rust
#[transactional]
pub(super) fn transfer(
/*--snip--*/
```

## 더스트 계정 처리

송신 및 수신 계정이 더스트 계정이 아닌지 확인하세요. `T::MinBalance::get()`을 사용하세요:

```rust
/*--snip--*/
let new_bal = bal.checked_sub(&amount).ok_or(Error::<T>::InsufficientBalance)?;
ensure!(new_bal >= T::MinBalance::get(), Error::<T>::BelowMinBalance);
/*--snip--*/
```

## 예제

- [`reward-coin` 예제 팔레트의 테스트](https://github.com/substrate-developer-hub/substrate-how-to-guides/blob/main/example-code/template-node/pallets/reward-coin/src/tests.rs).

## 자원

- [`assert_ok!`](https://paritytech.github.io/substrate/master/frame_support/macro.assert_ok.html)
- [`assert_noop!`](https://paritytech.github.io/substrate/master/frame_support/macro.assert_noop.html)
- [`ensure!`](https://paritytech.github.io/substrate/master/frame_support/macro.ensure.html)
- [`try_mutate`](https://paritytech.github.io/substrate/master/frame_support/storage/trait.StorageMap.html#tymethod.try_mutate)