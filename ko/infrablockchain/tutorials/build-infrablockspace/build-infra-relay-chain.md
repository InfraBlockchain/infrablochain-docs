---
title: 인프라 릴레이 체인 구축하기
description: 이 튜토리얼은 인프라 릴레이 체인을 구축하는 방법에 대해 알아봅니다.
keywords:
  - 릴레이 체인
---

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [Rust 및 Rust 툴체인](/ko/substrate/install/)을 통해 Substrate 개발 환경을 구성 방법에 대해 확인합니다. 

- 인프라 블록체인의 기반이 되는 [멀티체인 아키텍처 및 용어](https://wiki.polkadot.network/docs/learn-architecture)에 대한 배경 지식을 확인합니다.

## 튜토리얼 목표

이 튜토리얼을 완료함으로써 다음 목표를 달성할 수 있습니다:

- **_인프라 릴레이 체인(InfraRelayChain)_** 런타임을 바이너리로 빌드하는 방법 
- 해당 바이너리로 부터 평문 체인 스펙과 _SCALE로 인코딩된_ 원시(raw) 체인 스펙을 추출하는 방법
- 체인 스펙으로부터 노드를 시작하는 방법 

## 릴레이 체인 노드 빌드하기

1. 최신 **_인프라 블록 스페이스(InfraBlockspace)_ SDK** 를 불러옵니다.
   
   ```bash
   git clone --branch master https://github.com/InfraBlockchain/infrablockspace-sdk.git
   ```

2. 다음 세개의 명령을 실행하여 **_인프라 블록스페이스(InfraBlockspace)_** 런타임을 빌드합니다:
   
   ```bash
   1. cargo build --release --bin infrablockspace

   2. cargo build --release --bin infrablockspace-execute-worker

   3. cargo build --release --bin infrablockspace-prepare-worker
   ```

3. 같은 경로에서 다음 명령을 실행하여 노드가 올바르게 빌드되었는지 확인합니다:
   
   ```bash
   ./target/release/infrablockspace --help
   ```

   명령줄 도움말이 표시되면 노드를 구성할 준비가 된 것입니다.

## 체인 스펙

모든 Substrate 기반 체인은 [체인 스펙](../../../substrate/build/chain-spec.ko.md)이 필요합니다. 체인 스펙에는 해당 네트워크의 초기 상태(genesis state) 및 여러 네트워크 설정값들이 명시되어 있습니다. 결정적인(Deterministic) 네트워크 구성을 위해 네트워크에 참여하는 모든 노드는 동일한 체인 스펙을 가지고 있습니다. 

### 평문(Plain) 체인 스펙 추출하기

같은 경로에서 다음과 같은 명령어로 평문 체인 스펙을 추출할 수 있습니다. 명령어를 입력하고 나면 같은 경로에 `plain-infra-relay-chainspec.json` 파일이 생성됩니다.

```bash
./target/release/infrablockspace build-spec --chain infra-relay-local --disable-default-bootnode > plain-infra-relay-chainspec.json
```
_본 튜토리얼에서는 네 개의 [Seed Trust 밸리데이터](https://docs.infrablockchain.net/infrablockspace-docs/v/ko/infrablockchain/learn/proof-of-transaction#undefined-1)로 구성된 인프라 릴레이 체인 체인 스펙 파일을 사용합니다. 실제 네트워크 구성은 `alice` 와 `bob` 밸리데이터 노드만 띄워 시뮬레이션합니다._

릴레이 체인은 연결된 파라체인 콜레이터의 총 수보다 적어도 하나 이상의 밸리데이터 노드가 실행되어야 하므로, 하나의 콜레이터를 가진 두 개의 파라체인을 연결하려면 세 개 이상의 릴레이 체인 검증자 노드를 실행해야 합니다.

### 원시(raw) 체인 스펙 파일 추출하기

[SCALE 인코딩](../../../substrate/reference/scale-codec.ko.md)된 원시(raw) 형식의 JSON 파일을 추출합니다. 해당 명령어를 입력하면 같은 경로에 `raw-infra-relay-chainspec.json` 파일이 생성됩니다. 

```bash
./target/release/infrablockspace build-spec --chain plain-infra-relay-chainspec.json --disable-default-bootnode --raw > raw-infra-relay-chainspec.json
```

일반 텍스트 버전의 체인 스펙 파일을 읽고 편집할 수 있습니다. 그러나 체인 스펙 파일을 노드를 시작하는 데 사용하려면 SCALE로 인코딩된 원시(raw) 형식으로 변환해야 합니다. 샘플 체인 스펙은 네 개의 밸리데이터로 구성된 네트워크입니다.
다른 밸리데이터를 추가하거나 릴레이 체인에 여러 개의 파라체인 추가 및 사전 정의된 계정 대신 커스텀 계정 키를 사용하려면 [커스텀 체인 스펙 파일](../../../substrate/reference/how-to-guides/basics/customize-a-chain-specification.ko.md)을 만들어야 합니다.

동일한 로컬 네트워크에서 누군가와 동시에 이 튜토리얼을 완료하는 경우, 해당 노드와 피어링되지 않도록 평문 체인 스펙을 수정해야 합니다. 평문 체인 스펙 JSON 파일에서 `protocolId` 키를 찾고 어떤 고유한 값으로 변경해줍니다.

```json
   "protocolId": "infrablockchain"
```

## 릴레이 체인 노드 시작하기

멀티체인 아키텍처인 **_인프라 블록 스페이스(InfraBlockspace)_** 를 구축하기 위해서 먼저 파라체인을 연결할 수 있는 **_인프라 릴레이 체인(InfraRelayChain)_** 을 시작해야 합니다.

원시(raw) 샘플 체인 스펙 파일을 사용하여 밸리데이터 노드를 시작합니다:

1. 다음 명령을 실행하여 `alice` 계정의 밸리데이터 노드를 시작합니다.
   
   ```bash
   ./target/release/infrablockspace \
   --alice \
   --validator \
   --base-path /tmp/relay/alice \
   --chain ./raw-infra-relay-chainspec.json \
   --port 30333 \
   --rpc-port 9944
   ```

- `--validator`: 밸리데이터 노드를 시작하기 위한 옵션입니다.
- `--base_path`: 체인 데이터가 저장될 위치를 지정합니다.
- `--chain`: 원시(raw) 체인 스펙 파일의 경로를 지정합니다.
- `--port, --rpc-port`: 체인과 통신할 포트를 지정합니다.

   _노드가 시작된 후에는 동일한 로컬 컴퓨터의 다른 노드가 이러한 포트를 사용할 수 없습니다._

2. 노드를 시작하게 되면 다음과 같은 로그를 확인할 수 있습니다:

   ```bash
   2023-11-16 18:05:43 bclabs InfraBlockspace
   2023-11-16 18:05:43 ✌️  version 1.1.0-e146a06602e
   2023-11-16 18:05:43 ❤️  by blockchain labs, 2017-2023
   2023-11-16 18:05:43 📋 Chain specification: Infra Relay Devnet
   2023-11-16 18:05:43 🏷  Node name: Alice
   2023-11-16 18:05:43 👤 Role: AUTHORITY
   2023-11-16 18:05:43 💾 Database: RocksDb at /tmp/relay/alice/chains/infra_relay_devnet/db/full
   2023-11-16 18:05:44 BEEFY is still experimental, usage on Polkadot network is discouraged.
   2023-11-16 18:05:45 👶 Creating empty BABE epoch changes on what appears to be first startup. 
   2023-11-16 18:05:45 🏷  Local node identity is: 12D3KooWBJyq6nyn6JJSmdy5QmemYBCofKvWh6w5Am6p33tYzxu1

   ...
   ```

2. 다른 노드가 연결할 수 있도록 `alice` 노드 `peerid` 를 기록하세요.
   
   ```bash
   🏷 Local node identity is: 12D3KooWBJyq6nyn6JJSmdy5QmemYBCofKvWh6w5Am6p33tYzxu1
   ```

3. 새 터미널을 열고 `bob` 계정의 두 번째 밸리데이터 노드를 시작합니다:

   ```bash
   ./target/release/infrablockspace \
   --bob \
   --validator \
   --base-path /tmp/relay/bob \
   --chain ./raw-infra-relay-chainspec.json \
   --port 30334 \
   --rpc-port 9945 
   ```

   _첫 번째 노드를 시작하는 데 사용한 명령과 유사한 명령을 사용하지만 몇 가지 중요한 차이점이 있습니다._
   
- 이 명령은 다른 베이스 경로(`/tmp/relay/bob`), 검증자 키(`--bob`), 포트(`30334` 및 `9945`)를 사용합니다.
   
- 두 검증자가 동일한 로컬 컴퓨터에서 실행되므로 체인 스펙 파일에 지정된 첫 번째 노드의 IP 주소와 피어 식별자를 지정할 필요가 없습니다. `bootnodes` 옵션은 로컬 네트워크 외부에서 실행되는 노드나 체인 스펙 파일에 식별되지 않은 노드에 연결하려는 경우 필요합니다.

- 릴레이 체인이 블록을 생성하지 않는 경우 방화벽을 비활성화하거나 `alice` 노드의 주소를 `--bootnodes` 옵션에 추가하여 노드를 시작해 보세요. 

   예시,
   ```bash
   ./target/release/infrablockspace \
   --bob \
   --validator \
   --base-path /tmp/relay/bob \
   --chain ./raw-infra-relay-chainspec.json \
   --port 30334 \
   --rpc-port 9945 
   --bootnodes /ip4/127.0.0.1/tcp/30333/p2p/12D3KooWGjsmVmZCM1jPtVNp6hRbbkGBK3LADYNniJAKJ19NUYiq
   ```

- `bob` 의 노드까지 시작하게 되면 블록이 생성되는 것을 확인할 수 있습니다. 

   ```bash
   2023-11-16 18:08:57 discovered: 12D3KooWP2vgMARpeD9H5innUPaBR7LFQqJSP6dX4TRS9DtkqsBQ /ip4/172.16.72.194/tcp/30334
   2023-11-16 18:09:00 💤 Idle (1 peers), best: #0 (0xe521…3cca), finalized #0 (0xe521…3cca), ⬇ 1.5kiB/s ⬆ 1.5kiB/s
   2023-11-16 18:09:05 💤 Idle (1 peers), best: #0 (0xe521…3cca), finalized #0 (0xe521…3cca), ⬇ 0.2kiB/s ⬆ 0.2kiB/s
   2023-11-16 18:09:10 💤 Idle (1 peers), best: #0 (0xe521…3cca), finalized #0 (0xe521…3cca), ⬇ 0 ⬆ 0
   2023-11-16 18:09:12 🙌 Starting consensus session on top of parent 0xe5212b368879d4a38e84693a0f1582402ac100948a895217823de534cf753cca
   🎁 Prepared block for proposing at 1 (2 ms) [hash: 0x2a02735687bb7ec53f34e17424a313b8b05ecce8ac855216dfae3c254980efdc; parent_hash: 0xe521…3cca; extrinsics (2): [0x62c3…6593, 0xf265…0515]
   ```


## 다음 단계로 넘어가기

- [파라체인 구축하기](./build-a-parachain.md)