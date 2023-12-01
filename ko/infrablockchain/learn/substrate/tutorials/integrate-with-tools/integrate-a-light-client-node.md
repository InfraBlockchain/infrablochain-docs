---
title: 경량 클라이언트 노드 통합
description: 브라우저에서 Substrate Connect를 사용하여 Substrate 기반 블록체인에 연결하는 방법을 보여줍니다.
keywords:
---

Substrate Connect에서 경량 클라이언트 노드를 배웠듯이, 경량 클라이언트 노드는 하드웨어 및 소프트웨어 요구 사항이 최소한인 상태에서 블록체인 데이터에 안전하고 탈중앙화된 액세스를 제공합니다.

이 튜토리얼에서는 경량 클라이언트를 사용하여 어떤 Substrate 기반 블록체인에 연결할 수 있는 방법을 보여줍니다.
이를 설명하기 위해 [Statemint 파라체인](https://wiki.polkadot.network/docs/learn-statemint)에 애플리케이션을 연결하는 방법을 배우게 될 것입니다.
Statemint는 Polkadot에 연결되어 있으며 공개적으로 액세스 가능한 체인 사양 파일을 가지고 있는 시스템 파라체인입니다.

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- Javascript 및 Typescript로 작업하기 위해 코드 편집기를 설정했습니다.
- [`yarn` 패키지 관리자](https://classic.yarnpkg.com/lang/en/docs/install/)가 설치되어 있습니다.

## 튜토리얼 목표

이 튜토리얼을 완료함으로써 다음 목표를 달성할 수 있습니다:

- Substrate Connect Javascript 라이브러리를 사용하여 Polkadot 릴레이 체인에 연결합니다.
- Substrate Connect에서 사용할 사용자 정의 체인 사양 파일을 지정하는 방법을 배웁니다.
- 사용자 정의 체인 사양에 연결된 파라체인에 연결합니다.

## 잘 알려진 체인에 연결하기

경량 클라이언트가 네트워크에 연결하려면 경량 클라이언트가 연결할 네트워크, 통신할 노드 및 제네시스에서 필요한 합의 중요 상태를 지정하는 웹 애플리케이션이 있어야 합니다.
이 정보는 네트워크의 [체인 사양](../../build/chain-spec.md) 파일에서 사용할 수 있습니다.

Substrate Connect는 [WellKnownChain](https://paritytech.github.io/substrate-connect/api/enums/connect_src.WellKnownChain.html) 열거 목록에 정의된 여러 체인을 인식하도록 사전 구성되어 있습니다.
이러한 잘 알려진 체인은 다음과 같습니다:

- `polkadot`으로 식별된 Polkadot
- `ksmcc3`으로 식별된 Kusama
- `rococo_v2_2`로 식별된 Rococo
- `westend2`로 식별된 Westend

이러한 체인 중 하나에 연결하려면:

1. 컴퓨터에서 새로운 터미널 쉘을 엽니다.

2. 다음 명령을 사용하여 `empty-webapp` 템플릿을 복제하여 Substrate Connect를 사용하는 웹 애플리케이션을 만듭니다.

   ```bash
   git clone https://github.com/bernardoaraujor/empty-webapp
   ```

3. 다음 명령을 실행하여 `empty-webapp` 디렉토리로 이동합니다.

   ```bash
   cd empty-webapp
   ```

4. 다음 명령을 실행하여 Substrate Connect를 설치합니다.

   ```bash
   yarn add @substrate/connect
   ```

5. 다음 명령을 실행하여 Polkadot-JS RPC 공급자에서 종속성을 설치합니다.

   ```bash
   yarn add @polkadot/rpc-provider
   ```

6. 다음 명령을 실행하여 Polkadot-JS API에서 종속성을 설치합니다.

   ```bash
   yarn add @polkadot/api
   ```

   이러한 종속성을 설치한 후에는 샘플 애플리케이션에서 사용할 수 있습니다.

7. 선호하는 코드 편집기에서 `empty-webapp/index.ts` 파일을 엽니다.
8. 다음 애플리케이션 코드를 복사하여 `substrate-connect`를 공급자로 사용하는 Substrate Connect 인스턴스를 만듭니다. 이 인스턴스는 `polkadot` 체인 사양 파일을 사용하여 Polkadot 릴레이 체인에 연결합니다.

   ```typescript
   import { ScProvider } from "@polkadot/rpc-provider/substrate-connect";
   import * as Sc from "@substrate/connect";
   import { ApiPromise } from "@polkadot/api";

   window.onload = () => {
     void (async () => {
       try {
         const provider = new ScProvider(Sc, Sc.WellKnownChain.polkadot);

         await provider.connect();
         const api = await ApiPromise.create({ provider });
         await api.rpc.chain.subscribeNewHeads((lastHeader: { number: unknown; hash: unknown }) => {
           console.log(`💡 새로운 블록 #${lastHeader.number}의 해시는 ⚡️ ${lastHeader.hash}입니다.`);
         });
       } catch (error) {
         console.error(<Error>error);
       }
     })();
   };
   ```

   이 샘플 애플리케이션 코드에서 Substrate Connect 인스턴스를 생성하는 방법은 Polkadot-JS API를 사용하여 WebSocket 인스턴스를 생성하는 방법과 유사합니다.
   Polkadot-JS API만 사용하는 경우 다음과 같이 인스턴스를 생성합니다:

   ```javascript
   // Import
   import { ApiPromise, WsProvider } from "@polkadot/api";

   // Construct
   const wsProvider = new WsProvider("wss://rpc.polkadot.io");
   const api = await ApiPromise.create({ provider: wsProvider });
   ```

   Substrate Connect의 경우 WebSocket (`WsProvider`) 공급자를 Substrate Connect (`ScProvider`) 공급자로 대체하고 WebSocket URL 클라이언트 주소 대신 네트워크에 연결할 체인 사양 (`WellKnownChain.polkadot`)을 지정합니다.

9. 다음 명령을 실행하여 남은 종속성을 설치합니다.

   ```bash
   yarn
   ```

10. 다음 명령을 실행하여 웹 애플리케이션을 시작합니다.

```bash
yarn dev
```

로컬 서버를 시작할 때 컴파일러 오류가 발생하면 현재 `yarn` 구성에서 고려되지 않은 종속성이 누락된 것일 수 있습니다.
종속성이 누락된 경우 다음과 유사한 명령을 실행하여 패키지를 추가할 수 있습니다.

```bash
yarn add -D buffer
```

11. 브라우저가 `http://localhost:3001/` URL을 열었는지 확인합니다.

