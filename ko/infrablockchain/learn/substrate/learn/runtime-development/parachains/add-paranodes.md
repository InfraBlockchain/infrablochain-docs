---
title: 로컬 파라체인 노드 추가하기
description: 이미 설정된 로컬 릴레이 테스트넷에 추가적인 파라체인 노드를 연결하는 방법
keywords:
  - 파라체인
  - 콜레이터
  - 풀 노드
---

[로컬 릴레이 체인 준비](/ko/infrablockchain/tutorials/build/build-infra-relay-chain.md)에서 배운 대로, 하나의 콜레이터로 파라체인을 작동시킬 수 있습니다.

## 추가적인 릴레이 체인 노드

네트워크에서 각 콜레이터(파라체인 블록 생성 노드)마다 최소한 두 개의 [**밸리데이터**](/ko/infrablockchain/learn/substrate/learn/basic/glossary.md#밸리데이터)(릴레이 체인 노드)가 실행되고 있어야 합니다.

제공된 **일반적인** [릴레이 체인 스펙 파일](/ko/infrablockchain/tutorials/build/build-infra-relay-chain.md#체인-스펙)을 수정하여 더 많은 검증자를 포함시킬 수 있거나, 더 "정확한" 방법으로 제네시스 상태의 소스를 수정하여 **릴레이 체인**에 테스트넷 검증자를 제네시스에 추가할 수 있습니다.
[신뢰할 수 있는 노드 추가](/ko/infrablockchain/learn/substrate/tutorials/build-a-blockchain/add-trusted-nodes.md) 튜토리얼에서 체인 스펙을 생성하는 방법을 상기해보세요.

### 콜레이터 시작하기

체인 스펙을 가지고 있다면, 이제 콜레이터 노드를 시작할 수 있습니다.
명령어의 두 번째 절반에서 릴레이 체인 노드를 시작할 때 사용한 동일한 릴레이 체인 체인 스펙을 제공해야 함에 주의하세요:

```bash
./target/release/parachain-collator \
--alice \
--collator \
--force-authoring \
--chain rococo-local-parachain-2000-raw.json \
--base-path /tmp/parachain/alice \
--port 40333 \
--rpc-port 8844 \
-- \
--execution wasm \
--chain <릴레이 체인 체인 스펙> \
--port 30343 \
--rpc-port 9977
```

### 두 번째 콜레이터 시작하기

`Alice` 노드가 이미 실행 중인 것으로 가정하고, 추가적인 콜레이터를 실행하기 위한 명령어는 다음과 같습니다.
첫 번째 콜레이터를 시작할 때 사용한 명령어와 거의 동일하지만, 충돌하는 포트와 `base-path` 디렉토리를 피해야 합니다:

```bash
./target/release/parachain-collator \
--bob \
--collator \
--force-authoring \
--chain rococo-local-parachain-2000-raw.json \
--base-path /tmp/parachain/bob \
--bootnodes <실행 중인 콜레이터 노드> \
--port 40334 \
--rpc-port 9946 \
-- \
--execution wasm \
--chain <릴레이 체인 체인 스펙> \
--port 30344 \
--rpc-port 9978
--bootnodes <다른 릴레이 체인 노드>
```

## 관련 자료

- [Polkadot 위키: 콜레이터](https://wiki.polkadot.network/docs/learn-collator).
- [로컬 릴레이 체인 준비](/ko/infrablockchain/tutorials/build/build-infra-relay-chain.md) 튜토리얼.