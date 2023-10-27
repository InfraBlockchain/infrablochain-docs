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

- [인프라 블록 스페이스 테스트 환경 구축하기](../tutorial/build-infrablockspace/) 를 통해 간단하게 테스트 환경을 구축할 수 있는 방법 및 실제 네트워크 환경 구축하는 방법에 대해 알아보기

- [Proof-of-Transaction](../learn/proof-of-transaction.md) 에 대해 알아보기

- [시스템 토큰]() 으로 트랜잭션 수수료를 내는 방법에 대해 알아보기

## 튜토리얼 단계

- 좀비넷으로 테스트할 환경을 구축합니다. 본 튜토리얼에서는 _Infra Asset Hub_ 와 _Parachain Templete Node_ 두 개의 패러체인을 이용하여 진행할 예정입니다.

- 각 패러체인에서 _시스템 토큰(Sysetm Token)_  을 이용하여 투표할 대상을 포함하여 트랜잭션을 요청합니다.

- **_인프라 릴레이 체인(InfraRelayChain)_** 에서 투표가 수집되는 것을 확인합니다.

- 특정 시점이 지났을 때, Proof-of-Transaction 에 기반하여 밸리데이터가 선출되어 블록을 생성하는지 확인합니다.








