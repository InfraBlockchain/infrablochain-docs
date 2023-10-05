---
title: 스마트 계약 개발하기
description: 값을 증가시키는 스마트 계약을 개발하세요.
keywords:
---

[첫 번째 계약 준비](/tutorials/smart-contracts/prepare-your-first-contract/)에서는 Substrate 기반 블록체인에 기본 프로젝트를 사용하여 스마트 계약을 구축하고 배포하는 기본 단계를 배웠습니다.

이 튜토리얼에서는 함수 호출할 때마다 카운터 값을 증가시키는 새로운 스마트 계약을 개발합니다.

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- 좋은 인터넷 연결과 로컬 컴퓨터의 셸 터미널에 액세스할 수 있어야 합니다.

- 소프트웨어 개발과 명령 줄 인터페이스 사용에 대해 일반적으로 알고 있어야 합니다.

- 블록체인과 스마트 계약 플랫폼에 대해 일반적으로 알고 있어야 합니다.

- [설치](/install/)에 설명된 대로 Rust를 설치하고 개발 환경을 설정했어야 합니다.

- [첫 번째 계약 준비](/tutorials/smart-contracts/prepare-your-first-contract/)를 완료하고 Substrate 계약 노드를 로컬에 설치했어야 합니다.

## 튜토리얼 목표

이 튜토리얼을 완료함으로써 다음 목표를 달성할 수 있습니다:

- 스마트 계약 템플릿 사용 방법 배우기

- 스마트 계약을 사용하여 간단한 값 저장하기

- 스마트 계약을 사용하여 저장된 값 증가 및 검색하기

- 스마트 계약에 공개 및 비공개 함수 추가하기

## 스마트 계약과 ink!

[첫 번째 계약 준비](/tutorials/smart-contracts/prepare-your-first-contract/)에서는 ink! 프로그래밍 언어에 대한 명령 줄 액세스를 위해 `cargo-contract` 패키지를 설치했습니다.

