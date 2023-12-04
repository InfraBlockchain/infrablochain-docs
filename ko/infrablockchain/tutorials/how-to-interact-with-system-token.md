---
title: 시스템 토큰 관리 프로세스 배우기
description: 시스템 토큰과 관련된 여러 절차에 대한 전반적인 내용을 다룹니다.
keywords:
---

***인프라 블록체인(InfraBlockchain)*** 은 시스템 토큰을 가스비로 사용하는 차별화된 블록체인 시스템을 제공합니다. 
아래는 시스템 토큰의 등록, 사용 및 폐기과정에 대해 개발자들이 쉽게 이해하고 개발할 수 있도록 정리한 내용입니다.
해당 내용에 대한 실습은 [how-to-pay-transaction-fee](./how-to-pay-transaction-fee.md) 에서 진행할 수 있습니다.

## Bootstrap 과정

- Test Token 활용: Bootstrap 중에는 가스비로 사용 가능한 "test token"이 존재합니다. 

    ![test-token](/media/images/docs/infrablockchain/tutorials/test-token.png)

- 첫 시스템 토큰 승인: 첫 시스템 토큰이 승인 및 유통된 후 bootstrap이 완료됩니다.
- Test Token 소각: Bootstrap 완료 후 테스트 토큰은 모두 소각되며, 더 이상 가스비로 사용할 수 없습니다


## 시스템 토큰 등록

- 토큰 생성 및 발행: 특정 체인에서 토큰을 생성 및 발행합니다.

    ![create_token](/media/images/docs/infrablockchain/tutorials/create_token.png)

    ![mint_token](/media/images/docs/infrablockchain/tutorials/mint_token.png)

- 거버넌스 제출: 토큰의 오너는 해당 토큰을 시스템 토큰으로 등록하기 위해 릴레이 체인 거버넌스를 제출합니다.

    ![register_system_token1](/media/images/docs/infrablockchain/tutorials/register_system_token1.png)

- 시스템 토큰 투표: 릴레이 체인의 밸리데이터들은 거버넌스에 참여하며, 2/3 이상이 동의할 경우 해당 시스템 토큰이 등록됩니다.

    ![governance_voting](/media/images/docs/infrablockspace/tutorials/governance_voting.png)

- 트랜잭션 수수료 사용: 해당 토큰이 시스템 토큰으로 등록되면, 토큰을 발행한 체인에서 시스템 토큰을 트랜잭션 수수료로 사용할 수 있게 됩니다.

    ![parachain_sufficient_true](/media/images/docs/infrablockchain/tutorials/parachain_sufficient_true.png)

## 다른 parachain에서 시스템 토큰 사용

- _Wrapped 시스템 토큰_ 사용 승인: 다른 체인에서 시스템 토큰을 가스비로 사용하고자 할 때 해당 토큰의 "wrapped 시스템 토큰" 사용에 대한 안건을 거버넌스에 제출해야 합니다.
    
    ![register-wrapped](/media/images/docs/infrablockchain/tutorials/register-wrapped.png)

- 거버넌스 승인: 해당 안건이 거버넌스에 승인되면, 해당 체인에서는 _Wrapped 시스템 토큰_ 을 가스비로 사용할 수 있게 됩니다.

## 시스템 토큰 Suspend 및 Deregister

- 문제 발생 시 조치: 파라체인이나 시스템 토큰 등에 문제가 발생하면, 생태계에 해를 끼치지 않도록 시스템 토큰 혹은 _Wrapped 시스템 토큰_ 을 일시 사용 금지시키거나 영구 삭제할 수 있습니다.

    ![suspend](/media/images/docs/infrablockchain/tutorials/suspend.png)

    ![deregister](/media/images/docs/infrablockchain/tutorials/deregister.png)
    
- 거버넌스 승인 필요: 해당 시스템 토큰의 사용 중지는 거버넌스의 승인을 받아야 하며, 승인된 후에는 해당 시스템 토큰 및 _Wrapped 시스템 토큰_ 은 가스비로 사용될 수 없습니다.
