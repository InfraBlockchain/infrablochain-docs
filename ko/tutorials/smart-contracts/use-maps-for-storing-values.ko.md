---
title: 값 저장을 위한 맵 사용
description: 맵을 사용하여 데이터를 키-값 쌍으로 저장하세요.
keywords:
---

[스마트 계약 개발](/tutorials/smart-contracts/develop-a-smart-contract/)에서는 하나의 숫자 값을 저장하고 검색하는 스마트 계약을 개발했습니다.

이 튜토리얼에서는 스마트 계약의 기능을 확장하여 각 사용자마다 하나의 숫자를 관리하는 방법을 설명합니다. 이를 위해 [`Mapping`](https://docs.rs/ink/4.0.0-beta.1/ink/storage/struct.Mapping.html) 타입을 사용합니다.

ink! 언어는 맵 타입을 제공하여 데이터를 키-값 쌍으로 저장할 수 있게 합니다. 예를 들어, 다음 코드는 사용자를 숫자에 매핑하는 방법을 보여줍니다:

```rust
// `Mapping` 타입을 가져옵니다
use ink::storage::Mapping;

#[ink(storage)]
pub struct MyContract {
  // AccountId를 u32에 매핑하는 맵을 저장합니다
  my_map: Mapping<AccountId, u32>,
}
```

`Mapping` 데이터 타입을 사용하면 각 키에 대해 고유한 저장 값 인스턴스를 저장할 수 있습니다.

이 튜토리얼에서는 각 `AccountId`가 하나의 `my_map`에 매핑되는 키를 나타냅니다.

각 사용자는 자신의 `AccountId`와 연결된 값을 저장, 증가, 검색할 수 있습니다.

## `Mapping` 초기화

첫 번째 단계는 `AccountId`와 저장된 값 사이의 매핑을 초기화하는 것입니다.

- 매핑 키와 해당 키에 매핑된 값을 지정하세요.

다음 예제는 `Mapping`을 초기화하고 값을 검색하는 방법을 보여줍니다:

```rust
#![cfg_attr(not(feature = "std"), no_std)]

#[ink::contract]
mod mycontract {
    use ink::storage::Mapping;

    #[ink(storage)]
    pub struct MyContract {
        // AccountId를 u32에 매핑하는 맵을 저장합니다
        my_map: Mapping<AccountId, u32>,
    }

    impl MyContract {
        #[ink(constructor)]
        pub fn new(count: u32) -> Self {
            let mut my_map = Mapping::default();
            let caller = Self::env().caller();
            my_map.insert(&caller, &count);

            Self { my_map }
        }

        // 호출자의 AccountId에 연결된 숫자를 가져옵니다 (존재하는 경우)
        #[ink(message)]
        pub fn get(&self) -> u32 {
            let caller = Self::env().caller();
            self.my_map.get(&caller).unwrap_or_default()
        }
    }
}
```

### 계약 호출자 식별

앞의 예제에서 `Self::env().caller()` 함수 호출을 볼 수 있습니다.

이 함수는 계약 로직 전체에서 사용할 수 있으며 항상 **계약 호출자**를 반환합니다.

계약 호출자는 **원래 호출자**와 다릅니다.

사용자가 계약에 접근하여 다른 계약을 호출하는 경우, 두 번째 계약의 `Self::env().caller()`는 첫 번째 계약의 주소가 됩니다.

### 계약 호출자 사용

계약 호출자를 사용할 수 있는 많은 시나리오가 있습니다.

예를 들어, `Self::env().caller()`를 사용하여 사용자가 자신의 값에만 액세스할 수 있는 액세스 제어 계층을 만들 수 있습니다.

또한 `Self::env().caller()`를 사용하여 계약 배포 중에 계약 소유자를 저장할 수 있습니다.

예를 들어:

```rust
#![cfg_attr(not(feature = "std"), no_std)]

#[ink::contract]
mod my_contract {

    #[ink(storage)]
    pub struct MyContract {
        // 계약 소유자를 저장합니다
        owner: AccountId,
    }

    impl MyContract {
        #[ink(constructor)]
        pub fn new() -> Self {
            Self {
                owner: Self::env().caller(),
            }
        }
        /* --snip-- */
    }
}
```

`owner` 식별자를 사용하여 계약 호출자를 저장했으므로 나중에 현재 계약 호출자가 계약의 소유자인지 확인하는 함수를 작성할 수 있습니다.

## 스마트 계약에 맵 추가

이제 `incrementer` 계약에 저장 맵을 추가할 준비가 되었습니다.

`incrementer` 계약에 저장 맵을 추가하려면 다음 단계를 수행하세요:

1. 필요한 경우 컴퓨터에서 터미널 셸을 엽니다.

1. `incrementer` 프로젝트 폴더에 있는지 확인하세요.

1. 텍스트 편집기에서 `lib.rs` 파일을 엽니다.

1. `Mapping` 타입을 가져옵니다.

   ```rust
   #[ink::contract
   mod incrementer {
       use ink::storage::Mapping;
   ```

1. `AccountId`에서 `i32` 데이터 타입으로 저장되는 `my_map`에 매핑 키를 추가합니다.

   ```rust
   pub struct Incrementer {
       value: i32,
       my_map: Mapping<AccountId, i32>,
   }
   ```

1. `new` 생성자에서 새로운 `Mapping`을 생성하고 이를 계약에 초기화합니다.

   ```rust
   #[ink(constructor)]
   pub fn new(init_value: i32) -> Self {
       let mut my_map = Mapping::default();
       let caller = Self::env().caller();
       my_map.insert(&caller, &0);

       Self {
           value: init_value,
           my_map,
       }
   }
   ```

1. `default` 생성자에 기존의 기본 `value`와 함께 새로운 기본 `Mapping`을 추가합니다.

    ```rust
    #[ink(constructor)]
    pub fn default() -> Self {
    Self {
            value: 0,
            my_map: Mapping::default(),
        }
    }
    ```

1. `Mapping` API의 `get()` 메서드를 사용하여 `my_map`을 읽고 `my_map`을 계약 호출자에 대해 반환하는 `get_mine()` 함수를 추가합니다.

   ```rust
   #[ink(message)]
   pub fn get_mine(&self) -> i32 {
        let caller = self.env().caller();
        self.my_map.get(&caller).unwrap_or_default()
   }
   ```

1. 초기 계정을 확인하는 새로운 테스트를 추가합니다.

   ```rust
    #[ink::test]
   fn my_map_works() {
       let contract = Incrementer::new(11);
       assert_eq!(contract.get(), 11);
       assert_eq!(contract.get_mine(), 0);
   }
   ```

1. 변경 사항을 저장하고 파일을 닫습니다.

1. 다음 명령을 실행하여 `test` 하위 명령과 `nightly` 도구 체인을 사용하여 작업을 테스트합니다.

   ```bash
   cargo test
   ```

   명령은 다음과 유사한 출력을 표시하여 테스트가 성공적으로 완료되었음을 나타냅니다.

   ```text
   running 3 tests
   test incrementer::tests::default_works ... ok
   test incrementer::tests::it_works ... ok
   test incrementer::tests::my_map_works ... ok

   test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
   ```

## 값 삽입, 업데이트 또는 제거

`Incrementer` 계약의 마지막 단계는 사용자가 자신의 값을 업데이트할 수 있도록 하는 것입니다.

스마트 계약에서 이 기능을 제공하기 위해 Mapping API를 호출할 수 있습니다.

`Mapping`은 저장 항목에 직접 액세스할 수 있습니다.

예를 들어, `Mapping::insert()`를 호출하여 기존 키에 대한 이전 값이 저장 항목에서 대체될 수 있습니다.

또한 `Mapping::get()`을 사용하여 저장소에서 값을 읽은 다음 `Mapping::insert()`로 값을 업데이트할 수 있습니다.

주어진 키에 기존 값이 없는 경우 `Mapping::get()`은 `None`을 반환합니다.

Mapping API는 저장소에 직접 액세스할 수 있으므로 `Mapping::remove()` 메서드를 사용하여 주어진 키의 값을 저장소에서 제거할 수 있습니다.

계약에 삽입 및 제거 기능을 추가하려면 다음 단계를 수행하세요:

1. 필요한 경우 컴퓨터에서 터미널 셸을 엽니다.

1. `incrementer` 프로젝트 폴더에 있는지 확인하세요.

1. 텍스트 편집기에서 `lib.rs` 파일을 엽니다.

1. `inc_mine()` 함수를 추가합니다. 이 함수는 계약 호출자가 `my_map` 저장 항목을 가져오고 증가된 `value`를 매핑에 삽입합니다.

   ```rust
   #[ink(message)]
   pub fn inc_mine(&mut self, by: i32) {
       let caller = self.env().caller();
       let my_value = self.get_mine();
       self.my_map.insert(caller, &(my_value + by));
   }
   ```

1. `remove_mine()` 함수를 추가합니다. 이 함수는 계약 호출자가 `my_map` 저장 항목을 저장소에서 제거할 수 있도록 합니다.

   ```rust
   #[ink(message)]
   pub fn remove_mine(&self) {
       let caller = self.env().caller();
       self.my_map.remove(&caller)
   }
   ```

1. `inc_mine()` 함수가 예상대로 작동하는지 확인하는 새로운 테스트를 추가합니다.

   ```rust
   #[ink::test]
   fn inc_mine_works() {
       let mut contract = Incrementer::new(11);
       assert_eq!(contract.get_mine(), 0);
       contract.inc_mine(5);
       assert_eq!(contract.get_mine(), 5);
       contract.inc_mine(5);
       assert_eq!(contract.get_mine(), 10);
   }
   ```

1. `remove_mine()` 함수가 예상대로 작동하는지 확인하는 새로운 테스트를 추가합니다.

   ```rust
   #[ink::test]
   fn remove_mine_works() {
       let mut contract = Incrementer::new(11);
       assert_eq!(contract.get_mine(), 0);
       contract.inc_mine(5);
       assert_eq!(contract.get_mine(), 5);
       contract.remove_mine();
       assert_eq!(contract.get_mine(), 0);
   }
   ```

1. `test` 하위 명령을 사용하여 작업을 확인합니다.

   ```bash
   cargo test
   ```

   명령은 다음과 유사한 출력을 표시하여 테스트가 성공적으로 완료되었음을 나타냅니다.

   ```text
   running 5 tests
   test incrementer::tests::default_works ... ok
   test incrementer::tests::it_works ... ok
   test incrementer::tests::remove_mine_works ... ok
   test incrementer::tests::inc_mine_works ... ok
   test incrementer::tests::my_map_works ... ok

   test result: ok. 5 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
   ```

## 다음 단계

이 튜토리얼에서는 `ink::storage::Mapping` 타입과 Mapping API를 스마트 계약에서 사용하는 방법을 배웠습니다. 예를 들어, 이 튜토리얼에서는 다음과 같은 내용을 설명했습니다:

- 키-값 쌍을 저장하기 위해 매핑을 초기화하는 방법.

- 스마트 계약에서 계약 호출자를 식별하고 사용하는 방법.

- 스마트 계약을 사용하여 사용자가 맵에 저장된 값을 삽입하고 제거하는 함수를 추가하는 방법.

이 튜토리얼의 최종 코드 예제는 [smart-contracts](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/tutorials/smart-contracts/incrementer-mapping.rs)의 자산에서 찾을 수 있습니다.

다음 주제에서 스마트 계약 개발에 대해 더 자세히 알아볼 수 있습니다:

- [ERC20 토큰 계약 빌드](/tutorials/smart-contracts/build-a-token-contract/)
- [스마트 계약 문제 해결](/tutorials/smart-contracts/troubleshoot-smart-contracts/)