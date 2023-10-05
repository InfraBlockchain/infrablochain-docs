---
title: 계정 데이터 구조
description: FRAME에서 계정에 사용되는 저장 맵 데이터 구조를 설명합니다.
keywords:
---

계정은 블록체인의 기본 요소이며, Substrate에서는 계정을 다양한 방식으로 사용할 수 있습니다.
이 문서에서는 Substrate에서 계정이 어떻게 저장되고 계정 데이터 구조가 런타임 로직에서 계정 라이프사이클을 관리하는 데 사용되는지 설명합니다.

## Account

`Account` 데이터 타입은 [`frame-system` 팔렛](https://paritytech.github.io/substrate/master/src/frame_system/lib.rs.html#530)에서 일반적으로 정의된 저장 맵입니다:

```rust
/// 특정 계정 ID에 대한 전체 계정 정보입니다.
#[pallet::storage]
#[pallet::getter(fn account)]
pub type Account<T: Config> = StorageMap<
  _,
  Blake2_128Concat,
  T::AccountId,
  AccountInfo<T::Nonce, T::AccountData>,
  ValueQuery,
>;
```

`Account`의 `StorageMap`은 다음과 같은 매개변수로 구성됩니다:

- 첫 번째 매개변수 (\_)는 매크로 확장에 사용됩니다.
- `Blake2_128Concat`은 사용할 해시 알고리즘을 지정합니다.
- `T::AccountId`는 `AccountInfo<T::Nonce, T::AccountData>` 구조체의 키로 사용됩니다.

자세한 내용은 [`StorageMap` API](https://paritytech.github.io/substrate/master/frame_support/storage/types/struct.StorageMap.html#impl)를 참조하십시오.

## AccountInfo

계정의 `AccountInfo`는 [`frame_system` 팔렛](https://paritytech.github.io/substrate/master/src/frame_system/lib.rs.html#788-803)에서 정의됩니다:

```rust
#[derive(Clone, Eq, PartialEq, Default, RuntimeDebug, Encode, Decode)]
pub struct AccountInfo<Nonce, AccountData> {
  /// 이 계정이 보낸 트랜잭션 수입니다.
  pub nonce: Nonce,
  /// 현재 이 계정의 존재에 의존하는 다른 모듈의 수입니다. 이 값이 0이 될 때까지 계정을 삭제할 수 없습니다.
  pub consumers: RefCount,
  /// 이 계정이 존재할 수 있도록 허용하는 다른 모듈의 수입니다. 이 값과 `sufficients`가 모두 0이 될 때까지 계정을 삭제할 수 없습니다.
  pub providers: RefCount,
  /// 이 계정이 자체 목적으로만 존재할 수 있도록 허용하는 모듈의 수입니다. 이 값과 `providers`가 모두 0이 될 때까지 계정을 삭제할 수 없습니다.
  pub sufficients: RefCount,
  /// 이 계정에 속하는 추가 데이터입니다. 많은 체인에서 잔고를 저장하는 데 사용됩니다.
  pub data: AccountData,
}
```

각 계정은 다음과 같은 `AccountInfo`로 구성됩니다:

- 계정이 보낸 트랜잭션 수를 나타내는 `nonce`.
- 계정의 존재에 의존하는 다른 모듈의 수를 나타내는 `consumers` 참조 카운터.
- 계정이 존재할 수 있도록 허용하는 다른 모듈의 수를 나타내는 `providers` 참조 카운터.
- 계정이 자체 목적으로만 존재할 수 있도록 허용하는 모듈의 수를 나타내는 `sufficients` 참조 카운터.
- 다양한 종류의 데이터를 보유할 수 있도록 구성할 수 있는 `AccountData` 구조체.

## 계정 참조 카운터

계정 참조 카운터는 런타임에서 계정 종속성을 추적합니다.
예를 들어, 계정이 제어하는 맵에 데이터를 저장하는 경우, 계정이 제어하는 맵에 저장된 데이터가 삭제될 때까지 계정을 삭제하고 싶지 않을 것입니다.

`consumers`와 `providers` 참조 카운터는 함께 사용되도록 설계되었습니다.
예를 들어, `Session` 팔렛에서는 계정이 검증자가 되기 전에 세션 키를 설정할 때 `consumers` 참조 카운터가 증가합니다.
`providers` 참조 카운터는 `consumer` 카운터를 증가시키기 전에 0보다 커야 합니다.

`providers` 참조 카운터는 계정이 의존할 수 있는지 여부를 나타냅니다.
예를 들어, 존재 예치금보다 큰 금액으로 새 계정을 생성할 때 `providers` 참조 카운터가 증가합니다 [[2]](#ref-system-created).

`providers` 참조 카운터는 Substrate 팔렛이 계정에 대한 데이터를 저장하지 못하도록 합니다 (`providers` > 0).
`consumers` 참조 카운터는 모든 팔렛에서 계정에 대한 데이터가 제거될 때까지 계정을 삭제하지 못하도록 합니다 (`consumers` == 0).
`consumers` 참조 카운터는 사용자가 체인 상에 저장된 데이터에 대해 책임을 집니다.
사용자가 계정을 제거하고 존재 예치금을 반환받으려면, 모든 팔렛에 저장된 모든 데이터를 제거하여 `consumers` 카운터를 감소시켜야 합니다.

팔렛에는 또한 `providers` 참조 카운터를 감소시켜 팔렛 관리 범위 내에서 계정을 비활성화하는 정리 함수가 있습니다.
계정의 `providers` 참조 카운터가 0이고 `consumers`가 0이면, 모든 팔렛에서 계정은 **비활성화**된 것으로 간주됩니다.

`sufficients` 참조 카운터는 계정이 자체적으로 충분하고 자체적으로 존재할 수 있는지 여부를 나타냅니다.
예를 들어, 자산 팔렛에서는 계정이 특정 자산의 충분한 수를 가질 수 있지만 네이티브 계정 잔고를 소유하지 않을 수 있습니다.

런타임 개발자는 `frame-system` 팔렛이 노출한 `inc_consumers()`, `dec_consumers()`, `inc_providers()`, `dec_providers()`, `inc_sufficients()`, `dec_sufficients()` 메서드를 사용하여 이러한 카운터를 업데이트할 수 있습니다.
특정 카운터의 증가 호출은 계정 라이프사이클에서 해당 카운터의 감소 호출과 함께 이루어져야 합니다.

이러한 카운터를 사용하기 쉽게하기 위해 세 가지 쿼리 함수도 제공됩니다:

- `can_inc_consumer()`는 계정이 사용 준비가 되었는지 확인합니다 (`providers` > 0).
- `can_dec_provider()`는 계정이 런타임에서 더 이상 참조되지 않는지 확인한 후 (`consumers` == 0) `providers`를 0으로 감소시킵니다.
- `is_provider_required()`는 계정에 미결된 소비자 참조가 있는지 확인합니다 (`consumers` > 0).

자세한 내용은 [`frame-system` API](https://paritytech.github.io/substrate/master/frame_system/pallet/struct.Pallet.html#impl-11)를 참조하십시오.

## AccountData 트레이트와 구현

`AccountInfo`는 [`frame-system::pallet::Config` 트레이트](https://paritytech.github.io/substrate/master/frame_system/pallet/trait.Config.html#associatedtype.AccountData)에서 정의된 연관 타입 `AccountData` 트레이트 바운드를 만족하는 한 구조체일 수 있습니다.
기본적으로 Substrate 런타임은 `AccountInfo`를 [`pallet-balances`](https://paritytech.github.io/substrate/master/pallet_balances/struct.AccountData.html)에서 정의된 대로 구성합니다.

## 다음으로 이동할 위치

추가 기술적 세부 정보는 다음 리소스를 참조하십시오:

- [`frame_system::AccountInfo` API](https://paritytech.github.io/substrate/master/frame_system/struct.AccountInfo.html)
- [`pallet_balances::AccountData` API](https://paritytech.github.io/substrate/master/pallet_balances/struct.AccountData.html).
- [`pallet_session::Pallet::set_keys` 디스패처 호출](https://paritytech.github.io/substrate/master/src/pallet_session/lib.rs.html#508-571)
- [`frame_system::Provider` `HandleLifetime` 구현](https://paritytech.github.io/substrate/master/src/frame_system/lib.rs.html#1549-1561)
- [`pallet_assets` `new_account` 함수](https://paritytech.github.io/substrate/master/src/pallet_assets/functions.rs.html#46-61)