---
title: 제네시스 구성
description:
keywords:
---

어떤 블록체인에서 생성된 첫 번째 블록을 제네시스 블록이라고 합니다.
이 블록과 관련된 해시는 첫 번째 블록 이후에 생성된 모든 블록의 최상위 부모입니다.

Substrate 노드 템플릿은 기본적으로 일부 팔렛의 제네시스 구성(초기 상태)을 제공합니다.
런타임에 사용자 정의 로직을 추가할 때(예: 미리 정의된 또는 사용자 정의 팔렛 추가) 다른 스토리지 항목을 포함하거나 다른 초기값을 설정하려는 경우 제네시스 구성을 수정하고 싶을 수 있습니다.

[체인 스펙](/build/chain-spec/)에서 배운 대로, 노드를 시작하는 데 사용하는 체인 스펙은 해당 노드의 제네시스 구성을 결정합니다.
그러나 체인 스펙은 노드를 시작할 때 초기화되는 스토리지 항목을 생성하지 않습니다.
대신, 스토리지 항목은 [런타임 스토리지](/build/runtime-storage/)에서 설명한대로 런타임에 포함된 팔렛에 정의됩니다.

런타임을 위한 스토리지 항목을 생성한 후, 제네시스 구성에 초기 상태로 설정할지 여부를 선택할 수 있으며, 이를 위해 Substrate는 두 가지 특수화된 FRAME 속성 매크로를 제공합니다.

체인의 제네시스 구성에 스토리지 항목을 초기 상태로 설정하기 위해 사용할 수 있는 매크로는 다음과 같습니다:

- `#[pallet::genesis_config]` 매크로는 `GenesisConfig` 데이터 타입을 정의하고 스토리지 항목을 초기화합니다.
- `#[pallet::genesis_build]` 매크로는 제네시스 구성을 빌드합니다.

이러한 매크로는 체인 스펙과 함께 사용되어 런타임의 초기 상태를 정의하는 데 사용됩니다.

## 간단한 스토리지 값 구성

다음 예제는 `pallet_template`의 제네시스 구성에 하나의 스토리지 값을 추가하는 방법을 보여줍니다.
기본적으로 `pallet_template`에는 제네시스 블록에서 초기화되지 않은 하나의 스토리지 항목이 있습니다.
이 예제에서는 체인의 제네시스 구성으로 스토리지 값을 초기값으로 설정하기 위해 `#[pallet::genesis_config]` 및 `#[pallet::genesis_build]` 매크로를 사용하는 방법을 보여줍니다.

### 팔렛에서 매크로 구성

`pallet_template`의 스토리지 항목을 초기화하려면 다음 단계를 따르세요:

1. 새로운 터미널 쉘을 열고 노드 템플릿의 루트 디렉토리로 이동합니다.
2. 텍스트 편집기에서 `pallets/template/src/lib.rs` 파일을 엽니다.
3. `#[pallet::genesis_config]` 매크로를 추가하고 `GenesisConfig` 스토리지 항목으로 `something` 스토리지 값을 추가합니다.
   예를 들어, 다음 매크로를 파일에 추가합니다:
   ```rust
   // Test Genesis Configuration
   #[pallet::genesis_config]
   #[derive(Default)]
    pub struct GenesisConfig {
          pub something: u32,
   }
   ```

이 예제에서 `#[derive(Default)]` 매크로는 `frame_support::traits::GenesisBuild`의 트레이트 바운드 요구 사항을 충족시키기 위해 필요합니다.

4. `#[pallet::genesis_build]` 매크로를 추가합니다:

 ```rust
 #[pallet::genesis_build]
 impl<T: Config> GenesisBuild<T> for GenesisConfig {
    fn build(&self) { }
 }
```

이 예제에서는 `build` 함수에 대한 특별한 처리가 없습니다.

5. 변경 사항을 저장하고 파일을 닫습니다.

