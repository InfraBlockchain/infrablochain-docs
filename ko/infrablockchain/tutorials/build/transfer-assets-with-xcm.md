---
title: XCM을 이용하여 원격으로 자산 전송하기
description: 릴레이 체인을 통해 파라체인으로 원격 전송을 실행하는 XCM 사용 방법을 보여줍니다.
keywords:
  - XCM
---

[Open message passing channels](./open-message-passing-channels.md)에서는 메시지를 릴레이 체인으로 보내어 체인 간 양방향 통신 채널을 열어보았습니다. 
본 튜토리얼에서는 파라체인 B가 파라체인 A의 릴레이 체인에 있는 sovereign 계정으로 자산을 이전합니다. [`balances` 팔렛](https://github.com/InfraBlockchain/infrablockspace-sdk/tree/master/substrate/frame/balances)의 `transfer` 함수를 사용하는 것과 유사하지만, 이 경우 XCM 을 이용하여 자산을 전송하는 예를 보여줍니다.

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [Zombienet](../test/simulate-parachains.ko.md) 또는 `infra-relay-local` 체인 스펙을 사용하여 [파라체인 테스트 네트워크](./build-a-parachain.md)를 설정하는 방법에 대해 확인합니다.

- 두 개의 로컬 파라체인을 설정했는지 확인하세요.

  - 본 튜토리얼에서는 파라체인 A의 고유 식별자가 1000이고 파라체인 B의 고유 식별자가 1001인 것으로 가정합니다.

  - 로컬 파라체인에서 [`Sudo 팔렛`](https://github.com/InfraBlockchain/infrablockspace-sdk/tree/master/substrate/frame/sudo)을 사용할 수 있는지 확인하세요.

- 파라체인 B와 파라체인 A 간의 통신을 허용하기 위해 메시지 패싱 채널(HRMP)를 열었는지 확인하세요.

## XCM 명령 구성하기

두 체인 간 상호 작용을 설명하기 위해 다음 예제에서는 파라체인 B가 XCM 명령을 사용하여 릴레이 체인의 파라체인 B soverign 계정의 자산을 릴레이 체인의 어떤 특정한 계정으로 전송하는 방법을 보여줍니다.

1. [InfraBlockspace Portal](https://portal.infrablockspace.net)을 사용하여 파라체인 B(1001)의 엔드포인트에 연결합니다.

2. **개발자**를 클릭하고 **외부 트랜잭션**을 선택합니다.

3. **sudo**를 선택한 다음 **sudo(call)** 을 선택하여 Sudo 팔렛을 사용하여 트랜잭션을 실행합니다.

4. **ibsXcm**을 선택한 다음 **send(dest, message)**를 선택합니다.

5. 메시지를 전달할 상대적인 위치를 나타내기 위해 대상 매개변수를 지정합니다.

   - 대상의 위치(여기서는 릴레이 체인)를 지정하기 위한 XCM 버전: V3
   - **parents: 1** 및 **interior: Here** 를 설정

6. 메시지의 XCM 버전을 지정합니다(V3).

7. 실행할 메시지를 구성하기 위해 **항목 추가**를 클릭합니다.

## WithdrawAsset 명령

Holding 레지스터로 자산을 이동하기 위해:

1. 이 메시지의 첫 번째 명령으로 *WithdrawAsset* 를 선택합니다.

2. **항목 추가**를 클릭하여 체인 상의 자산(MultiLocation)을 식별합니다.

3. 자산을 식별하기 위해 **Concrete**를 선택합니다.

4. sovereign 계정인 파라체인 B의 자산을 릴레이 체인에서 인출하기 위해 **parents: 0** 및 **interior: Here**를 설정합니다.

5. `StagingXcmV3MultiassetFungibility` 에서 **Fungible**을 선택합니다.

6. 인출할 총 자산을 지정합니다.

   예를 들어, 이 튜토리얼에서는 12000000000000을 사용합니다.
   
   ![파라체인 B에서 보낸 WithdrawAsset 명령](/media/images/docs/infrablockchain/tutorials/build/transfer-withdraw-asset-instruction-ui.png)

## BuyExecution 명령

Holding 레지스터에 예금된 자산으로 실행 비용을 지불하기 위해:

1. **항목 추가**를 클릭하여 이 메시지의 두 번째 명령으로 BuyExecution을 선택합니다.

2. 자산을 식별하기 위해 **Concrete**를 선택합니다.

3. XCM 명령 실행에 사용할 자산을 식별하기 위해 **parents: 0** 및 **interior: Here**를 설정합니다.

4. `StagingXcmV3MultiassetFungibility` 에서 **Fungible**을 선택합니다.

5. 사용할 총 자산을 지정합니다.
   
   예를 들어, 이 튜토리얼에서는 12000000000000을 사용합니다.

6. 이 명령에 대한 가중치(리소스 사용량, weight) 제한을 설정하지 않으려면 **Unlimited**를 선택합니다.
   
   ![파라체인 B에서 보낸 BuyExecution 명령](/media/images/docs/infrablockchain/tutorials/build/transfer-buy-execution-instruction-ui.png)

## DepositAsset 명령

Holding 레지스터에서 수수료를 제외한 자산을 특정 계정으로 예금하기 위해:

1. **항목 추가**를 클릭하여 이 메시지의 세 번째 명령으로 *DepositAsset*을 선택합니다.

2. Holding register 에 있는 자산을 예금할 수 있도록 **Wild**를 선택합니다.

3. 수수료를 지불한 후 남은 자산을 모두 예금할 수 있도록 **All**을 선택합니다.

4. 예금을 위해 보유 레지스터에서 제거할 고유 자산의 최대 수를 **1**로 설정합니다. 본 튜토리얼에서는 제거할 수 있는 자산 인스턴스가 하나뿐입니다.

1. 자산을 전송받을 계정(beneficary)를 지정합니다.

   - 남은 자산을 파라체인 A의 sovereign 계정에 예금하거나 특정 계정에 예금할 수 있습니다. 
   - 본 튜토리얼에서는 예금할 자산을 이전에 자금이 없던 계정 KRIS-PUBS에 지정된 계정 주소를 사용하여 예금합니다.
   - 이를 위해 수혜자를 선택하면 DepositAsset 명령은 다음과 같이 보입니다:

   ![계정을 수혜자로 지정](/media/images/docs/tutorials/parachains/transfer-deposit-asset-instruction-ui.png)
   
   파라체인 A(1000)의 sovereign 계정에 자산을 예금하려면 다음 설정을 사용하여 계정을 지정할 수 있습니다:
   
   - parents: 0, 
   - interior: X1, 
   - X1 junction: Parachain
   - Parachain index: 1000
  
   모든 XCM 명령을 구성한 후, 트랜잭션을 제출할 준비가 되었습니다.

## 트랜잭션 제출하기

트랜잭션을 제출하려면:

1. **트랜잭션 제출**을 클릭합니다.

1. **서명 및 제출**을 클릭합니다.

1. **네트워크**를 클릭하고 **탐색기**를 선택하여 메시지가 전송되었는지 확인합니다.
   
   이벤트를 확장하면 메시지 명령을 검토할 수 있습니다.
   트랜잭션을 포함한 블록으로 이동하는 링크를 클릭하면 추가 세부 정보를 볼 수 있습니다.

## 릴레이 체인에서 이벤트 확인하기

릴레이 체인에서 결과를 확인하려면:

1. [InfraBlockspace Portal](https://portal.infrablockspace.net)을 열고 릴레이 체인에 연결합니다.

2. **네트워크**를 클릭하고 **탐색기**를 선택하여 XCM 메시지의 이벤트를 확인합니다.
   
   ![릴레이 체인 이벤트](/media/images/docs/infrablockchain/tutorials/build/relay-chain-event-summary.png)

3. 변경 사항이 기록된 블록 번호를 클릭하여 세부 정보를 확인합니다.
   
   ![릴레이 체인에 기록된 인출 및 예금 자산](/media/images/docs/infrablockchain/tutorials/build/relay-chain-block.png)

## 예금된 자산 확인하기

계정에 예금된 자산을 확인하려면:

1. [InfraBlockspace Portal](https://portal.infrablockspace.net)을 열고 릴레이 체인에 연결합니다.

2. **계정**을 클릭하고 트랜잭션 수수료를 제외한 자산이 계정에 예금되었는지 확인합니다.
   
   예를 들어:

   ![지정된 계정에 자산이 예금되었습니다](/media/images/docs/infrablockchain/tutorials/build/transfer-account-funded.png)

   KRIS-PUBS 계정 대신 파라체인 A(1000)의 sovereign 계정으로 원격 이전을 수행했다면 **계정**을 클릭한 다음 **주소록**을 선택하여 파라체인 B의 sovereign 계정에서 인출된 자산이 파라체인 A의 sovereign 계정에 예금되었는지 확인할 수 있습니다.