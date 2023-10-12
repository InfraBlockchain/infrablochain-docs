---
title: 트랜잭션, Weight 및 수수료
description: 리소스를 실행하는 데 필요한 자원을 Substrate weight 시스템과 트랜잭션 수수료 계산을 통해 계산하는 방법을 설명합니다.
keywords:
---

트랜잭션이 실행되거나 데이터가 체인에 저장되면, 해당 활동은 체인의 상태를 변경하고 블록체인 리소스를 소비합니다.
블록체인의 리소스는 제한되어 있기 때문에, 체인 상의 작업이 리소스를 어떻게 소비하는지 관리하는 것이 중요합니다.
저장 용량과 같은 실질적인 제한 외에도, 블록체인 리소스는 악의적인 사용자에게 잠재적인 공격 경로를 제공합니다.
예를 들어, 악의적인 사용자는 네트워크를 과부하로 만들기 위해 메시지로 네트워크를 과부하로 만들 수 있습니다.
블록체인 리소스가 고갈되거나 과부하가 걸리지 않도록 하려면, 리소스가 어떻게 사용 가능하게 되고 어떻게 소비되는지 관리해야 합니다.
관리해야 할 리소스는 다음과 같습니다:

- 메모리 사용량
- 스토리지 입출력
- 연산량
- 트랜잭션 및 블록 크기
- 상태 데이터베이스 크기

Substrate는 블록 작성자가 리소스에 대한 액세스를 관리하고 체인의 개별 구성 요소가 단일 리소스를 지나치게 소비하지 않도록하는 여러 가지 방법을 제공합니다.
블록 작성자가 사용할 수 있는 가장 중요한 메커니즘 중 두 가지는 **weight**와 **트랜잭션 수수료**입니다.