6. 다음 명령을 실행하여 팔렛이 컴파일되는지 확인합니다:

   ```bash
   cargo build --package pallet-template
   ```

### 체인 스펙 구성

제네시스 블록에서 스토리지 항목을 초기값으로 설정할 수 있도록 팔렛을 구성했으므로, 체인 스펙에서 해당 스토리지 항목에 대한 초기값을 설정할 수 있습니다.

1. 텍스트 편집기에서 `node/src/chain_spec.rs` 파일을 엽니다.
1. `node_template_runtime`에 `TemplateModuleConfig`를 추가합니다.

   예를 들어:

   ```rust
   use node_template_runtime::{
      AccountId, AuraConfig, BalancesConfig, RuntimeGenesisConfig, GrandpaConfig, Signature, SudoConfig, SystemConfig, TemplateModuleConfig, WASM_BINARY,
   };
   ```

1. `GenesisConfig`를 찾아 `something` 스토리지 항목에 대한 초기값을 설정합니다.

   예를 들어, `node/src/chain_spec.rs` 파일에서 다음과 같이 설정합니다:

   ```rust
   -> GenesisConfig {
        GenesisConfig {
                system: SystemConfig {
                        // Add Wasm runtime to storage.
                        code: wasm_binary.to_vec(),
                },

                template_module: TemplateModuleConfig {
                       something: 221u32,
                },

                transaction_payment: Default::default(),
        }
      }
   ```

## 런타임에 제네시스 구성 추가

`#[pallet::genesis_config]` 매크로를 사용하여 각 팔렛에 `GenesisConfig`을 추가한 후, 런타임에서 제네시스 블록의 스토리지 항목을 초기화할 수 있도록 각 팔렛의 `Config` 트레이트를 런타임에 포함해야 합니다.

런타임 구성을 구성하는 팔렛들의 `GenesisConfig` 타입은 해당 런타임을 위한 단일 `RuntimeGenesisConfig` 타입으로 집계됩니다.

