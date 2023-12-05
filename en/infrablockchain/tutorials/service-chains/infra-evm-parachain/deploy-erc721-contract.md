---
title: ERC721 토큰 컨트랙트 배포하기
description: 이 튜토리얼은 InfraEVM 체인에서 ERC721 토큰 컨트랙트를 배포하는 방법에 대해서 설명합니다.
keywords:
  - 파라체인
  - EVM
---

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [*InfraEVM* 파라체인](../../../service-chains/infra-evm-parachain.md)

- [EVM으로 자산 이동하기](./deposit-and-withdraw-token.md)

## Remix를 사용하여 ERC721 토큰 컨트랙트 배포하기

1. [Remix](https://remix.ethereum.org)에 접속합니다. 

    ![remix-main](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-remix-main.png)

2. workspace를 생성해줍니다. ERC721 토큰 컨트랙트 템플릿을 선택하여 생성해줍니다.

    ![erc721-create-workspace](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-create-workspace.png)

3. 생성한 workspace를 확인합니다.

    ![erc721-check-code](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-check-code.png)

4. 생성된 코드를 컴파일 합니다.

    ![erc721-compile](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-compile.png)

5. 배포할 네트워크를 `Injected Provider - MetaMask` 를 선택해 준 뒤 메타마스크와 리믹스를 연결합니다.

    ![erc721-change-network](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-change-network.png)

6. 배포 전 owner와 같은 변수값들을 설정하고 컨트랙트를 배포합니다.

    ![erc721-set-owner-and-deploy](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-set-owner-and-deploy.png)

7. 메타마스크에서 확인 창이 나타나면 다시 한번 확인한 뒤 컨트랙트 배포를 진행합니다.

    ![erc721-deploy-check](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-deploy-check.png)

8. 배포가 완료되면 메타마스크에서도 다음과 같이 확인할 수 있습니다.

    ![erc721-deploy-complete](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-deploy-complete.png)

## 배포된 ERC721 토큰 컨트랙트로 NFT 민팅하기

위 튜토리얼에 이어서 진행됩니다.

1. 사진과 같이 아래 `Deployed Contracts` 에서 `safeMint` 함수가 있는 것을 확인할 수 있습니다. 이 함수를 이용하여 NFT를 민팅할 수 있습니다. to, tokenId에 적절한 값을 채워줍니다.

    ![erc721-mint](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-mint.png)

2. `transact` 버튼을 눌러주면 트랜잭션에 대해 컨펌할 수 있는 창이 나타납니다.

    ![erc721-mint-check](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-mint-check.png)

3. 트랜잭션이 정상적으로 실행 된다면 실제 NFT가 민팅이 되고 메타마스크와 같은 NFT토큰 보기를 월렛에서도 확인이 가능합니다.

    ![erc721-metamask-nft-check](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-metamask-nft-check.png)

## 다음 단계로 넘어가기

- [ERC1155 토큰 컨트랙트 배포하기](./deploy-erc1155-contract.md)