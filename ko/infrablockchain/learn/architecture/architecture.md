---
title: 인프라블록체인 아키텍처
description: 멀티체인 아키텍처에 대한 전반적인 내용을 다룹니다.
keywords:
  - 멀티체인
  - 릴레이 체인
  - 파라체인
---

![](/media/images/docs/infrablockchain/learn/architecture/relay-chain.png)

## Blockspace

**Blockspace** 는 블록이 확정되고 실행될 수 있는 추상적인 공간입니다. 탈중앙화된 시스템을 이루기 위해서 반드시 필요한 개념이며 블록체인의 존재이기도 합니다. 블록체인을 이용하는 애플리케이션 측면에서는 멈추지 않는 애플리케이션을 가능하게 합니다. 

이더리움은 Blockspace를 제공한 최초의 블록체인입니다. 가상머신과 블록체인 리소스를 사용한만큼 책정되는 가스 미터링 개념을 도입하여 Blockspace 안에서 하나의 블록이 가용할 수 있는 자원을 한정 시키고 블록을 실행 시킵니다. 이를 기반으로 Blockspace를 활용한 여러 블록체인 프로토콜(e.g. 폴카닷, 솔라나 등) 이 생겨났고 각각의 고유의 기능을 가지고 있습니다.

**Blockspace** 는 현대 컴퓨터의 CPU 처럼 서로 다른 블록체인(e.g. 인프라블록체인 혹은 스마트 컨트랙트)로 부터 생성되는 여러 블록들을 **다중 스레드 방식** 처럼 병렬적으로 실행시킵니다. 이것의 핵심은 한 코어당 한 개의 블록을 실행시킬 수 있는 **실행 코어(Execution Core)** 입니다.(현재 Polkadot은 하나의 코어를 여러 개로 분할하는 _Agile Core Time_ 개발 중) ***인프라블록체인*** 에서의 **밸리데이터**들은 **공유된 보안(shared security)** 을 이루며 Blockspace 를 제공할 뿐만 아니라, 동일한 품질의 Blockspace를 제공하는 서비스 제공자 역할을 수행합니다. 

기관 및 공공기관을 위한 엔터프라이즈 블록체인인 ***인프라블록체인(InfraBlockchain)*** 은 안전한 Blockspace를 제공하는 것이 핵심이며, 이것을 위한 Blockspace 할당 메커니즘이 존재합니다.

## 인프라릴레이체인(InfraRelayChain)

***인프라릴레이체인*** 은 ***_인프라블록체인_*** 의 중심으로써, 서로 다른 블록체인 간 상호 운용성(interoperablitiy)을 통한 확장성(scalability)과 공유된 보안을 통한 안정성(security)에 집중한 블록체인 네트워크입니다. ***인프라릴레이체인***의 가장 큰 역할은 서로 다른 블록체인이 유기적으로 연결될 수 있도록 하는 것입니다. 따라서 스마트 컨트랙트를 실행하거나 토큰 전송 같은 기본적인 기능들은 파라체인에 위임하여 가장 최소한의 기능(e.g. 파라체인 블록 검증)으로 동작할 수 있습니다.

## 파라체인(Parachain)

**파라체인(Parachain)** 은 특정 서비스(e.g. DID, STO, 등)에 특화된 블록체인으로써 릴레이체인에 "병렬적(parallel)"인 "체인(chain)" 에서 이름이 유래되었습니다. 파라체인의 일반적인 형태는 블록체인이지만 그 형태가 꼭 블록체인일 필요는 없습니다(e.g. 스마트 컨트랙트). 파라체인은 다음과 같은 특징을 갖고 있습니다:

- **상태 전이 기계(State Transition Machine)**: 파라체인은 각각의 상태를 가지고 있는 결정적인(deterministic) 상태 기계입니다.

- **콜레이터(Collator)**: 콜레이터는 파라체인의 여러 트랜잭션들을 모아 블록을 만들고 릴레이체인 노드에 전달하는 노드입니다. 콜레이터는 파라체인의 풀노드이면서 릴레이체인의 라이트 노드 역할을 수행합니다.

### 파라체인 상태 전이

