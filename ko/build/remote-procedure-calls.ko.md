---
title: RPC
description: RPC 및 RPC 방법을 사용하여 Substrate 노드와 상호 작용하는 방법을 설명합니다.
keywords:
  - rpc
  - frontend
---

원격 프로시저 호출(RPC) 또는 RPC 방법은 외부 프로그램(예: 브라우저 또는 프론트엔드 애플리케이션)이 Substrate 노드와 통신하는 방법입니다.
일반적으로 이러한 방법은 RPC 클라이언트가 RPC 서버 엔드포인트에 연결하여 특정 유형의 서비스를 요청할 수 있도록 합니다.
예를 들어, 저장된 값을 읽거나 트랜잭션을 제출하거나 노드가 연결된 체인에 대한 정보를 요청하는 데 RPC 방법을 사용할 수 있습니다.

Substrate 노드의 기본 [JSON-RPC 방법](https://polkadot.js.org/docs/substrate/rpc/)에 가장 편리하게 액세스하는 방법은 [Polkadot-JS API](https://polkadot.js.org/docs/api/)를 사용하는 것입니다.

## 안전한 및 안전하지 않은 RPC 방법

RPC 방법은 합의 및 저장소를 포함한 핵심 노드 작업에 액세스할 수 있으며, 블록체인에 대한 트랜잭션 제출 또는 정보 검색을 허용하기 위해 공개 인터페이스로 노출될 수도 있습니다.
따라서 블록체인의 보안을 위해 다른 RPC 방법이 노출하는 내용과 로컬 노드에서 실행되는 것으로 제한할지 또는 공개적으로 사용할 수 있도록 할지를 고려하는 것이 중요합니다.

### 공개 RPC 인터페이스

Substrate 노드는 다음과 같은 명령줄 옵션을 제공하여 RPC 인터페이스를 공개적으로 노출할 수 있도록 합니다:

```bash
--ws-external
--rpc-external
--unsafe-ws-external
--unsafe-rpc-external
```

기본적으로 노드는 RPC 인터페이스를 노출하려고 시도하고 동시에 검증자 노드를 실행하려고 하면 시작을 거부합니다.
`--unsafe-*` 플래그를 사용하면 이러한 보안 조치를 무시할 수 있습니다.
RPC 인터페이스를 노출하면 많은 공격 가능성이 열리므로 신중하게 검토해야 합니다.

노출해야 하는 RPC 방법은 상당히 많지만, 다음과 같은 RPC 방법은 노출해서는 안 됩니다:

- [`submit_extrinsic`](https://paritytech.github.io/substrate/master/sc_rpc_api/author/trait.AuthorApiClient.html) - 로컬 풀에 트랜잭션 제출을 허용합니다.
- [`insert_key`](https://paritytech.github.io/substrate/master/sc_rpc_api/author/trait.AuthorApiClient.html) - 로컬 키스토어에 개인 키를 삽입할 수 있습니다.
- [`rotate_keys`](https://paritytech.github.io/substrate/master/sc_rpc_api/author/trait.AuthorApiClient.html) - 세션 키 회전을 수행합니다.
- [`remove_extrinsic`](https://paritytech.github.io/substrate/master/substrate_rpc_client/trait.AuthorApi.html#method.remove_extrinsic) - 풀에서 트랜잭션을 제거하고 차단합니다.
- [`add_reserved_peer`](https://paritytech.github.io/substrate/master/sc_rpc_api/system/trait.SystemApiClient.html) - 예약된 노드를 추가합니다.
- [`remove_reserved_peer`](https://paritytech.github.io/substrate/master/sc_rpc_api/system/trait.SystemApiClient.html) - 예약된 노드를 제거합니다.

또한 실행에 오랜 시간이 걸릴 수 있는 RPC 방법도 노출해서는 안 됩니다. 이러한 RPC 방법을 사용하는 것은 클라이언트의 동기화를 차단할 수 있습니다.
예를 들어, 다음과 같은 RPC 방법을 사용하는 것을 피해야 합니다:

- [`storage_keys_paged`](https://paritytech.github.io/substrate/master/sc_rpc_api/state/trait.StateApiClient.html) - 특정 접두사를 가진 상태의 모든 키와 페이지네이션 지원을 가져옵니다.
- [`storage_pairs`](https://paritytech.github.io/substrate/master/sc_rpc_api/state/trait.StateApiClient.html) - 특정 접두사를 가진 상태의 모든 키와 해당 값들을 가져옵니다.

이러한 RPC는 `#[rpc(name = "rpc_method")]` 매크로를 사용하여 선언됩니다. 여기서 `rpc_method`는 함수의 이름이 됩니다. 예를 들어, `submit_extrinsic`입니다.

이러한 종류의 호출은 신뢰할 수 없는 사용자로부터 요청이 오는 경우에는 반드시 필터링해야 합니다.
이를 위해 호출을 검사하고 허용된 API 호출 세트만 전달할 수 있는 [JSON-RPC](/reference/glossary#json-rpc) 프록시를 사용해야 합니다.

## Remote_externalities를 위한 RPC

Substrate 노드에 대해 [`remote_externalities`](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/utils/frame/remote-externalities/src/lib.rs#L347-#L746)를 호출하기 위한 몇 가지 특수화된 RPC 방법을 제공합니다.
원격 externalities를 위한 이러한 특수화된 방법을 사용하면 Substrate 노드에 일회성 RPC 호출을 수행하여 블록 및 헤더에 대한 정보를 얻을 수 있습니다.
이러한 호출로 반환되는 정보는 [`try-runtime`](/reference/command-line-tools/try-runtime/)과 같은 도구를 사용하여 테스트 목적으로 유용할 수 있습니다.

## 엔드포인트

로컬에서 Substrate 노드를 시작하면 기본적으로 다음과 같은 엔드포인트가 제공됩니다:

- HTTP 및 WebSocket 엔드포인트: `ws://localhost:9944/`

Substrate 프론트엔드 라이브러리 및 도구 대부분은 이 엔드포인트를 사용하여 블록체인과 상호 작용합니다.
예를 들어, 로컬 노드 또는 공개 체인에 연결하기 위해 Polkadot-JS 애플리케이션을 사용하는 경우 일반적으로 HTTP 및 WebSocket 엔드포인트에 연결합니다.
WebSocket 연결은 프론트엔드 애플리케이션과 요청에 응답하는 백엔드 노드 간에 양방향 통신을 가능하게 합니다.
그러나 `curl` 명령을 사용하여 통신 채널을 유지하지 않고 개별적으로 RPC 방법을 호출할 수도 있습니다.
예를 들어, 시스템 정보를 가져오거나 특정 유형의 블록 상태 변경이 발생할 때 알림을 받기 위해 체인에 구독하는 등의 작업을 수행하기 위해 `curl` 명령을 사용할 수 있습니다.

엔드포인트를 사용하여 RPC 방법을 호출하려면 다음 단계를 따릅니다:

1. 터미널 쉘을 열고 Substrate 노드 템플릿의 루트 디렉토리로 이동합니다.

2. 다음 명령을 실행하여 개발 모드에서 로컬 노드를 시작합니다:

   ```bash
   ./target/release/node-template --dev
   ```

3. 다음 명령을 실행하여 로컬 노드에 연결하고 `rpc_methods` 엔드포인트를 호출합니다:

   ```bash
   curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "rpc_methods"}' http://localhost:9944/
   ```

   이 명령은 로컬 노드에 노출된 JSON-RPC 방법 목록을 반환합니다.

4. 적절한 메서드 이름을 사용하여 추가적인 메서드를 호출합니다.

   예를 들어, 다음 명령을 실행하여 로컬 노드에 대한 버전 정보를 가져올 수 있습니다:

   ```bash
   curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "system_version"}' http://localhost:9944/
   ```

   대부분의 경우, RPC 엔드포인트에 직접 연결하면 JSON 형식의 결과가 반환됩니다.
   예를 들어:

   ```bash
   {"jsonrpc":"2.0","result":"4.0.0-dev-de262935ede","id":1}
   ```

반환된 값을 사람이 읽을 수 있도록 하려면 SCALE 코덱을 사용하여 디코딩할 수 있습니다.
인코딩 및 디코딩에 대한 자세한 내용은 [타입 인코딩 (SCALE)](/reference/scale-codec/)을 참조하십시오.

각 저장소 항목에는 해당하는 상대적인 저장소 키가 있으며, 이를 사용하여 [저장소 쿼리](/main-docs/build/runtime-storage#querying-storage)를 수행합니다.
이것이 RPC 엔드포인트가 어디에서 찾아야 하는지를 알 수 있는 방법입니다.

## 예제

### state_getMetadata

로컬 노드의 메타데이터를 가져오려면 다음 명령을 실행할 수 있습니다:

```bash
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "state_getMetadata"}' http://localhost:9944/
```

이 명령은 메타데이터를 인코딩된 바이트 형식으로 반환합니다. 이를 사람이 읽을 수 있는 형식으로 디코딩하려면 다음과 같은 JavaScript 코드를 사용할 수 있습니다:

```javascript
function get_metadata_request(endpoint) {
  let request = new Request(endpoint, {
    method: "POST",
    body: JSON.stringify({
      id: 1,
      jsonrpc: "2.0",
      method: "state_getMetadata",
    }),
    headers: { "Content-Type": "application/json" },
  });
  return request;
}
```

간단한 텍스트 디코딩:

```javascript
function decode_metadata(metadata) {
  return new TextDecoder().decode(util.hexToU8a(metadata));
}
```

### `state_getStorage`

RPC 요청:

```json
Request:   {"id":1,"jsonrpc":"2.0","method":"state_getStorage",["{storage_key}"]}
```

여기서 `storage_key`는 팔레트, 함수 및 키의 이름을 기반으로 생성된 매개변수입니다:

```javascript
function get_runtime_storage_parameter_with_key(module_name, function_name, key) {
  // We use xxhash 128 for strings the runtime developer can control
  let module_hash = util_crypto.xxhashAsU8a(module_name, 128);
  let function_hash = util_crypto.xxhashAsU8a(function_name, 128);

  // We use blake2 256 for strings the end user can control
  let key_hash = util_crypto.blake2AsU8a(keyToBytes(key));

  // Special syntax to concatenate Uint8Array
  let final_key = new Uint8Array([...module_hash, ...function_hash, ...key_hash]);

  // Return a hex string
  return util.u8aToHex(final_key);
}
```

다음 단계로 넘어가기

- [JSON-RPC의 Rust 구현](https://github.com/paritytech/jsonrpc)
- [유형 인코딩 (SCALE)](/reference/scale-codec)
- [런타임 저장소](/main-docs/build/runtime-storage/)