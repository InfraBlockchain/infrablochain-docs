---
title: 기본 테스트 설정하기
description:
keywords:
  - 기본
  - 테스트
  - 런타임
---

팔렛에 대한 테스트를 작성하기 위해 필요한 틀을 설정하는 방법을 배워보세요.

이 가이드에서는 `mock.rs`와 `test.rs`를 테스트의 기초로 사용하는 방법을 안내합니다.
`mock.rs` 파일의 틀은 노드 템플릿을 사용하고, 이 가이드에 맥락을 부여하기 위해 `pallet-testing`이라는 임의의 팔렛을 사용합니다.
이 팔렛은 `add_value`라는 하나의 함수를 포함하며, 이 함수는 원본(origin)과 `u32`를 인자로 받고, 값을 `MaxValue`라는 상수와 비교하여 `Ok(())`를 반환합니다. 이 상수는 모의 런타임에서 지정합니다.

## 템플릿 노드를 보일러플레이트로 사용하기

`pallet-testing/src` 안에서 먼저 `mock.rs`와 `tests.rs`라는 두 개의 빈 파일을 생성해야 합니다.

[`template/src/mock.rs`](https://github.com/substrate-developer-hub/substrate-node-template/blob/main/pallets/template/src/mock.rs)의 내용을 복사하여 붙여넣습니다.
이를 `pallet-testing` 팔렛에 맞게 사용자 정의할 것입니다.

## 팔렛을 테스트하기 위한 모의 런타임 생성하기

1. 먼저 `pallet-testing/src/mock.rs`를 수정하여 `pallet-testing` 팔렛을 포함하도록 합니다. 이를 위해 다음 코드 부분을 변경해야 합니다:

2. 첫 줄을 팔렛의 이름으로 대체합니다. 여기서는 `pallet_testing`입니다:

   ```rust
   use crate as pallet_testing;
   /*--snip--*/
   ```

## 모의 런타임 구성하기

1. `frame_support::construct_runtime!`에서 `pallet_template`을 팔렛의 이름으로 대체합니다. 여기서는 `pallet_testing`입니다:

   ```rust
   /*--snip--*/
   TestingPallet: pallet_testing::{Pallet, Call, Storage, Event<T>},
   /*--snip--*/
   ```

2. 모의 런타임에서 팔렛을 구현합니다. `impl pallet_template::Config for Test {...}`를 팔렛의 구성 타입과 팔렛이 필요로 하는 상수 값을 사용하여 대체합니다:

   ```rust
   parameter_types! {
     pub const MaxValue: u32 = 50;
   }

   impl pallet_testing::Config for Test {
     type RuntimeEvent = RuntimeEvent;
     type MaxValue = MaxValue;
   }
   ```

모의 런타임을 사용하려면 `pallet-testing` 팔렛을 위해 `tests.rs` 파일을 설정해야 합니다.

## 테스트 설정 및 생성하기

`tests.rs`에서 `lib.rs`에서 필요한 종속성을 `super`를 사용하여 가져옵니다:

```rust
use super::*;
```

1. 에러가 의도한 대로 작동하는지 테스트합니다:

   ```rust
   #[test]
   fn error_works(){
     new_test_ext().execute_with(|| {
       assert_err!(
         TestingPallet::add_value(RuntimeOrigin::signed(1), 51),
         "value must be <= maximum add amount constant"
       );
     })
   }
   ```

2. 정상적으로 작동해야 하는 테스트를 생성합니다:

   ```rust
   #[test]
   fn test_should_work() {
     new_test_ext().execute_with(|| {
       assert_ok!(
         TestingPallet::add_value(RuntimeOrigin::signed(1), 10)
       );
     })
   }
   ```

3. 실패해야 하는 다른 테스트를 생성합니다:

   ```rust
   #[test]
   fn test_should_fail() {
     new_test_ext().execute_with(|| {
       assert_ok!(
         TestingPallet::add_value(RuntimeOrigin::signed(1), 100)
       );
     })
   }
   ```

## 테스트 실행하기

팔렛의 디렉토리에서 다음 명령을 실행합니다:

```bash
cargo test
```

## 예시

- [`pallet_template`](https://github.com/substrate-developer-hub/substrate-node-template/blob/master/pallets/template/src/tests.rs#L1-L23)의 테스트
- [`reward-coin`](https://github.com/substrate-developer-hub/substrate-how-to-guides/blob/main/example-code/template-node/pallets/reward-coin/src/tests.rs) 예시 팔렛의 테스트

## 관련 자료

- [단위 테스트](/ko/infrablockchain/tutorials/test/unit-testing.md)
- [트랜스퍼 함수 테스트하기](./test-a-transfer-function.md)
- [`assert_ok!`](https://paritytech.github.io/substrate/master/frame_support/macro.assert_ok.html)
- [`assert_err!`](https://paritytech.github.io/substrate/master/frame_support/macro.assert_err.html)