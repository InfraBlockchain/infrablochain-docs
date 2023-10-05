---
title: 빠른 참조 가이드
description:
keywords:
---

서브스트레이트 "빠른 참조 가이드"는 특정 목표를 달성하기 위한 지침을 제공합니다.
각 가이드는 이미 서브스트레이트와 러스트 프로그래밍에 익숙한 상태에서 특정 작업을 수행하는 방법을 설명합니다.

## 기본 사항

런타임 개발에서 일반적인 패턴을 배우려면 다음 가이드를 참조하세요.

- [팔레트 가져오기](/reference/how-to-guides/basics/import-a-pallet/)
- [런타임 상수 구성](/reference/how-to-guides/basics/configure-runtime-constants/)
- [제네시스 상태 구성](/reference/how-to-guides/basics/configure-genesis-state)
- [체인 사양 사용자 정의](/reference/how-to-guides/basics/customize-a-chain-specification)
- [도우미 함수 사용](/reference/how-to-guides/basics/use-helper-functions)

## 팔레트 디자인

FRAME을 사용하여 팔레트를 구축하는 최상의 방법에 대한 가이드를 참조하세요.

- [잠금 가능한 통화 구현](/reference/how-to-guides/pallet-design/implement-lockable-currency/)
- [임의성 통합](/reference/how-to-guides/pallet-design/incorporate-randomness/)
- [크라우드펀딩 구성](/reference/how-to-guides/pallet-design/configure-crowdfunding/)
- [저장 구조 (struct) 생성](/reference/how-to-guides/pallet-design/create-a-storage-structure/)
- [타이트한 팔레트 결합 사용](/reference/how-to-guides/pallet-design/use-tight-coupling/)
- [느슨한 팔레트 결합 사용](/reference/how-to-guides/pallet-design/use-loose-coupling/)

## 가중치

벤치마킹 및 가중치 구성에 도움이 되는 가이드를 참조하세요.

- [수수료 계산](/reference/how-to-guides/weights/calculate-fees/)
- [벤치마크 추가](/reference/how-to-guides/weights/add-benchmarks/)
- [사용자 정의 가중치 사용](/reference/how-to-guides/weights/use-custom-weights/)
- [조건부 가중치 사용](/reference/how-to-guides/weights/use-conditional-weights/)

## 테스트

팔레트 및 런타임 로직 테스트에 도움이 되는 가이드를 참조하세요.

- [기본 테스트 설정](/reference/how-to-guides/testing/set-up-basic-tests/)
- [전송 함수 테스트](/reference/how-to-guides/testing/test-a-transfer-function/)

## 스토리지 마이그레이션

스토리지 마이그레이션에 도움이 되는 가이드를 참조하세요.

- [기본 스토리지 마이그레이션](/reference/how-to-guides/storage-migrations/basic-storage-migration/)
- [마이그레이션 트리거](/reference/how-to-guides/storage-migrations/trigger-migration/)

## 합의 모델

런타임에 합의 메커니즘을 구현하기 위한 가이드를 참조하세요.

- [하이브리드 노드 생성](/reference/how-to-guides/consensus-models/create-a-hybrid-node/)
- [작업 증명 합의 추가](/reference/how-to-guides/consensus-models/add-proof-of-work-consensus/)

## 파라체인

서브스트레이트 파라체인 작업에 도움이 되는 가이드를 참조하세요.

- [솔로 체인 변환](/reference/how-to-guides/parachains/convert-a-solo-chain/)
- [릴레이 체인에 연결](/reference/how-to-guides/parachains/connect-to-a-relay-chain/)
- [콜레이터 선택](/reference/how-to-guides/parachains/select-collators/)
- [시작 준비](/reference/how-to-guides/parachains/prepare-to-launch/)
- [파라체인 업그레이드](/reference/how-to-guides/parachains/upgrade-a-parachain/)
- [경매 및 크라우드론](/reference/how-to-guides/parachains/auctions-and-crowdloans/)
- [HRMP 채널 추가](/reference/how-to-guides/parachains/add-hrmp-channels/)

## 도구

서브스트레이트 체인을 운영하는 데 도움이 되는 추가 도구에 대한 가이드를 참조하세요.

- [try-runtime 사용](/reference/how-to-guides/tools/use-try-runtime/)
- [체인에 대한 txwrapper 생성](/reference/how-to-guides/tools/create-a-txwrapper/)
- [REST 엔드포인트를 사용하여 체인 데이터 가져오기](/reference/how-to-guides/tools/use-sidecar/)
- [Wasm 바이너리 검증](/reference/how-to-guides/tools/verify-wasm/)

## 오프체인 워커

오프체인 데이터 작업에 도움이 되는 가이드를 참조하세요.

- [오프체인 HTTP 요청](/reference/how-to-guides/offchain-workers/offchain-http-requests/)
- [오프체인 로컬 스토리지](/reference/how-to-guides/offchain-workers/offchain-local-storage/)
- [오프체인 인덱싱](/reference/how-to-guides/offchain-workers/offchain-indexing/)

<!--
- [가중치 계산](/reference/how-to-guides/basics/calc-weights/)
- [기본 토큰 생성](/reference/how-to-guides/basics/mint-basic-tokens/)
- [계약 팔레트 추가](/reference/how-to-guides/pallet-design/add-contracts-pallet/)
-->