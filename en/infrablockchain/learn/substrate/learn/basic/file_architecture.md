---
title: FRAME File Architecture
description:
keywords:
---

As explained in [Architecture and Rust Libraries](./architecture.md), Substrate libraries use naming conventions to indicate whether they belong to the Substrate core client (`sc*\*`), FRAME and runtime (`frame*\_` and `pallet\_\_`), or Substrate raw (`sp*\*`) categories.

![Client and Runtime](/media/images/docs/infrablockchain/learn/substrate/learn/libraries.png)

If a specific crate is not explicitly mentioned, use the following link as an entry point to explore the core Rust libraries.

## Substrate Core Client Libraries

Explore Substrate libraries belonging to the Substrate core client (`sc_*`) using the following link.

- [`sc_authority_discovery`](https://paritytech.github.io/substrate/master/sc_authority_discovery/index.html)
- [`sc_block_builder`](https://paritytech.github.io/substrate/master/sc_block_builder/index.html)
- [`sc_chain_spec`](https://paritytech.github.io/substrate/master/sc_chain_spec/index.html)
- [`sc_cli`](https://paritytech.github.io/substrate/master/sc_cli/index.html)
- [`sc_client_api`](https://paritytech.github.io/substrate/master/sc_client_api/index.html)
- [`sc_client_db`](https://paritytech.github.io/substrate/master/sc_client_db/index.html)
- [`sc_consensus`](https://paritytech.github.io/substrate/master/sc_consensus/index.html)
- [`sc_network`](https://paritytech.github.io/substrate/master/sc_network/index.html)
- [`sc_rpc`](https://paritytech.github.io/substrate/master/sc_rpc/index.html)
- [`sc_service`](https://paritytech.github.io/substrate/master/sc_service/index.html)
- [`sc_state_db`](https://paritytech.github.io/substrate/master/sc_state_db/index.html)
- [`sc_transaction_pool`](https://paritytech.github.io/substrate/master/sc_transaction_pool/index.html)

## FRAME Library

Use the following link to explore the core FRAME libraries that are used in the Substrate runtime (`frame_* ` and `pallet_*`).

- [`frame_benchmarking`](https://paritytech.github.io/substrate/master/frame_benchmarking/index.html)
- [`frame_executive`](https://paritytech.github.io/substrate/master/frame_executive/index.html)
- [`frame_remote_externalities`](https://paritytech.github.io/substrate/master/frame_remote_externalities/index.html)
- [`frame_support`](https://paritytech.github.io/substrate/master/frame_support/index.html)
- [`frame_system`](https://paritytech.github.io/substrate/master/frame_system/index.html)
- [`pallet_assets`](https://paritytech.github.io/substrate/master/pallet_assets/index.html)
- [`pallet_balances`](https://paritytech.github.io/substrate/master/pallet_balances/index.html)
- [`pallet_collective`](https://paritytech.github.io/substrate/master/pallet_collective/index.html)
- [`pallet_identity`](https://paritytech.github.io/substrate/master/pallet_identity/index.html)
- [`pallet_membership`](https://paritytech.github.io/substrate/master/pallet_membership/index.html)
- [`pallet_proxy`](https://paritytech.github.io/substrate/master/pallet_proxy/index.html)

## Substrate Primitive Library

다음 링크를 사용하여 Substrate 원시(raw) 라이브러리(`sp_*`)를 탐색하세요.

- [`sp_api`](https://paritytech.github.io/substrate/master/sp_api/index.html)
- [`sp_blockchain`](https://paritytech.github.io/substrate/master/sp_blockchain/index.html)
- [`sp_core`](https://paritytech.github.io/substrate/master/sp_core/index.html)
- [`sp_io`](https://paritytech.github.io/substrate/master/sp_io/index.html)
- [`sp_runtime`](https://paritytech.github.io/substrate/master/sp_runtime/index.html)
- [`sp_state_machine`](https://paritytech.github.io/substrate/master/sp_state_machine/index.html)
- [`sp_storage`](https://paritytech.github.io/substrate/master/sp_storage/index.html)

## Other

Use the following link to explore other libraries.

- [`kitchensink_runtime`](https://paritytech.github.io/substrate/master/kitchensink_runtime)
- [`node_primitives`](https://paritytech.github.io/substrate/master/node_primitives/index.html)
- [`node_rpc`](https://paritytech.github.io/substrate/master/node_rpc/index.html)
- [`node_template`](https://paritytech.github.io/substrate/master/node_template/index.html)
- [`node_template_runtime`](https://paritytech.github.io/substrate/master/node_template_runtime/index.html)
