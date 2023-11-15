---
title: 스토리지 구조체 (struct) 생성하기
description:
keywords:
  - 팔레트 디자인
  - 중간
  - 런타임
---

유사한 그룹으로 구성된 스토리지 항목의 구조체를 생성하는 것은 그들을 추적하는 질서정연한 방법입니다.
이렇게 그룹화된 경우 개별적인 `StorageValue` 항목을 따로 유지하는 것보다 참조하기가 더 쉽습니다.
이는 테스트와 제네시스 구성을 더 쉽게 만들 수 있습니다.

이 가이드에서는 [`on_initialize`](https://paritytech.github.io/substrate/master/frame_support/traits/trait.Hooks.html#method.on_initialize)에서 사용되는 구조체를 보유하는 [`StorageValue`](https://paritytech.github.io/substrate/master/frame_support/storage/trait.StorageValue.html) 스토리지 항목을 생성하는 방법을 보여줍니다.
이 구조체는 다음과 같은 작업을 수행합니다:

- 초기 금액 (`issuance`)을 추적합니다.
- 해당 금액을 받는 계정 (`minter`)을 추적합니다.
- 일부 금액을 소모할 수 있는 계정 (`burner`)을 추적합니다.
- `on_initialize`에서 (부분적으로) 사용됩니다.

시작하기 전에

구조체를 작성할 팔레트가 작동하는지 확인하세요.
이미 작업 중인 팔레트가 없다면 [템플릿 팔레트](https://github.com/substrate-developer-hub/substrate-node-template/blob/main/pallets/template/src/lib.rs)를 사용하세요.

1. 구조체 생성하기

   `MetaData`라는 이름의 구조체를 생성하고 다른 유형을 선언하세요:

   ```rust
   #[derive(Clone, Encode, Decode, Eq, PartialEq, RuntimeDebug, Default)]
   pub struct MetaData<AccountId, Balance> {
   	issuance: Balance,
   	minter: AccountId,
   	burner: AccountId,
   }
   ```

1. 구조체를 스토리지 항목으로 선언하기

   `StorageValue`를 사용하여 구조체를 새로운 단일 스토리지 항목으로 선언하세요:

   ```rust
   #[pallet::storage]
   #[pallet::getter(fn meta_data)]
   	pub(super) type MetaDataStore<T: Config> = StorageValue<_, MetaData<T::AccountId, T::Balance>, ValueQuery>;
   ```

1. `GenesisConfig` 구성하기

   `MetaData` 구조체에서 값을 초기화하기 위해 `#[pallet::genesis_config]` 속성을 사용하세요.

   ```rust
   // `admin`를 `T::AccountId` 타입으로 선언합니다.
   #[pallet::genesis_config]
   pub struct GenesisConfig<T: Config> {
   	pub admin: T::AccountId,
   	}
   // 기본값을 지정합니다.
   #[cfg(feature = "std")]
   impl<T: Config> Default for GenesisConfig<T> {
   	fn default() -> Self {
   		Self {
   			admin: Default::default(),
   			}
   		}
   	}
   ```

   이 `admin` 변수는 `node/chainspec.rs` 파일 내의 `fn testnet_genesis` 안에서 사용된 변수와 일치해야 합니다.

1. `GenesisBuild` 구성하기

   `#[pallet::genesis_build]` 속성을 사용하여 `admin`을 사용하여 구조체의 값을 초기화하세요. 이때 `T::AccountId` 타입의 값을 초기화합니다:

   ```rust
   #[pallet::genesis_build]
   impl<T: Config> GenesisBuild<T> for GenesisConfig<T> {
   	fn build(&self) {
   		MetaDataStore::<T>::put(MetaData {
   			issuance: Zero::zero(),
   			minter: self.admin.clone(),
   			burner: self.admin.clone(),
   		});
   	}
   }
   ```

1. `on_initialize()`에서 구조체 사용하기

   `MetaData`의 `issuance` 필드에 초기화할 금액을 할당하세요:

   ```rust
   fn on_initialize(_n: T::BlockNumber) -> Weight {
   	// StorageValue 구조체에 대한 별칭 생성하기
   	let mut meta = MetaDataStore::<T>::get();

   	// `issuance` 필드에 값을 추가합니다.
   	let value: T::Balance = 50u8.into();
   	meta.issuance = meta.issuance.saturating_add(value);

   	// 스토리지의 `minter` 계정에 금액 추가하기
   	Accounts::<T>::mutate(&meta.minter, |bal| {
   		*bal = bal.saturating_add(value);
   	});
   }
   ```

   `on_initialize` 함수는 지정된 항목의 값을 체인이 시작될 때 스토리지에 기록합니다.

예제

- [`reward-coin`](https://github.com/substrate-developer-hub/substrate-how-to-guides/blob/d3602a66d66be5b013f2e3330081ea4e0d6dd978/example-code/template-node/pallets/reward-coin/src/lib.rs#L24-L29) 예제 팔레트

자원

- [`Default::default()`](https://paritytech.github.io/substrate/master/sp_std/default/trait.Default.html)