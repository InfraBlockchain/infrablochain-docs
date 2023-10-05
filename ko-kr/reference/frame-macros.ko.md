---
title: FRAME 매크로
description:
keywords:
---

Substrate은 런타임을 위해 구현한 팔렛들에서 코드를 생성하고 로직을 집계하기 위해 커스텀 [Rust 매크로](https://doc.rust-lang.org/book/ch19-06-macros.html)를 사용합니다.
이러한 런타임 매크로를 사용하면 체인 상의 변수를 인코딩 및 디코딩하는 데 시간을 소비하지 않고 런타임 로직에 집중할 수 있으며, [기본적인 블록체인 개발](/learn/runtime-development#core-primitives)에 필요한 코드를 중복해서 작성할 필요가 없습니다.

이 섹션에서는 Rust에서 사용 가능한 매크로 유형에 대한 개요를 제공하고, 특정 FRAME 매크로가 런타임 개발에서 어떻게 사용되는지 강조합니다.

## 매크로 기본 사항

컴퓨터 프로그래밍에서 매크로는 실행할 지시문의 미리 정의된 시퀀스를 캡슐화하는 코드 라인입니다.
코드를 작성하는 코드로, 매크로를 사용하면 반복적인 작업을 추상화하고 작성해야 하는 코드를 단순화할 수 있습니다.
매크로를 사용하면 복잡한 데이터 구조를 암묵적으로 선언할 수 있습니다.

Rust에서 매크로는 **선언적 매크로** 또는 **절차적 매크로**로 구분됩니다.

[선언적 매크로](https://doc.rust-lang.org/book/ch19-06-macros.html#declarative-macros-with-macro_rules-for-general-metaprogramming)는 패턴을 선언하고 표현식의 결과를 패턴과 비교한 다음, 패턴이 일치하는지 여부에 따라 추가 코드 라인을 실행할 수 있습니다.
선언적 매크로는 Rust 프로그래밍에서 널리 사용됩니다.

[절차적 매크로](https://doc.rust-lang.org/book/ch19-06-macros.html#procedural-macros-for-generating-code-from-attributes)는 함수와 유사합니다.
선언적 매크로에서 수행되는 패턴 매칭과 달리, 절차적 매크로는 코드를 입력으로 받아 일련의 명령을 수행하고 코드를 출력으로 생성합니다.
절차적 매크로에는 세 가지 유형이 있습니다:

- [사용자 정의 파생 매크로](https://doc.rust-lang.org/book/ch19-06-macros.html#how-to-write-a-custom-derive-macro)는 특정 타입에 대한 특성의 정의와 재사용을 가능하게 합니다.
  `derive` 매크로는 특히 특정 특성을 만족해야 하는 사용자 정의 런타임 타입의 구현을 정의하는 데 유용합니다.
- [속성과 유사한 매크로](https://doc.rust-lang.org/book/ch19-06-macros.html#attribute-like-macros)는 코드를 생성하기 위해 새로운 속성을 생성할 수 있습니다.
- [함수와 유사한 매크로](https://doc.rust-lang.org/book/ch19-06-macros.html#function-like-macros)는 코드를 생성하기 위해 함수 호출처럼 동작하는 매크로를 정의할 수 있습니다.

매크로는 컴파일러가 포함된 코드 라인을 해석하기 전에 확장되기 때문에 복잡한 데이터 구조와 작업을 정의할 수 있습니다.
FRAME 매크로는 종종 복잡한 코드 블록에 대한 단축 추상화를 제공하기 위해 다양한 유형의 매크로를 활용합니다.
그러나 매크로가 제공하는 추상화는 런타임 코드를 다소 이해하기 어렵게 만들 수 있습니다.

Substrate 런타임에서 사용되는 FRAME 매크로에 대해 자세히 알아보려면 [cargo-expand](https://docs.rs/crate/cargo-expand)를 설치하고 사용할 수 있습니다.
cargo-expand를 설치한 후 `cargo expand` 명령을 사용하여 런타임에서 사용하는 매크로에 포함된 코드의 결과를 표시할 수 있습니다.

## FRAME 지원 및 시스템 매크로

Substrate 기본 요소와 FRAME은 각각 다양한 매크로 집합에 의존합니다.
이 섹션에서는 FRAME 지원 및 FRAME 시스템 라이브러리에서 제공하는 매크로에 대한 개요를 제공합니다.
대부분의 경우, 이러한 매크로는 다른 팔렛들이 의존하는 프레임워크를 제공하며, 런타임 로직에서 어떻게 사용되는지에 대해 알아야 합니다.
개요 이후에는 런타임 개발자로서 가장 자주 사용할 매크로에 대해 설명합니다.

### FRAME 지원 매크로

`frame_support` 크레이트는 런타임에서 사용되는 많은 중요한 선언적, 파생, 속성과 유사한, 함수와 유사한 매크로를 제공합니다.
`frame_support` 크레이트에서 알아야 할 중요한 매크로 중 일부는 다음과 같습니다:

- `construct_runtime`: 구현한 팔렛 목록에서 런타임을 구성하는 데 사용됩니다.
- `match_types`: `matches!`와 유사한 구문을 사용하여 `Contains` 특성을 구현하는 타입을 생성하는 데 사용됩니다.
- `parameter_types`: `Get` 특성의 새로운 구현을 생성하는 데 사용됩니다.

`frame_support` 크레이트의 매크로에 대한 자세한 정보는 [Macros](https://paritytech.github.io/substrate/master/frame_support/index.html#macros), [Derive macros](https://paritytech.github.io/substrate/master/frame_support/index.html#derives), [Attribute macros](https://paritytech.github.io/substrate/master/frame_support/index.html#attributes)를 위한 Rust 문서를 참조하십시오.

### FRAME 시스템 매크로

`frame_system` 크레이트는 핵심 데이터 타입 및 공유 유틸리티에 대한 액세스를 제공하는 원시 타입을 정의하기 위해 매크로를 사용합니다.
이러한 원시 타입과 관련된 매크로는 외부 노드 및 런타임에서 많은 노드 작업에 기반을 제공하며, 다른 팔렛들이 Substrate 프레임워크와 상호 작용하는 기본 레이어 역할을 합니다.

`frame_system` 크레이트에서 알아야 할 중요한 원시 타입과 매크로 중 일부는 다음과 같습니다:

- [`sp_core`](https://paritytech.github.io/substrate/master/sp_core/index.html)

  - `map`: 배열에서 키-값 컬렉션을 초기화하는 데 사용됩니다.
  - `RuntimeDebug`: 런타임을 디버깅하는 데 사용됩니다.

  `sp_core` 함수와 유사한 매크로에 대한 자세한 정보는 [Macros](https://paritytech.github.io/substrate/master/sp_core/index.html#macros)를 위한 문서를 참조하십시오.

- [`sp_runtime`](https://paritytech.github.io/substrate/master/sp_runtime/index.html)

  - `bounded_btree_map`: 주어진 리터럴에서 제한된 `btree-map`을 빌드하는 데 사용됩니다.
  - `bounded_vec`: 주어진 리터럴에서 제한된 `vec`을 빌드하는 데 사용됩니다.
  - `impl_opaque_keys`: 설명된 데이터 구조에 대해 `OpaqueKeys`를 구현하는 데 사용됩니다.
  - `parameter_types`: `Get` 특성의 새로운 구현을 생성하는 데 사용됩니다.

  `sp_runtime` 함수와 유사한 매크로에 대한 자세한 정보는 [Macros](https://paritytech.github.io/substrate/master/sp_runtime/index.html#macros)를 위한 문서를 참조하십시오.
  `sp_runtime` 파생 매크로에 대한 정보는 [Derive macros](https://paritytech.github.io/substrate/master/sp_runtime/index.html#derives)를 위한 문서를 참조하십시오.

- [`sp_api`](https://paritytech.github.io/substrate/master/sp_api/index.html)

  - `decl_runtime_apis`: 지정된 특성을 런타임 API로 선언하는 데 사용됩니다.
  - `impl_runtime_apis`: 지정된 특성 구현을 런타임 API로 태그하는 데 사용됩니다.

  `sp_api` 함수와 유사한 매크로에 대한 자세한 정보는 [Macros](https://paritytech.github.io/substrate/master/sp_api/index.html#macros)를 위한 문서를 참조하십시오.

- [`sp_std`](https://paritytech.github.io/substrate/master/sp_std/index.html)

  - `if_std`: `std` 기능 집합이 활성화된 경우에만 실행되어야 하는 코드를 나타냅니다.
  - `map`: 배열에서 키-값 컬렉션을 초기화하는 데 사용됩니다.
  - `vec`: 인수를 포함하는 벡터를 생성하는 데 사용됩니다.

  `sp_std` 함수와 유사한 매크로에 대한 자세한 정보는 [Macros](https://paritytech.github.io/substrate/master/sp_std/index.html#macros)를 위한 문서를 참조하십시오.

- [`sp_version`](https://paritytech.github.io/substrate/master/sp_version/index.html)

  - `create_apis_vec`: API 선언의 벡터를 생성하는 데 사용됩니다.
  - `create_runtime_str`: `RuntimeString` 상수를 생성하는 데 사용됩니다.
  - `runtime_version`: 런타임의 버전 선언을 받아들이고 해당 내용과 동등한 사용자 정의 WebAssembly 섹션을 생성하는 속성으로 사용됩니다.

이러한 크레이트는 노드 템플릿의 런타임 및 노드 `Cargo.toml` 파일의 종속성 목록에 많이 나열됩니다.

## 팔렛 구성을 위한 매크로

[사용자 정의 팔렛 빌드](/learn/runtime-development#building-custom-pallets)에서 설명한 대로, 대부분의 FRAME 팔렛은 공통 섹션 집합을 사용하여 구성됩니다.

매크로를 사용하면 각 섹션을 더 모듈화하고 확장 가능하게 구축할 수 있습니다.
이 섹션에서는 사용 가능한 매크로와 사용자 정의 런타임을 구축하기 위해 이러한 매크로를 사용하는 방법에 대해 설명합니다.

### #[pallet]

`#[pallet]` 매크로는 팔렛을 선언하는 데 필요합니다.
이 [속성 매크로](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html)는 팔렛 모듈(`mod pallet`)의 속성입니다.
`pallet` 모듈 내에서 `#[pallet]` 매크로는 팔렛이 필요로 하는 특정 항목을 식별하는 데 사용되는 `#[pallet::*]` 매크로에 대한 진입점으로 작동합니다.
예를 들어, 팔렛에는 일반적으로 `construct_runtime!` 매크로를 통해 런타임을 구축하는 데 사용되는 `#[pallet::*]` 매크로로 집계되는 일련의 타입, 함수 및 특성 구현이 포함됩니다.

```rust
#[pallet]
pub mod pallet {
...
}
```

#### 개발 모드

`#[pallet]` 또는 `#[frame_support::pallet]` 속성 매크로에 `dev_mode`를 인수로 지정하여 팔렛에 대한 개발 모드를 활성화할 수 있습니다.
예를 들어, 작업 중인 팔렛에 개발 모드를 활성화하려면 `#[pallet]`을 `#[pallet(dev_mode)]`로, 또는 `#[frame_support::pallet]`을 `#[frame_support::pallet(dev_mode)]`로 바꿉니다.

개발 모드는 제품 팔렛에 대해 적용되는 제약과 요구 사항을 완화하여 개발 및 테스트 주기 동안 코드를 반복적으로 작성하기 쉽게 만듭니다.
예를 들어, 팔렛에 개발 모드를 활성화하면 다음과 같은 제약이 완화됩니다:

- `#[pallet::call]` 선언에 대해 모든 호출에 가중치를 지정할 필요가 없습니다.
  기본적으로 개발 모드는 가중치가 명시적으로 지정되지 않은 호출에 가중치 0을 할당합니다.

- 스토리지 타입에 대해 `MaxEncodedLen`을 구현할 필요가 없습니다.
  기본적으로 개발 모드는 모든 스토리지 항목을 무제한으로 표시합니다.

개발 모드를 활성화한 팔렛을 제품 네트워크에 배포해서는 안 됩니다. 제품 런타임에서 팔렛을 배포하기 전에 `#[pallet]` 선언에서 `dev_mode` 인수를 제거하고 컴파일러 오류를 수정하고 개발 모드를 비활성화한 상태에서 테스트를 완료해야 합니다.

#### 팔렛 모듈 사용

모듈 내에서 매크로는 `#[pallet::*]` 속성을 가진 항목을 구문 분석합니다.
일부 `#[pallet::*]` 속성은 필수이고 일부는 선택적입니다.

`frame_support` 및 `frame_system` 크레이트의 시스템 수준 타입을 `frame_support` 및 `frame_system` 크레이트의 `pallet_prelude`를 사용하여 자동으로 가져올 수 있습니다.
예를 들어:

```rust
#[pallet]
pub mod pallet {
		use frame_support::pallet_prelude::*;
		use frame_system::pallet_prelude::*;
		...
}
```

`#[pallet]` 매크로는 팔렛 타입과 특성 구현을 확장하기 위해 입력을 읽어 팔렛 타입과 특성 구현을 생성하는 파생 매크로와 유사한 방식으로 동작합니다.
대부분의 경우 매크로는 입력을 수정하지 않습니다.
그러나 매크로가 입력을 수정하는 몇 가지 특정 시나리오가 있습니다.

매크로는 다음과 같은 경우에 입력을 수정합니다:

- **일반적인**이 **타입**으로 대체될 때

  예를 들어, `pallet::storage` 매크로에서 `pub struct Pallet<..>(_)`의 내부 타입이 `StorageInstance` 특성을 구현하는 타입으로 대체될 때 발생할 수 있습니다.

- **함수 또는 데이터 구조**가 **변경**될 때

  예를 들어, `pallet::type_value` 매크로에서 함수 항목을 구조체와 특성 구현으로 변경할 수 있습니다.

- **문서**가 **제공되지 않은** 경우

  예를 들어, 문서가 제공되지 않은 경우 `pallet::pallet` 매크로는 `struct Pallet<T>(_);` 항목 위에 문서를 추가하기 위해 입력을 수정합니다.

### #[pallet::config]

[`#[pallet::config]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#config-trait-palletconfig-mandatory) 매크로는 팔렛이 사용하는 제네릭 데이터 타입을 정의하는 데 필요합니다.

이 매크로는 팔렛에 대한 시스템 수준 [`Config` 특성](https://paritytech.github.io/substrate/master/frame_system/pallet/trait.Config.html)의 상수를 제공합니다.

이 매크로에 대한 Config 특성은 `frame_system::Config` 특성을 포함하는 일반적인 특성 정의인 `Config`라는 일반적인 특성으로 정의되어야 합니다.
이 정의에는 다른 최상위 특성과 where 절이 포함될 수 있습니다.
예를 들어:

```rust
#[pallet::config]
pub trait Config: frame_system::Config + $optionally_some_other_supertraits
$optional_where_clause
{
...
}
```

`frame_system::Config` 요구 사항을 우회하기 위해 `#[pallet::disable_frame_system_supertrait_check]` 속성을 사용할 수 있습니다.
예를 들어:

```rust
#[pallet::config]
#[pallet::disable_frame_system_supertrait_check]
pub trait Config: pallet_timestamp::Config {}
```

### #[pallet::constant]

`#[pallet::constant]` 매크로는 `Config` 특성 - [`#[pallet::config]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#config-trait-palletconfig-mandatory) 매크로 내에서 - 에게 런타임 및 메타데이터 생성에 필요한 유형과 속성을 제공합니다.

이 매크로는 팔렛에서 사용하는 상수에 대한 정보를 런타임 메타데이터에 추가합니다. 이 정보에는 다음이 포함됩니다:

- 상수 이름
- 연관된 타입의 이름
- 상수 값
- 상수에 대한 `Get::get()`의 반환 값

예를 들어, 다음과 같이 `#[pallet::constant]`를 사용하여 `type MyGetParam`을 메타데이터에 추가할 수 있습니다:

```rust
#[pallet::config]
pub trait Config: frame_system::Config {
	#[pallet::constant] // 속성을 메타데이터에 추가
	type MyGetParam: Get<u32>;
}
```

### #[pallet::extra_constants]

[`#[pallet::extra_constants]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#extra-constants-palletextra_constants-optional) 매크로를 사용하면 메타데이터에 상수를 추가할 수 있습니다.

예를 들어, 생성된 값을 반환하는 함수를 선언할 수 있습니다.
그런 다음 `#[pallet::extra_constants]` 매크로를 사용하여 생성된 값에 대한 정보를 메타데이터에 추가할 수 있습니다:

```rust
#[pallet::extra_constants]
impl<T: Config> Pallet<T> {
  // extra_constants를 사용하는 예제 함수
  fn example_extra_constants() -> u128 { 4u128 }
}
```

### #[pallet::pallet]

[`#[pallet::pallet]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#pallet-struct-placeholder-palletpallet-mandatory) 매크로는 `construct_runtime!` 매크로에서 사용할 팔렛 데이터 구조 플레이스홀더를 선언하는 데 필요합니다.
이 매크로는 제네릭 타입 `<T>`와 where 절이 없는 `Pallet`이라는 이름의 구조체로 정의되어야 합니다.

예를 들어:

```rust
#[pallet::pallet]
pub struct Pallet<T>(_);
```

이 매크로는 `pallet::storage` 매크로에서 스토리지 항목마다 연관 타입을 포함하는 `Store` 특성을 생성할 수 있습니다. 이를 위해 `#[pallet::generate_store($vis trait Store)]` 속성 매크로를 제공해야 합니다.

예를 들어:

```rust
#[pallet::pallet]
pub struct Pallet<T>(_);
```

스토리지와 이 매크로를 사용하는 방법에 대한 자세한 정보는 `struct Pallet<T>` 정의에 추가된 [매크로 확장](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#macro-expansion-1)을 참조하십시오.

### #[pallet::without\_storage\_info]

`#[pallet::without_storage_info]` 매크로를 사용하면 고정 크기가 없는 팔렛 스토리지 항목을 정의할 수 있습니다.

기본적으로 모든 팔렛 스토리지 항목은 `traits::StorageInfoTrait`를 구현해야 하므로 모든 키 및 값 타입이 `pallet_prelude::MaxEncodedLen` 속성에서 정의된 바운드를 기반으로 고정 크기를 가져야 합니다.
이 크기 제한은 파라체인 개발에서 유효성 증명(Proof of Validity, PoV) 블롭의 크기를 추정하기 위해 필요합니다.

`#[pallet::without_storage_info]` 속성 매크로를 사용하면 특정 팔렛의 모든 스토리지 항목에 대한 기본 동작을 재정의할 수 있습니다.
사용하려면 팔렛 구조체에 `#[pallet::without_storage_info]` 속성을 추가하면 됩니다:

```rust
#[pallet::pallet]
#[pallet::generate_store(pub(super) trait Store)]
#[pallet::without_storage_info]
pub struct Pallet<T>(_);
```

`#[pallet::without_storage_info]` 매크로는 팔렛의 모든 스토리지 항목에 대해 기본 동작을 재정의해야 하는 경우에만 사용해야 합니다.
특정 스토리지 항목에 대해 정의되지 않은 스토리지를 필요로 하는 경우에는 고정 크기 제약을 재정의하기 위해 `#[pallet::unbounded]` 속성 매크로를 사용할 수 있습니다.

`#[pallet::without_storage_info]` 매크로는 팔렛의 모든 스토리지 항목에 적용되므로 테스트 또는 개발 환경에서만 사용해야 합니다.
제품 환경에서는 `#[pallet::without_storage_info]` 속성 매크로를 사용해서는 안 됩니다.

스토리지와 이 매크로를 사용하는 방법에 대한 자세한 정보는 `struct Pallet<T>` 정의에 추가된 [매크로 확장](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#pallet-struct-placeholder-palletpallet-mandatory)을 참조하십시오.

### #[pallet::unbounded]

`#[pallet::unbounded]` 속성 매크로를 사용하면 특정 스토리지 항목을 고정 크기가 없는 것으로 선언할 수 있습니다.
기본적으로 모든 팔렛 스토리지 항목은 고정 크기를 가져야 하므로 이 속성 매크로를 사용하여 특정 스토리지 항목의 기본 요구 사항을 재정의할 수 있습니다.
파라체인 개발자인 경우 이 매크로를 사용하여 PoV(Proof of Validity) 블롭에 포함되지 않을 스토리지 항목에 대해 사용할 수 있습니다.

### #[pallet::hooks]

[`#[pallet::hooks]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#hooks-pallethooks-optional) 매크로를 사용하면 팔렛에서 블록 생성 프로세스의 특정 시점에서 팔렛별 로직을 실행할 수 있는 선택적 팔렛 후크를 선언할 수 있습니다.
`#[pallet::hooks]` 매크로 내에서 [`Hooks`](https://paritytech.github.io/substrate/master/frame_support/traits/trait.Hooks.html#provided-methods) 특성을 구현하여 블록이 초기화되거나 완료되기 전, 런타임이 업그레이드되기 전 또는 런타임 업그레이드가 완료된 후에 로직을 실행할 수 있습니다.

예를 들어:

```rust
#[pallet::hooks]
impl<T: Config> Hooks<BlockNumberFor<T>> for Pallet<T> {
  // 후크 함수와 로직을 여기에 작성합니다.
}
```

후크는 런타임이 초기화되거나 완료되기 전, 런타임이 업그레이드되기 전 또는 런타임 업그레이드가 완료된 후에 로직을 실행하기 위해 사용될 수 있습니다.

후크에 대한 자세한 정보는 [pallet::hooks](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html##hooks-pallethooks-optional) 및 [매크로 확장](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#macro-expansion-2)을 위한 Rust 문서를 참조하십시오.

### #[pallet::call]

[`#[pallet::call]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet#call-palletcall-optional)은 팔렛에 대해 런타임으로 디스패치할 수 있는 함수를 구현하는 데 필요합니다. 각 함수는 다음을 충족해야 합니다:

- `#[pallet::weight($expr)]` 속성으로 가중치를 정의해야 합니다.
- 첫 번째 인수로 `origin: OriginFor<T>`를 가져야 합니다.
- 인수에는 `#[pallet::compact]`를 사용하여 인코딩을 압축해야 합니다.
- `DispatchResultWithPostInfo` 또는 `DispatchResult`를 반환해야 합니다.

런타임으로 들어오는 외부 요청은 호출을 사용하여 특정 로직을 트리거할 수 있습니다.
호출은 런타임에서 이벤트를 발생시키는 팔렛에 사용될 수도 있습니다.
`#[pallet::call]`은 런타임에서 팔렛의 모든 함수 호출 로직을 [`Call` 열거형](https://paritytech.github.io/substrate/master/frame_system/pallet/enum.Call.html)을 사용하여 집계합니다.
이 [집계](/reference/glossary#aggregation)는 FRAME이 동일한 유형의 함수를 단일 런타임 호출로 묶을 수 있도록 합니다.
런타임은 그런 다음 런타임에서 생성된 항목을 팔렛으로부터 생성된 이벤트, 오류 및 호출 함수로 매핑합니다.

자세한 내용은 [pallet::call](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#call-palletcall-optional)을 위한 Rust 문서를 참조하십시오.

### #[pallet::error]

[`#[pallet::error]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#error-palleterror-optional) 매크로를 사용하면 런타임으로 디스패치되는 함수 호출에서 반환될 수 있는 오류 타입을 정의할 수 있습니다.
오류 정보는 런타임 메타데이터에 포함됩니다.

매크로는 `Error`라는 제네릭 타입 <T>에 대한 열거형으로 정의되어야 하며, 필드가 있는 또는 없는 변형을 포함할 수 있습니다.

예를 들어, 다음과 같이 `#[pallet::error]`를 사용하여 `type MyError`를 메타데이터에 추가할 수 있습니다:

```rust
#[pallet::error]
pub enum Error<T> {
	/// $some_optional_doc
	$SomeFieldLessVariant,
	/// $some_more_optional_doc
	$SomeVariantWithOneField(FieldType),
	...
}
```

열거형 변형에 대해 지정한 필드 타입은 `scale_info::TypeInfo` 특성을 구현해야 하며, 인코딩된 크기는 가능한 작아야 합니다.
열거형 변형의 필드 타입은 또한 [PalletError](https://paritytech.github.io/substrate/master/frame_support/traits/trait.PalletError.html) 특성을 구현해야 합니다.

자세한 내용은 [pallet::error](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#error-palleterror-optional)을 위한 Rust 문서를 참조하십시오.

### #[pallet::event]

[`#[pallet::event]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#event-palletevent-optional) 매크로를 사용하면 팔렛에 대한 이벤트 타입을 정의할 수 있습니다.

이 매크로는 `pallet::error` 매크로와 유사하지만 더 많은 정보를 포함할 수 있습니다.
매크로는 이벤트라는 이름의 열거형으로 정의됩니다.

예를 들어:

```rust
#[pallet::event]
#[pallet::generate_deposit($visibility fn deposit_event)] // 선택적
pub enum Event<$some_generic> $optional_where_clause {
	/// Some doc
	$SomeName($SomeType, $YetanotherType, ...),
	...
}
```

자세한 내용은 [pallet::event](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#event-palletevent-optional)을 위한 Rust 문서를 참조하십시오.

### #[pallet::storage]

[`#[pallet::storage]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#storage-palletstorage-optional) 매크로를 사용하면 런타임 스토리지 내에서 추상적인 스토리지를 정의하고 해당 스토리지에 대한 메타데이터를 설정할 수 있습니다.
이 속성 매크로는 여러 번 사용할 수 있습니다.

`[pallet::storage]` 매크로는 StorageValue, StorageMap 또는 StorageDoubleMap의 타입 별칭을 사용하여 명명된 또는 무명의 제네릭을 사용하여 정의될 수 있습니다.

```rust
#[pallet::storage]
#[pallet::getter(fn $getter_name)] // 선택적
$vis type $StorageName<$some_generic> $optional_where_clause
	= $StorageType<$generic_name = $some_generics, $other_name = $some_other, ...>;
```

자세한 내용은 [pallet::storage](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#storage-palletstorage-optional)를 위한 Rust 문서와 다음 스토리지 데이터 구조를 참조하십시오:

- [StorageDoubleMap](https://paritytech.github.io/substrate/master/frame_support/pallet_prelude/struct.StorageDoubleMap.html)
- [StorageMap](https://paritytech.github.io/substrate/master/frame_support/pallet_prelude/struct.StorageMap.html#implementations)
- [StorageValue](https://paritytech.github.io/substrate/master/frame_support/pallet_prelude/struct.StorageValue.html)

### #[pallet::type_value]

`#[pallet::type_value]` 매크로를 사용하면 스토리지 타입에 대한 `Get` 특성을 구현하는 구조체를 정의할 수 있습니다. 이 속성 매크로는 `#[pallet::storage]` 매크로와 함께 사용하여 스토리지의 기본 값을 정의할 수 있습니다.

```rust
#[pallet::type_value]
fn MyDefault<T: Config>() -> T::Balance { 3.into() }
```

이 매크로를 사용하는 방법에 대한 자세한 정보는 [pallet::type_value](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#type-value-pallettype_value-optional)를 위한 Rust 문서를 참조하십시오.

### #[pallet::genesis_build]

[`#[pallet::genesis_build]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#genesis-build-palletgenesis_build-optional) 매크로를 사용하면 제네시스 구성이 어떻게 구축되는지 정의할 수 있습니다.

이 매크로는 제네시스 구성을 빌드하는 데 사용되는 제네릭 타입 <T: Config>에 대한 GenesisBuild<T> 특성의 트레이트 구현으로 정의됩니다.

예를 들어:

```rust
#[pallet::genesis_build]
impl<T: Config> GenesisBuild<T> for GenesisConfig {
	fn build(&self) {}
}
```

자세한 내용은 [pallet::genesis_build](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#genesis-build-palletgenesis_build-optional)을 위한 Rust 문서를 참조하십시오.

### #[pallet::genesis_config]

[`#[pallet::genesis_config]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#genesis-config-palletgenesis_config-optional) 매크로를 사용하면 팔렛의 제네시스 구성을 정의할 수 있습니다.

이 매크로는 열거형 또는 구조체로 정의될 수 있지만 공개되어야 하며, `#[pallet::genesis_build]` 매크로와 함께 GenesisBuild 특성을 구현해야 합니다.

예를 들어:

```rust
#[pallet::genesis_config]
pub struct GenesisConfig<T: Config> {
	_myfield: BalanceOf<T>,
}
```

자세한 내용은 [pallet::genesis_config](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#genesis-config-palletgenesis_config-optional)을 위한 Rust 문서를 참조하십시오.

### #[pallet::inherent]

[`#[pallet::inherent]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#inherent-palletinherent-optional) 매크로를 사용하면 팔렛이 서명되지 않은 인허런트 트랜잭션에서 데이터를 제공할 수 있습니다.

이 매크로는 제네릭 타입 <T: Config>에 대한 trait ProvideInherent를 타입 Pallet<T>에 대한 trait 구현으로 정의됩니다.

예를 들어:

```rust
#[pallet::inherent]
impl<T: Config> ProvideInherent for Pallet<T> {
	// ... 일반적인 trait 구현
}
```

자세한 내용은 [pallet::inherent](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#inherent-palletinherent-optional)을 위한 Rust 문서를 참조하십시오.

### #[pallet::origin]

[`#[pallet::origin]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#origin-palletorigin-optional) 매크로를 사용하면 팔렛에 대한 origin을 정의할 수 있습니다.

이 매크로는 타입 별칭, 열거형 또는 구조체로 정의해야 하며, 공개되어야 합니다.

예를 들어:

```rust
#[pallet::origin]
pub struct Origin<T>(PhantomData<(T)>);
```

자세한 내용은 [pallet::origin](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#origin-palletorigin-optional)을 위한 Rust 문서를 참조하십시오.

### #[pallet::validate_unsigned]

[`#[pallet::validate_unsigned]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#validate-unsigned-palletvalidate_unsigned-optional) 매크로를 사용하면 팔렛에서 서명되지 않은 트랜잭션을 검증할 수 있습니다.

이 매크로는 제네릭 타입 <T: Config>에 대한 trait ValidateUnsigned를 타입 Pallet<T>에 대한 trait 구현으로 정의합니다.

예를 들어:

```rust
#[pallet::validate_unsigned]
impl<T: Config> ValidateUnsigned for Pallet<T> {
	// ... 일반적인 trait 구현
}
```

자세한 내용은 [pallet::validate_unsigned](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#validate-unsigned-palletvalidate_unsigned-optional)을 위한 Rust 문서를 참조하십시오.

## 런타임 구축 매크로

매크로 사용에 대한 소개로 [Substrate 런타임 매크로](#substrate-runtime-m

`construct_runtime!` 매크로에서 팔렛들이 나열된 순서는 중요하다는 점을 유념해야 합니다.
기본적으로, 첫 번째 팔렛은 0부터 시작하여 각 팔렛마다 증가하는 팔렛 인덱스를 가집니다.
`construct_runtime!` 매크로에서 팔렛의 인덱싱을 수동으로 조정할 수도 있습니다. 이를 위해 팔렛에 대한 인덱스 번호를 추가하면 됩니다.
예를 들어, 다음과 같은 구문으로 `MyCustomPallet`에 대한 모든 팔렛 속성을 생성하고 인덱스를 8로 설정할 수 있습니다:

```rust
MyCustomPallet: pallet_my_custom_pallet use_parts = 8,
```

하지만, `construct_runtime!` 매크로에서 팔렛을 정의하는 순서는 제네시스 스토리지 구성에도 영향을 미친다는 점도 유념해야 합니다.
한 팔렛이 다른 팔렛에 의존하는 경우, 의존하는 팔렛이 의존하는 팔렛보다 먼저 나와야 한다는 것을 확인해야 합니다. 즉, 의존하는 팔렛이 나열되거나 인덱스 값이 더 낮아야 합니다.

더 자세한 정보는 [construct_runtime](https://paritytech.github.io/substrate/master/frame_support/macro.construct_runtime.html)에 대한 러스트 문서를 참조하세요.

### parameter_types!

`parameter_types!` 매크로는 런타임 구성 동안 각 팔렛에 할당되는 매개변수 유형을 선언합니다.

이 매크로는 지정된 각 매개변수를 `get()` 함수를 반환하는 지정된 유형으로 변환하는 구조체 유형으로 변환합니다.
각 매개변수 구조체 유형은 또한 `frame_support::traits::Get<I>` 트레이트를 구현하여 유형을 지정된 값으로 변환합니다.

더 자세한 정보는 [parameter_types](https://paritytech.github.io/substrate/master/frame_support/macro.parameter_types.html)에 대한 러스트 문서를 참조하세요.

### impl_runtime_apis!

`impl_runtime_apis!` 매크로는 매크로로 구현된 모든 트레이트에 대한 런타임 API를 생성합니다.
이 매크로에서 구현된 트레이트는 먼저 `decl_runtime_apis` 매크로에서 선언되어야 합니다.
매크로는 이러한 트레이트를 [런타임 API](/reference/runtime-apis/)로 노출하기 위해 `RuntimeApi` 및 `RuntimeApiImpl` 구조체를 생성합니다.
매크로로 노출된 트레이트는 외부 노드 구성 요소가 `RuntimeApi` 유형을 통해 런타임과 통신할 수 있도록 합니다.

매크로는 `RuntimeApi` 및 `RuntimeApiImpl` 구조체를 선언하고 `RuntimeApiImpl` 구조체에 대해 다양한 도우미 트레이트를 구현합니다.
`impl_runtime_apis!` 매크로에서 런타임이 노출할 추가적인 인터페이스를 정의하는 경우, 해당 인터페이스는 기본 `RuntimeApiImpl` 구현에 추가됩니다.

매크로는 또한 모든 구현된 `api` 트레이트에 대한 버전 정보를 노출하는 `RUNTIME_API_VERSIONS` 상수를 생성합니다.
이 상수는 [`RuntimeVersion`](https://paritytech.github.io/substrate/master/sp_version/struct.RuntimeVersion.html)의 `apis` 필드를 인스턴스화하는 데 사용됩니다.

더 자세한 정보는 [impl_runtime_apis](https://paritytech.github.io/substrate/master/sp_api/macro.impl_runtime_apis.html)에 대한 러스트 문서를 참조하세요.

### app_crypto!

`app_crypto!` 매크로는 지정된 서명 알고리즘을 사용하여 응용 프로그램별 암호화 키 쌍을 생성합니다.

매크로는 다음과 같은 구조체 유형을 선언합니다:

- `Public`

  `Public` 유형에 대해 매크로는 `sp_application_crypto::AppKey` 트레이트를 구현하여 공개 키 유형을 정의하고, 키 쌍을 생성하고, 트랜잭션에 서명하고, 서명을 검증할 수 있도록 합니다.

- `Signature`

  `Signature` 유형에 대해 매크로는 `core::hash::Hash` 트레이트를 구현하여 서명을 해싱하는 데 사용되는 서명 알고리즘(예: SR25519 또는 ED25519)을 지정합니다.

- `Pair`

  `Pair` 유형에 대해 매크로는 `sp_application_crypto::Pair` 및 `sp_application_crypto::AppKey` 트레이트를 구현하여 비밀 문구나 시드로부터 공개-개인 키 쌍을 생성합니다.

이러한 구조체에 대한 트레이트뿐만 아니라 매크로는 도우미 트레이트도 구현합니다.

더 자세한 정보는 [app_crypto](https://paritytech.github.io/substrate/master/sp_application_crypto/macro.app_crypto.html)에 대한 러스트 문서를 참조하세요.

## 벤치마킹 매크로

FRAME 벤치마킹 프레임워크는 팔렛을 벤치마킹하기 위해 여러 매크로를 정의합니다.
다음 매크로들은 벤치마킹에 사용됩니다:

- `add_benchmark`: 팔렛 벤치마크를 `Vec<BenchmarkBatch>` 객체에 추가하는 매크로로, 팔렛 크레이트 이름과 생성된 모듈 구조체를 사용합니다.
- `benchmarks`: 함수 호출의 실행 시간을 테스트하기 위한 벤치마크 로직을 구성하는 매크로입니다.
- `benchmarks_instance`: 인스턴스화 가능한 모듈에 대해 `benchmarks` 매크로와 동일한 기능을 제공합니다.
- `benchmarks_instance_pallet`: [`frame_support::pallet`] 매크로로 선언된 인스턴스화 가능한 팔렛에 대해 `benchmarks` 매크로와 동일한 기능을 제공합니다.
- `cb_add_benchmarks`: `define_benchmarks` 매크로의 콜백으로 `add_benchmark`를 호출하는 매크로입니다.
- `cb_list_benchmarks`: `define_benchmarks` 매크로의 콜백으로 `list_benchmark`를 호출하는 매크로입니다.

- `define_benchmarks`: 런타임에 대해 벤치마크를 수행할 모든 팔렛을 정의하는 매크로입니다.

- `impl_benchmark_test_suite`: 벤치마킹 모듈에서 정의된 벤치마크를 실행하는 테스트 스위트를 생성하는 매크로입니다.

- `list_benchmark`: 런타임에서 구성된 팔렛에 대한 벤치마크 목록을 생성하는 매크로입니다.

- `whitelist`: 테스트 목적으로 계정을 허용 목록에 추가하는 매크로입니다.

## 참고 자료

- [The Rust Programming Language](https://doc.rust-lang.org/book/ch19-06-macros.html)
- [The Little Book of Rust Macros](https://danielkeep.github.io/tlborm/book)