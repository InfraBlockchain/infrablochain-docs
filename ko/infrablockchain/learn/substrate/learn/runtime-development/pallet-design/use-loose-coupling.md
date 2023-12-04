---
title: 느슨한 팔레트 결합 사용하기
description:
keywords:
---

이 가이드는 **느슨한 결합**이라는 기술을 사용하여 한 팔레트에서 다른 팔레트로 함수 또는 타입을 재사용하는 방법을 보여줍니다.

느슨한 결합은 현재 팔레트 내에서 외부 팔레트에 정의된 일부 로직을 재사용할 수 있게 해줍니다.
이 가이드에서는 팔레트의 `Config` 트레이트 내의 트레이트 바운드를 사용하여 외부 팔레트에 선언된 타입을 재사용하는 방법을 설명합니다.
이 예제에서는 현재 팔레트가 [`frame_support`](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/currency/trait.Currency.html) 팔레트의 `Currency` 트레이트에 대한 메소드를 사용합니다.

## 단계 미리보기

1. workspace 매니페스트를 구성하여 외부 팔레트를 포함시킵니다.
2. 사용하려는 트레이트를 가져옵니다.
3. `Config` 트레이트에 타입을 추가합니다.
4. 런타임에 구현 내용을 제공합니다.

## workspace 매니페스트 구성하기

현재 작업 디렉토리의 팔레트가 다른 팔레트에서 코드를 재사용하려면, 외부 팔레트는 팔레트에 가져온 패키지 종속성 목록에 포함되어야 합니다.
따라서 프로젝트의 `Cargo.toml` 파일에 있는 매니페스트는 가져올 팔레트를 지정해야 합니다.
이 예제에서는 `frame-support` 팔레트에서 `Currency` 트레이트 정보를 재사용하므로, 매니페스트의 `dependencies` 및 `features` 섹션에 `frame-support`가 포함되어 있는지 확인해야 합니다.

workspace 매니페스트를 구성하려면:

1. 컴퓨터에서 터미널 쉘을 열고 프로젝트의 루트 디렉토리로 이동합니다.
   
2. 텍스트 편집기에서 매니페스트인 `Cargo.toml` 파일을 엽니다.
   
3. 느슨하게 결합할 팔레트를 종속성에 추가합니다.
   
   예를 들어:
   
   ```text
   [dependencies]
   frame-support = { default-features = false, git = "https://github.com/paritytech/substrate.git", branch = "polkadot-v1.0.0"}
   ```
   
   모든 팔레트가 서로 호환되도록 하기 위해 모든 팔레트에 동일한 브랜치와 버전 정보를 사용해야 합니다.
   다른 브랜치의 팔레트를 사용하면 컴파일러 오류가 발생할 수 있습니다.
   이 예제는 다른 팔레트가 `branch = "polkadot-v1.0.0"`를 사용하는 경우 `frame-support` 팔레트를 `Cargo.toml` 파일에 추가하는 방법을 보여줍니다.
   
   빌드 프로세스는 표준 바이너리와 WebAssembly 대상을 모두 컴파일하므로, 팔레트의 기능에 `frame-support/std`를 포함해야 합니다.

1. 팔레트의 `std` 기능에 `frame-support/std`를 추가합니다.
   
   예를 들어:
   
   ```text
   [features]
   default = ["std"]
   std = [
     "frame-support/std",
   ]
   ```

1. 변경 사항을 저장하고 `Cargo.toml` 파일을 닫습니다.

## 사용하려는 트레이트 가져오기

이 예제에서는 현재 팔레트가 [`frame_support`](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/currency/trait.Currency.html) 팔레트의 `Currency` 트레이트를 사용하여 현재 팔레트에서 해당 메소드에 액세스할 수 있도록 합니다.

다른 팔레트에서 트레이트를 가져오려면:

1. 컴퓨터에서 터미널 쉘을 열고 프로젝트의 루트 디렉토리로 이동합니다.
   
2. 텍스트 편집기에서 현재 팔레트의 `src/lib.rs` 파일을 엽니다.
   
