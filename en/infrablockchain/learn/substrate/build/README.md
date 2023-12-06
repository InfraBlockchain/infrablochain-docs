---
title: Build
description: Exposes details about how Substrate nodes are constructed and compiled.
keywords:
---

The topics in this section provide a more detailed exploration of the code used to construct the runtime logic, including the libraries and tools available for building and interacting with the node and a closer look at how the logic is compiled to build a Substrate node.

- [Remote procedure calls](./remote-procedure-calls.md) summarizes how you can use remote procedure calls and RPC methods to interact with a Substrate node.
- [Application development](./application-development.md) introduces the role of metadata and front-end libraries as tools for building applications that run on the blockchain.
- [Chain specification](./chain-spec.md) discusses the use of chain specifications, including what you can and can't modify, and how to distribute customized chain specifications.
- [Genesis configuration](./genesis-configuration.md) describes the main elements of the genesis configuration.
- [Build process](./build-process.md) delves into the details of how the Rust code compiles to a Rust binary and a WebAssembly target and how these two targets are used to optimize node operations.
- [Build a deterministic runtime](./build-a-deterministic-runtime.md) explains how to use the Substrate runtime toolbox (`srtool`) and Docker to build the WebAssembly runtime for Substrate-based chains.
- [Troubleshoot your code](./troubleshoot-your-code.md) highlights general and Substrate-specific coding techniques for troubleshooting issues and following best practices.
