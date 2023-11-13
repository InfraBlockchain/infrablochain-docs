---
title: Substrate를 위한 Rust
description: Substrate 블록체인을 개발할 때 특히 중요한 Rust 프로그래밍 규칙을 강조합니다.
keywords:
---

Substrate가 미션 크리티컬 소프트웨어를 만들기 위한 유연하고 확장 가능한 프레임워크로 인정받는 것은 [Rust](https://www.rust-lang.org/)에 기인합니다.
Substrate의 선택 언어인 Rust는 고성능 프로그래밍 언어로, 다음과 같은 이유로 첫 번째 선택지입니다:

- Rust는 빠릅니다: 컴파일 시 정적으로 타입이 지정되어 있어 컴파일러가 코드를 최적화하고 개발자가 특정 컴파일 타겟에 최적화할 수 있습니다.

- Rust는 이식성이 뛰어납니다: 임베디드 장치에서 실행되도록 설계되었으며 모든 종류의 운영 체제를 지원합니다.

- Rust는 메모리 안전성이 높습니다: 가비지 컬렉터가 없으며 사용하는 모든 변수와 참조하는 모든 메모리 주소를 확인하여 메모리 누수를 방지합니다.

- Rust는 Wasm 우선입니다: WebAssembly로 컴파일하는 데 대한 일급 지원을 제공합니다.

## Substrate에서의 Rust

[아키텍처]() 섹션에서 알 수 있듯이 Substrate는 두 가지 구성 요소로 이루어져 있습니다: 외부 노드와 런타임.
Rust의 더 복잡한 기능인 멀티스레딩과 비동기 Rust는 외부 노드 코드에서 사용되지만 런타임 엔지니어에게 직접 노출되지 않으므로 런타임 엔지니어는 노드의 비즈니스 로직에 집중하기 쉽습니다.

일반적으로 개발자는 다음과 같은 내용을 알아두면 좋습니다:

- 기본적인 [Rust 관용구](https://rust-unofficial.github.io/patterns/idioms/index.html), [`no_std`로 작업하기](https://docs.rust-embedded.org/book/intro/no-std.html) 및 어떤 매크로가 사용되고 왜 사용되는지(runtime 엔지니어링을 위해).
- [비동기 Rust](https://rust-lang.github.io/async-book/01_getting_started/01_chapter.html) (외부 노드(클라이언트) 코드와 작업하는 고급 개발자를 위해).

Substrate로 들어가기 전에 Rust에 대한 일반적인 이해가 필수적이며, Rust를 배우기 위한 많은 자료가 있습니다. 예를 들어, [Rust 언어 프로그래밍 책](https://doc.rust-lang.org/book/)과 [Rust by Example](https://doc.rust-lang.org/rust-by-example/) 등이 있습니다. 이 섹션의 나머지 부분에서는 Substrate가 런타임 엔지니어링을 시작하는 개발자들을 위해 Rust의 핵심 기능을 어떻게 활용하는지 강조합니다.

## 컴파일 타겟

Substrate 노드를 빌드할 때는 `wasm32-unknown-unknown` 컴파일 타겟을 사용합니다. 이는 Substrate 런타임 엔지니어가 Wasm으로 컴파일해야 하는 런타임을 작성해야 함을 의미합니다.
이는 일반적인 표준 라이브러리 유형과 함수에 의존할 수 없으며 런타임 코드의 대부분에는 `no_std` 호환 크레이트만 사용해야 한다는 것을 의미합니다.
Substrate에는 `no_std` 요구 사항을 해결할 수 있는 많은 기본 유형과 관련된 특성이 있습니다.

## 매크로

FRAME 팔레트를 사용하고 작성하는 방법을 배우면 공통 작업을 추상화하거나 런타임별 요구 사항을 강제하는 재사용 가능한 코드로 많은 매크로가 제공됨을 알 수 있습니다.
이러한 매크로를 사용하면 일반적인 코드 대신 관용적인 Rust와 응용 프로그램별 로직에 집중할 수 있습니다.

Rust 매크로는 특정 요구 사항을 충족시키기 위해 (코드를 다시 작성하지 않고) 특정 방식으로 서식을 지정하거나 특정 검사를 수행하거나 특정 데이터 구조로 이루어진 로직을 제공하는 데 유용합니다.
이는 Substrate 런타임의 복잡성과 통합할 수 있는 코드를 개발자가 작성하는 데 특히 유용합니다.
예를 들어, `#[frame_system::pallet]` 매크로는 모든 FRAME 팔레트에서 필요하며, 이를 통해 스토리지 항목이나 외부에서 호출 가능한 함수와 같은 특정 필수 속성을 올바르게 구현할 수 있으며 `construct_runtime`의 빌드 프로세스와 호환됩니다.

Substrate 런타임 개발은 Rust의 속성 매크로를 적극적으로 활용하며, 이는 파생 속성과 사용자 정의 속성 두 가지 유형으로 나뉩니다.
Substrate를 시작할 때 정확히 어떻게 작동하는지 알 필요는 없지만, 존재한다는 것과 올바른 런타임 코드를 작성할 수 있도록 도와준다는 것을 알고 있어야 합니다.

파생 속성은 런타임 실행 중 노드에서 디코딩할 수 있는 사용자 정의 런타임 유형에 유용합니다.

다른 속성 매크로도 Substrate의 코드베이스 전반에 일반적으로 사용됩니다:

- 코드 스니펫이 `no_std`로만 컴파일되는지 또는 `std` 라이브러리를 사용할 수 있는지 지정합니다.
- 사용자 정의 FRAME 팔레트를 빌드합니다.
- 런타임 빌드 방식을 지정합니다.

## 제네릭과 구성 특성

Rust에서 Java와 같은 언어의 인터페이스와 비교되는 특성은 타입에 고급 기능을 제공하는 방법을 제공합니다.

팔레트에 대해 읽어보셨다면 모든 팔레트에는 팔레트가 의존하는 유형과 인터페이스를 정의할 수 있는 `Config` 특성이 있다는 것을 알아채셨을 것입니다.

이 특성 자체는 `frame_system::pallet::Config` 특성에서 일부 핵심 런타임 유형을 상속받아 런타임 로직 작성 시 공통 유형에 쉽게 액세스할 수 있도록 합니다.
또한, 모든 FRAME 팔레트에서 `Config` 특성은 `T`에 대해 제네릭하게 구현됩니다(제네릭에 대한 자세한 내용은 다음 섹션에서 다룹니다).
이러한 핵심 런타임 유형의 일반적인 예로는 런타임에서 사용자 계정을 식별하는 데 사용되는 `T::AccountId`와 런타임에서 사용되는 블록 번호 유형인 `T::BlockNumber` 등이 있습니다.

Rust에서 제네릭 유형과 특성에 대한 자세한 내용은 Rust 책의 [제네릭 유형](https://doc.rust-lang.org/book/ch10-01-syntax.html), [특성](https://doc.rust-lang.org/book/ch10-02-traits.html) 및 [고급 특성](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html) 섹션을 참조하십시오.

Rust의 제네릭을 사용하면 Substrate 런타임 개발자는 구체적인 구현 세부 사항에 대해 완전히 알지 못하는 팔레트를 작성할 수 있으며, 이로 인해 Substrate의 유연성, 확장성 및 모듈성을 최대한 활용할 수 있습니다.

`Config` 특성의 모든 유형은 특성 바운드를 사용하여 제네릭하게 정의되고 런타임 구현에서 구체화됩니다.
이는 동일한 유형에 대해 다른 사양을 지원하는 팔레트를 작성할 수 있을 뿐만 아니라 최소한의 오버헤드로 제네릭 구현을 사용자 정의할 수도 있음을 의미합니다(예: 블록 번호를 `u32`로 변경).

이를 통해 개발자는 핵심 블록체인 아키텍처 결정에 대한 가정을 하지 않고 코드를 작성할 수 있는 유연성을 얻을 수 있습니다.

Substrate는 최대한의 유연성을 제공하기 위해 제네릭 유형을 최대한 활용합니다.
제네릭 유형이 어떻게 해결되는지는 목적에 맞게 정의합니다.

Rust에서 제네릭 유형과 특성에 대한 자세한 내용은 Rust 책의 [제네릭 유형](https://doc.rust-lang.org/book/ch10-01-syntax.html) 섹션을 참조하십시오.

## 다음으로 어디로 가야 할까요

이제 Substrate가 특성, 제네릭 유형 및 매크로와 같은 몇 가지 핵심 Rust 기능에 의존한다는 것을 알았으므로, 더 자세히 알아보기 위해 다음 자료를 살펴볼 수 있습니다.

- [Rust 책](https://doc.rust-lang.org/book/)
- [왜 Rust인가?](https://www.parity.io/blog/why-rust) (Parity의 블로그)
- [Cargo와 crates.io](https://doc.rust-lang.org/book/ch14-00-more-about-cargo.html)
- [스마트 컨트랙트를 위한 Rust는 왜?](https://paritytech.github.io/ink-docs/why-rust-for-smart-contracts) (ink! 문서)