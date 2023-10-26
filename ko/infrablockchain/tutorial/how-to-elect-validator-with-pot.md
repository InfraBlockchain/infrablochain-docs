---
title: How to elect validator with PoT
description: 이 튜토리얼은 PoT 를 이용하여 인프라 릴레이체인의 밸리데이터를 선출하는 방법을 배웁니다.
keywords:
  - Proof-of-Transaction
  - 시스템 토큰
---

이 튜토리얼은 실제로 릴레이 체인(e.g **_InfraRelayChain_** )과 패러체인(e.g **_InfraBlockchain_**) 구조인 멀티체인 환경에서 [Aggregated TaaV(Transaction-as-a-Vote)]()를 이용하여 [PoT(Proof-of-Transaction)]() 에 기반한 릴레이 체인의 밸리데이터를 선출하는 방법을 알아 보겠습니다. 

## 튜토리얼 목표

이 튜토리얼을 완료함으로써 다음 목표를 달성할 수 있습니다:

- 좀비넷을 활용하여 릴레이체인/파라체인 테스트 네트워크를 구축할 수 있습니다.
- 시스템 토큰을 이용하여 트랜잭션 수수료를 지불하고 인프라 릴레이체인의 밸리데이터 후보에 투표할 수 있습니다.
- 인프라 릴레이체인의 밸리데이터를 _Seed Trust_ 노드와 _PoT_ 노드로 구성된 것을 확인할 수 있습니다.

## 시작하기전에

시작하기 전에 다음을 확인하세요:

- [로컬 릴레이 체인 준비하기](../../tutorials/build-a-parachain/prepare-a-local-relay-chain.ko.md)

- [로컬 Infra Asset Hub 준비하기]()

- [로컬 패러체인 준비하기](../../tutorials/build-a-parachain/connect-a-local-parachain.ko.md) 

