---
title: 네트워크 시뮬레이션
description: 미리 정의된 계정을 사용하여 권한이 부여된 유효성 검사자로 개인 블록체인 네트워크를 시작합니다.
keywords:
---

이 튜토리얼은 **권한 집합**으로 개인 **유효성 검사자**를 사용하여 개인 블록체인 네트워크를 시작하는 방법에 대한 기본 소개를 제공합니다.

Substrate 노드 템플릿은 권한 합의 모델을 사용하여 블록 생성을 권한이 있는 계정의 회전 목록으로 제한합니다.
권한이 있는 계정인 **권한자**는 라운드 로빈 방식으로 블록을 생성하는 역할을 담당합니다.

이 튜토리얼에서는 권한 합의 모델이 어떻게 작동하는지를 확인하기 위해 사전에 정의된 계정을 사용하여 노드가 블록을 생성할 수 있도록 하는 권한자로서의 역할을 하는 두 개의 계정을 사용하여 시뮬레이션된 네트워크를 시작하는 방법을 살펴보겠습니다.
이 시뮬레이션된 네트워크에서 두 개의 노드는 서로 다른 계정과 키를 사용하여 시작되지만 하나의 컴퓨터에서 실행됩니다.

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [Rust와 Rust 툴체인](/install/)을 설치하여 Substrate 개발을 위한 환경을 구성했는지 확인하세요.

- [로컬 블록체인 구축](/tutorials/build-a-blockchain/build-local-blockchain/)을 완료하고 Substrate 노드 템플릿을 로컬에 설치했는지 확인하세요.

- 소프트웨어 개발과 명령 줄 인터페이스 사용에 대해 일반적으로 알고 있는지 확인하세요.

- 블록체인과 스마트 컨트랙트 플랫폼에 대해 일반적으로 알고 있는지 확인하세요.

## 튜토리얼 목표

이 튜토리얼을 완료함으로써 다음 목표를 달성할 수 있습니다:

- 사전에 정의된 계정을 사용하여 블록체인 노드를 시작합니다.

- 노드를 시작하는 데 사용되는 주요 명령 줄 옵션을 배웁니다.

- 노드가 실행 중이며 블록을 생성하는지 확인합니다.

- 실행 중인 네트워크에 두 번째 노드를 연결합니다.

- 피어 컴퓨터가 블록을 생성하고 최종화하는지 확인합니다.

## 첫 번째 블록체인 노드 시작하기

자체 Substrate 네트워크를 시작하기 전에 사전에 정의된 네트워크 사양인 `local`을 사용하고 미리 정의된 사용자 계정에서 실행하여 기본 원리를 배울 수 있습니다.

이 튜토리얼에서는 `alice`와 `bob`이라는 이름의 미리 정의된 계정을 사용하여 한 대의 로컬 컴퓨터에서 두 개의 Substrate 노드를 실행하여 개인 네트워크를 시뮬레이션합니다.

블록체인을 시작하려면 다음 단계를 따르세요:

1. 컴퓨터에서 터미널 쉘을 엽니다.

1. Substrate 노드 템플릿을 컴파일한 루트 디렉토리로 이동합니다.

1. 다음 명령을 실행하여 이전 체인 데이터를 제거합니다:

   ```bash
   ./target/release/node-template purge-chain --base-path /tmp/alice --chain local
   ```

   명령은 작업을 확인하도록 요청합니다:

   ```bash
   "/tmp/alice/chains/local_testnet/db"를 삭제하시겠습니까? [y/N]:
   ```

1. `y`를 입력하여 체인 데이터를 제거하려는 것을 확인합니다.

   새 네트워크를 시작할 때마다 이전 체인 데이터를 항상 제거해야 합니다.

1. 다음 명령을 실행하여 `alice` 계정을 사용하여 로컬 블록체인 노드를 시작합니다:

   ```bash
   ./target/release/node-template \
   --base-path /tmp/alice \
   --chain local \
   --alice \
   --port 30333 \
   --rpc-port 9945 \
   --node-key 0000000000000000000000000000000000000000000000000000000000000001 \
   --telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
   --validator
   ```

