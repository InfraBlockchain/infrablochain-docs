---
title: 파라체인 시뮬레이션
description: 검증자와 파라체인 콜레이터 노드를 포함한 릴레이체인을 시뮬레이션하기 위해 로컬 테스트 네트워크를 설정하는 방법을 설명합니다.
keywords:
  - 파라체인
  - 테스트넷
  - 콜레이터 노드
  - 밸리데이터 노드
  - 릴레이체인
  - 좀비넷
  - 임시 포트
  - XCM
  - HRMP
---

`zombienet` 명령줄 도구를 사용하여 검증자와 파라체인 콜레이터 노드를 포함한 릴레이체인을 시뮬레이션하는 로컬 테스트 네트워크를 설정할 수 있습니다.
테스트 네트워크를 구성하여 여러 검증자와 다수의 파라체인을 포함하도록 설정할 수 있습니다.

이 튜토리얼에서는 다음과 같은 구성으로 기본 테스트 네트워크를 설정하는 방법을 설명합니다:

- 네 개의 밸리데이터
- 두 개의 파라체인
- 파라체인 당 하나의 콜레이터
- 파라체인 간 메시지 교환을 가능하게 하는 메시지 전달 채널

## 바이너리 파일이 있는 작업 폴더 준비하기

`zombienet` 명령줄 인터페이스는 테스트 네트워크의 특성을 지정하는 구성 파일을 사용합니다. 이 구성 파일에는 사용할 바이너리 파일, 도커 이미지 또는 쿠버네티스 배포의 이름과 위치를 지정하는 정보가 포함됩니다.

이 튜토리얼에서는 네이티브 릴레이체인과 콜레이터 바이너리 파일을 사용하는 테스트 네트워크를 구성하는 방법을 설명하므로, 테스트 네트워크를 설정하기 위해 필요한 바이너리 파일이 있는 작업 폴더를 준비하는 것이 첫 번째 단계입니다.

테스트 네트워크를 위한 바이너리 파일이 있는 작업 폴더를 준비하려면 다음 단계를 수행하세요.

1. 새로운 터미널 쉘을 엽니다.

2. 홈 디렉토리로 이동하고 테스트 네트워크를 생성하는 데 필요한 바이너리 파일을 보관할 새 폴더를 만듭니다.

   예를 들어:

   ```bash
   mkdir binaries
   ```

3. 다음 명령을 실행하여 _InfraBlockSpace_ 저장소를 복제합니다.

   ```bash
   git clone https://github.com/InfraBlockchain/infrablockchain-substrate
   ```

4. 다음 명령을 실행하여 `infrablockchain-substrate` 디렉토리의 루트로 이동합니다.

   ```bash
   cd infrablockchain-substrate
   ```

5. 최신 infrablockchain-substrate 릴리스를 확인합니다.

   릴리스 브랜치는 `release-v<n.n.n>` 형식을 따릅니다.
   예를 들어, 이 튜토리얼에서 사용하는 릴리스 브랜치는 `release-v1.0.0`입니다.
   `release-v1.0.0` 대신 더 최신의 릴리스 브랜치를 사용할 수 있습니다.

   예를 들어:

   ```bash
   git checkout release-v1.0.0
   ```

6. 다음 명령을 실행하여 릴레이체인 노드를 컴파일합니다.

   ```bash
   cargo build --release
   ```

7. 다음 명령을 실행하여 _InfraBlockSpace_ 바이너리 파일을 작업 `binaries` 폴더로 복사합니다.

   ```bash
   cp ./target/release/infrablockspace ../binaries/infrablockspace-v1.0.0
   ```

   이 예시에서는 일반적으로 `binaries` 폴더의 파일을 정리하기 위해 바이너리 파일 이름에 `infrablockspace` 버전을 추가하는 것이 좋은 관행입니다.

8. 홈 디렉토리로 이동합니다.

### 파라체인 바이너리 파일 추가하기

작업 폴더에는 릴레이체인의 바이너리 파일이 있지만, 파라체인 콜레이터 노드의 바이너리 파일도 필요합니다.

본 테스트에서는 `parachain-template-node` 바이너리 파일을 빌드할 예정입니다. 

작업 폴더에 파라체인 콜레이터 바이너리 파일을 추가하려면 다음 단계를 수행하세요.

1. `infrablockchain-substrate` 의 루트 디렉토리로 이동한 후 다음과 같은 명령어를 입력합니다.

   ```bash
   cargo build --release -bin parachain-template-node-
   ```
   이제 paraId 1000에 대한 파라체인 콜레이터 바이너리 파일이 준비되었습니다.
   
