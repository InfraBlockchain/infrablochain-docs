---
title: privileged 호출과 origin
description:
keywords:
  - Origin
---

런타임 origin은 호출이 어디에서 왔는지 확인하기 위해 호출 가능한 함수에서 사용됩니다.

## 원시(raw) origins

Substrate는 런타임 팔레트에서 사용할 수 있는 세 가지 원시(raw) origin을 정의합니다:

```rust
pub enum RawOrigin<AccountId> {
	Root,
	Signed(AccountId),
	None,
}
```

- Root: 시스템 레벨 origin입니다. 이것은 가장 높은 특권 수준으로, 런타임 origin의 슈퍼유저로 생각할 수 있습니다.

- Signed: 트랜잭션 origin입니다. 이것은 온체인 계정의 개인 키로 서명되며, 서명자의 계정 식별자를 포함합니다. 이를 통해 런타임은 디스패치의 출처를 인증하고, 관련된 계정에 트랜잭션 수수료를 부과할 수 있습니다.

- None: origin의 부재입니다. 이것은 검증자들에 의해 합의되거나 모듈에 의해 검증되어 포함되어야 합니다.
  이 origin 유형은 특정 런타임 메커니즘을 우회하기 위해 설계되어 더 복잡합니다.
  예를 들어, 이 origin 유형은 검증자들이 데이터를 블록에 직접 삽입할 수 있도록 허용하기 위해 사용될 수 있습니다.

## Origin 호출

런타임 내에서 어떤 origin으로든 호출을 생성할 수 있습니다. 예를 들어:

```rust
// Root
proposal.dispatch(system::RawOrigin::Root.into())

// Signed
proposal.dispatch(system::RawOrigin::Signed(who).into())

// None
proposal.dispatch(system::RawOrigin::None.into())
```

이를 실제로 구현한 [Sudo 모듈](https://paritytech.github.io/substrate/master/pallet_sudo/index.html)의 소스 코드를 살펴볼 수 있습니다.

## 사용자 정의 origin

세 가지 핵심 origin 유형 외에도, 런타임 개발자는 사용자 정의 origin을 정의할 수 있습니다.
이는 런타임 팔레트 내에서 특정 모듈의 함수에서 권한 확인에 사용하거나, 런타임 요청의 origin에 대한 사용자 정의 액세스 제어 논리를 정의하는 데 사용될 수 있습니다.

origin을 사용자 정의하는 것은 런타임 개발자가 런타임 로직에 따라 유효한 origin을 지정할 수 있도록 합니다. 예를 들어, 특정 함수의 액세스를 특별한 사용자 정의 origin으로 제한하고, [collective](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/collective)의 멤버에서만 디스패치 호출을 인가하는 것이 바람직할 수 있습니다. 사용자 정의 origin을 사용하는 장점은 런타임 개발자에게 런타임으로의 디스패치 호출에 대한 특권적 액세스를 구성할 수 있는 방법을 제공한다는 것입니다.

## 다음 단계

### 더 알아보기

- [`#[pallet::call]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#call-palletcall-optional) 매크로에서 origin이 어떻게 사용되는지 알아보세요.

### 예제

- `Root` 및 `Signed` origin으로 호출할 수 있도록 하는 방법을 살펴보려면 [Sudo 팔레트](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/sudo)를 확인하세요.

- `None` origin으로 호출을 검증하는 방법을 살펴보려면 [Timestamp 팔레트](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/timestamp)를 확인하세요.

- 사용자 정의 `Member` origin을 구성하는 방법을 살펴보려면 [Collective 팔레트](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/collective)를 확인하세요.

- 사용자 정의 origin을 생성하고 사용하는 레시피를 확인하세요.

### 참고 자료

- [`RawOrigin` enum](https://paritytech.github.io/substrate/master/frame_system/enum.RawOrigin.html)에 대한 참조 문서 방문.