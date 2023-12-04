---
title: 메시지 전달 채널 열기
description: 파라체인 간 통신을 위해 HRMP를 사용하는 방법을 배웁니다.
keywords:
  - 크로스 컨센서스 메시징
  - XCM
---

**_인프라 블록체인(InfraBlockchain)_** 은 안전한 채널을 통해 서로 다른 컨센서를 가진 블록체인 네트워크 간 통신을할 수 있습니다.
세 가지 주요 통신 채널은 다음과 같습니다:

- UMP: 파라체인이 릴레이 체인으로 메시지를 전달할 수 있도록 합니다.
- DMP:  릴레이 체인이 파라체인으로 메시지를 전달할 수 있도록 합니다.
- XCMP: 파라체인이 릴레이체인을 거치지 않고 서로 메시지를 보낼 수 있도록 합니다.
- HRMP: 파라체인간 메세지를 릴레이 체인을 거처서 메세지를 보낼 수 있도록 합니다.

본 튜토리얼에서는 HRMP를 사용하여 파라체인 간 통신을 가능하게 하는 메시지 전달 채널을 열 수 있는 방법을 설명합니다.

## HRMP 채널 열기

*HRMP 채널은 단방향입니다.* 파라체인 1000이 파라체인 1001과 통신하려면, 먼저 파라체인 1000에서 파라체인 1001로 HRMP 채널을 열기 위한 요청을 보내야 합니다.
파라체인 1001은 이 요청을 수락해야만 파라체인 1000이 메시지를 전달할 수 있습니다. 그러나 채널이 단방향이기 때문에 파라체인 1000은 채널을 통해 파라체인 1001로부터 메시지를 받을 수 없습니다.

파라체인 1000이 파라체인 1001로부터 메시지를 받으려면, 파라체인 1001에서 파라체인 1000으로 또 다른 채널을 열어야 합니다. 파라체인 1000이 파라체인 1001로부터 메시지를 수락할 것임을 확인한 후, 다음 세션에서 메시지를 교환할 수 있습니다.

## 시작하기 전에

이 튜토리얼에서는 고유 식별자 1000을 가진 파라체인과 고유 식별자 1001을 가진 파라체인 간에 메시지를 교환할 수 있는 HRMP 채널을 엽니다.

시작하기 전에 다음을 확인하세요:

- Zombienet을 사용하여 [파라체인 테스트 네트워크](../test/simulate-parachains.md)를 구성하는 방법을 확인합니다. 

- 테스트 목적으로 두 개의 로컬 파라체인을 설정했습니다.

  - 이 튜토리얼에서는 파라체인 A의 고유 식별자가 1000이고, 파라체인 B의 고유 식별자가 1001인 것으로 가정합니다.

  - 로컬 파라체인에서 두 파라체인 모두가 Sudo 팔레트를 사용할 수 있도록 설정했습니다. *실제 환경에서는 Sudo 팔레트 대신 거버넌스 제안 및 투표를 사용해야 합니다.*

## Sovereign 계정 추가

파라체인이 다른 파라체인과 메시지를 교환하려면, 실행할 XCM 명령에 대한 비용을 지불하기 위해 릴레이 체인 계정에 ***해당 파라체인이 컨트롤할 수 있는 계정(Soverign 계정)*** 에 자산이 있어야 합니다.

릴레이 체인에 Sovereign 계정 주소를 추가하려면 다음을 수행하세요:

