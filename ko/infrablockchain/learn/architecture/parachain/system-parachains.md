---
title: 시스템 파라체인
description: 시스템 파라체인에 대한 내용을 다룹니다.
keywords:
  - 시스템 파라체인
  - Asset Hub
  - Bridge Hub
--- 

시스템 파라체인은 핵심 ***인프라 블록체인(InfraBlockchain)*** 프로토콜 기능을 포함하지만 그 기능이 릴레이 체인이 아닌 파라체인에 있는 것입니다. 릴레이 체인이 아닌 파라체인에 핵심 프로토콜 로직을 호스팅함으로써 ***인프라 블록체인(InfraBlockchain)*** 는 자체 확장 기술인 병렬 실행을 사용하여 자신을 호스팅합니다. 시스템 파라체인은 프로토콜 기능을 수행하는 트랜잭션을 릴레이 체인에서 제거하여 블록 공간([blockspace](../architecture.md#block-space))을 본연의 기능(e.g [파라체인 유효성 검증](../architecture.md#파라체인-상태-전이))에 더 많이 확보할 수 있게 됩니다. 

시스템 파라체인의 거버넌스는 릴레이 체인에 의해 이루어지며 ***인프라 블록체인(InfraBlockchain)*** 전체 네트워크를 위한 특수한 파라체인입니다. ***인프라 블록체인(InfraBlockchain)*** 에 존재하는 시스템 파라체인은 다음과 같습니다: 
- [Asset Hub](./system-parachains.md#asset-hub): 토큰(e.g [시스템 토큰](../../protocol/system-token.md))과 관련된 기능을 관장하는 시스템 체인입니다. 
- [Bridge Hub](./system-parachains.md#bridge-hub): 서로 다른 네트워크간 브릿지를 관장하는 시스템 체인입니다.

## Asset Hub

### 개요 

*Asset Hub*는 ***인프라 블록체인(InfraBlockchain)*** 생태계 안에서 토큰(e.g 시스템 토큰)과 관련된 기능을 관장하는 시스템 체인입니다. 이는 토큰 생성자(e.g 시스템 토큰 발행자)가 ***인프라 블록체인(InfraBlockchain)*** 네트워크에서 토큰을 생성하고 토큰의 총 발행량([다른 파라체인으로 전송](../../../tutorials/build/transfer-assets-with-xcm.md)된 토큰 포함)을 추적하는 데 도움이 됩니다. 또한 토큰을 생성(create)하고 예치한 만큼 만들어내며(mint) 소각하는 온체인 자산을 관리할 수 있는 장소이기도 합니다. 또한 [Uniques 팔렛](https://github.com/InfraBlockchain/infrablockspace-sdk/tree/822bc6c9706774a98122eb432f412b871a98a4bd/substrate/frame/uniques)과 새로운 [NFTs 팔렛](https://github.com/InfraBlockchain/infrablockspace-sdk/tree/822bc6c9706774a98122eb432f412b871a98a4bd/substrate/frame/nfts)을 통해 NFT도 지원하고 있습니다. 

토큰과 관련된 로직은 스마트 컨트랙트가 아닌 런타임에서 직접 실행기 떄문에 파라체인에서 트랜잭션을 실행하는 비용이 릴레이 체인의 약 1/10 정도입니다. **이러한 낮은 수수료 수준은 *Asset Hub*가 시스템 토큰 잔액 및 전송을 처리하고 온체인 자산을 관리하는 데 적합하다는 것을 의미합니다.** 

## Bridge Hub

한개 이상의 ***인프라 블록체인(InfraBlockchain)*** 생태계가 서로 연결된 ***연합 멀티 체인(Federated Multi-BlockChain)*** 을 구축함에 있어서 브릿징과 관련된 기능을 관장하는 시스템 체인입니다. 