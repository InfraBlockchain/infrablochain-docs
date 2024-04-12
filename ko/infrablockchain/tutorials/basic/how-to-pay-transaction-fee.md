---
title: 시스템 토큰 등록 및 수수료 사용해보기
description: 이 튜토리얼은 토큰 생성부터 시스템 토큰 등록을 위한 거버넌스, 시스템 토큰 사용까지의 일련의 과정에 대해 배웁니다.
keywords:
  - Assets
  - System token manager
  - Governance
  - Transaction fee
---

이 튜토리얼은 멀티체인 환경에서 발행한 토큰을 시스템 토큰으로 등록하고 가스비로 사용하는 방법에 대해 배워보겠습니다.

## 튜토리얼 목표

이 튜토리얼을 완료함으로써 다음 목표를 달성할 수 있습니다:

- 좀비넷을 활용하여 `릴레이체인 <> 파라체인` 테스트 네트워크를 구축할 수 있습니다.
- 파라체인에서 토큰을 발행하고 릴레이체인에서 해당 토큰을 시스템 토큰으로 등록하기 위한 거버넌스를 발의할 수 있습니다.
- 거버넌스가 통과된 후, 파라체인에서 발행한 시스템 토큰으로 가스비를 낼 수 있습니다.

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [좀비넷을 이용한 파라체인 네트워크 시뮬레이션 하기](./test/simulate-parachains.md)

- [시스템 토큰 등록 절차](./how-to-interact-with-system-token.md) 를 통해 시스템 토큰으로 트랜잭션 수수료를 지불할 준비를 마칩니다.

## 시스템 토큰 가스비로 사용하기

아래처럼 Assets 의 `transfer` 트랜잭션을 호출해 보겠습니다:

![parachain_asset_transfer](/media/images/docs/infrablockchain/tutorials/parachain_asset_transfer.png)

트랜잭션 수수료로 사용할 토큰을 지정합니다.

![send_transaction](/media/images/docs/infrablockchain/tutorials/send_transaction.png)

트랜잭션이 정상적으로 실행되면 다음과 같이 시스템 토큰으로 수수료를 지불했다는 이벤트를 볼 수 있습니다.

![tx_fee_paid_event](/media/images/docs/infrablockchain/tutorials/tx_fee_paid_event.png)
