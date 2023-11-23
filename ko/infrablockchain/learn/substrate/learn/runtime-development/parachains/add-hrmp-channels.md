---
title: HRMP 채널 추가하기
keywords:
  - 파라체인
  - 채널
  - HRMP
  - XCM
  - 크로스체인
---

이 가이드에서는 파라체인 B가 파라체인 A와 채널을 수락하도록 요청하는 방법을 배우게 됩니다.
HRMP 채널에 대해 자세히 알아보려면 [Polkadot 위키](https://wiki.polkadot.network/docs/build-hrmp-channels)를 참조하세요.

**채널은 단방향입니다**

일반적인 흐름은 채널을 열고자 하는 파라체인 중 하나가 채널을 열기 위해 다른 파라체인에 요청을 보내야 합니다.
그런 다음, 다른 파라체인은 이 채널 개방 요청을 수락해야 하며, 이는 기본적으로 단방향 채널을 의미합니다.
양방향 통신을 위해서는 반대로 또 다른 채널을 열어야 합니다.

채널은 수신자가 확인한 후에만 열 수 있으며, 세션 변경 시에만 열립니다.

이 가이드를 위해 다음과 같은 환경을 가정합니다:

- 릴레이 체인
- ParaId가 2000인 파라체인 A
- ParaId가 3000인 파라체인 B

## 파라체인 A의 동작

1. 릴레이 체인에서 다음 호출을 준비합니다:

   ```
   hrmp.hrmpInitOpenChannel(
       recipient: 3000                    // 채널을 열고자 하는 다른 파라체인
       proposedMaxCapacity: 1000          // 채널에 동시에 존재할 수 있는 메시지의 최대 개수를 지정합니다
       proposed_max_message_size: 102400  // 메시지의 최대 크기를 지정합니다
   )
   ```

   이 숫자들은 릴레이 체인의 구성 제한 사항에 따라 달라질 수 있습니다.

   원하는 매개변수를 설정한 후, 인코딩된 호출 데이터를 저장합니다.
   예를 들어, Rococo에서 이 호출의 인코딩된 호출 데이터는 다음과 같습니다: [`0x3c00b80b0000e803000000900100`](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frococo-rpc.polkadot.io#/extrinsics/decode/0x3c00b80b0000e803000000900100).

1. 이제 파라체인에서는 XCM 메시지를 구성하여 릴레이 체인에게 파라체인 B와의 채널 개방을 알리도록 해야 합니다.

   다음과 같이 메시지를 구성해야 합니다 (이전 단계에서 인코딩된 호출 데이터를 사용):

   ```
   polkadotXcm.send(
       dest: V1
           parents: 1
           interior: Here
       message: V2
           XcmV2Instruction: WithdrawAsset
               id: Concrete
                   parents: 0
                   interior: Here
               fun: Fungible
                   Fungible: 1_000_000_000_000
           XcmV2Instruction: BuyExecution
               id: Concrete
                   parents: 0
                   interior: Here
               fun: Fungible
                   Fungible: 1_000_000_000_000
               weightLimit: Unlimited
           XcmV2Instruction: Transact
               originType: Native
               requireWeightAtMost: 4_000_000_000
                   encoded: 0x3c00b80b0000e803000000900100 // hrmpInitOpenChannel의 인코딩된 호출 데이터
           XcmV2Instruction: DepositAsset
             assets: Wild::All
             maxAssets: 1
             beneficiary:
               parents: 0
               interior: Parachain(2000)           
   )
   ```

   주의할 점은 활성화된 XCM 구성을 고려하여 메시지를 구성해야 한다는 것입니다. 이는 단지 예시입니다.

## 파라체인 B의 동작

지금까지 파라체인 A는 자신의 역할을 수행했습니다: 파라체인 B에 대한 요청이 전송되었습니다.
이제 이 요청은 파라체인 B에서 수락되어야 합니다.

과정은 반복되지만, 이번에는 다른 호출을 인코딩해야 합니다.

1. 릴레이 체인에서 다음 익스트린식을 제출하고, 인코딩된 호출 데이터를 저장합니다:

   ```
   hrmp.hrmpAcceptOpenChannel(
       sender: 2000
   )
   ```

   Rococo에서의 인코딩된 호출 데이터는 다음과 같습니다: [`0x1701d0070000`](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frococo-rpc.polkadot.io#/extrinsics/decode/0x1701d0070000)

1. 이제 파라체인 B에서는 마지막 XCM 메시지를 기반으로 이 호출을 구성하고, 이전 단계에서의 인코딩된 호출 데이터를 사용하여 릴레이 체인에서 호출로 실행합니다.

   ```
   polkadotXcm.send(
       dest: V1
           parents: 1
           interior: Here
       message: V2
           XcmV2Instruction: WithdrawAsset
               id: Concrete
                   parents: 0
                   interior: Here
               fun: Fungible
                   Fungible: 1_000_000_000_000
           XcmV2Instruction: BuyExecution
               id: Concrete
                   parents: 0
                   interior: Here
               fun: Fungible
                   Fungible: 1_000_000_000_000
               weightLimit: Unlimited
           XcmV2Instruction: Transact
               originType: Native
               requireWeightAtMost: 4_000_000_000
                   encoded: 0x1701d0070000 // hrmpAcceptOpenChannel의 인코딩된 호출 데이터

   )
   ```

활성화된 XCM 구성을 고려하여 메시지를 구성해야 한다는 점을 기억하세요. 이는 단지 예시입니다.

그리고 이것으로 끝입니다. 채널이 수락되었으며, 이제 파라체인 A에서 파라체인 B로의 통신이 가능한 상태로 유지됩니다.
양방향 채널로 만들기 위해서는 파라체인 B에서 파라체인 A로의 또 다른 채널을 열어야 합니다.
이 가이드에서 설명한 단계를 따라 B에서 요청을 만들고 A에서 수락함으로써 이를 수행할 수 있습니다.