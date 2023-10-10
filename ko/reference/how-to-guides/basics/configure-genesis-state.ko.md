---
title: 제네시스 상태 구성하기
description:
keywords:
  - 기본
  - 초보자
  - 런타임
---

제네시스 구성은 계정, 잔액, 사용자 정의 팔레트의 제네시스 등과 같은 저장소 항목의 초기 상태를 정의합니다.
이 가이드에서는 다음과 같은 저장소 항목을 가진 팔레트의 제네시스 상태를 구성하는 방법을 보여줍니다:

- 단일 `StorageValue` 저장소 항목에 대한 `SingleValue<T>` 타입.
- 열거 가능한 항목을 가진 단일 키 `StorageMap` 저장소 항목에 대한 `AccountMap<T>` 타입.

## 구성 단계 미리보기

1. 팔레트에 저장소 항목 추가하기.
2. 팔레트에 제네시스 구성 매크로 추가하기.
3. 체인 스펙에서 초기 값 설정하기.

## 팔레트에 저장소 항목 추가하기

1. 텍스트 편집기에서 팔레트의 `src//lib.rs` 파일을 엽니다.

2. `StorageValue` 저장소 항목을 추가합니다.

   예시:

   ```rust
   #[pallet::storage]
   #[pallet::getter(fn something)]
   pub type SingleValue<T: Config> = StorageValue<
      _, 
      T::Balance
   >;
   ```

3. 열거 가능한 항목을 가진 `StorageMap` 저장소 항목을 추가합니다.

   예시:

   ```rust
   #[pallet::storage]
   #[pallet::getter(fn accounts)]
   pub type AccountMap<T: Config> = StorageMap<
      _, 
      Blake2_128Concat, 
      T::AccountId, 
      T::Balance
   >;
   ```

## 제네시스 구성 매크로 추가하기

제네시스 구성 매크로는 저장소 항목 이후에 작성되어야 합니다.

1. `#[pallet::genesis_config]` 속성 매크로를 추가하고 초기화할 저장소 항목에 대한 `GenesisConfig` 구조체를 정의합니다.

   ```rust
   #[pallet::genesis_config]
   pub struct GenesisConfig<T: Config> {
   	pub single_value: T::Balance,
   	pub account_map: Vec<(T::AccountId, T::Balance)>,
   }
   ```

2. `GenesisConfig` 구조체의 기본값을 설정합니다.

   ```rust
   #[cfg(feature = "std")]
   impl<T: Config> Default for GenesisConfig<T> {
   	fn default() -> Self {
   		Self { single_value: Default::default(), account_map: Default::default() }
   	}
   }
   ```

3. `#[pallet::genesis_build]` 속성 매크로를 추가하고 [`GenesisBuild`](https://paritytech.github.io/substrate/master/frame_support/traits/trait.GenesisBuild.html) 트레이트를 구현합니다.

   ```rust
   #[pallet::genesis_build]
   impl<T: Config> GenesisBuild<T> for GenesisConfig<T> {
   	fn build(&self) {
   		<SingleValue<T>>::put(&self.single_value);
   		for (a, b) in &self.account_map {
   			<AccountMap<T>>::insert(a, b);
   		}
   	}
   }
   ```

   `#[pallet::genesis_build]` 매크로를 사용하면 `GenesisConfig` 구조체가 저장소에 어떻게 배치되는지를 정의하는 논리를 실행할 수 있습니다.

## 초기 값 설정하기

팔레트에서 저장소 항목과 제네시스 구성을 구성한 후, 체인의 제네시스 상태에 설정할 값을 지정할 수 있습니다.
이 예시에서는 런타임의 `construct_runtime!` 매크로가 팔레트 이름으로 `PalletSomething`을, 팔레트의 경로로 `pallet_something`을 참조한다고 가정합니다.

```rust
construct_runtime!(
   pub struct Runtime
   where
      Block = Block,
      NodeBlock = opaque::Block,
      UncheckedExtrinsic = UncheckedExtrinsic,
   {
      PalletSomething: pallet_something,
   }
)
```

1. 텍스트 편집기에서 `node/chain_spec.rs` 파일을 엽니다.

2. `chain_spec.rs` 파일 상단에 가져온 `use node_template_runtime::BalanceConfig;`가 있는지 확인합니다.

3. `<SingleValue<T>>`에 저장될 `T::Balance` 타입의 상수 값을 생성합니다 (`testnet_genesis` 메서드 내부).

   ```rust
   const VALUE: Balance = 235813;
   ```

4. `testnet_genesis` 메서드 내부에서 `<AccountMap<T>>`을 초기화하기 위해 사용할 계정 벡터를 생성합니다.

   ```rust
   let accounts_to_map: Vec<AccountId> =
   	vec![
   		get_account_id_from_seed::<sr25519::Public>("Alice"),
   		get_account_id_from_seed::<sr25519::Public>("Bob"),
   		get_account_id_from_seed::<sr25519::Public>("Charlie"),
   	];
   ```

5. `testnet_genesis` 함수 내에서 `GenesisConfig` 절에 팔레트를 추가합니다.

   `runtime/src/lib.rs`의 `construct_runtime!` 내에서 팔레트 이름을 소문자로 표기하는 것이 관례입니다.
   예를 들어 `CamelCase`로 선언된 팔레트의 경우, `testnet_genesis` 함수에서는 `camel_case`로 참조하는 것이 관례입니다.

   이 예시 팔레트의 코드는 다음과 같습니다:

   ```rust
   pallet_something: PalletSomethingConfig {
   	single_value: VALUE,
   	account_map: accounts_to_map.iter().cloned().map(|x| (x, VALUE)).collect(),
   }
   ```

   이 샘플 코드는 `accounts_to_map`의 각 계정을 `VALUE`와 같은 금액으로 매핑합니다.
   이 패턴은 Balances 팔레트의 `GenesisConfig`와 매우 유사합니다.

## 예시

- [노드 템플릿 'chain_spec.rs'](https://github.com/substrate-developer-hub/substrate-node-template/blob/master/node/src/chain_spec.rs)
- [예시 팔레트](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/basic/src/lib.rs)