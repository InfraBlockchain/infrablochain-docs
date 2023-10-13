---
title: 벤치마크 추가
description: 벤치마크 프레임워크를 사용하여 팔렛의 실행 요구 사항을 추정하는 방법을 보여줍니다.
keywords:
  - 가중치
  - 벤치마크
  - 런타임
---

이 가이드는 팔렛의 함수에 필요한 실행 시간에 대한 현실적인 추정을 생성하기 위해 [벤치마크](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/benchmarking) 명령을 실행하고 벤치마크를 테스트하는 방법을 설명합니다.
이 가이드에서는 벤치마크 결과를 사용하여 트랜잭션 가중치를 업데이트하는 방법에 대해서는 다루지 않습니다.

## 팔렛에 벤치마크 추가

1. 팔렛의 [`Cargo.toml`](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/basic/Cargo.toml) 파일을 텍스트 편집기에서 엽니다.

2. 팔렛의 [dependencies]에 `frame-benchmarking` 크레이트를 추가하고 다른 의존성과 동일한 버전과 브랜치를 사용합니다.

   예를 들어:

   ```toml
   frame-benchmarking = { version = "4.0.0-dev", default-features = false, git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-v1.0.0", optional = true }
   ```

3. 팔렛의 [features] 목록에 `runtime-benchmarks`를 추가합니다.

   예를 들어:

   ```toml
   [features]
   runtime-benchmarks = ["frame-benchmarking/runtime-benchmarks"]
   ```

4. 팔렛의 `std` 기능 목록에 `frame-benchmarking/std`를 추가합니다.

   예를 들어:

   ```toml
   std = [
      ...
      "frame-benchmarking/std",
      ...
   ]
   ```

## 벤치마크 모듈 추가

1. 팔렛의 `src` 폴더에 `benchmarking.rs`와 같은 이름의 새 텍스트 파일을 생성합니다.

2. `benchmarking.rs` 파일을 텍스트 편집기에서 열고, 팔렛에 대한 벤치마크를 정의하는 Rust 모듈을 생성합니다.

   Rust 모듈에는 팔렛의 함수에 대한 벤치마크를 정의하는 코드가 포함되어야 합니다.
   기존 팔렛의 `benchmarking.rs`를 참고하여 Rust 모듈에 포함해야 할 내용을 알 수 있습니다.
   일반적으로, 모듈은 다음과 유사한 코드를 포함해야 합니다:

   ```rust
   #![cfg(feature = "runtime-benchmarks")]
   mod benchmarking;

   use crate::*;
   use frame_benchmarking::{benchmarks, whitelisted_caller};
   use frame_system::RawOrigin;
   
   benchmarks! {
      // 개별 벤치마크를 여기에 추가하세요
      benchmark_name {
         /* 초기 상태를 설정하는 코드 */
      }: {
         /* 벤치마크 대상 함수를 테스트하는 코드 */
      }
      verify {
         /* 선택적 검증 */
      }
   }
   ```

3. 팔렛의 함수 중 가장 계산 비용이 많이 드는 경로를 테스트하기 위해 개별 벤치마크를 작성합니다.

   벤치마크 매크로는 벤치마크 모듈에 포함된 각 벤치마크에 대해 테스트 함수를 자동으로 생성합니다.
   예를 들어, 매크로는 다음과 유사한 테스트 함수를 생성합니다:

   ```rust
   fn test_benchmarking_[benchmark_name]<T>::() -> Result<(), &'static str>
   ```

   [pallet-example-basic](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/basic/src/benchmarking.rs)의 벤치마크 모듈은 몇 가지 간단한 샘플 벤치마크를 제공합니다.
   예를 들어:

   ```rust
   benchmarks! {
     set_dummy_benchmark {
       // 벤치마크 설정 단계
       let b in 1 .. 1000;
     }: set_dummy(RawOrigin::Root, b.into()) // 실행 단계
     verify {
		   // 선택적 검증 단계
       assert_eq!(Pallet::<T>::dummy(), Some(b.into()))
     }
   }  
   ```

   이 샘플 코드에서:

   - 벤치마크의 이름은 `set_dummy_benchmark`입니다.
   - 변수 `b`는 `set_dummy` 함수의 실행 시간을 테스트하는 데 사용되는 입력을 저장합니다.
   - `b`의 값은 1부터 1,000까지 변동하므로, 다른 입력 값을 사용하여 벤치마크 테스트를 반복적으로 실행하여 실행 시간을 측정할 수 있습니다.

## 벤치마크 테스트

팔렛의 벤치마크 모듈에 `benchmarks!` 매크로를 사용하여 벤치마크를 추가한 후, 모의 런타임을 사용하여 단위 테스트를 수행하고 벤치마크의 테스트 함수가 `Ok(())`를 반환하는지 확인할 수 있습니다.

1. 텍스트 편집기에서 `benchmarking.rs` 벤치마크 모듈을 엽니다.

