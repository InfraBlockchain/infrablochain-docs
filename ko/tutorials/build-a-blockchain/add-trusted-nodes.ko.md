---
title: 신뢰할 수 있는 노드 추가
description: 개인 블록체인 네트워크를 위한 계정 키와 사용자 정의 체인 스펙 생성하기
keywords:
  - 기업
  - 개인
  - 허가
---

이 튜토리얼은 개인 **검증자**의 **권한 집합**을 가진 작은 독립형 블록체인 네트워크를 시작하는 방법을 보여줍니다.

[블록체인 기본](/learn/blockchain-basics/)에서 배운 대로, 모든 블록체인은 네트워크의 노드들이 특정 시점에서 데이터의 상태에 대해 동의하는 것을 요구하며, 이 상태에 대한 동의는 **합의**라고 합니다.

Substrate 노드 템플릿은 **권한 라운드** 또는 **Aura** 합의로도 알려진 권한 증명 합의 모델을 사용합니다.
Aura 합의 프로토콜은 블록 생성을 권한이 있는 계정의 회전 목록으로 제한합니다.
권한이 있는 계정인 **권한자**들은 라운드 로빈 방식으로 블록을 생성하며, 일반적으로 네트워크에서 신뢰할 수 있는 참여자로 간주됩니다.

이 합의 모델은 제한된 참여자 수를 위한 솔로 블록체인을 시작하는 간단한 방법을 제공합니다.
이 튜토리얼에서는 네트워크에 참여할 노드에 권한을 부여하기 위해 필요한 키를 생성하는 방법, 네트워크에 대한 구성 및 정보를 다른 권한이 부여된 계정과 공유하는 방법, 승인된 검증자 집합으로 네트워크를 시작하는 방법을 살펴볼 것입니다.

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [Rust 및 Rust 툴체인](/install/)을 설치하여 Substrate 개발 환경을 구성했는지 확인하세요.

- [로컬 블록체인 빌드](/tutorials/build-a-blockchain/build-local-blockchain/)을 완료하고 Substrate 노드 템플릿을 로컬에 설치했는지 확인하세요.

- [네트워크 시뮬레이션](/tutorials/build-a-blockchain/simulate-network/)에서 설명한 대로 미리 정의된 계정을 사용하여 단일 컴퓨터에서 노드를 시작했는지 확인하세요.

- 소프트웨어 개발 및 명령 줄 인터페이스 사용에 대해 일반적으로 알고 있는지 확인하세요.

- 블록체인 및 스마트 컨트랙트 플랫폼에 대해 일반적으로 알고 있는지 확인하세요.

## 튜토리얼 목표

이 튜토리얼을 완료하면 다음 목표를 달성할 수 있습니다:

- 네트워크 권한으로 사용할 키 쌍 생성하기.

- 사용자 정의 체인 스펙 파일 생성하기.

- 개인 두 노드 블록체인 네트워크 시작하기.

## 계정 및 키 생성하기

[네트워크 시뮬레이션](/tutorials/build-a-blockchain/simulate-network/)에서는 미리 정의된 계정과 키를 사용하여 피어 노드를 시작했습니다.
이 튜토리얼에서는 네트워크의 검증자 노드에 대한 키를 생성하기 위해 직접 비밀 키를 생성합니다.
블록체인 네트워크의 각 참여자는 고유한 키를 생성하는 것이 중요하다는 점을 기억해야 합니다.

### 키 생성 옵션

키를 생성하는 여러 가지 방법이 있습니다.
예를 들어, `node-template` 하위 명령, 독립형 [subkey](/reference/command-line-tools/subkey/) 명령 줄 프로그램, Polkadot-JS 애플리케이션 또는 타사 키 생성 도구를 사용하여 키 쌍을 생성할 수 있습니다.