### 명령 줄 옵션 검토

계속하기 전에 다음 옵션을 사용하여 노드를 시작하는 방법을 살펴보세요.

| <div style="min-width:110pt;font-weight:bold;">옵션</div> | <div style="font-weight:bold;">설명</div>                                                                                                                                                                                           |
| :---------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--base-path`                                               | 이 체인과 관련된 모든 데이터를 저장할 디렉토리를 지정합니다.                                                                                                                                                                 |
| `--chain local`                                             | 사용할 체인 스펙을 지정합니다. 유효한 사전 정의된 체인 스펙으로는 `local`, `development`, `staging`이 있습니다.                                                                                                             |
| `--alice`                                                   | `alice` 계정의 미리 정의된 키를 노드의 키스토어에 추가합니다. 이 설정으로 `alice` 계정이 블록 생성과 최종화에 사용됩니다.                                                                             |
| `--port 30333`                                              | 피어 간 (`p2p`) 트래픽을 수신할 포트를 지정합니다. 이 튜토리얼에서는 네트워크를 시뮬레이션하기 위해 동일한 컴퓨터에서 실행되는 두 개의 노드를 사용하므로 적어도 하나의 계정에 대해 명시적으로 다른 포트를 지정해야 합니다. |
| `--rpc-port 9945`                                            | 서버가 WebSocket 및 HTTP를 통해 수신하는 JSON-RPC 트래픽을 위한 포트를 지정합니다. 기본 포트는 `9944`입니다. 이 튜토리얼에서는 사용자 정의 웹 소켓 포트 번호(`9945`)를 사용합니다.                                                                                   |
| `--node-key <key>`                                          | `libp2p` 네트워킹에 사용할 Ed25519 비밀 키를 지정합니다. 이 옵션은 개발 및 테스트에만 사용해야 합니다.                                                                                                              |
| `--telemetry-url`                                           | 텔레메트리 데이터를 보낼 위치를 지정합니다. 이 튜토리얼에서는 누구나 사용할 수 있는 Parity에서 호스팅되는 서버로 텔레메트리 데이터를 보낼 수 있습니다.                                                                                   |
| `--validator`                                               | 이 노드가 네트워크의 블록 생성과 최종화에 참여한다는 것을 지정합니다.                                                                                                                                                |

노드 템플릿에 대해 사용 가능한 명령 줄 옵션에 대한 자세한 내용은 다음 명령을 실행하여 사용법 도움말을 확인하세요:

`./target/release/node-template --help`

### 표시된 노드 메시지 검토

노드가 성공적으로 시작되면 터미널에 네트워크 작업에 대한 메시지가 표시됩니다.
예를 들어, 다음과 유사한 출력을 볼 수 있어야 합니다:

```text
2022-08-16 15:29:55 Substrate Node    
2022-08-16 15:29:55 ✌️  version 4.0.0-dev-de262935ede    
2022-08-16 15:29:55 ❤️  by Substrate DevHub <https://github.com/substrate-developer-hub>, 2017-2022    
2022-08-16 15:29:55 📋 Chain specification: Local Testnet    
2022-08-16 15:29:55 🏷  Node name: Alice    
2022-08-16 15:29:55 👤 Role: AUTHORITY    
2022-08-16 15:29:55 💾 Database: RocksDb at /tmp/alice/chains/local_testnet/db/full    
2022-08-16 15:29:55 ⛓  Native runtime: node-template-100 (node-template-1.tx1.au1)    
2022-08-16 15:29:55 🔨 Initializing Genesis block/state (state: 0x6894…033d, header-hash: 0x2cdc…a07f)    
2022-08-16 15:29:55 👴 Loading GRANDPA authority set from genesis on what appears to be first startup.    
2022-08-16 15:29:56 Using default protocol ID "sup" because none is configured in the chain specs    
2022-08-16 15:29:56 🏷  Local node identity is: 12D3KooWEyoppNCUx8Yx66oV9fJnriXwCcXwDDUA2kj6vnc6iDEp    
2022-08-16 15:29:56 💻 Operating system: macos    
2022-08-16 15:29:56 💻 CPU architecture: x86_64    
2022-08-16 15:29:56 📦 Highest known block at #0    
2022-08-16 15:29:56 〽️ Prometheus exporter started at 127.0.0.1:9615    
2022-08-16 15:29:56 Running JSON-RPC server: addr=127.0.0.1:9945, allowed origins=Some(["http://localhost:*", "http://127.0.0.1:*", "https://localhost:*", "https://127.0.0.1:*", "https://polkadot.js.org"])      
2022-08-16 15:29:56 creating instance on iface 192.168.1.125    
2022-08-16 15:30:01 💤 Idle (0 peers), best: #0 (0x2cdc…a07f), finalized #0 (0x2cdc…a07f), ⬇ 0 ⬆ 0
...
```

특히, 출력에서 다음 메시지를 주목해야 합니다:

- `🔨 Initializing Genesis block/state (state: 0xea47…9ba8, header-hash: 0x9d07…7cce)`는 노드가 사용하는 초기 또는 **제네시스 블록**을 식별합니다.
  다음 노드를 시작할 때 이 값이 동일한지 확인하세요.

- `🏷 Local node identity is: 12D3KooWEyoppNCUx8Yx66oV9fJnriXwCcXwDDUA2kj6vnc6iDEp`는 이 노드를 고유하게 식별하는 문자열을 지정합니다.
  이 문자열은 `alice` 계정을 사용하여 노드를 시작할 때 사용한 `--node-key`에 의해 결정됩니다.
  이 문자열은 두 번째 노드를 시작할 때 노드에 연결하기 위해 사용합니다.
- `2021-03-10 17:34:37 💤 Idle (0 peers), best: #0 (0x9d07…7cce), finalized #0 (0x9d07…7cce), ⬇ 0 ⬆ 0`는 네트워크에 다른 노드가 없고 블록이 생성되지 않고 있다는 것을 나타냅니다.
  블록이 생성되기 위해서는 다른 노드가 네트워크에 참여해야 합니다.

