---
title: 사용자 정의 팔레트
description:
keywords:
---

사용자 정의 런타임을 구축하는 가장 일반적인 방법은 [기존 팔레트](/reference/frame-pallets/)를 기반으로 시작하는 것입니다.
예를 들어, 기존의 Collective와 Balances 팔레트에서 노출된 유형을 사용하지만, 응용 프로그램 및 스테이킹 규칙에 필요한 사용자 정의 런타임 로직을 포함하는 응용 프로그램별 스테이킹 팔레트를 구축할 수 있습니다.

[FRAME 팔레트](/reference/frame-pallets)는 가장 일반적인 팔레트에 대한 개요를 제공하지만, 현재 존재하는 팔레트에 대한 최신 정보를 찾는 가장 좋은 장소는 `pallet_*` 네이밍 규칙을 사용하는 크레이트의 [Rust API](/reference/rust-api/) 문서입니다.

필요한 팔레트를 찾지 못한 경우, FRAME 매크로를 사용하여 사용자 정의 팔레트의 뼈대를 구축할 수 있습니다.

## 팔레트 매크로와 속성

FRAME은 복잡한 코드 블록을 캡슐화하기 위해 Rust 매크로를 광범위하게 사용합니다.
사용자 정의 팔레트를 구축하는 데 가장 중요한 매크로는 [`pallet`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html) 매크로입니다.
`pallet` 매크로는 팔레트가 제공해야 하는 핵심 속성 집합을 정의합니다.
예를 들어:

- `#[pallet::pallet]`은 팔레트에 데이터를 저장할 수 있는 구조체(struct)를 정의할 수 있도록 하는 필수 팔레트 속성입니다.
- `#[pallet::config]`는 팔레트의 구성 트레이트를 정의할 수 있도록 하는 필수 팔레트 속성입니다.

`pallet` 매크로는 또한 팔레트가 일반적으로 제공하는 핵심 속성 집합을 정의합니다.
예를 들어:

- `#[pallet::call]`은 팔레트에 대한 호출 가능한 함수 호출을 구현할 수 있도록 하는 속성입니다.
- `#[pallet::error]`는 호출 가능한 오류를 생성할 수 있도록 하는 속성입니다.
- `#[pallet::event]`는 호출 가능한 이벤트를 생성할 수 있도록 하는 속성입니다.
- `#[pallet::storage]`는 런타임에 저장소 인스턴스와 해당 메타데이터를 생성할 수 있도록 하는 속성입니다.

이러한 핵심 속성은 사용자 정의 팔레트를 작성할 때 필요한 결정과 일치합니다.
예를 들어, 팔레트가 저장하는 데이터와 해당 데이터가 체인 상에 저장되는지 오프 체인에 저장되는지를 고려해야 합니다.
또한 팔레트가 노출하는 호출 가능한 함수와 해당 함수 호출이 원자적으로 저장소를 수정하는지 여부, 팔레트가 런타임 후크에 호출을 수행하는지 여부 등을 고려해야 합니다.

매크로는 사용자 정의 런타임 로직을 구현하기 위해 작성해야 하는 코드를 단순화합니다.
그러나 일부 매크로는 함수 선언에 특정 요구 사항을 강제합니다.
예를 들어, `Config` 트레이트는 `frame_system::Config`에 바인딩되어야 하며, `#[pallet::pallet]` 구조체는 `pub struct Pallet<T>(_);`로 선언되어야 합니다.
FRAME 팔레트에서 사용되는 매크로에 대한 개요는 [FRAME 매크로](/reference/frame-macros/)를 참조하십시오.

<!-- ## 유용한 FRAME 트레이트

- Pallet Origin
- Origins: EnsureOrigin, EnsureOneOf
  ...

## 런타임 구현

팔레트를 작성하고 런타임에 구현하는 것은 서로 밀접하게 관련되어 있습니다.
팔레트의 `Config` 트레이트는 `Runtime`에 구현되며, 이는 `construct_runtime` 매크로에서 구현된 모든 팔레트를 컴파일하는 데 사용되는 특수한 구조체입니다.

- [`parameter_types`](https://paritytech.github.io/substrate/master/frame_support/macro.parameter_types.html) 및 [`ord_parameter_types`](https://paritytech.github.io/substrate/master/frame_support/macro.ord_parameter_types.html) 매크로는 구성 가능한 팔레트 상수에 값을 전달하는 데 유용합니다.
- [기타 고려 사항, 예를 들어 no_std]
- 최소한의 런타임 참조
- 사이드 체인 아키텍처 참조
- API 엔드포인트: on_initialize, off_chain workers ?

기본 및 중급 사용 설명서로 연결되는 내용을 작성하십시오. -->