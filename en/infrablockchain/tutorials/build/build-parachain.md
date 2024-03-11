---
title: Build a Parachain
description:
keywords:
  - Parachain
  - InfraRelayChain
---

This tutorial explains how to connect a parachain to the **_InfraRelayChain_**.

## Tutorial Objectives

By completing this tutorial, you will achieve the following goals:

- Understand the features of the Relay Chain and parachain.
- Connect different chains (Relay Chain and parachain) to build a multi-chain architecture.

## Before Getting Started

Before you begin, make sure to:

- Review the[ Local Relay Chain Setup tutorial](./build-infra-relay-chain.md) for configuring the local Relay Chain.
- Familiarize yourself with [configuring a Custom Chain Spec](../../learn/substrate/build/chain-spec.md).

## Building Parachain Template

In this tutorial, we will use _Parachain Template_ to connect it to the **_InfraRelayChain_**. The parachain template can also serve as a starting point for parachain development.

To build the parachain template, follow these steps:

To build the parachain template, follow these steps:

1. Clone the latest **_InfraBlockchain_** SDK:

   ```bash
   git clone --branch master https://github.com/InfraBlockchain/infrablockchain-substrate.git
   ```

2. Execute the following command to build the parachain template runtime:

   ```bash
   cargo build --release --bin parachain-template-node
   ```

## Reserve Parachain Identifier (ParaId)

