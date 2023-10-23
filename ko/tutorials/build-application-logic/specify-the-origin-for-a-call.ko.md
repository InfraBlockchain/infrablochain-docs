---
title: 함수 호출의 출처 지정하기
description: 함수 호출의 출처로 사용할 계정을 지정하는 방법을 보여줍니다.
keywords:
  - FRAME
  - 런타임
  - 출처
---

[런타임에 팔렛 추가하기](/tutorials/build-application-logic/add-a-pallet)에서 `pallet_nicks`의 함수를 [Substrate 노드 템플릿](https://github.com/substrate-developer-hub/substrate-node-template) 런타임에 추가했습니다.

Nicks 팔렛은 블록체인 사용자가 자신이 소유한 계정에 대한 닉네임을 예약하기 위해 예치금을 지불할 수 있도록 합니다.
다음과 같은 함수를 구현합니다:

- `set_name` 함수는 계정 소유자가 이름을 설정할 수 있도록 합니다. 단, 이미 예약된 이름이 아닌 경우에만 가능합니다.
- `clear_name` 함수는 계정 소유자가 계정과 연결된 이름을 제거하고 예치금을 반환할 수 있도록 합니다.
- `kill_name` 함수는 예치금을 반환하지 않고 다른 당사자의 계정 이름을 강제로 제거할 수 있도록 합니다.
- `force_name` 함수는 예치금을 요구하지 않고 다른 당사자의 계정 이름을 설정할 수 있도록 합니다.

이 튜토리얼에서는 이러한 함수를 다른 출처 계정을 사용하여 호출하는 방법과 다른 출처 계정을 사용하여 함수를 호출하는 것이 왜 중요한지에 대해 설명합니다.

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [Rust와 Rust 툴체인](/install/)을 설치하여 Substrate 개발 환경을 구성했습니다.

- Substrate 노드 템플릿을 로컬에 설치했습니다.

- Substrate 프론트엔드 템플릿을 로컬에 설치했습니다.

- [런타임에 팔렛 추가하기](/tutorials/build-application-logic/add-a-pallet) 튜토리얼을 완료하고 `nicks` 팔렛이 포함된 런타임을 성공적으로 컴파일했습니다.

- 소프트웨어 개발과 명령 줄 인터페이스 사용에 대해 일반적으로 알고 있습니다.

## 튜토리얼 목표

이 튜토리얼을 완료함으로써 다음 목표를 달성할 수 있습니다:

- 호출을 실행할 권한이 있는 계정을 사용하여 `set_name` 함수를 호출합니다.
- 호출을 실행할 권한이 있는 계정을 사용하여 `clear_name` 함수를 호출합니다.
- 호출을 실행할 수 없는 계정을 사용하여 `force_name` 함수를 호출합니다.
- 관리 권한이 없는 출처를 사용하여 `kill_name` 함수를 호출합니다.
- `Root` 출처 계정을 사용하여 `kill_name` 함수를 호출합니다.
- 다른 출처 계정을 사용하여 함수를 호출하는 것이 실패 또는 성공적인 결과로 이어질 수 있는지 확인합니다.

## 관리 계정 식별하기

[런타임에 팔렛 추가하기](/tutorials/build-application-logic/add-a-pallet)에서 `nicks` 팔렛의 `Config` 트레이트는 여러 유형을 선언합니다.
이 튜토리얼에서는 `ForceOrigin` 유형에 초점을 맞춥니다.
`ForceOrigin` 유형은 특정 작업을 수행할 수 있는 계정을 지정하는 데 사용됩니다.
이 팔렛의 경우, `ForceOrigin` 유형은 다른 계정의 이름을 설정하거나 제거할 수 있는 계정을 지정합니다.
일반적으로 관리 권한을 가진 계정(예: 루트 슈퍼유저 계정)만 다른 계정을 대신하여 작업을 수행할 수 있습니다.
Nicks 팔렛의 경우, 계정 소유자 또는 Root 계정만 예약된 닉네임을 설정하거나 제거할 수 있습니다.
이 Root 계정은 `nicks` 팔렛 관리자로 구성할 때 구현(`impl`) 블록에서 FRAME System [`Root` 출처](https://paritytech.github.io/substrate/master/frame_system/enum.RawOrigin.html#variant.Root)를 식별할 때 구성했습니다.
예를 들면:

```rust
type ForceOrigin = frame_system::EnsureRoot<AccountId>;
```

노드 템플릿의 개발용 [체인 스펙](https://github.com/substrate-developer-hub/substrate-node-template/blob/main/node/src/chain_spec.rs)에서 [Sudo 팔렛](https://paritytech.github.io/substrate/master/pallet_sudo/index.html)은 Alice 계정을 FRAME 시스템 `Root` 출처로 사용하도록 구성되어 있습니다.
이 구성으로 인해 기본적으로 Alice 계정만 `ForceOrigin` 유형을 필요로 하는 함수를 호출할 수 있습니다.

Alice 계정 이외의 계정으로 `kill_name` 또는 `force_name`을 호출하려고 하면 호출이 실행되지 않습니다.

## 계정에 이름 설정하기

호출의 출처가 작업에 어떤 영향을 미치는지 보여주기 위해 다른 계정의 계정 이름을 설정하고 강제로 제거해 보겠습니다.
이 데모를 위해 다음을 확인하세요:

- 개발 모드에서 실행 중인 노드 템플릿: `./target/release/node-template --dev`
- 실행 중인 프론트엔드 템플릿이 로컬 노드에 연결되어 있는지 확인하세요: `yarn start`
- 브라우저가 로컬 웹 서버에 연결되어 있는지 확인하세요: <http://localhost:8000/><!-- markdown-link-check-disable-line -->

1. 프론트엔드 템플릿에서 활성 계정을 Alice에서 Bob으로 변경하세요.

1. 팔렛 인터랙터에서 **Extrinsic**를 선택한 상태에서 다음을 수행하세요:

   - `nicks` 팔렛을 선택하세요.
   - `setName` 함수를 선택하세요.
   - 계정에 대한 이름을 입력하세요.
   - 이 트랜잭션을 Bob이 서명한 상태로 제출하기 위해 **Signed**를 클릭하세요.

   Bob은 이 계정의 소유자이므로 트랜잭션이 성공합니다.
   계정 소유자인 Bob은 서명된 트랜잭션으로 `clearName` 함수를 실행하여 계정의 닉네임을 제거할 수도 있습니다.

1. **Extrinsic**를 선택한 상태에서 다음을 수행하세요:

   - `nicks` 팔렛을 선택하세요.
   - `clearName` 함수를 선택하세요.
   - 이 트랜잭션을 Bob이 서명한 상태로 제출하기 위해 **Signed**를 클릭하세요.

   Bob은 이 계정의 소유자이므로 트랜잭션이 성공합니다.
   Bob이 다른 계정의 닉네임을 설정하거나 제거하려면 `ForceOrigin`을 사용하여 `forceName` 또는 `killName` 함수를 호출해야 합니다.

1. **Extrinsic**를 선택한 상태에서 다음을 수행하세요:

   - `nicks` 팔렛을 선택하세요.
   - `forceName` 함수를 선택하세요.
   - 대상으로 Charlie의 계정 주소를 복사하여 붙여넣으세요.
   - 계정에 대한 이름을 입력하세요.
   - 이 트랜잭션을 Bob이 서명한 상태로 제출하기 위해 **Signed**를 클릭하세요.

   이 트랜잭션을 Bob의 계정으로 서명했기 때문에 함수는 `Root` 출처 대신 [`Signed` 출처](https://paritytech.github.io/substrate/master/frame_system/enum.RawOrigin.html#variant.Signed)를 사용하여 디스패치됩니다.
   이 경우, 함수 호출 자체는 성공합니다.
   그러나 이름 예약을 완료할 수 없으며 `BadOrigin` 오류가 발생합니다.

   ![BadOrigin 오류](/media/images/docs/tutorials/add-a-pallet/badOrigin.png)

   이벤트에서 볼 수 있듯이, 이 트랜잭션은 트랜잭션 제출을 위한 [수수료](/build/tx-weights-fees/)로 Bob의 계정에서 인출되었지만, `Root` 출처가 트랜잭션을 제출하지 않았기 때문에 상태 변경이 없습니다.
   상태 변경 실패는 또한 [verify-first-write-last](/build/runtime-storage#verify-first-write-last) 원칙을 보여주며, 디스크에 성공적인 작업만 커밋되도록 데이터베이스 읽기와 쓰기를 보장합니다.

## Root 출처를 사용하여 호출 디스패치하기

Sudo 팔렛을 사용하면 `Root` 출처를 사용하여 호출을 디스패치할 수 있습니다.
Nick 팔렛에서는 `ForceOrigin` 구성에 따라 `Root` 출처를 사용하여 `forceName` 및 `killName` 함수를 호출해야 합니다.
프론트엔드 템플릿에서 `Root` 출처를 사용하여 호출을 디스패치하려면 **SUDO**를 클릭하세요.

이 데모를 위해 다음을 확인하세요:

- 개발 모드에서 실행 중인 노드 템플릿: `./target/release/node-template --dev`
- 실행 중인 프론트엔드 템플릿이 로컬 노드에 연결되어 있는지 확인하세요: `yarn start`
- 브라우저가 로컬 웹 서버에 연결되어 있는지 확인하세요: <http://localhost:8000/><!-- markdown-link-check-disable-line -->

1. 활성 계정을 Alice로 변경하세요.

   [관리 계정 식별하기](#관리-계정-식별하기)에서 언급한 대로, 개발 모드에서 체인을 실행할 때 `Root` 출처와 연결된 계정은 Alice입니다.

1. 팔렛 인터랙터에서 **Extrinsic**를 선택한 상태에서 다음을 수행하세요:

   - `nicks` 팔렛을 선택하세요.
   - `forceName` 함수를 선택하세요.
   - 대상으로 Charlie의 계정 주소를 복사하여 붙여넣으세요.
   - 계정에 대한 이름을 입력하세요.
   - 이 트랜잭션을 `Root` 출처를 사용하여 제출하기 위해 **SUDO**를 클릭하세요.

   ![SUDO를 사용하여 트랜잭션 제출](/media/images/docs/tutorials/add-a-pallet/sudo-tx.png)

1. **Extrinsic**를 선택한 상태에서 다음을 수행하세요:

   - `nicks` 팔렛을 선택하세요.
   - `killName` 함수를 선택하세요.
   - 대상으로 Bob의 계정 주소를 복사하여 붙여넣으세요.
   - 이 트랜잭션을 `Root` 출처를 사용하여 제출하기 위해 **SUDO**를 클릭하세요.

   이 경우, Sudo 팔렛은 [`Root` 출처가 호출을 디스패치했음을](https://paritytech.github.io/substrate/master/pallet_sudo/pallet/enum.Event.html) 네트워크 참가자에게 알리기 위해 [`Sudid` 이벤트](https://paritytech.github.io/substrate/master/pallet_sudo/pallet/enum.Event.html)를 발생시킵니다.

   ![killName 함수 제출 시 오류 발생](/media/images/docs/tutorials/add-a-pallet/sudo-error.png)

   이 디스패치 오류에는 두 가지 메타데이터가 포함됩니다:

   - 오류가 발생한 팔렛을 나타내는 `index` 번호입니다.
   - 해당 팔렛의 `Error` enum에서 발생한 오류를 나타내는 `error` 번호입니다.

   `index` 번호는 `construct_runtime!` 매크로 내에서 팔렛의 위치에 해당합니다. `construct_runtime!` 매크로의 _첫 번째_ 팔렛은 0의 인덱스 번호를 가집니다.

   이 예에서 `index`는 `6`(일곱 번째 팔렛)이고 `error`는 `2`입니다(세 번째 오류).

   ```rust
   construct_runtime!(
    pub enum Runtime where
      Block = Block,
      NodeBlock = opaque::Block,
      UncheckedExtrinsic = UncheckedExtrinsic
    {
      System: frame_system,                                        // index 0
      RandomnessCollectiveFlip: pallet_randomness_collective_flip, // index 1
      Timestamp: pallet_timestamp,                                 // index 2
      Aura: pallet_aura,                                           // index 3
      Grandpa: pallet_grandpa,                                     // index 4
      Balances: pallet_balances,                                   // index 5
      Nicks: pallet_nicks,                                         // index 6
    }
   ```

   `index` 값에 관계없이 `error` 값 `2`는 Nicks 팔렛의 [`Unnamed` 오류](https://paritytech.github.io/substrate/master/pallet_nicks/pallet/enum.Error.html)에 해당합니다.
   이는 Bob이 닉네임을 예약하지 않았거나 이전에 이름 예약을 제거한 경우에 기대할 수 있는 오류입니다.

   Alice가 현재 이름이 예약된 모든 계정에 대해 `killName` 함수를 호출할 수 있는지 확인할 수 있습니다.

1. **Extrinsic**를 선택한 상태에서 다음을 수행하세요:

   - `nicks` 팔렛을 선택하세요.
   - `killName` 함수를 선택하세요.
   - 대상으로 Charlie의 계정 주소를 복사하여 붙여넣으세요.
   - 이 트랜잭션을 `Root` 출처를 사용하여 제출하기 위해 **SUDO**를 클릭하세요.

   ![성공적인 killName 트랜잭션 제출](/media/images/docs/tutorials/add-a-pallet/sudo-error.png)

## 다음 단계로 넘어가기

이 튜토리얼에서는 트랜잭션을 제출하는 데 사용되는 계정을 지정하기 위해 `Root` 및 `Signed` 출처를 사용하는 방법을 소개하고, 다른 출처 계정을 사용하여 함수를 호출하는 결과를 보여주었습니다.
Substrate 개발에 대해 더 알아보기 위해 다음 [튜토리얼](/tutorials/)을 참고할 수 있습니다.

튜토리얼 외에도 다음 리소스를 탐색하여 더 많은 정보를 얻을 수 있습니다.

- [특권 호출과 출처](/build/origins)는 기본적인 원시(raw) 출처 유형과 사용자 정의 출처를 생성하는 방법에 대해 자세히 설명합니다.
- [이벤트와 오류](/build/events-and-errors)는 런타임에서 이벤트와 오류를 발생시키는 방법을 설명합니다.
- [FRAME 팔렛](/reference/frame-pallets/)은 가장 일반적으로 사용되는 미리 정의된 FRAME 팔렛에 대한 개요를 제공합니다.