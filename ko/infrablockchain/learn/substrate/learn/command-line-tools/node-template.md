---
title: 노드 템플릿
description: 노드 템플릿에 대한 명령 줄 참조 정보입니다.
keywords:
---

`node-template` 프로그램은 FRAME 시스템 팔레트와 일반적인 블록체인 기능 작업을 위한 추가 팔레트의 하위 집합을 가진 작동하는 Substrate 노드를 제공합니다.
기능 팔레트의 기준으로, `node-template`은 자체 블록체인을 구축하고 사용자 정의 런타임을 개발하기 위한 스타터 킷으로 사용됩니다.
`node-template` 프로그램을 사용하여 Substrate 노드를 시작하고 [하위 명령](#subcommands)에 나열된 작업을 수행할 수 있습니다.

## 기본 명령 사용법

`node-template` 명령을 실행하는 기본 구문은 다음과 같습니다:

```shell
node-template [subcommand] [flags] [options]
```

지정한 하위 명령에 따라 추가 인수, 옵션 및 플래그가 적용되거나 필요할 수 있습니다.
특정 `node-template` 하위 명령에 대한 사용 정보를 보려면 하위 명령과 `--help` 플래그를 지정하십시오.
예를 들어, `node-template key`의 사용 정보를 보려면 다음 명령을 실행할 수 있습니다:

```shell
node-template key --help
```

### 플래그

`node-template` 명령과 함께 다음 선택적 플래그를 사용할 수 있습니다.

| 플래그   | 설명
| ------ | -----------
| `--alice`  | 미리 정의된 `Alice` 계정의 세션 키를 로컬 키스토어에 추가합니다. 이 플래그는 명령 줄 옵션으로 `--name alice --validator`를 사용하여 노드를 실행하는 것과 동일합니다.
| `--allow-private-ipv4` | 노드가 사설 IPv4 주소에 연결할 수 있도록 허용합니다. 이 플래그는 노드의 체인 스펙이 `local`로 식별되거나 `--dev` 플래그로 개발 모드에서 노드를 시작하는 경우 기본적으로 활성화됩니다.
| `--bob` | 미리 정의된 `Bob` 계정의 세션 키를 로컬 키스토어에 추가합니다. 이 플래그는 명령 줄 옵션으로 `--name bob --validator`를 사용하여 노드를 실행하는 것과 동일합니다.
| `--charlie` | 미리 정의된 `Charlie` 계정의 세션 키를 로컬 키스토어에 추가합니다. 이 플래그는 명령 줄 옵션으로 `--name charlie --validator`를 사용하여 노드를 실행하는 것과 동일합니다.
| `--dave` | 미리 정의된 `Dave` 계정의 세션 키를 로컬 키스토어에 추가합니다. 이 플래그는 명령 줄 옵션으로 `--name dave --validator`를 사용하여 노드를 실행하는 것과 동일합니다.
| `--dev` | 노드를 개발 모드에서 새로운 상태로 시작합니다. 이 플래그로 노드를 실행하면 상태가 유지되지 않습니다.
| `--disable-log-color` | 로그 메시지에서 색상 사용을 비활성화합니다.
| `--disable-log-reloading` | 로그 필터 업데이트 및 다시로드를 비활성화합니다. 기본적으로 동적 로그 필터링이 활성화되어 있습니다. 그러나 이 기능은 성능에 영향을 줄 수 있습니다. 이 플래그로 노드를 시작하면 `system_addLogFilter` 및 `system_resetLogFilter` 원격 프로시저 호출이 효과가 없습니다.
| `--discover-local`| 로컬 네트워크에서 피어 검색을 활성화합니다. 기본적으로 이 플래그는 `--dev` 플래그로 노드를 시작하거나 체인 스펙이 `Local` 또는 `Development`으로 식별되는 경우 `true`입니다. 그렇지 않으면 `false`입니다.
| `--eve` | 미리 정의된 `Eve` 계정의 세션 키를 로컬 키스토어에 추가합니다. 이 플래그는 명령 줄 옵션으로 `--name eve --validator`를 사용하여 노드를 실행하는 것과 동일합니다.
| `--ferdie` | 미리 정의된 `Ferdie` 계정의 세션 키를 로컬 키스토어에 추가합니다. 이 플래그는 명령 줄 옵션으로 `--name ferdie --validator`를 사용하여 노드를 실행하는 것과 동일합니다.
| `--force-authoring` | 노드가 오프라인 상태에서도 블록 작성을 활성화합니다.
| `-h`, `--help` | 사용 정보를 표시합니다.
| `--ipfs-server` | IPFS 네트워크에 참여하고 bitswap 프로토콜을 통해 트랜잭션을 제공합니다.
| `--kademlia-disjoint-query-paths` | 반복적인 Kademlia 분산 해시 테이블(DHT) 쿼리가 분리된 경로를 사용하도록 요구합니다. 이 옵션은 잠재적으로 적대적인 노드가 존재할 때 내구성을 높입니다. 자세한 내용은 S/Kademlia 논문을 참조하십시오.
| `--no-grandpa` | 노드가 검증자 모드로 실행되지 않으면 GRANDPA 투표를 비활성화합니다. 노드가 검증자로 실행되지 않는 경우, 이 옵션은 GRANDPA 관찰자를 비활성화합니다.
| `--no-mdns` | mDNS 검색을 비활성화합니다. 기본적으로 네트워크는 로컬 네트워크에서 다른 노드를 검색하기 위해 mDNS를 사용합니다. 이 옵션은 검색을 비활성화하며 `--dev` 옵션으로 노드를 시작하는 경우 자동으로 적용됩니다.
| `--no-private-ipv4` | "실시간"으로 표시된 체인의 경우 `--reserved-nodes` 또는 `--bootnodes` 옵션으로 주소가 전달되지 않은 경우, 사설 IPv4 주소에 연결하지 못하도록 합니다. 이 설정은 기본적으로 활성화되어 있습니다.
| `--no-prometheus` | 메트릭을 수신하기 위한 Prometheus 엔드포인트 노출을 비활성화합니다. 기본적으로 메트릭은 Prometheus 엔드포인트로 내보내집니다.
| `--no-telemetry` | Substrate 텔레메트리 서버에 연결하지 않도록 합니다. 텔레메트리는 기본적으로 글로벌 체인에 대해 활성화됩니다.
| `--one` | `--name One --validator`를 지정하는 단축키로 `One`의 세션 키를 키스토어에 추가합니다.
| `--password-interactive` | 터미널 셸에서 키스토어에 연결하는 비밀번호를 지정할 수 있도록 합니다.
| `--prometheus-external` | 모든 인터페이스에서 Prometheus 익스포터를 노출합니다. 기본값은 로컬입니다.
| `--reserved-only` | 체인을 예약된 노드와 동기화하기 위해서만 지정합니다. 이 옵션은 자동 피어 검색도 비활성화합니다. 특히, 검증자인 경우 예약된 노드로 정의되었는지 여부에 관계없이 노드는 여전히 다른 검증자 노드 및 콜레이터 노드에 연결할 수 있습니다.
| `--rpc-external` | 모든 RPC 인터페이스에서 수신 대기합니다. 기본적으로 노드는 로컬 RPC 호출만 수신합니다. 이 명령 줄 옵션을 설정하면 모든 RPC 메서드가 공개적으로 노출되는 것은 안전하지 않을 수 있습니다. 위험한 메서드를 필터링하기 위해 RPC 프록시 서버를 사용하십시오. 공개적으로 노출되지 않아야 하는 RPC 메서드에 대한 자세한 정보는 [원격 프로시저 호출](/build/remote-procedure-calls/)을 참조하십시오. 위험을 이해하고 위험에 대한 경고를 억제하려면 `--unsafe-rpc-external`을 사용하십시오.
| `--storage-chain` | 스토리지 체인 모드를 활성화합니다. 이 옵션을 설정하면 각 트랜잭션이 트랜잭션 데이터베이스 열에 별도로 저장되고 블록 본문 열에서 해시로만 참조됩니다.
| `--tmp` | 임시 노드를 실행합니다. 이 옵션은 블록체인 구성을 저장하는 임시 디렉토리를 생성합니다. 이 디렉토리에는 노드 데이터베이스, 노드 키 및 키스토어가 포함됩니다.
| `--two` | `--name Two --validator`를 지정하는 단축키로 `Two`의 세션 키를 키스토어에 추가합니다.
| `--unsafe-pruning` | 노드를 위험한 가지치기 설정으로 시작합니다. 검증자로 실행할 때 상태 가지치기(즉, 아카이브)를 비활성화하는 것이 매우 권장됩니다. 이 옵션을 설정하지 않으면 가지치기가 활성화되어 있으면 노드가 검증자로 시작되지 않습니다.
| `--unsafe-rpc-external` | 모든 RPC 인터페이스에서 수신 대기합니다. 이 옵션은 `--rpc-external`과 동일합니다.
| `--unsafe-ws-external` | 모든 웹소켓 인터페이스에서 수신 대기합니다. 이 옵션은 `--ws-external`와 동일하지만 경고가 표시되지 않습니다.
| `--validator` | 노드를 권한 역할로 시작하고 가능한 모든 합의 작업에 적극적으로 참여할 수 있도록 활성화합니다(예: 로컬 키의 가용성에 따라 다름).
| `-V`, `--version` | 버전 정보를 표시합니다.
| `--ws-external` | 모든 웹소켓 인터페이스에서 수신 대기합니다. 기본적으로 노드는 로컬에서만 수신합니다. 모든 RPC 메서드가 공개적으로 노출되는 것은 안전하지 않을 수 있습니다. 위험을 이해하고 위험에 대한 경고를 억제하려면 RPC 프록시 서버를 사용하십시오. 위험을 이해하고 위험에 대한 경고를 억제하려면 `--unsafe-ws-external`을 사용하십시오.

### 옵션

`node-template` 명령과 함께 다음 옵션을 사용할 수 있습니다.

| 옵션 | 설명
| ------ | -----------
| `-d`, `--base-path <path>` | 사용자 정의 기본 경로를 지정합니다.
| `--bootnodes <node-identifier>...` | 피어 간 통신을 위한 부트 노드 식별자 목록을 지정합니다.
| `--chain <chain-specification>` | 사용할 체인 스펙을 지정합니다. 미리 정의된 체인 스펙 이름(예: `dev`, `local`, `staging`) 또는 체인 스펙을 포함하는 파일의 경로를 지정할 수 있습니다. 예를 들어, `build-spec` 하위 명령을 사용하여 생성된 체인 스펙 파일의 경로를 지정할 수 있습니다.
| `--database <database>` | 사용할 데이터베이스 백엔드를 선택합니다. `rocksdb`, `paritydb-experimental`, 또는 `auto`를 유효한 값으로 지정할 수 있습니다.
| `--db-cache <MiB>` | 데이터베이스 캐시가 사용할 수 있는 최대 메모리 용량을 제한합니다.
| `--offchain-worker <execution>` | 오프체인 워커 프로세스가 실행되는 시기를 결정합니다. 기본적으로 오프체인 워커는 새로운 블록을 작성하는 노드에서만 활성화되며 블록 유효성 검사 중에 실행됩니다. `Always`, `Never`, 또는 `WhenValidating`을 유효한 값으로 지정할 수 있습니다.
| `--execution <strategy>` | 모든 실행 컨텍스트에서 사용되는 실행 전략을 결정합니다. `Native`, `Wasm`, `Both`, 또는 `NativeElseWasm`을 유효한 값으로 지정할 수 있습니다.
| `--execution-block-construction <strategy>` | 블록을 구성하기 위해 런타임에 호출될 때 사용되는 실행 유형을 지정합니다. `Native`, `Wasm`, `Both`, 또는 `NativeElseWasm`을 유효한 값으로 지정할 수 있습니다.
| `--execution-import-block <strategy>` | 블록(로컬로 작성된 블록 포함)을 가져오기 위해 런타임에 호출될 때 사용되는 실행 유형을 지정합니다. `Native`, `Wasm`, `Both`, 또는 `NativeElseWasm`을 유효한 값으로 지정할 수 있습니다.
| `--execution-offchain-worker <strategy>` | 오프체인 워커를 사용하기 위해 런타임에 호출될 때 사용되는 실행 유형을 지정합니다. `Native`, `Wasm`, `Both`, 또는 `NativeElseWasm`을 유효한 값으로 지정할 수 있습니다.
| `--execution-other <strategy>` | 동기화, 가져오기, 또는 블록 구성 외에 런타임에 호출될 때 사용되는 실행 유형을 지정합니다. `Native`, `Wasm`, `Both`, 또는 `NativeElseWasm`을 유효한 값으로 지정할 수 있습니다.
| `--execution-syncing <strategy>` | 초기 동기화의 일부로 블록을 가져오기 위해 런타임에 호출될 때 사용되는 실행 유형을 지정합니다. `Native`, `Wasm`, `Both`, 또는 `NativeElseWasm`을 유효한 값으로 지정할 수 있습니다.
| `--in-peers <count>` | 수락할 최대 수신 연결 수를 지정합니다. 기본값은 25개 피어입니다.
| `--enable-offchain-indexing <database>` | 오프체인 인덱싱 API를 활성화합니다. 오프체인 인덱싱 API를 사용하면 런타임이 블록 가져오기 중에 오프체인 워커 데이터베이스에 직접 쓸 수 있습니다.
| `--ipc-path <path>` | 원격 프로시저 호출(RPC) 서버로의 프로세스 간 통신(IPC)을 보내기 위한 경로를 지정합니다.
| `--keep-blocks <count>` | 데이터베이스에 유지할 최종 블록 수를 지정합니다. 기본값은 모든 블록을 유지하는 것입니다.
| `--keystore-path <path>` | 사용자 정의 키스토어의 경로를 지정합니다.
| `--keystore-uri <keystore-uri>` | 키스토어 서비스에 연결하기 위한 사용자 정의 URI를 지정합니다.
| `--listen-addr <listen-address>... ` | 노드가 수신 대기할 주소를 지정합니다. 기본적으로 `--validator` 옵션으로 노드를 시작하면 `/ip4/0.0.0.0/tcp/<port>` 및 `/ip6/[::]/tcp/<port>` 주소가 사용됩니다. 그렇지 않으면 `/ip4/0.0.0.0/tcp/<port>/ws` 및 `/ip6/[::]/tcp/<port>/ws` 주소가 사용됩니다.
| `-l`, `--log <log-pattern>...` | 사용자 정의 로깅 필터를 설정합니다. 사용할 구문은 `<log-target>=<level>`입니다. 예를 들어, `-lsync=debug`와 같이 사용할 수 있습니다. 가장 적은에서 가장 자세한 순서로 유효한 로그 레벨은 `error`, `warn`, `info`, `debug`, `trace`입니다. 기본적으로 모든 대상은 `info` 수준 메시지를 기록합니다. 전역 로그 수준은 `-l<level>`로 설정할 수 있습니다.
| `--max-parallel-downloads <count>` | 동시에 동일한 블록을 요청하기 위해 사용할 수 있는 최대 피어 수를 지정합니다. 이 옵션을 사용하면 노드는 여러 피어로부터 발표된 블록을 다운로드할 수 있습니다. 트래픽을 줄이려면 카운트를 감소시킬 수 있지만 지연 시간이 증가할 수 있습니다. 기본값은 5개의 병렬 다운로드입니다.
| `--max-runtime-instances <max-runtime-instances>` | 각 런타임에 대한 인스턴스 캐시의 최대 크기를 지정합니다. 기본값은 8이며 256보다 큰 값은 무시됩니다.
| `--name <name>` | 이 노드의 사람이 읽을 수 있는 이름을 지정합니다. 노드 이름은 텔레메트리 서버에 보고됩니다(활성화된 경우).
| `--node-key <key>` | `libp2p` 네트워킹에 사용할 비밀 키를 지정합니다. 값은 `--node-key-type`에 따라 구문 분석되는 문자열입니다. 예를 들어, 노드 키 유형이 `ed25519`인 경우 노드 키는 16진수로 인코딩된 Ed25519 32바이트 비밀 키(64개의 16진수 문자)로 구문 분석됩니다. 이 옵션의 값은 `--node-key-file`보다 우선합니다. 명령 줄 인수로 제공된 비밀은 쉽게 노출됩니다. 이 옵션은 개발 및 테스트에만 사용해야 합니다. 외부에서 관리되는 비밀 키를 사용하려면 `--node-key-file` 옵션을 사용하십시오.
| `--node-key-file <file>` | `libp2p` 네트워킹에 사용할 노드의 비밀 키가 포함된 파일을 지정합니다. 파일의 내용은 `--node-key-type`에 따라 구문 분석됩니다. 예를 들어, 노드 키 유형이 `ed25519`인 경우 파일은 16바이트 또는 16진수로 인코딩된 Ed25519 비밀 키를 포함해야 합니다. 파일이 존재하지 않으면 `--node-key-type` 옵션으로 지정한 유형의 새로 생성된 비밀 키로 파일이 생성됩니다.
| `--node-key-type <type>` | 피어 간(`libp2p`) 네트워킹에 사용할 비밀 키의 유형을 지정합니다. 비밀 키를 명령 줄 옵션으로 지정하려면 `--node-key` 옵션을 사용하거나 `--node-key-file` 옵션을 사용하여 파일에서 키를 읽거나 `--base-dir` 옵션으로 지정한 기본 디렉토리 내의 체인별 `config` 디렉토리에 지정된 파일에서 키를 읽을 수 있습니다. 이 파일이 존재하지 않으면 선택한 유형의 새로 생성된 비밀 키로 파일이 생성됩니다. 노드의 비밀 키는 `libp2p` 라이브러리를 사용하여 노드와 통신하는 데 사용되는 피어 식별자인 공개 키를 결정합니다. 기본 유형은 Ed25519입니다.
| `--out-peers <count>` | 유지할 최대 발신 연결 수를 지정합니다. 기본값은 25개입니다.
| `--password <password>` | 키스토어에 사용할 비밀번호를 지정합니다.
| `--password-filename <path>` | 키스토어에 사용할 비밀번호가 포함된 파일의 경로를 지정합니다.
| `--pool-kbytes <count>` | 트랜잭션 풀에 저장된 모든 트랜잭션에 대한 최대 킬로바이트 수를 지정합니다. 기본값은 20480 KB입니다.
| `--pool-limit <count>` | 트랜잭션 풀에 포함될 수 있는 최대 트랜잭션 수를 지정합니다. 기본값은 8192개의 트랜잭션입니다.
| `--port <port>` | 피어 간 통신에 사용할 TCP 포트를 지정합니다.
| `--prometheus-port <port>` | Prometheus 익스포터 서비스에 사용할 TCP 포트를 지정합니다.
| `--pruning <pruning-mode>` | 유지할 블록 상태의 최대 수 또는 모든 블록 상태를 유지하기 위해 `archive`를 지정합니다. 노드가 검증자로 실행되는 경우 기본값은 모든 블록 상태를 유지하는 것입니다. 노드가 검증자로 실행되지 않는 경우 마지막 256개 블록의 상태만 유지됩니다.
| `--public-addr <public-address>...` | 다른 노드가 노드에 연결하는 데 사용할 수 있는 공개 주소를 지정합니다. 이 옵션을 사용하여 프록시를 통해 노드에 연결할 수 있습니다.
| `--reserved-nodes <address>...` | 예약된 노드 주소 목록을 지정합니다.
| `--rpc-cors <origins>` | HTTP 및 WS RPC 서버에 액세스할 수 있는 브라우저 출처를 지정합니다. `protocol://domain` 구문, `null` 또는 `all`을 사용하여 이 옵션을 쉼표로 구분된 출처 목록으로 지정할 수 있습니다. `all` 값은 출처 유효성 검사를 비활성화합니다. 기본적으로 `localhost` 및 `https://polkadot.js.org` 출처가 허용됩니다. `--dev` 옵션으로 노드를 시작하면 기본적으로 모든 출처가 허용됩니다.
| `--rpc-http-threads <count>` | RPC HTTP 서버 스레드 풀의 크기를 지정합니다.
| `--rpc-max-payload <rpc-max-payload>` | 요청 및 응답(HTTP 및 웹 소켓 모두)의 최대 RPC 페이로드 크기를 메가바이트 단위로 설정합니다. 기본값은 15 MiB입니다.
| `--rpc-methods <method-set>` | 노출할 RPC 메서드를 지정합니다. `Unsafe`는 모든 RPC 메서드를 노출하고, `Safe`는 안전한 RPC 메서드의 하위 집합만 노출하고, 위험한 RPC 메서드는 거부합니다. `Auto`는 RPC가 외부에서 제공되는 경우(`--rpc-external` 또는 `--rpc-external`로 노드를 실행하는 경우) `Safe` RPC 메서드를 노출하거나, 외부에서 RPC가 제공되지 않는 경우 `Unsafe` RPC 메서드를 노출합니다. 기본값은 `Auto`입니다.  
| `--rpc-port <port>` | HTTP RPC 서버에 사용할 TCP 포트를 지정합니다.
| `--state-cache-size <bytes>` | 상태 캐시 크기를 지정합니다. 기본값은 67108864바이트입니다.
| `--sync <sync-mode>` | 블록체인 동기화 모드를 지정합니다. `Full`은 전체 블록체인 기록을 다운로드하고 유효성을 검사합니다. `Fast`는 블록과 최신 상태만 다운로드합니다. `FastUnsafe`는 최신 상태를 다운로드하지만 상태 증명은 건너뜁니다. 기본값은 `Full`입니다.
| `--telemetry-url <url verbosity>...` | 연결할 텔레메트리 서버의 URL을 지정합니다. 여러 번 이 플래그를 전달하여 여러 텔레메트리 엔드포인트를 지정할 수 있습니다. 상세도 수준은 0에서 9까지이며, 0은 가장 적은 상세도를 나타냅니다. URL 다음에 상세도 옵션을 지정하려면 `--telemetry-url 'wss://foo/bar 0'`와 같은 형식을 사용하십시오.
| `--tracing-receiver <receiver>` | 추적 메시지를 처리할 수신자를 지정합니다. 기본값은 Log입니다.
| `--tracing-targets <targets>` | 사용자 정의 프로파일링 필터를 설정합니다. 구문은 로깅과 동일합니다: `<target>=<level>`입니다.
| `--wasm-execution <method>` | Wasm 런타임 코드를 실행하기 위한 메서드를 지정합니다. `interpreted` 또는 `compiled`을 유효한 값으로 지정할 수 있습니다. 기본값은 `Compiled`입니다.
| `--wasm-runtime-overrides <path>` | 로컬 WASM 런타임이 저장된 경로를 지정합니다. 이러한 런타임은 버전이 일치하는 경우 체인별 `config` 디렉토리 내에서 체인별 런타임을 재정의합니다.
| `--ws-max-connections <count>` | WS RPC 서버 연결의 최대 수를 지정합니다.
| `--rpc-port <port>` | 웹소켓 RPC 서버에 사용할 TCP 포트를 지정합니다.

### 하위 명령

`node-template` 명령과 함께 다음 하위 명령을 사용할 수 있습니다.
이러한 하위 명령을 사용하는 예제 및 참조 정보를 보려면 적절한 명령을 선택하십시오.

| 명령 | 설명
| ------- | -----------
| `benchmark` | 런타임 팔레트를 벤치마크합니다.
| `build-spec` | 체인 스펙을 빌드합니다.
| `check-block` | 블록을 유효성 검사합니다.
| `export-blocks` | 블록을 내보냅니다.
| `export-state` | 주어진 블록의 상태를 체인 스펙으로 내보냅니다.
| `help` | `node-template` 또는 지정된 하위 명령에 대한 사용 정보를 표시합니다.
| `import-blocks` | 블록을 가져옵니다.
| `key` | 로컬 키 관리 유틸리티를 제공합니다.
| `purge-chain` | 모든 체인 데이터를 제거합니다.
| `revert` | 체인을 이전 상태로 되돌립니다.

## benchmark

`node-template benchmark` 명령을 사용하여 런타임 팔레트에서 구성한 외부 호출의 트랜잭션을 실행하는 데 필요한 리소스를 분석할 수 있습니다.
특정 팔레트의 개별 외부 호출 또는 모든 팔레트의 모든 외부 호출을 분석할 수 있습니다.
`benchmark` 하위 명령을 사용하면 다양한 실행 시나리오를 테스트하고 결과를 비교하기 위해 추가적인 명령 줄 옵션을 사용할 수 있습니다.

`node-template benchmark`의 모든 하위 명령을 사용하려면 벤치마킹 기능이 활성화된 상태에서 노드를 컴파일해야 합니다. 벤치마킹 기능이 활성화된 상태에서 노드를 컴파일하려면 다음 명령을 실행하십시오:

```shell
cargo build --package node-template --release --features runtime-benchmarks
```

#### 기본 명령 사용법

```shell
node-template benchmark [subcommand] [flags] [options]
```

지정한 하위 명령에 따라 추가 인수, 옵션 및 플래그가 적용되거나 필요할 수 있습니다.
특정 `benchmark` 하위 명령에 대한 사용 정보를 보려면 하위 명령과 `--help` 플래그를 지정하십시오.
예를 들어, `benchmark pallet`의 사용 정보를 보려면 다음 명령을 실행할 수 있습니다:

```shell
node-template benchmark pallet --help
```
#### 하위 명령

`node-template benchmark` 명령과 함께 다음 하위 명령을 사용할 수 있습니다.

| 명령 | 설명
| ------- | -----------
| `block` | 과거 블록의 실행 시간을 벤치마크합니다.
| `help` | `node-template benchmark` 또는 지정된 하위 명령에 대한 사용 정보를 표시합니다.
| `overhead` | 블록당 및 외부 호출당 실행 오버헤드를 벤치마크합니다.
| `pallet` | FRAME 팔레트의 외부 호출 가중치를 벤치마크합니다.
| `storage` | 체인 스냅샷의 스토리지 속도를 벤치마크합니다.

#### 플래그

`node-template benchmark` 명령과 함께 다음 선택적 플래그를 사용할 수 있습니다.

| 플래그   | 설명
| ------ | -----------
| `-h`, `--help` | 사용 정보를 표시합니다.
| `-V`, `--version` | 버전 정보를 표시합니다.

#### 옵션

`node-template benchmark` 명령과 함께 모든 일반적인 node-template 명령 줄 옵션을 조합하여 사용할 수 있습니다.
예를 들어, `--base-path <path>`를 사용하여 블록체인 데이터를 위한 사용자 정의 디렉토리를 지정하거나 `--chain <chain-specification>`를 사용하여 어떤 `benchmark` 하위 명령에도 사용할 체인 스펙을 지정할 수 있습니다.

그러나 벤치마킹 작업을 수행하는 특정 명령 줄 옵션도 많이 있습니다.
예를 들어, `node-template benchmark block` 하위 명령은 분석할 블록을 지정하기 위해 `--from` 및 `--to` 명령 줄 옵션을 지원합니다.

FRAME 팔레트를 벤치마크하는 것이 가장 일반적인 벤치마킹 작업이므로 `node-template benchmark pallet` 하위 명령은 가장 많은 작업별 명령 줄 옵션을 지원합니다. 
예를 들어, `node-template benchmark pallet` 하위 명령과 다음 옵션을 사용할 수 있습니다.

| 옵션 | 설명
| ------ | -----------
| `--external-repeat` <count> | 클라이언트의 벤치마크를 반복 실행할 횟수를 지정합니다. 이 옵션은 느린 결과를 제공할 수 있지만 Wasm 메모리를 최대화합니다. 기본값은 한 번 실행하는 것입니다.
| `--extra` | 가중치 구성에 필요하지 않은 추가 벤치마크를 표시하고 실행합니다.
| `-e`, `--extrinsic <extrinsic>` | 벤치마크할 팔레트의 개별 함수를 지정하거나 팔레트의 모든 함수 호출을 벤치마크하려면 `*`를 지정합니다.
| `--header <header>` | 벤치마크 출력에 헤더 파일을 추가합니다.
| `--heap-pages <heap-pages>` | 벤치마크 실행 중 힙 페이지를 설정합니다. 설정하지 않으면 노드의 기본값이 사용됩니다.
| `--high <highest-range-values>...` | 각 구성 요소 범위의 최고 값을 나타냅니다.
| `json-input <json-input-file>` | 이전에 생성된 벤치마크 결과가 포함된 JSON 파일의 경로를 지정합니다. 이 옵션을 사용하면 실제 벤치마크 테스트를 다시 실행하지 않고도 `--json-file`로 생성된 벤치마크 원시(raw) 결과를 재사용하여 팔레트의 가중치를 다시 생성하고 분석할 수 있습니다.
| `--list` | 현재 정의된 모든 벤치마크를 나열하고 실행하지 않습니다.
| `--low <lowest-range-values>...` | 각 구성 요소 범위의 최소 값을 나타냅니다.
| `--no-median-slopes` | 중앙 기울기 선형 회귀 분석을 비활성화합니다.
| `--no-min-squares` | 최소 제곱 선형 회귀 분석을 비활성화합니다.
| `--no-storage-info` | 분석 출력에서 스토리지 정보 표시를 비활성화합니다. 이는 출력 파일에 나타나는 스토리지 정보와는 독립적입니다. 이를 위해 Handlebar 템플릿을 사용하십시오.
| `--no-verify` | 벤치마크 실행 시 검증 로직을 비활성화합니다.
| `--

| Flag   | Description
| ------ | -----------
| `--detailed-log-output` | Enables detailed log output, including the log target name, the log level, and the thread name. This option is used automatically if you enable a logging level any higher level than `info`.
| `--dev` | Starts the node in development mode. Using this flag also enables the `--chain=dev`, `--force-authoring`, `--rpc-cors=all`, `--alice`, and `--tmp` flags by default.
| `--disable-log-color` | Disables log color output.
| `--enable-log-reloading` | Enables the log filter to be dynamically updated and reloaded. Note that this option can significantly decrease performance. Setting this option does not affect the `system_addLogFilter` and `system_resetLogFilter` RPC methods.
| `-h`, `--help` | Displays usage information.
| `-V`, `--version` | Displays version information.

#### Options

You can use the following command-line options with the `node-template purge-chain` command.

| Option | Description
| ------ | -----------
| `-d`, `--base-path <path>` | Specifies a custom base path.
| `--chain <chain-specification>` | Specifies the chain specification to use. You can set this option using a predefined chain specification name, such as `dev`, `local`, or `staging`or you can specify the path to a file that contains the chain specification, for example, the chain specification generated by using the `build-spec` subcommand.
| `--database <database>` | Selects the database backend to use. Valid values are `rocksdb`, `paritydb-experimental`, or `auto`.
| `--db-cache <MiB>` | Limits how much memory the database cache can use. The default is 128 MiB.
| `--keep-blocks <count>` | Specifies the number of finalized blocks to keep in the database. The default is to keep all blocks.
| `-l`, `--log <log-pattern>...` | Sets a custom logging filter. The syntax to use is `<log-target>=<level>`, for example `-lsync=debug`. The valid log levels from least to most verbose are `error`, `warn`, `info`, `debug`, and `trace`. By default, all targets log `info` level messages. You can set the global log level with `-l<level>`.
| `--pruning <pruning-mode>` | Specifies the maximum number of block states to keep or `archive` to keep all block states. If the node is running as a validator, the default is to keep all block states. If the node does not run as a validator, only state for the last 256 blocks is kept.
| `--tracing-receiver <receiver>` | Specifies the receiver to process tracing messages. The default is Log.
| `--tracing-targets <targets>` | Sets a custom profiling filter. Syntax is the same as for logging: `<target>=<level>`.

#### Examples

To remove the blockchain and all blockchain-related information, run the following command:

```shell
node-template purge-chain
```

To remove the blockchain and all blockchain-related information using a custom base path, run the following command:

```shell
node-template purge-chain --base-path /path/to/custom/base
```

| 플래그   | 설명
| ------ | -----------
| `--detailed-log-output` | 로그 출력에 자세한 정보를 포함하여 로그 대상 이름, 로그 레벨 및 스레드 이름을 표시합니다. 이 옵션은 `info`보다 높은 로깅 레벨을 활성화하면 자동으로 사용됩니다.
| `--dev` | 개발 모드에서 노드를 시작합니다. 이 플래그를 사용하면 `--chain=dev`, `--force-authoring`, `--rpc-cors=all`, `--alice`, `--tmp` 플래그도 기본적으로 활성화됩니다.
| `--disable-log-color` | 로그 색상 출력을 비활성화합니다.
| `--enable-log-reloading` | 로그 필터를 동적으로 업데이트하고 다시로드할 수 있도록 합니다. 이 옵션은 성능을 크게 저하시킬 수 있습니다. 이 옵션은 `system_addLogFilter` 및 `system_resetLogFilter` RPC 메서드에 영향을 주지 않습니다.
| `-h`, `--help` | 사용법 정보를 표시합니다.
| `--storage-chain` | 블록의 저장 형식을 변경합니다. 이 옵션을 지정하면 각 트랜잭션이 트랜잭션 데이터베이스 열에 별도로 저장되고 블록 본문 열에서는 해당 해시로만 참조됩니다.

| `-V`, `--version` | 버전 정보를 표시합니다.
 `-y` | 대화형 확인 프롬프트를 건너뛰기 위해 사전에 `yes` 응답을 제공합니다.

#### 옵션

`node-template purge-chain` 명령과 함께 다음 명령줄 옵션을 사용할 수 있습니다.

| 옵션 | 설명
| ------ | -----------
| `-d`, `--base-path <path>` | 사용자 정의 기본 경로를 지정합니다.
| `--chain <chain-specification>` | 사용할 체인 스펙을 지정합니다. 미리 정의된 체인 스펙 이름(예: `dev`, `local`, `staging`)을 사용하거나 `build-spec` 하위 명령을 사용하여 생성된 체인 스펙이 포함된 파일의 경로를 지정할 수 있습니다.
| `--database <database>` | 사용할 데이터베이스 백엔드를 선택합니다. 유효한 값은 `rocksdb`, `paritydb-experimental`, `auto`입니다.
| `--db-cache <MiB>` | 데이터베이스 캐시가 사용할 수 있는 메모리 양을 제한합니다. 기본값은 128 MiB입니다.
| `-l`, `--log <log-pattern>...` | 사용자 정의 로깅 필터를 설정합니다. 사용할 구문은 `<log-target>=<level>`입니다. 예를 들어 `-lsync=debug`입니다. 가장 적은부터 가장 상세한 로그 레벨은 `error`, `warn`, `info`, `debug`, `trace`입니다. 기본적으로 모든 대상은 `info` 레벨 메시지를 기록합니다. 전역 로그 레벨을 `-l<level>`로 설정할 수 있습니다.
| `--tracing-receiver <receiver>` | 추적 메시지를 처리할 수신자를 지정합니다. 기본값은 Log입니다.
| `--tracing-targets <targets>` | 사용자 정의 프로파일링 필터를 설정합니다. 구문은 로깅과 동일합니다: `<target>=<level>`.

## 되돌리기

`node-template revert` 명령을 사용하여 체인을 이전 상태로 되돌립니다.

#### 기본 명령 사용법

```shell
node-template revert [flags] [options] [--] [num]
```

#### 플래그

`node-template revert` 명령과 함께 다음 선택적 플래그를 사용할 수 있습니다.

| 플래그   | 설명
| ------ | -----------
| `--detailed-log-output` | 로그 출력에 자세한 정보를 포함하여 로그 대상 이름, 로그 레벨 및 스레드 이름을 표시합니다. 이 옵션은 `info`보다 높은 로깅 레벨을 활성화하면 자동으로 사용됩니다.
| `--dev` | 개발 모드에서 노드를 시작합니다. 이 플래그를 사용하면 `--chain=dev`, `--force-authoring`, `--rpc-cors=all`, `--alice`, `--tmp` 플래그도 기본적으로 활성화됩니다.
| `--disable-log-color` | 로그 색상 출력을 비활성화합니다.
| `--enable-log-reloading` | 로그 필터를 동적으로 업데이트하고 다시로드할 수 있도록 합니다. 이 옵션은 성능을 크게 저하시킬 수 있습니다. 이 옵션은 `system_addLogFilter` 및 `system_resetLogFilter` RPC 메서드에 영향을 주지 않습니다.
| `-h`, `--help` | 사용법 정보를 표시합니다.
| `-V`, `--version` | 버전 정보를 표시합니다.

#### 옵션

`node-template revert` 명령과 함께 다음 명령줄 옵션을 사용할 수 있습니다.

| 옵션   | 설명
| -------- | -----------
| `-d`, `--base-path <path>` | 사용자 정의 기본 경로를 지정합니다.
| `--chain <chain-specification>` | 사용할 체인 스펙을 지정합니다. 미리 정의된 체인 스펙 이름(예: `dev`, `local`, `staging`)을 사용하거나 `build-spec` 하위 명령을 사용하여 생성된 체인 스펙이 포함된 파일의 경로를 지정할 수 있습니다.
| `--keep-blocks <count>` | 데이터베이스에서 유지할 최종 블록 수를 지정합니다. 기본값은 모든 블록을 유지하는 것입니다.
| `-l`, `--log <log-pattern>...` | 사용자 정의 로깅 필터를 설정합니다. 사용할 구문은 `<log-target>=<level>`입니다. 예를 들어 `-lsync=debug`입니다. 가장 적은부터 가장 상세한 로그 레벨은 `error`, `warn`, `info`, `debug`, `trace`입니다. 기본적으로 모든 대상은 `info` 레벨 메시지를 기록합니다. 전역 로그 레벨을 `-l<level>`로 설정할 수 있습니다.
| `--pruning <pruning-mode>` | 유지할 블록 상태의 최대 수 또는 모든 블록 상태를 유지할지를 지정합니다. 노드가 검증자로 실행 중인 경우 기본값은 모든 블록 상태를 유지하는 것입니다. 노드가 검증자로 실행되지 않는 경우 마지막 256개 블록의 상태만 유지됩니다.
| `--tracing-receiver <receiver>` | 추적 메시지를 처리할 수신자를 지정합니다. 기본값은 Log입니다.
| `--tracing-targets <targets>` | 사용자 정의 프로파일링 필터를 설정합니다. 구문은 로깅과 동일합니다: `<target>=<level>`.

#### 인수

`node-template revert` 명령과 함께 다음 명령줄 인수를 사용할 수 있습니다.

| 인수 | 설명
| -------- | -----------
| `<num>` | 되돌릴 블록 수를 지정합니다. 기본값은 256개 블록입니다.