집계된 `RuntimeGenesisConfig`는 [`BuildStorage`](https://paritytech.github.io/substrate/master/sp_runtime/trait.BuildStorage.html) 트레이트를 구현하여 런타임의 모든 초기 스토리지 항목을 빌드합니다.
예를 들어, 노드 템플릿 런타임은 기본적으로 다음 팔렛들을 위해 `RuntimeGenesisConfig`를 지정합니다:

- [System 팔렛](#system-pallet)
- [Aura 팔렛](#aura-pallet)
- [Grandpa 팔렛](#grandpa-pallet)
- [Balance 팔렛](#balances-pallet)
- [TransactionPayment 팔렛](#transactionpayment-pallet)
- [Sudo 팔렛](#sudo-pallet)

### 시스템 팔렛

```rust
#[pallet::genesis_config]
	pub struct GenesisConfig {
		#[serde(with = "sp_core::bytes")]
		pub code: Vec<u8>,
	}
```

### Aura 팔렛

```rust
#[pallet::genesis_config]
	pub struct GenesisConfig<T: Config> {
		pub authorities: Vec<T::AuthorityId>,
	}
```

### Grandpa 팔렛

```rust
#[pallet::genesis_config]
	pub struct GenesisConfig {
		pub authorities: AuthorityList,
	}
```

### Balances 팔렛

```rust
#[pallet::genesis_config]
	pub struct GenesisConfig<T: Config<I>, I: 'static = ()> {
		pub balances: Vec<(T::AccountId, T::Balance)>,
	}
```

### TransactionPayment 팔렛

```rust
#[pallet::genesis_config]
	pub struct GenesisConfig {
		pub multiplier: Multiplier,
	}
```

### Sudo 팔렛

```rust
#[pallet::genesis_config]
	pub struct GenesisConfig<T: Config> {
		/// The `AccountId` of the sudo key.
		pub key: Option<T::AccountId>,
	}
```

이러한 팔렛들은 `#[pallet::genesis_config]` 매크로를 사용하여 `GenesisConfig`와 `Config` 트레이트를 런타임에서 정의하므로, 이들은 런타임을 위한 [`node_template_runtime::RuntimeGenesisConfig`](https://paritytech.github.io/substrate/master/node_template_runtime/struct.RuntimeGenesisConfig.html) 구조체로 집계됩니다:

```rust
pub struct RuntimeGenesisConfig {
    pub system: SystemConfig,
    pub aura: AuraConfig,
    pub grandpa: GrandpaConfig,
    pub balances: BalancesConfig,
    pub transaction_payment: TransactionPaymentConfig,
    pub sudo: SudoConfig,
}
```

최종적으로, `RuntimeGenesisConfig`는 [`ChainSpec`](https://paritytech.github.io/substrate/master/sc_chain_spec/trait.ChainSpec.html) 트레이트를 통해 노출됩니다.

Substrate의 제네시스 스토리지 구성에 대한 더 완전한 예는 Substrate 코드 베이스와 함께 제공되는 [체인 스펙](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/bin/node/cli/src/chain_spec.rs)을 참조하십시오.

## 팔렛 내에서 스토리지 항목 초기화

[`#[pallet::genesis_build]`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#genesis-build-palletgenesis_build-optional) 매크로를 사용하여 팔렛 자체에서 스토리지 항목의 초기 상태를 정의할 수 있습니다.
팔렛 내에서 제네시스 구성을 정의하면 팔렛의 비공개 함수에 액세스할 수 있습니다.

다음 예제는 스토리지 항목의 초기값을 설정하기 위해 `#[pallet::genesis_config]`와 `#[pallet::genesis_build]`를 사용하는 방법을 보여줍니다.
이 예제에서는 두 개의 스토리지 항목이 있습니다:

- 멤버 계정 식별자 목록.
- 목록에서 멤버를 대표하는 특정 계정 식별자.

이 예제의 매크로와 데이터 타입은 `my_pallet/src/lib.rs` 파일에 정의되어 있습니다:

```rust
#[pallet::genesis_config]
struct GenesisConfig {
    members: Vec<T::AccountId>,
    prime: T::AccountId,
}

#[pallet::genesis_build]
impl<T: Config> GenesisBuild<T> for GenesisConfig {
    fn build(&self) {
        Pallet::<T>::initialize_members(&self.members);
        SomeStorageItem::<T>::put(self.prime);
    }
}
```

제네시스 구성은 `node/src/chain_spec.rs` 파일에서 정의됩니다:

```rust
GenesisConfig {
    my_pallet: MyPalletConfig {
        members: LIST_OF_IDS,
        prime: ID,
    },
}
```

또한 `genesis_build` 매크로를 사용하여 특정 스토리지 항목에 바인딩되지 않은 `GenesisConfig` 속성을 정의할 수도 있습니다.
이는 팔렛 내에서 여러 스토리지 항목을 설정하는 비공개 도우미 함수를 호출하거나 팔렛 내에 포함된 다른 팔렛에서 정의된 함수를 호출하는 경우에 유용할 수 있습니다.
예를 들어, `intitialize_members`라는 가상의 비공개 함수를 사용하는 경우, 코드는 다음과 같을 수 있습니다:

`my_pallet/src/lib.rs`:

```rust
#[pallet::genesis_config]
struct GenesisConfig {
    members: Vec<T::AccountId>,
    prime: T::AccountId,
}

#[pallet::genesis_build]
impl<T: Config> GenesisBuild<T> for GenesisConfig {
    fn build(&self) {
        Pallet::<T>::initialize_members(&config.members);
        SomeStorageItem::<T>::put(self.prime);
    }
}
```

`chain_spec.rs`:

```rust
GenesisConfig {
    my_pallet: MyPalletConfig {
        members: LIST_OF_IDS,
        prime: ID,
    },
}
```