각각의 파라체인은 상태를 변경할 수 있는 고유의 로직인 _상태 전이 함수(State Transition Function)_ 을 갖고 있습니다. 이러한 상태 전이 함수는 실행 가능한 웹어셈블리(WASM) 형태로 **_인프라릴레이체인_** 에 저장됩니다. 이렇기 때문에, 릴레이체인에 연결된 모든 파라체인은 릴레이체인 밸리데이터에 의해 검증받게 됩니다.

### Proof-of-Validity(PoV)

파라체인은 모든 상태 변화를 블록의 형태로 만들고 정상적으로 상태 변화를 했다는 proof 인 _Proof-of-Validity(PoV)_ 라고 하는 특수한 블록을 전달합니다. 릴레이 체인 밸리데이터는 해당 PoV를 릴레이 체인에 저장되어있는 각각의 파라체인 상태 전이 함수를 이용하여 블록을 실행한 후 나온 최종값과 파라체인이 전달한 값을 비교하여 검증합니다. _현재 **_인프라릴레이체인_** 이 전달받을 수 있는 PoV 최대 크기는 5MB입니다._

```rust
#[derive(codec::Encode, codec::Decode, Clone)]
pub struct ParachainBlockData<B: BlockT> {
	/// The header of the parachain block.
	header: B::Header,
	/// The extrinsics of the parachain block.
	extrinsics: sp_std::vec::Vec<B::Extrinsic>,
	/// The data that is required to emulate the storage accesses executed by all extrinsics.
	storage_proof: sp_trie::CompactProof,
}

type BlockData = Vec<u8>;
pub struct PoV(BlockData);

// Create PoV
let pov = parachain_block_data.encode();
```

#### header
파라체인 블록의 헤더

#### extrinsics
파라체인 블록의 상태 전이 함수

#### storage_proof
상태 전이 시 변경되었던 스토리지 목록

## 공유된 보안(Shared Security)

![](/media/images/docs/infrablockchain/learn/architecture/shared-security.png)

파라체인이 되었을 때 가장 큰 이점은 **_공유된 보안(Shared Security)_** 입니다. 각각의 파라체인은 독립적으로 밸리데이터를 구성할 필요 없이 릴레이체인 밸리데이터에 의해 블록 생성과 확정에 대한 안전을 보장받을 수 있습니다. 이로 인해 파라체인은 이러한 민감한 요소들을 신경 쓰지 않고 각 서비스에 맞는 비즈니스 로직에만 신경 쓸 수 있습니다. 


## 파라체인 프로토콜(Parachain Protocol)

![](/media/images/docs/infrablockchain/learn/architecture/parachain-protocol.png)

파라체인 프로토콜의 목표는 파라체인 블록 생성부터 릴레이체인에 포함(Inclusion) 및 승인(Approval)될 때까지 프로세스를 반복적으로 병렬 수행하는 것입니다. 이 프로토콜은 강력한 보안을 유지하는 동시에 파라체인이 효율적으로 운영될 수 있도록 합니다. 

파라체인 프로토콜의 역할은 다음과 같이 구분됩니다:

- **밸리데이터(Validator)**: 밸리데이터의 역할은 파라체인으로부터 받은 PoV를 검증하고 일정 기간 동안의 가용성(Availability)를 보장하는 역할을 수행합니다. 

- **콜레이터(Collator)**: PoV를 생성하고 밸리데이터에게 전달하는 역할을 수행합니다.


파라체인 프로토콜은 크게 2단계로 구분됩니다:

- **포함 파이프라인(Inclusion Pipeline)** : 1) 콜레이터들이 보낸 블록들을 2) 밸리데이터들이 검증한 후 3) 정족수 이상의 유효성을 받으면 해당 블록을 4) **_지지(backed)_** 하고 릴레이체인에 5) **_포함(included)_** 시키는 단계입니다. 하지만 아직 완전히 승인된 것은 아닌 단계(**_pending approval_**)입니다. 

- **승인 과정(Approval Process)**: 포함 단계에 참여하지 않은 랜덤으로 선택된 릴레이체인 밸리데이터에 의해 검증받은 후 해당 파라체인 블록을 최종적으로 승인(**_approved_**)하는 단계입니다.

## 다음 단계로 넘어가기

- [Blockspace over Blockchains](https://www.rob.tech/blog/polkadot-blockspace-over-blockchains/)

<!-- - [Agile Core Time]() -->