---
title: XCM을 이용하여 원격으로 자산 전송하기
description: 릴레이 체인을 통해 파라체인으로 원격 전송을 실행하는 XCM 사용 방법을 보여줍니다.
keywords:
    - XCM
---

In [Open message passing channels](open-message-passing-channels), you saw how to open a two-way communication channel between chains by sending messages to the relay chain.
You can use a similar strategy to send messages that allow a local chain to manage an account on a remote chain.
In this tutorial, parachain B transfers assets into the sovereign account on the relay chain for parachain A.

The outcome for this tutorial is similar to using the `transfer` function from the balances pallet, except in this case the transfer is initiated by a parachain and demonstrates how the holding register is used when executing the `WithdrawAsset` and `DepositAsset`XCM instructions.

## Before you begin

Before you begin, verify the following:

-   You have set up a [parachain test network](../test/simulate-parachains) using Zombienet or a local relay chain using the `rococo-local` chain specification.
-   You have set up two local or virtual parachains for testing purposes.

    For the purposes of this tutorial, parachain A has the unique identifier 1000 and parachain B has the unique identifier 1001.

-   You have the Sudo pallet available for both local parachains to use.

-   You have opened the message passing channel to allow communication between parachain B and parachain A.

## Configure XCM instructions

To illustrate the interaction between the two chains, in the following example, parachain B sends XCM instructions to deposit assets into an account on parachain A.

1. Connect to the endpoint for parachain B (1001) using the [InfraBlockchain Portal](https://portal.infrablockspace.net/#/explorer/).

2. Click **Developer** and select **Extrinsics**.

3. Select **sudo**, then select **sudo(call)** to use the Sudo pallet to execute privileged transactions.

4. Select **ibsXcm**, then select **send(dest, message)**.
5. Specify the destination parameters to indicate the relative location for the message to be delivered.

    - The XCM version for specifying the location of the destination: V1
    - The relay chain is the destination for the message, so the parent location: 1
    - In the context of the parent, the interior setting: Here

6. Specify the XCM version for the message (V2).
7. Click **Add item** to construct the message to be executed.

## WithdrawAsset instruction

To move assets into the virtual holding register:

1. Select _WithdrawAsset_ as the first instruction for this message.

2. Click **Add item** to identify the on-chain assets to withdraw.

3. Select **Concrete** to use the location of the asset to identify the asset to be withdrawn.

4. Set **parents: 0** and **interior: Here** to withdraw assets from the parachain B sovereign account on the relay chain.

5. Select **Fungible** to identify the asset as a fungible asset.

6. Specify the total fungible assets to withdraw.

    For example, this tutorial uses 12000000000000.

    ![WithdrawAsset instruction sent from parachain B](/media/images/docs/infrablockchain/tutorials/build/transfer-withdraw-asset-instruction-ui.png)

## BuyExecution instruction

To pay for execution from assets deposited in the holding register:

1. Click **Add item** to select _BuyExecution_ as the second instruction for this message.

2. Select **Concrete** to use the location of the asset to identify the asset to be used to pay for executing XCM instructions.

3. Set **parents: 0** and **interior: Here** to use the assets withdrawn from the parachain B sovereign account on the relay chain.

4. Select **Fungible** to identify the asset as a fungible asset.

5. Specify the total fungible assets to use.

    For example, this tutorial uses 12000000000000.

6. Select **Unlimited** to skip setting a weight limit for this instruction.

    ![BuyExecution instruction sent from parachain B](/media/images/docs/infrablockchain/tutorials/build/transfer-buy-execution-instruction-ui.png)

## DepositAsset instruction

To deposit assets after fees from the holding register into a specific account:

1. Click **Add item** to select _DepositAsset_ as the third instruction for this message.

1. Select **Wild** to allow an unspecified number of assets to be deposited.

1. Select **All** to allow all of the remaining assets after fees are paid to be deposited.

1. Set **1** as the maximum number of unique assets to remove from the holding register for the deposit.

    In this tutorial, there's only one asset instance available to be removed.

1. Specify the beneficiary to receive the deposited assets.

    You can deposit the assets remaining into the sovereign account for parachain A or into a specific account.
    For this tutorial, the assets are deposited using a specified account address for the previously unfunded account KRIS-PUBS.
    To select this beneficiary, the DepositAsset instruction looks like this:

    ![Specify an account as a beneficiary](/media/images/docs/infrablockchain/tutorials/build/transfer-withdraw-asset-instruction-ui.png)

    If you want to deposit the assets into the sovereign account for parachain A, you could specify the beneficiary using the following settings:

    - parents: 0,
    - interior: X1,
    - X1 junction: Parachain
    - Parachain index: 1000

    After you configure all of the XCM instructions to be executed, you're ready to submit the transaction.

## Submit the transaction

To submit the transaction:

1. Click **Submit Transaction**.

1. Click **Sign and Submit**.

1. Click **Network** and select **Explorer** to verify the message is sent.

    If you expand the event, you can review the message instructions.
    If you click the link to the block that includes the transaction, you can see additional details

## Check events on the relay chain

To check the result on the relay chain:

1. Open the [InfraBlockchain Portal](https://portal.infrablockspace.net/#/explorer/) and connect to the relay chain.

2. Click **Network** and select **Explorer** to view the events for the XCM message.

    ![Relay chain events](/media/images/docs/infrablockchain/tutorials/build/relay-chain-event-summary.png)

3. Click the block number where the change was recorded to view details.

    ![Assets withdrawn and deposited are recorded on the relay chain](/media/images/docs/infrablockchain/tutorials/build/relay-chain-block.png)

## Check the assets deposited

To verify the assets deposited into the account:

1. Open the [InfraBlockchain Portal](https://portal.infrablockspace.net/#/explorer/) and connect to the relay chain.

2. Click **Accounts** and see the assets minus transaction fees have been deposited into the account.

    For example:

    ![Assets have been deposited in the specified account](/media/images/docs/infrablockchain/tutorials/build/transfer-account-funded.png)

    If you had made the remote transfer to the parachain A (1000) sovereign account instead of the KRIS-PUBS account, you would click **Accounts**, then select **Address book** to see the assets withdrawn from the parachain B sovereign account deposited into the parachain A sovereign account.