## 블록체인 네트워크에 두 번째 노드 추가하기

`alice` 계정 키를 사용하여 시작한 노드가 실행 중인 상태에서 `bob` 계정을 사용하여 네트워크에 다른 노드를 추가할 수 있습니다.
이미 실행 중인 네트워크에 참여하기 때문에 실행 중인 노드를 사용하여 새 노드가 참여할 네트워크를 식별할 수 있습니다.
명령은 이전에 사용한 것과 유사하지만 몇 가지 중요한 차이점이 있습니다.

실행 중인 블록체인에 노드를 추가하려면 다음 단계를 따르세요:

1. 컴퓨터에서 **새로운** 터미널 쉘을 엽니다.

1. Substrate 노드 템플릿을 컴파일한 루트 디렉토리로 이동합니다.

1. 다음 명령을 실행하여 이전 체인 데이터를 제거합니다:

   ```bash
   ./target/release/node-template purge-chain --base-path /tmp/bob --chain local -y
   ```

   `-y`를 명령에 추가하여 확인 없이 체인 데이터를 제거할 수 있습니다.

1. 다음 명령을 실행하여 `bob` 계정을 사용하여 두 번째 로컬 블록체인 노드를 시작합니다:

   ```bash
   ./target/release/node-template \
   --base-path /tmp/bob \
   --chain local \
   --bob \
   --port 30334 \
   --rpc-port 9946 \
   --telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
   --validator \
   --bootnodes /ip4/127.0.0.1/tcp/30333/p2p/12D3KooWEyoppNCUx8Yx66oV9fJnriXwCcXwDDUA2kj6vnc6iDEp
   ```

   이 명령과 이전 명령 사이에 다음과 같은 차이점을 주목하세요:

   - 두 개의 노드가 동일한 컴퓨터에서 실행되므로 `--base-path`, `--port`, `--rpc-port` 옵션에 대해 다른 값을 지정해야 합니다.

   - 이 명령에는 `--bootnodes` 옵션이 포함되어 있으며 시작한 `alice` 노드를 단일 부트 노드로 지정합니다.

   `--bootnodes` 옵션은 다음 정보를 지정합니다:

   - `ip4`는 노드의 IP 주소가 IPv4 형식을 사용함을 나타냅니다.

   - `127.0.0.1`은 실행 중인 노드의 IP 주소를 지정합니다.
     이 경우 `localhost`의 주소입니다.

   - `tcp`는 피어 간 통신에 사용되는 프로토콜로 TCP를 지정합니다.

   - `30333`은 피어 간 통신에 사용되는 포트 번호를 지정합니다.
     이 경우 TCP 트래픽을 위한 포트 번호입니다.

   - `12D3KooWEyoppNCUx8Yx66oV9fJnriXwCcXwDDUA2kj6vnc6iDEp`은 이 네트워크에 대해 통신할 실행 중인 노드를 식별합니다.
     이 경우 `alice` 계정을 사용하여 시작한 노드의 식별자입니다.

