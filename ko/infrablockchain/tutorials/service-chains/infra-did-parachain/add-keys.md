---
title: 인프라DID에 공개키 추가하기
description: 이 튜토리얼은 인프라블록체인 파라체인 중 하나인 인프라DID 체인에 등록된 DID 에 공개키를 추가하는 방법을 설명합니다.
keywords:
  - 파라체인
  - DID
---

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [**인프라DID 생성하기**](./create-infra-did.md)

## 인프라DID 에 공개키 추가하기

체인에 등록된 인프라DID는 여러가지 항목들을 관리할 수 있도록 되어있습니다. 그 중 하나는 공개키 목록입니다. 특정 DID가 사용할 수 있는 공개 키 목록을 체인에 저장하여 공개함으로써 다른 사용자들이 특정 DID의 Document를 확인하여 알 수 있도록 합니다.

인프라DID에 공개키를 추가하기 위해선 아래와 같은 과정을 거칩니다.

1. [인프라블록체인 익스플로러](https://portal.infrablockspace.net)에 접속하여 아래 과정을 따릅니다.

- `개발자` - `익스트린식` - `didModule` 팔레트의 `addKeys` 익스트린식을 선택합니다.

  아래와 같이 구성하고 익스트린식을 발생시킵니다.

  ![add-keys](/media/images/docs/infrablockchain/tutorials/service-chains/infra-did-parachain/add-keys.png)

  Encode call data:

  ```shell
  0x3404d43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d04012e7222343997d83b3571b176837092986630c3de8fcdc91ba74d31bbb11c1181000000000000d43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d0100000001f62b5a39b0fceeb51d16b2a189da0e73f15f384497dcdd63a1f9a7c626f7649941b4a8cb66fed18c6557417a2f1ed607591dda6f6ec3bdfff40a09a71e202803
  ```

## 인프라DID 공개키 제거하기

인프라DID의 공개키를 제거하기 위해선 아래와 같은 과정을 거칩니다.

1. [인프라블록체인(InfraBlockchain) 익스플로러](https://portal.infrablockspace.net) 에 접속하여 아래 과정을 따릅니다.

- `개발자` - `익스트린식` - `didModule` 팔레트의 `removeKeys` 익스트린식을 선택합니다.

  아래와 같이 구성하고 익스트린식을 발생시킵니다.

  ![remove-keys](/media/images/docs/infrablockchain/tutorials/service-chains/infra-did-parachain/remove-keys.png)

## `infra-did-js` 라이브러리를 사용하여 인프라DID 공개키 추가 및 제거하기

`infra-did-js` 라이브러리를 사용해서 인프라DID에 공개키를 추가하거나 제거하는 방법은 아래와 같습니다. 

1. [`infra-did-js`](https://github.com/InfraBlockchain/infra-did-js) 라이브러리를 설치합니다.

   ```shell
   yarn add infra-did-js
   ```

2. 아래와 같이 코드를 작성하여 인프라DID 체인에 접근하기 위한 기본 설정 코드를 작성합니다.

   ```typescript
   import  {InfraSS58, CRYPTO_INFO} from 'infra-did-js';

   const txfeePaterAccountKeyPair = await InfraSS58.getKeyPairFromUri('//Alice', 'sr25519');
   const confBlockchainNetwork = {
     networkId: 'space',
     address: 'ws://localhost:9944',
     // seed or keyPair required
     txfeePayerAccountKeyPair,
     // or txfeePayerAccountSeed: 'TX_FEE_PAYER_ACCOUNT_SEED'
   };
   const conf = {
     ...confBlockchainNetwork,
     did: 'did:infra:space:5CRV5zBdAhBALnXiBSWZWjca3rSREBg87GJ6UY9i2A7y1rCs',
     // seed or keyPair required
     seed: 'DID_SEED',
     // keyPair: keyPair,
     controllerDID: 'did:infra:space:5HdJprb8NhaJsGASLBKGQ1bkKkvaZDaK1FxTbJRXNShFuqgY'
     controllerSeed: 'DID_CONTROLLER_SEED',
     // or controllerKeyPair: controllerKeyPair
   };
   const infraApi = await InfraSS58.createAsync(conf);
   ```

3. 인프라DID에 공개키를 추가하기 위한 코드를 작성합니다.

   ```typescript
   // Add keys
   await infraApi.didModule.addKeys(SOME_DID_KEY);
   ```

4. 혹은 인프라DID의 공개키를 제거하기 위한 코드를 작성합니다.

   ```typescript
   // Remove Keys
   await infraApi.didModule.removeKeys(DID_KEY_IDS);
   ```

## 다음 단계로 넘어가기

- [인프라DID에 서비스 엔드포인트 등록하기](./add-services.md)