이 튜토리얼에서는 미리 정의된 키 쌍을 사용하여 튜토리얼을 완료할 수 있지만, 이러한 키는 실제 환경에서 사용해서는 안 됩니다.
미리 정의된 키 쌍이나 보안이 더 강화된 `subkey` 프로그램 대신에 Substrate 노드 템플릿과 `key` 하위 명령을 사용하여 키를 생성하는 방법을 보여주기 위해 이 튜토리얼에서는 키를 생성하는 방법을 설명합니다.

### 노드 템플릿을 사용하여 로컬 키 생성하기

생산용 블록체인에 사용할 키를 생성할 때는 인터넷에 연결된 적이 없는 오프라인 컴퓨터를 사용하는 것이 좋습니다.
최소한 인터넷에 연결을 끊은 후에 공개 또는 제어되지 않는 공공 또는 개인 블록체인에서 사용할 키를 생성하기 전에 인터넷에 연결을 끊어야 합니다.

그러나 이 튜토리얼에서는 인터넷에 연결된 상태에서 로컬로 무작위 키를 생성하기 위해 `node-template` 명령 줄 옵션을 사용할 수 있습니다.

노드 템플릿을 사용하여 키를 생성하려면 다음 단계를 따르세요:

1. 컴퓨터에서 터미널 쉘을 엽니다.

1. Substrate 노드 템플릿을 컴파일한 루트 디렉토리로 이동합니다.

1. 다음 명령을 실행하여 무작위 비밀 문구와 키를 생성합니다:

   ```bash
   ./target/release/node-template key generate --scheme Sr25519 --password-interactive
   ```

1. 생성된 키에 대한 비밀번호를 입력합니다.

   이 명령은 키를 생성하고 다음과 유사한 출력을 표시합니다:

   ```text
   Secret phrase:  pig giraffe ceiling enter weird liar orange decline behind total despair fly
   Secret seed:       0x0087016ebbdcf03d1b7b2ad9a958e14a43f2351cd42f2f0a973771b90fb0112f
   Public key (hex):  0x1a4cc824f6585859851f818e71ac63cf6fdc81018189809814677b2a4699cf45
   Account ID:        0x1a4cc824f6585859851f818e71ac63cf6fdc81018189809814677b2a4699cf45
   Public key (SS58): 5CfBuoHDvZ4fd8jkLQicNL8tgjnK8pVG9AiuJrsNrRAx6CNW
   SS58 Address:      5CfBuoHDvZ4fd8jkLQicNL8tgjnK8pVG9AiuJrsNrRAx6CNW
   ```

   이제 `aura`를 사용하여 블록을 생성하기 위한 Sr25519 키를 가지고 있습니다.
   이 예제에서 계정의 Sr25519 공개 키는 `5CfBuoHDvZ4fd8jkLQicNL8tgjnK8pVG9AiuJrsNrRAx6CNW`입니다.

1. `key` 하위 명령을 사용하여 방금 생성한 계정의 **비밀 문구**를 사용하여 키를 파생시킵니다.

   예를 들어, 다음과 유사한 명령을 실행합니다:

   ```bash
   ./target/release/node-template key inspect --password-interactive --scheme Ed25519 "pig giraffe ceiling enter weird liar orange decline behind total despair fly"
   ```

1. 키를 생성할 때 사용한 비밀번호를 입력합니다.

   이 명령은 다음과 유사한 출력을 표시합니다:

   ```text
   Secret phrase `pig giraffe ceiling enter weird liar orange decline behind total despair fly` is account:
   Secret seed:       0x0087016ebbdcf03d1b7b2ad9a958e14a43f2351cd42f2f0a973771b90fb0112f
   Public key (hex):  0x2577ba03f47cdbea161851d737e41200e471cd7a31a5c88242a527837efc1e7b
   Public key (SS58): 5CuqCGfwqhjGzSqz5mnq36tMe651mU9Ji8xQ4JRuUTvPcjVN
   Account ID:        0x2577ba03f47cdbea161851d737e41200e471cd7a31a5c88242a527837efc1e7b
   SS58 Address:      5CuqCGfwqhjGzSqz5mnq36tMe651mU9Ji8xQ4JRuUTvPcjVN
   ```

   이제 `grandpa`를 사용하여 블록을 최종화하기 위한 Ed25519 키를 가지고 있습니다.
   이 예제에서 계정의 Ed25519 공개 키는 `5CuqCGfwqhjGzSqz5mnq36tMe651mU9Ji8xQ4JRuUTvPcjVN`입니다.