## 블록 생성 및 최종화 확인하기

두 번째 노드를 시작한 후, 노드는 서로 피어로 연결되어 블록을 생성하기 시작해야 합니다.

블록이 최종화되고 있는지 확인하려면 다음 단계를 따르세요:

1. 첫 번째 노드를 시작한 터미널에서 다음과 유사한 줄을 확인하세요:

   ```text
   2022-08-16 15:32:33 discovered: 12D3KooWBCbmQovz78Hq7MzPxdx9d1gZzXMsn6HtWj29bW51YUKB /ip4/127.0.0.1/tcp/30334
   2022-08-16 15:32:33 discovered: 12D3KooWBCbmQovz78Hq7MzPxdx9d1gZzXMsn6HtWj29bW51YUKB /ip6/::1/tcp/30334
   2022-08-16 15:32:36 🙌 Starting consensus session on top of parent 0x2cdce15d31548063e89e10bd201faa63c623023bbc320346b9580ed3c40fa07f
   2022-08-16 15:32:36 🎁 Prepared block for proposing at 1 (5 ms) [hash: 0x9ab34110e4617454da33a3616efc394eb1ce95ee4bf0daab69aa4cb392d4104b; parent_hash: 0x2cdc…a07f; extrinsics (1): [0x4634…cebf]] 
   2022-08-16 15:32:36 🔖 Pre-sealed block for proposal at 1. Hash now 0xf0869a5cb8ebd0fcc5f2bc194ced84ca782d9749604e888c8b9b515517179847, previously 0x9ab34110e4617454da33a3616efc394eb1ce95ee4bf0daab69aa4cb392d4104b.
   2022-08-16 15:32:36 ✨ Imported #1 (0xf086…9847)
   2022-08-16 15:32:36 💤 Idle (1 peers), best: #1 (0xf086…9847), finalized #0 (0x2cdc…a07f), ⬇ 1.0kiB/s ⬆ 1.0kiB/s
   2022-08-16 15:32:41 💤 Idle (1 peers), best: #1 (0xf086…9847), finalized #0 (0x2cdc…a07f), ⬇ 0.6kiB/s ⬆ 0.6kiB/s
   2022-08-16 15:32:42 ✨ Imported #2 (0x0d5e…2a7f)
   2022-08-16 15:32:46 💤 Idle (1 peers), best: #2 (0x0d5e…2a7f), finalized #0 (0x2cdc…a07f), ⬇ 0.6kiB/s ⬆ 0.6kiB/s
   2022-08-16 15:32:48 🙌 Starting consensus session on top of parent 0x0d5ef31979c2aa17fb88497018206d3057151119337293fe85d9526ebd1e2a7f
   2022-08-16 15:32:48 🎁 Prepared block for proposing at 3 (0 ms) [hash: 0xa307c0112bce39e0dc689132452154da2079a27375b44c4d94790b46a601346a; parent_hash: 0x0d5e…2a7f; extrinsics (1): [0x63cc…39a6]]    
   2022-08-16 15:32:48 🔖 Pre-sealed block for proposal at 3. Hash now 0x0c55670e745dd12892c9e7d5205085a78ccea98df393a822fa9b3865cfb3d51b, previously 0xa307c0112bce39e0dc689132452154da2079a27375b44c4d94790b46a601346a.
   2022-08-16 15:32:48 ✨ Imported #3 (0x0c55…d51b)
   2022-08-16 15:32:51 💤 Idle (1 peers), best: #3 (0x0c55…d51b), finalized #1 (0xf086…9847), ⬇ 0.7kiB/s ⬆ 0.9kiB/s    
   ...
   ```

   이러한 줄에서 블록체인에 대한 다음 정보를 확인할 수 있습니다:

   - 두 번째 노드 식별자가 네트워크에서 발견되었습니다 (`12D3KooWBCbmQovz78Hq7MzPxdx9d1gZzXMsn6HtWj29bW51YUKB`).
   - 노드에는 하나의 피어가 있습니다 (`1 peers`).
   - 노드가 일부 블록을 생성했습니다 (`best: #3 (0x0c55…d51b)`).
   - 블록이 최종화되고 있습니다 (`finalized #1 (0xf086…9847)`).

