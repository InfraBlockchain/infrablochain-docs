---
title: Proof-of-Transaction
description: 이 튜토리얼은 인프라 블록체인의 독자적인 합의 알고리즘인 PoT 에 대해 배웁니다.
keywords:
  - Proof-of-Transaction
  - 시스템 토큰
---

## Transaction-as-a-Vote(TaaV)

인프라 블록체인의 독자적인 합의 메커니즘인 **_PoT(Proof-of-Transaction)_** 를 이루는 핵심 아이디어는 _Taav(Transaction-as-a-Vote)_ 입니다. 인프라 블록체인의 트랙잭션 메타데이터에는 블록체인 합의 과정에 참여할 수 있거나 이미 참여하고 있는 블록 생성자 후보에 대한 투표(vote)가 선택적으로 포함될 수 있습니다. 블록 생성자 투표가 담긴 트랜잭션 메세지는 트랜잭션을 발생시킨 블록체인 계정의 개인키로 서명이 이루어져 각 트랜잭션 투표마다 자체적인 암호학적 증명을 가집니다. **_인프라 블록체인(InfraBlockchain)_** 의 투표는 다음과 같은 특성을 가지고 있습니다:

- 시스템 토큰 식별자: 투표의 가중치(weight)는 어떠한 [시스템 토큰](../learn/system-token.md) 으로 트랜잭션 수수료로 지불했는지에 따라 다르기 때문에 해당 토큰을 식별할 수 있는 `id` 가 포함됩니다.

- 투표 대상: 투표할 대상은 블록체인 계정을 갖고 있어야 하며 그 타입은 **_인프라 릴레이체인(InfraRelaychain)_** 의 [블록체인 계정](../../learn/accounts-addresses-keys.ko.md) 이여야 합니다. 보통은 `AccountId32` 입니다.

- 트랜잭션 가중치(Weight): 트랙잭션 투표의 가중치는 트랜잭션 수수료에 따라 다르게 책정됩니다. 

```rust 
// For example,
pub struct PoTVote {
    system_token_id: u32,
    account_id: infra_relay::AccountId,
    vote_weight: u128
}
```

### 트랜잭션 메타데이터



## Aggregated Proof-of-Transaction(PoT)

**_인프라 블록체인(InfraBlockchain)_** 은 **_인프라 릴레이체인(InfraRelaychain)_** 을 중심으로 여러 개의 인프라 블록체인 블록들이 병렬적으로 실행되는 멀티체인 아키텍처입니다. 각 블록마다 블록 생성자(밸리데이터) 후보에 대한 투표가 선택적으로 포함되어 있고 **_인프라 블록스페이스(InfraBlockspace)_** 안에서 블록이 실행될 때 해당 투표들이 수집되어 **_인프라 릴레이체인(InfraRelayChain)_** 의 한 상태로 저장됩니다. 특정 시점이 지나면 블록체인 프로토콜에 의해 수집된 투표를 바탕으로 블록 생성자 후보들에 대한 집계가 이루어지고 투표를 많이 받은 후보가 블록 생성자로 선출되게 됩니다. 