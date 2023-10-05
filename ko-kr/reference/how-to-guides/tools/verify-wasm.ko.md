---
title: Wasm 바이너리 검증하기
description: 어떤 Substrate 체인의 런타임 기능을 노출합니다.
keywords:
  - 도구
  - wasm
  - 런타임
---

[subwasm](https://github.com/chevdor/subwasm) 도구는 어떤 Substrate 체인의 런타임 기능을 노출하는 방법을 제공합니다.
이 가이드에서는 Subwasm을 사용하여 다음과 같은 작업을 수행하는 방법을 안내합니다:

- 런타임의 모듈 스냅샷 가져오기
- 프론트 엔드 애플리케이션을 위한 체인의 런타임의 검증 가능한 과거 정보 노출
- 두 개의 런타임 간의 변경 사항을 diff를 사용하여 확인

## 시작하기 전에

다음 사항을 확인하세요:

- [최신 릴리스](https://github.com/chevdor/subwasm/releases)의 Subwasm이 설치되어 있는지 확인합니다.
- 검사하려는 런타임을 실행하는 로컬에서 실행 중인 체인이 있어야 합니다.
- 메타데이터를 필터링하기 위해 환경에 [JQ](https://stedolan.github.io/jq/download/)가 설치되어 있어야 합니다.

## 로컬 체인의 런타임에 대한 정보 가져오기

1. 체인 디렉토리 내부에서 로컬 노드를 실행합니다.
   이 예제에서는 개발 모드에서 노드 템플릿을 실행합니다.

   ```bash
   ./target/release/node-template --tmp --dev
   ```

1. 새 터미널에서 `TempWasms`라는 임시 폴더를 생성합니다.

   ```bash
   mkdir TempWasms
   ```

1. 새 폴더 내에서 Subwasm을 사용하여 로컬 체인의 Wasm을 가져와 저장합니다.

   ```bash
   cd TempWasms
   subwasm get --chain local -o mychain.wasm
   ```

   `-o` 플래그는 Wasm 파일을 `mychain.wasm`으로 저장합니다.

1. 다음 명령을 실행하여 체인의 메타데이터를 JSON 파일로 저장합니다:

   ```bash
    subwasm --json meta mychain.wasm | jq 'del( .. | .documentation? )' > mychain-metadata.json
   ```

   `subwasm --json meta mychain.wasm` 명령을 사용하여 메타데이터 세부 정보를 필터링하기 위해 JQ를 사용할 수 있습니다.
   이 예제에서는 모든 문서 메타데이터를 무시합니다.
   Substrate 체인이 노출하는 메타데이터 필드에 대한 정보는 [Substrate Metadata](https://polkadot.js.org/docs/substrate) 문서를 참조하세요.

1. 메타데이터를 JSON 파일에 저장한 후에는 프론트 엔드 애플리케이션에서 사용할 수 있습니다.

## 라이브 체인의 두 개의 다른 런타임 비교하기

Subwasm을 사용하여 다른 블록에서 체인의 런타임을 비교할 수도 있습니다.
Subwasm은 Polkadot, Kusama, Westend와 같은 일반적인 Substrate 체인의 URL을 사용하여 노드의 엔드포인트를 반환하는 함수를 사용합니다.
다음 예제에서는 Polkadot 런타임의 500,000 블록과 최신 런타임을 비교합니다.

1. 블록 익스플로러로 이동하여 비교하려는 런타임의 블록 해시를 가져옵니다.

   예를 들어, [PolkadotJS Apps](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frpc.polkadot.io#/explorer)로 이동하여 [500000 블록](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frpc.polkadot.io#/explorer/query/500000)의 블록 해시를 가져옵니다.

   `0xfdeab19649426aadbdab046c9b5dac0a5b0c850b8fd0927e3b4e6c896d5fb0b7`

1. 이전 상태의 Wasm 블롭을 저장합니다.

   ```bash
   BLOCK_HASH=0xfdeab19649426aadbdab046c9b5dac0a5b0c850b8fd0927e3b4e6c896d5fb0b7
   subwasm get --chain polkadot --block $BLOCK_HASH -o polkadot-500000.wasm
   ```

1. 최신 런타임의 Wasm 블롭을 가져옵니다.

   ```bash
   subwasm get --chain polkadot -o polkadot-latest.wasm
   ```

1. 두 개의 런타임을 비교하고 출력을 JSON 파일에 저장합니다.

   ```bash
   subwasm diff polkadot-latest.wasm polkadot-500000.wasm --json > polkadot-wasm-diff.json
   ```

## 예제

위에서 설명한 것처럼 `subwasm`에는 많은 사용 사례가 있습니다.
추가적인 예제로는 다음이 있습니다:

- Wasm 파일 압축 및 압축 해제
- Wasm 파일 검사를 통한 거버넌스에서의 런타임 업그레이드 확인