1. 두 번째 노드를 시작한 터미널에서도 유사한 출력을 확인하세요.

1. 한 노드를 종료하려면 터미널 쉘에서 Control-c를 누릅니다.
   
   노드를 종료한 후, 남아 있는 노드가 피어가 없고 블록 생성을 중지한 것을 확인할 수 있습니다.
   예를 들어:

   ```text
   2022-08-16 15:53:45 💤 Idle (1 peers), best: #143 (0x8f11…1684), finalized #141 (0x5fe3…5a25), ⬇ 0.8kiB/s ⬆ 0.7kiB/s
   2022-08-16 15:53:50 💤 Idle (0 peers), best: #143 (0x8f11…1684), finalized #141 (0x5fe3…5a25), ⬇ 83 B/s ⬆ 83 B/s
   ```  

2. 두 번째 노드를 종료하려면 터미널 쉘에서 Control-c를 누릅니다.
   
   시뮬레이션된 네트워크에서 체인 상태를 제거하려면 `purge-chain` 하위 명령을 `/tmp/bob` 및 `/tmp/alice` 디렉토리에 대한 `--base-path` 명령 줄 옵션과 함께 사용하세요.

## 다음 단계

이 튜토리얼에서는 개인 블록체인 네트워크를 시작하는 첫 번째 기본 단계를 소개했습니다.
이 튜토리얼에서는 두 개의 노드를 한 대의 컴퓨터에서 실행하고 사전에 정의된 계정을 참여자로 사용하여 개인 네트워크를 시뮬레이션했습니다.

다음을 배웠습니다:

- 노드 템플릿 명령과 명령 줄 옵션을 사용하는 방법.

- 서로 피어로 통신하는 두 개의 블록체인 노드를 시작하는 방법.

- 개인 블록체인 노드가 블록을 생성하는지 확인하는 방법.

다음 튜토리얼에서는 이 튜토리얼에서 배운 내용을 기반으로 다른 참여자와 별도의 컴퓨터에서 실행되는 노드를 사용하여 개인 네트워크를 시작하는 방법을 설명합니다.

[신뢰할 수 있는 노드 추가](/tutorials/build-a-blockchain/add-trusted-nodes/)에서 다음을 배울 수 있습니다:

- 자체 비밀 키 쌍을 생성하는 방법.

- 생성한 키를 사용하는 사용자 정의 체인 스펙을 생성하는 방법.

- 사용자 정의 체인 스펙을 사용하는 개인 네트워크에 유효성 검사자를 추가하는 방법.

이 튜토리얼에 문제가 있거나 질문이 있거나 피드백을 제공하려면 다음을 이용하세요.

- [이슈 제출](https://github.com/substrate-developer-hub/substrate-docs/issues/new/choose).

- [Substrate Stack Exchange](https://substrate.stackexchange.com/).