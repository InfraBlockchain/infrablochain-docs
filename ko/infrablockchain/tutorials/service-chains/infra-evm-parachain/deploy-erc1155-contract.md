---
title: ERC1155 토큰 컨트랙트 배포하기
description: 이 튜토리얼은 InfraEVM 체인에서 ERC1155 토큰 컨트랙트를 배포하는 방법에 대해서 설명합니다.
keywords:
  - 파라체인
  - EVM
---

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [*InfraEVM* 파라체인](../../../service-chains/infra-evm-parachain.md)

- [EVM으로 자산 이동하기](./deposit-and-withdraw-token.md)

## Remix를 사용하여 ERC1155 토큰 컨트랙트 배포하기

1. [Remix](https://remix.ethereum.org)에 접속합니다. 

    ![erc-1155remix-main](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc1155-remix-main.png)

2. workspace를 생성해줍니다. erc1155 토큰 컨트랙트 템플릿을 선택하여 생성해줍니다.

    ![erc1155-create-workspace](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc1155-create-workspace.png)
    ![erc1155-choose-template](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc1155-choose-template.png)

3. 생성한 workspace를 확인합니다.

    ![erc1155-check-code](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc1155-check-code.png)

4. 생성된 코드를 컴파일 합니다.

    ![erc1155-compile-code](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc1155-compile-code.png)

5. 배포할 네트워크를 `Injected Provider - MetaMask` 를 선택해 준 뒤 메타마스크와 리믹스를 연결합니다.

    ![erc1155-inject-provider](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc1155-inject-provider.png)

6. 배포 전 owner와 같은 변수값들을 설정하고 컨트랙트를 배포합니다.

    ![erc1155-deploy](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc1155-deploy.png)

7. 메타마스크에서 확인 창이 나타나면 다시 한번 확인한 뒤 컨트랙트 배포를 진행합니다.

    ![erc1155-deploy-check](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc1155-deploy-check.png)