---
title: How-to quick reference guides
description:
keywords:
---

Substrate _How-to_ quick reference guides provide instructions for achieving specific goals.
Each guide explains how to perform a specific task with the assumption that you are already familiar with Substrate and programming in Rust.

## Basics

See the following guides to learn common patterns in runtime development.

- [Import a pallet](./basics/import-a-pallet.md)
- [Configure runtime constants](./basics/configure-runtime-constants.md)
- [Configure genesis state](./basics/configure-genesis-state.md)
- [Customize a chain specification](./basics/customize-a-chain-specification.md)

## Pallet design

See the following guides for best practices on building pallets using FRAME.

- [Implement lockable currency](./pallet-design/implement-lockable-currency.md)
- [Incorporate randomness](./pallet-design/incorporate-randomness.md)
- [Configure crowdfunding](./pallet-design/configure-crowdfunding.md)
- [Create a storage structure (struct)](./pallet-design/create-a-storage-structure.md)
- [Use tight pallet coupling](./pallet-design/use-tight-coupling.md)
- [Use loose pallet coupling](./pallet-design/use-loose-coupling.md)

## Weights

See the following guides for help with benchmarking and weight configurations.

- [Calculate fees](./weights/calculate-fees.md)
- [Add benchmarks](./weights/add-benchmarks.md)
- [Use custom weights](./weights/use-custom-weights.md)
- [Use conditional weights](./weights/use-conditional-weights.md)

## Testing

See the following guides for help with testing pallets and runtime logic.

- [Set up basic tests](./testing/set-up-basic-tests.md)
- [Test a transfer function](./testing/test-a-transfer-function.md)

## Storage migration

See the following guides for help with a storage migration.

- [Basic storage migration](./storage-migrations/basic-storage-migration.md)
- [Trigger migration](./storage-migrations/trigger-migration.md)

## Consensus models

See the following guides to implement consensus mechanisms in your runtimes.

- [Create a hybrid node](./consensus-models/create-a-hybrid-node.md)
- [Add proof-of-work consensus](./consensus-models/add-proof-of-work-consensus.md)

## Parachains

See the following guides for help working with Substrate parachains.

- [Convert a solo chain](./parachains/convert-a-solo-chain.md)
- [Connect to a relay chain](./parachains/connect-to-a-relay-chain.md)
- [Select collators](./parachains/select-collators.md)
- [Prepare to launch](./parachains/prepare-to-launch.md)
- [Upgrade a parachain](./parachains/upgrade-a-parachain.md)
- [Auctions and crowdloans](./parachains/auctions-and-crowdloans.md)
- [Add HRMP channels](./parachains/add-hrmp-channels.md)

## Tools

See the following guides for add-on tools that help you manage Substrate chains in production.

- [Use try-runtime](./tools/use-try-runtime.md)
- [Create a txwrapper for a chain](./tools/create-a-txwrapper.md)
- [Use REST endpoints to get chain data](./tools/use-sidecar.md)
- [Verify a Wasm binary](./tools/verify-wasm.md)

## Offchain workers

See the following guides for help working with offchain data.

- [Make offchain HTTP requests](./offchain-workers/offchain-http-requests.md)
- [Offchain local storage](./offchain-workers/offchain-local-storage.md)
- [Offchain indexing](./offchain-workers/offchain-indexing.md)

<!--
- [Calculate weight](/reference/how-to-guides/basics/calc-weights/)
- [Mint primitive tokens](/reference/how-to-guides/basics/mint-basic-tokens/)
- [Add the contracts pallet](/reference/how-to-guides/pallet-design/add-contracts-pallet/)
-->