3. 다음 줄을 추가하여 `Currency` 트레이트를 가져옵니다:
      
   ```rust
   use frame_support::traits::Currency;
   ```

4. 변경 사항을 저장합니다.

## `Config` 트레이트에 타입 추가하기

다음 단계는 현재 팔레트에서 외부 팔레트에서 액세스하려는 타입에 바운드된 타입을 생성하는 것입니다.

팔레트의 `Config` 트레이트를 업데이트하려면:

1. 컴퓨터에서 터미널 쉘을 열고 프로젝트의 루트 디렉토리로 이동합니다.
   
2. 텍스트 편집기에서 현재 팔레트의 `src/lib.rs` 파일을 엽니다.
   
3. 외부 팔레트에서 액세스하려는 타입에 바운드된 타입을 생성합니다:

   예를 들어:
   
   ```rust
   pub trait Config: frame_system::Config {
       // --snip--

       /// 우리의 느슨하게 결합된 팔레트 `my-pallet`에 액세스하는 타입
       type LocalCurrency: Currency<Self::AccountId>;
   }
   ```

5. 생성한 타입을 사용하여 느슨하게 결합된 팔레트의 트레이트가 제공하는 메소드를 사용합니다.
   
   예를 들어:

   ```rust
   // `my-pallet`에서 게터를 사용합니다.
   let total_balance = T::LocalCurrency::total_issuance();
   ```
   
   이 예제에서 [`total_issuance`](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/currency/trait.Currency.html#tymethod.total_issuance)는 `Currency` 트레이트가 `frame_support` 팔레트에서 노출하는 메소드입니다.

1. 변경 사항을 저장하고 프로젝트의 `src/lib.rs` 파일을 닫습니다.

## 런타임에서 구현 제공하기

프로젝트에서 업데이트를 완료한 후, `Currency` 트레이트에 기반한 `LocalCurrency`를 포함하는 런타임 구성을 구현할 준비가 되었습니다.

팔레트의 런타임 구성을 업데이트하려면:

1. 컴퓨터에서 터미널 쉘을 열고 노드 템플릿의 루트 디렉토리로 이동합니다.
   
2. 텍스트 편집기에서 `runtime/src/lib.rs` 파일을 엽니다.

1. 팔레트의 런타임 구성을 추가하여 `LocalCurrency` 타입을 `Balances` 팔레트에 정의된 구현을 사용하도록 지정합니다.
   
   ```rust
   impl my_pallet::Config for Runtime {
     type LocalCurrency = Balances;
   }
   ```

1. `construct_runtime!` 매크로 내에서 `Balances` 정의를 확인합니다.
   
   ```rust
   construct_runtime! (
     pub enum Runtime where
     Block = Block,
     NodeBlock = opaque::Block,
     UncheckedExtrinsic = UncheckedExtrinsic
     {
       Balances: pallet_balances,
     }
   )
   ```

   이 예제에서는 팔레트가 느슨하게 결합된 `frame-support` 팔레트에서 메소드에 액세스할 수 있도록 [`pallet_balances`](https://paritytech.github.io/substrate/master/pallet_balances/index.html#implementations-1) 팔레트의 `Currency` 트레이트 구현을 상속받을 수 있습니다.
   
   기본적으로 `construct_runtime!` 매크로는 매크로 정의에 나열된 모든 팔레트의 모든 팔레트 속성을 포함합니다. 

## 예제

- [Democracy 팔레트](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/democracy/src/lib.rs#L298-L335)의 [`EnsureOrigin`](https://paritytech.github.io/substrate/master/frame_support/traits/trait.EnsureOrigin.html) 트레이트
- [Identity 팔레트](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/identity/src/lib.rs#L149-L151)의 [가중치 메소드](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/identity/src/weights.rs#L46-L64)
- [Grandpa 팔레트](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/grandpa/src/lib.rs#L106)의 [`KeyOwnerProofSystem`](https://paritytech.github.io/substrate/master/frame_support/traits/trait.KeyOwnerProofSystem.html)

## 자원

- [팔레트 결합](../../frame/pallet-coupling.md)
- [타이트한 팔레트 결합 사용하기](./use-tight-coupling.md)