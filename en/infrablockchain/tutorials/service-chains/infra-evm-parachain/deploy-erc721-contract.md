---
title: Deploying an ERC721 Token Contract
description: This tutorial explains how to deploy an ERC721 token contract on the InfraBlockchain EVM parachain.
keywords:
  - parachain
  - evm
---

## Before you begin

Before you begin, make sure to do the following:

- [*InfraBlockchain EVM* Parachain](../../../service-chains/infra-evm-parachain.md)

- [Moving Assets with EVM](./deposit-and-withdraw-token.md)

## Deploying an ERC721 Token Contract Using Remix

1. Visit [Remix](https://remix.ethereum.org).

    ![remix-main](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-remix-main.png)

2. Create a workspace. Choose the ERC721 token contract template.

    ![erc721-create-workspace](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-create-workspace.png)

3. Confirm the created workspace.

    ![erc721-check-code](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-check-code.png)

4. Compile the generated code.

    ![erc721-compile](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-compile.png)

5. Select the network to deploy as `Injected Provider - MetaMask` and connect MetaMask to Remix.

    ![erc721-change-network](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-change-network.png)

6. Before deployment, set variable values such as the owner, and deploy the contract.

    ![erc721-set-owner-and-deploy](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-set-owner-and-deploy.png)

7. If a confirmation window appears in MetaMask, double-check and proceed with contract deployment.

    ![erc721-deploy-check](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-deploy-check.png)

8. Once deployment is complete, you can verify it in MetaMask as shown below.

    ![erc721-deploy-complete](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-deploy-complete.png)

## Minting NFTs with the Deployed ERC721 Token Contract

This continues from the previous tutorial.

1. As shown below in "Deployed Contracts," you can see the `safeMint` function. You can use this function to mint NFTs by filling in appropriate values for "to" and "tokenId."

    ![erc721-mint](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-mint.png)

2. Press the "transact" button, and a confirmation window for the transaction will appear.

    ![erc721-mint-check](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-mint-check.png)

3. If the transaction is executed successfully, the NFT will be minted, and you can confirm it in wallets like MetaMask under NFT token views.

    ![erc721-metamask-nft-check](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc721-metamask-nft-check.png)

## Moving on to the Next Step

- [Deploying an ERC1155 Token Contract](./deploy-erc1155-contract.md)