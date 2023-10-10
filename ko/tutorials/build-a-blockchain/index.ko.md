---
title: 블록체인 구축하기
description: 이 튜토리얼은 Substrate 블록체인을 구축하고 기능을 추가하며 간단한 트랜잭션을 제출하고 노드 작업을 관찰하는 실습 경험을 제공합니다.
keywords:
  - 블록체인
  - 노드
  - 메트릭스
  - 검증자
  - 개인 네트워크
  - 키 생성
---

**블록체인 구축하기** 튜토리얼은 Substrate 기반 블록체인 노드 작업에 대한 기본 사항을 설명하며, 노드들이 피어 네트워크에서 서로 통신하고 노드 작업에 대한 메트릭스를 수집하는 방법을 알려줍니다.
일반적으로, 나중에 나오는 튜토리얼을 시도하거나 더 복잡한 작업을 수행하기 위해 기초 튜토리얼을 먼저 완료해야 합니다.
나중의 튜토리얼은 시작 튜토리얼에서 배운 기본 주제를 강화하거나 확장합니다.

- [로컬 블록체인 구축하기](/tutorials/build-a-blockchain/build-local-blockchain/)은 개발 환경에서 로컬 노드를 설정하고 상호작용하는 방법을 보여줍니다.
- [네트워크 시뮬레이션](/tutorials/build-a-blockchain/simulate-network/)은 미리 정의된 계정을 사용하여 두 개의 노드로 구성된 네트워크를 시뮬레이션하는 방법을 안내합니다.
- [신뢰할 수 있는 노드 추가하기](/tutorials/build-a-blockchain/add-trusted-nodes/)는 키를 생성하고 체인 스펙을 배포하여 신뢰할 수 있는 검증자 노드의 작은 네트워크를 생성하는 방법을 보여줍니다.
- [특정 노드 권한 부여하기](/tutorials/build-a-blockchain/authorize-specific-nodes/)는 권한이 있는 노드와 제한된 액세스 권한을 가진 노드가 혼합된 네트워크를 구성하는 방법을 설명합니다.
- [노드 메트릭스 모니터링](/tutorials/build-a-blockchain/monitor-node-metrics/)은 Substrate가 노출하는 노드 메트릭스를 활용하는 방법을 강조합니다.
- [운영 중인 네트워크 업그레이드하기](/tutorials/build-a-blockchain/upgrade-a-running-network)는 실행 중인 Substrate 노드의 런타임을 수정하여 포크 없는 업그레이드를 실현하는 방법을 설명합니다.