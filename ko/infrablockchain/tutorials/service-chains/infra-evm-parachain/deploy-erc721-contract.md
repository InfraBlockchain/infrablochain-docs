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

2. ERC721 토큰 컨트랙트 템플릿을 선택하여 workspace를 생성합니다. 

    ![erc721-create-workspace](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-create-workspace.png)

3. 생성한 workspace를 확인합니다.

    ![erc721-check-code](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-check-code.png)

4. 생성된 코드를 컴파일 합니다.

    ![erc721-compile](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-compile.png)

5. 배포할 네트워크를 `Injected Provider - MetaMask`로 선택해 준 뒤 MetaMask와 Remix를 연결합니다.

    ![erc721-change-network](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-change-network.png)

6. 배포하기 전에 소유자 등의 변수 값들을 설정하고 컨트랙트를 배포합니다.

    ![erc721-set-owner-and-deploy](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-set-owner-and-deploy.png)

7. MetaMask에서 확인 창이 나타나면, 다시 한번 확인한 뒤 컨트랙트 배포를 진행합니다.

    ![erc721-deploy-check](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-deploy-check.png)

8. 배포가 완료되면 MetaMask에서 아래와 같은 화면을 확인할 수 있습니다.

    ![erc721-deploy-complete](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-deploy-complete.png)

## 배포된 ERC721 토큰 컨트랙트로 NFT 발행하기

위 튜토리얼에서 계속 이어집니다.

1. 아래 이미지 `Deployed Contracts` 영역에, `safeMint` 함수가 있는 것을 확인할 수 있습니다. 이 함수를 이용하여 NFT를 발행할 수 있습니다. 
'to', 'tokenId' 영역에 적절한 값을 채워줍니다.

    ![erc721-mint](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-mint.png)

2. `transact` 버튼을 누르면, 트랜잭션에 대해 컨펌할 수 있는 창이 나타납니다.

    ![erc721-mint-check](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-mint-check.png)

3. 트랜잭션이 정상적으로 실행되면, NFT가 발행됩니다. MetaMask와 같은 월렛 내 NFT토큰 보기를 통해 확인이 가능합니다.

    ![erc721-metamask-nft-check](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-metamask-nft-check.png)

## 다음 단계로 넘어가기

- [ERC1155 토큰 컨트랙트 배포하기](./deploy-erc1155-contract.md)