Parachains connected to Relay Chain need to reserve a unique identifier known as `ParaId`. This identifier is used by Relay Chain to identify and connect parachains. The ParaId is also used to identify the slot occupied by the parachain or parathread, as explained in [_Parachain_](https://wiki.polkadot.network/docs/learn-parachains) or [_Parathread_](https://wiki.polkadot.network/docs/learn-parathreads) documentation.

To reserve a parachain identifier, follow these steps:

1. Set up a local Relay Chain network with at least two validator nodes running.

2. Open the [InfraBlockchain Explorer](https://portal.infrablockspace.net/#/explorer) in your browser.

3. Connect to the local Relay Chain by clicking on the left-top tab in the explorer and selecting the `DEVELOPMENT` section using the RPC port you configured when starting the local Relay Chain.

   ![Connect Local Network](/media/images/docs/infrablockchain/tutorials/build/connect-network.png)

4. Click on **Network** and select **Parachains**.

   ![Move to Parachain](/media/images/docs/infrablockchain/tutorials/build/network-parachains.png)

5. Click on Parathreads and then click on ParaId.

   ![Reserve a Parachain Identifier](/media/images/docs/infrablockchain/tutorials/build/paraid-reserve.png)

6. Confirm the transaction configuration for reserving the parachain identifier and click `Submit`.

   The account used to reserve the parachain identifier is the account that will be charged for the transaction fee, and it will be associated with the reserved identifier.

7. Click on **Sign and Submit** to approve the transaction.

8. After submitting the transaction, click on **Network** and select Explorer.

9. Check the list of `registrar.Reserved` events and click on an event to see detailed information about the transaction.

Now you are ready to prepare the chain spec and generate the necessary files to connect the parachain to the relay chain using the reserved parachain identifier (`paraId 2000` in this case).

## Parachain Registration Process

### Modify Plain Chain Spec

To register the parachain on the local Relay Chain, you need to modify the plain chain spec to include the reserved parachain identifier.

To generate the plain chain spec of the parachain template node, execute the following command:

```bash
./target/release/parachain-template-node build-spec --disable-default-bootnode > plain-para-template-chainspec.json
```

- Open the generated plain chain spec in a text editor.

- Set the `para_id` field to the reserved identifier.

  ```json
  ...
  "relay_chain": "infra-relay-local",
  "para_id": 2000,
  "codeSubstitutes": {},
  "genesis": {
     ...
  }
  ```

- Set `parachainId` to the reserved parachain identifier.

- For example, if the reserved identifier is 2000, set the `para_id` field to 2000:

  ```json
  ...
     "parachainSystem": null,
     "parachainInfo": {
       "parachainId": 2000
     },
  ...
  ```

- If you are completing this tutorial concurrently with someone else on the same local network, make sure to modify the plain chain spec to avoid peering with each other. Find the `protocolId` key in the plain chain spec JSON file and change it to a unique value.

  ```json
     "protocolId": "infrablockchain"
  ```

- Save the changes and close the plain chain spec file.

To generate the raw chain spec file from the modified plain chain spec, execute the following command:

```bash
./target/release/parachain-template-node build-spec --chain plain-para-template-chainspec.json --disable-default-bootnode --raw > raw-para-template-chainspec.json
```

This command generates a new raw chain spec file with two new collators for a new session at block #0.

```text
2023-12-01 13:00:50 Building chain spec
2023-12-01 13:00:50 assembling new collators for new session 0 at #0
2023-12-01 13:00:50 assembling new collators for new session 1 at #0
```

### Export Parachain WebAssembly Runtime and Genesis State

To prepare the parachain collators for registration, follow these steps:

1. Export the WebAssembly runtime for the parachain.

   To export the WebAssembly runtime for the parachain, execute a command similar to the following:

   ```bash
   ./target/release/parachain-template-node export-genesis-wasm --chain raw-para-template-chainspec.json para-2000-wasm
   ```

2. Generate the parachain genesis state.

   To register parachain, you should know the genesis state. To export the entire genesis state to a file encoded in _hexadecimal_, execute a command similar to the following:

   ```bash
   ./target/release/parachain-template-node export-genesis-state --chain raw-para-template-chainspec.json para-2000-genesis-state
   ```

   The exported runtime and state are specifically for the genesis block. **A parachain must start from block height 0.** A parachain with a previous state cannot be connected to the relay chain.

## Start Collator Nodes

Collator nodes are specialized nodes for parachains with the following roles:

- Retrieve the latest state from the relay chain.
- Generate parachain blocks and submit them to the relay chain.

For more details, refer to [Collators Document](https://wiki.polkadot.network/docs/learn-parachains-protocol#collators).

1. Start the collator node with a command similar to the following:

   ```bash
   ./target/release/parachain-template-node \
   --alice \
   --collator \
   --force-authoring \
   --chain raw-para-template-chainspec.json \
   --base-path /tmp/parachain/alice \
   --port 40333 \
   --rpc-port 8844 \
   -- \
   --execution wasm \
   --chain ../infrablockspace/raw-infra-relay-chainspec.json \
   --port 30343 \
   --rpc-port 9977
   ```

- The arguments passed before the `--` in this command are for the collator.

- Those after the `--` are for the embedded relay chain node.

- Specify that the node is a collator node through the `--collator` option.

- This command specifies both the raw chain spec of the parachain and the raw chain spec of the relay chain.

- In this example, it uses the raw chain spec of the local relay chain named `raw-infra-relay-chainspec.json` in the `infrablockspace` directory.

- Make sure the second `--chain` command-line specifies the path to the raw chain spec file of the local relay chain.

- When starting other parachain nodes, use the same relay chain spec file but different base paths and port numbers.

- In the terminal where the collator node is started, you should see output similar to the following:

  ```text
  Parachain Collator Template
  ✌️  version 0.1.0-336530d3bdd
  ❤️  by Anonymous, 2020-2023
  📋 Chain specification: Local Testnet
  🏷  Node name: Alice
  👤 Role: AUTHORITY
  💾 Database: RocksDb at /tmp/parachain/alice/chains/local_testnet/db/full
  no effect anymore and will be removed in the future!
  Parachain Account: 5Ec4AhPUwPeyTFyuhGuBbD224mY85LKLMSqSSo33JYWCazU4
  Is collating: yes
  [Relaychain] 🏷  Local node identity is: 12D3KooWR8wJbGWrjzKTpuXQvbuM1rE2GAE9JVEFEwAyNX6LV9nN
  [Relaychain] 💻 Operating system: ...
  ......
  [Relaychain] 📦 Highest known block at #95
  ...
  [Parachain] 🏷  Local node identity is: 12D3KooWF464pkLaHbsfc4DzDkYuhdVE4zSqBHR1gPapLvwsfZtg
  ...
  ```

- The template collator node runs as an independent node, and the embedded Relay Chain node connects as a peer to the local Relay Chain verifier node. (If the embedded Relay Chain is not peered with the local Relay Chain node, try disabling the firewall or adding the address of the Relay Chain node to the `bootnodes` parameter.)

## Registering with the Local Relay Chain

With the local Relay Chain and collator node running, you are ready to register the parachain with the local Relay Chain. While on the actual public main network, parachain registration usually occurs through a governance process, in this tutorial, we'll proceed using a Sudo transaction.

### Parachain Registration Steps:

1. Ensure that the local Relay Chain is running with at least two validator nodes.

2. Open the [InfraBlockchain Explorer](https://portal.infrablockspace.net/#/explorer) in your browser. Connect to the Relay Chain using RPC and WebSocket ports, just like when reserving the parachain identifier.

3. Click on **Developer** and select **Sudo**.

4. Choose _paraSudoWrapper_ and then select `sudoScheduleParaInitialize(id, genesis)`.

- Transaction parameters:

  - `id`: Enter the reserved ParaId. In this tutorial, let's assume the reserved identifier is `2000`.
  - `genesisHead`: Click on **File Upload** and upload the exported genesis state for the parachain. In this tutorial, select the para-2000-genesis file.
  - `validationCode`: Click on **File Upload** and upload the exported WebAssembly runtime for the parachain. In this tutorial, select the para-2000-wasm file.
  - `paraKind`: Choose **Yes**.

  ![Selecting Sudo for Parachain Registration](/media/images/docs/infrablockchain/tutorials/build/register-para-with-sudo.png)

5. Click on **Submit Sudo**.

6. Review the transaction details and click on **Sign and Submit** to approve the transaction.

   After submitting the transaction, click on **Network** and choose **Explorer**.

7. In the recent events list, verify the events `sudo.Sudid` and `paras.PvfCheckAccepted`. Click on the events to see details about the transaction.

   ![Sudo Event](/media/images/docs/sudo-event.jpg)

   ![PVF Check Accepted](/media/images/docs/infrablockchain/tutorials/build/pvf-check-accepted.png)

8. Once the parachain registration is complete, you can verify it in the Explorer. Click on Network and select **Parachains**.

9. Wait until a new epoch starts.

- Relay Chain tracks the latest block (head) of each parachain.
- Once the Relay Chain block is finalized, the parachain block, which has completed the [verification process](../../learn/architecture/architecture.md#parachain-protocol), is also finalized.
- This is how the **_shared security_** in the multi-chain architecture of **_InfraBlockchain_** is achieved.
- When the parachain is connected to the Relay Chain in the next epoch, you should be able to see information about that parachain in the Explorer by moving to the port you specified when starting the collator node.

## Submitting Parachain Transactions

Now that the parachain is running and connected to the relay chain, you can use the Explorer to submit transactions to the parachain.

To submit a transaction, follow these steps:

1. Open the [InfraBlockchain Explorer](https://portal.infrablockspace.net/#/explorer/) in your browser.

2. Click on the network selector at the top left of the application.

3. Change the custom endpoint to connect to the WebSocket port of the parachain. If you follow the settings in this tutorial, connect to port `8844`.

4. Click on **Accounts** and select **Transfer** to send funds from Alice to another account.

- Choose the account from which to send funds.
  Enter the amount.
- Enter the amount.
- Click on **Generate Transfer**.
- Review the transaction and click on **Sign and Submit** to approve the transfer.

5. Click on **Accounts** to verify that the transfer is complete and the parachain transaction has been successful.

## Resetting the Blockchain State

In networks for testing and development purposes, the blockchain state is reset and restarted each time a node is started. However, be cautious when deleting the chain state or manually deleting the database, as recovering data or restoring the chain state will not be possible. If there is data that needs to be preserved, make sure to keep a copy before resetting the parachain state.

In this tutorial, we are simulating a network environment for testing and development purposes, not an actual network. Therefore, to start fresh, you need to completely remove the chain state of the local relay chain node and the parachain.

To reset the blockchain state, follow these steps:

1. Press `Control-c` in the terminal where the parachain template node is running.
2. Execute the following command to remove the parachain collator state:

   ```bash
   rm -rf /tmp/parachain
   ```

3. Press `Control-c` in the terminal where the `alice` validator node or `bob` validator node is running.

4. Execute the following command to remove the validator state:

   ```bash
   rm -rf /tmp/relay
   ```
