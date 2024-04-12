---
title: 밸리데이터 보상 받기
description: 이 튜토리얼은 인프라블록체인의 밸리데이터가 보상을 받는 방법을 설명합니다.
keywords:
  - 시스템 토큰
  - 밸리데이터
---

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [시스템 토큰 등록 및 수수료로 사용해보기](./how-to-pay-transaction-fee.md)

## 개요

인프라블록체인에서는 밸리데이터에게 블록 생성 및 검증에 대한 댓가로 보상을 지급합니다. 밸리데이터에게 지급되는 보상은 유저들이 제출한 트랜잭션의 수수료로 구성됩니다.

![validator-reward-process](/media/images/docs/infrablockchain/tutorials/validator-reward-process.png)

이 문서에서는 밸리데이터가 보상을 받는 방법에 대해서 설명합니다.

## 보상 체계

인프라 릴레이체인의 밸리데이터는 각 체인에서 발생한 트랜잭션 수수료에 대한 일부를 보상으로 받게 됩니다. 보상이 발생할때마다 릴레이 체인에서는 다음과 같은 이벤트를 확인할 수 있습니다. 해당 이벤트에는 어느 시점(era)에 어떤 토큰(asset)으로 얼마 만큼의 보상이 발생했는지에 대한 정보를 확인할 수 있습니다.

![validator_reward_event](/media/images/docs/infrablockchain/tutorials/validator_reward_event.png)

## 밸리데이터 보상 확인하기

`ValidatorManagement` 스토리지를 조회하여 밸리데이터가 얼마나 보상을 받을 수 있는지 확인할 수 있습니다.

1. [Portal](https://portal.infrablockspace.net)에 접속합니다.

2. `개발자` > `체인 상태` > `스토리지`에 접속합니다.

3. `validatorManagement` 팔레트의 `RewardInfo` 를 선택합니다.

4. 조회하고자 하는 주소를 넣어 확인합니다.

![reward_info](/media/images/docs/infrablockchain/tutorials/reward_info.png)

## 밸리데이터 보상 받기

위 과정을 통해서 밸리데이터가 받을 보상이 있다는 것이 확인 되었다면, 실제로 보상을 받기 위한 트랜잭션을 발생 시킬 수 있습니다.

1. [Portal](https://portal.infrablockspace.net)에 접속합니다.

2. `개발자` > `익스트랜식` > `Submission`에 접속합니다.

3. `validatorManagement` 팔레트의 `claim` 를 선택합니다.

4. 적절한 값을 넣어 트랜잭션을 제출합니다.

![claim_reward](/media/images/docs/infrablockchain/tutorials/claim_reward.png)

_한 번의 익스트린식에 한 종류의 시스템 토큰 ID에 대해서만 보상을 받을 수 있습니다. 만약 밸리데이터가 받을 수 있는 토큰이 여러 종류가 있다면, 익스트린식을 여러번 발생 시켜야 합니다._
