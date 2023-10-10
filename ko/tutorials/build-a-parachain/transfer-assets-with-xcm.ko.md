---
title: XCM을 사용하여 자산 이전하기
description: 릴레이 체인을 통해 패러체인으로 원격 이전을 실행하는 교차 합의 메시지 사용 방법을 보여줍니다.
keywords:
  - XCM
  - 패러체인 통신
  - 메시징
  - 교차 체인
  - 교차 합의
---

[Open message passing channels](/tutorials/build-a-parachain/open-message-passing-channels)에서는 메시지를 릴레이 체인으로 보내어 체인 간 양방향 통신 채널을 열어보았습니다.
이와 비슷한 전략을 사용하여 로컬 체인이 원격 체인의 계정을 관리할 수 있는 메시지를 보낼 수 있습니다.
이 튜토리얼에서는 패러체인 B가 패러체인 A의 릴레이 체인에 있는 주권 계정으로 자산을 이전합니다.

이 튜토리얼의 결과는 잔액 팔렛의 `transfer` 함수를 사용하는 것과 유사하지만, 이 경우 이전이 패러체인에 의해 시작되고 `WithdrawAsset` 및 `DepositAsset` XCM 명령을 실행할 때 보유 레지스터가 사용되는 방법을 보여줍니다.

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- Zombienet 또는 `rococo-local` 체인 스펙을 사용하여 [패러체인 테스트 네트워크](/test/simulate-parachains)를 설정했는지 확인하세요.

- 테스트 목적으로 두 개의 로컬 또는 가상 패러체인을 설정했는지 확인하세요.

  이 튜토리얼에서는 패러체인 A의 고유 식별자가 1000이고 패러체인 B의 고유 식별자가 1001인 것으로 가정합니다.

- 로컬 패러체인이나 가상 패러체인에서 Sudo 팔렛을 사용할 수 있는지 확인하세요.

- 패러체인 B와 패러체인 A 간의 통신을 허용하기 위해 메시지 패싱 채널을 열었는지 확인하세요.

## XCM 명령 구성하기

두 체인 간 상호 작용을 설명하기 위해 다음 예제에서는 패러체인 B가 XCM 명령을 사용하여 자산을 패러체인 A의 계정에 예금하는 메시지를 보냅니다.