[Weight](/reference/glossary/#weight)는 블록을 검증하는 데 걸리는 시간을 관리하는 데 사용됩니다.
일반적으로, weight는 블록 안의 트랜잭션을 실행하는 데 걸리는 시간을 나타내는 데 사용됩니다.
블록이 소비할 수 있는 실행 시간을 제어함으로써, weight는 스토리지 입출력 및 연산에 제한을 둡니다.

블록에 허용된 일부 weight는 블록의 초기화 및 완료의 일부로 소비됩니다.
Weight는 필수적인 내부 트랜잭션을 실행하는 데도 사용될 수 있습니다.
블록이 실행 시간을 너무 많이 소비하지 않도록 하고 악의적인 사용자가 불필요한 호출로 시스템을 과부하로 만드는 것을 방지하기 위해, weight는 **트랜잭션 수수료**와 함께 사용됩니다.

트랜잭션 수수료는 실행 시간, 계산 및 작업 수행에 필요한 호출 수를 제한하는 경제적 인센티브를 제공합니다.
트랜잭션 수수료는 블록체인이 경제적으로 지속 가능하도록 하는 데에도 사용됩니다. 사용자가 시작한 트랜잭션에 일반적으로 적용되며 트랜잭션 요청이 실행되기 전에 차감됩니다.

## 수수료 계산 방법

트랜잭션의 최종 수수료는 다음 매개변수를 사용하여 계산됩니다:

- _기본 수수료_: 사용자가 트랜잭션에 지불하는 최소 금액입니다. 런타임에서 **기본 weight**로 선언되며 `WeightToFee`를 사용하여 수수료로 변환됩니다.

- _weight 수수료_: 트랜잭션이 소비하는 실행 시간(입출력 및 연산)에 비례하는 수수료입니다.

- _길이(Length) 수수료_: 트랜잭션의 인코딩된 길이에 비례하는 수수료입니다.

- _팁_: 트랜잭션의 우선순위를 높이기 위해 선택적으로 추가되는 팁으로, 트랜잭션 큐에 포함될 확률을 높입니다.

기본 수수료와 비례 weight 및 길이 수수료는 **포함 수수료**를 구성합니다.
포함 수수료는 트랜잭션이 블록에 포함되기 위해 필요한 최소 수수료입니다.

## 트랜잭션 수수료 결제 팔레트 사용

[트랜잭션 수수료 결제 팔레트](https://paritytech.github.io/substrate/master/pallet_transaction_payment/index.html)는 포함 수수료를 계산하는 기본 로직을 제공합니다.

트랜잭션 수수료 결제 팔레트를 사용하여 다음을 수행할 수도 있습니다:

- `Config::WeightToFee`를 사용하여 통화 유형을 기반으로 weight 값을 공제 가능한 수수료로 변환합니다.

- `Config::FeeMultiplierUpdate`를 사용하여 이전 블록의 최종 상태를 기반으로 다음 블록의 수수료를 정의하는 데 사용할 배수를 정의합니다.

- `Config::OnChargeTransaction`을 사용하여 트랜잭션 수수료의 인출, 환불 및 예치를 관리합니다.

이러한 구성 특성에 대한 자세한 내용은 [트랜잭션 수수료 결제](https://paritytech.github.io/substrate/master/pallet_transaction_payment/index.html) 문서에서 알아볼 수 있습니다.

트랜잭션 수수료는 트랜잭션이 실행되기 전에 인출됩니다.
트랜잭션이 실행된 후에는 트랜잭션의 실제 리소스 사용량을 반영하기 위해 트랜잭션 weight를 조정할 수 있습니다.
트랜잭션이 예상보다 적은 리소스를 사용하는 경우, 트랜잭션 수수료가 보정되고 조정된 트랜잭션 수수료가 환불됩니다.

### 포함 수수료 자세히 살펴보기

최종 수수료를 계산하는 공식은 다음과 같습니다:

```
포함 수수료 = 기본 수수료 + 길이 수수료 + [목표 수수료 조정 * weight 수수료];
최종 수수료 = 포함 수수료 + 팁;
```

이 공식에서 `목표 수수료 조정`은 네트워크 혼잡도에 기반하여 최종 수수료를 조정할 수 있는 배수입니다.

- `기본 수수료`는 서명 검증과 같은 포함 오버헤드를 포함합니다.

- `길이 수수료`는 인코딩된 외부 호출의 길이에 비례하는 바이트 단위 수수료입니다.

- `weight 수수료`는 다음 두 매개변수를 사용하여 계산됩니다:

  런타임에서 선언된 `ExtrinsicBaseWeight`는 모든 외부 호출에 적용됩니다.

  `#[pallet::weight]` 주석은 외부 호출의 복잡성을 고려합니다.

weight를 통화로 변환하려면, 런타임은 `Convert<Weight,Balance>` 변환 함수를 구현하는 `WeightToFee` 구조체를 정의해야 합니다.

트랜잭션이 실행되기 전에 호출자에게 포함 수수료가 청구됩니다.
트랜잭션이 실행될 때 호출자의 잔액에서 수수료가 차감됩니다.

### 잔고가 부족한 계정

계정에 포함 수수료를 지불하고 **최소한의 예치(existential deposit)**를 유지하기에 충분한 잔액이 없는 경우, 트랜잭션이 취소되어 수수료가 차감되지 않고 트랜잭션이 실행되지 않도록 해야 합니다.

Substrate는 이 롤백 동작을 강제하지 않습니다.
그러나 이러한 시나리오는 트랜잭션 큐 및 블록 생성 로직이 해당 트랜잭션이 블록에 추가하기 전에 이를 방지하기 위해 검사를 수행하기 때문에 드물게 발생할 것입니다.

### 수수료 배수

포함 수수료 공식은 항상 동일한 입력에 대해 동일한 수수료를 결과로 합니다.
그러나 weight는 동적일 수 있으며, [`WeightToFee`](https://paritytech.github.io/substrate/master/pallet_transaction_payment/pallet/trait.Config.html#associatedtype.WeightToFee)가 정의된 방식에 따라 최종 수수료에 일정한 변동성이 포함될 수 있습니다.

이러한 변동성을 고려하기 위해, 트랜잭션 수수료 결제 팔레트는 [`FeeMultiplierUpdate`](https://paritytech.github.io/substrate/master/pallet_transaction_payment/pallet/trait.Config.html#associatedtype.FeeMultiplierUpdate) 구성 가능한 매개변수를 제공합니다.

기본 업데이트 함수는 Polkadot 네트워크에서 영감을 받아, 블록의 포화 수준을 기준으로 수수료를 조정합니다.
이전 블록이 더 포화되어 있는 경우, 수수료가 약간 증가합니다.
마찬가지로, 이전 블록이 대상보다 적은 트랜잭션를 가지고 있는 경우, 수수료가 약간 감소합니다.
수수료 배수 조정에 대한 자세한 내용은 [Web3 Foundation 페이지](https://research.web3.foundation/Polkadot/overview/token-economics#relay-chain-transaction-fees-and-per-block-transaction-limits)를 참조하십시오.

## 특수 요구 사항을 갖는 트랜잭션

포함 수수료는 실행 전에 계산 가능해야 하므로, 고정된 로직만을 나타낼 수 있습니다.
일부 트랜잭션는 다른 전략으로 리소스를 제한할 필요가 있습니다.
예를 들어:

- 보증금은 체인 상의 이벤트 이후에 반환되거나 벌금으로 사용될 수 있는 수수료 유형입니다.

  예를 들어, 투표에 참여하려면 사용자가 보증금을 지불하도록 요구할 수 있습니다. 보증금은 투표 종료 시 반환되거나 투표자가 악의적인 행동을 시도한 경우 벌금으로 사용될 수 있습니다.

- 예치금은 나중에 반환될 수 있는 수수료입니다.

  예를 들어, 사용자가 스토리지를 사용하는 작업을 실행하기 위해 예치금을 지불하도록 요구할 수 있습니다. 이후 작업에서 스토리지가 해제되면 사용자의 예치금이 반환될 수 있습니다.

* 소각 작업은 내부 로직에 따라 트랜잭션에 대한 지불에 사용됩니다.

  예를 들어, 트랜잭션이 상태 크기를 증가시키기 위해 새로운 스토리지 항목을 생성하는 경우 트랜잭션이 발신자의 자금을 소각할 수 있습니다.

- 특정 작업에 대한 상수 또는 구성 가능한 제한을 강제할 수 있습니다.

  예를 들어, 기본 스테이킹 팔레트는 검증자 선출 프로세스의 복잡성을 제한하기 위해 nominator가 최대 16개의 검증자를 지명할 수 있도록만 허용합니다.

트랜잭션 수수료에 대한 체인 쿼리를 수행하는 경우, 포함 수수료만 반환됩니다.

## 기본 weight 주석

Substrate의 모든 트랜잭션은 weight를 지정해야 합니다. 이를 위해 주석 기반 시스템을 사용하여 데이터베이스 읽기/쓰기 weight에 대한 고정값 및/또는 벤치마크에 기반한 고정값을 결합할 수 있습니다. 가장 기본적인 예는 다음과 같습니다:

```rust
#[pallet::weight(100_000)]
fn my_dispatchable() {
    // ...
}
```

`ExtrinsicBaseWeight`는 빈 외부 호출을 블록에 포함하는 비용을 고려하여 선언된 weight에 자동으로 추가됩니다.

### weight와 데이터베이스 읽기/쓰기 작업

weight 주석을 배포된 데이터베이스 백엔드와 독립적으로 만들기 위해, weight는 상수로 정의되고, 이를 주석에서 사용하여 디스패처가 수행하는 데이터베이스 액세스를 표현합니다:

```rust
#[pallet::weight(T::DbWeight::get().reads_writes(1, 2) + 20_000)]
fn my_dispatchable() {
    // ...
}
```

이 디스패처는 다른 작업 외에도 데이터베이스 읽기와 쓰기를 각각 한 번씩 수행하며, 추가 20,000을 추가합니다.
데이터베이스 액세스는 일반적으로 `#[pallet::storage]` 블록 내에서 선언된 값이 액세스될 때마다 발생합니다.
그러나 고유한 액세스만이 계산되며, 값이 액세스되면 캐시되어 다시 액세스되어도 데이터베이스 작업이 발생하지 않습니다.
즉,

- 동일한 값에 대한 여러 읽기는 하나의 읽기로 계산됩니다.
- 동일한 값에 대한 여러 쓰기는 하나의 쓰기로 계산됩니다.
- 동일한 값에 대한 여러 읽기 후 해당 값에 대한 쓰기는 하나의 읽기와 하나의 쓰기로 계산됩니다.
- 쓰기 후 읽기는 하나의 쓰기로 계산됩니다.

### 트랜잭션 클래스

트랜잭션은 세 가지 클래스로 나뉩니다:

- `Normal`
- `Operational`
- `Mandatory`

Weight 주석에서 `Operational` 또는 `Mandatory`로 정의되지 않은 디스패치는 기본적으로 `Normal`로 식별됩니다.
다음과 같이 디스패치 가능한 함수가 다른 클래스를 사용하도록 지정할 수 있습니다:

```rust
#[pallet::dispatch((DispatchClass::Operational))]fn my_dispatchable() {
    // ...
}
```

이 튜플 표기법은 주석된 weight에 기반하여 사용자가 청구되는지 여부를 결정하는 최종 인수를 지정하는 데에도 사용할 수 있습니다. 다른 경우를 지정하지 않으면, `Pays::Yes`로 가정됩니다:

```rust
#[pallet::dispatch(DispatchClass::Normal, Pays::No)]
fn my_dispatchable() {
    // ...
}
```

#### Normal 디스패치

이 클래스의 디스패치는 일반 사용자 트리거 트랜잭션을 나타냅니다.
이러한 유형의 디스패치는 블록의 총 weight 제한의 일부만을 소비합니다.
Normal 디스패치가 소비할 수 있는 블록의 최대 부분에 대한 정보는 [`AvailableBlockRatio`](https://paritytech.github.io/substrate/master/frame_system/limits/struct.BlockLength.html#method.max_with_normal_ratio)를 참조하십시오.
Normal 디스패치는 [트랜잭션 풀](/reference/glossary/#transaction-pool)로 전송됩니다.

#### Operational 디스패치

Normal 디스패치와 달리, Operational 디스패치는 네트워크 기능을 제공하는 디스패치입니다.
Operational 디스패치는 블록의 weight 제한 전체를 소비할 수 있습니다.
[`AvailableBlockRatio`](https://paritytech.github.io/substrate/master/frame_system/limits/struct.BlockLength.html#method.max_with_normal_ratio)에 의해 제한되지 않습니다.
이 클래스의 디스패치는 최고 우선순위를 가지며 `length_fee`를 지불하지 않습니다.

#### Mandatory 디스패치

Mandatory 디스패치는 블록의 weight 제한을 초과하더라도 블록에 포함됩니다.
이 디스패치 클래스는 블록 작성자에 의해 제출되는 [내부 트랜잭션](/reference/glossary/#inherent-transactions)에만 사용할 수 있습니다.
이 디스패치 클래스는 블록 유효성 검사 프로세스의 일부인 함수를 나타내는 데 사용됩니다.
이러한 디스패치는 weight와 관계없이 항상 블록에 포함되므로, 검증 프로세스가 악의적인 노드가 블록을 생성하지 못하도록 막는 것이 매우 중요합니다.
일반적으로 다음을 확인하여 이를 수행할 수 있습니다:

- 수행되는 작업이 항상 가볍습니다.
- 작업은 한 번만 블록에 포함될 수 있습니다.

악의적인 노드가 Mandatory 디스패치를 남용하는 것을 어렵게 하기 위해, 오류가 발생하는 블록에는 포함될 수 없습니다.
이 디스패치 클래스는 과부하가 발생하지 않도록 하는 것보다 어떤 블록이라도 생성되는 것이 더 좋다는 가정을 제공하기 위해 존재합니다.

### 동적 weight

고정된 weight와 상수 외에도, weight 계산은 디스패치 가능한 함수의 입력 인수를 고려할 수 있습니다.
weight는 입력 인수를 기반으로 간단한 산술 연산으로 계산될 수 있어야 합니다:

```rust
use frame_support:: {
    dispatch:: {
        DispatchClass::Normal,
        Pays::Yes,
    },
   weights::Weight,

#[pallet::weight(FunctionOf(
  |args: (&Vec<User>,)| args.0.len().saturating_mul(10_000),
  )
]

fn handle_users(origin, calls: Vec<User>) {
    // Do something per user
}
```

## 디스패치 후 weight 보정

디스패치 가능한 함수는 실행 로직에 따라 지정된 weight보다 적은 weight를 소비할 수 있습니다.
weight를 보정하기 위해, 함수는 다른 반환 유형을 선언하고 실제 weight를 반환해야 합니다:

```rust
#[pallet::weight(10_000 + 500_000_000)]
fn expensive_or_cheap(input: u64) -> DispatchResultWithPostInfo {
    let was_heavy = do_calculation(input);

    if (was_heavy) {
        // None은 weight 주석에서 보정이 없음을 의미합니다.
        Ok(None.into())
    } else {
        // 실제로 소비된 weight를 반환합니다.
        Ok(Some(10_000).into())
    }
}
```

## 커스텀 수수료

커스텀 weight 함수 또는 포함 수수료 함수를 통해 커스텀 수수료 시스템을 정의할 수도 있습니다.

### 커스텀 weight

기본 weight 주석 대신, [weights](https://paritytech.github.io/substrate/master/frame_support/weights/index.html) 모듈을 사용하여 커스텀 weight 계산 유형을 만들 수 있습니다.
커스텀 weight 계산 유형은 다음 특성을 구현해야 합니다:

- `WeighData<T>`: 디스패치의 weight를 결정합니다.
- `ClassifyDispatch<T>`: 디스패치의 클래스를 결정합니다.
- `Pays<T>`: 디스패치 발신자가 수수료를 지불해야 하는지 여부를 결정합니다.

Substrate는 이러한 세 가지 특성의 출력 정보를 `DispatchInfo` 구조체로 묶어 모든 `Call` 변형 및 불투명한 트랜잭션 유형에 대해 `GetDispatchInfo`를 구현하여 제공합니다.
이는 System 및 Executive 모듈에서 내부적으로 사용됩니다.

`ClassifyDispatch`, `WeighData` 및 `PaysFee`는 `T`에 대해 일반화되어 있으며, `T`는 원본을 제외한 모든 디스패치 인수의 튜플로 해결됩니다.
다음 예제는 `m * len(args)`로 weight를 계산하는 `struct`를 보여줍니다. 여기서 `m`은 주어진 배수이고 `args`는 모든 디스패치 인수의 연결된 튜플입니다.
이 예제에서, 트랜잭션의 길이가 100바이트를 초과하는 경우 디스패치 클래스는 `Operational`이 되며, 인코딩된 길이가 10바이트보다 큰 경우 수수료가 청구됩니다.

```rust
struct LenWeight(u32);
impl<T> WeighData<T> for LenWeight {
    fn weigh_data(&self, target: T) -> Weight {
        let multiplier = self.0;
        let encoded_len = target.encode().len() as u32;
        multiplier * encoded_len
    }
}

impl<T> ClassifyDispatch<T> for LenWeight {
    fn classify_dispatch(&self, target: T) -> DispatchClass {
        let encoded_len = target.encode().len() as u32;
        if encoded_len > 100 {
            DispatchClass::Operational
        } else {
            DispatchClass::Normal
        }
    }
}

impl<T> PaysFee<T> {
    fn pays_fee(&self, target: T) -> Pays {
        let encoded_len = target.encode().len() as u32;
        if encoded_len > 10 {
            Pays::Yes
        } else {
            Pays::No
        }
    }
}
```

또한 weight 계산 함수는 인수의 최종 유형으로 강제 변환될 수도 있으며, 애매한 유형이 인코딩될 수 있는 것과 달리 특정한 시그니처 `(u32, u64)`를 가진 디스패치와만 사용될 수 있습니다.

```rust
struct CustomWeight;
impl WeighData<(&u32, &u64)> for CustomWeight {
    fn weigh_data(&self, target: (&u32, &u64)) -> Weight {
        ...
    }
}

// 주어진 디스패치:
#[pallet::call]
impl<T: Config<I>, I: 'static> Pallet<T, I> {
    #[pallet::weight(CustomWeight)]
    fn foo(a: u32, b: u64) { ... }
```

이 예에서 `CustomWeight`는 `<T>`에 대한 어떤 가정도 없으므로 특정 시그니처 `(u32, u64)`를 가진 디스패치와만 사용할 수 있습니다.

### 커스텀 포함 수수료

다음 예제는 커스텀 포함 수수료를 어떻게 정의하는지 보여줍니다.
해당 모듈에서 적절한 연관 유형을 구성해야 합니다.

```rust
// 잔고 유형이라고 가정합니다
type Balance = u64;

// 모든 weight에 대해 `100 + 2 * w` 변환을 수수료로 지정하려면
struct CustomWeightToFee;
impl WeightToFee<Weight, Balance> for CustomWeightToFee {
    fn convert(w: Weight) -> Balance {
        let a = Balance::from(100);
        let b = Balance::from(2);
        let w = Balance::from(w);
        a + b * w
    }
}

parameter_types! {
    pub const ExtrinsicBaseWeight: Weight = 10_000_000;
}

impl frame_system::Config for Runtime {
    type ExtrinsicBaseWeight = ExtrinsicBaseWeight;
}

parameter_types! {
    pub const TransactionByteFee: Balance = 10;
}

impl transaction_payment::Config {
    type TransactionByteFee = TransactionByteFee;
    type WeightToFee = CustomWeightToFee;
    type FeeMultiplierUpdate = TargetedFeeAdjustment<TargetBlockFullness>;
}

struct TargetedFeeAdjustment<T>(sp_std::marker::PhantomData<T>);
impl<T: Get<Perquintill>> WeightToFee<Fixed128, Fixed128> for TargetedFeeAdjustment<T> {
    fn convert(multiplier: Fixed128) -> Fixed128 {
        // 아무런 변경 사항이 없습니다. 여기에 수수료 업데이트 정보를 넣으십시오.
        multiplier
    }
}
```

## 다음 단계

이제 weight 시스템이 무엇인지, 트랜잭션 수수료 계산에 어떤 영향을 미치는지, 디스패치 가능한 호출에 대한 weight를 지정하는 방법을 알게 되었습니다.
다음 단계는 디스패치 가능한 함수가 수행하는 작업을 고려하여 올바른 weight를 결정하는 것입니다.
Substrate의 **벤치마킹 함수**와 `frame-benchmarking` 호출을 사용하여 다양한 매개변수로 함수를 테스트하고, 최악의 경우의 weight를 경험적으로 결정할 수 있습니다.

- [벤치마크](/test/benchmark/)
- [SignedExtension](https://paritytech.github.io/substrate/master/sp_runtime/traits/trait.SignedExtension.html)
- [Example 팔레트를 위한 커스텀 weight](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/basic/src/weights.rs)
- [Web3 Foundation 연구](https://research.web3.foundation/Polkadot/overview/token-economics#relay-chain-transaction-fees-and-per-block-transaction-limits)

<!-- - [weight 계산](/reference/how-to-guides/weights/) -->