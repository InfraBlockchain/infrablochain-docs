---
title: 런타임 개발
description: Substrate 블록체인의 런타임 개발에 필수적인 핵심 프로그래밍 인터페이스, 기본 요소 및 모듈을 소개합니다.
keywords:
---

[아키텍처](/learn/architecture/)에서 설명한 대로 Substrate 노드의 **런타임**은 트랜잭션 실행, 상태 전이 저장 및 외부 노드와의 상호작용을 위한 모든 비즈니스 로직을 포함합니다.
Substrate는 일반적인 블록체인 구성 요소를 구축하는 데 필요한 모든 도구를 제공하여 블록체인 동작을 정의하는 런타임 로직에 집중할 수 있도록 합니다.

## 상태 전이 및 런타임

가장 기본적인 수준에서 모든 블록체인은 사실상 체인 상에서 발생하는 각 변경 사항의 원장 또는 기록입니다.
Substrate 기반 체인에서는 이러한 상태 변경 사항이 런타임에 기록됩니다.
상태 전이가 런타임에서 처리되기 때문에 런타임은 때때로 [상태 전이 함수](/reference/glossary#state-transition-function-stf)를 제공한다고 설명됩니다.

상태 전이가 런타임에서 발생하기 때문에 런타임에서는 블록체인 [상태](/reference/glossary#state)를 나타내는 **저장소 항목**과 이 상태를 변경할 수 있도록 하는 [트랜잭션](/learn/transaction-types)을 정의합니다.

![런타임의 상태 및 함수](/media/images/docs/state-transition-function.png)

Substrate 런타임은 어떤 트랜잭션이 유효하고 유효하지 않은지, 그리고 트랜잭션에 대한 응답으로 체인 상태가 어떻게 변경되는지를 결정합니다.

## 런타임 인터페이스

[아키텍처](/learn/architecture/)에서 배운 대로 외부 노드는 피어 검색, 트랜잭션 풀링, 블록 및 트랜잭션 공유, 합의, 외부 세계로부터의 RPC 호출에 대한 응답을 처리하는 역할을 담당합니다.
이러한 작업은 종종 외부 노드가 런타임에 정보를 쿼리하거나 런타임에 정보를 제공해야 함을 의미합니다.
런타임 API는 외부 노드와 런타임 간의 이러한 유형의 통신을 용이하게 합니다.

Substrate에서 `sp_api` 크레이트는 런타임 API를 구현하기 위한 인터페이스를 제공합니다.
이는 [`impl_runtime_apis`](https://paritytech.github.io/substrate/master/sp_api/macro.impl_runtime_apis.html) 매크로를 사용하여 사용자 정의 인터페이스를 정의하는 데 유연성을 제공하도록 설계되었습니다.
그러나 모든 런타임은 [`Core`](https://paritytech.github.io/substrate/master/sp_api/trait.Core.html) 및 [`Metadata`](https://paritytech.github.io/substrate/master/sp_api/trait.Metadata.html) 인터페이스를 구현해야 합니다.
이러한 필수 인터페이스 외에도 노드 템플릿과 같은 대부분의 Substrate 노드는 다음과 같은 런타임 인터페이스를 구현합니다:

- [`BlockBuilder`](https://paritytech.github.io/substrate/master/sp_block_builder/trait.BlockBuilder.html): 블록을 구축하는 데 필요한 기능을 제공합니다.
- [`TaggedTransactionQueue`](https://paritytech.github.io/substrate/master/sp_transaction_pool/runtime_api/trait.TaggedTransactionQueue.html): 트랜잭션을 유효성 검사합니다.
- [`OffchainWorkerApi`](https://paritytech.github.io/substrate/master/sp_offchain/trait.OffchainWorkerApi.html): 오프체인 작업을 활성화합니다.
- [`AuraApi`](https://paritytech.github.io/substrate/master/sp_consensus_aura/trait.AuraApi.html): 순환형 합의 방법을 사용하여 블록 작성 및 유효성 검사를 수행합니다.
- [`SessionKeys`](https://paritytech.github.io/substrate/master/sp_session/trait.SessionKeys.html): 세션 키를 생성하고 디코딩합니다.
- [`GrandpaApi`](https://paritytech.github.io/substrate/master/sp_consensus_grandpa/trait.GrandpaApi.html): 블록 최종화를 런타임으로 수행합니다.
- [`AccountNonceApi`](https://paritytech.github.io/substrate/master/frame_system_rpc_runtime_api/trait.AccountNonceApi.html): 트랜잭션 인덱스를 쿼리합니다.
- [`TransactionPaymentApi`](https://paritytech.github.io/substrate/master/pallet_transaction_payment_rpc_runtime_api/trait.TransactionPaymentApi.html): 트랜잭션에 대한 정보를 쿼리합니다.
- [`Benchmark`](https://paritytech.github.io/substrate/master/frame_benchmarking/trait.Benchmark.html): 트랜잭션 완료에 필요한 실행 시간을 추정하고 측정합니다.

## 핵심 기본 요소

Substrate는 런타임이 다른 Substrate 계층에 제공해야 하는 핵심 기본 요소를 정의합니다.
Substrate 프레임워크는 런타임이 다른 Substrate 계층에 제공해야 하는 내용에 대해 최소한의 가정을 합니다.
그러나 Substrate 프레임워크 내에서 작동하기 위해 특정 인터페이스를 정의하고 특정 인터페이스를 충족해야 하는 몇 가지 데이터 유형이 있습니다.

이러한 핵심 기본 요소는 다음과 같습니다:

- `Hash`: 어떤 데이터의 암호화 다이제스트를 인코딩하는 유형입니다. 일반적으로 256비트 수량입니다.

- `DigestItem`: "하드-와이어드" 대안 중 하나를 인코딩할 수 있는 유형입니다. 이는 합의 및 변경 추적과 관련된 것뿐만 아니라 런타임 내의 특정 모듈과 관련된 "소프트-코드" 변형 중 어떤 것이든 인코딩할 수 있어야 합니다.

- `Digest`: DigestItem의 연속입니다. 이는 블록 내에서 가지고 있어야 하는 경량 클라이언트에게 관련된 모든 정보를 인코딩합니다.

- `Extrinsic`: 블록체인에서 인식되는 블록체인 외부의 단일 데이터를 나타내는 유형입니다. 일반적으로 하나 이상의 서명과 일종의 인코딩된 명령(예: 자금 소유권 이전 또는 스마트 계약 호출)이 포함됩니다.

- `Header`: 블록에 관련된 모든 정보를 대표하는 유형입니다(암호화 또는 기타 방법으로). 부모 해시, 저장소 루트 및 엑스트린시스 트라이 루트, 다이제스트 및 블록 번호를 포함합니다.

- `Block`: 사실상 `Header`와 일련의 `Extrinsic`을 조합한 것으로, 사용할 해시 알고리즘의 사양도 포함합니다.

- `BlockNumber`: 유효한 블록이 가질 수 있는 조상의 총 수를 인코딩하는 유형입니다. 일반적으로 32비트 수량입니다.

## FRAME

런타임 개발자로서 사용할 수 있는 가장 강력한 도구 중 하나는 [FRAME](/reference/glossary/#frame)입니다.
[Substrate 개발자를 강력하게 하는](/)에서 언급한 대로 FRAME은 **Framework for Runtime Aggregation of Modularized Entities**의 약자로, 런타임 개발을 단순화하는 많은 모듈과 지원 라이브러리를 포함합니다.
Substrate에서 이러한 모듈인 **팔렛(pallets)**은 다양한 사용 사례와 런타임에 포함하려는 기능에 대한 사용자 정의 비즈니스 로직을 제공합니다.
예를 들어, 스테이킹, 합의, 거버넌스 및 기타 일반적인 활동을 위한 비즈니스 로직 프레임워크를 제공하는 팔렛이 있습니다.

사용 가능한 팔렛에 대한 요약은 [FRAME 팔렛](/reference/frame-pallets/)을 참조하십시오.

팔렛 외에도 FRAME은 다음 라이브러리와 모듈을 통해 런타임과 상호작용하기 위한 서비스를 제공합니다.

- [FRAME 시스템 크레이트 `frame_system`](https://paritytech.github.io/substrate/master/frame_system/index.html): 런타임을 위한 저수준 유형, 저장소 및 함수를 제공합니다.

- [FRAME 지원 크레이트 `frame_support`](https://paritytech.github.io/substrate/master/frame_support/index.html): Substrate 팔렛 개발을 단순화하는 러스트 매크로, 유형, 트레이트 및 모듈의 모음입니다.

- [FRAME 실행 팔렛 `frame_executive`](https://paritytech.github.io/substrate/master/frame_executive/index.html): 런타임 내의 해당 팔렛으로의 들어오는 함수 호출 실행을 조정합니다.

다음 다이어그램은 FRAME 및 해당 시스템, 지원 및 실행 모듈이 런타임 환경에 대한 서비스를 제공하는 방식을 보여줍니다.

![FRAME 및 런타임 아키텍처](/media/images/docs/runtime-and-frame.png)

### 팔렛으로 런타임 구성

FRAME을 사용하지 않고도 Substrate 기반 블록체인을 구축할 수 있습니다.
그러나 FRAME 팔렛을 사용하면 사전 정의된 구성 요소를 시작점으로 사용하여 사용자 정의 런타임 로직을 구성할 수 있습니다.
각 팔렛은 특정한 기능 또는 런타임의 기능을 구현하기 위한 특정한 유형, 저장소 항목 및 함수를 정의합니다.

다음 다이어그램은 FRAME 팔렛을 선택하고 조합하여 런타임을 구성하는 방법을 보여줍니다.

![FRAME으로 런타임 구성](/media/images/docs/compose-runtime.png)

### 사용자 정의 팔렛 구축

사전 구축된 FRAME 팔렛 라이브러리 외에도 FRAME 라이브러리와 서비스를 사용하여 사용자 정의 팔렛을 구축할 수 있습니다.
사용자 정의 팔렛을 사용하면 목적에 가장 적합한 런타임 동작을 정의할 수 있습니다.
각 팔렛은 고유한 로직을 가지고 있기 때문에 사전 구축된 팔렛과 사용자 정의 팔렛을 조합하여 블록체인이 제공하는 기능과 기능을 제어하고 원하는 결과를 얻을 수 있습니다.

예를 들어, 런타임에 계정 잔액이 변경될 때 사용자가 작성한 팔렛을 호출하는 사용자 정의 로직을 추가하면서 토큰 관리와 관련된 저장소 항목 및 함수를 사용하기 위해 [잔액 팔렛](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/balances)을 런타임에 포함시킬 수 있습니다.

대부분의 팔렛은 다음과 같은 몇 가지 섹션의 조합으로 구성됩니다:

- 가져오기 및 종속성
- 팔렛 유형 선언
- 런타임 구성 트레이트
- 런타임 저장소
- 런타임 이벤트
- 특정 컨텍스트에서 실행되어야 하는 로직을 정의하는 훅
- 트랜잭션을 실행하는 데 사용할 수 있는 함수 호출

예를 들어, 사용자 정의 팔렛을 정의하려면 다음과 유사한 팔렛의 뼈대 구조로 시작할 수 있습니다:

```rust
// 필요한 가져오기 및 종속성 추가
pub use pallet::*;

#[frame_support::pallet]
pub mod pallet {
 use frame_support::pallet_prelude::*;
 use frame_system::pallet_prelude::*;

 // 팔렛 유형 선언
 // 트레이트 및 메서드를 구현하기 위한 플레이스홀더입니다.
 #[pallet::pallet]
 #[pallet::generate_store(pub(super) trait Store)]
 pub struct Pallet<T>(_);

 // 런타임 구성 트레이트 추가
 // 모든 유형 및 상수는 여기에 포함됩니다.
 #[pallet::config]
 pub trait Config: frame_system::Config { ... }

 // 런타임 저장소 추가하여 저장소 항목을 선언합니다.
 #[pallet::storage]
 #[pallet::getter(fn something)]
 pub type MyStorage<T: Config> = StorageValue<_, u32>;

 // 런타임 이벤트 추가
 #[pallet::event]
 #[pallet::generate_deposit(pub(super) fn deposit_event)]
 pub enum Event<T: Config> { ... }

 // 특정 컨텍스트에서 실행되어야 하는 로직을 정의하는 훅 추가
 #[pallet::hooks]
 impl<T: Config> Hooks<BlockNumberFor<T>> for Pallet<T> { ... }

 // 런타임 외부에서 호출할 수 있는 함수 추가
 #[pallet::call]
 impl<T:Config> Pallet<T> { ... }
}
```

필요에 따라 팔렛을 일부 또는 모든 섹션으로 조합할 수 있습니다.
사용자 정의 런타임을 설계하고 구축하는 과정에서 구성 트레이트, 저장소 항목, 이벤트 및 오류를 정의하는 데 사용되는 FRAME 라이브러리 및 런타임 기본 요소에 대해 자세히 알아가며, 실행을 위해 런타임으로 디스패치되는 함수 호출을 작성하는 방법을 배우게 될 것입니다.

다음 단계로 넘어가기

Substrate 런타임 개발의 기본 사항과 팔렛을 사용하는 방법에 대해 알게 되었으므로, 다음 주제 및 튜토리얼을 탐색하여 더 많은 내용을 학습하세요.

- [FRAME 팔렛](/reference/frame-pallets/)
- [런타임에 모듈 추가](/tutorials/build-application-logic/add-a-pallet)
- [Substrate용 Rust](/learn/rust-basics/)
- [매크로 참조](/reference/frame-macros/)
- [사용자 정의 팔렛에서 매크로 사용](/tutorials/build-application-logic/use-macros-in-a-custom-pallet/)