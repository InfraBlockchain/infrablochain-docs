---
title: Substrate Connect에서 라이트 클라이언트 사용하기
description: Substrate Connect를 사용하여 애플리케이션에 라이트 클라이언트를 통합하고 Substrate 기반 체인과 상호 작용할 수 있습니다.
keywords:
  - Substrate Connect
  - 라이트 클라이언트 노드
  - Smoldot 라이트 클라이언트
  - Polkadot
  - Kusama
  - Westend
  - Rococo
---

일반적으로 블록체인의 P2P 네트워킹을 제공하는 노드는 강력한 고속 프로세서와 대용량 저장 장치와 같은 상당한 자원을 필요로 합니다.
반면에 라이트 클라이언트 노드는 자원이 제한된 환경에서 실행되며 응용 프로그램에 내장될 수 있으면서도 블록체인에서 데이터를 동기화할 수 있습니다.

라이트 클라이언트 노드를 사용하면 고성능 하드웨어와 네트워크 용량을 투자하지 않고도 안전하고 탈중앙화된 방식으로 블록체인과 상호 작용할 수 있습니다.

## JavaScript 생태계에서의 라이트 클라이언트

Substrate 기반 체인에서 라이트 클라이언트 노드는 `smoldot`이라는 웹어셈블리 클라이언트로 구현되어 있으며, 이 클라이언트는 브라우저에서 실행되고 JSON-RPC 호출을 사용하여 체인과 상호 작용할 수 있습니다.
`smoldot` 웹어셈블리 라이트 클라이언트를 JavaScript 및 TypeScript 애플리케이션과 쉽게 통합할 수 있도록 `smoldot` 소스 위에 구축된 JavaScript 패키지인 Substrate Connect가 있습니다.

Substrate Connect는 `npm` 패키지 관리자를 사용하여 설치할 수 있는 Node.js 패키지로, 이 패키지를 사용하면 JavaScript 생태계의 애플리케이션과 라이트 클라이언트 노드를 통합할 수 있습니다.
Substrate Connect를 애플리케이션에 추가한 후에는 애플리케이션에서 라이트 클라이언트와 JSON-RPC 메시지를 통해 블록체인 데이터에 액세스할 수 있습니다.

## 브라우저에서 직접 블록체인에 연결하기

Substrate Connect를 사용하면 애플리케이션을 구성하여 컴퓨터에서 로컬로 실행되는 브라우저 내에서 라이트 클라이언트 노드를 실행할 수 있습니다.
브라우저에서 애플리케이션 사용자는 제3자 노드나 다른 서버에 연결하지 않고도 직접적으로 블록체인과 상호 작용할 수 있습니다.

중개 서버의 필요성을 제거함으로써 Substrate Connect는 블록체인 빌더, 애플리케이션 개발자 및 최종 사용자에게 이점을 제공합니다.
주요 이점 중 일부는 다음과 같습니다:

- 향상된 보안
- 단순화된 네트워크 인프라
- 유지 관리 비용 절감
- 초보자 블록체인 사용자의 쉬운 도입
- Web3 애플리케이션의 채택을 빠르게 할 수 있음

## 잘 알려진 블록체인 네트워크

