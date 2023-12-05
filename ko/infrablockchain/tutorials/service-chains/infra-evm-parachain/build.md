---
title: InfraEVM 체인 구축하기 
description: 이 튜토리얼은 InfraEVM 체인을 빌드하고 실행 하는 과정을 설명합니다.
keywords:
  - 파라체인
  - 서비스체인
  - EVM
---

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [*InfraEVM*](../../../service-chains/infra-evm-parachain.md)

## *InfraEVM* 체인

이전 튜토리얼을 완료한 경우 로컬에 *인프라 릴레이 체인* 레포지토리가 있어야 합니다.

1. 컴퓨터의 터미널 셸을 엽니다.

2. 다음 명령을 실행하여 Infra EVM 체인 저장소를 복제합니다:

   ```bash
   git clone https://github.com/InfraBlockchain/infra-evm-parachain
   ```

   이 명령은 `master` 브랜치를 클론합니다.

3. 다음 명령을 실행하여 노드 템플릿 디렉토리의 루트로 이동합니다:

   ```bash
   cd infra-evm-substrate
   ```

   작업을 포함할 새 브랜치를 만듭니다:

   ```bash
   git switch -c my-learning-branch-yyyy-mm-dd
   ```

   `yyyy-mm-dd`를 원하는 식별 정보로 바꾸세요. 숫자로 된 연도-월-일 형식을 권장합니다. 예를 들어:

   ```bash
   git switch -c my-learning-branch-2023-03-01
   ```

4.  다음 명령을 실행하여 노드 템플릿을 컴파일합니다:

   ```bash
   cargo build --release
   ```

   최적화된 빌드를 위해 항상 `--release` 플래그를 사용해야 합니다.
   처음으로 이를 컴파일하는 경우 완료까지 시간이 다소 소요됩니다.

   다음과 유사한 줄이 표시되면 완료됩니다:

   ```bash
   Finished release [optimized] target(s) in 11m 23s
   ```

## 로컬 노드 시작하기

노드가 컴파일되면 좀비넷을 사용하여 릴레이 체인과 Infra DID 체인을 로컬 환경에서 구축 할 준비가 되었습니다.

로컬 Infra DID 체인을 시작하려면 다음 단계를 따르세요:

1. 좀비넷 설정을 확인합니다

   ```bash
   cat zombienet-config.toml
   ```

   ```toml
    [relaychain]
    default_command = "../infra-relay-chain/target/release/infrablockspace"
    default_args = ["-lparachain=debug", "-l=xcm=trace"]
    chain = "infrablockspace-local"

    [[relaychain.nodes]]
    name = "alice"
    validator = true
    rpc_port = 7100
    ws_port = 7101

    [[relaychain.nodes]]
    name = "bob"
    validator = true
    rpc_port = 7200
    ws_port = 7201

    [[relaychain.nodes]]
    name = "charlie"
    validator = true
    rpc_port = 7300
    ws_port = 7301

    [[relaychain.nodes]]
    name = "dave"
    validator = true
    rpc_port = 7400
    ws_port = 7401

    [[relaychain.nodes]]
    name = "eve"
    validator = true
    rpc_port = 7500
    ws_port = 7501

    [[relaychain.nodes]]
    name = "ferdie"
    validator = true
    rpc_port = 7600
    ws_port = 7601

    [[parachains]]
    id = 1338
    chain = "local"
    cumulus_based = true

    # run alice & bob as parachain collator
    [[parachains.collators]]
    name = "alice"
    validator = true
    command = "./target/release/infra-evm"
    args = ["-lparachain=debug", "-l=xcm=trace"]
    rpc_port = 9800
    ws_port = 9801
   ```

   `relaychain`과 `parachains`의 `default_command` 경로가 실제 로컬에 존재하는 경로와 일치하는지 확인합니다. 

   만약 일치하지 않는다면 로컬 환경에 맞게 변경 해 줍니다.

2. 좀비넷을 실행하여 릴레이 체인과 체인을 실행합니다.

    ```shell
    zombienet spawn --provider native zombienet-config.toml
    ```

3. 정상적으로 실행 되었다면 다음과 유사한 터미널 쉘을 확인할 수 있습니다.
  
   ![zombienet](/media/images/docs/infrablockchain/service-chains/infra-evm-parachain-zombienet.png)