1. [인프라 블록체인 익스플로러](https://portal.infrablockspace.net/#/explorer/)를 열고 릴레이 체인 엔드포인트에 연결합니다.

2. 파라체인 [Sovereign 계정 주소](https://substrate.stackexchange.com/questions/1200/how-to-calculate-sovereignaccount-for-parachain)를 계산하여 릴레이 체인에서 사용할 수 있도록 합니다.

   파라체인 A (1000) 주소: 5Ec4AhPZk8STuex8Wsi9TwDtJQxKqzPJRCH7348Xtcs9vZLJ

   파라체인 B (1001) 주소: 5Ec4AhPZwkVeRmswLWBsf7rxQ3cjzMKRWuVvffJ6Uuu89s1P

   _파라체인에 등록된 파라체인 식별자가 변경되면 sovereign 계정과 주소도 변경됩니다._ 또한 릴레이 체인에서 파라체인에 사용되는 계정 주소는 다른 파라체인에서 사용되는 주소와 다릅니다.

3. **계정**을 클릭하고 **주소록**을 선택합니다.

4. **연락처 추가**를 클릭합니다.

5. sovereign 계정 A (1000)의 주소와 이름을 추가하고 **저장**을 클릭합니다.

6. **계정**을 클릭하고 Alice에서 파라체인 A (1000) 계정으로 일부 자산을 이체합니다. 파라체인 B (1001)에 대해서도 3단계에서 6단계까지 반복합니다.

## 채널 열기 및 인코딩된 호출 준비

파라체인 간 통신을 설정하려면, 메시지 전달 채널을 열기 위한 요청을 보내야 합니다.
요청은 수신자 파라체인, 채널의 메시지 용량 및 최대 메시지 크기를 지정하는 매개변수로 인코딩된 호출 형식으로 이루어져야 합니다.
요청을 만들기 위해 준비한 인코딩된 버전의 정보를 생성한 메시지에 포함해야 하지만, 요청을 제출하지 않고도 인코딩된 호출을 생성할 수 있습니다.

채널을 열기 위한 인코딩된 호출을 준비하려면 다음을 수행하세요:

1. [인프라 블록체인 익스플로러](https://portal.infrablockspace.net/#/explorer/)를 열고 릴레이 체인 엔드포인트에 연결합니다.

2. 필요한 경우 현재 릴레이 체인의 구성값을 확인합니다.

   현재 릴레이 체인의 구성값을 확인하려면:

   - **개발자**를 클릭하고 **체인 상태**를 선택합니다.
   - **configuration**을 선택한 다음 **activeConfig()** 를 선택하고 **+** 를 클릭합니다.
   - `hrmpChannelMaxCapacity` 및 `hrmlChannelMaxMessageSize` 매개변수의 매개변수 값을 확인합니다.

   예를 들면:

      ```text
      hrmpChannelMaxCapacity: 8
      hrmlChannelMaxMessageSize: 1,048,576
      ```

3. **개발자** 를 클릭하고 **외부 트랜잭션**을 선택합니다.

4. **hrmp** 를 선택한 다음 **hrmpInitOpenChannel(recipient, proposedMaxCapacity, proposedMaxMessageSize)** 를 선택하여 새 채널을 열기 위한 요청을 초기화합니다.

   트랜잭션 매개변수로 다음을 지정하여 호출 데이터를 준비합니다:

   - recipient: 채널을 열고자 하는 파라체인의 식별자를 입력합니다 (1001).
   - proposedMaxCapacity: 한 번에 채널에 있는 메시지의 최대 수를 입력합니다 (8).
   - proposedMaxMessageSize: 메시지의 최대 크기를 지정합니다 (1048576).

   **proposedMaxCapacity** 및 **proposedMaxMessageSize**에 설정하는 값은 릴레이 체인의 `hrmpChannelMaxCapacity` 및 `hrmpChannelMaxMessageSize` 매개변수에 정의된 값보다 크지 않아야 합니다.

5. **인코딩된 트랜잭션 데이터**를 복사합니다.

   ![인코딩된 트랜잭션 데이터 복사](/media/images/docs/tutorials/parachains/hrmp-encoded-call.png)

   이 정보는 XCM Transact 명령을 구성하는 데 사용됩니다.
   다음은 `infra-relay` 에서 인코딩된 호출 데이터의 예입니다: `0x3c00e90300000800000000001000`

## 채널 요청 구성

인코딩된 호출을 보유하고 있으므로, 릴레이 체인을 통해 파라체인 A에서 파라체인 B로 채널을 열기 위한 요청을 구성할 수 있습니다.

1. [인프라 블록체인 익스플로러](https://portal.infrablockspace.net/#/explorer/)를 열고 파라체인 A (1000)의 엔드포인트에 연결합니다.

2. **개발자**를 클릭하고 **외부 트랜잭션**을 선택합니다.

3. **sudo**를 선택한 다음 특권 트랜잭션을 실행하기 위해 **sudo(call)** 을 선택합니다.

4. **ibsXcm** 을 선택한 다음 파라체인 B (1001)과 채널을 열고자 하는 요청을 전달하기 위해 **send(dest, message)**를 선택합니다.

   ![Sudo 팔레트를 사용하여 메시지 전송](/media/images/docs/infrablockchain/tutorials/build/hrmp-sudo-call.png)

5. 메시지를 전달할 상대 위치를 나타내기 위해 대상(dest) 매개변수를 지정합니다.

   대상 매개변수는 XCM이 실행될 위치를 지정합니다. 본 튜토리얼에서는 파라체인 1000의 상위 체인이 릴레이 체인이며, 
   메세지를 받는 체인의 컨텍스트에서 `MultiLocation::Here` 는 릴레이 체인 안에서 XCM을 실행할 것임을 의미합니다.

   ![대상 매개변수](/media/images/docs/infrablockchain/tutorials/build/hrmp-destination.png)

6. XCM 버전을 지정한 다음 실행할 메시지를 구성하기 위해 **항목 추가**를 클릭합니다.

   이 메시지를 실행하기 위해 다음과 같은 일련의 명령을 추가해야 합니다:

   - *WithdrawAsset*: 지정된 온체인 자산을 가상 *holding 레지스터* 로 이동시킵니다.

   - *BuyExecution*: *WithdrawAsset* 명령을 사용하여 *holding 레지스터* 에 예치된 자산을 사용하여 현재 메시지의 실행 비용을 지불합니다.

   - *Transact*: 준비한 *인코딩된 트랜잭션* 을 지정합니다.

   대부분의 경우 다음 명령도 포함해야 합니다:

   - *RefundSurplus*: 실제 지불해야할 양보다 BuyExecution 명령을 사용하여 지불한 수수료가 작을 경우 나머지를 *refunded-weight* 레지스터로 이동시킵니다.

   - *DepositAsset*: *refunded-weight* 레지스터에서 자산을 빼내고, 지정 계정에 입금시킵니다.

   각 명령을 수신받는 시스템이 송신한 XCM 명령을 실행할 수 있도록 추가 정보를 지정해야 합니다.
   각 명령에 대한 정보를 수신받는 시스템의 관점에서 구성해야 합니다.

   명령에 대해 자세히 살펴보겠습니다.

### WithdrawAsset

자산을 holding 레지스터로 이동시키려면:

1. 이 메시지의 첫 번째 명령으로 *WithdrawAsset* 을 선택합니다.

2. **항목 추가** 를 클릭하여 인출할 온체인 자산을 식별합니다.

3. 자산을 식별하기 위해 자산의 위치를 사용하려면 **concrete**를 선택합니다.

4. **parents: 0** 및 **interior: Here** 를 설정하여 sovereign 계정에서 자산을 인출합니다.

5. `StagingXCMV3MultiAssetFungibility` 에서 **Fungible** 을 선택합니다.

6. 인출할 총 가치 교환 가능한 자산을 지정합니다.
   
   ![WithdrawAsset 및 설정](/media/images/docs/infrablockchain/tutorials/build/withdraw-asset-instruction-settings.png)

### BuyExecution 명령

Holding 레지스터에 예치된 자산을 사용하여 실행 비용을 지불하려면:

1. 최상위 **항목 추가**를 클릭하여 *BuyExecution* 을 선택합니다.

2. 용할 자산을 식별하기 위해 **concrete**를 선택합니다.

3. 사용할 자산을 식별하기 위해 **parents: 0** 및 **interior: Here**를 설정합니다.

4. `StagingXCMV2MultiAssetFungibility` 에서 **Fungible** 를 선택합니다.

5. 사용할 총 자산을 지정합니다.

6. 이 명령에 대한 가중치(리소스 사용량, weight) 제한을 설정하지 않으려면 **Unlimited** 를 선택합니다.

   ![BuyExecution 및 설정](/media/images/docs/tutorials/parachains/buy-execution-open.png)

### Transact 

인코딩된 트랜잭션을 실행하려면:

1. 최상위 **항목 추가**를 클릭하여 세 번째 명령으로 *Transact* 를 선택합니다.

2. 명령을 실행하기 위한 메시지 원본으로 **Native** 를 선택합니다.

3. XCM 호출을 디스패치하는 데 사용할 최대 가중치를 지정하기 위해 **requireWeightAtMost** 를 설정합니다.

   - XCM 실행하는데 사용한 가중치가 지정한 가중치보다 큰 경우 트랜잭션이 실패합니다.
   - XCM 실행하는데 사용한 가중치가 지정한 가중치보다 작은 경우 해당 차액은 _refundable-fund_ 레지스터에 추가될 수 있습니다.

4. 실행할 트랜잭션의 인코딩된 트랜잭션을 지정합니다.

   예를 들어, hrmp 채널을 요청하는 인코딩된 호출을 붙여넣습니다.

   ![Transact 및 설정](/media/images/docs/infrablockchain/tutorials/build/transact-open-request.png)

### RefundSurplus 및 DepositAsset

트랜잭션 수수료 관련 메세지를 추가합니다.

1. 최상위 **항목 추가**를 클릭하여 네 번째 명령으로 *RefundSurplus* 를 선택합니다.

2. 최상위 **항목 추가**를 클릭하여 다섯 번째 명령으로 *DepositAsset* 를 선택합니다.

3. 지불할 자산의 수를 지정하지 않고, 자산을 _refundable-fund_ 레지스터에서 제거하고 받는 사람(beneficiary)에게 입금하기 위해 **Wild**를 선택합니다.

4. 환불된 자산 중에서 제거할 고유 자산의 최대 수를 1로 설정합니다.

   - 본 튜토리얼에서는 제거할 수 있는 자산 인스턴스가 하나뿐입니다.

5. 자산을 받을 계정(beneficiary)를 지정합니다.

   - 일반적으로 DepositAsset 명령의 수혜자는 메시지 발신자의 *sovereign 계정* 입니다.
   - 이 경우, 파라체인 A (1000)을 **parents: 0**, **interior: X1**, **Parachain: 1000** 으로 지정하여 잉여 자산이 soverign 계정으로 반환되고 다른 XCM 명령을 전달하거나 추가 HRMP 채널을 열기 위해 사용될 수 있습니다.

      ![RefundSurplus 및 DepositAsset 명령 및 설정](/media/images/docs/infrablockchain/tutorials/build/refund-and-deposit.png)

   - 또는 자산을 받는 계정을 **parents: 0**, **interior: X1**, **AccountId** 로 지정하고 자산을 수신할 네트워크 및 계정 주소를 식별할 수 있습니다.


### 전체 명령 세트 검토

위의 명령 세트는 다음 작업을 수행합니다:

- 파라체인 A의 sovereign 계정에서 holding 레지스터로 자산을 인출합니다.

- Holding 레지스터에 있는 자산을 사용하여 XCM 명령에 대한 수수료를 지불합니다.

- 릴레이 체인에서 HRMP 채널 요청을 실행합니다.

- 남은 자산을 환불하고 환불된 자산을 _soverign 게정_ 에 입금합니다.

추가 구성 및 구체적인 기술적 질문에 대한 답변을 위해 [Stack Exchange](https://substrate.stackexchange.com/)에서 다음 태그를 사용해 보세요:

- xcm
- hrmp
- weight
- cumulus

### 트랜잭션 제출

트랜잭션을 제출하려면:

1. **트랜잭션 제출** 을 클릭합니다.

1. **서명 및 제출**을 클릭합니다.

1. **네트워크**를 클릭하고 **탐색기**를 선택하여 메시지가 전송되었는지 확인합니다.

## 요청 확인

트랜잭션을 제출한 후, 릴레이 체인에서 메시지가 수신되고 실행되었는지 확인해야 합니다.

요청을 확인하려면 다음을 수행하세요:

1. 두 개의 별도 [브라우저 탭 또는 창](https://portal.infrablockspace.net/#/explorer/)을 열고, 하나의 인스턴스는 릴레이 체인에 연결하고 다른 인스턴스는 파라체인 엔드포인트에 연결합니다.

2. 파라체인에 연결된 브라우저 인스턴스에서 **네트워크**를 클릭한 다음 **탐색기**를 선택합니다.

3. 최근 이벤트 목록을 확인하고 **sudo.Sudid** 이벤트와 **ibsXcm.Sent** 이벤트가 있는지 확인합니다.

   ![파라체인에서 XCM 명령을 위한 이벤트](/media/images/docs/tutorials/parachains/hrmp-parachain-events.png)

   **ibsXcm.Sent** 이벤트를 확장하여 전송된 메시지에 포함된 명령에 대한 자세한 정보를 볼 수 있습니다.

4. 릴레이 체인에 연결된 브라우저 인스턴스에서 **네트워크**를 클릭한 다음 **탐색기**를 선택합니다.

3. 최근 이벤트 목록을 확인하고 **hrmp.OpenChannelRequested** 및 **ump.ExecutedUpward** 이벤트가 표시되는지 확인합니다.

   ![릴레이 체인에서 XCM 명령을 위한 이벤트](/media/images/docs/tutorials/parachains/hrmp-relay-chain-request-complete.png)

   다음 세션에서 파라체인 A (1000)에서 파라체인 B (1001)로 채널을 열었는지 확인하기 위해 **hrmpChannels**를 쿼리하여 체인 상태를 확인할 수 있습니다.

   ![Soverign 계정](/media/images/docs/infrablockchain/tutorials/build/soverign-account-balance.png)

## 승인 트랜잭션 준비

채널을 요청한 후에는 파라체인 B (1001)가 요청을 수락해야만 요청한 채널을 통해 메시지를 보낼 수 있습니다.
요청을 수락하기 위해 수행하는 단계는 채널 요청을 초기화하는 단계와 유사하지만, 다른 인코딩된 트랜잭션을 사용하고 파라체인 A 대신 파라체인 B에서 메시지를 보냅니다.

첫 번째 단계는 릴레이 체인에서 인코딩된 트랜잭션을 준비하는 것입니다.

인코딩된 호출을 준비하려면 다음을 수행하세요:

1. [인프라 블록체인(InfraBlockchain) Portal](https://portal.infrablockspace.net)을 열고 릴레이 체인에 연결합니다.

2. **개발자**를 클릭하고 **외부 트랜잭션**을 선택합니다.

3. **hrmp**를 선택한 다음 **hrmpAcceptOpenChannel(sender)** 를 선택하고 **sender**를 지정합니다. 이 경우 파라체인 A (1000)입니다.

4. **인코딩된 트랜잭션 데이터**를 복사합니다.

   ![인코딩된 트랜잭션 데이터 복사](/media/images/docs/infrablockchain/tutorials/build/hrmp-relay-chain-accept-request.png)

   이 정보는 XCM Transact 명령을 구성하는 데 사용됩니다.
   다음은 `infra-relay` 에서 인코딩된 호출 데이터의 예입니다: 0x3c01e8030000

## 채널 승인하기

인코딩된 트랜잭션을 보유하고 있으므로, 파라체인 A가 요청한 채널을 수락하기 위한 XCM 명령 세트를 구성할 수 있습니다.

1. 파라체인 B(1001)의 엔드포인트에 연결합니다. [인프라 블록체인 익스플로러](https://portal.infrablockspace.net/#/explorer/)을 사용합니다.

2. **개발자**를 클릭하고 **외부 트랜잭션**을 선택합니다.

3. **sudo**를 선택한 다음 트랜잭션을 실행하기 위해 **sudo(call)** 을 선택합니다.

4. **ibsXcm**을 선택한 다음 파라체인 B(1001)과 채널을 열기 위해 **send(dest, message)** 를 선택합니다.

5. 메시지를 전달할 상대 위치를 나타내기 위해 대상 매개변수를 지정합니다.

6. XCM 버전을 지정한 다음 실행할 메시지를 구성하기 위해 **항목 추가**를 클릭합니다.

   이전 메시지와 유사한 명령 세트를 사용할 수 있습니다:

   - *WithdrawAsset*: 지정된 온체인 자산을 가상 *holding 레지스터* 로 이동시킵니다.

   - *BuyExecution*: *WithdrawAsset* 명령을 사용하여 *holding 레지스터* 에 예치된 자산을 사용하여 현재 메시지의 실행 비용을 지불합니다.

   - *Transact*: 준비한 *인코딩된 트랜잭션* 을 지정합니다.

   - *RefundSurplus*: 실제 지불해야할 양보다 BuyExecution 명령을 사용하여 지불한 수수료가 작을 경우 나머지를 *refunded-weight* 레지스터로 이동시킵니다.

   - *DepositAsset*: *refunded-weight* 레지스터에서 자산을 빼내고, 지정 계정에 입금시킵니다.

7. **트랜잭션 제출**을 클릭합니다.

   트랜잭션을 제출한 후, 다음 에포크에서 변경 사항이 체인 상태에 반영되기까지 기다려야 합니다.

## 채널 확인하기

트랜잭션을 제출한 후, 릴레이 체인에서 메시지가 수신되고 실행되었는지 확인할 수 있습니다.

요청을 확인하려면 다음을 수행하세요:

1. 두 개의 별도 [브라우저 탭 또는 창](https://portal.infrablockspace.net/#/explorer/)을 열고, 하나의 인스턴스는 릴레이 체인에 연결하고 다른 인스턴스는 파라체인 엔드포인트에 연결합니다.
   
2. 파라체인에 연결된 브라우저 인스턴스에서 **네트워크**를 클릭한 다음 **탐색기**를 선택합니다.

3. 최근 이벤트 목록을 확인하고 **sudo.Sudid** 이벤트와 **ibsXcm.Sent** 이벤트가 있는지 확인합니다.

4. 릴레이 체인에 연결된 브라우저 인스턴스에서 **네트워크**를 클릭한 다음 **탐색기**를 선택합니다.

5. 최근 이벤트 목록을 확인하고 **hrmp.OpenChannelAccepted** 및 **ump.ExecutedUpward** 이벤트가 표시되는지 확인합니다.

   ![릴레이 체인에서 XCM 명령을 위한 이벤트](/media/images/docs/tutorials/parachains/hrmp-relay-chain-request-accepted.png)

   다음 에포크 시작 후에는 **hrmpChannels**를 쿼리하여 파라체인 A(1000)에서 파라체인 B(1001)로 채널을 열었는지 확인할 수 있습니다.
   
   ![HRMP 채널 쿼리](/media/images/docs/infrablockchain/tutorials/build/hrmp-verify-channel.png)

## 두 번째 채널 열기

파라체인 B(1001)로만 XCM 명령을 보내려는 경우 작업이 완료되었습니다. 그러나 두 체인이 서로가 보낸 명령을 수신하고 실행할 수 있는 양방향 통신을 활성화하려면 방금 파라체인 A에 대해 완료한 작업을 파라체인 B에 대해 다시 수행해야 합니다. 파라체인 A가 채널 요청을 수락한 후, 두 체인은 XCM 명령을 보내고 받을 수 있습니다.

인코딩된 트랜잭션을 준비하고, 채널 요청을 보내고, 요청을 수락하기 위해 파라체인 B를 위해 방금 완료한 모든 단계를 반복하여 파라체인 B(1001)에서 파라체인 A(1000)로의 통신을 활성화합니다. 파라체인 B에서 파라체인 A로 채널을 열면 두 파라체인은 서로에게 직접 메시지를 보낼 수 있거나 메시지를 릴레이 체인을 통해 라우팅할 수 있습니다.

이 시점에서 파라체인 간에 XCM 명령을 보낼 수 있지만, 원격 체인에서 성공적으로 실행될 수 있는 메시지를 구성하려면 자산을 전송할 수 있는 공유된 이해관계를 정의하거나 자산을 이동할 수 있는 상호 신뢰 관계를 정의하기 위해 추가 구성이 필요합니다.

다음 튜토리얼에서는 파라체인 간 통신의 몇 가지 일반적인 시나리오에 대한 예제를 살펴보겠습니다.

## 다음 단계로 넘어가기

- [XCM Part I: The Cross-Consensus Message Format](https://polkadot.network/blog/xcm-the-cross-consensus-message-format)
- [XCM Part II: Versioning and Compatibility](https://polkadot.network/blog/xcm-part-two-versioning-and-compatibility)
- [XCM Part III: Execution and Error Management](https://polkadot.network/blog/xcm-part-three-execution-and-error-management)