2. 벤치마크 모듈의 맨 아래에 `impl_benchmark_test_suite!` 매크로를 추가합니다:

   ```rust
   impl_benchmark_test_suite!(
     MyPallet,
     crate::mock::new_test_ext(),
     crate::mock::Test,
   );
   ```

   `impl_benchmark_test_suite!` 매크로는 다음과 같은 입력을 사용합니다:
   
   - 팔렛에서 생성된 Pallet 구조체 (예: `MyPallet`).
   - 테스트 genesis storage를 생성하는 함수 `new_text_ext()`.
   - 전체 모의 런타임 구조체 `Test`.
   
   이 정보는 단위 테스트를 위해 모의 런타임을 설정하는 데 사용하는 정보와 동일합니다.
   모의 런타임 테스트 환경에서 모든 벤치마크 테스트가 통과하면, 실제 런타임에서 벤치마크를 실행할 때 작동할 가능성이 높습니다.

3. 다음과 유사한 명령을 실행하여 팔렛의 벤치마크 단위 테스트를 모의 런타임에서 실행합니다. 여기서 `pallet-mycustom`이라는 팔렛을 예로 들었습니다:

   ```bash
   cargo test --package pallet-mycustom --features runtime-benchmarks
   ```

4. 테스트 결과를 확인합니다.

   예를 들어:

   ```text
   running 4 tests
   test mock::__construct_runtime_integrity_test::runtime_integrity_tests ... ok
   test tests::it_works_for_default_value ... ok
   test tests::correct_error_for_none_value ... ok
   test benchmarking::bench_do_something ... ok
   
   test result: ok. 4 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
   ```

## 런타임에 벤치마크 추가

팔렛에 벤치마크를 추가한 후, 팔렛을 포함하고 팔렛의 벤치마크를 포함하여 런타임을 업데이트해야 합니다.

1. 텍스트 편집기에서 런타임의 `Cargo.toml` 파일을 엽니다.

2. 팔렛을 런타임의 `[dependencies]` 목록에 추가합니다:

   ```toml
   pallet-mycustom = { default-features = false, path = "../pallets/pallet-mycustom"}
   ```

3. 런타임의 `[features]`를 업데이트하여 팔렛의 `runtime-benchmarks`를 포함합니다:

   ```toml
   [features]
   runtime-benchmarks = [
     ...
     'pallet-mycustom/runtime-benchmarks'
     ...
   ]
   ```

4. 런타임의 `std` 기능을 업데이트하여 팔렛을 포함합니다:

   ```toml
    std = [
     # -- snip --
     'pallet-mycustom/std'
   ]
   ```

5. 팔렛에 대한 configuration 트레이트를 런타임에 추가합니다.

6. 팔렛을 `construct_runtime!` 매크로에 추가합니다.

   팔렛을 런타임에 추가하는 방법에 대한 자세한 내용은 [팔렛을 런타임에 추가](/tutorials/build-application-logic/add-a-pallet) 또는 [팔렛 가져오기](/reference/how-to-guides/basics/import-a-pallet)를 참조하세요.

7. `runtime-benchmarks` 기능의 `define_benchmark!` 매크로에 팔렛을 추가합니다.

   ```rust
   #[cfg(feature = "runtime-benchmarks")]
   mod benches {
       define_benchmarks!(
         [frame_benchmarking, BaselineBench::<Runtime>]
         [pallet_assets, Assets]
         [pallet_babe, Babe]
         ...
         [pallet_mycustom, MyPallet]
         ...
       );
   }
   ```

## 벤치마크 실행

런타임을 업데이트한 후, `runtime-benchmarks` 기능을 활성화하여 프로젝트를 컴파일하고 팔렛에 대한 벤치마크 분석을 시작할 준비가 되었습니다.

1. 다음 명령을 실행하여 `runtime-benchmarks` 기능이 활성화된 상태로 프로젝트를 빌드합니다:

   ```bash
   cargo build --package node-template --release --features runtime-benchmarks
   ```

2. 노드 `benchmark pallet` 하위 명령의 명령줄 옵션을 검토합니다:

   ```bash
   ./target/release/node-template benchmark pallet --help
   ```
   
   `benchmark pallet` 하위 명령은 벤치마크를 자동화하는 데 도움이 되는 여러 명령줄 옵션을 지원합니다.
   예를 들어, `--steps` 및 `--repeat` 명령줄 옵션을 설정하여 다양한 값으로 여러 번 함수 호출을 실행할 수 있습니다.
   
3. 다음과 유사한 명령을 실행하여 팔렛에 대한 벤치마크를 시작합니다:

   ```bash
   ./target/release/node-template benchmark pallet \
    --chain dev \
    --pallet pallet_mycustom \
    --extrinsic '*' \
    --steps 20 \
    --repeat 10 \
    --output pallets/pallet-mycustom/src/weights.rs
   ```
   
   이 명령은 지정된 디렉토리에 `weights.rs` 파일을 생성합니다.
   해당 가중치를 사용하도록 팔렛을 구성하는 방법에 대한 자세한 내용은 [사용자 정의 가중치 사용](/reference/how-to-guides/weights/use-custom-weights/)를 참조하세요.

## 예제

다양한 유형의 함수에 대한 벤치마크를 알아보기 위해 `benchmarking.rs` 및 `weights.rs` 파일을 기존 팔렛에 사용할 수 있습니다.

- [예제 팔렛: 벤치마크](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/basic/src/benchmarking.rs)
- [예제 팔렛: 가중치](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/basic/src/weights.rs)
- [잔액 팔렛: 벤치마크](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/balances/src/benchmarking.rs)
- [잔액 팔렛: 가중치](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/balances/src/weights.rs)