### 두 번째 키 쌍 생성하기

이 튜토리얼에서는 개인 네트워크가 두 개의 노드로 구성되므로 두 개의 키 쌍이 필요합니다.
튜토리얼을 계속 진행하기 위해 다음 옵션 중 하나를 선택할 수 있습니다:

- 미리 정의된 계정 중 하나의 키를 사용할 수 있습니다.

- 이전 섹션의 단계를 반복하여 로컬 컴퓨터에서 다른 신원으로 두 번째 키 쌍을 생성할 수 있습니다.

- 로컬 컴퓨터에서 두 번째 신원을 시뮬레이션하기 위해 자식 키 쌍을 파생시킬 수 있습니다.

- 개인 네트워크에 참여할 수 있는 키를 생성하기 위해 다른 참여자를 모집할 수 있습니다.

이 튜토리얼에서는 두 번째 키 쌍으로 다음을 사용합니다:

- Sr25519: `aura`에 대한 5EJPj83tJuJtTVE2v7B9ehfM7jNT44CBFaPWicvBwYyUKBS6.
- Ed25519: `grandpa`에 대한 5FeJQsfmbbJLTH1pvehBxrZrT5kHvJFj84ZaY5LK7NU87gZS.

## 사용자 정의 체인 스펙 생성하기

블록체인에 사용할 키를 생성한 후, 해당 키 쌍을 사용하여 사용자 정의 **체인 스펙**을 생성하고, 이 사용자 정의 체인 스펙을 **검증자**라고 불리는 신뢰할 수 있는 네트워크 참여자와 공유할 수 있도록 준비합니다.

다른 사람들이 개인 블록체인 네트워크에 참여할 수 있도록 하려면, 그들이 고유한 키를 생성하도록 해야 합니다.
네트워크 참여자의 키를 수집한 후, `local` 체인 스펙을 대체할 사용자 정의 체인 스펙을 생성할 수 있습니다.

이 튜토리얼에서는 두 노드 네트워크를 보여주기 위해 `local` 체인 스펙을 수정한 사용자 정의 체인 스펙을 생성합니다.
만약 필요한 키를 가지고 있다면, 네트워크에 더 많은 노드를 추가하는 방법은 동일한 단계를 따르면 됩니다.

### 로컬 체인 스펙 수정하기

완전히 새로운 체인 스펙을 작성하는 대신, 미리 정의된 `local` 체인 스펙을 수정할 수 있습니다.

로컬 사양을 기반으로 새로운 체인 스펙을 생성하려면 다음 단계를 따르세요:

1. 컴퓨터에서 터미널 쉘을 엽니다.

1. Substrate 노드 템플릿을 컴파일한 루트 디렉토리로 이동합니다.

1. 다음 명령을 실행하여 `local` 체인 스펙을 `customSpec.json`이라는 파일로 내보냅니다:

   ```bash
   ./target/release/node-template build-spec --disable-default-bootnode --chain local > customSpec.json
   ```

   `customSpec.json` 파일을 텍스트 편집기에서 열면 여러 필드가 포함되어 있는 것을 볼 수 있습니다. 그 중 하나는 `cargo build --release` 명령을 사용하여 빌드한 런타임의 WebAssembly (Wasm) 바이너리입니다.
   WebAssembly (Wasm) 바이너리는 큰 덩어리이므로 필드를 변경해야 하는지 미리보기하기 위해 처음 몇 줄과 마지막 몇 줄을 볼 수 있습니다.

