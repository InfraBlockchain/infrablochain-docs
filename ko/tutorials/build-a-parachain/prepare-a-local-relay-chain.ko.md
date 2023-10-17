---
title: 로컬 릴레이 체인 준비하기
description: 로컬 릴레이 체인을 설정하는 방법을 설명합니다. 로컬 테스트 네트워크를 구성하기 위해 테스트 파라체인 노드가 연결할 수 있는 로컬 릴레이 체인이 필요합니다.
keywords:
  - 폴카닷
  - 커뮬러스
  - 릴레이체인
  - 파라체인
  - 파라스레드
  - 파라이드
  - 로코코
  - xcm
  - xcmp
  - 콜레이터
  - 테스트넷
  - 로컬
---

## 시작하기 전에

시작하기 전에 다음 사항을 고려해보세요:

- 엄격한 전제 조건은 아니지만, [신뢰할 수 있는 검증자를 위한 개인 네트워크의 체인 사양을 생성하는 방법](/tutorials/build-a-blockchain/add-trusted-nodes/)에 설명된 대로 먼저 배우는 것이 좋습니다.

시작하기 전에 다음을 확인하세요:

- [Rust 및 Rust 도구 체인](/install/)을 설치하여 Substrate 개발 환경을 구성했는지 확인하세요.

- [로컬 블록체인 구축](/tutorials/build-a-blockchain/build-local-blockchain/)을 완료하고 Substrate 노드를 컴파일하고 실행하는 방법을 알고 있는지 확인하세요.