12. 브라우저 콘솔을 엽니다.

사용하는 브라우저 및 운영 체제에 따라 브라우저 콘솔로 이동하고 열기하는 방법이 다릅니다.
예를 들어, Chrome에서는 **기타 도구**, **개발자 도구**를 선택한 다음 **콘솔**을 클릭합니다.

13. `smoldot` 프로세스가 초기화되고 Polkadot에서 수신된 블록의 해시가 표시되는지 확인합니다.

예를 들어, 콘솔에 다음과 유사한 로그 항목이 표시되어야 합니다:

```console
[substrate-connect-extension] [smoldot] Smoldot v0.7.7
[substrate-connect-extension] [smoldot] polkadot에 대한 체인 초기화 완료. 이름: "Polkadot". 제네시스 해시: 0x91b1…90c3. 상태 루트 해시: 0x29d0d972cd27cbc511e9589fcb7a4506d5eb6a9e8df205f00472e5ab354a4e17. 네트워크 식별자: 12D3KooWRse9u6Z9ukP4C92YCCH2gXziNm8ThRch2owaaFh9H6D1. 체인 사양 또는 데이터베이스 시작 위치: 0x7f52…8902 (#14614672)
...
💡 새로운 블록 #14614893의 해시는 ⚡️ 0x18f8086952aa5f8f1f8a36ea05af462f6bb26615b481145f7c5daa24ebc0c4cd입니다.
💡 새로운 블록 #14614894의 해시는 ⚡️ 0x92ca6fd51bc7a2fc5991441e9736bcccf3be45cee6fc88d40d145fc4211ba477입니다.
💡 새로운 블록 #14614894의 해시는 ⚡️ 0x2353ce49f06206c6dd9882200666fa7d51fc43c1cc6a61cca81ce9fa543409cb입니다.
```

