---
title: Build application logic
description: Demonstrates how you can add pallets to customize the Substrate runtime environment.
keywords:
---

The **Build application logic** tutorials focus on how you can customize the runtime using pallets, including how to add simple and complex pallets to the runtime and how to use pallets in combination with smart contracts.
You'll learn how to:

- [Add a pallet to the runtime](./add-a-pallet.md) introduces the common steps for adding a simple predefined pallet to the node template runtime.
- [Use macros in a custom pallet](./use-macros-in-a-custom-pallet.md) illustrates how to create a custom pallet using macros.
- [Specify the origin for a call](./specify-the-origin-for-a-call.md) demonstrates how you can specify the account to use as the originator of a function call.
- [Add offchain workers](./add-offchain-workers.md) illustrates how to modify a pallet to include an offchain worker and configure the pallet and runtime to enable the offchain worker to submit transactions that update the on-chain state.
- [Publish custom pallets](./publish-custom-pallets.md) illustrates how to publish custom pallets and crates so they are available to the community.

If you want to experiment with smart contract development for your blockchain, you should use the preconfigured [substrate-contracts-node](https://github.com/paritytech/substrate-contracts-node) instead of the standard node template.
There are compatibility issues between the pallets that support smart contracts and the current node template.
To resolve these compatibility issues would require extensive changes to the configuration of the standard node template, including using older versions of all crates and modifying multiple files.

<!--
- [Configure the contracts pallet](/tutorials/build-application-logic/contracts-pallet/) demonstrates how to configure a complex pallet to work with smart contracts.

-->