- Polkadot [아키텍처 및 용어](https://wiki.polkadot.network/docs/learn-architecture)에 대해 일반적으로 알고 있는지 확인하세요.

- 파라체인 버전 및 종속성은 해당 파라체인이 연결되는 릴레이 체인의 버전과 꽤 밀접하게 관련되어 있음을 알고 있어야 합니다.

  파라체인은 릴레이 체인의 업그레이드와 동기화되어야 정상적으로 작동할 수 있습니다.
  릴레이 체인의 최신 버전과 동기화되지 않으면 네트워크가 블록을 생성하지 못할 가능성이 높습니다.

  튜토리얼은 일반적으로 기능을 설명하기 위해 최신 Polkadot 브랜치를 사용합니다.
  튜토리얼이 예상대로 작동하지 않는 경우 로컬 환경에 최신 Polkadot 브랜치가 있는지 확인하고 필요한 경우 로컬 소프트웨어를 업데이트해야 합니다.

## 튜토리얼 목표

이 튜토리얼을 완료함으로써 다음 목표를 달성할 수 있습니다:

- 파라체인 빌드 환경을 설정합니다.
- 로컬 릴레이 체인 사양을 준비합니다.
- 로컬에서 릴레이 체인을 시작합니다.

## 릴레이 체인 노드 빌드하기

Polkadot은 Substrate 기반의 릴레이 체인입니다.
따라서 이 튜토리얼에서는 로컬 릴레이 체인을 준비하기 위해 Polkadot 저장소에서 코드를 사용합니다.

1. 안정적인 작업 환경을 준비하기 위해 Polkadot 저장소의 가장 최신 릴리스 브랜치를 복제합니다.
   
   릴리스 브랜치는 가장 신뢰할 수 있으며 `release-v<n..n.n>`과 같은 네이밍 컨벤션을 사용합니다.
   예를 들어, 이 튜토리얼에서 사용하는 릴리스 브랜치는 `release-v1.0.0`입니다.
   더 최신의 릴리스가 가능하며, 대부분의 경우 `release-v1.0.0` 브랜치 대신 더 최신의 릴리스 브랜치를 사용할 수 있습니다.
   각 릴리스에 대한 정보는 GitHub의 [릴리스](https://github.com/paritytech/polkadot/releases) 탭에서 확인할 수 있습니다.
   
   ```bash
   git clone --branch release-v1.0.0 https://github.com/paritytech/polkadot-sdk.git
   ```

2. 다음 명령을 실행하여 `polkadot` 디렉토리의 루트로 이동합니다:
   
   ```bash
   cd polkadot
   ```

3. 다음 명령을 실행하여 릴레이 체인 노드를 빌드합니다:
   
   ```bash
   cargo build --release
   ```
   
   노드를 컴파일하는 데에는 15분에서 60분이 소요될 수 있습니다.

4. 다음 명령을 실행하여 노드가 올바르게 빌드되었는지 확인합니다:
   
   ```bash
   ./target/release/polkadot --help
   ```

   명령줄 도움말이 표시되면 노드를 구성할 준비가 된 것입니다.

## 릴레이 체인 사양

모든 Substrate 기반 체인은 [체인 사양](/build/chain-spec/)이 필요합니다.
릴레이 체인 네트워크의 체인 사양은 다른 네트워크의 체인 사양과 동일한 유형의 구성 설정을 제공합니다.
체인 사양 파일의 많은 설정은 네트워크 작업에 중요합니다.
예를 들어, 체인 사양은 네트워크에 참여하는 피어, 검증자 키, 부트 노드 주소 및 기타 정보를 식별합니다.

### 샘플 체인 사양

이 튜토리얼에서 로컬 릴레이 체인은 두 개의 검증자 릴레이 체인 노드인 Alice와 Bob을 사용하는 샘플 체인 사양 파일을 사용합니다.
릴레이 체인은 연결된 파라체인 콜레이터의 총 수보다 적어도 하나 이상의 검증자 노드가 실행되어야 하므로, 이 튜토리얼의 체인 사양은 **단일 파라체인**을 가진 로컬 릴레이 체인 네트워크에만 사용할 수 있습니다.

하나의 콜레이터를 가진 두 개의 파라체인을 연결하려면 세 개 이상의 릴레이 체인 검증자 노드를 실행해야 합니다.
일반적으로 두 개 이상의 파라체인을 설정하려면 체인 사양을 수정하고 추가적인 검증자를 하드코딩해야 합니다.

### 일반 체인 사양 파일과 원시 체인 사양 파일

샘플 체인 사양에는 두 가지 형식이 있습니다. 일반 텍스트 형식의 JSON 파일과 SCALE로 인코딩된 원시 형식의 JSON 파일입니다.

- [일반 샘플 릴레이 체인 사양](/assets/tutorials/relay-chain-specs/plain-local-chainspec.json)
- [원시 샘플 릴레이 체인 사양](/assets/tutorials/relay-chain-specs/raw-local-chainspec.json)

일반 텍스트 버전의 체인 사양 파일을 읽고 편집할 수 있습니다.
그러나 체인 사양 파일을 노드를 시작하는 데 사용하려면 SCALE로 인코딩된 원시 형식으로 변환해야 합니다.
원시 형식의 체인 사양을 사용하여 노드를 시작하는 방법에 대한 정보는 [체인 사양 사용자 정의](/reference/how-to-guides/basics/customize-a-chain-specification/)를 참조하세요.

샘플 체인 사양은 두 개의 검증자 노드를 가진 단일 파라체인에만 유효합니다.
다른 검증자를 추가하거나 릴레이 체인에 여러 개의 파라체인을 추가하거나 미리 정의된 계정 대신 사용자 정의 계정 키를 사용하려면 사용자 정의 체인 사양 파일을 만들어야 합니다.

동일한 로컬 네트워크에서 누군가와 동시에 이 튜토리얼을 완료하는 경우, 그들의 노드와 실수로 피어링되지 않도록 일반 샘플 릴레이 체인 사양을 다운로드하고 수정해야 합니다. 일반 체인 사양에서 다음 줄을 찾아 고유한 protocolId를 만들기 위해 문자를 추가하세요:

```json
   "protocolId": "dot"
```

## 릴레이 체인 노드 시작하기

파라체인의 블록 생성을 시작하려면 먼저 파라체인이 연결할 수 있는 릴레이 체인을 시작해야 합니다.

[원시 샘플 체인 사양 파일](/assets/tutorials/relay-chain-specs/raw-local-chainspec.json)을 사용하여 검증자 노드를 시작하려면 다음을 수행하세요:

1. 로컬 컴퓨터의 작업 디렉토리에 원시 체인 사양 파일을 다운로드합니다.
   
   예를 들어, `/tmp` 디렉토리에 `raw-local-chainspec.json`로 파일을 저장합니다.
   노드를 시작하는 명령에서 파일의 경로를 지정해야 합니다.

2. 다음 명령을 실행하여 `alice` 계정을 사용하여 첫 번째 검증자를 시작합니다:
   
   ```bash
   ./target/release/polkadot \
   --alice \
   --validator \
   --base-path /tmp/relay/alice \
   --chain /tmp/raw-local-chainspec.json \
   --port 30333 \
   --rpc-port 9944
   ```

   이 명령은 샘플 체인 사양 파일의 위치로 `/tmp/raw-local-chainspec.json`을 사용합니다.
   명령줄의 `--chain` 옵션에 다운로드한 원시 체인 사양 파일의 경로를 지정하는지 확인하세요.
   이 명령은 포트(`port`)와 웹소켓 포트(`ws-port`)의 기본값을 사용합니다.
   이 값들은 여기에 명시적으로 포함되어 있으며 항상 이러한 설정을 확인해야 함을 상기시킵니다.
   노드가 시작된 후에는 동일한 로컬 컴퓨터의 다른 노드가 이러한 포트를 사용할 수 없습니다.

3. 노드가 시작될 때 로그 메시지를 검토하고 로컬 노드 식별자를 기록하세요.
   
   다른 노드가 연결할 수 있도록 이 식별자를 지정해야 합니다.
   
   ```bash
   🏷 Local node identity is: 12D3KooWGjsmVmZCM1jPtVNp6hRbbkGBK3LADYNniJAKJ19NUYiq
   ```

4. 새 터미널을 열고 `bob` 계정을 사용하여 두 번째 검증자를 시작하세요.
   
   첫 번째 노드를 시작하는 데 사용한 명령과 유사한 명령을 사용하지만 몇 가지 중요한 차이점이 있습니다.
   
   ```bash
   ./target/release/polkadot \
   --bob \
   --validator \
   --base-path /tmp/relay/bob \
   --chain /tmp/raw-local-chainspec.json \
   --port 30334 \
   --rpc-port 9945
   ```
   
   이 명령은 다른 베이스 경로(`/tmp/relay/bob`), 검증자 키(`--bob`), 포트(`30334` 및 `9945`)를 사용합니다.
   
   두 검증자가 동일한 로컬 컴퓨터에서 실행되므로 체인 사양 파일에 지정된 첫 번째 노드의 IP 주소와 피어 식별자를 지정할 필요가 없습니다.
   `bootnodes` 옵션은 로컬 네트워크 외부에서 실행되는 노드나 체인 사양 파일에 식별되지 않은 노드에 연결하려는 경우 필요합니다.

   릴레이 체인이 블록을 생성하지 않는 경우 방화벽을 비활성화하거나 `alice` 노드의 주소를 `bootnodes` 명령줄 옵션에 추가하여 노드를 시작해 보세요.
   `bootnodes` 옵션을 추가하는 방법은 다음과 같습니다(위의 노드 식별자를 사용): `--bootnodes /ip4/127.0.0.1/tcp/30333/p2p/12D3KooWGjsmVmZCM1jPtVNp6hRbbkGBK3LADYNniJAKJ19NUYiq`.

## 다음 단계로 넘어가기

이 튜토리얼에서는 로컬 릴레이 체인을 빌드하고 시작하는 방법을 배웠습니다.
여기서는 로컬 릴레이 체인에 로컬 파라체인을 연결하는 방법을 배우거나 테스트 네트워크를 자동화하는 도구를 실험해 볼 수 있습니다.

- [로컬 파라체인 연결](/tutorials/build-a-parachain/connect-a-local-parachain/)
- [파라체인 테스트 네트워크 시작](https://github.com/open-web3-stack/parachain-launch)
- [zombienet 설정](https://github.com/paritytech/zombienet)은 일시적인 Polkadot 및 Substrate 네트워크를 생성하고 테스트를 수행할 수 있는 CLI 도구입니다.

<!-- TODO NEW CONTENT docker and using prebuilt bins suggested https://github.com/substrate-developer-hub/substrate-docs/issues/1073 -->

<!-- TODO NEW CONTENT add details about these in HTG pages and link here in stead on these https://github.com/substrate-developer-hub/substrate-docs/issues/1098 -->