2. 다음 명령을 실행하여 파라체인 바이너리 파일을 작업 `bin` 폴더로 복사합니다:

   ```bash
   cp ./target/release/parachain-template-node ../bin/parachain-template-node-v1.0.0-1000
   ```

   이 예시에서는 일반적으로 `bin` 폴더의 파일을 정리하기 위해 바이너리 파일 이름에 버전과 `paraId`를 추가하는 것이 좋은 관행입니다.

## 테스트 네트워크 설정 구성하기

이제 작업 폴더에 필요한 바이너리 파일이 준비되었으므로 좀비넷이 사용할 테스트 네트워크의 설정을 구성할 준비가 되었습니다.

좀비넷을 다운로드하고 구성하는 방법:

1. Linux 또는 macOS 운영 체제용 적절한 [좀비넷 실행 파일](https://github.com/paritytech/zombienet/releases)을 다운로드합니다.

   보안 설정에 따라 실행 파일에 대한 액세스를 명시적으로 허용해야 할 수 있습니다.

   실행 파일을 시스템 전역에서 사용할 수 있도록 하려면 다음과 같은 명령을 다운로드한 실행 파일 이후에 실행하세요.

   ```bash
   chmod +x zombienet-macos
   cp zombienet-macos /usr/local/bin
   ```

2. 다음 명령을 실행하여 좀비넷이 올바르게 설치되었는지 확인합니다.

   ```bash
   ./zombienet-macos --help
   ```

   명령줄 도움말이 표시되면 좀비넷을 구성할 준비가 된 것입니다.

3. 다음 명령을 실행하여 좀비넷을 위한 구성 파일을 생성합니다.

   ```bash
   touch config.toml
   ```

   구성 파일을 사용하여 다음 정보를 지정합니다.

   - 테스트 네트워크의 바이너리 파일 위치
   - 사용할 릴레이체인 스펙(`infra-relay-local`)
   - 네 개의 릴레이체인 검증자에 대한 정보
   - 테스트 네트워크에 포함된 파라체인 식별자
   - 각 파라체인의 콜레이터에 대한 정보
   - 각 노드에 연결하기 위해 사용할 WebSocket 엔드포인트 포트

   예를 들어:

   ```toml
   [relaychain]

   default_command = "../binaries/infrablockspace-v.1.0.0"
   default_args = ["-lparachain=debug", "-l=xcm=trace"]

   chain = "infra-relay-local"

   [[relaychain.nodes]]
   name = "alice"
   validator = true
   args = ["-lparachain=debug", "-l=xcm=trace"]

   rpc_port = 7100
   ws_port = 7101

   [[relaychain.nodes]]
   name = "bob"
   validator = true
   args = ["-lparachain=debug", "-l=xcm=trace"]
   rpc_port = 7200
   ws_port = 7201

   [[relaychain.nodes]]
   name = "charlie"
   validator = true
   args = ["-lparachain=debug", "-l=xcm=trace"]
   rpc_port = 7300
   ws_port = 7301

   [[relaychain.nodes]]
   name = "dave"
   validator = true
   args = ["-lparachain=debug", "-l=xcm=trace"]
   rpc_port = 7400
   ws_port = 7401

   [[parachains]]
   id = 1000
   cumulus_based = true

      [parachains.collator]
      name = "parachain-A-1000-collator01"
      command = "../binaries/parachain-template-node-v1.0.0"
      rpc_port = 9900
      ws_port = 9901

   [[parachains]]
   id = 1001
   cumulus_based = true

      [parachains.collator]
      name = "parachain-B-1001-collator01"
      command = "../binaries/parachain-template-node-v1.0.0"
      rpc_port = 10000
      ws_port = 10001
   ```

4. 변경 사항을 저장하고 파일을 닫습니다.

5. 다음 명령을 실행하여 이 구성 파일을 사용하여 테스트 네트워크를 시작합니다.

   ```bash
   ./zombienet-macos spawn config.toml -p native
   ```

   명령은 시작된 테스트 네트워크 노드에 대한 정보를 표시합니다.

   릴레이체인 및 파라체인 노드 엔드포인트에 대한 직접 링크는 다음과 유사한 형식이어야 합니다.

   - alice: https://portal.infrablockspace.net/?rpc=ws://127.0.0.1:7101#/explorer
   - bob: https://portal.infrablockspace.net/?rpc=ws://127.0.0.1:7201#/explorer
   - charlie: https://portal.infrablockspace.net/?rpc=ws://127.0.0.1:7301#/explorer
   - dave: https://portal.infrablockspace.net/?rpc=ws://127.0.0.1:7401#/explorer

   파라체인 콜레이터 엔드포인트에 대한 직접 링크는 다음과 유사한 형식이어야 합니다.

   - parachain-1000-collator: https://portal.infrablockspace.net/?rpc=ws://127.0.0.1:9901#/explorer
   - parachain-1001-collator: https://portal.infrablockspace.net/?rpc=ws://127.0.0.1:10001#/explorer

   모든 노드가 실행된 후에는 [*인프라블록체인(InfraBlockchain)* 익스플로러](https://portal.infrablockspace.net)을 열고 노드 엔드포인트에 연결하여 노드와 상호 작용할 수 있습니다.

## 메시지 전달 채널 열기

테스트 네트워크가 준비되었으므로 파라체인 A(1000)와 파라체인 B(1001) 사이의 수평 릴레이 메시지 전달 채널을 열어 파라체인 간 통신을 가능하게 할 수 있습니다.
채널은 단방향이므로 다음을 수행해야 합니다.

- 파라체인 A(1000)에서 파라체인 B(1001)로 채널 열기 요청을 보냅니다.
- 파라체인 B(1001)에서 요청을 수락합니다.
- 파라체인 B(1001)에서 파라체인 A(1000)로 채널 열기 요청을 보냅니다.
- 파라체인 A(1000)에서 요청을 수락합니다.

좀비넷은 테스트 목적으로 구성 파일에 기본 채널 설정을 포함하여 이러한 채널을 열기를 간소화합니다.

테스트 네트워크에서 파라체인 간 통신을 설정하는 방법:

1. 텍스트 편집기에서 `config.toml` 파일을 엽니다.

2. 다음과 유사한 채널 정보를 구성 파일에 추가합니다.

   ```toml
   [[hrmp_channels]]
   sender = 1000
   recipient = 1001
   max_capacity = 8
   max_message_size = 8000

   [[hrmp_channels]]
   sender = 1001
   recipient = 1000
   max_capacity = 8
   max_message_size = 8000
   ```

   **max_capacity** 및 **max_message_size**에 설정하는 값은 릴레이체인의 `hrmpChannelMaxCapacity` 및 `hrmpChannelMaxMessageSize` 매개변수에 정의된 값보다 크지 않아야 합니다.

   [인프라블록체인(InfraBlockchain) 익스플로러](https://portal.infrablockspace.net)에서 현재 릴레이체인의 구성 설정을 확인하려면 다음 단계를 수행하세요.

   - **개발자**를 클릭하고 **Chain State**를 선택합니다.
   - **configuration**을 선택한 다음 **activeConfig()**를 선택합니다.
   - 다음 매개변수 값을 확인합니다.

     ```text
     hrmpChannelMaxCapacity: 8
     hrmpChannelMaxTotalSize: 8,192
     hrmlChannelMaxMessageSize: 1,048,576
     ```

3. 변경 사항을 저장하고 파일을 닫습니다.

4. 다음 명령을 실행하여 좀비넷을 다시 시작합니다.

   ```bash
   ./zombienet-macos spawn config.toml -p native
   ```

   이제 파라체인 A(1000)와 파라체인 B(1001) 사이에 양방향 HRMP 채널이 열린 테스트 네트워크가 준비되었습니다.

   [*인프라블록체인(InfraBlockchain)* 익스플로러](https://portal.infrablockspace.net)을 사용하여 파라체인에 연결하고 메시지를 보낼 수 있습니다.

5. **개발자**를 클릭하고 **Extrinsics**를 선택합니다.

6. **ibsXcm** 또는 **xcmPallet**을 선택한 다음 **send(dest, message)**를 선택하여 보낼 XCM 메시지를 작성합니다.

   XCM 메시지는 다른 트랜잭션과 마찬가지로 메시지를 보내는 사람이 작업을 실행하기 위해 지불해야 합니다.
   필요한 모든 정보는 메시지 자체에 포함되어야 합니다.
   HRMP 채널을 열었으니 XCM을 사용하여 메시지를 작성하는 방법에 대한 정보는 [Cross-consensus communication](../../learn/xcm/xcm.md) 및 [Transfer assets with XCM](../build/transfer-assets-with-xcm.md)을 참조하세요.

## 다음 단계로 넘어가기

좀비넷을 사용하는 더 복잡한 사전 구성된 환경인 [Trappist playground](https://github.com/paritytech/trappist)을 다운로드하고 탐색해 보세요.
구성 파일에서 설정할 수 있는 속성에 대한 자세한 정보는 [Network definition specification](https://paritytech.github.io/zombienet/network-definition-spec.html)을 참조하세요.