1. 다음 명령을 실행하여 `customSpec.json` 파일의 처음 몇 필드를 미리보기합니다:

   ```bash
   head customSpec.json
   ```

   이 명령은 파일의 처음 필드를 표시합니다.
   예를 들어:

   ```json
   {
     "name": "Local Testnet",
     "id": "local_testnet",
     "chainType": "Local",
     "bootNodes": [],
     "telemetryEndpoints": null,
     "protocolId": null,
     "properties": null,
     "consensusEngine": null,
     "codeSubstitutes": {},
   ```

1. 다음 명령을 실행하여 `customSpec.json` 파일의 마지막 필드를 미리보기합니다:

   ```bash
   tail -n 80 customSpec.json
   ```

   이 명령은 Wasm 바이너리 필드를 포함한 마지막 섹션을 표시합니다. 이 섹션에는 런타임에서 사용되는 `sudo` 및 `balances` 팔렛 등 몇 가지 팔렛의 세부 정보가 포함되어 있습니다.

1. 텍스트 편집기에서 `customSpec.json` 파일을 엽니다.

1. `name` 필드를 수정하여 이 체인 스펙을 사용자 정의 체인 스펙으로 식별합니다.

   예를 들어:

   ```json
   "name": "My Custom Testnet",
   ```

1. `aura` 필드를 수정하여 블록을 생성할 권한이 있는 노드를 지정합니다. 네트워크 참여자마다 Sr25519 SS58 주소 키를 추가합니다.

   ```json
   "aura": { "authorities": [
       "5CfBuoHDvZ4fd8jkLQicNL8tgjnK8pVG9AiuJrsNrRAx6CNW", 
       "5CXGP4oPXC1Je3zf5wEDkYeAqGcGXyKWSRX2Jm14GdME5Xc5"
     ]
   },
   ```

1. `grandpa` 필드를 수정하여 블록을 최종화할 권한이 있는 노드를 지정합니다. 네트워크 참여자마다 Ed25519 SS58 주소 키와 투표 가중치를 추가합니다.

   ```json
   "grandpa": {
       "authorities": [
         [
           "5CuqCGfwqhjGzSqz5mnq36tMe651mU9Ji8xQ4JRuUTvPcjVN",
           1
         ],
         [
           "5DpdMN4bVTMy67TfMMtinQTcUmLhZBWoWarHvEYPM4jYziqm",
           1
         ]
       ]
     },
   ```

   `grandpa` 섹션의 `authorities` 필드에는 두 개의 데이터 값이 있음에 유의하세요.
   첫 번째 값은 주소 키입니다.
   두 번째 값은 **가중 투표**를 지원하기 위해 사용됩니다.
   이 예제에서 각 검증자의 가중치는 **1** 투표입니다.

1. 변경 사항을 저장하고 파일을 닫습니다.

### 검증자 추가하기

앞에서 보았듯이, `aura` 및 `grandpa` 섹션을 수정하여 체인 스펙에 권한 주소를 추가하거나 변경할 수 있습니다.
이 기술을 사용하여 원하는 만큼 많은 검증자를 추가할 수 있습니다.

검증자를 추가하려면 다음을 수행하세요:

- `aura` 섹션을 수정하여 **Sr25519** 주소를 포함합니다.

- `grandpa` 섹션을 수정하여 **Ed25519** 주소와 투표 가중치를 포함합니다.

각 검증자에 대해 고유한 키를 사용해야 합니다.
두 개의 검증자가 동일한 키를 가지고 있는 경우 충돌하는 블록이 생성됩니다.

