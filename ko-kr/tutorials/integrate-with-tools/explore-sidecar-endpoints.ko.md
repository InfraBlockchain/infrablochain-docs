---
title: 사이드카 엔드포인트 탐색하기
description: Postman을 사용하여 Substrate REST 서비스 (사이드카) API를 탐색하는 방법을 보여줍니다.
keywords:
---

Substrate [사이드카](https://github.com/paritytech/substrate-api-sidecar) 서비스는 FRAME을 사용하여 구축된 Substrate 블록체인 노드와 상호작용하기 위한 REST API를 제공합니다.
사이드카 REST 서비스는 Substrate 기반 블록체인의 노드, 계정, 트랜잭션, 파라체인 등 다양한 구성 요소와 상호작용할 수 있는 많은 엔드포인트를 노출합니다.

이 튜토리얼에서는 `sidecar` 서비스를 탐색하기 위해 Postman을 사용합니다.
Postman은 API를 검사, 구축 및 테스트하는 데 사용되는 인기있는 플랫폼입니다.
Postman은 브라우저 기반 또는 데스크톱 클라이언트를 사용하여 새로운 및 경험있는 API 개발자 모두가 협업하고 실험할 수 있도록 쉽게 사용할 수 있는 인터페이스를 제공합니다.

`sidecar`가 제공하는 기능을 RESTful 엔드포인트를 통해 탐색하기 위해 [미리 정의된 API 컬렉션](https://documenter.getpostman.com/view/24602305/2s8YsqWaj8#intro)을 사용합니다.
미리 정의된 API 컬렉션은 Postman을 사용하여 생성한 저장된 요청의 집합을 제공하며, 사용자 정의 및 재사용할 수 있습니다.
Postman으로 생성된 API 컬렉션을 사용하면 인라인 문서, 데이터에 액세스하기 위한 재사용 가능한 변수, 매개변수 형성 오류 감지 등과 같은 기능을 활용할 수 있습니다.

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [빠른 시작](/quick-start/)을 완료하고 Substrate 노드 템플릿을 로컬에 설치하고 실행했습니다.
- 로컬 컴퓨터에 Node.js 14 버전 이상이 설치되어 있어 `sidecar` 패키지를 다운로드하고 설치할 수 있습니다.
- [Postman](https://www.postman.com/) 계정을 가지고 있거나 생성할 수 있습니다.
- **Postman for Web** 또는 **Postman for Mac** 클라이언트에 액세스하거나 설치할 수 있습니다.

## 튜토리얼 목표

이 튜토리얼을 완료함으로써 다음을 포함하여 Postman을 사용하여 `sidecar` API REST 서비스를 탐색하는 방법을 배우게 됩니다.

- `sidecar` 서비스를 다운로드하고 설치합니다.
- Postman API 컬렉션을 가져옵니다.
- 작업 환경을 설정합니다.
- `sidecar` API에 대한 API 요청을 수행합니다.
- 데이터를 저장하여 나중에 사용합니다.

## 사이드카 다운로드 및 설치

`sidecar`를 다운로드하고 설치하려면 다음을 수행하세요:

1. 터미널 쉘을 엽니다.
2. `npm` 또는 `yarn`을 사용하여 `@substrate/api-sidecar` 서비스를 전역 또는 로컬로 설치합니다.

   예를 들어, `yarn`을 사용하는 경우 다음 명령을 실행하여 `@substrate/api-sidecar` 서비스를 전역으로 설치합니다:

   ```bash
   yarn global add @substrate/api-sidecar
   ```

   패키지 관리자로 `npm`을 사용하는 경우 다음 명령을 실행하여 `@substrate/api-sidecar` 서비스를 전역으로 설치합니다:

   ```bash
   npm install -g @substrate/api-sidecar
   ```

1. 서비스가 연결할 Substrate 노드가 실행 중인지 확인합니다.

   기본적으로 서비스는 ws://127.0.0.1:9944를 사용하여 로컬 호스트에 연결을 시도합니다.
   SAS_SUBSTRATE_URL 환경 설정을 수정하여 서비스가 다른 URL을 사용하도록 구성할 수 있습니다.
   로컬에서 기본 포트를 사용하여 로컬에서 실행 중인 노드 템플릿에 연결하려는 경우 구성이 필요하지 않습니다.

2. 다음 명령을 실행하여 서비스를 시작합니다:

   ```bash
   substrate-api-sidecar
   ```

   기본 구성을 사용하는 경우 다음과 유사한 출력이 표시됩니다:

   ```text
   SAS:
     📦 LOG:
        ✅ LEVEL: "info"
        ✅ JSON: false
        ✅ FILTER_RPC: false
        ✅ STRIP_ANSI: false
     📦 SUBSTRATE:
        ✅ URL: "ws://127.0.0.1:9944"
        ✅ TYPES_BUNDLE: undefined
        ✅ TYPES_CHAIN: undefined
        ✅ TYPES_SPEC: undefined
        ✅ TYPES: undefined
     📦 EXPRESS:
        ✅ BIND_HOST: "127.0.0.1"
        ✅ PORT: 8080
   2023-01-04 14:35:41 info: Version: 14.2.2
   2023-01-04 14:35:41 warn: API/INIT: RPC methods not decorated: transaction_unstable_submitAndWatch, transaction_unstable_unwatch
   2023-01-04 14:35:41 info: Connected to chain Development on the node-template client at ws://127.0.0.1:9944
   2023-01-04 14:35:41 info: Listening on http://127.0.0.1:8080/
   2023-01-04 14:35:41 info: Check the root endpoint (http://127.0.0.1:8080/) to see the available endpoints for the current node
   ```

## API 컬렉션 가져오기

`sidecar`를 위한 미리 정의된 API 컬렉션을 사용하려면 다음을 수행하세요:

1. 브라우저에서 미리 정의된 [Substrate API Sidecar](https://documenter.getpostman.com/view/24602305/2s8YsqWaj8#intro) API 컬렉션을 엽니다.
2. 페이지의 오른쪽 상단에 있는 **Postman에서 실행**을 클릭합니다.
3. Postman for Web 또는 Postman for Mac 데스크톱 클라이언트 중 하나를 사용하여 컬렉션을 실행할지 선택합니다.

   macOS 컴퓨터를 사용하는 경우 데스크톱 클라이언트를 사용하는 것이 좋습니다. 데스크톱 클라이언트는 일반적으로 더 안정적이며 더 많은 기능을 지원합니다.
   로컬 컴퓨터에 Postman for Mac이 설치되어 있지 않은 경우 **앱 다운로드**를 클릭하여 다운로드할 수 있습니다.

   ![Postman에서 API 컬렉션 실행](/media/images/docs/tutorials/postman-sidecar/run-in-postman.png)

1. Postman에서 워크스페이스를 선택한 다음 **가져오기**를 클릭하여 미리 정의된 컬렉션을 Postman 워크스페이스에 추가합니다.

   Substrate API `sidecar` 컬렉션을 Postman에서 열면 환경 변수를 정의하기 시작할 준비가 되었습니다.

## 환경 변수 정의하기

미리 정의된 Postman API 컬렉션에는 `Dev`라는 내장 개발 환경이 포함되어 있습니다.
이 환경에는 미리 정의된 변수가 포함되어 있으며, 이 변수들은 미리 정의된 쿼리를 사용하여 `sidecar` 서비스에 대한 다양한 API 요청에서 사용할 수 있습니다.
`Dev` 환경에 정의된 변수는 다음과 같습니다:

- `url`: 사용할 `sidecar` REST API의 URL을 지정합니다.
  기본값은 `http://127.0.0.1:8080`입니다.
- `account`: 블록체인의 특정 사용자의 계정 식별자를 지정합니다.
- `number`: 블록체인의 특정 블록 번호를 지정합니다.
- `extrensicIndex`: 특정 블록의 특정 extrinsic의 인덱스 번호를 지정합니다.
- `assetId`: 팔렛 자산의 식별자를 지정합니다.
- `storageItemId`: 팔렛 저장소 항목의 식별자를 지정합니다.
- `paraId`: 특정 파라체인의 고유 숫자 식별자를 지정합니다.

모든 요청은 `url` 변수를 필요로 합니다.
`Dev` 환경은 새로운 `sidecar` 서비스 인스턴스를 시작할 때 생성되는 기본 REST API 주소인 `http://127.0.0.1:8080`을 기본값으로 제공합니다.
`sidecar` 서비스를 외부 호스팅 위치에서 사용하거나 환경 설정을 사용하여 기본 로컬 URL을 변경한 경우 `url` 변수의 기본값을 해당 URL에 맞게 변경해야 합니다.
`url` 변수 외에도 다른 유형의 요청에는 다른 변수를 정의해야 할 수 있습니다.

예를 들어, 이 튜토리얼에서는 블록체인의 특정 계정의 잔액을 반환하는 쿼리를 실행하기 위해 `account` 변수에 값을 정의해야 합니다.

## 엔드포인트 목록 가져오기

Postman을 사용하여 `sidecar`에 `GET` 요청을 보내 API 컬렉션에 사용 가능한 모든 활성 엔드포인트 목록을 반환할 수 있습니다.
이 요청은 `url` 변수만 정의하면 됩니다.
대부분의 경우 로컬 개발 환경 외에 `sidecar`를 사용하지 않는 한 로컬 호스트 IP 주소와 포트의 기본값을 사용할 수 있습니다.

엔드포인트 목록을 가져오려면 다음을 수행하세요:

1. Postman 클라이언트에서 **컬렉션**을 클릭하고 **Substrate API Sidecar** 컬렉션을 엽니다.
2. 목록에서 **GET List of API Endpoints** 요청을 선택합니다.

   ![GET Endpoints 요청 선택](/media/images/docs/tutorials/postman-sidecar/select-get-endpoints.png)

   **Get the List of API Endpoints** 요청을 선택하면 Postman에서 이 요청에 대한 사용 가능한 옵션을 표시합니다.
   이 요청은 URL만 필요로 하며 다른 매개변수를 사용하지 않으므로 여기에서 구성할 설정이 없습니다.

   ![요청에 대한 매개변수 설정](/media/images/docs/tutorials/postman-sidecar/endpoint-parameters.png)

   요청이 추가 매개변수를 사용하는 경우 매개변수에 대해 다른 값을 제공하여 해당 값이 API에서 다른 응답을 제공하는지 확인할 수 있습니다.

3. 미리 정의된 `url` 변수를 사용하여 연결할 URL을 지정합니다. `{{url}}`을 입력합니다.

   ![요청에 대한 URL 지정](/media/images/docs/tutorials/postman-sidecar/specify-url.png)

4. **Send**를 클릭합니다.

   요청을 보낸 후에는 Postman의 응답 섹션이 데이터로 채워집니다.
기본적으로 데이터는 JSON 형식으로 표시되지만 응답을 XML, HTML 또는 일반 텍스트로 변경할 수 있습니다.
다음과 유사한 부분적인 응답을 볼 수 있어야 합니다:

   ```json
   {
     "docs": "https://paritytech.github.io/substrate-api-sidecar/dist",
     "github": "https://github.com/paritytech/substrate-api-sidecar",
     "version": "14.2.2",
     "listen": "127.0.0.1:8080",
     "routes": [
        {
            "path": "/accounts/:address/asset-balances",
            "method": "get"
        },
        {
            "path": "/accounts/:address/asset-approvals",
            "method": "get"
        },
      ]
   }
   ```

5. 이 응답을 **Save response**를 클릭하여 출력 파일로 저장하면 나중에 사용하여 자체 애플리케이션의 기능을 구축하는 데 도움이 됩니다.

   응답을 예제로 저장할 수도 있습니다.
   응답을 예제로 저장하면 GET List of API Endpoints 요청 옵션 아래에 추가됩니다.

   이 엔드포인트 목록 요청은 간단한 요청의 유용한 데모입니다.
   그러나 더 자주 사용하는 경우 사이드카 엔드포인트를 사용하여 특정 데이터에 액세스하려면 다른 변수에 정의된 값을 조작하여 원하는 특정 정보를 검색해야 합니다.

   이 튜토리얼의 다음 단계입니다.

## 계정 정보 가져오기

간단한 요청을 수행하는 방법을 알게 되었으므로, 이번에는 환경 변수 중 하나의 값을 수정하여 다른 `GET` 요청을 수행해 보겠습니다.
이 요청에서는 체인 상의 특정 계정의 잔액을 요청합니다.
작업 블록체인으로 [substrate node template](https://github.com/substrate-developer-hub/substrate-node-template)을 사용하여 이 요청은 `Alice` 사용자의 계정 잔액을 쿼리합니다.

Alice 주소의 계정 정보를 가져오려면 다음을 수행하세요:

1. 새로운 터미널 쉘을 엽니다.
2. Alice의 계정 주소를 복사합니다.

   예를 들어, 다음 명령을 실행하여 `//Alice` 계정의 키 정보를 검사합니다:

   ```bash
   ./target/release/node-template key inspect //Alice
   ```

   계정의 공개 키를 복사합니다:

   ```
   5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
   ```

3. Postman 클라이언트에서 **Dev** 환경을 선택하여 환경 변수와 해당 값의 목록을 표시합니다.

   각 변수에는 초기 값과 현재 값이 있습니다.

   - `초기 값`은 `Dev` 환경을 처음 시작할 때 변수에 로드되는 값입니다.
   - `현재 값`은 현재 세션의 값을 나타냅니다.

   `현재 값`은 로컬에만 저장되며 Postman에게 전송되거나 동일한 Postman API 컬렉션을 사용하는 다른 팀원과 공유되지 않습니다.

1. `Alice`의 계정 주소를 `현재 값` 필드에 붙여넣습니다.

   ![Postman 환경에서 계정 변수](/media/images/docs/tutorials/postman-sidecar/dev-variables.png)

1. **컬렉션**을 클릭하고 **Substrate API Sidecar** 컬렉션을 엽니다.

1. **Accounts** 폴더를 선택한 다음 **GET Account Balance Info**를 선택합니다.

   ![Account Balance Info 요청 선택](/media/images/docs/tutorials/postman-sidecar/account-balance.png)

2. 이전에 사용한 `url` 변수와 Step 4에서 설정한 `account` 변수를 사용하여 요청을 구성합니다:

   ![요청에 대한 엔드포인트 지정](/media/images/docs/tutorials/postman-sidecar/account-request.png)


3. **Send**를 클릭합니다.

   JSON 형식으로 다음과 유사한 응답을 받아야 합니다:

   ```json
   {
     "at": {
        "hash": "0xfc354903a665c7847ba4f83dd9e5fb0389e31bc2015086aca56a68bb345493a5",
        "height": "1189"
     },
     "nonce": "0",
     "tokenSymbol": "UNIT",
     "free": "1152921504606846976",
     "reserved": "0",
     "miscFrozen": "0",
     "feeFrozen": "0",
     "locks": []
   }
   ```

  이 응답을 파일로 저장하거나 Postman 내에서 예제로 저장하여 테스트를 생성하거나 자체 애플리케이션을 구축하는 데 도움이 됩니다.

## 다음 단계는?

이 튜토리얼에서는 다음을 배웠습니다:

- Postman에 API 컬렉션을 가져오는 방법.
- 컬렉션에서 환경 변수를 정의하는 방법.
- 사용자 정의 변수 정의를 사용하여 API 요청을 보내는 방법.
- API의 응답을 검사하고 저장하는 방법.

이제 Postman에서 `sidecar` 엔드포인트를 사용하는 데 좋은 기초를 갖추었으며, 자체 목적으로 `sidecar` API를 사용하는 애플리케이션을 구축하기 시작할 수 있습니다.
자체적으로 더 탐색하려면 다음 작업을 고려해 보세요:

- 미리 정의된 컬렉션에서 다른 엔드포인트를 사용하여 요청을 보냅니다.
- 디버깅 목적으로 Postman에서 테스트를 작성합니다.
- API REST 요청의 일련의 연결을 수행할 수 있는 Postman 플로우를 설정합니다.