이 간단한 웹 애플리케이션은 블록 해시를 검색하기 위해 Polkadot에만 연결합니다.
이 애플리케이션의 주요 목적은 네트워크의 특정 RPC 노드의 URL과 같은 중앙 집중식 진입점을 사용하지 않고 체인에 연결하는 것을 보여주는 것입니다.
하지만 `WsProvider`를 `ScProvider`로 대체한 후에는 기존의 [Polkadot-JS API](https://polkadot.js.org/docs/)를 사용하여 애플리케이션에 대한 코드를 작성할 수 있으므로 이 애플리케이션을 확장하여 더 많은 작업을 수행할 수 있습니다.

14. Control-c를 눌러 `smoldot` 경량 클라이언트 노드를 중지합니다.

## 사용자 정의 체인 사양에 연결하기

사용자 정의 체인 사양 또는 공개적으로 액세스 가능한 파라체인에 연결하는 방법은 잘 알려진 체인에 연결하는 방법과 유사합니다.
코드의 주요 차이점은 Substrate Connect에서 사용할 체인 사양을 명시적으로 식별해야 한다는 것입니다.

Statemint에 연결하려면 다음을 수행합니다:

1. [cumulus 저장소](https://github.com/paritytech/cumulus/blob/445f9277ab55b4d930ced4fbbb38d27c617c6658/parachains/chain-specs/statemint.json)에서 사용자 정의 체인 사양 파일을 다운로드합니다.

2. [잘 알려진 체인에 연결하기](#connect-to-a-well-known-chain)에서 생성한 `empty-webapp` 디렉토리로 다운로드한 체인 사양을 복사합니다.

3. 코드 편집기에서 `index.ts` 파일을 엽니다.
4. 다음 코드로 내용을 대체합니다.

   ```typescript
   import { ApiPromise } from "@polkadot/api";
   import { ScProvider } from "@polkadot/rpc-provider/substrate-connect";
   import * as Sc from "@substrate/connect";
   import jsonParachainSpec from "./statemint.json";

   window.onload = () => {
     void (async () => {
       try {
         const relayProvider = new ScProvider(Sc, Sc.WellKnownChain.polkadot);
         const parachainSpec = JSON.stringify(jsonParachainSpec);
         const provider = new ScProvider(Sc, parachainSpec, relayProvider);

         await provider.connect();
         const api = await ApiPromise.create({ provider });
         await api.rpc.chain.subscribeNewHeads((lastHeader: { number: unknown; hash: unknown }) => {
           console.log(`💡 새로운 블록 #${lastHeader.number}의 해시는 ⚡️ ${lastHeader.hash}입니다.`);
         });
       } catch (error) {
         console.error(<Error>error);
       }
     })();
   };
   ```

   이 코드에는 몇 가지 중요한 차이점이 있습니다.

   - `statemint.json` 체인 사양 파일이 `jsonParachainSpec` 객체로 가져와집니다.
   - 체인 사양은 JSON 인코딩된 문자열로 변환되어 `parachainSpec` 변수에 저장되어 웹 서버와 교환할 수 있도록 합니다.
   - `ScProvider` 공급자는 `polkadot` 릴레이 체인을 위해 생성되지만 파라체인 공급자를 생성하고 연결하는 데 사용됩니다.

Substrate Connect는 이 정보를 사용하여 파라체인이 통신하는 릴레이 체인을 결정합니다.

6. 다음 명령을 실행하여 웹 애플리케이션을 시작합니다.

   ```bash
   yarn dev
   ```

7. 브라우저가 `http://localhost:3001/` URL을 열었는지 확인합니다.