Substrate Connect를 사용하여 Substrate 기반 블록체인에 연결할 수 있습니다.
그러나 연결하려는 체인의 올바른 이름을 지정해야 합니다.
[`WellKnownChain`](https://paritytech.github.io/substrate-connect/api/enums/connect_src.WellKnownChain.html) 열거형에 정의된 몇 가지 잘 알려진 체인 이름이 있습니다.

다음 목록에 나열된 이름을 사용하여 다음 공개 블록체인 네트워크에 연결할 수 있습니다:

| 이 체인에 연결하려면                                                               | 이 체인 식별자를 사용하세요 |
| :--------------------------------------------------------------------------------- | :------------------------ |
| [Polkadot](https://polkadot.network/)                                              | `polkadot`                |
| [Kusama](https://kusama.network/)                                                  | `ksmcc3`                  |
| [Westend](https://wiki.polkadot.network/docs/en/maintain-networks#westend-test-network) | `westend2`                |
| [Rococo](https://polkadot.network/rococo-v1-a-holiday-gift-to-the-polkadot-community/)  | `rococo_v2_2`             |

참고로, 특정 네트워크의 체인 스펙에서 사용되는 일반적으로 사용되는 네트워크 이름보다는 체인 식별자를 사용해야 합니다.
예를 들어, Kusama에 연결하려면 체인 식별자로 `ksmcc3`을 지정해야 합니다.
특히 포크된 체인의 경우 올바른 이름을 사용하는 것이 매우 중요합니다.
예를 들어, `rococo_v2`와 `rococo_v2_2`는 서로 다른 체인입니다.

## Polkadot-JS API를 사용하는 앱에 통합하기

기존의 Polkadot-JS API를 사용하는 애플리케이션을 개발한 경우 `@polkadot/rpc-provider` 패키지에 이미 `substrate-connect` RPC 공급자가 포함되어 있습니다.

`substrate-connect`를 애플리케이션에 추가하려면 다음 단계를 수행하세요:

1. 사용하는 패키지 관리자에 맞는 명령을 실행하여 `@polkadot/rpc-provider` 패키지를 설치합니다.

   예를 들어, `yarn`을 사용하는 경우 다음 명령을 실행하세요:

   ```bash
   yarn add @polkadot/rpc-provider
   ```

   패키지 관리자로 `npm`을 사용하는 경우 다음 명령을 실행하세요:

   ```bash
   npm i @polkadot/rpc-provider
   ```

1. 사용하는 패키지 관리자에 맞는 명령을 실행하여 `@polkadot/api` 패키지를 설치합니다.

   예를 들어, `yarn`을 사용하는 경우 다음 명령을 실행하세요:

   ```bash
   yarn add @polkadot/api
   ```

   패키지 관리자로 `npm`을 사용하는 경우 다음 명령을 실행하세요:

   ```bash
   npm i @polkadot/api
   ```

### 잘 알려진 네트워크에 연결하기 위해 RPC 공급자 사용하기

다음 예제는 `rpc-provider`를 사용하여 Polkadot, Kusama, Westend 또는 Rococo와 같은 잘 알려진 네트워크에 연결하는 방법을 보여줍니다.

```js
import { ScProvider, WellKnownChain } from "@polkadot/rpc-provider/substrate-connect";
import { ApiPromise } from "@polkadot/api";
// 알려진 체인에 대한 공급자 생성
const provider = new ScProvider(WellKnownChain.westend2);
// 연결 설정 (가능한 오류 처리)
await provider.connect();
// PolkadotJS api 인스턴스 생성
const api = await ApiPromise.create({ provider });
await api.rpc.chain.subscribeNewHeads(lastHeader => {
  console.log(lastHeader.hash);
});
await api.disconnect();
```

### 사용자 정의 네트워크에 연결하기 위해 RPC 공급자 사용하기

다음 예제는 `rpc-provider`를 사용하여 체인 스펙을 지정하여 사용자 정의 네트워크에 연결하는 방법을 보여줍니다.

```js
import { ScProvider } from "@polkadot/rpc-provider/substrate-connect";
import { ApiPromise } from "@polkadot/api";
import jsonCustomSpec from "./jsonCustomSpec.json";
// 사용자 정의 체인을 위한 공급자 생성
const customSpec = JSON.stringify(jsonCustomSpec);
const provider = new ScProvider(customSpec);
// 연결 설정 (가능한 오류 처리)
await provider.connect();
// PolkadotJS api 인스턴스 생성
const api = await ApiPromise.create({ provider });
await api.rpc.chain.subscribeNewHeads(lastHeader => {
  console.log(lastHeader.hash);
});
await api.disconnect();
```

### 파라체인에 연결하기 위해 RPC 공급자 사용하기

다음 예제는 `rpc-provider`를 사용하여 체인 스펙을 지정하여 파라체인에 연결하는 방법을 보여줍니다.

```js
import { ScProvider, WellKnownChain } from "@polkadot/rpc-provider/substrate-connect";
import { ApiPromise } from "@polkadot/api";
import jsonParachainSpec from "./jsonParachainSpec.json";
// 릴레이 체인을 위한 공급자 생성
const relayProvider = new ScProvider(WellKnownChain.westend2);
// 파라체인을 위한 공급자 생성. 릴레이 체인의 공급자를 두 번째 인수로 전달해야 함을 주목하세요.
const parachainSpec = JSON.stringify(jsonParachainSpec);
const provider = new ScProvider(parachainSpec, relayProvider);
// 연결 설정 (가능한 오류 처리)
await provider.connect();
// PolkadotJS api 인스턴스 생성
const api = await ApiPromise.create({ provider });
await api.rpc.chain.subscribeNewHeads(lastHeader => {
  console.log(lastHeader.hash);
});
await api.disconnect();
```

## 다른 라이브러리와 함께 Substrate Connect 사용하기

이전 섹션에서는 Substrate Connect 공급자를 Polkadot-JS API를 사용하는 애플리케이션에 통합하는 방법을 보여주었습니다.
이 공급자를 사용하여 Polkadot-JS API 메서드를 호출하여 브라우저를 통해 체인과 상호 작용하는 애플리케이션을 만들 수 있습니다.
그러나 Polkadot-JS API에 의존하지 않는 애플리케이션에도 @substrate-connect를 설치하고 사용할 수 있습니다. 예를 들어, 자체 애플리케이션 라이브러리나 프로그래밍 인터페이스를 구축하는 경우 @substrate-connect 종속성을 설치하고 사용할 수 있습니다. 이를 위해 사용하는 패키지 관리자에 맞는 명령을 실행하세요.

예를 들어, `yarn`을 사용하는 경우 다음 명령을 실행하세요:

```bash
yarn add @substrate/connect
```

패키지 관리자로 `npm`을 사용하는 경우 다음 명령을 실행하세요:

```bash
npm i @substrate/connect
```

### 잘 알려진 체인에 연결하기

다음 예제는 Substrate Connect를 사용하여 Polkadot, Kusama, Westend 또는 Rococo와 같은 잘 알려진 네트워크에 연결하는 방법을 보여줍니다.

```js
import { WellKnownChain, createScClient } from "@substrate/connect";
// 클라이언트 생성
const client = createScClient();
// `jsonRpcCallback` 함수를 전달하면서 체인 연결 생성
const chain = await client.addWellKnownChain(WellKnownChain.polkadot, function jsonRpcCallback(response) {
  console.log("response", response);
});
// RpcRequest 전송
chain.sendJsonRpc('{"jsonrpc":"2.0","id":"1","method":"system_health","params":[]}');
```

### 파라체인에 연결하기

다음 예제는 Substrate Connect를 사용하여 파라체인에 연결하는 방법을 보여줍니다.

```js
import { WellKnownChain, createScClient } from "@substrate/connect";
import jsonParachainSpec from "./jsonParachainSpec.json";
// 클라이언트 생성
const client = createScClient();
// 릴레이 체인 연결 생성. 파라체인 연결을 통해 메시지를 보내고 받을 것이므로
// 콜백 함수를 전달할 필요가 없습니다.
await client.addWellKnownChain(WellKnownChain.westend2);
// 파라체인 연결 생성
const parachainSpec = JSON.stringify(jsonParachainSpec);
const chain = await client.addChain(parachainSpec, function jsonRpcCallback(response) {
  console.log("response", response);
});
// 요청 전송
chain.sendJsonRpc('{"jsonrpc":"2.0","id":"1","method":"system_health","params":[]}');
```

## API 문서

substrate-connect API에 대한 자세한 내용은 [Substrate Connect](https://paritytech.github.io/substrate-connect/api/)를 참조하세요.

## 브라우저 확장 프로그램

Substrate Connect 브라우저 확장 프로그램은 [Substrate Connect](https://github.com/paritytech/substrate-connect) 및 [Smoldot 라이트 클라이언트](https://github.com/smol-dot/smoldot) 노드 모듈을 사용하며, 브라우저가 시작될 때마다 잘 알려진 substrate 체인 스펙(**Polkadot, Kusama, Rococo, Westend**)을 업데이트하고 동기화하여 확장 프로그램 내에서 최신 상태로 유지합니다.

[Substrate Connect](https://github.com/paritytech/substrate-connect)를 통합한 dApp(예: [polkadotJS/apps](https://polkadot.js.org/apps/?rpc=light%3A%2F%2Fsubstrate-connect%2Fpolkadot#/explorer))이 브라우저 탭에서 시작되면, 확장 프로그램에서 마지막으로 가져온 것이 아닌 확장 프로그램 내에서 최신 사양을 받게 됩니다. 동시에, dApp은 확장 프로그램 내에서 "connected"로 표시되며, 이는 확장 프로그램의 부트노드와 사양을 사용하고 있다는 의미입니다.

Chrome 및 Firefox 확장 프로그램을 [Substrate Connect](https://substrate.io/developers/substrate-connect/)에서 다운로드하거나 [Github 저장소](https://github.com/paritytech/substrate-connect/tree/main/projects/extension)에서 자세한 정보를 찾을 수 있습니다.

## 예제 프로젝트

- [Burnr](https://paritytech.github.io/substrate-connect/burnr/)

  보안이 취약한 교환 가능한 지갑: Substrate 기반의 브라우저 내 라이트 클라이언트 기반 지갑입니다.
  사용하기 쉽고 빠르지만 다른 솔루션보다 보안이 덜 강력합니다.
  [Github](https://github.com/paritytech/substrate-connect/tree/main/projects/burnr)

- [Multi-demo](https://paritytech.github.io/substrate-connect/demo/)

  멀티체인 및 파라체인 예제를 다루는 간단한 데모입니다.
  [Github](https://github.com/paritytech/substrate-connect/tree/main/projects/demo)

## Brave 브라우저 WebSocket 문제

**Brave v1.36**부터 확장 프로그램 및 웹 페이지는 사이드 채널 공격을 방지하기 위해 최대 10개의 활성 WebSocket 연결로 제한됩니다.
이 변경 사항에 대한 자세한 내용은 [Partition WebSockets Limits to prevent side channels](https://github.com/brave/brave-browser/issues/19990)에서 확인할 수 있습니다.

Brave 브라우저를 사용하고 WebSocket 연결의 최대 개수를 열어둔 상태로 연결할 수 없는 경우 이 제한을 해제할 수 있습니다.

WebSocket 제한을 해제하려면 다음 단계를 수행하세요:

1. Brave 브라우저에서 새 탭을 엽니다.

2. URL [brave://flags/#restrict-websockets-pool](brave://flags/#restrict-websockets-pool)을 복사합니다.

3. 주소 표시줄에 URL을 붙여넣어 **Restrict WebSockets pool** 설정을 선택합니다.

4. 설정 목록을 클릭하고 **Disabled**를 선택합니다.

   ![Disable the Restrict WebSockets pool setting](/media/images/docs/brave-setting.png)

5. 브라우저를 다시 시작합니다.