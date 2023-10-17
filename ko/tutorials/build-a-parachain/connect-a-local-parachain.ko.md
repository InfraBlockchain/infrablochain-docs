---
title: 로컬 파라체인 연결하기
description: 로컬 릴레이 체인에 연결하여 파라체인을 배포하는 방법을 보여줍니다.
keywords:
  - 파라체인
  - 테스트넷
  - 로컬
  - 콜레이터
  - 체인 사양
---

이 튜토리얼은 로컬 릴레이 체인에 파라체인 식별자를 예약하고 로컬 파라체인을 해당 릴레이 체인에 연결하는 방법을 설명합니다.

## 튜토리얼 목표

이 튜토리얼을 완료함으로써 다음 목표를 달성할 수 있습니다:

- 로컬 파라체인 노드를 컴파일합니다.
- 파라체인이 사용할 로컬 릴레이 체인에서 고유 식별자를 예약합니다.
- 파라체인을 위한 체인 사양을 구성합니다.
- 파라체인을 위한 런타임 및 제네시스 상태를 내보냅니다.
- 로컬 파라체인을 시작하고 로컬 릴레이 체인에 연결되는지 확인합니다.

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [로컬 릴레이 체인 준비](/tutorials/build-a-parachain/prepare-a-local-relay-chain/)에서 설명한 대로 두 개의 검증자로 구성된 로컬 릴레이 체인을 구성했는지 확인하세요.

- 파라체인 버전 및 종속성은 연결할 릴레이 체인의 버전과 꽤 밀접하게 관련되어 있으며, 사용한 소프트웨어 버전을 알고 있어야 합니다.

  튜토리얼은 일반적으로 최신 Polkadot 브랜치를 사용하여 기능을 보여줍니다.
  튜토리얼이 예상대로 작동하지 않는 경우 로컬 환경에 최신 Polkadot 브랜치가 있는지 확인하고 필요한 경우 로컬 소프트웨어를 업데이트해야 합니다.

## 파라체인 템플릿 빌드하기

