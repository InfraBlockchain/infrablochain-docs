---
title: 팔레트 결합
description:
keywords:
  - 결합
  - 팔레트 디자인
---

**결합(coupling)**이란 두 개의 소프트웨어 모듈이 서로 얼마나 의존하는지를 설명하는 용어로 자주 사용됩니다.
예를 들어, 객체 지향 프로그래밍에서는 강한 결합(tight coupling)과 약한 결합(loose coupling)이 객체 클래스 간의 관계를 설명하는 데 사용됩니다:

- 강한 결합은 두 개의 클래스 그룹이 서로 의존하는 경우입니다.
- 약한 결합은 한 클래스가 다른 클래스가 노출하는 인터페이스를 사용하는 경우입니다.

Substrate에서는 **강한 팔레트 결합(tight pallet coupling)**과 **약한 팔레트 결합(loose pallet coupling)**이라는 용어를 사용하여 팔레트가 다른 팔레트의 함수를 호출하는 방식을 설명합니다.
두 기술 모두 동일한 결과를 다른 방식으로 달성하며, 각각은 특정한 트레이드오프를 가지고 있습니다.

## 강하게 결합된 팔레트

강한 결합은 팔레트를 유연하고 확장 가능하게 만드는 데 제약이 있기 때문에, 특정 유형이나 메서드가 아닌 결합된 상대방을 **전체적으로 상속**해야 하는 경우에만 강한 팔레트 결합을 사용합니다.

강한 결합이 필요한 팔레트를 작성할 때는, 팔레트의 `Config` 트레이트를 명시적으로 결합할 팔레트의 `Config` 트레이트에 바인딩하도록 지정합니다.

모든 FRAME 팔레트는 `frame_system` 팔레트와 강하게 결합되어 있습니다.
다음 예제는 `some_pallet`이라는 팔레트의 `Config` 트레이트를 `frame_system` 팔레트와 강하게 결합하기 위해 사용하는 방법을 보여줍니다:

```rust
pub trait Config: frame_system::Config + some_pallet::Config {
    // --snip--
}
```

이는 객체 지향 프로그래밍에서 클래스 상속을 사용하는 것과 매우 유사합니다.
이 트레이트 바운드를 제공하면, 이 팔레트는 `some_pallet` 팔레트가 설치된 런타임에만 설치될 수 있다는 것을 의미합니다.
`frame_system`과 마찬가지로, 이 예제에서의 강한 결합은 결합된 팔레트의 **Cargo.toml** 파일에서 `some_pallet`을 지정해야 합니다.

강한 결합은 개발자가 고려해야 할 몇 가지 단점이 있습니다:

- **유지보수성**: 한 팔레트의 변경 사항은 종종 다른 팔레트를 수정해야 하는 결과를 초래합니다.
- **재사용성**: 한 모듈을 사용하려면 두 모듈 모두 포함되어야 하므로, 강하게 결합된 팔레트를 통합하는 것이 더 어려워집니다.

## 약하게 결합된 팔레트

약한 팔레트 결합에서는 특정 유형이 바인딩되어야 하는 트레이트와 함수 인터페이스를 지정할 수 있습니다.

이러한 유형의 실제 구현은 런타임 구성 중 **팔레트 외부에서** 발생하며, 일반적으로 `/runtime/src/lib.rs` 파일에 정의된 코드로 이루어집니다. 약한 결합을 사용하면, 트레이트를 구현한 다른 팔레트의 유형과 인터페이스를 사용하거나, 완전히 새로운 구조체를 선언하고 해당 트레이트를 구현한 다음 팔레트를 런타임에서 구현할 때 할당할 수 있습니다.

예를 들어, 계정 잔액에 액세스하고 다른 계정으로 이체할 수 있는 팔레트가 있다고 가정해 봅시다.
이 팔레트는 `Currency` 트레이트를 정의하며, 이 트레이트에는 나중에 실제 이체 로직을 구현할 **추상 함수 인터페이스**가 있습니다.

```rust
pub trait Currency<AccountId> {
    // -- snip --
    fn transfer(
        source: &AccountId,
        dest: &AccountId,
        value: Self::Balance,
        // don't worry about the last parameter for now
        existence_requirement: ExistenceRequirement,
    ) -> DispatchResult;
}
```

두 번째 팔레트에서는 `MyCurrency` 연관 타입을 정의하고 `Currency<Self::AccountId>` 트레이트에 바인딩하여 `T::MyCurrency::transfer(...)`를 호출하여 balance transfer 로직을 사용할 수 있도록 합니다.

```rust
pub trait Config: frame_system::Config {
    type MyCurrency: Currency<Self::AccountId>;
}

impl<T: Config> Pallet<T> {
    pub fn my_function() {
        T::MyCurrency::transfer(&buyer, &seller, price, ExistenceRequirement::KeepAlive)?;
    }
}
```

이 시점에서 `Currency::transfer()` 로직이 어떻게 구현될지는 지정하지 않았습니다.
구현될 위치에 대해서만 합의된 상태입니다.

이제 런타임 구성(`runtime/src/lib.rs`)을 사용하여 팔레트를 구현하고 타입을 `Balances`로 지정할 수 있습니다.

```rust
impl my_pallet::Config for Runtime {
    type MyCurrency = Balances;
}
```

`Balances` 타입은 [`pallet_balances`](https://paritytech.github.io/substrate/master/pallet_balances/index.html)의 일부로 `construct_runtime!` 매크로에서 지정되며 [`Currency` 트레이트](https://paritytech.github.io/substrate/master/pallet_balances/index.html#implementations-1)를 구현합니다.

런타임에서 제공되는 구현을 통해 약하게 결합된 팔레트에서 `Currency<AccountId>` 트레이트를 사용할 수 있습니다.

많은 FRAME 팔레트가 이러한 방식으로 [Currency 트레이트](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/currency/trait.Currency.html)에 결합되어 있습니다.

## 팔레트 결합 전략 선택하기

일반적으로, 약한 결합은 강한 결합보다 더 많은 유연성을 제공하며 시스템 설계 관점에서 더 나은 실천 방법으로 간주됩니다.
코드의 유지보수성, 재사용성 및 확장성을 보장합니다.
그러나 강한 결합은 복잡하지 않거나 메서드와 유형 사이에 차이가 더 적은 팔레트에 유용할 수 있습니다.

FRAME에서는 [`pallet_treasury`](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/treasury)에 강하게 결합된 두 개의 팔레트가 있습니다:

- [Bounties 팔레트](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/bounties)
- [Tipping 팔레트](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/tips)

일반적인 규칙으로는, 팔레트가 복잡할수록 강하게 결합되는 것이 덜 바람직합니다.
이는 소프트웨어 시스템의 전반적인 품질을 검토하는 데 사용되는 컴퓨터 과학 개념인 [응집도](<https://en.wikipedia.org/wiki/Cohesion_(computer_science)>)를 떠올리게 합니다.

## 다음으로 어디로 가야 할까요

- [약한 결합 사용하기](/reference/how-to-guides/pallet-design/use-loose-coupling/)
- [강한 결합 사용하기](/reference/how-to-guides/pallet-design/use-tight-coupling/)