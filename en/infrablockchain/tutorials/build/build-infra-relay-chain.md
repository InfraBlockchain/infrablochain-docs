---
title: Build InfraRelayChain
description: 
keywords:
  - Relay Chain
---

## Before You Begin

Before you start, make sure to:

- Review the [Rust and Rust Toolchain](../../learn/substrate/tutorials/install/rust-toolchain.md) to set up the Substrate development environment.

- Familiarize yourself with Multi-Chain Architecture and [Terms](../../learn/architecture/architecture.md).

## Tutorial Objectives

By completing this tutorial, you will achieve the following objectives:

- Learn how to build the ***InfraRelayChain*** runtime as a binary.
- Extract the plain chain spec and the SCALE-encoded raw chain spec from the binary.
- Start nodes from the chain spec.

## Build the Relay Chain Node

1. Clone the latest ***InfraBlockchain*** SDK:
   
   ```bash
   git clone --branch master https://github.com/InfraBlockchain/infrablockchain-substrate.git
   ```

2. Build the ***InfraBlockchain*** runtime by executing the following three commands:
   
   ```bash
   1. cargo build --release --bin infrablockspace

   2. cargo build --release --bin infrablockspace-execute-worker

   3. cargo build --release --bin infrablockspace-prepare-worker
   ```

3. Verify that the node has been built correctly by running the following command in the same directory:
   
   ```bash
   ./target/release/infrablockspace --help
   ```

   If the command displays the command-line help, the node is ready for configuration.

## Chain Specs

All Substrate-based chains require a [chain spec](../../learn/substrate/build/chain-spec.md). The chain spec specifies the initial state of the network (genesis state) and various network settings. For deterministic network configuration, all nodes participating in the network have the same chain spec.

### Extracting the Plain Chain Spec

Use the following command in the same directory to extract the plain chain spec. After running the command, the `plain-infra-relay-chainspec.json` file is created in the same path.

```bash
./target/release/infrablockspace build-spec --chain infra-relay-local --disable-default-bootnode > plain-infra-relay-chainspec.json
```