이 튜토리얼에서는 로컬 릴레이 체인에 연결되는 파라체인을 시작하는 방법을 설명하기 위해 [Substrate 파라체인 템플릿](https://github.com/substrate-developer-hub/substrate-parachain-template)을 사용합니다.
파라체인 템플릿은 솔로 체인 개발에 사용되는 [노드 템플릿](https://github.com/substrate-developer-hub/substrate-node-template)과 유사합니다.
파라체인 템플릿은 사용자 정의 파라체인 프로젝트 개발의 시작점으로도 사용할 수 있습니다.

파라체인 템플릿을 빌드하려면 다음을 수행하세요:

1. 필요한 경우 컴퓨터에서 새로운 터미널 셸을 엽니다.

2. 로컬 릴레이 체인을 구성하는 데 사용한 릴리즈 브랜치와 일치하는 `substrate-parachain-template` 저장소의 브랜치를 복제합니다.

   예를 들어, 로컬 릴레이 체인을 구성하는 데 `release-v1.0.0` Polkadot 릴리즈 브랜치를 사용한 경우 파라체인 템플릿에는 `polkadot-v1.0.0` 브랜치를 사용합니다.

   ```bash
   git clone --depth 1 --branch polkadot-v1.0.0 https://github.com/substrate-developer-hub/substrate-parachain-template.git
   ```

3. 다음 명령을 실행하여 파라체인 템플릿 디렉토리의 루트로 이동합니다.

   ```bash
   cd substrate-parachain-template
   ```

   이제 분리된 브랜치가 있습니다.
   변경 사항을 저장하고 이 브랜치를 쉽게 식별할 수 있도록 하려면 다음과 유사한 명령을 실행하여 새 브랜치를 생성할 수 있습니다.

   ```bash
   git switch -c my-branch-v1.0.0
   ```

4. 다음 명령을 실행하여 파라체인 템플릿 콜레이터를 빌드합니다.

   ```bash
   cargo build --release
   ```

   노드 컴파일에는 하드웨어 및 소프트웨어 구성에 따라 최대 60분이 소요될 수 있습니다.

## 고유 식별자 예약하기

각 파라체인은 고유 식별자인 `ParaID`를 예약해야 합니다. 이 식별자를 통해 파라체인이 특정 릴레이 체인에 연결될 수 있습니다.
각 릴레이 체인은 자체적으로 연결된 파라체인의 고유 식별자 집합을 관리합니다.
이 식별자는 [파라체인](https://wiki.polkadot.network/docs/learn-parachains)이나 [파라스레드](https://wiki.polkadot.network/docs/learn-parathreads)가 차지한 슬롯을 식별하는 데 사용될 수 있기 때문에 `ParaID`라고 합니다.

파라체인 식별자를 예약하려면 다음을 수행하세요:

1. 로컬 릴레이 체인 검증자가 실행 중인지 확인합니다.
2. 브라우저에서 [Polkadot/Substrate Portal](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/parachains/parathreads)을 엽니다.
3. 로컬 릴레이 체인 노드에 연결합니다.
4. **네트워크**를 클릭하고 **파라체인**을 선택합니다.

   ![파라체인으로 이동](/media/images/docs/tutorials/parachains/network-parachains.png)

5. **파라스레드**를 클릭한 다음 **ParaId**를 클릭합니다.

   ![식별자 예약](/media/images/docs/tutorials/parachains/paraid-reserve.png)

6. 식별자를 예약하는 트랜잭션의 설정을 검토한 다음 **제출**을 클릭합니다.

   식별자를 예약하는 데 사용한 계정은 트랜잭션에 대한 요금이 청구되는 계정이며, 식별자와 관련된 파라스레드의 원본 계정이 됩니다.

7. 트랜잭션을 승인하기 위해 **서명 및 제출**을 클릭합니다.

   트랜잭션을 제출한 후 **네트워크**를 클릭하고 **탐색기**를 선택합니다.

8. 성공적인 `registrar.Reserved` 이벤트 목록을 확인하고 이벤트를 클릭하여 트랜잭션에 대한 세부 정보를 확인합니다.

이제 파라체인이 릴레이 체인에 연결할 수 있도록 체인 사양을 준비하고 필요한 파일을 생성할 준비가 되었습니다. 이때 예약한 식별자(`paraId` `2000`)를 사용하여 파라체인이 릴레이 체인에 연결되도록 해야 합니다.

## 기본 체인 사양 수정하기

로컬 릴레이 체인에 파라체인을 등록하려면 기본 체인 사양을 수정하여 예약한 파라체인 식별자를 사용해야 합니다.

파라체인 템플릿 노드의 평문 체인 사양을 생성하려면 다음 명령을 실행하세요:

   ```bash
   ./target/release/parachain-template-node build-spec --disable-default-bootnode > plain-parachain-chainspec.json
   ```

평문 체인 사양을 텍스트 편집기에서 엽니다.

이전에 예약한 파라체인 식별자로 `para_id`를 설정합니다.

예를 들어, 예약한 식별자가 `2000`인 경우 `para_id` 필드를 `2000`으로 설정합니다:

   ```json
   ...
   "relay_chain": "rococo-local",
   "para_id": 2000,
   "codeSubstitutes": {},
   "genesis": {
      ...
   }
   ```

`parachainId`를 예약한 파라체인 식별자로 설정합니다.

예를 들어, 예약한 식별자가 `2000`인 경우 `para_id` 필드를 `2000`으로 설정합니다:

   ```json
   ...
      "parachainSystem": null,
      "parachainInfo": {
        "parachainId": 2000
      },
   ...
   ```

동일한 로컬 네트워크에서 이 튜토리얼을 동시에 완료하는 경우 프로토콜 ID를 고유하게 만들기 위해 다음 줄을 찾아 문자를 추가해야 합니다:

```json
   "protocolId": "template-local"
```

변경 사항을 저장하고 평문 체인 사양 파일을 닫습니다.

수정된 체인 사양 파일에서 원시 체인 사양 파일을 생성하려면 다음 명령을 실행하세요:

   ```bash
   ./target/release/parachain-template-node build-spec --chain plain-parachain-chainspec.json --disable-default-bootnode --raw > raw-parachain-chainspec.json
   ```

이 명령은 두 개의 콜레이터가 있는 새로운 원시 체인 사양 파일을 생성합니다.

   ```text
   2022-08-30 13:00:50 Building chain spec
   2022-08-30 13:00:50 assembling new collators for new session 0 at #0
   2022-08-30 13:00:50 assembling new collators for new session 1 at #0
   ```

파라체인 콜레이터를 등록하기 위해 로컬 릴레이 체인이 실행 중이고 파라체인 템플릿의 원시 체인 사양이 업데이트되었으므로 파라체인 콜레이터 노드를 시작하고 런타임 및 제네시스 상태에 대한 정보를 내보냅니다.

등록할 파라체인 콜레이터를 준비하려면 다음을 수행하세요:

1. 파라체인을 위한 WebAssembly 런타임을 내보냅니다.

   릴레이 체인은 파라체인 블록을 유효성 검사하기 위해 파라체인에 특화된 런타임 유효성 검사 로직이 필요합니다.
   파라체인 콜레이터 노드에 대한 WebAssembly 런타임을 내보내려면 다음과 유사한 명령을 실행합니다:

   ```bash
   ./target/release/parachain-template-node export-genesis-wasm --chain raw-parachain-chainspec.json para-2000-wasm
   ```

2. 파라체인 제네시스 상태를 생성합니다.

   파라체인을 등록하려면 릴레이 체인이 파라체인의 제네시스 상태를 알아야 합니다.
   전체 제네시스 상태를 16진수로 인코딩하여 파일로 내보내려면 다음과 유사한 명령을 실행합니다:

   ```bash
   ./target/release/parachain-template-node export-genesis-state --chain raw-parachain-chainspec.json para-2000-genesis-state
   ```

   내보낸 런타임 및 상태는 _제네시스_ 블록을 위한 것이어야 합니다.
   이전 상태를 가진 파라체인을 릴레이 체인에 연결할 수 없습니다.
   모든 파라체인은 릴레이 체인의 블록 0부터 시작해야 합니다.
   파라체인 템플릿이 어떻게 생성되었고 체인 로직(이력 또는 상태 마이그레이션은 아님)을 파라체인으로 변환하는 방법에 대한 자세한 내용은 [솔로 체인 변환](/reference/how-to-guides/parachains/convert-a-solo-chain/)을 참조하세요.

3. 다음과 유사한 명령을 실행하여 다음과 같이 콜레이터 노드를 시작합니다:

   ```bash
   ./target/release/parachain-template-node \
   --alice \
   --collator \
   --force-authoring \
   --chain raw-parachain-chainspec.json \
   --base-path /tmp/parachain/alice \
   --port 40333 \
   --rpc-port 8844 \
   -- \
   --execution wasm \
   --chain ../polkadot/raw-local-chainspec.json \
   --port 30343 \
   --rpc-port 9977
   ```

   이 명령에서 `--` 인자 이전에 전달된 인수는 파라체인 템플릿 콜레이터를 위한 것입니다.
   `--` 이후의 인수는 내장된 릴레이 체인 노드를 위한 것입니다.
   이 명령에서는 파라체인의 원시 체인 사양과 릴레이 체인의 원시 체인 사양을 모두 지정합니다.
   이 예제에서는 `polkadot` 디렉토리에 있는 `raw-local-chainspec.json`이라는 로컬 릴레이 체인의 원시 체인 사양을 사용합니다.
   두 번째 `--chain` 명령줄이 로컬 릴레이 체인의 원시 체인 사양 파일 경로를 지정하도록 확인하세요.

   파라체인을 위해 다른 노드를 시작하는 경우 동일한 릴레이 체인 사양 파일을 사용하지만 다른 기본 경로와 포트 번호를 사용해야 합니다.

   파라체인 템플릿 노드를 시작한 터미널에서 다음과 유사한 출력을 볼 수 있어야 합니다:

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
   [Relaychain] Running JSON-RPC server: addr=127.0.0.1:9977, allowed origins=["http://localhost:*", "http://127.0.0.1:*", "https://localhost:*", "https://127.0.0.1:*", "https://polkadot.js.org"]
   [Relaychain] 〽️ Prometheus exporter started at 127.0.0.1:9616
   [Parachain] 🏷  Local node identity is: 12D3KooWF464pkLaHbsfc4DzDkYuhdVE4zSqBHR1gPapLvwsfZtg
   [Relaychain] discovered: 12D3KooWCq8n5PzroHvxEyCbHigb5ZWc6q8WL7E25iuWYbJod9D2 /ip4/10.105.172.196/tcp/30334
   [Relaychain] discovered: 12D3KooWCzfdGstKxFiZ5QdKQVmQVgcBgE8r7FfBrG8bCRsia6Bo /ip4/10.105.172.196/tcp/30333
   [Relaychain] discovered: 12D3KooWCzfdGstKxFiZ5QdKQVmQVgcBgE8r7FfBrG8bCRsia6Bo /ip4/10.96.0.2/tcp/30333
   [Relaychain] discovered: 12D3KooWCq8n5PzroHvxEyCbHigb5ZWc6q8WL7E25iuWYbJod9D2 /ip4/10.96.0.2/tcp/30334
   [Parachain] 💻 Operating system: ...
   ......
   [Parachain] 📦 Highest known block at #0
   [Parachain] Running JSON-RPC server: addr=127.0.0.1:8844, allowed origins=["http://localhost:*", "http://127.0.0.1:*", "https://localhost:*", "https://127.0.0.1:*", "https://polkadot.js.org"]
   ...
   ```

   템플릿 콜레이터 노드가 독립적인 노드로 실행되고 내장된 릴레이 체인 노드가 로컬 릴레이 체인 검증자 노드와 피어로 연결되는 것을 볼 수 있어야 합니다.
   내장된 릴레이 체인이 로컬 릴레이 체인 노드와 피어링되지 않는 경우 방화벽을 비활성화하거나 릴레이 노드의 주소를 `bootnodes` 매개변수에 추가해 보세요.

## 로컬 릴레이 체인에 등록하기

로컬 릴레이 체인과 콜레이터 노드가 실행 중인 상태에서 로컬 릴레이 체인에 파라체인을 등록할 준비가 되었습니다.
실제 공개 네트워크에서 등록은 일반적으로 [파라체인 경매](https://wiki.polkadot.network/docs/en/learn-auction)를 포함합니다.
이 튜토리얼과 로컬 테스트를 위해 Sudo 트랜잭션과 Polkadot/Substrate Portal을 사용할 수 있습니다.
Sudo 트랜잭션을 사용하면 파라체인 또는 파라스레드 슬롯을 얻기 위해 필요한 단계를 우회할 수 있습니다.

파라체인을 등록하려면 다음을 수행하세요:

1. 로컬 릴레이 체인 검증자가 실행 중인지 확인합니다.
2. 브라우저에서 [Polkadot/Substrate Portal](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/parachains/parathreads)을 엽니다.
3. 로컬 릴레이 체인 노드에 연결합니다(필요한 경우).
4. **개발자**를 클릭하고 **Sudo**를 선택합니다.

   ![파라체인 등록을 위해 Sudo 선택](/media/images/docs/tutorials/parachains/developer-sudo.png)

5. **paraSudoWrapper**를 선택한 다음 **sudoScheduleParaInitialize(id, genesis)**를 선택하여 다음 릴레이 체인 세션 시작 시 예약한 paraID를 초기화합니다.

   트랜잭션 매개변수에 대한 description:

   - `id`: 예약한 `ParaId`를 입력합니다.
     이 튜토리얼에서는 예약한 식별자가 2000인 경우입니다.

   - `genesisHead`: **파일 업로드**를 클릭하고 파라체인을 위해 내보낸 제네시스 상태를 업로드합니다.
     이 튜토리얼에서는 `para-2000-genesis` 파일을 선택합니다.

   - `validationCode`: **파일 업로드**를 클릭하고 파라체인을 위해 내보낸 WebAssembly 런타임을 업로드합니다.
     이 튜토리얼에서는 `para-2000-wasm` 파일을 선택합니다.
   - `paraKind`: **예**를 선택합니다.

   ![등록을 위한 매개변수 설정](/media/images/docs/tutorials/parachains/register-with-sudo.png)

6. **Sudo 제출**을 클릭합니다.
7. 트랜잭션 세부 정보를 검토한 다음 **서명 및 제출**을 클릭하여 트랜잭션을 승인합니다.

   트랜잭션을 제출한 후 **네트워크**를 클릭하고 **탐색기**를 선택합니다.

8. 최근 이벤트 목록에서 성공적인 `sudo.Sudid` 및 `paras.PvfCheckAccepted` 이벤트를 확인하고 이벤트를 클릭하여 트랜잭션에 대한 세부 정보를 확인합니다.

   ![Sudo 등록 이벤트 보기](/media/images/docs/tutorials/parachains/sudo-registration-event.png)

   파라체인이 초기화되면 Polkadot/Substrate Portal에서 확인할 수 있습니다. **네트워크**를 클릭하고 **파라체인**을 선택하세요.

9. **네트워크**를 클릭하고 **파라체인**을 선택하고 새로운 epoch가 시작될 때까지 기다립니다.

   릴레이 체인은 각 파라체인의 최신 블록(헤드)을 추적합니다.
   릴레이 체인 블록이 최종화되면 [검증 과정](https://polkadot.network/the-path-of-a-parachain-block)을 완료한 파라체인 블록도 최종화됩니다.
   이것이 Polkadot이 파라체인에 대해 **풀된 공유 보안**을 달성하는 방법입니다.

   다음 epoch에서 파라체인이 릴레이 체인에 연결되고 첫 번째 블록을 최종화하면 Polkadot/Substrate Portal에서 해당 파라체인에 대한 정보를 볼 수 있습니다.

   파라체인이 실행되고 릴레이 체인에 연결된 후, Polkadot/Substrate Portal을 사용하여 파라체인에 트랜잭션을 제출할 수 있습니다.

## 연결 및 트랜잭션 제출

지금까지 로컬 네트워크에 연결하고 로컬 릴레이 체인에 트랜잭션을 제출하기 위해 Polkadot/Substrate Portal을 사용했습니다.
이제 파라체인이 실행되고 릴레이 체인에 연결되었으므로 Polkadot/Substrate Portal을 사용하여 파라체인에 트랜잭션을 제출할 수 있습니다.

파라체인에 연결하고 트랜잭션을 제출하려면 다음을 수행하세요:

1. 새로운 브라우저 창이나 탭에서 [Polkadot/Substrate Portal](https://polkadot.js.org/apps/#/explorer)을 엽니다.

2. 응용 프로그램의 왼쪽 상단에 있는 네트워크 선택기를 클릭합니다.

3. 사용자 정의 엔드포인트를 파라체인의 WebSocket 포트에 연결하도록 변경합니다.

   이 튜토리얼의 설정을 따르면 포트 `8844`에 연결합니다.

4. **계정**을 클릭하고 **전송**을 선택하여 Alice에서 다른 계정으로 자금을 보냅니다.

   - 자금을 보낼 계정을 선택합니다.
   - 금액을 입력합니다.
   - **전송 생성**을 클릭합니다.
   - 트랜잭션을 검토한 다음 **서명 및 제출**을 클릭하여 전송을 승인합니다.

5. **계정**을 클릭하여 전송이 완료되고 파라체인 트랜잭션이 성공적으로 수행되었는지 확인합니다.

   트랜잭션이 성공한 경우 작동하는 파라체인이 있습니다.

## 블록체인 상태 재설정하기

이 튜토리얼에서 로컬 릴레이 체인에 연결한 파라체인 콜레이터에는 파라체인의 모든 블록체인 데이터가 포함되어 있습니다.
이 파라체인 네트워크에는 노드가 하나뿐이므로 제출한 트랜잭션은 이 노드에만 저장됩니다.
릴레이 체인은 파라체인 상태를 저장하지 않습니다.
릴레이 체인은 연결된 파라체인의 헤더 정보만 저장합니다.

테스트 목적으로는 주기적으로 블록체인 상태를 초기화하여 처음부터 다시 시작할 수 있습니다.
그러나 체인 상태를 삭제하거나 데이터베이스를 수동으로 삭제하는 경우 데이터를 복구하거나 체인 상태를 복원할 수 없으므로 주의해야 합니다.
보존하려는 데이터가 있는 경우 파라체인 상태를 초기화하기 전에 데이터의 사본을 보관하도록 해야 합니다.

테스트를 위해 깨끗한 환경에서 다시 시작하려면 로컬 릴레이 체인 노드와 파라체인의 체인 상태를 완전히 제거해야 합니다.

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

다음 단계로 넘어가기

- [추가 파라체인 노드 추가하기](/reference/how-to-guides/parachains/add-paranodes/)
- [솔로 체인 변환](/reference/how-to-guides/parachains/convert-a-solo-chain/)
- [Polkadot 위키](https://wiki.polkadot.network/docs/learn-collator)
- [테스트넷 슬롯 확보](/tutorials/build-a-parachain/acquire-a-testnet-slot/)