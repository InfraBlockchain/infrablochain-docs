---
title: InfraDID 생성하기
description: InfraDID 체인에서 DID를 생성하는 방법에 대해 설명합니다.
keywords:
  - 파라체인
  - DID
---

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [인프라DID 파라체인](../../../service-chains/infra-did-parachain.md)

## 인프라DID 생성하기

인프라DID를 생성하기 위해 블록체인 자체와 통신할 필요는 없습니다.

![infra-did-method](/media/images/docs/infrablockchain/service-chains/infra-did-method.png)

인프라DID는 위 그림 속 `DID method-specific identifier` 부분을 SS58 address로 구성한 형식을 따릅니다. 즉 `did:infra:{networkID}:{SS58 address}` 형식으로 구성하면 개인의 DID가 됩니다. SS58 address에서 얻을 수 있는 개인키와 공개키는 기본적으로 쌍으로 사용할 수 있습니다.

만약 동일한 DID를 사용하여 여러가지 공개키를 등록하거나 그 외 다른 데이터들을 등록하여 사용하고 싶다면, 체인에 DID를 등록하는 과정을 거쳐야 합니다.

## 인프라DID 체인에 등록하기

인프라DID를 체인에 등록하기 위해선 아래와 같은 과정을 거칩니다.

1. [인프라블록체인 익스플로러](https://portal.infrablockspace.net)에 접속하여 아래 과정을 따릅니다.

- `개발자` - `익스트린식` - `didModule` 팔레트의 `newOnchain` 익스트린식을 선택합니다.

  아래와 같이 구성하고 익스트린식을 발생시킵니다.

  ![new-onchain](/media/images/docs/infrablockchain/tutorials/service-chains/infra-did-parachain/new-onchain.png)

2. 이벤트를 확인하여 정상적으로 DID가 생성되었는지 확인합니다.

   ![new-onchain-success](/media/images/docs/infrablockchain/tutorials/service-chains/infra-did-parachain/new-onchain-success.png)

3. 스토리지를 조회하여 체인에 정상적으로 DID가 등록되었는지 확인할 수 있습니다.

   ![new-onchain-storage](/media/images/docs/infrablockchain/tutorials/service-chains/infra-did-parachain/new-onchain-storage.png)

## `infra-did-js` 라이브러리를 사용하여 InfraDID 생성 및 등록하기

[인프라블록체인 익스플로러](https://portal.infrablockspace.net)을 사용하여 인프라DID를 생성할 수 있지만, `infra-did-js` Javascript 라이브러리를 사용해서 인프라DID를 생성할 수도 있습니다.

`infra-did-js` 라이브러리를 사용해서 인프라DID를 생성하기 위해선 아래와 같은 과정을 거칩니다.

1. [`infra-did-js`](https://github.com/InfraBlockchain/infra-did-js) 라이브러리를 설치합니다.

   ```shell
   yarn add infra-did-js
   ```

2. 아래와 같이 코드를 작성하여 InfraDID 체인에 접근하기 위한 기본 설정 코드를 작성합니다.

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

3. 인프라DID를 생성하는 코드를 작성합니다.

   ```typescript
   DIDSet = await InfraSS58.createNewSS58DIDSet(networkId);
   console.log({ DIDSet });
   ```

   `console.log` 출력 결과는 아래와 같습니다.

   ```json
       {
     DIDSet: {
       did: 'did:infra:space:5Cq2Za1Z4HJx5eTvxT5iFyXZLM1XTwVZSafQsEuK4ujNKJEF',
       didKey: DidKey_SS58 {
         publicKey: PublicKey_SS58 {
         value: '0x21cdc3dc94f8cccd889759fbc282f4272f89c8d974aea4d3051e8efa85e738b7',
         sigType: 'Ed25519'
       },
       verRels: VerificationRelationship { _value: 0 }
       },
       keyPair: {
         address: [Getter],
         addressRaw: [Getter],
         isLocked: [Getter],
         meta: [Getter],
         publicKey: [Getter],
         type: [Getter],
         // -- snip --
       },
         publicKey: PublicKey_SS58 {
         value: '0x21cdc3dc94f8cccd889759fbc282f4272f89c8d974aea4d3051e8efa85e738b7',
         sigType: 'Ed25519'
       },
       verRels: VerificationRelationship { _value: 0 },
       cryptoInfo: {
         CRYPTO_TYPE: 'ed25519',
         KEY_NAME: 'Ed25519VerificationKey2018',
         SIG_TYPE: 'Ed25519',
         SIG_NAME: 'Ed25519Signature2018',
         SIG_CLS: [class Ed25519Signature2018 extends CustomLinkedDataSignature],
         LDKeyClass: [class Ed25519VerificationKey2018]
       },
       seed: '0x8c9971953c5c82a51e3ab0ec9a16ced7054585081483e2489241b5b059f5f3cf',
       keyPairJWK: {
         publicJwk: {
           alg: 'EdDSA',
           kty: 'OKP',
           crv: 'Ed25519',
           x: 'Ic3D3JT4zM2Il1n7woL0Jy-JyNl0rqTTBR6O-oXnOLc'
         },
         privateJwk: {
           alg: 'EdDSA',
           kty: 'OKP',
           crv: 'Ed25519',
           x: 'Ic3D3JT4zM2Il1n7woL0Jy-JyNl0rqTTBR6O-oXnOLc',
           d: 'jJlxlTxcgqUeOrDsmhbO1wVFhQgUg-JIkkG1sFn1888'
         }
       }
     }
   }
   ```

4. 생성한 인프라DID를 체인에 등록하기 위한 코드를 작성합니다.

   ```typescript
   // Register DID
   await infraApi.didModule.registerOnchain();
   ```

## 다음 단계로 넘어가기

- [인프라DID에 공개키 추가하기](./add-keys.md)
