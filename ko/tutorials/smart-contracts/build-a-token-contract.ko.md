---
title: 토큰 계약 작성하기
description: ink! 언어로 작성된 스마트 컨트랙트를 사용하여 ERC-20 토큰 공급을 구축합니다.
keywords:
  - erc20
  - 이더리움
  - ink
  - 토큰
  - 교환 가능한
---

이 튜토리얼에서는 ink! 언어를 사용하여 [ERC-20 토큰](https://eips.ethereum.org/EIPS/eip-20) 계약을 작성하는 방법을 설명합니다.

ERC-20 사양은 교환 가능한 토큰에 대한 공통 표준을 정의합니다.

토큰을 정의하는 속성에 대한 표준을 갖는 것은 사양을 따르는 개발자가 다른 제품 및 서비스와 상호 운용할 수 있는 애플리케이션을 구축할 수 있도록 돕습니다.

ERC-20 토큰 표준은 유일한 토큰 표준은 아니지만, 가장 일반적으로 사용되는 표준 중 하나입니다.

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- 스마트 컨트랙트, 토큰 및 암호화폐 개념과 용어에 대해 일반적으로 알고 있습니다.

- Rust를 설치하고 [설치](/install/)에 설명된대로 개발 환경을 설정했습니다.

- [첫 번째 계약 준비](/tutorials/smart-contracts/prepare-your-first-contract/)를 완료하고 로컬에 Substrate 계약 노드를 설치했습니다.

- [스마트 컨트랙트 개발](/tutorials/smart-contracts/develop-a-smart-contract/)을 완료하고 ink!가 어떻게 Rust 속성 매크로를 사용하여 스마트 컨트랙트를 작성하는지 알고 있습니다.

## 튜토리얼 목표

이 튜토리얼을 완료함으로써 다음 목표를 달성할 수 있습니다:

- ERC-20 표준에 정의된 기본 속성 및 인터페이스를 이해합니다.

- ERC-20 표준을 준수하는 토큰을 생성합니다.

- 계약 간에 토큰을 이전합니다.

- 승인 또는 제3자를 포함한 이전 활동의 경로를 처리합니다.

- 토큰 활동과 관련된 이벤트를 생성합니다.

## ERC-20 표준 기본 사항

[ERC-20 토큰 표준](https://eips.ethereum.org/EIPS/eip-20)은 이더리움 블록체인에서 실행되는 대부분의 스마트 컨트랙트에 대한 인터페이스를 정의합니다.

이러한 표준 인터페이스를 사용하면 개인들은 기존 스마트 컨트랙트 플랫폼 위에 자체 암호화폐를 배포할 수 있습니다.

표준을 검토하면 다음과 같은 핵심 기능이 정의되어 있는 것을 알 수 있습니다.

```javascript
// ----------------------------------------------------------------------------
// ERC Token Standard #20 Interface
// https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md
// ----------------------------------------------------------------------------

contract ERC20Interface {
    // 스토리지 Getter
    function totalSupply() public view returns (uint);
    function balanceOf(address tokenOwner) public view returns (uint balance);
    function allowance(address tokenOwner, address spender) public view returns (uint remaining);

    // 공개 함수
    function transfer(address to, uint tokens) public returns (bool success);
    function approve(address spender, uint tokens) public returns (bool success);
    function transferFrom(address from, address to, uint tokens) public returns (bool success);

    // 계약 이벤트
    event Transfer(address indexed from, address indexed to, uint tokens);
    event Approval(address indexed tokenOwner, address indexed spender, uint tokens);
}
```

사용자 잔액은 계정 주소에 매핑되며 인터페이스를 통해 사용자는 소유한 토큰을 이전하거나 제3자가 토큰을 이전할 수 있습니다.

가장 중요한 것은 스마트 컨트랙트 로직이 토큰이 의도치 않게 생성되거나 파괴되지 않도록 구현되어야 하며, 사용자의 자금이 악의적인 행위자로부터 보호되어야 한다는 것입니다.

모든 공개 함수가 호출이 성공했는지 여부만 나타내는 `bool`을 반환한다는 점에 유의하세요.

Rust에서는 이러한 함수들이 일반적으로 `Result`를 반환하는 것이 일반적입니다.

## 토큰 공급 생성

ERC-20 토큰을 처리하는 스마트 컨트랙트는 [값 저장을 위한 맵 사용](/tutorials/smart-contracts/use-maps-for-storing-values/)에서 값 저장을 위해 맵을 사용한 Incrementer 계약과 유사합니다.

이 튜토리얼에서는 ERC-20 계약이 배포될 때 계약 소유자와 연결된 계정에 모든 토큰이 예치된 고정 공급 토큰으로 구성됩니다.

계약 소유자는 그런 다음 토큰을 다른 사용자에게 배포할 수 있습니다.

이 튜토리얼에서 생성하는 간단한 ERC-20 계약은 토큰을 생성하고 배포하는 유일한 방법을 나타내는 것은 아닙니다.

그러나 이 ERC-20 계약은 다른 튜토리얼에서 배운 내용을 확장하고 더 견고한 스마트 컨트랙트를 구축하기 위해 ink! 언어를 사용하는 데 좋은 기반이 됩니다.

ERC-20 토큰 계약의 초기 스토리지는 다음과 같이 구성됩니다.

- 계약에 있는 토큰의 총 공급을 나타내는 `total_supply`.
- 각 계정의 개별 잔액을 나타내는 `balances`.

시작하려면 일부 템플릿 코드가 있는 새 프로젝트를 만들어 보겠습니다.

ERC-20 토큰 스마트 컨트랙트를 빌드하려면 다음 단계를 수행하세요:

1. 로컬 컴퓨터에서 터미널 셸을 엽니다(이미 열려 있다면 생략).

1. 다음 명령을 실행하여 `erc20`이라는 새 프로젝트를 생성합니다.

   ```bash
   cargo contract new erc20
   ```

1. 다음 명령을 실행하여 새 프로젝트 디렉토리로 이동합니다.

   ```bash
   cd erc20/
   ```

1. 텍스트 편집기에서 `lib.rs` 파일을 엽니다.

1. 다음 [ERC-20 템플릿](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/tutorials/smart-contracts/erc20-template.rs)의 기본 템플릿 소스 코드로 기본 템플릿 소스 코드를 바꿉니다.

1. 변경 사항을 `lib.rs` 파일에 저장한 다음 파일을 닫습니다.

1. 다음 명령을 실행하여 프로그램이 컴파일되고 단위 테스트를 통과하는지 확인합니다.

   ```bash
   cargo test
   ```

   다음과 유사한 출력이 표시되어야 테스트가 성공적으로 완료되었음을 나타냅니다.

   ```text
   running 2 tests
   test erc20::tests::new_works ... ok
   test erc20::tests::balance_works ... ok

   test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
   ```

1. 다음 명령을 실행하여 계약의 WebAssembly를 빌드할 수 있는지 확인합니다.

   ```bash
   cargo contract build
   ```

   프로그램이 성공적으로 컴파일되면 현재 상태에서 업로드하거나 계약에 기능을 추가할 준비가 된 것입니다.

## 계약 업로드 및 인스턴스화

새로 작성한 계약을 테스트하려면 [Contracts UI](https://contracts-ui.substrate.io)를 사용하여 계약을 업로드할 수 있습니다.

새로운 기능을 추가하기 전에 ERC-20 계약을 테스트하려면 다음을 수행하세요:

1. 로컬 계약 노드를 시작합니다. 필요한 경우 [첫 번째 계약 준비](/tutorials/smart-contracts/prepare-your-first-contract/) 튜토리얼을 참조하여 지침을 확인할 수 있습니다.

2. `new()` 생성자를 사용하여 계약을 인스턴스화합니다.

   ```bash
   cargo contract instantiate --constructor new --args 1_000_000 --suri //Alice --salt $(date +%s)
   ```

3. `total_supply()` 메시지를 호출하여 `total_supply`를 확인합니다. 체인 상태에서 읽기만 하려면 `--dry-run` 플래그를 추가하는 것을 잊지 마세요.

   ```bash
   cargo contract call --contract $INSTANTIATED_CONTRACT_ADDRESS \
       --message total_supply --suri //Alice --dry-run
   ```

4. `balance_of()`를 사용하여 초기 토큰 소유자인 Alice의 토큰 양을 확인합니다.

   ```bash
   cargo contract call --contract $INSTANTIATED_CONTRACT_ADDRESS \
       --message balance_of --args 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY \
       --suri //Alice --dry-run
   ```

   `cargo-contract`는 (아직) 해당 위치에 잘 알려진 키를 지원하지 않기 때문에 Alice의 전체 주소(`5Grw...utQY`)를 인수로 사용해야 함에 유의하세요.

   다른 `AccountId`를 제공하면 모든 토큰이 계약 소유자에 의해 소유되므로 잔액은 0이어야 합니다.

## 토큰 이전

이 시점에서 ERC-20 계약에는 계약 소유자가 계약의 `total_supply`에 대한 토큰을 소유하는 하나의 사용자 계정이 있습니다.

이 계약을 유용하게 만들려면 계약 소유자가 다른 계정으로 토큰을 이전할 수 있어야 합니다.

이 간단한 ERC-20 계약에서는 계약 호출자로서 토큰을 소유하는 계정에서 다른 사용자에게 토큰을 이전할 수 있는 공개 `transfer` 함수를 추가합니다.

공개 `transfer` 함수는 비공개 `transfer_from_to()` 함수를 호출합니다.

이 함수는 내부 함수이므로 인증 확인 없이 호출할 수 있습니다.

그러나 이전 로직은 `from` 계정이 이전할 토큰을 보낼 수 있는지 여부를 결정할 수 있어야 합니다.

`transfer_from_to()` 함수는 계약 호출자(`self.env().caller()`)를 `from` 계정으로 사용합니다.

이 문맥에서 `transfer_from_to()` 함수는 다음 작업을 수행합니다.

- `from` 및 `to` 계정의 현재 잔액을 가져옵니다.

- `from` 잔액이 보낼 `value` 토큰 수보다 작은지 확인합니다.

  ```rust
  let from_balance = self.balance_of(*from);
    if from_balance < value {
    return Err(Error::InsufficientBalance)
  }
  ```

- 이전하는 계정에서 `value`를 빼고 수신 계정에 `value`를 더합니다.

  ```rust
  self.balances.insert(&from, &(from_balance - value));
  let to_balance = self.balance_of(*to);
  self.balances.insert(&to, &(to_balance + value));
  ```

스마트 컨트랙트에 전송 함수를 추가하려면:

1. 로컬 컴퓨터에서 터미널 셸을 엽니다(이미 열려 있다면 생략).

1. `erc20` 프로젝트 디렉토리에 있는지 확인합니다.

1. 텍스트 편집기에서 `lib.rs` 파일을 엽니다.

1. `Error` 선언을 추가하여 계정의 잔액이 요청을 충족시킬 수 없는 경우 오류를 반환합니다.

   ```rust
   /// ERC-20 오류 유형을 지정합니다.
   #[derive(Debug, PartialEq, Eq, scale::Encode, scale::Decode)]
   #[cfg_attr(feature = "std", derive(scale_info::TypeInfo))]
   pub enum Error {
   /// 잔액이 요청을 충족시킬 수 없는 경우 반환합니다.
       InsufficientBalance,
   }
   ```

1. `InsufficientBalance` 오류를 반환하기 위해 `Result` 반환 유형을 추가합니다.

   ```rust
   /// ERC-20 결과 유형을 지정합니다.
   pub type Result<T> = core::result::Result<T, Error>;
   ```

1. 계약 호출자가 토큰을 다른 사용자에게 이전할 수 있도록 `transfer()` 공개 함수를 추가합니다.

   ```rust
   #[ink(message)]
   pub fn transfer(&mut self, to: AccountId, value: Balance) -> Result<()> {
       let from = self.env().caller();
       self.transfer_from_to(&from, &to, value)
   }
   ```

1. `from` 계정에서 수신 계정으로 토큰을 이전하는 비공개 `transfer_from_to()` 함수를 추가합니다.

   ```rust
   fn transfer_from_to(
       &mut self,
       from: &AccountId,
       to: &AccountId,
       value: Balance,
   ) -> Result<()> {
        let from_balance = self.balance_of(*from);
        if from_balance < value {
            return Err(Error::InsufficientBalance)
        }

        self.balances.insert(&from, &(from_balance - value));
        let to_balance = self.balance_of(*to);
        self.balances.insert(&to, &(to_balance + value));

        Ok(())
   }
   ```

1. 한 계정에서 다른 계정으로 토큰을 이전하는 테스트를 추가합니다.

   ```rust
   #[ink::test]
   fn transfer_works() {
       let mut contract = Erc20::new(100);
       assert_eq!(contract.balance_of(alice()), 100);
       assert!(contract.transfer(bob(), 10).is_ok());
       assert_eq!(contract.balance_of(bob()), 10);
       assert!(contract.transfer(bob(), 100).is_err());
   }
   ```

1. 다음 명령을 실행하여 프로그램이 컴파일되고 테스트 케이스를 통과하는지 확인합니다.

   ```bash
   cargo test
   ```

   다음과 유사한 출력이 표시되어야 테스트가 성공적으로 완료되었음을 나타냅니다.

   ```text
   running 3 tests
   test erc20::tests::new_works ... ok
   test erc20::tests::balance_works ... ok
   test erc20::tests::transfer_works ... ok

   test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
   ```

우리의 솔루션을 보려면 [여기](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/tutorials/smart-contracts/erc20-transfer.rs)를 확인할 수 있습니다.

## 이벤트 생성

ERC-20 토큰 표준은 트랜잭션을 제출할 때 직접 값을 반환할 수 없다고 명시합니다. 그러나 스마트 컨트랙트에서 이벤트가 발생했음을 어떤 방식으로든 알리고 싶을 수 있습니다. 예를 들어, 트랜잭션이 수행되었거나 전송이 승인되었음을 나타내기를 원할 수 있습니다. 이러한 종류의 신호를 보내기 위해 [이벤트](https://use.ink/basics/events)를 사용할 수 있습니다.

이벤트를 사용하여 어떤 종류의 데이터든지 통신할 수 있습니다. 이벤트 데이터를 정의하는 것은 `struct`를 정의하는 것과 유사합니다. 이벤트는 `#[ink(event)]` 속성을 사용하여 선언해야 합니다.

### 전송 이벤트 추가

이 튜토리얼에서는 완료된 전송 작업에 대한 정보를 제공하기 위해 `Transfer` 이벤트를 선언합니다. `Transfer` 이벤트에는 다음과 같은 정보가 포함됩니다.

- `Balance` 유형의 값.
- `from` 계정에 대한 옵션으로 래핑된 `AccountId` 변수.
- `to` 계정에 대한 옵션으로 래핑된 `AccountId` 변수.

이벤트 데이터에 빠르게 액세스하기 위해 _인덱싱된 필드_를 가질 수 있습니다. 이를 위해 해당 필드에 `#[ink(topic)]` 속성 태그를 사용할 수 있습니다.

`Transfer` 이벤트를 추가하려면:

1. 텍스트 편집기에서 `lib.rs` 파일을 엽니다.

1. `#[ink(event)]` 속성 매크로를 사용하여 이벤트를 선언합니다.

   ```rust
   #[ink(event)]
   pub struct Transfer {
       #[ink(topic)]
       from: Option<AccountId>,
       #[ink(topic)]
       to: Option<AccountId>,
       value: Balance,
     }
   ```

### 이벤트 발생

이벤트를 선언하고 이벤트에 포함된 정보를 정의했으므로 이벤트를 발생시키는 코드를 추가해야 합니다.

이를 위해 `self.env().emit_event()` 함수를 호출하여 이벤트 이름을 호출의 유일한 인수로 사용합니다.

이 ERC-20 계약에서는 전송이 발생할 때마다 `Transfer` 이벤트를 발생시키려고 합니다. 이는 다음 두 가지 위치에서 발생합니다.

- 계약을 초기화하는 `new` 호출 중.

- `transfer_from_to`가 호출될 때마다.

`from` 및 `to` 필드의 값은 `Option<AccountId>` 데이터 유형입니다. 그러나 토큰의 *초기 공급*에 대한 값은 다른 계정에서 가져오는 것이 아닙니다. 이 경우 Transfer 이벤트에는 `from` 값이 `None`입니다.

Transfer 이벤트를 발생시키려면:

1. 텍스트 편집기에서 `lib.rs` 파일을 엽니다.

1. `new()` 생성자에 `Transfer` 이벤트를 추가합니다.

   ```rust
   #[ink(constructor)]
   pub fn new(total_supply: Balance) -> Self {
       // -- 생략 --

       Self::env().emit_event(Transfer {
           from: None,
           to: Some(caller),
           value: total_supply,
       });

       // -- 생략 --
   }
   ```

1. `transfer_from_to()` 함수에 `Transfer` 이벤트를 추가합니다.

   ```rust
   fn transfer_from_to(
       &mut self,
       from: &AccountId,
       to: &AccountId,
       value: Balance,
   ) -> Result<()> {
      // -- 생략 --

       self.env().emit_event(Transfer {
           from: Some(*from),
           to: Some(*to),
           value,
       });

       // -- 생략 --
   }
   ```

   `value`는 `Option`에 저장되지 않으므로 `Some()`이 필요하지 않음에 유의하세요.


1. 다음 명령을 실행하여 프로그램이 컴파일되고 모든 테스트를 통과하는지 확인합니다.

   ```bash
   cargo test
   ```

   다음과 유사한 출력이 표시되어야 테스트가 성공적으로 완료되었음을 나타냅니다.

   ```text
   running 3 tests
   test erc20::tests::new_works ... ok
   test erc20::tests::balance_works ... ok
   test erc20::tests::transfer_works ... ok

   test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
   ```

우리의 솔루션을 보려면 [여기](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/tutorials/smart-contracts/erc20-event.rs)를 확인할 수 있습니다.

## 제3자 이전 활성화

ERC-20 토큰 계약은 이제 계정 간에 토큰을 이전하고 이러한 작업이 발생할 때마다 이벤트를 발생시킬 수 있습니다. 마지막 단계로 승인 및 `transfer_from` 함수를 추가하여 제3자 이전을 활성화할 수 있습니다.

한 계정이 다른 계정을 대신하여 토큰을 소비할 수 있도록 승인하는 것은 탈중앙화된 거래소를 지원하기 위한 제3자 이전을 지원하는 스마트 컨트랙트를 구축하는 데 도움이 됩니다.

계약에 직접 토큰을 이전하는 대신 소유한 토큰 중 일부를 다른 계정이 대신 거래할 수 있도록 승인할 수 있습니다.

트랜잭션이 실행되기를 기다리는 동안 필요한 경우 토큰을 제어하고 사용할 수 있습니다.

토큰에 액세스할 수 있는 여러 계약 또는 사용자를 승인할 수 있으므로 하나의 계약에서 최상의 트랜잭션을 제공하는 경우 토큰을 한 계약에서 다른 계약으로 이동시킬 필요가 없으므로 비용이 많이 들고 시간이 오래 걸리는 프로세스가 될 수 있습니다.

승인 및 이전을 안전하게 수행할 수 있도록 하기 위해 ERC-20 토큰 계약은 별도의 **승인** 및 **transfer From** 작업을 사용하는 두 단계 프로세스를 사용합니다.

### 승인 로직 추가

다른 계정이 토큰을 소비할 수 있도록 승인하는 것은 제3자 이전 프로세스의 첫 번째 단계입니다.

토큰 소유자로서 지정된 계정이 다른 계정을 대신하여 전송할 수 있는 토큰의 양을 지정할 수 있습니다.

모든 토큰을 승인할 필요는 없으며 승인된 계정이 전송할 수 있는 최대 수를 지정할 수 있습니다.

여러 번 `approve`를 호출하면 이전에 승인된 값을 새 값으로 덮어씁니다.
기본적으로 두 계정 간의 승인된 값은 `0`입니다.
계정의 토큰에 대한 액세스 권한을 취소하려면 `0` 값을 사용하여 `approve` 함수를 호출할 수 있습니다.

ERC-20 계약에 승인을 저장하려면 약간 더 복잡한 `Mapping` 키를 사용해야 합니다.

각 계정이 다른 계정에 대해 사용할 수 있는 다른 금액을 가질 수 있으므로 잔액 값을 매핑하는 키로 튜플을 사용해야 합니다.

예를 들어:

```rust
pub struct Erc20 {
 // -- 생략 --

 /// 소유자가 소비할 수 있는 잔액: (소유자, 승인자) -> 허용량
 allowances: Mapping<(AccountId, AccountId), Balance>,
}
```

튜플은 `(소유자, 승인자)`를 사용하여 `소유자` 계정에 대해 `승인자`가 `허용량`까지 액세스할 수 있는지 여부를 식별하는 데 사용됩니다.

스마트 컨트랙트에 승인 로직을 추가하려면:

1. 텍스트 편집기에서 `lib.rs` 파일을 엽니다.

1. `#[ink(event)]` 속성 매크로를 사용하여 `Approval` 이벤트를 선언합니다.

   ```rust
   #[ink(event)]
   pub struct Approval {
       #[ink(topic)]
       owner: AccountId,
       #[ink(topic)]
       spender: AccountId,
       value: Balance,
   }
   ```

1. `InsufficientAllowance` 오류를 나타내기 위해 `Error` 변형을 추가합니다.

   ```rust
   #[derive(Debug, PartialEq, Eq, scale::Encode, scale::Decode)]
   #[cfg_attr(feature = "std", derive(scale_info::TypeInfo))]
   pub enum Error {
       InsufficientBalance,
       InsufficientAllowance,
   }
   ```

1. 소유자 및 비소유자 조합에 대한 계정 잔액을 매핑하는 스토리지 선언에 `allowances` `Mapping`을 추가합니다.

   ```rust
   allowances: Mapping<(AccountId, AccountId), Balance>,
   ```

1. `new()` 생성자에서 `allowances` `Mapping`을 인스턴스화하고 추가합니다.

   ```rust
   #[ink(constructor)]
   pub fn new(total_supply: Balance) -> Self {
       // -- 생략 --

       let allowances = Mapping::default();

       Self {
           total_supply,
           balances,
           allowances
      }
   }
   ```

1. `approve()` 함수를 추가하여 `spender` 계정이 호출자의 계정에서 최대 `value`를 인출할 수 있도록 승인합니다.

   ```rust
   #[ink(message)]
   pub fn approve(&mut self, spender: AccountId, value: Balance) -> Result<()> {
       let owner = self.env().caller();
       self.allowances.insert((owner, spender), &value);

       self.env().emit_event(Approval {
         owner,
         spender,
         value,
       });

       Ok(())
   }
   ```

1. `owner` 계정이 `spender` 계정에서 인출할 수 있는 토큰 수를 반환하는 `allowance()` 함수를 추가합니다.

   ```rust
   #[ink(message)]
   pub fn allowance(&self, owner: AccountId, spender: AccountId) -> Balance {
       self.allowances.get((owner, spender)).unwrap_or_default()
   }
   ```

### 이전 로직 추가

다른 계정이 다른 계정을 대신하여 토큰을 이전할 수 있도록 승인한 후 승인된 사용자가 토큰을 이전할 수 있도록 `transfer_from` 함수를 만들어야 합니다.

`transfer_from` 함수는 대부분의 이전 로직을 처리하기 위해 비공개 `transfer_from_to` 함수를 호출합니다.

승인된 사용자가 토큰을 이전할 수 있도록 하려면 다음 요구 사항을 충족해야 합니다.

- `self.env().caller()` 계약 호출자가 `from` 계정에서 사용 가능한 토큰을 할당받았는지 확인합니다.

- `allowance`로 저장된 할당량이 전송할 값보다 큰지 확인합니다.

이러한 요구 사항이 충족되면 계약은 `allowance` 변수에 업데이트된 할당량을 삽입하고 지정된 `from` 및 `to` 계정을 사용하여 `transfer_from_to()` 함수를 호출합니다.

`transfer_from`을 호출할 때 `self.env().caller()` 및 `from` 계정은 현재 할당량을 조회하는 데 사용되지만 `transfer_from` 함수는 지정된 `from` 및 `to` 계정 사이에서 호출되므로 이러한 계정 변수가 세 개 있으며 올바르게 사용해야 합니다.

스마트 컨트랙트에 `transfer_from` 로직을 추가하려면:

1. 텍스트 편집기에서 `lib.rs` 파일을 엽니다.

1. `transfer_from()` 함수를 추가하여 `from` 계정에서 `to` 계정으로 `value` 개수의 토큰을 이전합니다.

   ```rust
   /// `from` 계정에서 `to` 계정으로 토큰을 이전합니다.
   #[ink(message)]
   pub fn transfer_from(
       &mut self,
       from: AccountId,
       to: AccountId,
       value: Balance,
   ) -> Result<()> {
       let caller = self.env().caller();
       let allowance = self.allowance(from, caller);
       if allowance < value {
           return Err(Error::InsufficientAllowance);
       }

       self.transfer_from_to(&from, &to, value)?;

       self.allowances.insert((from, caller), &(allowance - value));

       Ok(())
      }
   ```

1. `transfer_from()` 함수에 대한 테스트를 추가합니다.

   ```rust
   #[ink::test]
   fn transfer_from_works() {
       let mut contract = Erc20::new(100);
       assert_eq!(contract.balance_of(alice()), 100);
       let _ = contract.approve(alice(), 20);
       let _ = contract.transfer_from(alice(), bob(), 10);
       assert_eq!(contract.balance_of(bob()), 10);
   }
   ```

1. `allowance()` 함수에 대한 테스트를 추가합니다.

   ```rust
   #[ink::test]
   fn allowances_works() {
       let mut contract = Erc20::new(100);
       assert_eq!(contract.balance_of(alice()), 100);
       let _ = contract.approve(alice(), 200);
       assert_eq!(contract.allowance(alice(), alice()), 200);

       assert!(contract.transfer_from(alice(), bob(), 50).is_ok());
       assert_eq!(contract.balance_of(bob()), 50);
       assert_eq!(contract.allowance(alice(), alice()), 150);

       assert!(contract.transfer_from(alice(), bob(), 100).is_err());
       assert_eq!(contract.balance_of(bob()), 50);
       assert_eq!(contract.allowance(alice(), alice()), 150);
   }
   ```

1. 다음 명령을 실행하여 프로그램이 컴파일되고 모든 테스트를 통과하는지 확인합니다.

   ```bash
   cargo test
   ```

   다음과 유사한 출력이 표시되어야 테스트가 성공적으로 완료되었음을 나타냅니다.

   ```text
   running 5 tests
   test erc20::tests::new_works ... ok
   test erc20::tests::balance_works ... ok
   test erc20::tests::transfer_works ... ok
   test erc20::tests::transfer_from_works ... ok
   test erc20::tests::allowances_works ... ok

   test result: ok. 5 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
   ```

1. 다음 명령을 실행하여 계약의 WebAssembly를 빌드할 수 있는지 확인합니다.

   ```bash
   cargo contract build
   ```

   계약의 WebAssembly를 빌드한 후 `cargo-contract`를 사용하여 업로드하고 인스턴스화할 수 있습니다. [계약 업로드 및 인스턴스화](#계약-업로드-및-인스턴스화)를 참조하세요.

## 테스트 케이스 작성

이 튜토리얼에서는 `lib.rs` 파일에 간단한 단위 테스트를 추가했습니다.

기본 테스트 케이스는 지정된 입력 값과 반환된 결과를 확인하여 함수가 예상대로 작동하는지를 보여줍니다.

더 많은 테스트 케이스를 작성하여 코드의 품질을 개선할 수 있습니다.

예를 들어, 유효하지 않은 입력, 빈 값 또는 예상 범위를 벗어난 값에 대한 오류 처리를 테스트하는 테스트를 추가할 수 있습니다.

## 다음 단계

이 튜토리얼에서는 ink!를 사용하여 Substrate 블록체인에서 실행되는 간단한 ERC-20 토큰 스마트 컨트랙트를 작성하는 방법을 배웠습니다.

예를 들어, 이 튜토리얼에서는 다음과 같은 내용을 설명했습니다:

- 고정된 수의 토큰을 가진 계약을 생성하는 방법.

- 계약 소유자에서 다른 계정으로 토큰을 이전하는 방법.

- 스마트 컨트랙트에 테스트를 추가하는 방법.

- 제3자 이전을 활성화하는 방법.

이 튜토리얼에 대한 코드 예제는 [smart contracts](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/tutorials/smart-contracts/erc20-final.rs)의 자산에서 찾을 수 있습니다.

다음 주제에서 스마트 컨트랙트 개발에 대해 자세히 알아볼 수 있습니다:

- [값 저장을 위한 맵 사용](/tutorials/smart-contracts/use-maps-for-storing-values/)

- [스마트 컨트랙트 문제 해결](/tutorials/smart-contracts/troubleshoot-smart-contracts/)

- [ink! 문서](https://paritytech.github.io/ink-docs/)