*In this tutorial, a sample chain spec file for the ***InfraRelayChain*** with four [Seed Trust Validators](../../learn/protocol/proof-of-transaction.md#블록-생성자밸리데이터-풀) is used. For actual network configuration, only `alice` and `bob` validators are started for simulation purposes.*


Relay Chain must have at least one validator node running for each connected parachain collator. Therefore, to connect two parachains with one collator each, you need to run at least three Relay Chain validator nodes.

### Extracting the Raw Chain Spec File

Extract a [SCALE-encoded](../../learn/substrate/learn/frame/scale-codec.md) raw JSON file. After running the command, the `raw-infra-relay-chainspec.json` file is created in the same path. 

```bash
./target/release/infrablockspace build-spec --chain plain-infra-relay-chainspec.json --disable-default-bootnode --raw > raw-infra-relay-chainspec.json
```

You can read and edit the plain text version of the chain spec file. However, to use the chain spec file to start a node, it needs to be converted to the SCALE-encoded raw format. The sample chain spec represents a network with four validators. If you want to add different validators, multiple relay chains, or use custom account keys instead of predefined accounts, you will need to create a [custom chain spec file](../../learn/substrate/build/chain-spec.md#커스텀-체인-스펙-생성하기).

If someone else is completing this tutorial concurrently on the same local network, modify the plain chain spec to avoid connecting nodes. In the plain chain spec JSON file, find the `protocolId` key and change it to a unique value:

```json
   "protocolId": "infrablockchain"
```

## Start the Relay Chain Node

To build the ***InfraBlockchain***, you need to start the ***InfraRelayChain***, which can connect parachains.

Start a validator node for `alice` account:
   
   ```bash
   ./target/release/infrablockspace \
   --alice \
   --validator \
   --base-path /tmp/relay/alice \
   --chain ./raw-infra-relay-chainspec.json \
   --port 30333 \
   --rpc-port 9944
   ```

- `--validator`: Option to start a validator node.
- `--base_path`: Specifies the location where chain data is stored.
- `--chain`: Path to the raw chain spec file.
- `--port, --rpc-port`: Specifies the ports for communication with the chain.

   _Ensure that the same ports are not used by other nodes on the local computer._

1. After starting the node, you should see logs similar to the following:

   ```bash
   2023-11-16 18:05:43 bclabs InfraBlockspace
   2023-11-16 18:05:43 ✌️  version 1.1.0-e146a06602e
   2023-11-16 18:05:43 ❤️  by blockchain labs, 2017-2023
   2023-11-16 18:05:43 📋 Chain specification: Infra Relay Devnet
   2023-11-16 18:05:43 🏷  Node name: Alice
   2023-11-16 18:05:43 👤 Role: AUTHORITY
   2023-11-16 18:05:43 💾 Database: RocksDb at /tmp/relay/alice/chains/infra_relay_devnet/db/full
   2023-11-16 18:05:44 BEEFY is still experimental, usage on Polkadot network is discouraged.
   2023-11-16 18:05:45 👶 Creating empty BABE epoch changes on what appears to be first startup. 
   2023-11-16 18:05:45 🏷  Local node identity is: 12D3KooWBJyq6nyn6JJSmdy5QmemYBCofKvWh6w5Am6p33tYzxu1

   ...
   ```

2. Record the `peerid` for the `alice` node so that other nodes can connect:
   
   ```bash
   🏷 Local node identity is: 12D3KooWBJyq6nyn6JJSmdy5QmemYBCofKvWh6w5Am6p33tYzxu1
   ```

3. Open a new terminal and start the second validator node for the `bob` account:

   ```bash
   ./target/release/infrablockspace \
   --bob \
   --validator \
   --base-path /tmp/relay/bob \
   --chain ./raw-infra-relay-chainspec.json \
   --port 30334 \
   --rpc-port 9945 
   ```

   _The command used to start the first node is similar to the one used for starting the second node, but there are some important differences.._
   
- Use a command similar to the one used for the first node, but with a different base path (`/tmp/relay/bob`), validator key (`--bob`), and ports (30334 and 9945).
   
- Since both validators are running on the same local computer, there is no need to specify the IP address and peer identifier of the first node as specified in the chain spec file. The `bootnodes` option is necessary when connecting to nodes running outside the local network or nodes not identified in the chain spec file.

- If the relay chain is not producing blocks, try disabling the firewall or start the node with the address of the `alice` node in the `--bootnodes` option:

   Example,

   ```bash
   ./target/release/infrablockspace \
   --bob \
   --validator \
   --base-path /tmp/relay/bob \
   --chain ./raw-infra-relay-chainspec.json \
   --port 30334 \
   --rpc-port 9945 
   --bootnodes /ip4/127.0.0.1/tcp/30333/p2p/12D3KooWGjsmVmZCM1jPtVNp6hRbbkGBK3LADYNniJAKJ19NUYiq
   ```

- Once the `bob` node is started, you can confirm that blocks are being generated.

   ```bash
   2023-11-16 18:08:57 discovered: 12D3KooWP2vgMARpeD9H5innUPaBR7LFQqJSP6dX4TRS9DtkqsBQ /ip4/172.16.72.194/tcp/30334
   2023-11-16 18:09:00 💤 Idle (1 peers), best: #0 (0xe521…3cca), finalized #0 (0xe521…3cca), ⬇ 1.5kiB/s ⬆ 1.5kiB/s
   2023-11-16 18:09:05 💤 Idle (1 peers), best: #0 (0xe521…3cca), finalized #0 (0xe521…3cca), ⬇ 0.2kiB/s ⬆ 0.2kiB/s
   2023-11-16 18:09:10 💤 Idle (1 peers), best: #0 (0xe521…3cca), finalized #0 (0xe521…3cca), ⬇ 0 ⬆ 0
   2023-11-16 18:09:12 🙌 Starting consensus session on top of parent 0xe5212b368879d4a38e84693a0f1582402ac100948a895217823de534cf753cca
   🎁 Prepared block for proposing at 1 (2 ms) [hash: 0x2a02735687bb7ec53f34e17424a313b8b05ecce8ac855216dfae3c254980efdc; parent_hash: 0xe521…3cca; extrinsics (2): [0x62c3…6593, 0xf265…0515]
   ```

## Next Steps

- [Build a parachain](./build-a-parachain.md)