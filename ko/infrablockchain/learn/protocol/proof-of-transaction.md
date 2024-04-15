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

인프라블록체인의 독자적인 합의 메커니즘인 **_PoT(Proof-of-Transaction)_** 의 핵심 개념은 **Transaction-as-a-Vote** 입니다. 인프라블록체인 내 들어오는 트랜잭션 메타데이터에는 블록 생성자 후보 계정에 대한 정보가 포함될 수 있습니다. 후보 계정 정보가 담긴 트랜잭션 메세지는 트랜잭션을 발생시킨 주체의 개인키로 서명이 이루어져 자체적인 암호학적 증명을 가집니다. 또한 트랜잭션을 수행하게 되면 트랜잭션 수수료가 발생하게 되고 블록 생성자에 대한 보상과 트랜잭션 호출자에게 환불되는 양을 제외한 나머지는 인프라 릴레이 체인 밸리데이터에 대한 보상으로 돌아갑니다.

## 트랜잭션 메타데이터

Substrate 에서는 필수적으로 트랜잭션을 검증하기 위한 요소들을 제외하고 추가적으로 각 체인별로 검증 요소들을 자유롭게 추가할 수 있습니다. _TaaV_ 를 구현하기 위해 인프라블록체인 트랜잭션 메타데이터로 어떠한 시스템 토큰으로 수수료를 지불할지에 대한 토큰 식별자(`asset_id`) 와 릴레이 체인 밸리데이터 후보 계정에 대한 정보를 추가하였습니다.

```rust
pub struct ChargeSystemToken<AssetId, Account> {
    #[codec(compact)]
    tip: Balance,
    asset_id: Option<AssetId>,
    candidate: Option<Account>
}
```

- `asset_id`: 시스템 토큰 식별자를 나타냅니다.
- `candidate`: 투표 대상에 대한 계정을 나타냅니다.

위의 메타데이터가 포함된 트랜잭션이 들어오면 트랜잭션 수수료량에 기반한 투표량이 결정됩니다. 다음은 투표 구조체를 나타냅니다:

### 투표

```rust
// For example,
pub struct Vote<Account, Weight> {
  pub candidate: Account,
  pub amount: Weight,
  }
```

- **투표 대상**: 투표할 대상, 즉 릴레이 체인 블록 생성자 후보, 를 나타냅니다.

- **투표량**: 트랜잭션 수수료에 따른 투표량을 나타냅니다.

### 보상

트랜잭션 수수료에 대한 일부에 대한 보상을 나타냅니다. 해당 보상은 릴레이 체인 밸리데이터에게 분배됩니다.

```rust
pub struct Reward<DestId, AssetId, Amount> {
  pub origin: RewardOrigin<DestId>,
  pub asset: AssetId,
  pub amount: Amount,
}
```

- `origin`: 보상이 발생한 위치를 나타냅니다. 트랜잭션마다 발생한 보상은 릴레이 체인 `ValidatorManagement` 모듈에서 관리됩니다. 릴레이 체인 밸리데이터가 `claim()` 트랜잭션을 호출하게 되면 보상이 발생했던 체인에서 해당 계정으로 분배됩니다.

- `asset`: 트랜잭션 수수료를 지불했던 토큰 식별자를 나타냅니다.

- `amount`: 보상에 대한 양을 나타냅니다.

## Proof of Transaction

트랜잭션에 대한 증명으로 매 트랜잭션 마다 **투표** 와 **보상** 이 나옵니다. 이는 모두 릴레이 체인에서 파라체인 블록 혹은 자신의 블록을 검증할 때 `ValidatorManagement` 모듈에 집계됩니다. 다음은 `Proof-of-Transaction` 데이터 타입입니다:

```rust
#[derive(Encode)]
pub struct PoT<Account, DestId, AssetId, Amount, Weight> {
  /// Reward amount for each transaction
  pub reward: Reward<DestId, AssetId, Amount>,
  /// Amount of vote for `Account` based on `fee_amount`
  pub vote: Option<Vote<Account, Weight>>,
}
```

- `reward`: 매 트랜잭션마다 발생하며 릴레이 체인에서 집계됩니다.

- `vote` : `Option<_>` 타입으로 트랜잭션안에 포함되어 있으면 릴레이 체인에서 집계하여 관리합니다.

## 블록 생성자(밸리데이터) 풀

**_인프라블록체인(InfraBlockchain)_** 은 기관 및 공공기관을 위한 엔터프라이즈 블록체인으로써, 어떠한 상황에서도 정직하게 노드가 동작하여 BFT 합의를 이루어 나가야 합니다. 또한 공개형/허가형 하이브리드 블록체인으로써 어떤 주체든 블록 생성자로 선출될 수 있어야 합니다. 이러한 이상적인 속성을 설계하기 위해 **_인프라블록체인(InfraBlockchain)_**에는 다음과 같은 두 개의 블록 생성자(밸리데이터) 풀이 존재합니다:

- **Proof-of-Transaction Node Pool**: PoT에 의해 선출된 노드를 관리하는 풀입니다.

- **Seed Trust Node Pool**: 금융 기관이나 정부 조직과 같이 어떠한 상황에서도 정직한 노드를 운영하는 노드를 관리하는 풀입니다.

![밸리데이터 풀](/media/images/docs/infrablockchain/learn/protocol/validator-pool.png)

**_인프라블록체인(InfraBlockchain)_** 의 초기 밸리데이터는 Seed Trust 밸리데이터로 구성되어 허가형 블록체인을 형성합니다. 네트워크가 안정됨에 따라 PoT 컨센서스 메커니즘 을 이용하여 누구나 블록 생성자로 참여할 수 있는 퍼블릭 블록체인으로 전환될 수 있습니다.

## Aggregated Proof-of-Transaction(PoT)

**_인프라블록체인(InfraBlockchain)_** 은 **_인프라릴레이체인(InfraRelaychain)_** 을 중심으로 여러 개의 파라체인 블록들이 병렬적으로 실행되는 멀티체인 아키텍처입니다. **_인프라릴레이체인(InfraRelayChain)_** 밸리데이터는 각 파라체인 블록을 검증하고 해당 블록에 포함된 투표를 수집하는 역할을 수행하며 이를 **Aggregated Proof-of-Transaction** 이라 합니다.

각 블록마다 **_인프라릴레이체인(InfraRelaychain)_** 의 블록 생성자(밸리데이터) 후보에 대한 투표가 선택적으로 포함되어 있고 검증 과정에서 해당 투표들이 수집되어 **_인프라릴레이체인(InfraRelayChain)_** 의 한 상태로 저장됩니다. 특정 시점이 지나면 수집된 투표를 바탕으로 블록 생성자 후보들에 대한 집계가 이루어지고 투표를 많이 받은 후보가 블록 생성자로 선출되게 됩니다.

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
