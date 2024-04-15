---
title: Including a Vote in a Transaction
description: This tutorial explains how to use PoT to elect validators for the Infra Relay Chain.
keywords:
  - Proof-of-Transaction(PoT)
  - System Token
---

This tutorial will teach you how to use [Aggregated TaaV(Transaction-as-a-Vote)](../learn/protocol/proof-of-transaction.md#aggregated-proof-of-transactionpot) in a real multi-chain environment to elect validators for the **_Infra Relay Chain_** based on [Proof-of-Transaction, PoT](../learn/protocol/proof-of-transaction.md).

## Tutorial Objectives

By completing this tutorial, you will be able to:

- Build a `RelayChain <> Parachain` test network using a zombie network.
- Use the system token to pay transaction fees and vote for validator candidates for the **_Infra Relay Chain_**.
- Confirm that the validators for the **_Infra Relay Chain_** are composed of Seed Trust nodes and PoT nodes.

## Before You Start

Before you start, make sure to:

- Learn how to build a relay chain and a parachain through [Building a Relay Chain](../tutorials/build/build-infra-relay-chain.md) and [Building a Parachain](../tutorials/build/build-a-parachain.md) to learn how to generate binaries for testing.

- Learn about [Proof-of-Transaction](../../learn/protocol/proof-of-transaction.md) methods.

- Learn how to pay transaction gas fees with the [System Token](../../learn/protocol/system-token.md).

## Tutorial Steps

This tutorial will proceed in the following order:

- Build a test environment with a zombie network. In this tutorial, we will use two parachains, **InfraAssetHub** and **Parachain Template Node**.

- Request a transaction that includes the target to vote for using the [System Token](../learn/protocol/system-token.md) in each parachain.

- Confirm that the votes are aggregated in the **_Infra Relay Chain_**.

- At a specific point in time, confirm that validators are elected and blocks are created based on [Proof-of-Transaction](../learn/protocol/proof-of-transaction.md).

## Building the Test Environment

Build the **_InfraBlockchain_** test environment for the tutorial. First, install the zombie network.

- Install the latest release version from the zombie network [GitHub](https://github.com/paritytech/zombienet).

- Create a `toml` file in `infrablockchain-substrate/infra-cumulus/zombienet/examples/`. Here, we will create `example.toml` and write it as follows. This code sets up the following test environment:
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

- Enter the zombie network command in the terminal:

```bash
./zombienet-macos spawn --provider native ./zombienet/examples/example.toml

chmod +x zombienet-macos
```

## Including a Vote in a Transaction

- To pay transaction fees and vote using the system token, first refer to the [System Token Creation](./how-to-interact-with-system-token.md) document. In this tutorial, we assume that a system token has been created in the **Parachain Template Node**.

- Go to the [InfraBlockchain Explorer](https://portal.infrablockspace.net).

- In this explorer, you can include a vote in any transaction. In this tutorial, we assume a situation where a `balance-transfer` transaction is sent from the **Parachain Template Node**.

- Go to the `Developer` section and enter the `Extrinsics` tab. Select `transfer` in `balances`. Fill in the values required for the transaction and click `Submit Transaction`.

  ![transfer](/media/images/docs/infrablockchain/tutorials/tx-detail.png)

- Turn on the `include an optional asset id` (required option) and `include an optional vote` (optional) options.

- The `asset_id` option specifies the system token to be used as the transaction fee. At the runtime point, specify the `MultiLocation` of the token. For example, in the case of a parachain, if you use the [wrapped token](../../learn/protocol/system-token.md#랩드wrapped-시스템-토큰) of the relay chain, the MultiLocation is as follows:

```rust
RelayChainAsset = MultiLocation { parents: 1, interior: X2(Pallet, GeneralIndex)}
```

- In `vote_candidate`, enter the account information of the candidate to vote for.

- Wait until the block is processed and check the following event in the **_Infra Relay Chain_**:

  ![Event](/media/images/docs/infrablockchain/tutorials/infra-relay-event.png)

- The votes are aggregated according to the weight, and you can also check it in the **_Infra Relay Chain_** storage.

  ![Storage](/media/images/docs/infrablockchain/tutorials/infra-relay-storage.png)

## Moving on to the Next Step

- [How to Receive Rewards as a Validator](./how-to-get-validator-reward.md)

- [System Token Weights](../learn/protocol/transaction-fee.md#system-token-weights)
