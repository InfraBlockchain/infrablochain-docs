---
title: ERC20 토큰 컨트랙트 배포하기
description: 이 튜토리얼은 인프라EVM 체인에서 ERC20 토큰 컨트랙트를 배포하는 방법에 대해서 설명합니다.
keywords:
  - 파라체인
  - 인프라EVM
---

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [**인프라EVM 파라체인**](../../../service-chains/infra-evm-parachain.md)

- [**EVM으로 자산 이동하기**](./deposit-and-withdraw-token.md)

## Remix를 사용하여 ERC20 토큰 컨트랙트 배포하기

1. [Remix](https://remix.ethereum.org)에 접속합니다. 

    ![remix-main](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/remix-main.png)

    본 문서에서는 `0XPROJECT ERC20` 템플릿을 사용하여 진행합니다.

2. 스마트 컨트랙트에 대해 컴파일을 진행합니다.

    ![remix-compile](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc20-remix-compile.png)

3. 배포할 네트워크를 `Injected Provider - MetaMask` 로 선택하고, MetaMask와 Remix를 연결합니다.

    ![remix-inject-provider](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc20-remix-inject-provider.png)

4. 배포하기 전에 기호, 소수점 등의 변수 값들을 설정하고 컨트랙트를 배포합니다.

    ![deploy-contract](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc20-deploy-contract.png)

5. MetaMask에서 컨트랙트 주소를 사용하여 실제 ERC20 토큰이 정상적으로 인식되는지 확인합니다.

    ![remix-inject-provider](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc20-token.png)

## 다음 단계로 넘어가기

- [ERC721 토큰 컨트랙트 배포하기](./deploy-erc721-contract.md)