ink! 언어는 [임베디드 도메인 특화 언어](https://wiki.haskell.org/Embedded_domain_specific_language)입니다.

이 언어를 사용하면 Rust 프로그래밍 언어를 사용하여 WebAssembly 기반 스마트 계약을 작성할 수 있습니다.

이 언어는 특수화된 `#[ink(...)]` 속성 매크로를 사용하여 스마트 계약의 다른 부분이 어떤 것을 나타내는지 설명합니다. 이렇게 하면 Substrate 호환 WebAssembly 바이트코드로 변환할 수 있습니다.

## 새로운 스마트 계약 프로젝트 생성하기

Substrate에서 실행되는 스마트 계약은 프로젝트로 시작합니다.
`cargo contract` 명령을 사용하여 프로젝트를 생성합니다.

이 튜토리얼에서는 `incrementer` 스마트 계약을 위한 새 프로젝트를 생성합니다.

새 프로젝트를 생성하면 프로젝트 디렉토리에 새 프로젝트 디렉토리와 기본 스타터 파일(템플릿 파일이라고도 함)이 추가됩니다.

이 스타터 템플릿 파일을 수정하여 `incrementer` 프로젝트의 스마트 계약 로직을 구축합니다.

스마트 계약을 위한 새 프로젝트를 생성하려면 다음 단계를 따르세요:

1. 로컬 컴퓨터의 셸 터미널을 엽니다(이미 열려 있다면 건너뛰세요).

2. 다음 명령을 실행하여 `incrementer`라는 새 프로젝트를 생성합니다:

   ```bash
   cargo contract new incrementer
   ```

3. 다음 명령을 실행하여 새 프로젝트 디렉토리로 이동합니다:

   ```bash
   cd incrementer/
   ```

4. 텍스트 편집기에서 `lib.rs` 파일을 엽니다.

   기본적으로 템플릿 `lib.rs` 파일에는 `flipper` 스마트 계약의 소스 코드가 포함되어 있으며 `incrementer`로 이름이 변경되어 있습니다.

5. 기본 템플릿 소스 코드를 새로운 [incrementer](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/tutorials/smart-contracts/incrementer-template.rs) 소스 코드로 대체합니다.

6. `lib.rs` 파일의 변경 사항을 저장한 다음 파일을 닫습니다.

7. 다음 명령을 실행하여 프로그램이 컴파일되고 단위 테스트가 통과하는지 확인합니다:

   ```bash
   cargo test
   ```

   이 템플릿 코드는 단지 뼈대일 뿐이므로 경고 메시지는 무시해도 됩니다.
   다음과 유사한 출력이 표시되어야 테스트가 성공적으로 완료되었음을 나타냅니다:

   ```text
   running 1 test
   test incrementer::tests::default_works ... ok

   test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
   ```

8. 다음 명령을 실행하여 계약의 WebAssembly를 빌드할 수 있는지 확인합니다:

   ```bash
   cargo contract build
   ```

   프로그램이 성공적으로 컴파일되면 프로그래밍을 시작할 준비가 된 것입니다.

## 간단한 값 저장하기

`incrementer` 스마트 계약에 대한 일부 스타터 소스 코드가 준비되었으므로 새로운 기능을 추가할 수 있습니다.

예를 들어, 이 스마트 계약은 간단한 값의 저장을 필요로 합니다.

이 섹션의 코드는 ink! 언어의 기능을 설명하기 위한 것입니다.
이 튜토리얼의 나머지 부분에서 사용할 코드는 다음 섹션인 [스마트 계약 업데이트](#스마트-계약-업데이트)에서 시작됩니다.

`#[ink(storage)]` 속성 매크로를 사용하여 계약에 대한 간단한 값들을 저장할 수 있습니다:

```rust
#[ink(storage)]
pub struct MyContract {
  // bool 값 저장
  my_bool: bool,
  // 숫자 값 저장
  my_number: u32,
}
```

### 지원되는 타입

ink! 스마트 계약은 부울, 부호 없는 정수 및 부호 있는 정수, 문자열, 튜플, 배열 등을 포함한 대부분의 Rust 일반 데이터 타입을 지원합니다.

이러한 데이터 타입은 효율적인 네트워크 전송을 위해 [Parity scale codec](https://github.com/paritytech/parity-codec)를 사용하여 인코딩 및 디코딩됩니다.

스케일 코덱을 사용하여 인코딩 및 디코딩할 수 있는 일반적인 Rust 타입 외에도, ink! 언어는 `AccountId`, `Balance`, `Hash`와 같은 Substrate 특정 타입을 원시 타입처럼 지원합니다.

다음 코드는 이 계약에 `AccountId`와 `Balance`를 저장하는 방법을 보여줍니다:

```rust
#[ink::contract]
mod MyContract {

  // 우리의 구조체는 기본 ink! 타입을 사용합니다
  #[ink(storage)]
  pub struct MyContract {
    // 일부 AccountId 저장
    my_account: AccountId,
    // 일부 Balance 저장
    my_balance: Balance,
  }
/* --snip-- */
}
```

### 생성자

모든 ink! 스마트 계약은 최소한 하나의 생성자가 있어야 합니다. 생성자는 계약이 생성될 때 실행됩니다. 그러나 필요한 경우 스마트 계약에는 여러 개의 생성자가 있을 수 있습니다.

다음 코드는 여러 개의 생성자를 사용하는 방법을 보여줍니다:

```rust
#[ink::contract]
mod my_contract {

    #[ink(storage)]
    pub struct MyContract {
        number: u32,
    }

    impl MyContract {
        /// `u32` 값을 `init_value`로 초기화하는 생성자입니다.
        #[ink(constructor)]
        pub fn new(init_value: u32) -> Self {
            Self {
                number: init_value,
            }
        }

        /// `u32` 값을 `u32` 기본값(0)으로 초기화하는 생성자입니다.
        ///
        /// 생성자는 다른 생성자로 위임할 수 있습니다.
        #[ink(constructor)]
        pub fn default() -> Self {
            Self {
                number: Default::default(),
            }
        }
    /* --snip-- */
    }
}
```

## 스마트 계약 업데이트

간단한 값 저장, 데이터 타입 선언 및 생성자 사용에 대해 배웠으므로 스마트 계약 소스 코드를 업데이트하여 다음을 구현할 수 있습니다:

- `value`라는 저장 값 생성 및 `i32` 데이터 타입으로 설정하기

- `Incrementer` 생성자 생성 및 `value`를 `init_value`로 설정하기

- `default`라는 두 번째 생성자 함수 생성하기. 이 함수는 입력이 없으며 `value`를 `0`으로 설정하는 새로운 `Incrementer`를 생성합니다.

스마트 계약을 업데이트하려면 다음 단계를 따르세요:

1. 텍스트 편집기에서 `lib.rs` 파일을 엽니다.

2. `Storage Declaration` 주석을 `value`라는 저장 항목으로 대체하고 데이터 타입을 `i32`로 설정합니다.

   ```rust
   #[ink(storage)]
   pub struct Incrementer {
       value: i32,
   }
   ```

3. `Incrementer` 생성자를 수정하여 `value`를 `init_value`로 설정합니다.

   ```rust
   impl Incrementer {
        #[ink(constructor)]
        pub fn new(init_value: i32) -> Self {
            Self { value: init_value }
        }
   }
   ```

4. `default`라는 두 번째 생성자 함수를 추가하여 `value`가 `0`으로 설정된 새로운 `Incrementer`를 생성합니다.

   ```rust
   #[ink(constructor)]
   pub fn default() -> Self {
       Self {
           value: 0,
       }
   }
   ```

5. 변경 사항을 저장한 다음 파일을 닫습니다.

6. `test` 하위 명령을 다시 실행하면 테스트가 실패하는 것을 볼 수 있습니다. 이는 `get` 함수를 업데이트하고 구현한 변경 사항과 일치하도록 테스트를 수정해야 하기 때문입니다. 다음 섹션에서 이를 수행합니다.

## 저장 값 가져오는 함수 추가하기

이제 저장 값이 생성되고 초기화되었으므로 공개 및 비공개 함수를 사용하여 이를 조작할 수 있습니다. 이 튜토리얼에서는 저장 값 가져오기 위한 공개 함수(메시지)를 추가합니다.

모든 공개 함수는 `#[ink(message)]` 속성 매크로를 사용해야 함에 유의하세요.

스마트 계약에 공개 함수를 추가하려면 다음 단계를 따르세요:

1. 텍스트 편집기에서 `lib.rs` 파일을 엽니다.

2. `get` 공개 함수를 업데이트하여 `i32` 데이터 타입을 가진 `value` 저장 항목의 데이터를 반환하도록 합니다.

   ```rust
   #[ink(message)]
   pub fn get(&self) -> i32 {
       self.value
   }
   ```

   이 함수는 계약 저장소에서 _읽기_ 만 하므로 `&self` 매개변수를 사용하여 계약 함수와 저장 항목에 액세스합니다.

   이 함수는 `value` 저장 항목의 상태를 변경하는 것을 허용하지 않습니다.

   함수의 마지막 표현식에 세미콜론(;)이 없으면 Rust는 해당 표현식을 반환 값으로 처리합니다.

3. 비공개 `default_works` 함수의 `Test Your Contract` 주석을 `get` 함수를 테스트하는 코드로 대체합니다.

   ```rust
   #[ink::test]
   fn default_works() {
       let contract = Incrementer::default();
       assert_eq!(contract.get(), 0);
   }
   ```

4. 변경 사항을 저장한 다음 파일을 닫습니다.

5. `test` 하위 명령을 사용하여 작업을 확인합니다. 여전히 실패하는 것을 볼 수 있습니다. 이는 `it_works` 테스트를 업데이트하고 `value` 저장 항목을 증가시키는 새로운 공개 함수를 추가해야 하기 때문입니다.

   ```bash
   cargo test
   ```

## 저장 값 수정하는 함수 추가하기

현재 스마트 계약은 사용자가 저장소를 수정할 수 없습니다. 사용자가 저장소 항목을 _수정_할 수 있도록 하려면 `value`를 명시적으로 가변 변수로 표시해야 합니다.

저장 값 증가를 위한 함수를 추가하려면 다음 단계를 따르세요:

1. 텍스트 편집기에서 `lib.rs` 파일을 엽니다.

2. `inc`라는 새로운 공개 함수를 추가하여 `by` 매개변수를 사용하여 `value`를 증가시킵니다. `by` 매개변수의 데이터 타입은 `i32`입니다.

   ```rust
   #[ink(message)]
   pub fn inc(&mut self, by: i32) {
       self.value += by;
   }
   ```

3. 이 함수를 확인하기 위해 새로운 테스트를 소스 코드에 추가합니다.

   ```rust
   #[ink::test]
   fn it_works() {
       let mut contract = Incrementer::new(42);
       assert_eq!(contract.get(), 42);
       contract.inc(5);
       assert_eq!(contract.get(), 47);
       contract.inc(-50);
       assert_eq!(contract.get(), -3);
   }
   ```

4. 변경 사항을 저장한 다음 파일을 닫습니다.

5. `test` 하위 명령을 사용하여 작업을 확인합니다:

   ```bash
   cargo test
   ```

   다음과 유사한 출력이 표시되어야 테스트가 성공적으로 완료되었음을 나타냅니다:

   ```text
   running 2 tests
   test incrementer::tests::it_works ... ok
   test incrementer::tests::default_works ... ok

   test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
   ```

### 계약의 WebAssembly 빌드하기

`incrementer` 계약을 테스트한 후 이 프로젝트를 WebAssembly로 컴파일할 준비가 되었습니다.

이 스마트 계약의 WebAssembly를 빌드하려면 다음 단계를 따르세요:

1. 필요한 경우 컴퓨터에서 터미널 셸을 엽니다.

2. `incrementer` 프로젝트 폴더에 있는지 확인합니다.

3. 다음 명령을 실행하여 `incrementer` 스마트 계약을 컴파일합니다:

   ```bash
   cargo contract build
   ```

   다음과 유사한 출력이 표시됩니다:

   ```text
   Your contract artifacts are ready. You can find them in:
   /Users/dev-docs/incrementer/target/ink

   - incrementer.contract (code + metadata)
   - incrementer.wasm (the contract's code)
   - incrementer.json (the contract's metadata)
   ```

## 스마트 계약 배포 및 테스트하기

로컬에 [`substrate-contracts-node`](https://github.com/paritytech/substrate-contracts-node) 노드가 설치되어 있다면 스마트 계약을 위해 로컬 블록체인 노드를 시작할 수 있습니다.

그런 다음 `cargo-contract`를 사용하여 스마트 계약을 배포하고 테스트할 수 있습니다.

로컬 노드에 배포하기 위해 다음 단계를 따르세요:

1. 필요한 경우 컴퓨터에서 터미널 셸을 엽니다.

2. 다음 명령을 실행하여 개발 모드에서 계약 노드를 시작합니다:

   ```bash
   substrate-contracts-node --log info,runtime::contracts=debug 2>&1
   ```

3. 계약을 업로드하고 인스턴스화합니다.

   ```bash
   cargo contract instantiate --constructor default --suri //Alice --salt $(date +%s)
   ```

   ```text
   Dry-running default (skip with --skip-dry-run)
       Success! Gas required estimated at Weight(ref_time: 321759143, proof_size: 0)
   Confirm transaction details: (skip with --skip-confirm)
    Constructor default
           Args
      Gas limit Weight(ref_time: 321759143, proof_size: 0)
   Submit? (Y/n):
      Events
       Event Balances ➜ Withdraw
         who: 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
         amount: 2.953956313mUNIT
       ... snip ...
       Event System ➜ ExtrinsicSuccess
         dispatch_info: DispatchInfo { weight: Weight { ref_time: 2772097885, proof_size: 0 }, class: Normal, pays_fee: Yes }

   Code hash 0x71ddef2422fdb8358b503d5ef122c088a2dc6486dd460c37b01d672a8d319959
   Contract 5Cf6wFEyZnqvNJaKVxnWswefo7uT4jVsgzWKh8b78GLDV6kN
   ```

4. 값을 증가시킵니다.

   ```bash
   cargo contract call --contract $INSTANTIATED_CONTRACT_ADDRESS --message inc --args 42 --suri //Alice
   ```

  ```text
   Dry-running inc (skip with --skip-dry-run)
    Success! Gas required estimated at Weight(ref_time: 8013742080, proof_size: 262144)
   Confirm transaction details: (skip with --skip-confirm)
        Message inc
           Args 42
      Gas limit Weight(ref_time: 8013742080, proof_size: 262144)
   Submit? (Y/n):
      Events
       Event Balances ➜ Withdraw
         who: 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
         amount: 98.97416μUNIT
       Event Contracts ➜ Called
         caller: 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
         contract: 5Cf6wFEyZnqvNJaKVxnWswefo7uT4jVsgzWKh8b78GLDV6kN
       Event TransactionPayment ➜ TransactionFeePaid
         who: 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
         actual_fee: 98.97416μUNIT
         tip: 0UNIT
       Event System ➜ ExtrinsicSuccess
         dispatch_info: DispatchInfo { weight: Weight { ref_time: 1383927346, proof_size: 13255 }, class: Normal, pays_fee: Yes }
  ```

5. 현재 값을 가져옵니다.

   ```bash
   cargo contract call --contract 5Cf6wFEyZnqvNJaKVxnWswefo7uT4jVsgzWKh8b78GLDV6kN --message get --suri //Alice --dry-run
   ```

   ```text
   Result Success!
   Reverted false
     Data Tuple(Tuple { ident: Some("Ok"), values: [Int(42)] })
   ```

계약에서 읽은 `value`가 이전 단계와 일치하는 `42`임을 확인할 수 있습니다!

## 다음 단계

이 튜토리얼에서는 ink! 프로그래밍 언어와 속성 매크로를 사용하여 스마트 계약을 생성하는 기본 기술 몇 가지를 배웠습니다.

예를 들어, 이 튜토리얼에서는 다음과 같은 내용을 설명했습니다:

- 새 스마트 계약 프로젝트에 저장 항목을 추가하고 데이터 타입을 지정하며 생성자를 구현하는 방법

- 스마트 계약에 함수를 추가하는 방법

- 스마트 계약에 테스트를 추가하는 방법

- `cargo-contract`를 사용하여 계약을 업로드하고 인스턴스화하는 방법

이 튜토리얼의 최종 코드 예제는 [smart-contracts](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/tutorials/smart-contracts/incrementer-basics.rs)의 에셋에서 찾을 수 있습니다.

다음 주제에서 스마트 계약 개발에 대해 자세히 알아볼 수 있습니다:

- [값 저장을 위한 맵 사용](/tutorials/smart-contracts/use-maps-for-storing-values/)

- [ERC20 토큰 계약 개발](/tutorials/smart-contracts/build-a-token-contract/)

- [스마트 계약 문제 해결](/tutorials/smart-contracts/troubleshoot-smart-contracts/)