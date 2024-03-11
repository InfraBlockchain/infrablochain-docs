---
title: Deploying an ERC20 Token Contract
description: This tutorial explains how to deploy an ERC20 token contract on the InfraEVM parachain.
keywords:
  - parachain
  - InfraEVM
---

## Before you begin

Before you begin, make sure to do the following:

- [**InfraEVM Parachain**](../../../service-chains/infra-evm-parachain.md)

- [**Moving Assets with EVM**](./deposit-and-withdraw-token.md)

## Deploying an ERC20 Token Contract Using Remix

1. Visit [Remix](https://remix.ethereum.org).

    ![remix-main](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/remix-main.png)

    In this document, we will proceed using the `0XPROJECT ERC20` template.

2. Compile the smart contract.

    ![remix-compile](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc20-remix-compile.png)

3. Select the network to deploy as `Injected Provider - MetaMask` and connect MetaMask to Remix.

    ![remix-inject-provider](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc20-remix-inject-provider.png)

4. Before deployment, set variable values such as symbol, decimals, and deploy the contract.

    ![deploy-contract](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc20-deploy-contract.png)

5. Verify that the ERC20 token is recognized correctly using the contract address in MetaMask.

    ![remix-inject-provider](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/erc20-token.png)

## Moving on to the Next Step

- [Deploying an ERC721 Token Contract](./deploy-erc721-contract.md)