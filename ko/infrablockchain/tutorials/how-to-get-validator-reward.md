---
title: 벨리데이터 보상 받기
description: 이 튜토리얼은 인프라 블록체인의 벨리데이터가 보상을 받는 방법을 설명합니다.
keywords:
  - 시스템 토큰
  - 벨리데이터
---

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [시스템 토큰 등록 및 수수료로 사용해보기](/ko/infrablockchain/tutorials/how-to-pay-transaction-fee.md)

## 개요

인프라 블록체인에서는 벨리데이터에게 블록 생성 및 검증에 대한 댓가로 보상을 지급합니다. 벨리데이터에게 지급되는 보상은 유저들이 제출한 트랜잭션의 수수료로 구성됩니다. 

![validator-reward-process](/media/images/docs/infrablockchain/tutorials/validator-reward-process.png)


이 문서에서는 벨리데이터가 보상을 받는 방법에 대해서 설명합니다.

## 벨리데이터 보상 확인하기

`ValidatorReward` 스토리지를 조회하여 벨리데이터가 얼마나 보상을 받을 수 있는지 확인할 수 있습니다.

1. [Portal](https://portal.infrablockspace.net)에 접속합니다.


2. `개발자` > `체인 상태` > `스토리지`에 접속합니다. 


3. `validatorRewardManager` 팔레트의 `validatorReward` 를 선택합니다.


4. 조회하고자 하는 주소를 넣어 확인합니다.

![storage](/media/images/docs/infrablockchain/tutorials/validator-reward-storage.png)

## 벨리데이터 보상 받기

위 과정을 통해서 벨리데이터가 받을 보상이 있다는 것이 확인 되었다면 실제로 보상을 받기 위한 트랜잭션을 발생 시킬 수 있습니다.

1. [Portal](https://portal.infrablockspace.net)에 접속합니다.


2. `개발자` > `익스트랜식` > `Submission`에 접속합니다. 


3. `validatorRewardManager` 팔레트의 `claim` 를 선택합니다.


4. 적절한 값을 넣어 트랜잭션을 제출합니다.

![claim](/media/images/docs/infrablockchain/tutorials/reward-claim.png)

`validatorRewardManager` 팔레트의 `claim` 익스트린식은 벨리데이터의 계정과 시스템 토큰 ID를 입력으로 받습니다. 

`claim` 익스트린식은 벨리데이터 본인만 수행할 수 있습니다.

또한 한번의 익스트린식에 한 종류의 시스템 토큰 ID에 대해서만 보상을 받을 수 있습니다. 만약 벨리데이터가 받을 수 있는 토큰이 여러 종류가 있다면, 여러번 익스트린식을 발생 시켜야 합니다.