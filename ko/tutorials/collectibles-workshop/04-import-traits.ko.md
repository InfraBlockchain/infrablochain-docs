---
title: 공유 및 사용자 정의 타입
description:
tutorial: 1
---

이제 팔렛의 기본 구조가 마련되었으므로, 응용 프로그램별 인터페이스를 구현하기 위해 코드를 사용자 정의하는 작업을 시작할 준비가 되었습니다. 이는 응용 프로그램 설계가 필요한 곳입니다.
FRAME은 모듈식이며 Rust 트레이트와 제네릭 타입의 유연성을 활용하기 때문에, 종종 `frame_system`, `frame_support` 또는 다른 미리 정의된 팔렛에서 이미 제공하는 인터페이스를 찾을 수 있으며 이를 직접 팔렛에 가져올 수 있습니다.

## 외부 인터페이스 가져오기 및 선언하기

`collectibles` 팔렛의 경우, 어떤 종류의 원장을 사용하여 어떤 사용자가 어떤 collectibles을 소유하고 있는지 추적하고, collectibles을 한 계정에서 다른 계정으로 이전하는 수단이 필요합니다.
또한 collectibles을 고유하게 만들기 위해 무작위 값을 통합하고 싶습니다. 다행히, 이러한 것들은 많은 문맥에서 유용한 인터페이스로 자주 사용되는 일반적인 사용 사례이므로, 이미 `frame_support` 라이브러리의 트레이트로 정의되어 있습니다.

`frame_support`에서 제공하는 트레이트는 다음과 같습니다:

- `Currency`: 계정 잔액, 이체 작업 및 Balance 타입에 대한 액세스를 활성화합니다.
- `Randomness`: 체인 상의 무작위 값을 액세스할 수 있도록 활성화합니다.

Rust 트레이트를 사용하면 특정 타입에 대한 기능을 정의하여 다른 타입과 공유할 수 있습니다.
이를 활용하기 위해 `frame_support` 모듈에서 `Currency` 및 `Randomness` 트레이트를 가져온 다음, 이를 타입으로 정의하고 `collectibles` 팔렛의 컨텍스트에서 어떻게 작동하는지 지정할 수 있습니다.

`Currency` 및 `Randomness` 트레이트 외에도, `collectibles` 팔렛은 단일 사용자가 소유할 수 있는 collectibles 자산의 최대 수를 지정하는 인터페이스가 필요합니다.
이 인터페이스에 대해 `collectibles` 팔렛은 `Get<u32>` 트레이트를 정의하여 `MaximumOwned` 상수를 지정합니다.

이러한 외부 인터페이스를 `collectibles` 팔렛의 구성에 포함시키면, `collectibles` 팔렛은 다음을 수행할 수 있게 됩니다:

- 사용자 계정 및 잔액에 액세스하고 조작합니다.
- 체인 상의 무작위 값을 생성합니다.
- 단일 사용자가 소유할 수 있는 collectibles 수를 제한합니다.

이러한 인터페이스를 가져오고 선언하려면 다음을 수행하세요:

1. 코드 편집기에서 `collectibles` 팔렛의 `src/lib.rs` 파일을 엽니다.

2. `frame_support` 모듈에서 `Currency` 및 `Randomness` 트레이트를 프로젝트에 가져옵니다.

    ```rust
    use frame_support::traits::{Currency, Randomness};
    ```

3. `collectibles` 팔렛의 `Config` 트레이트를 업데이트하여 `Currency`, `Randomness` 및 `Get<u32>` 트레이트를 선언합니다.

    ```rust
    #[pallet::config]
    pub trait Config: frame_system::Config {
        type Currency: Currency<Self::AccountId>;
        type CollectionRandomness: Randomness<Self::Hash, Self::BlockNumber>;

        #[pallet::constant]
        type MaximumOwned: Get<u32>;
    }
    ```
   
4. 다음 명령을 실행하여 프로그램이 컴파일되는지 확인합니다:
   
   ```bash
   cargo build --package collectibles
   ```
   
   현재는 사용되지 않는 코드에 대한 컴파일러 경고를 무시할 수 있습니다.

