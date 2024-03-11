---
title: Including Votes in Transactions
description: This tutorial describes how to use Proof-of-Transaction(PoT) to elect validators on the InfraRelayChain.
keywords:
  - Proof-of-Transaction(PoT)
  - System Token
---

This tutorial will explore how to select validators for the relay chain in the **_InfraRelayChain_** based on [Proof-of-Transaction](../learn/protocol/proof-of-transaction.md) using [Aggregated TaaV (Transaction-as-a-Vote)](../learn/protocol/proof-of-transaction.md#aggregated-proof-of-transactionpot) in a real multi-chain environment.

## Tutorial Objectives

By completing this tutorial, you will achieve the following objectives:

- Build a test network for `Relay Chain <> Parachains` using the [ZombieNet](../tutorials/test/simulate-parachains.md).
- Pay transaction fees with System Token and vote for validators in the **_InfraRelayChain_**.
- Confirm the configuration of validators in the **_InfraRelayChain_** with [Seed Trust](../learn/protocol/proof-of-transaction.md#validator-pool) and PoT nodes.

## Before You Begin

Before getting started, make sure to:

- Learn how to generate the binaries for testing by following [Building a Relay Chain](../tutorials/build/build-infra-relay-chain.md) and [Building a Parachain](../tutorials/build/build-a-parachain.md).

- Familiarize yourself with [Proof-of-Transaction](../learn/protocol/proof-of-transaction.md).

- Understand how to pay transaction fees using [System Token](../learn/protocol/system-token.md).

## Tutorial Steps

This tutorial will proceed in the following steps:

1. Set up the test environment using **ZombieNet**. In this tutorial, we will use two parachains, which are **InfraAssetHub**, and **Parachain Template Node**.

2. Request transactions, including the target for the vote, using [System Token](../learn/protocol/system-token.md) in each parachain.

3. Verify that the votes are collected in the **_InfraRelayChain_**.

4. Check whether validators based on [Proof-of-Transaction(PoT)](../learn/protocol/proof-of-transaction.md) are elected to generate blocks at a certain point in time.

## Build the Test Environment

Build the test environment for the **_InfraBlockchain_** using **ZombieNet** binaries. First, install the ZombieNet binaries:

- Install the latest release version from the [ZombieNet GitHub](https://github.com/paritytech/zombienet).

- Create a TOML file in `infrablockspace-sdk/infra-cumulus/zombienet/examples/`. In this example, create `example.toml` with the following content, which sets up the test environment with:

  - 5 validator relay chain nodes
  - 2 **InfraAssetHub** collator nodes
  - 2 **Parachain Template Node** collator nodes

```toml
[relaychain]
default_command = "{path-to-binary}/infrablockspace"
default_args = ["-lparachain=debug", "-l=xcm=trace"]
chain = "infra-relay-local"

[[relaychain.nodes]]
name = "alice"
validator = true
rpc_port = 7100
ws_port = 7101

[[relaychain.nodes]]
name = "bob"
validator = true
rpc_port = 7200
ws_port = 7201

[[relaychain.nodes]]
name = "charlie"
validator = true
rpc_port = 7300
ws_port = 7301

[[relaychain.nodes]]
name = "dave"
validator = true
rpc_port = 7400
ws_port = 7401

[[relaychain.nodes]]
name = "eve"
validator = true
rpc_port = 7500
ws_port = 7501

[[relaychain.nodes]]
name = "ferdie"
validator = true
rpc_port = 7600
ws_port = 7601

# Asset-Hub
[[parachains]]
id = 1000
chain = "infra-asset-hub-local"
cumulus_based = true

# run alice & bob as parachain collator
[[parachains.collators]]
name = "alice"
validator = true
command = "{path-to-binary}/infra-parachain"
args = ["-lparachain=debug", "-l=xcm=trace"]
rpc_port = 9500
ws_port = 9501

[[parachains.collators]]
name = "bob"
validator = true
command = "{path-to-binary}/infra-parachain"
args = ["-lparachain=debug", "-l=xcm=trace"]
rpc_port = 9510
ws_port = 9511

# Template-node
[[parachains]]
id = 2000
chain = "local"
cumulus_based = true

# run alice & bob as parachain collator
[[parachains.collators]]
name = "alice"
validator = true
command = "{path-to-binary/parachain-template-node}"
args = ["-lparachain=debug", "-l=xcm=trace"]
rpc_port = 9600
ws_port = 9601

# run alice as parachain collator
[[parachains.collators]]
name = "bob"
validator = true
command = "{path-to-binary/parachain-template-node}"
args = ["-lparachain=debug", "-l=xcm=trace"]
rpc_port = 9610
ws_port = 9611

[[hrmp_channels]]
sender = 1000
recipient = 2000
max_capacity = 8
max_message_size = 1048576
[[hrmp_channels]]
sender = 2000
recipient = 1000
max_capacity = 8
max_message_size = 1048576
```

- Run the ZombieNet command in the terminal:

```bash
./zombienet-macos spawn --provider native ./zombienet/examples/example.toml

chmod +x zombienet-macos
```

## Including Votes in Transactions

- To pay transaction fees with System Token and vote, refer to [How to Interact with System Token](./how-to-interact-with-system-token.md). For this tutorial, assume that System Token is generated from **Parachain Template Node**.

- Go to the [InfraBlockchain Explorer](https://portal.infrablockspace.net/#/explorer).

- In this explorer, you can include votes in any transaction. For this tutorial, let's assume a scenario where a `balance-transfer` transaction is sent from **Parachain Template Node**.

- Navigate to the **Developer** section and enter **Extrinsics** tab. Select `balances` and then `transfer`. Fill in the required values for the transaction and click on **Submit** Transaction.

- Enable the include an optional `asset id` and include an optional vote options among the transaction's sub-options.

  - The `asset id` option specifies System Token to be used for transaction fees.

  ```
  - parachain_id: 파라체인 식별자
  - pallet_index: Assets 팔렛 식별자
  - asset_id: 시스템 토큰의 식별자
  ```

  - The `vote` option designates the candidate for **Proof-of-Transaction(PoT)** to be elected.

  ![Transaction Details](/media/images/docs/infrablockchain/tutorials/tx-detail.png)

- Wait until the block is processed, then check for the following event in **_InfraRelayChain_**.

  ![Relay Chain Event](/media/images/docs/infrablockchain/tutorials/infra-relay-event.png)

- The votes have been aggregated according to the specified weight, and you can also verify this in the storage of **_InfraRelayChain_**.

  ![Storage](/media/images/docs/infrablockchain/tutorials/infra-relay-storage.png)

## Next Steps

- [How to Get Validator Reward](./how-to-get-validator-reward.md)

- [System Token Weight](../learn/protocol/transaction-fee.md)
