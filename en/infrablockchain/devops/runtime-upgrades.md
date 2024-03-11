---
title: Runtime Upgrade
description: This document explains how to support forkless upgrades in Substrate-based networks through runtime versioning and storage migration.
keywords:
---

Forkless runtime upgrade is a core feature of the Substrate framework for blockchain development. The ability to update runtime logic without forking the codebase allows the blockchain to evolve and improve over time. This feature is made possible by including the definition of the runtime WebAssembly blob, which is a runtime state element, in the blockchain's runtime state.

Since the runtime is part of the blockchain state, network administrators can safely improve the runtime using a trusted decentralized consensus mechanism.

In the FRAME system for runtime development, the `set_code` call is defined, which is used to update the runtime definition. The [Upgrade a Running Network](/tutorials/build-a-blockchain/upgrade-a-running-network/) tutorial demonstrates two methods to upgrade the runtime without shutting down nodes or interrupting operations. However, both upgrades in the tutorial only demonstrate adding functionality to the runtime, not updating the existing runtime state. If a runtime upgrade requires changes to the existing state, storage migration is likely necessary.

### Runtime Versioning

In the [build process](/main-docs/build/build-process/), we learned that compiling a node generates platform-specific binaries and WebAssembly binaries, and we can control which binaries to use at different points in the block production process through command-line options for execution strategy. The component used to communicate with the runtime execution environment is called the **executor**. While it is possible to override the default execution strategy in custom scenarios, in most cases, the executor evaluates the following information from the native runtime and WebAssembly runtime to select the binary to use:

- `spec_name`
- `spec_version`
- `authoring_version`

