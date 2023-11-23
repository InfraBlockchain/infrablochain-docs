---
title: 인프라 DID
description: DID 에 특화된 블록체인에 대한 전반적인 내용을 다룹니다.
keywords:
  - 파라체인
  - DID
---

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

<!-- 
  해당 내용이 담긴 문서가 생성되면 그 문서로 링크를 연결 해 주세요
-->

- [인프라 파라체인 구축하기](/ko/infrablockchain/tutorials/build-infrablockspace/build-a-parachain.md)

- [인프라 릴레이 체인 구축하기](/ko/infrablockchain/tutorials/build-infrablockspace/build-infra-relay-chain.md)

- [좀비넷 사용하기](./infra-did-parachain.md)

## DID란

**DID**는 Decentralized Identifier의 약자로, 디지털 신원의 주체가 생성, 소유하고 제어하는 새로운 유형의 식별자를 나타냅니다.

![did-method](/media/images/docs/infrablockchain/service-chains/did-method.png)

DID는 중앙화된 시스템, 권한 또는 중개자와는 독립적으로 완전히 DID 주체의 제어 하에 있습니다. 이 기능은 블록체인 및 분산 원장 기술을 통해 가능하게 됩니다.

DID의 주요 특징 및 이점은 다음과 같습니다:

- 분산화: 중앙화된 시스템이 발행하고 관리하는 기존의 식별자와 달리, DID는 분산 네트워크에서 발행 및 관리됩니다.

- 자기 주권: DID 주체는 자신의 식별자에 대한 전체 제어권을 가지고 있습니다. 어떤 권한의 허락 없이 자신의 DID를 생성, 업데이트 또는 삭제할 수 있습니다.

- 보안: DID는 블록체인 또는 분산 원장 기술에 고착되므로 이러한 플랫폼의 암호학적 보안 속성을 상속받습니다. 또한 DID는 인증 및 통신을 안전하게 유지하기 위해 DPKI (Decentralized Public Key Infrastructure)와 함께 작동합니다.

- 상호 운용성: DID는 다양한 시스템과 네트워크 간에 상호 운용될 수 있도록 설계되었습니다. 즉, 하나의 네트워크에서 생성된 DID는 다른 네트워크에서 사용되고 인식될 수 있습니다.

## Infra DID

인프라 블록스페이스의 파라체인 중 하나인 *Infra DID 파라체인* 은 DID 시스템을 제공하고 있으며 *Infra DID 파라체인* 이 제공하는 DID 를 *Infra DID* 라고 합니다.

![infra-did-method](/media/images/docs/infrablockchain/service-chains/infra-did-method.png)

*Infra DID 파라체인* 은 기능에 따라 여러가지 팔렛을 포함하고 있습니다.

![infra-did-pallet](/media/images/docs/infrablockchain/service-chains/infra-did-pallet.png)


Infra DID 파라체인과 통신할 수 있는 npm 라이브러리인 [infra-did-js](https://github.com/InfraBlockchain/infra-did-js/tree/main)가 존재합니다. 이를 사용하여 node.js 기반의 시스템에서 Infra DID을 사용할 수 있습니다.

## 다음 단계로 넘어가기

- [Infra DID 체인 구축하기](/ko/infrablockchain/tutorials/service-chains/infra-did-parachain/build.md)

- [infra-did-js 레포지토리](https://github.com/InfraBlockchain/infra-did-js/tree/main)