## 사용자 정의 타입 추가하기

Substrate은 Rust에서 사용 가능한 모든 기본 타입을 지원합니다. 예를 들어, bool, u8, u32 및 기타 일반적인 타입 등이 있습니다.
Substrate은 또한 Substrate에 특화된 여러 일반적인 사용자 정의 타입을 제공합니다. 예를 들어, `AccountId`, `BlockNumber`, `Hash`와 같은 타입은 가져온 `frame_system` 및 `frame_support` 모듈을 통해 사용할 수 있습니다.
`collectibles` 팔렛에서 이미 일부 외부 인터페이스를 가져왔습니다.
이제 collectibles을 설명하는 몇 가지 사용자 정의 속성을 정의할 수 있습니다.
이러한 사용자 정의 속성을 정의하기 위해 두 가지 사용자 정의 타입을 추가합니다:

- `Color` 속성의 가능한 변형을 나열하기 위한 enum 데이터 타입.
- `Collectible`의 속성을 그룹화하는 구조체(struct) 데이터 타입.

### enum 변형

```rust
 pub enum Color {
            Red,
            Yellow,
            Blue,
            Green
        }
```

`Collectible` 구조체는 다음과 같은 속성으로 구성됩니다:

- `unique_id`: 블록체인에서 각 collectibles이 고유한 개체임을 보장하기 위한 16바이트의 부호 없는 정수입니다.
- `price`: 가격이 설정된 경우 `Some(value)`를 반환하거나 판매용이 아님을 나타내는 경우 `None`을 반환하는 `Option`입니다.
- `color`: collectibles에 대한 사용자 정의 `Color` 타입의 변형입니다.
- `owner`: collectibles을 소유하는 계정을 식별합니다.

Currency 트레이트를 가져왔으므로, `collectibles` 팔렛에서 Currency 인터페이스의 `Balance` 및 `AccountId` 타입을 사용할 수도 있습니다.

`Balance` 타입에 대한 `BalanceOf`라는 타입 별칭을 만듭니다:

```rust
type BalanceOf<T> =
	<<T as Config>::Currency as Currency<<T as frame_system::Config>::AccountId>>::Balance;
```

`Collectible` 구조체에서 `BalanceOf` 및 `AccountId`를 사용합니다:

```rust
        pub struct Collectible<T: Config> {
            // 블록체인에서 고유 식별자를 나타내는 16바이트의 부호 없는 정수
            pub unique_id: [u8; 16],
            // 판매용이 아닌 경우 `None`으로 가정합니다.
            pub price: Option<BalanceOf<T>>,
            pub color: Color,
            pub owner: T::AccountId,
        }
```

다음 명령을 실행하여 코드가 컴파일되는지 확인할 수 있습니다:
   
```bash
cargo build --package collectibles
```

컴파일러 경고가 표시되지만 오류는 없어야 합니다.
그러나 이 시점에서 사용자 정의 타입을 사용하려고 하면 컴파일러가 오류를 표시합니다.
이는 아직 팔렛이 사용자 정의 타입에서 기대하는 모든 트레이트를 구현하지 않았기 때문입니다.

## 필요한 트레이트 구현하기

Substrate에서는 모든 데이터 타입에 대해 필요한 여러 트레이트가 있습니다.
예를 들어, 모든 데이터 타입은 데이터를 직렬화하고 역직렬화하여 효율적으로 네트워크를 통해 전송할 수 있도록 하는 `Encode` 및 `Decode` 트레이트를 구현해야 합니다.
다행히도, `#[derive]` 매크로를 사용하여 팔렛이 사용자 정의 타입에서 기대하는 모든 트레이트를 구현할 수 있습니다.
각 사용자 정의 타입에 `#[derive]` 매크로와 다음 트레이트를 추가하세요:

```rust
#[derive(Clone, Encode, Decode, PartialEq, Copy, RuntimeDebug, TypeInfo, MaxEncodedLen)]
```