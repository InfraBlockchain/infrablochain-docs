---
title: Develop smart contracts
description: Demonstrates using the ink! programming language to create and deploy smart contracts on a Substrate-based network.
keywords:
  - smart contracts
---

The **Develop smart contracts** tutorials guide you through how you can use [the ink! programming language](https://use.ink) to build smart contracts that run on a Substrate-based blockchain.

The tutorials in this section use a preconfigured
[`substrate-contracts-node`](https://github.com/paritytech/substrate-contracts-node) to keep your work focused on the basics of writing smart contracts.

If you use the [standard node template](https://github.com/substrate-developer-hub/substrate-node-template), you must manually add the [Contracts pallet](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/contracts) and make other changes to your development environment.
You can compare the runtime code for the two nodes to see additional information about the differences between them.

- [Prepare your first contract](./prepare-your-first-contract.md) describes how to update your
  development environment and create a smart contract project using the ink! programming language.
- [Develop a smart contract](./develop-a-smart-contract.md) demonstrates how to store, increment,
  and retrieve simple values using a smart contract.
- [Use maps for storing values](./use-maps-for-storing-values.md) extends the previous tutorial
  by illustrating how to use maps to store and retrieve values in a smart contract.
- [Build a token contract](./build-a-token-contract.md) illustrates how you can build a simple
  smart contract for transferring ERC-20 tokens.
- [Troubleshoot smart contracts](./troubleshoot-smart-contracts.md) describes a few common issues
  that you might encounter when writing and deploying smart contracts.
