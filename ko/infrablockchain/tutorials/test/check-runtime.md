---
title: 런타임 확인
description: 지정된 런타임 상태를 프로덕션 스냅샷의 체인 상태와 테스트하기 위한 try-runtime 명령행 도구에 대해 설명합니다.
keywords:
  - 테스트
  - 스냅샷
  - 통합 테스트
  - 프로덕션
  - 런타임 업그레이드
  - 스토리지 마이그레이션
---

`try-runtime` 명령행 도구를 사용하면 [in-memory-externalities](https://github.com/InfraBlockchain/infrablockspace-sdk/blob/2e28ab448c5e5e27198ba80b726701479cc982fd/substrate/primitives/state-machine/src/testing.rs#L42C11-L42C11) 데이터 구조를 사용하여 런타임 스토리지의 스냅샷을 쿼리하여 상태를 저장할 수 있습니다.
인메모리 스토리지를 사용하여 지정된 런타임 상태에 대한 테스트를 작성하여 프로덕션으로 이동하기 전에 실제 체인 상태에 대해 테스트할 수 있습니다.

가장 간단한 형태로 `try-runtime`을 사용하여 다음과 같이 런타임 상태를 테스트할 수 있습니다:

1. 원격 노드에 연결합니다.
2. 특정 런타임 API를 호출합니다.
3. 특정 블록에서 노드로부터 상태를 검색합니다.
4. 검색한 데이터에 대해 테스트를 작성합니다.

## 동기

`try-runtime`의 초기 동기는 실제 체인의 상태에 대해 런타임 변경 사항을 테스트해야 하는 필요성에서 비롯되었습니다.
이전에는 단위 및 통합 테스트를 위해 `TestExternalities` 와 [`BasicExternalities`](https://github.com/InfraBlockchain/infrablockspace-sdk/blob/2e28ab448c5e5e27198ba80b726701479cc982fd/substrate/primitives/state-machine/src/basic.rs#L41)가 있었지만, 실제 체인의 상태에 대해 테스트할 수 있는 기능이 부족했습니다.
`try-runtime` 도구는 다음과 같은 노드의 RPC 엔드포인트를 사용하여 상태를 검색함으로써 `TestExternalities`와 `BasicExternalities`를 확장합니다:

- `rpc_get_storage`
- `rpc_get_keys_paged`

(자세한 내용은 [`remote externalities lib`](https://github.com/InfraBlockchain/infrablockspace-sdk/blob/2e28ab448c5e5e27198ba80b726701479cc982fd/substrate/utils/frame/remote-externalities/src/lib.rs#L108C12-L108C23)를 참조하세요.)

Key-Value 데이터베이스를 사용하여 상태를 검색한 후, try-runtime은 해당 데이터를 `TestExternalities`에 삽입합니다.

## 작동 방식

`try-runtime` 도구는 `remote_externalities` 라는 자체적인 externalities 구현을 가지고 있으며, 이는 `TestExternalities`를 래핑하고 데이터가 [타입 인코딩](../../learn/substrate/learn/frame/scale-codec.md)된 [Key-Value 저장소](../../learn/substrate/learn/frame/state-transitions-and-storage.md)를 사용합니다.

아래 다이어그램은 externalities가 컴파일된 런타임 외부에 위치하여 해당 런타임의 스토리지를 캡처하는 방식을 보여줍니다.

![스토리지 externalities](/media/images/docs/reference/try-runtime-ext-1.png)

`remote_externalities`를 사용하면 일부 체인 상태를 캡처하고 이를 기반으로 테스트를 실행할 수 있습니다. 기본적으로 `RemoteExternalities`는 실제 체인의 데이터로 `TestExternalities`를 채웁니다.

![externalities로 테스트하기](/media/images/docs/reference/try-runtime-ext-2.png)

상태를 쿼리하기 위해 `try-runtime`은 [`StateApiClient`](https://github.com/InfraBlockchain/infrablockspace-sdk/blob/2e28ab448c5e5e27198ba80b726701479cc982fd/substrate/client/rpc-api/src/state/mod.rs#L35)에서 제공하는 RPC 메서드를 사용합니다.
특히:

- [`storage`](https://github.com/InfraBlockchain/infrablockspace-sdk/blob/2e28ab448c5e5e27198ba80b726701479cc982fd/substrate/client/rpc-api/src/state/mod.rs#L67)
  이 메서드는 지정한 블록을 나타내는 키에 대한 스토리지 값을 반환합니다.

- [`storage_key_paged`](https://github.com/InfraBlockchain/infrablockspace-sdk/blob/2e28ab448c5e5e27198ba80b726701479cc982fd/substrate/client/rpc-api/src/state/mod.rs#L57)
  이 메서드는 지정한 접두사와 페이지네이션 지원과 일치하는 키를 반환합니다.

## 사용법

`try-runtime`의 가장 일반적인 사용 사례는 스토리지 마이그레이션과 런타임 업그레이드를 준비하는 데 도움을 줍니다.

스토리지를 쿼리하는 RPC 호출은 계산 비용이 많이 들기 때문에 `try-runtime` 명령을 사용하기 전에 실행 중인 노드에 다음과 같은 몇 가지 명령행 옵션을 설정해야 합니다. `try-runtime` 테스트를 위해 노드를 준비하려면 다음 옵션을 설정하세요:

- 대용량 RPC 쿼리가 작동할 수 있도록 `--rpc-max-payload 1000`을 설정합니다.
- WebSocket 연결이 가능하도록 `--rpc-cors all`을 설정합니다.

`try-runtime`을 [`fork-off-substrate`](https://github.com/maxsam4/fork-off-substrate)와 결합하여 프로덕션 이전에 체인을 테스트할 수 있습니다.
`try-runtime`을 사용하여 체인의 마이그레이션 및 사전 및 사후 상태를 테스트한 다음, 마이그레이션 이후에도 블록 생성이 계속되는지 확인하려면 `fork-off-substrate`를 사용하세요.

### 런타임 업그레이드 후크

기본적으로 런타임 업그레이드 후크는 런타임 내부 또는 팔렛 내부에 정의할 수 있는데, 런타임 업그레이드가 발생했을 때 어떤 작업이 수행되어야 하는지를 지정합니다.
즉, 기본 `on_runtime_upgrade` 메서드는 업그레이드 이후의 런타임 상태만을 설명합니다.
그러나 `try-runtime`에서 제공하는 메서드를 사용하여 테스트 목적으로 런타임 업그레이드 전후의 런타임 상태를 검사하고 비교할 수 있습니다.

런타임에 `try-runtime` 기능을 활성화하면 다음과 같이 런타임에 대한 `pre-upgrade` 및 `post-upgrade` 후크를 정의할 수 있습니다:

```rust
#[cfg(feature = "try-runtime")]
fn pre_upgrade() -> Result<Vec<u8>, &'static str> {
		Ok(Vec::new())
}

#[cfg(feature = "try-runtime")]
fn post_upgrade(_state: Vec<u8>) -> Result<(), &'static str> {
		Ok(())
}
```

이러한 함수를 사용하면 `pre_upgrade` 후크를 사용하여 런타임 상태를 검색하고 Vec<u8> 결과로 반환할 수 있습니다.
그런 다음 Vec<u8>를 `post_upgrade` 후크의 입력 매개변수로 전달할 수 있습니다.

## 명령행 예제

명령행에서 `try-runtime`을 사용하려면 노드를 `--features=try-runtime` 플래그와 함께 실행하세요.
예를 들어:

```bash
cargo run --release --features=try-runtime try-runtime
```

`try-runtime`과 다음과 같은 하위 명령을 사용할 수 있습니다:

- `on-runtime-upgrade`: 주어진 런타임 상태에 대해 `tryRuntime_on_runtime_upgrade`를 실행합니다.
- `offchain-worker`: 주어진 런타임 상태에 대해 `offchainWorkerApi_offchain_worker`를 실행합니다.
- `execute-block`: 주어진 블록과 부모 블록의 런타임 상태를 사용하여 `core_execute_block`을 실행합니다.
- `follow-chain`: 주어진 체인의 최종 블록을 따라가고 모든 익스트린식에 적용합니다.
  이를 통해 새로운 런타임의 동작을 오랜 기간 동안 실제 트랜잭션을 입력으로 받아 검사할 수 있습니다.

특정 `try-runtime` 하위 명령의 사용 정보를 보려면 하위 명령과 `--help` 플래그를 지정하세요.
예를 들어, `try-runtime on-runtime-upgrade`의 사용 정보를 보려면 다음 명령을 실행할 수 있습니다:

```bash
cargo run --release --features=try-runtime try-runtime on-runtime-upgrade --help
```

예를 들어, 로컬에서 실행 중인 체인에 `on-runtime-upgrade` 하위 명령을 사용하여 다음과 같이 `try-runtime`을 실행할 수 있습니다:

```bash
cargo run --release --features=try-runtime try-runtime on-runtime-upgrade live ws://localhost:9944
```

다음과 같이 `try-runtime`을 사용하여 `ElectionProviderMultiPhase` 오프체인 워커의 코드를 `localhost:9944`에서 다시 실행할 수 있습니다:

```bash
cargo run -- --release \
   --features=try-runtime \
   try-runtime \
   --execution Wasm \
   --wasm-execution Compiled \
   offchain-worker \
   --header-at 0x491d09f313c707b5096650d76600f063b09835fd820e2916d3f8b0f5b45bec30 \
   live \
   -b 0x491d09f313c707b5096650d76600f063b09835fd820e2916d3f8b0f5b45bec30 \
   -m ElectionProviderMultiPhase \
   --uri wss://localhost:9944
```

다음과 같이 로컬 런타임의 마이그레이션을 SomeChain의 상태에 대해 실행할 수 있습니다:

```bash
RUST_LOG=runtime=trace,try-runtime::cli=trace,executor=trace \
   cargo run try-runtime \
   --execution Native \
   --chain somechain-dev \
   on-runtime-upgrade \
   live \
   --uri wss://rpc.polkadot.io
```

다음과 같이 특정 블록 번호의 상태에 대해 try-runtime을 실행할 수 있습니다:

```bash
RUST_LOG=runtime=trace,try-runtime::cli=trace,executor=trace \
   cargo run try-runtime \
   --execution Native \
   --chain dev \
   --no-spec-name-check \
   on-runtime-upgrade \
   live \
   --uri wss://rpc.polkadot.io \
   --at <block-hash>
```

이 명령은 `--no-spec-name-check` 명령행 옵션이 필요합니다.

## 다음 단계로 넘어가기

- [스토리지 키](../../learn/substrate/learn/frame/runtime-storage.md)
- [`OnRuntimeUpgrade`](https://github.com/InfraBlockchain/infrablockspace-sdk/blob/2e28ab448c5e5e27198ba80b726701479cc982fd/substrate/frame/support/src/traits/hooks.rs#L103)
- [`try-runtime-upgrade`](https://github.com/InfraBlockchain/infrablockspace-sdk/blob/2e28ab448c5e5e27198ba80b726701479cc982fd/substrate/frame/support/src/traits/hooks.rs#L120)