1. [Polkadot/Substrate Portal](https://polkadot.js.org/apps)을 사용하여 패러체인 B(1001)의 엔드포인트에 연결합니다.

2. **개발자**를 클릭하고 **엑스트린식**을 선택합니다.

3. **sudo**를 선택한 다음 **sudo(call)**을 선택하여 Sudo 팔렛을 사용하여 특권이 있는 트랜잭션을 실행합니다.

4. **polkadotXcm**을 선택한 다음 **send(dest, message)**를 선택합니다.

5. 메시지를 전달할 상대적인 위치를 나타내기 위해 대상 매개변수를 지정합니다.

   - 대상의 위치를 지정하기 위한 XCM 버전: V1
   - 메시지의 대상은 릴레이 체인이므로 상위 위치: 1
   - 상위 위치의 컨텍스트에서의 내부 설정: Here

6. 메시지의 XCM 버전을 지정합니다(V2).

7. 실행할 메시지를 구성하기 위해 **항목 추가**를 클릭합니다.

## WithdrawAsset 명령

가상 보유 레지스터로 자산을 이동하기 위해:

1. 이 메시지의 첫 번째 명령으로 [WithdrawAsset](https://github.com/paritytech/xcm-format#withdrawasset)를 선택합니다.

2. **항목 추가**를 클릭하여 체인 상의 자산을 식별합니다.

3. 자산을 식별하기 위해 **Concrete**를 선택합니다.

4. 주권 계정인 패러체인 B의 자산을 릴레이 체인에서 인출하기 위해 **parents: 0** 및 **interior: Here**를 설정합니다.

5. 자산을 펀지블 자산으로 식별하기 위해 **Fungible**을 선택합니다.

6. 인출할 총 펀지블 자산을 지정합니다.

   예를 들어, 이 튜토리얼에서는 12000000000000을 사용합니다.
   
   ![패러체인 B에서 보낸 WithdrawAsset 명령](/media/images/docs/tutorials/parachains/transfer-withdraw-asset-instruction-ui.png)

## BuyExecution 명령

보유 레지스터에 예금된 자산으로 실행 비용을 지불하기 위해:

1. **항목 추가**를 클릭하여 이 메시지의 두 번째 명령으로 [BuyExecution](https://github.com/paritytech/xcm-format#buyexecution)을 선택합니다.

2. 자산을 식별하기 위해 **Concrete**를 선택합니다.

3. XCM 명령 실행에 사용할 자산을 식별하기 위해 **parents: 0** 및 **interior: Here**를 설정합니다.

4. 자산을 펀지블 자산으로 식별하기 위해 **Fungible**을 선택합니다.

5. 사용할 총 펀지블 자산을 지정합니다.
   
   예를 들어, 이 튜토리얼에서는 12000000000000을 사용합니다.

6. 이 명령에 대한 가중치 제한을 설정하지 않으려면 **Unlimited**를 선택합니다.
   
   ![패러체인 B에서 보낸 BuyExecution 명령](/media/images/docs/tutorials/parachains/transfer-buy-execution-instruction-ui.png)

## DepositAsset 명령

보유 레지스터에서 수수료를 제외한 자산을 특정 계정으로 예금하기 위해:

1. **항목 추가**를 클릭하여 이 메시지의 세 번째 명령으로 [DepositAsset](https://github.com/paritytech/xcm-format#depositasset)을 선택합니다.

1. 지정되지 않은 수의 자산을 예금할 수 있도록 **Wild**를 선택합니다.

1. 수수료를 지불한 후 남은 자산을 모두 예금할 수 있도록 **All**을 선택합니다.

2. 예금을 위해 보유 레지스터에서 제거할 고유 자산의 최대 수를 **1**로 설정합니다.

   이 튜토리얼에서는 제거할 수 있는 자산 인스턴스가 하나뿐입니다.

1. 예금할 자산을 수령할 수혜자를 지정합니다.

   남은 자산을 패러체인 A의 주권 계정에 예금하거나 특정 계정에 예금할 수 있습니다.
   이 튜토리얼에서는 예금할 자산을 이전에 자금이 없던 계정 KRIS-PUBS에 지정된 계정 주소를 사용하여 예금합니다.
   이를 위해 수혜자를 선택하면 DepositAsset 명령은 다음과 같이 보입니다:

   ![계정을 수혜자로 지정](/media/images/docs/tutorials/parachains/transfer-deposit-asset-instruction-ui.png)
   
   패러체인 A(1000)의 주권 계정에 자산을 예금하려면 다음 설정을 사용하여 수혜자를 지정할 수 있습니다:
   
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

1. [Polkadot/Substrate Portal](https://polkadot.js.org/apps)을 열고 릴레이 체인에 연결합니다.

2. **네트워크**를 클릭하고 **탐색기**를 선택하여 XCM 메시지의 이벤트를 확인합니다.
   
   ![릴레이 체인 이벤트](/media/images/docs/tutorials/parachains/relay-chain-event-summary.png)

1. 변경 사항이 기록된 블록 번호를 클릭하여 세부 정보를 확인합니다.
   
   ![릴레이 체인에 기록된 인출 및 예금 자산](/media/images/docs/tutorials/parachains/relay-chain-block.png)

## 예금된 자산 확인하기

계정에 예금된 자산을 확인하려면:

1. [Polkadot/Substrate Portal](https://polkadot.js.org/apps)을 열고 릴레이 체인에 연결합니다.

2. **계정**을 클릭하고 트랜잭션 수수료를 제외한 자산이 계정에 예금되었는지 확인합니다.
   
   예를 들어:

   ![지정된 계정에 자산이 예금되었습니다](/media/images/docs/tutorials/parachains/transfer-account-funded.png)

   KRIS-PUBS 계정 대신 패러체인 A(1000)의 주권 계정으로 원격 이전을 수행했다면 **계정**을 클릭한 다음 **주소록**을 선택하여 패러체인 B의 주권 계정에서 인출된 자산이 패러체인 A의 주권 계정에 예금되었는지 확인할 수 있습니다.