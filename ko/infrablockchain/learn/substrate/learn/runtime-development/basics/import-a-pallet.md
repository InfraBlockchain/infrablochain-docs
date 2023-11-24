---
title: 팔레트 가져오기
description:
keywords:
  - 기초
  - 초보자
  - 런타임
---

이 가이드는 런타임에 로컬 및 외부 팔레트를 빠르게 통합하는 방법을 보여줍니다.
더 자세한 단계별 지침은 [런타임에 팔레트 추가하기](../../../tutorials/build-application-logic/add-a-pallet.md)를 참조하세요.

이 가이드에서는 다음을 설명합니다:

- 런타임에 이벤트와 호출을 구현하는 사용자 정의 로컬 팔레트를 포함하는 방법.
- `Crates.io`에서 외부 팔레트를 포함하는 방법.

## 로컬 팔레트 생성

1. `pallet_something`이라는 로컬 팔레트를 생성하세요.

1. `/runtime/src/lib.rs`에 다음을 추가하여 이 팔레트를 가져옵니다:

   ```rust
   // 팔레트를 가져옵니다.
   pub use pallet_something;
   ```

1. 팔레트의 런타임 구현을 구성하세요.
   로컬 팔레트가 런타임에 노출되는 `Event`와 `Call` 타입만 가지고 있다고 가정합니다. `/runtime/src/lib.rs`에 다음을 추가하세요:

   ```rust
   // 팔레트를 구성합니다.
   impl pallet_something::Config for Runtime {
   	type RuntimeEvent = RuntimeEvent;
   	type RuntimeCall = RuntimeCall;
   }
   ```

1. [`construct_runtime` 매크로](../../frame/frame-macros.md)에 팔레트를 선언하세요:

   ```rust
   construct_runtime!(
   	pub enum Runtime where
   	Block = Block,
   	NodeBlock = opaque::Block,
   	UncheckedExtrinsic = UncheckedExtrinsic
   	{
   		/* --snip-- */
   		Something: pallet_something,
   		/* --snip-- */
   	}
   );
   ```

1. `/runtime/Cargo.toml`을 업데이트하세요.

   `/runtime/Cargo.toml`에서 팔레트를 `std`의 로컬 종속성으로 포함하고 `runtime-benchmarks`를 추가하세요.
   예를 들어:

   ```toml
   # --snip--
   pallet-something = { default-features = false,   path = '../pallets/something'
   version = '3.0.0'
   # --snip--
   [features]
   default = ['std']
   runtime-benchmarks = [
     # --snip--
     'pallet-something/runtime-benchmarks',
   ]
   std = [
     'pallet-something/std',
     # --snip--
   ]
   ```

## 외부 팔레트 가져오기

외부 팔레트를 추가하려면 로컬 팔레트와 유사한 방법을 사용하지만, 팔레트가 노출하는 모든 타입을 포함해야 합니다.
또한 관련된 매개변수 타입과 상수를 포함해야 합니다.
매개변수와 상수를 선언하는 예는 [`pallet_timestamp`](https://paritytech.github.io/substrate/master/pallet_timestamp/index.html)를 참조하세요.

다음은 팔레트가 [crates.parity.io](https://crates.parity.io/)에 호스팅되어 있는 경우 `/runtime/Cargo.toml` 종속성에 외부 팔레트를 추가하는 예입니다:

```toml
[dependencies]
pallet-external = {default-features = false, git = "https://github.com/paritytech/polkadot-sdk.git", version = "4.0.0-dev"}

# --snip--
runtime-benchmarks = [
  /* --snip */
  'pallet-external/runtime-benchmarks',
]
std = [
  'pallet-external/std',
  # --snip--
]
```

## 예시

- [템플릿 팔레트](https://paritytech.github.io/substrate/master/pallet_template/index.html)
- [타임스탬프 팔레트](https://paritytech.github.io/substrate/master/pallet_timestamp/index.html)

## 관련 자료

- [타임스탬프 팔레트 관련 타입](https://paritytech.github.io/substrate/master/pallet_timestamp/index.html)
- [FRAME `pallet-timestamp`](https://crates.io/crates/pallet-timestamp)<!-- markdown-link-check-disable-line -->
- [단위 테스트](../../../../../tutorials/test/unit-testing.md)