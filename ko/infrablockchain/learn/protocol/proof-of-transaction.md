---
title: 트랜잭션 증명(Proof-of-Transaction, PoT)
description: 인프라블록체인만의 독자적인 합의 메커니즘인 PoT에 대한 전반적인 내용을 다룹니다.
keywords:
  - 트랜잭션 증명
  - 보상
  - 시스템 토큰
---

## Transaction-as-a-Vote(TaaV)

![트랜잭션 투표](/media/images/docs/infrablockchain/learn/protocol/taav.png)

인프라블록체인의 독자적인 합의 메커니즘인 **_PoT(Proof-of-Transaction)_** 를 이루는 핵심 아이디어는 **Taav(Transaction-as-a-Vote)** 입니다. 인프라블록체인의 트랜잭션 메타데이터에는 블록체인 합의 과정에 참여할 수 있거나, 이미 참여하고 있는 블록 생성자 후보에 대한 투표(vote)가 선택적으로 포함될 수 있습니다. 블록 생성자 투표가 담긴 트랜잭션 메세지는 트랜잭션을 발생시킨 블록체인 계정의 개인키로 서명이 이루어져 각 트랜잭션 투표마다 자체적인 암호학적 증명을 가집니다. **_인프라블록체인(InfraBlockchain)_** 의 투표는 다음과 같은 특성을 가지고 있습니다:

- **시스템 토큰 식별자**: 투표의 가중치(weight)는 어떠한 [시스템 토큰](./system-token.md) 으로 트랜잭션 수수료를 지불했는지에 따라 다르기 때문에 해당 토큰을 식별할 수 있는 `id` 가 포함됩니다.

- **투표 대상**: 투표할 대상은 블록체인 계정을 갖고 있어야 하며 그 타입은 **_인프라릴레이체인(InfraRelaychain)_** 의 [블록체인 계정](../substrate/learn/basic/accounts-addresses-keys.md) 이여야 합니다. 보통은 `AccountId32` 입니다.

- **트랜잭션 가중치(Weight)**: 트랜잭션 투표의 가중치는 트랜잭션 수수료에 따라 다르게 책정됩니다.

```rust
// For example,
pub struct PoTVote {
    system_token_id: u32,
    account_id: infra_relay::AccountId,
    vote_weight: u128
}
```

### 트랜잭션 메타데이터

Transaction-as-a-Vote(TaaV) 는 트랜잭션의 메타데이터에 투표를 포함합니다. Substrate 에서는 `SignedExtension` 트레이트를 이용하여 트랜잭션 메타데이터를 추가할 수 있습니다.
다음은 해당 트레이트를 이용한 예시입니다:

```rust
// 트랜잭션에 들어갈 메타데이터 타입 지정
// 투표 대상이 선택적으로 포함될 수 있다.
pub struct ChargeSystemToken<Account, Balance> {
    #[codec(compact)]
    tip: Balance,
    system_token_id: Option<u32>,
    vote_candidate: Option<Account>
}

// 트랜잭션을 처리할 때, 수행되어야하는 로직 작성
impl SignedExtension for ChargeSystemToken {
    ...
    pub fn post_dispatch(..) {
        // Handling votes
    }
}
```

## 블록 생성자(밸리데이터) 풀

**_인프라블록체인(InfraBlockchain)_** 은 기관 및 공공기관을 위한 엔터프라이즈 블록체인으로써, 어떠한 상황에서도 정직하게 노드가 동작하여 BFT 합의를 이루어 나가야 합니다. 또한 공개형/허가형 하이브리드 블록체인으로써 어떤 주체든 블록 생성자로 선출될 수 있어야 합니다. 이러한 이상적인 속성을 설계하기 위해 **_인프라블록체인(InfraBlockchain)_**에는 다음과 같은 두 개의 블록 생성자(밸리데이터) 풀이 존재합니다:

- **Proof-of-Transaction Node Pool**: PoT에 의해 선출된 노드를 관리하는 풀입니다.

- **Seed Trust Node Pool**: 금융 기관이나 정부 조직과 같이 어떠한 상황에서도 정직한 노드를 운영하는 노드를 관리하는 풀입니다.

![밸리데이터 풀](/media/images/docs/infrablockchain/learn/protocol/validator-pool.png)

**_인프라블록체인(InfraBlockchain)_** 의 초기 밸리데이터는 Seed Trust 밸리데이터로 구성되어 허가형 블록체인을 형성합니다. 네트워크가 안정됨에 따라 PoT 컨센서스 메커니즘 을 이용하여 누구나 블록 생성자로 참여할 수 있는 퍼블릭 블록체인으로 전환될 수 있습니다.

## Aggregated Proof-of-Transaction(PoT)

**_인프라블록체인(InfraBlockchain)_** 은 **_인프라릴레이체인(InfraRelaychain)_** 을 중심으로 여러 개의 파라체인 블록들이 병렬적으로 실행되는 멀티체인 아키텍처입니다. **_인프라릴레이체인(InfraRelayChain)_** 밸리데이터는 각 파라체인 블록을 검증하고 해당 블록에 포함된 투표를 수집하는 역할을 수행하며 이를 **Aggregated Proof-of-Transaction** 이라 합니다.

각 블록마다 **_인프라릴레이체인(InfraRelaychain)_** 의 블록 생성자(밸리데이터) 후보에 대한 투표가 선택적으로 포함되어 있고 검증 과정에서 해당 투표들이 수집되어 **_인프라릴레이체인(InfraRelayChain)_** 의 한 상태로 저장됩니다. 특정 시점이 지나면 수집된 투표를 바탕으로 블록 생성자 후보들에 대한 집계가 이루어지고 투표를 많이 받은 후보가 블록 생성자로 선출되게 됩니다.

```rust
#[pallet::storage]
#[pallet::unbounded]
pub type PotValidatorPool<T: Config> = StorageValue<_, VotingStatus<T>, ValueQuery>;

#[derive(Encode, Decode, Clone, PartialEq, RuntimeDebug, TypeInfo)]
#[scale_info(skip_type_params(T))]
pub struct VotingStatus<T: Config> {
	pub status: Vec<(T::InfraVoteAccountId, T::InfraVotePoints)>,
}
```

## 시간에 따라 증가하는 PoT 투표 가중치(Block time weight)

최근 트랜잭션 투표에 더 많은 가중치를 두기 위해 **블록 시간 가중치(Block time weight)** 가 곱해집니다. 가중치는 1년에 2배의 비율로 증가합니다. 기준 시간은 _인프라릴레이체인_ 의 제네시스 블록(0번째 블록)이며, 릴레이체인 블록 기준으로 가중치가 곱해집니다. 블록 시간이 반영된 트랜잭션 투표 가중치의 정확한 계산은 아래와 같이 이뤄집니다:

예를 들어,

```
- 1년에 해당하는 릴레이체인 블록 수(6초당 한 블럭) = 5_256_000

- 블록 시간이 반영된 트랜잭션 투표 가중치 = 2^(현재 릴레이체인 블록 / 1년에 해당하는 릴레이체인 블록 수)
```

## 다음 단계로 넘어가기

- [PoT로 밸리데이터 선출하기](../../tutorials/basic/how-to-vote-with-taav.md)
- [시스템 토큰 알아보기](./system-token.md)
- [ParasInclusion](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/master/infrablockspace/runtime/parachains/src/inclusion/mod.rs)
