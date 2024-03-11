---
title: Depositing and Withdrawing Funds on EVM
description: This tutorial explains how to deposit and withdraw funds on the InfraEVM parachain.
keywords:
  - parachain
  - evm
---

## Before you begin

Before you begin, make sure to do the following:

- [**InfraEVM**](../../../service-chains/infra-evm-parachain.md)

## Converting Addresses

InfraEVM Parachain uses two address systems:

- SS58 address
- H160 address

SS58 addresses are used in the Substrate layer, while H160 addresses are used in the EVM layer.

### SS58 Address to H160 Address

![ss58-to-h160](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/ss58-to-h160.png)

Converting an SS58 address to an H160 address is done as shown in the image above.

It involves decoding the SS58 address and using the first 20 bytes of the resulting 32-byte buffer as the H160 address.

A JavaScript file that implements this conversion is included in the [repository](https://github.com/InfraBlockchain/infra-evm-parachain).

Here's how to convert SS58 addresses to H160 addresses:

1. Open your computer's terminal shell.

2. Clone InfraEVM Parachain repository by running the following command:

   ```bash
   git clone https://github.com/InfraBlockchain/infra-evm-parachain
   ```

   This command clones the `master` branch.

3. Navigate to the root of the node template directory with the following command:

   ```bash
   cd infra-evm-substrate
   ```

   Create a new branch for your work:

   ```bash
   git switch -c my-learning-branch-yyyy-mm-dd
   ```

   Replace `yyyy-mm-dd` with your desired identification information. We recommend using a numeric year-month-day format. For example:

   ```bash
   git switch -c my-learning-branch-2023-03-01
   ```

4. Execute the following command to perform the address conversion:

   ```bash
   node ./utils --evm-address {ss58-address}
   ```

### H160 Address to SS58 Address

![h160-to-ss58](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/h160-to-ss58.png)

Converting an H160 address to an SS58 address is done as shown in the image above.

It involves combining a buffer composed of the string `"evm:"` and a buffer composed of the H160 address, then obtaining the hash value using the `blake2` hash function and encoding the resulting hash value in SS58 format to obtain the SS58 address.

A JavaScript file that implements this conversion is included in the [repository](https://github.com/InfraBlockchain/infra-evm-parachain).

Here's how to convert H160 addresses to SS58 addresses:

1. Open your computer's terminal shell.

2. Clone InfraEVM Parachain repository by running the following command:

   ```bash
   git clone https://github.com/InfraBlockchain/infra-evm-parachain
   ```

   This command clones the `master` branch.

3. Navigate to the root of the node template directory with the following command:

   ```bash
   cd infra-evm-substrate
   ```

   Create a new branch for your work:

   ```bash
   git switch -c my-learning-branch-yyyy-mm-dd
   ```

   Replace `yyyy-mm-dd` with your desired identification information. We recommend using a numeric year-month-day format. For example:

   ```bash
   git switch -c my-learning-branch-2023-03-01
   ```

4. Execute the following command to perform the address conversion:

   ```bash
   node ./utils --ss58-address {evm-address}
   ```

## Moving Assets from Substrate to EVM

1. Use the method explained above to obtain the SS58 address mapped to the H160 address to be used in EVM.

   ```bash
   node ./utils --ss58-address {evm-address}
   ```

2. Transfer assets to the obtained SS58 address.

   ![transfer-asset](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/transfer-asset.png)

   By default, InfraEVM Parachain is integrated to use asset 99 as the native token of the EVM.

3. Verify that it reflects in EVM wallets like MetaMask.

   ![metamask-balance](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/metamask-balance.png)

## Moving Assets from EVM to Substrate

1. Use the method explained above to obtain the EVM address mapped to the SS58 address to be used in Substrate.

   ```bash
   node ./utils --evm-address {ss58-address}
   ```

2. Transfer assets to the obtained EVM address.

   ![transfer-asset](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/metamask-transfer.png)

3. Access the [InfraBlockchain Explorer](https://portal.infrablockspace.net) and follow the steps below.

- Navigate to `Developers` - `Extrinsics` - `evm` palette and select the `withdraw` extrinsic.

  Configure it as shown below and dispatch the extrinsic:

  ![withdraw](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/withdraw.png)

  The SS58 account used for address conversion in step 1 should execute the extrinsic.

4. Check the events to confirm that the assets were successfully deposited.

   ![withdraw-success](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/withdraw-success.png)

## Next Steps

- [Deploying an ERC20 Token Contract](./deploy-erc20-contract.md)