키 쌍 및 서명 작업에 대한 자세한 내용은 [공개 키 암호화](/learn/cryptography/#public-key-cryptography)를 참조하세요.

## 체인 스펙을 원시(raw) 형식으로 변환하기

검증자 정보가 포함된 체인 스펙을 준비한 후, 사용하기 전에 원시(raw) 사양 형식으로 변환해야 합니다.
원시(raw) 체인 스펙에는 변환되지 않은 사양과 동일한 정보가 포함되어 있습니다.
그러나 원시(raw) 체인 스펙에는 노드가 로컬 저장소에서 데이터를 참조하는 데 사용하는 인코딩된 저장소 키도 포함되어 있습니다.
원시(raw) 체인 스펙을 배포하면 각 노드가 올바른 저장소 키를 사용하여 데이터를 저장하도록 보장할 수 있습니다.

체인 스펙을 원시(raw) 형식으로 변환하려면 다음을 수행하세요:

1. 컴퓨터에서 터미널 쉘을 엽니다.

1. Substrate 노드 템플릿을 컴파일한 루트 디렉토리로 이동합니다.

1. 다음 명령을 실행하여 `customSpec.json` 체인 스펙을 `customSpecRaw.json` 파일로 원시(raw) 형식으로 변환합니다:

   ```bash
   ./target/release/node-template build-spec --chain=customSpec.json --raw --disable-default-bootnode > customSpecRaw.json
   ```

## 개인 네트워크 시작하기

사용자 정의 체인 스펙을 모든 네트워크 참여자에게 배포한 후, 자체 개인 블록체인을 시작할 준비가 되었습니다.
이 단계는 [첫 번째 블록체인 노드 시작](/tutorials/build-a-blockchain/simulate-network/#Start-the-first-blockchain-node)에서 수행한 단계와 유사합니다.
하지만 이 튜토리얼의 단계를 따르면 여러 컴퓨터를 네트워크에 추가할 수 있습니다.

계속하기 전에 다음을 확인하세요:

- 적어도 두 개의 권한 계정에 대한 계정 키를 생성하거나 수집했는지 확인하세요.

- 사용자 정의 체인 스펙을 업데이트하여 블록 생성 (`aura`) 및 블록 최종화 (`grandpa`)를 위한 키를 포함했는지 확인하세요.

- 사용자 정의 체인 스펙을 원시(raw) 형식으로 변환하고, 원시(raw) 체인 스펙을 개인 네트워크에 참여하는 노드에게 배포했는지 확인하세요.

이러한 단계를 완료한 경우, 개인 블록체인에서 첫 번째 노드를 시작할 준비가 되었습니다.

## 첫 번째 노드 시작하기

개인 블록체인 네트워크의 첫 번째 참여자로서, 첫 번째 노드인 **부트노드**를 시작하는 책임이 있습니다.

첫 번째 노드를 시작하려면 다음을 수행하세요:

1. 컴퓨터에서 터미널 쉘을 엽니다.

1. Substrate 노드 템플릿을 컴파일한 루트 디렉토리로 이동합니다.

1. 다음 명령을 실행하여 커스텀 체인 스펙을 사용하여 첫 번째 노드를 시작합니다:

   ```bash
   ./target/release/node-template \
      --base-path /tmp/node01 \
      --chain ./customSpecRaw.json \
      --port 30333 \
      --rpc-port 9945 \
      --telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
      --validator \
      --rpc-methods Unsafe \
      --name MyNode01 \
      --password-interactive
   ```
노드01 키를 생성할 때 사용한 비밀번호를 입력하라는 메시지가 표시됩니다.

다음 명령줄 옵션을 사용하여 노드를 시작하는데 유의하세요:

- `--base-path` 명령줄 옵션은 이 첫 번째 노드와 관련된 체인을 위한 사용자 정의 위치를 지정합니다.

- `--chain` 명령줄 옵션은 사용자 정의 체인 스펙을 지정합니다.

- `--validator` 명령줄 옵션은 이 노드가 체인의 권한자임을 나타냅니다.

- `--rpc-methods Unsafe` 명령줄 옵션은 이 블록체인이 실제 환경에서 사용되지 않고 있기 때문에 안전하지 않은 통신 모드를 사용하여 튜토리얼을 계속할 수 있도록 허용합니다.

- `--name` 명령줄 옵션을 사용하여 텔레메트리 UI에서 노드에 대한 사람이 읽을 수 있는 이름을 지정할 수 있습니다.

이 명령은 미리 정의된 계정 대신에 자체 키를 사용하여 노드를 시작합니다.
알려진 키가 있는 미리 정의된 계정을 사용하지 않으므로, 별도의 단계에서 키를 키스토어에 추가해야 합니다.

## 노드 작업 정보 보기

로컬 노드를 시작한 후, 작업에 대한 정보가 터미널 쉘에 표시됩니다.
해당 터미널에서 다음과 유사한 출력을 확인하세요:

```text
2021-11-03 15:32:14 Substrate Node
2021-11-03 15:32:14 ✌️  version 3.0.0-monthly-2021-09+1-bf52814-x86_64-macos
2021-11-03 15:32:14 ❤️  by Substrate DevHub <https://github.com/substrate-developer-hub>, 2017-2021
2021-11-03 15:32:14 📋 Chain specification: My Custom Testnet
2021-11-03 15:32:14 🏷 Node name: MyNode01
2021-11-03 15:32:14 👤 Role: AUTHORITY
2021-11-03 15:32:14 💾 Database: RocksDb at /tmp/node01/chains/local_testnet/db
2021-11-03 15:32:14 ⛓  Native runtime: node-template-100 (node-template-1.tx1.au1)
2021-11-03 15:32:15 🔨 Initializing Genesis block/state (state: 0x2bde…8f66, header-hash: 0x6c78…37de)
2021-11-03 15:32:15 👴 Loading GRANDPA authority set from genesis on what appears to be first startup.
2021-11-03 15:32:15 ⏱  Loaded block-time = 6s from block 0x6c78abc724f83285d1487ddcb1f948a2773cb38219c4674f84c727833be737de
2021-11-03 15:32:15 Using default protocol ID "sup" because none is configured in the chain specs
2021-11-03 15:32:15 🏷 Local node identity is: 12D3KooWLmrYDLoNTyTYtRdDyZLWDe1paxzxTw5RgjmHLfzW96SX
2021-11-03 15:32:15 📦 Highest known block at #0
2021-11-03 15:32:15 〽️ Prometheus exporter started at 127.0.0.1:9615
2021-11-03 15:32:15 Listening for new connections on 127.0.0.1:9945.
2021-11-03 15:32:20 💤 Idle (0 peers), best: #0 (0x6c78…37de), finalized #0 (0x6c78…37de), ⬇ 0 ⬆ 0
```

다음 정보를 확인하세요:

- 출력에는 사용자 정의 체인 스펙이 사용되고 있는 것이 나타납니다. 이는 `--chain` 명령줄 옵션을 사용하여 지정한 사용자 정의 체인 스펙입니다.
- 출력에는 `--validator` 명령줄 옵션을 사용하여 이 노드가 권한자임을 나타내고 있습니다.
- 출력에는 블록 해시 `(state: 0x2bde…8f66, header-hash: 0x6c78…37de)`로 제네시스 블록이 초기화되고 있음을 나타냅니다.
- 출력에는 노드의 **로컬 신원**이 표시됩니다.
  이 예제에서 노드 신원은 `12D3KooWLmrYDLoNTyTYtRdDyZLWDe1paxzxTw5RgjmHLfzW96SX`입니다.
- 출력에는 노드에 사용되는 **IP 주소**가 로컬 호스트 `127.0.0.1`임을 나타냅니다.

이러한 값은 이 특정 튜토리얼 예제에 대한 것입니다.
출력의 값은 노드에 특정되며, 다른 네트워크 참여자가 부트노드에 연결할 수 있도록 자신의 노드에 대한 값을 제공해야 합니다.

자체 키를 키스토어에 추가하고 노드를 다시 시작한 후, 블록이 생성되는 것을 확인하기 전에 노드를 중지하세요.

## 키스토어에 키 추가하기

첫 번째 노드를 시작한 후에는 아직 블록이 생성되지 않았습니다.
다음 단계는 네트워크의 각 노드에 대해 키스토어에 두 가지 유형의 키를 추가하는 것입니다.

각 노드에 대해 다음을 수행하세요:

- 블록 생성 (`aura`)을 가능하게 하기 위해 `aura` 권한 키를 추가합니다.

- 블록 최종화 (`grandpa`)를 가능하게 하기 위해 `grandpa` 권한 키를 추가합니다.

키를 키스토어에 추가하는 방법은 여러 가지가 있습니다.
이 튜토리얼에서는 `key` 하위 명령을 사용하여 로컬에서 생성된 비밀 키를 삽입하는 방법을 사용할 수 있습니다.

키를 키스토어에 삽입하려면 다음을 수행하세요:

1. 컴퓨터에서 터미널 쉘을 엽니다.

1. Substrate 노드 템플릿을 컴파일한 루트 디렉토리로 이동합니다.

1. 다음 명령을 실행하여 `aura` 키를 삽입합니다. `key` 하위 명령을 사용하여 생성된 비밀 키를 사용합니다.

   ```bash
   ./target/release/node-template key insert --base-path /tmp/node01 \
      --chain customSpecRaw.json \
      --scheme Sr25519 \
      --suri <your-secret-seed> \
      --password-interactive \
      --key-type aura
   ```

   `<your-secret-seed>`를 첫 번째 키 쌍을 생성할 때 사용한 비밀 문구 또는 비밀 키로 대체하세요.
   이 튜토리얼에서 사용한 비밀 문구는 `pig giraffe ceiling enter weird liar orange decline behind total despair fly`이므로 `--suri` 명령줄 옵션에 해당 문자열을 지정합니다.

   예를 들어:

   ```text
   --suri "pig giraffe ceiling enter weird liar orange decline behind total despair fly"
   ```

   지정된 파일 위치에서 키를 삽입할 수도 있습니다.
   사용 가능한 명령줄 옵션에 대한 정보는 다음 명령을 실행하세요:

   ```bash
   ./target/release/node-template key insert --help
   ```

1. 키를 생성할 때 사용한 비밀번호를 입력합니다.

1. 다음 명령을 실행하여 `grandpa` 키를 삽입합니다. `key` 하위 명령을 사용하여 생성된 비밀 키를 사용합니다.

   ```bash
   ./target/release/node-template key insert \
      --base-path /tmp/node01 \
      --chain customSpecRaw.json \
      --scheme Ed25519 \
      --suri <your-secret-key> \
      --password-interactive \
      --key-type gran
   ```

   `<your-secret-seed>`를 첫 번째 키 쌍을 생성할 때 사용한 비밀 문구 또는 비밀 키로 대체하세요.
   이 튜토리얼에서 사용한 비밀 문구는 `pig giraffe ceiling enter weird liar orange decline behind total despair fly`이므로 `--suri` 명령줄 옵션에 해당 문자열을 지정합니다.

1. 키를 생성할 때 사용한 비밀번호를 입력합니다.

1. 다음 명령을 실행하여 `node01`의 키스토어에 키가 있는지 확인합니다.

   ```bash
   ls /tmp/node01/chains/local_testnet/keystore
   ```

   다음과 유사한 출력이 표시됩니다:

   ```text
   617572617e016f19ab623ba5f487f540017c1edbab06c0b211a16d40531dbd62d94ceb24
   6772616e4ac976937e53fd836512cfd288bb438584ba366cbf9be403a0acd82c1c7c0739
   ```

노드01의 키스토어에 키를 추가한 후, 이전에 사용한 [첫 번째 노드 시작](#start-the-first-node)에서 사용한 명령을 사용하여 노드를 다시 시작하세요.

## 다른 참여자가 참여할 수 있도록 하기

이제 다른 검증자가 `--bootnodes` 및 `--validator` 명령줄 옵션을 사용하여 네트워크에 참여할 수 있도록 할 수 있습니다.

개인 네트워크에 두 번째 검증자를 추가하려면 다음을 수행하세요:

1. 두 번째 컴퓨터에서 터미널 쉘을 엽니다.

1. Substrate 노드 템플릿을 컴파일한 루트 디렉토리로 이동합니다.

1. 다음 명령을 실행하여 두 번째 블록체인 노드를 시작합니다:

   ```bash
   ./target/release/node-template \
      --base-path /tmp/node02 \
      --chain ./customSpecRaw.json \
      --port 30334 \
      --rpc-port 9946 \
      --telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
      --validator \
      --rpc-methods Unsafe \
      --name MyNode02 \
      --bootnodes /ip4/127.0.0.1/tcp/30333/p2p/12D3KooWLmrYDLoNTyTYtRdDyZLWDe1paxzxTw5RgjmHLfzW96SX \
      --password-interactive
   ```

   이 명령은 `base-path`, `name`, `validator` 명령줄 옵션을 사용하여 이 노드를 개인 네트워크의 두 번째 검증자로 식별합니다.
   `--chain` 명령줄 옵션은 사용할 체인 스펙 파일을 지정합니다.
   이 파일은 모든 검증자에서 _동일_해야 합니다.
   `--bootnodes` 명령줄 옵션에 올바른 정보를 설정했는지 확인하세요.
   특히, 첫 번째 네트워크 노드의 로컬 노드 식별자를 지정했는지 확인하세요.
   올바른 `bootnode` 식별자를 설정하지 않으면 다음과 같은 오류가 발생합니다:

   ```text
   The bootnode you want to connect to at ... provided a different peer ID than the one you expect: ...
   ```

1. 다음 명령을 실행하여 `aura` 비밀 키를 키스토어에 추가합니다. `key` 하위 명령을 사용하여 생성된 비밀 키를 사용합니다.

   ```bash
   ./target/release/node-template key insert --base-path /tmp/node02 \
      --chain customSpecRaw.json \
      --scheme Sr25519 \
      --suri <second-participant-secret-seed> \
      --password-interactive \
      --key-type aura
   ```

   `<second-participant-secret-seed>`를 두 번째 키 쌍을 생성할 때 사용한 비밀 문구 또는 비밀 키로 대체하세요.
   `aura` 키 유형은 블록 생성을 가능하게 하기 위해 필요합니다.

1. 키를 생성할 때 사용한 비밀번호를 입력합니다.

1. 다음 명령을 실행하여 `grandpa` 비밀 키를 키스토어에 추가합니다. `key` 하위 명령을 사용하여 생성된 비밀 키를 사용합니다.

   ```bash
   ./target/release/node-template key insert --base-path /tmp/node02 \
      --chain customSpecRaw.json \
      --scheme Ed25519 --suri <second-participant-secret-seed> \
      --password-interactive \
      --key-type gran
   ```

   `<second-participant-secret-seed>`를 두 번째 키 쌍을 생성할 때 사용한 비밀 문구 또는 비밀 키로 대체하세요.
   `gran` 키 유형은 블록 최종화를 가능하게 하기 위해 필요합니다.

   블록 최종화를 위해서는 최소한 검증자의 3분의 2 이상이 각각의 키를 자신의 키스토어에 추가해야 합니다.
   이 네트워크는 체인 스펙에 두 개의 검증자가 구성되어 있으므로, 두 번째 노드가 키를 추가한 후에야 블록 최종화가 시작될 수 있습니다.

1. 키를 생성할 때 사용한 비밀번호를 입력합니다.

1. 다음 명령을 실행하여 `node02`의 키스토어에 키