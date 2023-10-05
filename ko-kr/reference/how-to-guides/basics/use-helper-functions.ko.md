---
title: 도우미 함수 사용하기
description:
keywords:
  - 기본
  - 런타임
---

이 짧은 가이드는 팔렛의 코드 내에서 일반적인 "확인" 검사를 수행하기 위해 도우미 함수를 생성하고 재사용하는 예제를 안내합니다.

팔렛 내부의 호출 가능한 함수는 때로는 다른 호출 가능한 함수에서 공통으로 사용되는 로직을 재사용합니다.
이 경우, 이 로직을 자체적인 개인 함수로 리팩토링하는 것이 유용합니다.
또 다른 경우에는 호출 가능한 함수가 호출 가능한 내부에서 다양한 검사를 수행하기 위해 코드 양이 증가함에 따라 읽기 어려워집니다.
이러한 경우 팔렛 외부에서 액세스할 수 없는 도우미 함수를 사용하여 코드의 가독성과 재사용성을 최적화하는 유용한 도구입니다.
이 가이드에서는 산술 오버플로우를 확인하는 adder 도우미를 생성하는 방법을 살펴보겠습니다.

### 도우미 함수 생성하기

우리가 참조할 도우미 함수는 `fn _adder`라고 합니다.
이 함수는 `u32` 타입의 두 정수를 더할 때 오버플로우가 발생하지 않는지 확인합니다.

이 함수는 두 개의 `u32` 정수를 가져와 `checked_add`와 `ok_or`를 사용하여 오버플로우가 발생하지 않는지 확인합니다.
만약 오버플로우가 발생하면 오류를 반환하고, 그렇지 않으면 결과를 반환합니다.

다음은 도우미 함수로서의 모습입니다.
이것은 팔렛의 가장 아래에 위치하게 됩니다:

```rust
impl<T: Config> Pallet<T> {
    fn _adder(num1: u32, num2: u32) -> Result<u32, &'static str> {
        num1.checked_add(num2).ok_or("더할 때 오버플로우 발생")
    }
}
```

## 호출 가능한 함수에서 사용하기

덧셈을 수행할 때 오버플로우를 확인해야 하는 위치를 식별합니다.
같은 코드를 반복해서 작성하는 대신 도우미 함수를 사용합니다.
다음은 부호 있는 외부 호출이 기존 저장 값에 값을 추가할 수 있는 간단한 예입니다:

```rust
// 런타임 외부에서 호출 가능한 외부 호출.
#[pallet::call]
impl<T: Config> Pallet<T> {
  #[pallet::weight(0)]
  fn add_value(
    origin: OriginFor<T>,
    val_to_add: u32
    ) -> DispatchResultWithPostInfo {
      let _ = ensure_signed(origin)?;

      ensure!(val_to_add <= T::MaxAddend::get(), "값은 최대 추가 금액 상수보다 작거나 같아야 합니다");

      // 이전 값 가져오기
      let c_val = SingleValue::<T>::get();

      // 새로운 값 추가 시 오버플로우 확인
      let result = _adder(c_val, val_to_add);

      <SingleValue<T>>::put(result);
      Self::deposit_event(Event::Added(c_val, val_to_add, result));
      Ok(().into())
  }
}
```

## 예제

- [example-offchain-worker](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/offchain-worker/src/lib.rs): 이 팔렛의 호출 가능한 함수에서 사용된 `add_price` 도우미 함수.

## 자료

- [`num` 크레이트의 `checked_add`](https://docs.rs/num/0.4.0/num/traits/trait.CheckedAdd.html)
- [`Option` 크레이트의 `ok_or`](https://doc.rust-lang.org/std/option/enum.Option.html#method.ok_or)