8. 브라우저 콘솔을 엽니다.

9. `smoldot` 프로세스가 초기화되고 Polkadot에서 수신된 블록의 해시가 표시되는지 확인합니다.

   예를 들어, 콘솔에 다음과 유사한 로그 항목이 표시되어야 합니다:

   ```console
   [substrate-connect-extension] [smoldot] statemint에 대한 파라체인 초기화 완료. 이름: "Statemint". 제네시스 해시: 0x68d5…de2f. 상태 루트 해시: 0xc1ef26b567de07159e4ecd415fbbb0340c56a09c4d72c82516d0f3bc2b782c80. 네트워크 식별자: 12D3KooWNicu1ZCX6ZNUC96B4TQSiet2NkoQc7SfitxWWE4fQgpK. 릴레이 체인: polkadot (id: 1000)
   ...
   💡 새로운 블록 #3393027의 해시는 ⚡️ 0x2401313496be4b1792d704f83b684be6bd2188a618581d30b3addb3648c4ad3a입니다.
   [substrate-connect-extension] [smoldot] Smoldot v0.7.7. 현재 메모리 사용량: 222 MiB. 평균 다운로드: 63.1 kiB/s. 평균 업로드: 735 B/s.
   💡 새로운 블록 #3393028의 해시는 ⚡️ 0x512af8ad5577f509f3f5123916ff2da6ca0f86df8099eafbc0bc001febec62dd입니다.
   ```

축하합니다! 브라우저에서 경량 클라이언트를 사용하여 Statemint와 Polkadot에 연결하는 간단한 웹 애플리케이션을 만들었습니다.
동일한 애플리케이션에서 더 많은 체인에 대한 연결을 추가하는 방법을 알아보려면 [이 데모](https://github.com/paritytech/substrate-connect/tree/main/projects/demo)를 확인하세요.

<!--
## 고급 애플리케이션 개발

이 튜토리얼의 예제는 [Polkadot-JS API](https://polkadot.js.org/docs/)를 사용하여 체인과 상호 작용하는 애플리케이션을 만드는 데 유용한 `@polkadot/rpc-provider/substrate-connect`를 사용했습니다.
Polkadot-JS API에 의존하지 않는 더 고급 애플리케이션 개발을 위해 `@substrate-connect` 패키지를 설치하고 사용할 수 있습니다.
예를 들어, 직접 애플리케이션 라이브러리나 프로그래밍 인터페이스를 구축하는 경우 다음 명령을 실행하여 Substrate Connect 종속성을 추가해야 합니다.

```bash
yarn add @substrate/connect
```

`@substrate-connect`를 사용하려면 다음을 수행합니다.

1. 터미널 쉘을 열고 새로운 작업 디렉토리를 만듭니다.

   예를 들어:

   ```bash
   mkdir adv-app && cd adv-app
   ```

2. `empty-webapp` 템플릿을 복제하고 템플릿 디렉토리로 이동합니다.

   ```bash
   git clone https://github.com/bernardoaraujor/empty-webapp && cd empty-webapp
   ```

3. 다음 명령을 실행하여 Substrate Connect 종속성을 설치합니다.

   ```bash
   yarn add @substrate/connect
   ```

4. 다음 애플리케이션 코드를 복사하여 붙여넣습니다.

   ```typescript
   import { WellKnownChain, createScClient } from '@substrate/connect';
   // Create the client
   const client = createScClient();
   // Create the chain connection, while passing the `jsonRpcCallback` function.
   const chain = await client.addWellKnownChain(
    WellKnownChain.polkadot,
    function jsonRpcCallback(response) {
      console.log('response', response);
    }
   );
   // send a RpcRequest
   chain.sendJsonRpc(
    '{"jsonrpc":"2.0","id":"1","method":"system_health","params":[]}'
   );
   ```
-->