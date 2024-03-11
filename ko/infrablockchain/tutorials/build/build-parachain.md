---
title: 파라체인 구축하기
description: 이 튜토리얼은 파라체인을 구축하는 방법을 알아봅니다.
keywords:
  - 파라체인
  - 인프라릴레이체인
---

이 튜토리얼은 **_인프라릴레이체인(InfraRelayChain)_** 에 파라체인을 연결하는 방법을 설명합니다.

## 튜토리얼 목표

이 튜토리얼을 완료함으로써 다음 목표를 달성할 수 있습니다: 

- 릴레이체인과 파라체인의 특징을 알 수 있습니다.

- 서로 다른 두 체인(릴레이체인과 파라체인)을 연결하여 멀티체인 아키텍처를 구축할 수 있습니다.

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [로컬 릴레이체인 준비](./build-infra-relay-chain.md) 튜토리얼을 통해 로컬 릴레이체인을 구성 방법에 대해 확인합니다.

- [커스텀 체인 스펙](../../learn/substrate/build/chain-spec.md)을 구성하는 방법에 대해 확인합니다. 

## 파라체인 템플릿 빌드하기

이 튜토리얼에서는 _파라체인 템플릿_ 을 이용하여 **_인프라릴레이체인(InfraRelayChain)_** 에 연결하는 방법에 대해 설명합니다.
파라체인 템플릿은 파라체인 개발의 시작점으로도 사용할 수 있습니다.

파라체인 템플릿을 빌드하려면 다음을 수행하세요:

1. 최신 **_인프라블록체인(InfraBlockchain)_** SDK를 불러옵니다.

   ```bash
   git clone --branch master https://github.com/InfraBlockchain/infrablockspace-sdk.git
   ```

2. 다음 명령을 실행하여 파라체인 템플릿 런타임을 빌드합니다. 

   ```bash
   cargo build --release --bin parachain-template-node
   ```

## 파라체인 식별자(ParaId) 예약하기

