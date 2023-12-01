---
title: 사용자 정의 가중치 사용하기
description:
keywords:
  - 가중치
  - 벤치마킹
  - 런타임
---

여기에서는 팔레트의 extrinsic을 벤치마킹한 결과에서 가중치를 사용하는 방법을 배우게 됩니다.
FRAME의 벤치마킹 도구를 실행하여 생성된 `weights.rs`라는 파일이 팔레트의 디렉토리에 있다고 가정합니다.

## 목표

팔레트의 extrinsic에서 [FRAME의 벤치마킹 도구](https://paritytech.github.io/substrate/master/frame_benchmarking/macro.benchmarks.html)에서 생성된 가중치를 사용합니다.

## 사용 사례

벤치마킹을 통해 생성된 정확한 가중치를 팔레트의 extrinsic에 사용하는 경우입니다.

## 단계

### 1. 팔레트의 `weight.rs` 파일에 `WeightInfo` trait 정의하기

벤치마킹 도구에서 생성된 `weights.rs` 파일을 팔레트에서 사용하기 위해 (Substrate의 [`pallet-example` weights 예시](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/basic/src/weights.rs) 참조),
해당 함수에 쉽게 접근할 수 있도록 trait를 정의합니다.
예를 들어:

`pallets/example/src/weights.rs`

```rust
pub trait WeightInfo {
  fn example(len: usize,) -> Weight;
}
```

### 2. 팔레트에 `WeightInfo` 포함하기

팔레트의 `lib.rs`에서 맨 위에 `pub mod weights;`를 선언하고
`pub use crate::weights::WeightInfo;`를 사용하여 `WeightInfo` trait를 팔레트에 노출시킵니다. 그런 다음,
`Config` trait에 `WeightInfo` 타입을 추가합니다:

`pallets/example/src/lib.rs`

```rust
pub mod weights;
pub use weights::*;

// -- 생략 --

pub trait Config: frame_system::Config {
    // -- 생략 --

    /// 런타임 가중치에 대한 정보입니다.
    type WeightInfo: WeightInfo;
}
```

### 3. 사용자 정의 가중치 선언하기

각각의 dispatchable에 대해 적절한 가중치 라인을 도입하여
구성된 `WeightInfo` 타입을 사용하여 가중치를 결정합니다. 예를 들어, `T::WeightInfo::example`는 `example` extrinsic에 대해 `weights.rs`에서 반환된 가중치 함수입니다:

`pallets/example/src/lib.rs`

```rust
#[pallet::weight(T::WeightInfo::example(x.len()))]
fn example(origin: OriginFor<T>, arg: Vec<u32>) -> DispatchResult {
  // -- 생략 --
}
```

### 4. `Config` trait impls에서 `WeightInfo` 타입 정의하기

팔레트의 테스트 모의 `mock.rs`와 Substrate의 `node`의 런타임 모두에서 `WeightInfo` trait를 정의해야 합니다.

`mock.rs`의 팔레트의 `Config` trait impl에서 테스트 중에 가중치를 제외하기 위해 다음 줄을 추가합니다.

`pallets/example/src/mock.rs` 또는 `pallets/example/src/test.rs`:

```rust
impl Config for TestRuntime {
  // -- 생략 --
  type WeightInfo = ();
}
```

`runtime/src/lib.rs`의 팔레트의 `Config` trait impl에서 런타임에서 가중치를 포함하기 위해 다음 줄을 추가합니다.

```rust
impl pallet_example::Config for Runtime {
  // -- 생략 --
  type WeightInfo = pallet_example::weights::SubstrateWeight<Runtime>;
}
```

노드를 다시 컴파일하면, extrinsic은 이제 `pallets/example/src/weights.rs`에 정의된 사용자 정의 가중치를 사용하게 됩니다.

## 예시

- [Example 팔레트의 벤치마킹](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/basic/src/benchmarking.rs)
- [Example 팔레트의 가중치](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/basic/src/weights.rs)

## 관련 자료

- [벤치마킹](/ko/infrablockchain/tutorials/test/benchmark.md)
- [수수료 계산](./calculate-fees.md)
- [벤치마크 추가](./add-benchmarks.md)