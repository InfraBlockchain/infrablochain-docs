---
title: 조건부 가중치 사용
description:
keywords:
  - 가중치
  - 벤치마킹
  - 런타임
---

Substrate는 트랜잭션 실행 중에 소비되는 리소스를 측정하기 위해 [트랜잭션 가중치](/ko/infrablockchain/learn/substrate/learn/frame/tx-weights-fees.md)라고 하는 메커니즘을 제공합니다.
일반적으로 우리는 벤치마킹에서 반환된 가중치 함수를 사용합니다.
하지만 Substrate는 특정 조건에 따라 완전히 다른 가중치 함수를 적용할 수도 있습니다.
이 가이드에서는 예제를 통해 이를 살펴보겠습니다.
한 번 정의하면 다음과 같이 팔렛에서 직접 사용할 수 있습니다:

`#[pallet::weight(Conditional(\<조건\>)`

## 목표

- 팔렛에서 사용자 정의 가중치를 생성하고 사용합니다.

- 특정 조건에 따라 다른 가중치 함수를 적용하여 외부의 가중치 값을 계산합니다.

다음과 같은 다양한 트레이트를 구현할 것입니다:

- [\`WeighData\`](https://paritytech.github.io/substrate/master/frame_support/weights/trait.WeighData.html#): 함수 내의 데이터를 가중치를 매깁니다.
  \`pallet::weight\`는 \`T\`를 디스패치 인자들의 튜플로 대체하는 \`WeighData<T>\`를 기대합니다.
- [\`PaysFee\`](https://paritytech.github.io/substrate/master/frame_support/weights/trait.PaysFee.html): 디스패치가 수수료를 지불하는지 여부를 지정합니다.
- [\`ClassifyDispatch\`](https://paritytech.github.io/substrate/master/frame_support/weights/trait.ClassifyDispatch.html): 런타임에 수행되는 디스패치의 유형을 알려주는 방법입니다.

## Weight 구조체 작성

1. 텍스트 편집기에서 팔렛의 `lib.rs` 파일을 엽니다.

2. `DispatchClass`와 `Pays`를 선언하여 `use frame_support::dispatch::{DispatchClass, Pays}`로 가져옵니다.

3. 팔렛에 `weights` 기본 요소를 가져옵니다.
   
   ```rust
   use frame_support:: {
    dispatch::{DispatchClass, Pays},
   },
   weights::Weight,

4. `Conditional`이라는 구조체를 선언하고 `Conditional`에 대한 `WeighData` 구현을 작성합니다. 첫 번째 매개변수는 부울 값을 평가하는 조건입니다. 
   
   다음 예제에서는 조건이 true인 경우 가중치가 입력에 대해 선형적으로 증가합니다.
   그렇지 않으면 가중치는 상수입니다.

   ```rust
   pub struct Conditional(u32);
   impl WeighData<(&bool, &u32)> for Conditional {
      fn weigh_data(&self, (switch, val): (&bool, &u32)) -> Weight {
        
      // 첫 번째 매개변수가 true인 경우, 두 번째 매개변수에 대한 가중치는 선형적으로 증가합니다.
         if *switch {
            val.saturating_mul(self.0)
         }
      // 그렇지 않으면 가중치는 상수입니다.
         else {
            self.0
         }
      }
   }
   ```

## 디스패치 호출 분류

팔렛의 `frame_support` 가져오기에 `dispatch::{ClassifyDispatch, DispatchClass, Pays}`를 추가합니다.
이 구현은 [`DispatchClass`](https://paritytech.github.io/substrate/master/frame_support/dispatch/enum.DispatchClass.html)를 필요로 하므로 모든 호출을 일반으로 분류하기 위해 `default`를 사용합니다:

1. 텍스트 편집기에서 팔렛의 `lib.rs` 파일을 엽니다.

2. `DispatchClass`와 `Pays`를 선언하여 `use frame_support::dispatch::{DispatchClass, Pays}`로 가져옵니다.
   
   ```rust
   use frame_support::dispatch::{ClassifyDispatch, DispatchClass, Pays};
   // -- snip --
   
   // ClassifyDispatch 구현
   impl<T> ClassifyDispatch<T> for Conditional {
      fn classify_dispatch(&self, _: T) -> DispatchClass {
         Default::default()
      }
  }
  ```

## Pays 구현

사용자 정의 `WeighData` 구조체에 대해 `Pays`가 어떻게 사용되는지 지정합니다. 
이를 `true`로 설정하면 이 디스패치의 호출자가 수수료를 지불해야 합니다:

1. 텍스트 편집기에서 팔렛의 `lib.rs` 파일을 엽니다.

2. Conditional 구조체에 대해 `Pays`를 구현합니다.
   
   ```rust
   impl Pays for Conditional {
      fn pays_fee(&self) -> bool {
         true
      }
   }
   ```

## 가중치 구조체를 외부에 사용

다음과 같이 팔렛의 외부에서 조건부 가중치 구조체를 사용합니다:

```rust
#[pallet::weight(Conditional(200))]
   fn example(origin: OriginFor<T>, add_flag: bool, val: u32>) -> DispatchResult {
   //...
   }
```

## 예제

- [Example 팔렛에서 사용되는 사용자 정의 `WeightForSetDummy` 구조체](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/basic/src/lib.rs#L305-L350)

## 관련 자료

- [벤치마킹](/ko/infrablockchain/tutorials/test/benchmark.md)
- [수수료 계산](./calculate-fees.md)
- [사용자 정의 가중치 사용](./use-custom-weights.md)