릴레이체인에 연결된 파라체인은 각각의 고유 식별자인 `ParaID`를 예약해야 합니다. 이 식별자를 통해 릴레이체인은 파라체인을 식별하고 연결시킬 수 있습니다.
이 식별자는 [파라체인](https://wiki.polkadot.network/docs/learn-parachains)이나 [파라스레드](https://wiki.polkadot.network/docs/learn-parathreads)가 차지한 슬롯을 식별하는 데 사용될 수 있기 때문에 `ParaID`라고 합니다.

파라체인 식별자를 예약하려면 다음을 수행하세요:

1. 최소 두 개이상의 밸리데이터 노드가 구동 중인 로컬 릴레이체인 네트워크를 구성합니다. 

2. 브라우저에서 [인프라블록체인 익스플로러](https://portal.infrablockspace.net/#/explorer)를 엽니다. 
3. 익스플로러 탭에서 왼쪽 상단 탭을 클릭하여 `DEVELOPMENT` 섹션을 눌러 노드를 시작할 때 설정한 RPC 포트를 이용해 로컬 릴레이체인에 연결합니다.

   ![로컬 네트워크 연결](/media/images/docs/infrablockchain/tutorials/build/connect-network.png)

4. **네트워크**를 클릭하고 **파라체인**을 선택합니다.

   ![파라체인으로 이동](/media/images/docs/infrablockchain/tutorials/build/network-parachains.png)

5. **파라스레드**를 클릭한 다음 **ParaId**를 클릭합니다.

   ![파라체인 식별자 예약](/media/images/docs/infrablockchain/tutorials/build/paraid-reserve.png)

6. 파라체인 식별자를 예약하는 트랜잭션 구성을 확인한 후 **제출**을 클릭합니다.

   - 파라체인 식별자를 예약하는 데 사용한 계정은 트랜잭션에 대한 요금이 청구되는 계정이며, 해당 식별자와 연관된 계정이 됩니다.  

7. 트랜잭션을 승인하기 위해 **서명 및 제출**을 클릭합니다.

8. 트랜잭션을 제출한 후 **네트워크**를 클릭하고 **탐색기**를 선택합니다.

9. `registrar.Reserved` 이벤트 목록을 확인하고 이벤트를 클릭하여 트랜잭션에 대한 세부 정보를 확인합니다.

이제 파라체인이 릴레이체인에 연결될 수 있도록 _체인 스펙_ 을 준비하고 필요한 파일을 생성할 준비가 되었습니다. 이때 예약한 파라체인 식별자(`paraId` `2000`)를 사용하여 파라체인이 릴레이체인에 연결되도록 해야 합니다.

## 파라체인 등록 절차

### 평문 체인 스펙 수정하기

로컬 릴레이체인에 파라체인을 등록하려면 평문 체인 스펙을 수정하여 예약한 파라체인 식별자를 사용해야 합니다.

파라체인 템플릿 노드의 평문 체인 스펙을 생성하려면 다음 명령을 실행하세요:

   ```bash
   ./target/release/parachain-template-node build-spec --disable-default-bootnode > plain-para-template-chainspec.json
   ```

- 생성된 평문 체인 스펙을 텍스트 편집기에서 엽니다.

- 이전에 예약한 파라체인 식별자로 `para_id`를 설정합니다.

- 예를 들어, 예약한 식별자가 `2000`인 경우 `para_id` 필드를 `2000`으로 설정합니다:

   ```json
   ...
   "relay_chain": "infra-relay-local",
   "para_id": 2000,
   "codeSubstitutes": {},
   "genesis": {
      ...
   }
   ```

- `parachainId` 를 예약한 파라체인 식별자로 설정합니다.

- 예를 들어, 예약한 식별자가 `2000`인 경우 `para_id` 필드를 `2000`으로 설정합니다:

   ```json
   ...
      "parachainSystem": null,
      "parachainInfo": {
        "parachainId": 2000
      },
   ...
   ```

- 동일한 로컬 네트워크에서 누군가와 동시에 이 튜토리얼을 완료하는 경우, 해당 노드와 피어링되지 않도록 평문 체인 스펙을 수정해야 합니다. 평문 체인 스펙 JSON 파일에서 `protocolId` 키를 찾고 어떤 고유한 값으로 변경해줍니다.

   ```json
      "protocolId": "infrablockchain"
   ```

- 변경 사항을 저장하고 평문 체인 스펙 파일을 닫습니다.

수정된 체인 스펙 파일에서 원시(raw) 체인 스펙 파일을 생성하려면 다음 명령을 실행하세요:

   ```bash
   ./target/release/parachain-template-node build-spec --chain plain-para-template-chainspec.json --disable-default-bootnode --raw > raw-para-template-chainspec.json
   ```

이 명령은 두 개의 콜레이터가 있는 새로운 원시(raw) 체인 스펙 파일을 생성합니다.

   ```text
   2023-12-01 13:00:50 Building chain spec
   2023-12-01 13:00:50 assembling new collators for new session 0 at #0
   2023-12-01 13:00:50 assembling new collators for new session 1 at #0
   ```

### 파라체인 웹어셈블리 런타임 및 제네시스 상태 추출하기

등록할 파라체인 콜레이터를 준비하려면 다음을 수행하세요:

1. 파라체인을 위한 웹어셈블리 런타임을 내보냅니다.

   릴레이체인은 파라체인 블록의 유효성을 검사하기 위해 파라체인 등록시 각 파라체인의 웹어셈블리 런타임을 요구합니다. 파라체인에 대한 웹어셈블리 런타임을 내보내려면 다음과 유사한 명령을 실행합니다:

   ```bash
   ./target/release/parachain-template-node export-genesis-wasm --chain raw-para-template-chainspec.json para-2000-wasm
   ```

2. 파라체인 제네시스 상태를 생성합니다.

   파라체인을 등록하려면 릴레이체인이 파라체인의 제네시스 상태를 알아야 합니다.
   전체 제네시스 상태를 16진수로 인코딩하여 파일로 내보내려면 다음과 유사한 명령을 실행합니다:

   ```bash
   ./target/release/parachain-template-node export-genesis-state --chain raw-para-template-chainspec.json para-2000-genesis-state
   ```

   내보낸 런타임 및 상태는 _제네시스_ 블록을 위한 것이어야 합니다. 이전 상태를 가진 파라체인을 릴레이체인에 연결할 수 없습니다. **모든 파라체인은 블록 높이 0 부터 시작해야 합니다.** 

## 콜레이터 노드 시작하기

콜레이터 노드는 파라체인에 특화된 특수한 노드로써 다음과 같은 역할을 수행합니다:

- 릴레이체인의 최신 상태 가져오기
- 파라체인 블록을 생성하여 릴레이체인에 전달하기

더 자세한 내용은 [다음](https://wiki.polkadot.network/docs/learn-parachains-protocol#collators) 을 참고하시기 바랍니다.

3. 다음과 같이 명령어를 입력하여 콜레이터 노드를 시작합니다:

   ```bash
   ./target/release/parachain-template-node \
   --alice \
   --collator \
   --force-authoring \
   --chain raw-para-template-chainspec.json \
   --base-path /tmp/parachain/alice \
   --port 40333 \
   --rpc-port 8844 \
   -- \
   --execution wasm \
   --chain ../infrablockspace/raw-infra-relay-chainspec.json \
   --port 30343 \
   --rpc-port 9977
   ```

- 이 명령에서 `--` 인자 이전에 전달된 인수는 콜레이터를 위한 것입니다.
- `--` 이후의 내장된 릴레이체인 노드를 위한 것입니다.
- `--collator` 옵션을 통해 해당 노드가 콜레이터 노드임을 명시합니다.
- 이 명령에서는 파라체인의 원시(raw) 체인 스펙과 릴레이체인의 원시(raw) 체인 스펙을 모두 지정합니다.
- 이 예제에서는 `infrablockspace` 디렉토리에 있는 `raw-infra-relay-chainspec.json`이라는 로컬 릴레이체인의 원시(raw) 체인 스펙을 사용합니다.
- 두 번째 `--chain` 명령줄이 로컬 릴레이체인의 원시(raw) 체인 스펙 파일 경로를 지정하도록 확인하세요.
- 다른 파라체인 노드를 시작하는 경우 동일한 릴레이체인 스펙 파일을 사용하지만 다른 기본 경로와 포트 번호를 사용해야 합니다.
- 콜레이터 노드를 시작한 터미널에서 다음과 유사한 출력을 볼 수 있어야 합니다:

   ```text
   Parachain Collator Template
   ✌️  version 0.1.0-336530d3bdd
   ❤️  by Anonymous, 2020-2023
   📋 Chain specification: Local Testnet
   🏷  Node name: Alice
   👤 Role: AUTHORITY
   💾 Database: RocksDb at /tmp/parachain/alice/chains/local_testnet/db/full
   no effect anymore and will be removed in the future!
   Parachain Account: 5Ec4AhPUwPeyTFyuhGuBbD224mY85LKLMSqSSo33JYWCazU4
   Is collating: yes
   [Relaychain] 🏷  Local node identity is: 12D3KooWR8wJbGWrjzKTpuXQvbuM1rE2GAE9JVEFEwAyNX6LV9nN
   [Relaychain] 💻 Operating system: ...
   ......
   [Relaychain] 📦 Highest known block at #95
   ...
   [Parachain] 🏷  Local node identity is: 12D3KooWF464pkLaHbsfc4DzDkYuhdVE4zSqBHR1gPapLvwsfZtg
   ...
   ```

- 템플릿 콜레이터 노드가 독립적인 노드로 실행되고 콜레이터 노드에 내장된 릴레이체인 노드가 로컬 릴레이체인 검증자 노드와 피어로 연결되는 것을 볼 수 있어야 합니다.(내장된 릴레이체인이 로컬 릴레이체인 노드와 피어링되지 않는 경우 방화벽을 비활성화하거나 릴레이 노드의 주소를 `bootnodes` 매개변수에 추가해 보세요.)

## 로컬 릴레이체인에 등록하기

로컬 릴레이체인과 콜레이터 노드가 실행 중인 상태에서 로컬 릴레이체인에 파라체인을 등록할 준비가 되었습니다. 실제 퍼블릭 메인 네트워크에서 파라체인 등록은 일반적으로 거버넌스 과정을 통해 이루어지지만 본 튜토리얼에서는 Sudo 트랜잭션을 이용해 진행해 나갈 예정입니다. 

### 파라체인 등록 과정:

1. 최소 두개 이상의 밸리데이터 노드로 구성된 로컬 릴레이체인이 실행 중인지 확인합니다.
2. 브라우저에서 [인프라블록체인 익스플로러](https://portal.infrablockspace.net/#/explorer/)를 엽니다. 위의 파라체인 식별자를 예약할 때 처럼, RPC 및 웹소켓 포트를 이용해 릴레이체인에 연결합니다. 
3. **개발자**를 클릭하고 **Sudo**를 선택합니다.

4. _paraSudoWrapper_ 를 선택한 다음 _sudoScheduleParaInitialize(id, genesis)_ 를 선택합니다.

- 트랜잭션 매개변수 설정:

   - `id`: 예약한 `ParaId`를 입력합니다.본 튜토리얼에서는 예약한 식별자가 2000인 경우입니다.
   - `genesisHead`: **파일 업로드**를 클릭하고 파라체인을 위해 내보낸 제네시스 상태를 업로드합니다. 본 튜토리얼에서는 `para-2000-genesis` 파일을 선택합니다.
   - `validationCode`: **파일 업로드**를 클릭하고 파라체인을 위해 내보낸 웹어셈블리 런타임을 업로드합니다. 본 튜토리얼에서는 `para-2000-wasm` 파일을 선택합니다.
   - `paraKind`: **예**를 선택합니다.

   ![파라체인 등록을 위해 Sudo 선택](/media/images/docs/infrablockchain/tutorials/build/register-para-with-sudo.png)

5. **Sudo 제출**을 클릭합니다.
6. 트랜잭션 세부 정보를 검토한 다음 **서명 및 제출**을 클릭하여 트랜잭션을 승인합니다.

   트랜잭션을 제출한 후 **네트워크**를 클릭하고 **탐색기**를 선택합니다.

7. 최근 이벤트 목록에서 `sudo.Sudid` 및 `paras.PvfCheckAccepted` 이벤트를 확인하고 이벤트를 클릭하여 트랜잭션에 대한 세부 정보를 확인합니다.

   ![Sudo 등록 이벤트 보기](/media/images/docs/sudo-event.jpg)

   ![PVF Check Accepted](/media/images/docs/infrablockchain/tutorials/build/pvf-check-accepted.png)

8. 파라체인 등록이 완료되면 익스플로러에서 확인할 수 있습니다. **네트워크**를 클릭하고 **파라체인**을 선택하세요.

9. 새로운 epoch 가 시작될 때까지 기다립니다.

   - 릴레이체인은 각 파라체인의 최신 블록(헤드)을 추적합니다.
   - 릴레이체인 블록이 확정되면 [검증 과정](https://polkadot.network/the-path-of-a-parachain-block)을 완료한 파라체인 블록도 확정됩니다.
   - 이것이 멀티체인 아키텍처인 **_인프라블록체인(InfraBlockchain)_** **공유된 보안(shared security)** 를 달성하는 방법입니다.
   - 다음 epoch 에서 파라체인이 릴레이체인에 연결된 후 콜레이터 노드를 시작할 때 설정한 포트로 이동하면 익스플로러에서 해당 파라체인에 대한 정보를 볼 수 있습니다. 

## 파라체인 트랜잭션 제출하기

이제 파라체인이 실행되고 릴레이체인에 연결되었으므로 익스플로러를 사용하여 파라체인에 트랜잭션을 제출할 수 있습니다.

트랜잭션을 제출하려면 다음을 수행하세요:

1. 브라우저에서 [인프라블록체인 익스플로러](https://portal.infrablockspace.net/#/explorer/)를 엽니다.

2. 응용 프로그램의 왼쪽 상단에 있는 네트워크 선택기를 클릭합니다.

3. 사용자 정의 엔드포인트를 파라체인의 WebSocket 포트에 연결하도록 변경합니다. 이 튜토리얼의 설정을 따르면 포트 `8844`에 연결합니다.

4. **계정**을 클릭하고 **전송**을 선택하여 Alice에서 다른 계정으로 자금을 보냅니다.

   - 자금을 보낼 계정을 선택합니다.
   - 금액을 입력합니다.
   - **전송 생성**을 클릭합니다.
   - 트랜잭션을 검토한 다음 **서명 및 제출**을 클릭하여 전송을 승인합니다.

5. **계정**을 클릭하여 전송이 완료되고 파라체인 트랜잭션이 성공적으로 수행되었는지 확인합니다. 

## 블록체인 상태 재설정하기

테스트 및 개발 목적의 네트워크에서는 노드를 시작할 때 마다 블록체인 상태가 초기화되고 처음부터 다시 시작됩니다. 그러나 체인 상태를 삭제하거나 데이터베이스를 수동으로 삭제하는 경우 데이터를 복구하거나 체인 상태를 복원할 수 없으므로 주의해야 합니다. 보존하려는 데이터가 있는 경우 파라체인 상태를 초기화하기 전에 데이터의 사본을 보관하도록 해야 합니다.

본 튜토리얼에서는 테스트 및 개발 목적이 아닌 실제 네트워크를 시뮬레이션 하는 로컬 환경의 네트워크입니다. 따라서 새롭게 다시 시작하려면 로컬 릴레이체인 노드와 파라체인의 체인 상태를 완전히 제거해야 합니다.

블록체인 상태를 재설정하려면 다음을 수행하세요:

1. 파라체인 템플릿 노드가 실행 중인 터미널에서 Control-c를 누릅니다.
2. 다음 명령을 실행하여 파라체인 콜레이터 상태를 제거합니다:

   ```bash
   rm -rf /tmp/parachain
   ```

3. `alice` 검증자 노드 또는 `bob` 검증자 노드가 실행 중인 터미널에서 Control-c를 누릅니다.

4. 다음 명령을 실행하여 검증자 상태를 제거합니다:

   ```bash
   rm -rf /tmp/relay
   ```