To provide this information to the executor process, the runtime includes a [runtime version struct](https://paritytech.github.io/substrate/master/sp_version/struct.RuntimeVersion.html) similar to the following:

```rust
pub const VERSION: RuntimeVersion = RuntimeVersion {
  spec_name: create_runtime_str!("node-template"),
  impl_name: create_runtime_str!("node-template"),
  authoring_version: 1,
  spec_version: 1,
  impl_version: 1,
  apis: RUNTIME_API_VERSIONS,
  transaction_version: 1,
};
```

The parameters of the struct provide the following information:

| Parameter           | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `spec_name`         | An identifier that distinguishes different Substrate runtimes.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `impl_name`         | The implementation name of the specification. This is not significant for nodes but is used to differentiate code from different implementation teams.                                                                                                                                                                                                                                                                                                                                                                                      |
| `authoring_version` | If the authoring node does not have the same value as the native runtime, it will not produce blocks.                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `spec_version`      | The version of the runtime specification. The overall node does not use the WebAssembly runtime instead of the native runtime unless `spec_name`, `spec_version`, and `authoring_version` are the same in both WebAssembly and native binaries. Updating `spec_version` can be automated through a CI process and is done in the [Polkadot network](https://gitlab.parity.io/parity/mirrors/polkadot/-/blob/master/scripts/ci/gitlab/check_extrinsics_ordering.sh). This parameter typically increases when there is an update to `transaction_version`. |
| `impl_version`      | The implementation version of the specification. The node ignores this value. It is used to indicate that the code may be different. The native and WebAssembly binaries perform the same operations even if `authoring_version` and `spec_version` are the same. Changes to `impl_version` typically occur due to optimizations that do not break logic.                                                                                                                                                                                  |
| `transaction_version` | The version of the transaction processing interface. This parameter is useful for hardware wallets to synchronize firmware updates to ensure secure signing of runtime transactions. This number should be updated when there are changes to the index of palettes or the parameters or parameter types of dispatchable functions in the `construct_runtime!` macro. When this number is updated, `spec_version` should also be updated. |
| `apis`              | A list of supported [runtime APIs](https://paritytech.github.io/substrate/master/sp_api/macro.impl_runtime_apis.html) and their versions.                                                                                                                                                                                                                                                                                                                                                                                                                                                             |

The orchestration engine(sometimes referred to as the executor) verifies that the native runtime has the same consensus logic as the WebAssembly before executing it. However, since runtime versioning is manually set, if the runtime version is incorrectly indicated, the orchestration engine may make inappropriate decisions.

### Accessing the Runtime Version

The FRAME system exposes the runtime version information through the `state.getRuntimeVersion` RPC endpoint. This endpoint allows an optional block identifier. However, in most cases, the runtime [metadata](/main-docs/build/application-development/#exposing-runtime-information-as-metadata) is used to understand the APIs exposed by the runtime and how to interact with them. The runtime metadata should only be changed when the [runtime's spec_version](https://paritytech.github.io/substrate/master/sp_version/struct.RuntimeVersion.html#structfield.spec_version) changes.

### Forkless Runtime Upgrade

Traditional blockchains require a [hard fork](<https://en.wikipedia.org/wiki/Fork_(blockchain)>) when upgrading the state transition function, which means all node operators need to stop their nodes and manually upgrade to the latest executable. Coordinating a hard fork upgrade in a distributed production network can be a complex process.

The runtime versioning properties allow Substrate-based blockchains to upgrade the runtime logic in real-time without forking the network.

To perform a runtime upgrade without forking, Substrate updates the WebAssembly runtime stored on the blockchain with a new version of logic that breaks the existing consensus. This upgrade is propagated to all full nodes in the network as part of the consensus process. When the WebAssembly runtime is upgraded, the orchestration engine detects that the `spec_name`, `spec_version`, or `authoring_version` of the native runtime does not match the new WebAssembly runtime. As a result, the orchestration engine executes the normative WebAssembly runtime instead of the native runtime in any execution process.

## Storage Migration

**Storage migration** is a custom one-time function that updates the storage to adapt to changes in the runtime. For example, if a runtime upgrade changes the data type representing user balances from an **unsigned** integer to a **signed** integer, the storage migration reads the existing value as an unsigned integer and then writes the updated value converted to a signed integer. If such a change in data storage is required and not performed, the runtime cannot interpret the storage values correctly and may result in undefined behavior.

### Storage migration using FRAME

FRAME storage migration is implemented using the [`OnRuntimeUpgrade`](https://paritytech.github.io/substrate/master/frame_support/traits/trait.OnRuntimeUpgrade.html) trait. The `OnRuntimeUpgrade` trait defines a single function called `on_runtime_upgrade` that specifies the logic to be executed immediately after a runtime upgrade.

### Preparing for Storage Migration

Preparing for storage migration means understanding the changes defined by the runtime upgrade. The Substrate repository uses the [`E1-runtimemigration`](https://github.com/paritytech/substrate/pulls?q=is%3Apr+label%3AE1-runtimemigration) label to specify such changes.

### Writing Migrations

Each storage migration is different based on requirements and complexity. However, when performing a storage migration, the following recommendations can be used as guidance:

- Extract the migration into a reusable function and write tests for that function.
- Include logging in the migration for debugging purposes.
- The migration code should include deprecated types within the context of the upgraded runtime, as shown in [this example](https://github.com/hicommonwealth/substrate/blob/5f3933f5735a75d2d438341ec6842f269b886aaa/frame/indices/src/migration.rs#L5-L22).
- Use storage versions to make the migration more declarative and safer, as shown in [this example](https://github.com/paritytech/substrate/blob/c79b522a11bbc7b3cf2f4a9c0a6627797993cb79/frame/elections-phragmen/src/lib.rs#L119-L157).

### Migration Order

By default, FRAME orders the execution of the `on_runtime_upgrade` functions based on the order in which the palettes appear in the `construct_runtime!` macro. For upgrades, the functions are executed in reverse order, starting from the last palette. Custom order can also be specified if needed, as shown in [this example](https://github.com/hicommonwealth/edgeware-node/blob/7b66f4f0a9ec184fdebcccd41533acc728ebe9dc/node/runtime/src/lib.rs#L845-L866).

FRAME storage migrations are executed in the following order:

1. Custom `on_runtime_upgrade` function if using a custom order.
2. System `frame_system::on_runtime_upgrade` function.
3. All `on_runtime_upgrade` functions defined in the `construct_runtime!` macro, starting from the last palette.

### Migration Testing

Testing storage migration is important. Some tools that can be used to test storage migration are:

- The [Substrate Debug Kit](https://github.com/paritytech/substrate-debug-kit) includes the [remote externalities](https://github.com/paritytech/substrate-debug-kit/tree/master/remote-externalities) tool, which allows safe unit testing of storage migration with live chain data.
- The [fork-off-substrate](https://github.com/maxsam4/fork-off-substrate) script makes it easy to generate chain specs to bootstrap a local test chain for testing runtime upgrades and storage migration.


#### Next Steps

- [Upgrade a Running Network](/tutorials/build-a-blockchain/upgrade-a-running-network/)
- [Substrate Migrations](https://github.com/apopiak/substrate-migrations)