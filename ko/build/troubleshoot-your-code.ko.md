---
title: 코드 문제 해결 방법
description: 문제 해결 및 모범 사례를 따르기 위한 일반적인 코딩 기술 및 Substrate 특정 기술을 강조합니다.
keywords:
  - 코딩 기술
  - 소프트웨어 개발 원칙
  - 가독성
  - 디버깅
  - 보안 문제
---

Substrate와 FRAME은 블록체인 애플리케이션을 구축하기 위한 유연하고 모듈화된 프레임워크를 제공하기 때문에, 오류를 방지하고 코드를 디버깅하기 쉽게 만들기 위해 일반적인 모범 사례와 기본 코딩 원칙을 따르는 것이 중요합니다.

## 일반적인 코딩 기술

다음의 일반적인 원칙들은 Substrate나 FRAME에만 해당되는 것은 아니지만, 복잡한 소프트웨어를 구축할 때 특히 중요합니다. 이러한 소프트웨어는 보안 요구 사항과 제한된 리소스와 같은 제약 사항을 가지고 있습니다.

- [서식과 가독성](https://code.tutsplus.com/tutorials/top-15-best-practices-for-writing-super-readable-code--net-8118). 일관된 서식을 사용하고 가독성 좋은 코드를 작성하기 위한 모범 사례를 따르세요. 이렇게 하면 프로그램을 이해하고 유지보수하기 쉽게 만들 수 있습니다.
- [주석](https://stackoverflow.blog/2021/12/23/best-practices-for-writing-code-comments/). 코드에 명확하고 간결한 주석을 추가하여 코드가 무엇을 하는지 설명하고, 적용 가능한 경우 코드가 작성된 이유를 설명하세요.
- [스타일과 명명 규칙](https://doc.rust-lang.org/1.0.0/style/style/naming/README.html). Rust 스타일 가이드와 명명 규칙을 따르세요. 이렇게 하면 코드가 다른 Rust 프로그램과 일관성을 가지며, 다른 Rust 프로그래머가 코드를 읽고 디버깅하기 쉬워집니다.
- [라이선스](https://opensource.guide/legal/#which-open-source-license-is-appropriate-for-my-project). 리포지토리에 적절한 오픈 소스 라이선스와, 작성하지 않은 코드에 필요한 라이선스, 저작권 공지 및 어트리뷰션을 포함시키세요. 대부분의 경우, 작성하지 않은 코드를 사용하는 경우에는 원래의 라이선스를 유지하고 작성자를 언급해야 합니다.
- [리팩토링](https://en.wikipedia.org/wiki/Code_refactoring). 코드의 디자인, 구조 또는 구현을 개선하여 더 간단하고 깨끗하며 성능이 우수하며 표현력이 뛰어난 프로그램을 만드세요. 일반적으로 리팩토링은 코드의 기능을 변경하지 않고 코드 로직을 단순화시키며, 읽기 쉽고 유지보수 가능하며 확장 가능한 코드로 변환시킵니다.
- [Don't repeat yourself (DRY)](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself). 소프트웨어 개발의 DRY 원칙을 따르고, 데이터 추상화 또는 데이터 정규화를 사용하여 중복을 피하세요.
- [테스트](https://en.wikipedia.org/wiki/Unit_testing). 모든 개별 소프트웨어 구성 요소가 의도한 대로 작동하는지 확인하기 위해 단위 테스트를 작성하고 실행하세요. 런타임에서 테스트 모듈에 대한 자세한 정보는 [Test](/test/)를 참조하세요.
- [오류와 경고](https://rustc-dev-guide.rust-lang.org/diagnostics.html). 컴파일러에서 보고된 모든 오류와 경고를 해결하여 오류 또는 경고의 원인과 해결 방법을 이해하세요.
- [의존성](https://developerexperience.io/articles/updating-the-dependencies). 정기적으로 의존성을 업데이트하여 코드가 새로운 릴리스와 너무 뒤떨어지지 않고 최신 상태를 유지할 수 있도록 하세요. 주기적으로 Rust 컴파일러와 툴체인을 업데이트하고, InfraBlockspace 의 새로운 릴리스가 있는지 확인하세요.
- [하드 코딩](https://en.wikipedia.org/wiki/Hard_coding). 데이터를 소스 코드에 직접 포함하지 않도록 하세요.

## 일반적인 Substrate 문제점

Substrate에서는 올바르게 처리되지 않으면 오류를 발생시키거나 성능 문제를 일으킬 수 있는 몇 가지 일반적인 측면이 있습니다. 체인의 로직을 작성하는 동안 다음의 잠재적인 문제점에 특히 주의해야 합니다:

- [벤치마크](#벤치마크)
- [팔레트 결합](#팔레트-결합)
- [오프체인 워커](#오프체인-워커)
- [스토리지](#스토리지)
- [이벤트](#이벤트)

### 벤치마크

Substrate 벤치마킹 시스템은 팔레트 내 함수에 할당할 적절한 weight를 결정하는 데 도움을 줍니다. 적절한 weight를 설정하는 것은 블록체인의 신뢰성과 보안을 보장하기 위한 중요한 단계입니다. 개발 초기 단계에서는 벤치마킹 및 트랜잭션에 weight를 할당하는 것을 건너뛸 수 있지만, weight를 0으로 설정하면 코드가 공격에 취약해집니다. 함수 실행에 트랜잭션 수수료가 없는 경우, 악의적인 사용자는 함수를 반복해서 호출하여 네트워크를 거부하는 서비스 거부(DoS) 공격을 수행할 수 있습니다.

일반적으로 런타임에서 실행될 수 있는 모든 함수에는 weight가 정의되어 있어야 하며, 호출 계정에서 해당 수수료를 차감해야 합니다. 트랜잭션 수수료는 보통 서비스 거부(DoS) 공격을 방지하고 체인에 지속 가능한 경제 모델을 만들기 위한 중요한 경제적 인센티브입니다.

벤치마킹 시스템에 대한 자세한 내용은 [벤치마크](/test/benchmark/)를 참조하세요. 벤치마크를 작성하고 실행하는 간단한 예제는 [벤치마크 추가](/reference/how-to-guides/weights/add-benchmarks/)를 참조하세요.

### 팔레트 결합

Substrate에서는 한 팔레트가 다른 팔레트의 함수를 호출하는 두 가지 방법이 있습니다. 팔레트 결합은 한 팔레트가 다른 팔레트의 함수를 호출하는 방식에 대한 것입니다.

- **타이트 팔레트 결합**은 더 제한적이며, 한 팔레트가 다른 팔레트의 모든 또는 상당수의 타입과 메서드에 의존할 때 주로 사용됩니다.
- **느슨한 팔레트 결합**은 더 유연하며, 한 팔레트가 다른 팔레트가 노출하는 특정 트레이트나 함수 인터페이스에 의존할 때 주로 사용됩니다.

타이트 팔레트 결합은 두 팔레트가 런타임에 설치되어야 하며, 팔레트를 독립적으로 사용할 수 없습니다. 또한, 타이트하게 결합된 팔레트는 한 팔레트에서 변경이 발생하면 다른 팔레트에서도 변경이 필요하기 때문에 유지보수하기 어려울 수 있습니다. 대부분의 경우, 느슨한 결합은 팔레트를 런타임에 포함시키지 않고도 다른 팔레트에서 타입과 인터페이스를 재사용할 수 있기 때문에 더 유연한 솔루션입니다.

타이트 팔레트 결합과 느슨한 팔레트 결합에 대한 자세한 내용은 [팔레트 결합](/build/pallet-coupling/)과 이 [코드 예제](https://substrate.stackexchange.com/questions/922/pallet-loose-couplingtight-coupling-and-missing-traits?rq=1)를 참조하세요. 팔레트 결합의 간단한 예제는 [타이트 팔레트 결합 사용](/reference/how-to-guides/pallet-design/use-tight-coupling/)과 [느슨한 팔레트 결합 사용](/reference/how-to-guides/pallet-design/use-loose-coupling/)을 참조하세요.

### 오프체인 워커

오프체인 작업을 사용하면 오프체인 소스에서 데이터를 조회하거나 데이터를 오프체인에서 처리할 수 있습니다. 예를 들어, 오프체인 워커를 사용하면 블록 실행 시간 제한을 초과하는 작업을 실행할 수 있습니다. 그러나 오프체인 작업의 특성에는 의도하지 않은 결과가 발생할 수 있습니다. 오프체인 워커를 사용할 계획이 있다면 다음 사항을 고려해야 합니다:

- 기본적으로 오프체인 워커는 검증자 노드가 블록 작성을 수행할 때 실행됩니다.
- 검증자가 아닌 노드에서 오프체인 워커를 실행하려면 `--offchain-worker always` 명령줄 옵션을 사용해야 합니다.
- 어떤 노드(검증자든 아니든)에서든 오프체인 작업을 실행하지 않으려면 `--offchain-worker never` 명령줄 옵션을 사용할 수 있습니다.
- 네트워크에서 병렬로 실행되는 오프체인 워커가 있는 경우 경쟁 조건을 피하기 위해 동시성 프로그래밍 기법을 구현해야 할 수 있습니다.
- 기본적으로 오프체인 워커는 블록이 최종화되지 않았더라도 모든 블록 가져오기에 대해 트리거됩니다.
- 오프체인 워커는 상태에 대한 완전한 액세스 권한을 갖기 때문에 특정 경우에만 실행되도록 트리거하는 조건을 만들 수 있습니다.

오프체인 작업에 대한 자세한 내용은 [오프체인 작업](/learn/offchain-operations/)을 참조하세요. 오프체인 구성 요소 사용 예제는 [오프체인 워커](/reference/how-to-guides/offchain-workers/)를 참조하세요.

### 스토리지

[런타임 스토리지](/build/runtime-storage/)에서 설명한 대로, 블록체인 스토리지에 대한 기본 원칙은 저장하는 데이터 항목의 수와 크기를 최소화하는 것입니다. 불필요한 데이터를 저장하면 네트워크 성능이 느려지고 리소스가 고갈될 수 있습니다.

코드를 계획하고 검토할 때 다음 가이드라인을 염두에 두세요:

- 중요한 정보만 저장하세요.
- 중간 또는 일시적인 정보는 저장하지 마세요.
- 작업이 실패하는 경우 필요하지 않은 데이터는 저장하지 마세요.
- 가능하다면 다른 구조에 이미 저장된 정보를 저장하지 마세요.
- 가능한 경우 제한된 길이의 해시 데이터를 저장하세요.

일반적으로, 복잡성과 읽기 및 쓰기 작업 수를 줄이기 위해 하나의 큰 데이터 구조를 가지는 것이 좋습니다. 그러나 항상 그렇지는 않으며, 데이터를 케이스별로 측정하고 최적화하기 위해 벤치마킹을 사용해야 합니다.

리스트와 스토리지 맵은 스토리지 비용이 발생하므로 사용 방법에 주의해야 합니다. 리스트나 맵에 있는 항목이 많을수록 항목을 반복하는 것이 런타임 성능에 더 큰 영향을 미칩니다. 스토리지 맵은 종종 무제한 데이터 집합을 저장하며, 맵의 요소에 액세스하는 것은 리스트의 요소에 액세스하는 것보다 더 많은 데이터베이스 읽기가 필요하기 때문에, 스토리지 맵을 사용할 때 반복 작업은 상당히 비용이 많이 들 수 있습니다.

스토리지 맵에서 항목을 반복하는 데 필요한 시간이 특히 중요한 경우, 프로젝트가 파라체인인 경우에는 특히 중요합니다. 스토리지를 반복하는 데 필요한 시간이 블록 생성에 허용된 최대 시간을 초과하면 블록체인이 블록 생성을 중지하고 작동을 중지합니다.

일반적으로, 스토리지 맵에 무제한 데이터를 가지지 않도록하고, 큰 데이터 집합을 반복하는 것을 피해야 합니다. 벤치마크를 사용하여 런타임의 모든 함수의 성능을 다양한 조건에서 테스트하고, 많은 항목을 포함하는 리스트나 스토리지 맵의 반복 작업을 포함한 특정 조건에 대한 성능을 테스트함으로써 언제 최적화를 위해 요소 수나 루프 반복 횟수를 제한하는 것이 가장 좋은지 확인할 수 있습니다.

스토리지와 스토리지 구조에 대한 자세한 가이드라인은 [상태 전이와 스토리지](/learn/state-transitions-and-storage)와 [런타임 스토리지](/build/runtime-storage/)를 참조하세요. 스토리지 반복에 대한 자세한 내용은 [스토리지 맵 반복](/build/runtime-storage/#iterating-over-storage-maps)을 참조하세요.

### 이벤트

팔레트는 일반적으로 런타임의 데이터 변경 사항이나 조건에 대한 알림을 받는 사용자나 애플리케이션과 같은 외부 엔티티에게 이벤트를 발생시킵니다.

커스텀 팔레트에서 다음과 같은 이벤트 관련 정보를 정의할 수 있습니다:

- 이벤트의 유형.
- 이벤트에 포함된 정보.
- 이벤트를 발생시키는 조건.

일반적으로 이벤트는 변경 사항을 알리기 위해 사용자나 애플리케이션에게 정보를 전달합니다. 이벤트는 상태의 차이를 설명하거나 자세한 정보를 포함하지 않도록 해야 합니다. 추가 정보를 이벤트에 포함시키는 것은 저장 및 이벤트 생성에 필요한 저장 및 계산 오버헤드를 증가시키므로 필요한 정보 이상으로 이벤트에 추가 정보를 포함시키는 것에 주의해야 합니다. 변경 사항에 대한 추가 정보가 필요한 경우 사용자는 체인 상태를 쿼리할 수 있습니다.

커스텀 팔레트에 이벤트를 추가하는 방법에 대한 정보는 [이벤트 선언](/build/events-and-errors/#declaring-an-event)을 참조하세요.

## 안전하지 않거나 보안에 취약한 패턴

안전한 작업과 코딩 원칙은 데이터 무결성과 블록체인의 신뢰성을 보장하는 데 중요합니다. 올바르게 처리되지 않으면 다음과 같은 몇 가지 일반적인 안전하지 않거나 보안에 취약한 코딩 관행은 오류를 발생시킬 수 있거나 체인이 공격에 취약해질 수 있습니다. 체인의 로직을 작성하는 동안 다음의 잠재적인 문제점에 특히 주의해야 합니다:

- [오류 처리](#오류-처리)
- [안전하지 않은 수학: 부동 소수점 수](#안전하지-않은-수학-부동-소수점-수)
- [안전하지 않은 수학: 오버플로](#안전하지-않은-수학-오버플로)
- [무제한 Vec 데이터 유형](#무제한-vec-데이터-유형)

### 오류 처리

런타임 코드는 모든 오류 상황을 명시적으로 처리해야 합니다. 일반적으로 오류 처리를 위해 `panic!` 매크로를 사용해서는 안 됩니다(테스트와 벤치마크를 제외한 경우). 런타임 함수는 panic을 발생시키지 않아야 하며, 오류를 던지지 않아야 합니다. 컴파일러에서 감지한 버그만이 예외적으로 패닉 오류를 발생시킬 수 있습니다.

Rust에서는 `Result` 타입을 사용하여 `Err` 변형을 사용하여 오류를 반환하는 함수를 작성해야 합니다. `Result` 타입과 `Err` 변형을 사용하면 함수가 실패한 경우를 패닉 없이 나타낼 수 있습니다. 모범 사례로, 문제를 진단하기 쉽도록 여러 개별적이고 구체적인 오류 메시지를 가져야 합니다.

또한, 런타임에서 `Result` 타입과 `unwrap()`을 함께 사용하는 것은 정의되지 않은 동작을 발생시킬 수 있습니다. `unwrap()` 대신 `ok_or`, `unwrap_or`, `ensure`를 사용하거나 일치 패턴에서 `Err`을 반환하는 것을 시도해보세요. 예를 들면 다음과 같습니다:

```text
let a = TryInto::<u128>::try_into(id.fee).ok().unwrap();
let b = a.checked_mul(8).ok_or(Error::<T>::Overflow)?
         .checked_div(10).ok_or(Error::<T>::Overflow)?;

let b = id.fee
    .checked_mul(&8u32.saturated_into()).ok_or(Error::<T>::Overflow)?
    .checked_div(&10u32.saturated_into()).ok_or(Error::<T>::Overflow)?;
```

팔레트에서 오류 처리에 대한 자세한 내용은 [Error 팔레트 속성](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#error-palleterror-optional)과 [Errors](/build/events-and-errors/#errors)를 참조하세요.

### 안전하지 않은 수학: 부동 소수점 수

블록체인은 독립적인 노드가 신뢰할 수 있는 합의에 도달할 수 있도록 결정론적인 작업을 필요로 합니다. 부동 소수점 수는 결정론적인 결과를 얻을 수 없으므로, 부동 소수점 수를 사용하는 작업은 피해야 하며, 런타임에서는 항상 고정 소수점 산술을 사용해야 합니다. Substrate은 [`sp_arithmetic`](https://paritytech.github.io/substrate/master/sp_arithmetic/index.html) 크레이트에서 고정 소수점 산술을 사용할 수 있도록 기본 기능을 제공합니다.

필요한 정밀도에 따라 다음과 같은 **Per** 메서드를 사용하여 전체의 일부를 나타낼 수 있습니다.

- 퍼센트: [0, 100]에서 [0, 1]을 나타냅니다.
- 퍼밀: [0, 1_000_000]에서 [0, 1]을 나타냅니다.
- 퍼빌: [0, 1_000_000_000]에서 [0, 1]을 나타냅니다.

더 높은 정밀도를 사용하려면 더 큰 크기의 데이터 유형이 필요하므로, 더 많은 정밀도는 비용이 발생합니다.

### 안전하지 않은 수학: 오버플로

오버플로는 반환할 데이터의 계산된 값이 정의된 데이터 유형의 한계를 초과하는 경우 발생합니다. 데이터 오버플로를 처리하는 두 가지 방법이 있습니다: 포화 메서드 사용 또는 유효하지 않은 값의 경우를 처리하는 체크된 산술 연산 사용입니다.

- 포화 메서드 사용: 연산의 결과가 유형에 대한 한계를 초과하는 경우, 포화 메서드는 해당 유형의 최대값을 반환합니다. 결과가 너무 작을 경우, 포화 메서드는 해당 유형의 최소값을 반환합니다. 자세한 내용은 [Saturating](https://doc.rust-lang.org/std/num/struct.Saturating.html)을 참조하세요.

- `checked_*` 메서드 사용: 이러한 메서드는 계산을 격리된 환경에서 수행하고 결과에 따라 Some 또는 None을 반환합니다. 자세한 내용은 [checked_add](https://doc.rust-lang.org/std/primitive.u32.html#method.checked_add)를 참조하세요.

### 무제한 Vec 데이터 유형

[런타임 스토리지](/build/runtime-storage/)와 [스토리지](#스토리지)에서 언급한 대로, 체인의 성능과 보안을 보장하기 위해 저장하는 데이터 항목의 수와 크기를 최소화하는 것이 중요합니다. 크기에 제한을 설정하지 않고 Vec 데이터 유형을 사용하면, 제한된 리소스의 의도적인 또는 의도하지 않은 남용에 취약해질 수 있습니다. 예를 들어, 악의적인 사용자나 제한 없이 작동하는 종단 사용자가 제한된 스토리지 용량을 초과하는 무제한 데이터를 추가하여 런타임에서 정의되지 않은 동작이 발생할 수 있습니다. 일반적으로 사용자 동작에 따라 크기가 결정되는 스토리지 항목은 제한을 설정해야 합니다.

다음 코드는 무제한 `Vec` 데이터 유형을 보여줍니다:

```rust
type Proposal<T: Config> = StorageMap<_, Blake2_128Concat, T::Hash, Vec<T::Hash>, ValueQuery>;
```

더 안전한 코드를 위해 `Vec` 데이터 유형을 `BoundedVec` 데이터 유형으로 대체하세요:

```rust
type Proposal<T: Config> = StorageMap<_, Blake2_128Concat, T::Hash, BoundedVec<T::Hash, ValueQuery>;
```

기본적으로, 모든 팔레트 스토리지 항목은 `pallet_prelude::MaxEncodedLen` 속성에서 정의된 제한에 의해 제한됩니다. `#[pallet::without_storage_info]` 매크로를 사용하면 이 기본 동작을 재정의할 수 있습니다. 전체 팔레트에 대해 무제한 스토리지가 필요한 경우에만 이 매크로를 사용하세요. 예를 들면 다음과 같습니다:

```text
#[pallet::pallet]
#[pallet::without_storage_info]
pub struct Pallet<T>(_);
```

이 매크로는 팔레트의 모든 스토리지 항목에 적용되므로, 테스트나 개발 환경에서만 사용해야 합니다. 제품 환경에서는 절대로 `#[pallet::without_storage_info]` 매크로를 사용하지 마세요. 테스트가 끝난 후에 이 매크로를 제거하여 팔레트가 기본 동작을 따르도록 보장할 수 있습니다. 필요한 경우 `#[pallet::unbounded]` 매크로를 사용하여 특정 스토리지 항목을 무제한으로 선언할 수 있습니다.

`BoundedVec` 데이터 유형을 사용하여 스토리지를 제한하는 방법에 대한 자세한 내용은 [Bounds 생성](/build/runtime-storage/#create-bounds)과 [BoundedVec](https://crates.parity.io/frame_support/storage/bounded_vec/struct.BoundedVec.html)를 참조하세요. 팔레트 매크로에 대한 자세한 내용은 [FRAME 매크로](/reference/frame-macros/)를 참조하세요.

### 안전한 해시 알고리즘

Substrate은 기본적으로 다음과 같은 해시 알고리즘을 제공합니다:

- `xxHash`는 빠른 해싱 함수이지만 암호화적으로 안전하지 않습니다. 이 해시 알고리즘을 사용하는 함수는 외부 엔티티가 입력을 조작하고 시스템을 공격할 수 있는 해시 충돌(다른 입력이 동일한 출력으로 해시되는 경우)이 발생할 수 있습니다. 이러한 해시 알고리즘은 외부 엔티티가 액세스할 수 없는 함수에서만 사용해야 합니다.
- `Blake2`는 비교적 빠른 암호화 해싱 함수입니다. 대부분의 경우, 보안이 중요한 상황에서 Blake2 해시 알고리즘을 사용할 수 있습니다. 그러나 Substrate은 `Hasher` 트레이트를 구현하는 모든 해시 알고리즘을 지원할 수 있습니다.

해싱 알고리즘에 대한 자세한 내용은 [해싱 알고리즘](/learn/cryptography//#hashing-algorithms)을 참조하세요.

### 부정확한 Weight

Weight는 블록에서 트랜잭션을 실행하는 데 소비되는 리소스를 나타내는 Substrate의 개념입니다. 트랜잭션을 실행하는 데 필요한 적절한 weight는 하드웨어, 계산 복잡성, 스토리지 요구 사항 및 수행되는 데이터베이스 작업과 같은 여러 요소에 따라 달라집니다. 모든 실행 가능한 트랜잭션에 적절한 weight를 할당해야 합니다.

Weight가 동일한 여러 트랜잭션이 있는 경우, weight 할당이 실제 실행 시간을 정확하게 반영하지 않을 수 있습니다. 벤치마킹을 사용하면 런타임의 각 함수가 다양한 상황에서 소비할 리소스를 평가하고 예측할 수 있습니다. 각 런타임 함수의 예상 weight를 모델링하면 블록체인이 일정 기간 동안 얼마나 많은 트랜잭션 또는 시스템 수준 호출을 실행할 수 있는지 계산할 수 있습니다.

트랜잭션에 너무 낮은 weight를 설정하면 공격자나 알지 못하는 사용자가 과중된 블록을 생성하여 블록 생성 시간 초과를 발생시킬 수 있습니다. 모든 트랜잭션이 리소스 소비에 영향을 미치는 요소를 고려한 적절한 weight를 가지도록 모든 함수에 대해 적절한 벤치마크 테스트를 실행해야 합니다.

벤치마크와 weight 계산에 대한 자세한 내용은 [벤치마크와 Weight](/test/benchmark/#benchmarking-and-weight)와 [Weights](/reference/how-to-guides/weights/)를 참조하세요.

### 난수 생성

난수는 블록체인에서 여러 가지 응용 프로그램에서 사용됩니다. Substrate은 기본적으로 두 가지 난수 생성 방법을 제공합니다.

- [Insecure Randomness Collective Flip](https://paritytech.github.io/substrate/master/pallet_insecure_randomness_collective_flip/index.html) 팔레트는 이전 81개 블록의 블록 해시를 기반으로 난수 값을 생성합니다. 이 팔레트는 약한 공격자에 대한 방어나 테스트와 같은 낮은 보안 요구 사항에서 유용할 수 있습니다. 예를 들어, 난수를 사용하는 팔레트를 테스트할 때 이 팔레트를 사용할 수 있습니다. 제품 환경에서는 사용해서는 안됩니다.

- [BABE](https://paritytech.github.io/substrate/master/pallet_babe/index.html) 팔레트는 검증 가능한 난수 함수(VRF)를 사용하여 보다 안전한 난수를 생성합니다. 이 팔레트는 제품 수준의 난수를 제공합니다. 그러나 모든 목적에 적합하지는 않습니다. 예를 들어, BABE 팔레트가 제공하는 난수는 도박 애플리케이션에는 적합하지 않습니다.

이러한 팔레트 대신에 보안 난수로서 오라클을 사용할 수 있습니다.

난수 사용에 대한 자세한 내용은 [난수](/build/randomness/)와 [난수 통합](/reference/how-to-guides/pallet-design/incorporate-randomness/)을 참조하세요.

## Anti-patterns

Anti-patterns 는 문제를 해결하기 위해 고안된 솔루션이지만, 문제를 해결하는 대신 더 많은 문제를 야기하는 솔루션입니다. 피해야 할 여러 가지 코딩 패턴이 안티 패턴 범주에 속하며, 다음과 같은 패턴을 피해야 합니다.

### 스토리지에서 읽기 위해 함수를 디스패치하지 마세요

Substrate에서는 디스패치 가능한 함수 호출을 사용하여 스토리지에서 항목을 읽어오지 않아야 합니다. 대신 스토리지 항목에 대한 `getter` 매크로를 정의하거나 RPC 메서드를 사용해야 합니다. 예를 들어, 다음과 같은 스토리지 맵이 있다고 가정해 봅시다:

```rust
#[pallet::storage]
#[pallet::getter(fn product_information)]
pub type ProductInformation<T: Config> = StorageMap<_, Blake2_128Concat, T::Hash, Product<T::AccountId, T::Hash>>;
```

다음과 같이 `Self::product_information(id)`를 호출하여 항목을 읽을 수 있습니다:

```rust
let product = Self::product_information(id);
```

다음과 같이 별도의 디스패치 가능한 함수를 작성하는 대신에:

```rust
// !!! Don't create a dispatchable function to read storage state !!!
~~~~#[pallet::weight(T::WeightInfo::get_product())]
pub fn get_product(
origin: OriginFor<T>,
id: T::Hash
) -> DispatchResult {

let sender = ensure_signed(origin)?;
let product = if <ProductInformation<T>>::contains_key(&id) {
                Some(Self::product_information(&id))
            } else { None };

match product {
            Some(value) => {
                Self::deposit_event(Event::ItemFetched(value));
            }
            None => return Err(Error::<T>::NotFound.into()),
        }
   Ok(())
}
```

스토리지 선언과 `getter` 매크로에 대한 자세한 내용은 [스토리지 항목 선언](/build/runtime-storage/#declaring-storage-items)을 참조하세요.

스토리지 `getter` 매크로가 요구 사항에 충분하지 않은 경우, 커스텀 RPC 메서드를 만들 수 있습니다. 커스텀 RPC 메서드를 만드는 방법에 대한 정보는 [노드에 커스텀 RPC 추가](https://hackmd.io/@d9WGfolYQmSRBmnxyOQmAg/BkEANECJs)를 참조하세요.