4. (선택) Infra EVM 체인의 노드를 확인하면 아래와 유사한 로그를 확인할 수 있습니다.

   ```shell
    2023-10-30 14:49:31.601  INFO main sc_cli::runner: Infrablockspace EVM Parachain
    2023-10-30 14:49:31.601  INFO main sc_cli::runner: ✌️  version 0.9.400-de99471b695
    2023-10-30 14:49:31.601  INFO main sc_cli::runner: ❤️  by Anonymous, 2023-2023
    2023-10-30 14:49:31.601  INFO main sc_cli::runner: 📋 Chain specification: Infra EVM Local Testnet
    2023-10-30 14:49:31.601  INFO main sc_cli::runner: 🏷  Node name: alice-1
    2023-10-30 14:49:31.602  INFO main sc_cli::runner: 👤 Role: AUTHORITY
    2023-10-30 14:49:31.602  INFO main sc_cli::runner: 💾 Database: RocksDb at /var/folders/5s/7k4bxw5d257br6f0r_2s2szr0000gn/T/zombie-4d95fad6e5ea443c24d8ac966b51f680_-24823-ActivcN5BF4l/alice-1/data/chains/infra_evm_local_testnet/db/full
    2023-10-30 14:49:31.602  INFO main sc_cli::runner: ⛓  Native runtime: frontier-parachain-1 (frontier-parachain-0.tx1.au1)
    2023-10-30 14:49:38.352  INFO main infra_evm::command: Parachain id: Id(1338)
    2023-10-30 14:49:38.352  INFO main infra_evm::command: Parachain Account: 5Ec4AhNxt4ALZ7BF5BrpHraWTBwncCBVSBkEVhskp1BdaJTr
    2023-10-30 14:49:38.352  INFO main infra_evm::command: Parachain genesis state: 0x0000000000000000000000000000000000000000000000000000000000000000004961c439475f7931d35a32f06b7a22073cd124be21a4cda32a6db9f31911ae7c03170a2e7597b7b7e3d84c05391d139a62b157e78786d8c082f29dcf4c11131400
    2023-10-30 14:49:38.352  INFO main infra_evm::command: Is collating: yes
    2023-10-30 14:49:54.220  INFO main sc_service::client::client: [Parachain] 🔨 Initializing Genesis block/state (state: 0x4961…ae7c, header-hash: 0x3289…8cf0)
    2023-10-30 14:49:54.279 DEBUG main parachain: [Parachain] Restoring chain level monitor from last finalized block: 0 0x3289…8cf0
    2023-10-30 14:49:54.279 DEBUG main parachain: [Parachain] Restored chain level monitor up to height 1
    2023-10-30 14:50:18.757  INFO main sc_service::client::client: [Relaychain] 🔨 Initializing Genesis block/state (state: 0x7a29…50c4, header-hash: 0x7fb6…295c)
    2023-10-30 14:50:18.812 DEBUG main parachain::chain-selection: [Relaychain] Using dispute aware relay-chain selection algorithm
    2023-10-30 14:50:18.818  INFO main grandpa: [Relaychain] 👴 Loading GRANDPA authority set from genesis on what appears to be first startup.
    2023-10-30 14:50:30.168  INFO main babe: [Relaychain] 👶 Creating empty BABE epoch changes on what appears to be first startup.
    2023-10-30 14:50:30.189  INFO main sub-libp2p: [Relaychain] 🏷  Local node identity is: 12D3KooWGs9F69vuR2sd813FjBiyH11RS9iQ8hsBAAfe7JnLzWhN
    2023-10-30 14:50:30.669  INFO main sc_sysinfo: [Relaychain] 💻 Operating system: macos
    2023-10-30 14:50:30.669  INFO main sc_sysinfo: [Relaychain] 💻 CPU architecture: aarch64
    2023-10-30 14:50:30.669  INFO main sc_service::builder: [Relaychain] 📦 Highest known block at #0
    2023-10-30 14:50:30.852  INFO main sc_rpc_server: [Relaychain] Running JSON-RPC HTTP server: addr=127.0.0.1:52410, allowed origins=["http://localhost:*", "http://127.0.0.1:*", "https://localhost:*", "https://127.0.0.1:*", "https://polkadot.js.org"]
    2023-10-30 14:50:30.860  INFO main sc_rpc_server: [Relaychain] Running JSON-RPC WS server: addr=127.0.0.1:52621, allowed origins=["http://localhost:*", "http://127.0.0.1:*", "https://localhost:*", "https://127.0.0.1:*", "https://polkadot.js.org"]
    2023-10-30 14:50:30.953  INFO main sc_sysinfo: [Relaychain] 🏁 CPU score: 76.98 MiBs
    2023-10-30 14:50:30.953  INFO main sc_sysinfo: [Relaychain] 🏁 Memory score: 2.67 GiBs
    2023-10-30 14:50:30.953  INFO main sc_sysinfo: [Relaychain] 🏁 Disk score (seq. writes): 1.72 GiBs
    2023-10-30 14:50:30.953  INFO main sc_sysinfo: [Relaychain] 🏁 Disk score (rand. writes): 37.64 MiBs
    2023-10-30 14:50:30.976  INFO tokio-runtime-worker parachain::approval-voting: [Relaychain] Starting with an empty approval vote DB.
    2023-10-30 14:50:31.028  INFO tokio-runtime-worker babe: [Relaychain] 👶 New epoch 0 launching at block 0x64ab…27d7 (block slot 283107501 >= start slot 283107501).
    2023-10-30 14:50:31.028  INFO tokio-runtime-worker babe: [Relaychain] 👶 Next epoch starts at slot 283107505
    2023-10-30 14:50:31.032  WARN tokio-runtime-worker runtime::inclusion-inherent: [Relaychain] ParentBlockRandomness did not provide entropy
    2023-10-30 14:50:31.035  INFO tokio-runtime-worker substrate: [Relaychain] ✨ Imported #1 (0x64ab…27d7)
    2023-10-30 14:50:31.039  INFO tokio-runtime-worker substrate: [Relaychain] ✨ Imported #2 (0xf32c…28f1)
    2023-10-30 14:50:31.040 DEBUG tokio-runtime-worker parachain::infrablockspace-collator-protocol: [Relaychain] Removing relay parent because our view changed. relay_parent=0x64ab78efb9b8e676b386319384be0a80db7131d2a4778312ca425225618527d7
    2023-10-30 14:50:31.039  INFO                 main sub-libp2p: [Parachain] 🏷  Local node identity is: 12D3KooWHhaSXEhWFi3LibWRNgF9PezoqB9Xeae4fS3dxCowJEg3
    2023-10-30 14:50:31.042  INFO tokio-runtime-worker substrate: [Relaychain] ✨ Imported #3 (0x8a00…3b36)
    2023-10-30 14:50:31.042 DEBUG tokio-runtime-worker parachain::infrablockspace-collator-protocol: [Relaychain] Removing relay parent because our view changed. relay_parent=0xf32cf8199015e1c2c67e9d6dd3da136ac0d30b6cc3be7ef6da3109e4ce9628f1
    2023-10-30 14:50:31.046  WARN tokio-runtime-worker parachain::chain-selection: [Relaychain] Missing block weight for new head. Skipping chain. hash=0x64ab78efb9b8e676b386319384be0a80db7131d2a4778312ca425225618527d7
    2023-10-30 14:50:31.047 DEBUG tokio-runtime-worker parachain::gossip-support: [Relaychain] New session detected session_index=0
    2023-10-30 14:50:31.048  INFO tokio-runtime-worker substrate: [Relaychain] ✨ Imported #4 (0x2a13…5803)
    2023-10-30 14:50:31.048 DEBUG tokio-runtime-worker parachain::infrablockspace-collator-protocol: [Relaychain] Removing relay parent because our view changed. relay_parent=0x8a00ce9e0825886efa2eb70c24b408c2ea186e731c08402926e6aa5b26403b36
    2023-10-30 14:50:31.048 DEBUG tokio-runtime-worker parachain::dispute-distribution: [Relaychain] Dispute coordinator slow? We are still waiting for data on next active leaves update.
    2023-10-30 14:50:31.048  INFO tokio-runtime-worker babe: [Relaychain] 👶 New epoch 1 launching at block 0xb71e…2a13 (block slot 283107505 >= start slot 283107505).
    2023-10-30 14:50:31.048  INFO tokio-runtime-worker babe: [Relaychain] 👶 Next epoch starts at slot 283107509
    2023-10-30 14:50:31.048  INFO tokio-runtime-worker runtime::voting-manager: [Relaychain] [5] 🗳️ ⏰ ending session 0
    2023-10-30 14:50:31.048  INFO tokio-runtime-worker runtime::voting-manager: [Relaychain] [5] 🗳️ ⏰ starting session 1
    2023-10-30 14:50:31.049  INFO                 main sc_sysinfo: [Parachain] 💻 Operating system: macos
    2023-10-30 14:50:31.049  INFO                 main sc_sysinfo: [Parachain] 💻 CPU architecture: aarch64
    2023-10-30 14:50:31.049  INFO                 main sc_service::builder: [Parachain] 📦 Highest known block at #0
    2023-10-30 14:50:31.050  INFO tokio-runtime-worker substrate_prometheus_endpoint: [Parachain] 〽️ Prometheus exporter started at 0.0.0.0:52366
    2023-10-30 14:50:31.050  INFO tokio-runtime-worker substrate: [Relaychain] ✨ Imported #5 (0xb71e…2a13)
    2023-10-30 14:50:31.050 DEBUG tokio-runtime-worker parachain::infrablockspace-collator-protocol: [Relaychain] Removing relay parent because our view changed. relay_parent=0x2a13dd428322a679bd7a7d81e43a4a4488e7c504198e8fbd7759b2069b395803
    2023-10-30 14:50:31.052 DEBUG tokio-runtime-worker parachain::dispute-distribution: [Relaychain] Dispute coordinator slow? We are still waiting for data on next active leaves update.
    2023-10-30 14:50:31.053  INFO                 main sc_rpc_server: [Parachain] Running JSON-RPC HTTP server: addr=0.0.0.0:52365, allowed origins=["*"]
    2023-10-30 14:50:31.053  INFO                 main sc_rpc_server: [Parachain] Running JSON-RPC WS server: addr=0.0.0.0:52364, allowed origins=["*"]
    2023-10-30 14:50:31.053 DEBUG tokio-runtime-worker parachain::gossip-support: [Relaychain] Determined past/present/future authorities authority_count=4
    2023-10-30 14:50:31.053 DEBUG tokio-runtime-worker parachain::gossip-support: [Relaychain] Issuing a connection request num=0
    2023-10-30 14:50:31.053 DEBUG tokio-runtime-worker parachain::gossip-support: [Relaychain] error=NotAValidator
    2023-10-30 14:50:31.053 DEBUG tokio-runtime-worker parachain::validator-discovery: [Relaychain] New ConnectToValidators resolved request peer_set=Validation num_peers=0 removed=0
    2023-10-30 14:50:31.053 DEBUG tokio-runtime-worker parachain::dispute-distribution: [Relaychain] Dispute coordinator slow? We are still waiting for data on next active leaves update.
    2023-10-30 14:50:31.054 DEBUG tokio-runtime-worker parachain::dispute-distribution: [Relaychain] Dispute coordinator slow? We are still waiting for data on next active leaves update.
    2023-10-30 14:50:31.055  INFO                 main sc_sysinfo: [Parachain] 🏁 CPU score: 76.98 MiBs
    2023-10-30 14:50:31.055  INFO                 main sc_sysinfo: [Parachain] 🏁 Memory score: 2.67 GiBs
    2023-10-30 14:50:31.055  INFO                 main sc_sysinfo: [Parachain] 🏁 Disk score (seq. writes): 1.72 GiBs
    2023-10-30 14:50:31.055  INFO                 main sc_sysinfo: [Parachain] 🏁 Disk score (rand. writes): 37.64 MiBs
    2023-10-30 14:50:31.055  WARN                 main infra_evm::service: [Parachain] ⚠️  The hardware does not meet the minimal requirements for role 'Authority'.
    2023-10-30 14:50:31.055 DEBUG tokio-runtime-worker parachain::approval-voting: [Relaychain] Insta-approving all candidates block_hash=0x64ab78efb9b8e676b386319384be0a80db7131d2a4778312ca425225618527d7
    2023-10-30 14:50:31.055 DEBUG tokio-runtime-worker parachain::approval-voting: [Relaychain] Imported new block. block_number=1 block_hash=0x64ab78efb9b8e676b386319384be0a80db7131d2a4778312ca425225618527d7 num_candidates=0
    2023-10-30 14:50:31.055 DEBUG tokio-runtime-worker parachain::gossip-support: [Relaychain] New session detected session_index=1
    2023-10-30 14:50:31.055 DEBUG tokio-runtime-worker parachain::chain-selection: [Relaychain] Missing entry for freshly-approved block. Ignoring block_hash=0x64ab78efb9b8e676b386319384be0a80db7131d2a4778312ca425225618527d7
    2023-10-30 14:50:31.055 DEBUG tokio-runtime-worker parachain::approval-distribution: [Relaychain] Got new blocks [(0x64ab78efb9b8e676b386319384be0a80db7131d2a4778312ca425225618527d7, 1)]
    2023-10-30 14:50:31.055 DEBUG tokio-runtime-worker parachain::approval-distribution: [Relaychain] Got new blocks [(0x64ab78efb9b8e676b386319384be0a80db7131d2a4778312ca425225618527d7, 1)]
    2023-10-30 14:50:31.055 DEBUG tokio-runtime-worker parachain::gossip-support: [Relaychain] Determined past/present/future authorities authority_count=4
    2023-10-30 14:50:31.055 DEBUG tokio-runtime-worker parachain::gossip-support: [Relaychain] Issuing a connection request num=0
    2023-10-30 14:50:31.055 DEBUG tokio-runtime-worker parachain::gossip-support: [Relaychain] error=NotAValidator
    2023-10-30 14:50:31.055 DEBUG tokio-runtime-worker parachain::validator-discovery: [Relaychain] New ConnectToValidators resolved request peer_set=Validation num_peers=0 removed=0
    2023-10-30 14:50:31.056 DEBUG tokio-runtime-worker parachain::approval-voting: [Relaychain] Insta-approving all candidates block_hash=0xf32cf8199015e1c2c67e9d6dd3da136ac0d30b6cc3be7ef6da3109e4ce9628f1
    2023-10-30 14:50:31.056 DEBUG tokio-runtime-worker parachain::approval-voting: [Relaychain] Imported new block. block_number=2 block_hash=0xf32cf8199015e1c2c67e9d6dd3da136ac0d30b6cc3be7ef6da3109e4ce9628f1 num_candidates=0
    2023-10-30 14:50:31.056 DEBUG tokio-runtime-worker parachain::approval-distribution: [Relaychain] Got new blocks [(0xf32cf8199015e1c2c67e9d6dd3da136ac0d30b6cc3be7ef6da3109e4ce9628f1, 2)]
    2023-10-30 14:50:31.056 DEBUG tokio-runtime-worker parachain::chain-selection: [Relaychain] Missing entry for freshly-approved block. Ignoring block_hash=0xf32cf8199015e1c2c67e9d6dd3da136ac0d30b6cc3be7ef6da3109e4ce9628f1
    2023-10-30 14:50:31.057 DEBUG tokio-runtime-worker parachain::approval-voting: [Relaychain] Insta-approving all candidates block_hash=0x8a00ce9e0825886efa2eb70c24b408c2ea186e731c08402926e6aa5b26403b36
    2023-10-30 14:50:31.057 DEBUG tokio-runtime-worker parachain::approval-voting: [Relaychain] Imported new block. block_number=3 block_hash=0x8a00ce9e0825886efa2eb70c24b408c2ea186e731c08402926e6aa5b26403b36 num_candidates=0
    2023-10-30 14:50:31.057 DEBUG tokio-runtime-worker parachain::approval-distribution: [Relaychain] Got new blocks [(0x8a00ce9e0825886efa2eb70c24b408c2ea186e731c08402926e6aa5b26403b36, 3)]
    2023-10-30 14:50:31.057 DEBUG tokio-runtime-worker parachain::approval-voting: [Relaychain] Block finalized block_hash=0xf32cf8199015e1c2c67e9d6dd3da136ac0d30b6cc3be7ef6da3109e4ce9628f1 block_number=2
    2023-10-30 14:50:31.058 DEBUG tokio-runtime-worker parachain::approval-voting: [Relaychain] Insta-approving all candidates block_hash=0x2a13dd428322a679bd7a7d81e43a4a4488e7c504198e8fbd7759b2069b395803
    2023-10-30 14:50:31.058 DEBUG tokio-runtime-worker parachain::approval-voting: [Relaychain] Imported new block. block_number=4 block_hash=0x2a13dd428322a679bd7a7d81e43a4a4488e7c504198e8fbd7759b2069b395803 num_candidates=0
    2023-10-30 14:50:31.058 DEBUG tokio-runtime-worker parachain::approval-distribution: [Relaychain] Got new blocks [(0x2a13dd428322a679bd7a7d81e43a4a4488e7c504198e8fbd7759b2069b395803, 4)]
    2023-10-30 14:50:31.059  INFO tokio-runtime-worker parachain::approval-voting: [Relaychain] Advanced session window for approvals update=Advanced { prev_window_start: 0, prev_window_end: 0, new_window_start: 0, new_window_end: 1 }
    2023-10-30 14:50:31.059  INFO tokio-runtime-worker libp2p_mdns::behaviour: [Parachain] discovered: 12D3KooWGs9F69vuR2sd813FjBiyH11RS9iQ8hsBAAfe7JnLzWhN /ip4/172.16.72.203/tcp/52409/ws
    2023-10-30 14:50:31.060  INFO tokio-runtime-worker libp2p_mdns::behaviour: [Relaychain] discovered: 12D3KooWHhaSXEhWFi3LibWRNgF9PezoqB9Xeae4fS3dxCowJEg3 /ip4/172.16.72.203/tcp/52363/ws
    2023-10-30 14:50:31.061 DEBUG tokio-runtime-worker parachain::approval-voting: [Relaychain] Insta-approving all candidates block_hash=0xb71ef579cff9f27781d1fca8b65617b39e15bf2529132f35c4810aaa57852a13
    2023-10-30 14:50:31.061 DEBUG tokio-runtime-worker parachain::approval-voting: [Relaychain] Imported new block. block_number=5 block_hash=0xb71ef579cff9f27781d1fca8b65617b39e15bf2529132f35c4810aaa57852a13 num_candidates=0
    2023-10-30 14:50:31.061 DEBUG tokio-runtime-worker parachain::approval-distribution: [Relaychain] Got new blocks [(0xb71ef579cff9f27781d1fca8b65617b39e15bf2529132f35c4810aaa57852a13, 5)]
    2023-10-30 14:50:35.625 DEBUG tokio-runtime-worker parachain::approval-voting: [Relaychain] Block finalized block_hash=0x8a00ce9e0825886efa2eb70c24b408c2ea186e731c08402926e6aa5b26403b36 block_number=3
    2023-10-30 14:50:35.956  INFO tokio-runtime-worker substrate: [Relaychain] 💤 Idle (6 peers), best: #5 (0xb71e…2a13), finalized #3 (0x8a00…3b36), ⬇ 15.0kiB/s ⬆ 14.4kiB/s
    2023-10-30 14:50:35.981 DEBUG tokio-runtime-worker parachain::chain-selection: [Relaychain] Prepared 0 stagnant entries for pruning up_to=1698555035 min_ts=0 max_ts=0
    2023-10-30 14:50:36.060  INFO tokio-runtime-worker substrate: [Parachain] 💤 Idle (0 peers), best: #0 (0x3289…8cf0), finalized #0 (0x3289…8cf0), ⬇ 1.1kiB/s ⬆ 0.7kiB/s
    2023-10-30 14:50:36.061  INFO tokio-runtime-worker substrate: [Relaychain] ✨ Imported #6 (0x02fa…96de)
    2023-10-30 14:50:36.063 DEBUG tokio-runtime-worker parachain::infrablockspace-collator-protocol: [Relaychain] Removing relay parent because our view changed. relay_parent=0xb71ef579cff9f27781d1fca8b65617b39e15bf2529132f35c4810aaa57852a13
    2023-10-30 14:50:36.081 DEBUG tokio-runtime-worker parachain::approval-voting: [Relaychain] Imported new block. block_number=6 block_hash=0x02fa1c211e1a2b5dff753a3bac241dd82af05bf1a6991b12a20b269e945396de num_candidates=0
    2023-10-30 14:50:36.085 DEBUG tokio-runtime-worker parachain::approval-distribution: [Relaychain] Got new blocks [(0x02fa1c211e1a2b5dff753a3bac241dd82af05bf1a6991b12a20b269e945396de, 6)]
    2023-10-30 14:50:36.087  INFO tokio-runtime-worker cumulus-collator: [Parachain] Starting collation. relay_parent=0x02fa1c211e1a2b5dff753a3bac241dd82af05bf1a6991b12a20b269e945396de at=0x32897a80e3250a99056967665a0fbcf437de7dea41141bd7d946672da33f8cf0
    2023-10-30 14:50:36.090  INFO tokio-runtime-worker sc_basic_authorship::basic_authorship: [Parachain] 🙌 Starting consensus session on top of parent 0x32897a80e3250a99056967665a0fbcf437de7dea41141bd7d946672da33f8cf0
    2023-10-30 14:50:36.095  INFO tokio-runtime-worker sc_basic_authorship::basic_authorship: [Parachain] 🎁 Prepared block for proposing at 1 (1 ms) [hash: 0xe375ef5d1f674f185893a99ae3eea39950ee1bff3e7628bfddfe9d6b3f5996b3; parent_hash: 0x3289…8cf0; extrinsics (2): [0x325c…7a91, 0x6e56…640c]]
    2023-10-30 14:50:36.101  INFO tokio-runtime-worker aura: [Parachain] 🔖 Pre-sealed block for proposal at 1. Hash now 0x36d9078a18be2787aec8497e583483f609e27068723b5e9d8eee7bb3348214ee, previously 0xe375ef5d1f674f185893a99ae3eea39950ee1bff3e7628bfddfe9d6b3f5996b3.
    2023-10-30 14:50:36.102  INFO tokio-runtime-worker cumulus-collator: [Parachain] PoV size { header: 0.2177734375kb, extrinsics: 2.6240234375kb, storage_proof: 2.5107421875kb }
    2023-10-30 14:50:36.102  INFO tokio-runtime-worker substrate: [Parachain] ✨ Imported #1 (0x36d9…14ee)
    2023-10-30 14:50:36.104  INFO tokio-runtime-worker cumulus-collator: [Parachain] Compressed PoV size: 4.5986328125kb
    2023-10-30 14:50:36.104  INFO tokio-runtime-worker cumulus-collator: [Parachain] Produced proof-of-validity candidate. block_hash=0x36d9078a18be2787aec8497e583483f609e27068723b5e9d8eee7bb3348214ee
    2023-10-30 14:50:36.107 DEBUG tokio-runtime-worker parachain::collation-generation: [Relaychain] candidate is generated candidate_hash=0xc54d71d6bb5a4657a1a87106d1f137589e4019388b86a04188fbb5997534d64e pov_hash=0xa2c62ef0450c83fa28db74de570596e005ae7bf175d0ee942782492bf620a05f relay_parent=0x02fa1c211e1a2b5dff753a3bac241dd82af05bf1a6991b12a20b269e945396de para_id=1338 traceID=262260030952830431723299054625831860056
    2023-10-30 14:50:36.108 DEBUG tokio-runtime-worker parachain::infrablockspace-collator-protocol: [Relaychain] Received session info session_index=1
    2023-10-30 14:50:36.108 DEBUG tokio-runtime-worker parachain::infrablockspace-collator-protocol: [Relaychain] Accepted collation, connecting to validators. para_id=1338 relay_parent=0x02fa…96de candidate_hash=0xc54d71d6bb5a4657a1a87106d1f137589e4019388b86a04188fbb5997534d64e pov_hash=0xa2c62ef0450c83fa28db74de570596e005ae7bf175d0ee942782492bf620a05f core=CoreIndex(0) current_validators=[Public(8eaf04151687736326c9fea17e25fc5287613693c912909cb226aa4794f26a48 (5FHneW46...)), Public(d43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d (5GrwvaEF...)), Public(306721211d5404bd9da88e0204360a1a9ab8b87c66c1bc2fcdd37f3c2222cc20 (5DAAnrj7...)), Public(90b5ab205c6974c9ea841be688864633dc9ca8a357843eeacf2314649965fe22 (5FLSigC9...))] traceID=262260030952830431723299054625831860056
    2023-10-30 14:50:36.108 DEBUG tokio-runtime-worker parachain::validator-discovery: [Relaychain] Authority Discovery couldn't resolve Public(d43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d (5GrwvaEF...))
    2023-10-30 14:50:36.108 DEBUG tokio-runtime-worker parachain::validator-discovery: [Relaychain] New ConnectToValidators request peer_set=Collation requested=4 failed_to_resolve=1
    2023-10-30 14:50:36.108 DEBUG tokio-runtime-worker parachain::validator-discovery: [Relaychain] New ConnectToValidators resolved request peer_set=Collation num_peers=3 removed=0
    2023-10-30 14:50:36.110 DEBUG tokio-runtime-worker parachain::network-bridge-rx: [Relaychain] action="PeerConnected" peer_set=Collation version=1 peer=PeerId("12D3KooWRkZhiRhsqmrQ28rt73K7V3aCBpqKrLGSXmZ99PTcTZby") role=Authority
    2023-10-30 14:50:36.110 DEBUG tokio-runtime-worker parachain::network-bridge-rx: [Relaychain] action="PeerConnected" peer_set=Collation version=1 peer=PeerId("12D3KooWPKzmmE2uYgF3z13xjpbFTp63g9dZFag8pG6MgnpSLF4S") role=Authority
    2023-10-30 14:50:36.110 DEBUG tokio-runtime-worker parachain::network-bridge-rx: [Relaychain] action="PeerConnected" peer_set=Collation version=1 peer=PeerId("12D3KooWKM1HeSv61ryZwAiBTZnqmass5pYM48k9Z7obzhTbnphm") role=Authority
    2023-10-30 14:50:36.111 DEBUG tokio-runtime-worker parachain::infrablockspace-collator-protocol: [Relaychain] Advertising collation. relay_parent=0x02fa1c211e1a2b5dff753a3bac241dd82af05bf1a6991b12a20b269e945396de peer_id=12D3KooWPKzmmE2uYgF3z13xjpbFTp63g9dZFag8pG6MgnpSLF4S
    2023-10-30 14:50:36.111 DEBUG tokio-runtime-worker parachain::infrablockspace-collator-protocol: [Relaychain] Advertising collation. relay_parent=0x02fa1c211e1a2b5dff753a3bac241dd82af05bf1a6991b12a20b269e945396de peer_id=12D3KooWKM1HeSv61ryZwAiBTZnqmass5pYM48k9Z7obzhTbnphm
    2023-10-30 14:50:36.112 DEBUG tokio-runtime-worker parachain::infrablockspace-collator-protocol: [Relaychain] Advertising collation. relay_parent=0x02fa1c211e1a2b5dff753a3bac241dd82af05bf1a6991b12a20b269e945396de peer_id=12D3KooWRkZhiRhsqmrQ28rt73K7V3aCBpqKrLGSXmZ99PTcTZby
    2023-10-30 14:50:39.644 DEBUG tokio-runtime-worker parachain::approval-voting: [Relaychain] Block finalized block_hash=0x2a13dd428322a679bd7a7d81e43a4a4488e7c504198e8fbd7759b2069b395803 block_number=4
    2023-10-30 14:50:40.959  INFO tokio-runtime-worker substrate: [Relaychain] 💤 Idle (6 peers), best: #6 (0x02fa…96de), finalized #4 (0x2a13…5803), ⬇ 5.0kiB/s ⬆ 6.3kiB/s
    2023-10-30 14:50:40.985 DEBUG tokio-runtime-worker parachain::chain-selection: [Relaychain] Prepared 0 stagnant entries for pruning up_to=1698555040 min_ts=0 max_ts=0
    2023-10-30 14:50:41.066  INFO tokio-runtime-worker substrate: [Parachain] 💤 Idle (0 peers), best: #0 (0x3289…8cf0), finalized #0 (0x3289…8cf0), ⬇ 79 B/s ⬆ 81 B/s
    2023-10-30 14:50:42.040  INFO tokio-runtime-worker substrate: [Relaychain] ✨ Imported #7 (0x7f32…3263)
   ```
   
## 다음 단계로 넘어가기

- [EVM으로 자산 이동하기](./deposit-and-withdraw-token.md)