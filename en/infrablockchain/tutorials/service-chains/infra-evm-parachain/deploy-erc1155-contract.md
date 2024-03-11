---
title: Deploying an ERC1155 Token Contract
description: This tutorial explains how to deploy an ERC1155 token contract on the InfraEVM parachain.
keywords:
  - parachain
  - evm
---

## Before you begin

Before you begin, make sure to do the following:

- [**InfraEVM Parachain**](../../../service-chains/infra-evm-parachain.md)

- [**Moving Assets with EVM**](./deposit-and-withdraw-token.md)

## Deploying an ERC1155 Token Contract Using Remix

1. Go to [Remix](https://remix.ethereum.org).

    ![erc-1155remix-main](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc1155-remix-main.png)

2. Create a workspace. Choose the ERC1155 token contract template.

    ![erc1155-create-workspace](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc1155-create-workspace.png)
    ![erc1155-choose-template](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc1155-choose-template.png)

3. Verify the created workspace.

    ![erc1155-check-code](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc1155-check-code.png)

4. Compile the generated code.

    ![erc1155-compile-code](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc1155-compile-code.png)

5. Choose the network for deployment as `Injected Provider - MetaMask` and connect MetaMask to Remix.

    ![erc1155-inject-provider](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc1155-inject-provider.png)

6. Before deploying, set variable values like the owner and deploy the contract.

    ![erc1155-deploy](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc1155-deploy.png)

7. If a confirmation window appears in MetaMask, double-check and proceed with contract deployment.

    ![erc1155-deploy-check](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc1155-deploy-check.png)