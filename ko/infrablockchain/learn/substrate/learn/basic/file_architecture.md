---
title: FRAME 파일 구조
description: Substrate 의 기본 파일 구조에 대해 배웁니다.
keywords:
  - Rust 문서
  - 크레이트
  - 프로그래밍
---

[아키텍처와 Rust 라이브러리](../basic/architecture.md)에서 설명한 대로, Substrate 라이브러리는 Substrate 코어 클라이언트(`sc_*`), FRAME 및 런타임(`frame_*` 및 `pallet_*`), 또는 Substrate 원시(raw)(`sp_*`)인지를 나타내기 위해 네이밍 규칙을 사용합니다.

![외부 노드 및 런타임을 위한 코어 노드 라이브러리](/media/images/docs/infrablockchain/learn/substrate/learn/libraries.png)

특정 크레이트를 명확히 하지 않은 경우, 다음 링크를 사용하여 핵심 Rust 라이브러리를 탐색하는 진입점으로 사용하세요.

## Substrate 코어 클라이언트 라이브러리

다음 링크를 사용하여 Substrate 코어 클라이언트(`sc_*`)에 속하는 Substrate 라이브러리를 탐색하세요.

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

## FRAME 라이브러리

다음 링크를 사용하여 Substrate 런타임(`frame_*` 및 `pallet_*`)에서 사용되는 핵심 FRAME 라이브러리를 탐색하세요.

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

## Substrate 원시(raw) 라이브러리

다음 링크를 사용하여 Substrate 원시(raw) 라이브러리(`sp_*`)를 탐색하세요.

- [`sp_api`](https://paritytech.github.io/substrate/master/sp_api/index.html)
- [`sp_blockchain`](https://paritytech.github.io/substrate/master/sp_blockchain/index.html)
- [`sp_core`](https://paritytech.github.io/substrate/master/sp_core/index.html)
- [`sp_io`](https://paritytech.github.io/substrate/master/sp_io/index.html)
- [`sp_runtime`](https://paritytech.github.io/substrate/master/sp_runtime/index.html)
- [`sp_state_machine`](https://paritytech.github.io/substrate/master/sp_state_machine/index.html)
- [`sp_storage`](https://paritytech.github.io/substrate/master/sp_storage/index.html)

## 기타 라이브러리

다음 링크를 사용하여 기타 라이브러리를 탐색하세요.

- [`kitchensink_runtime`](https://paritytech.github.io/substrate/master/kitchensink_runtime)
- [`node_primitives`](https://paritytech.github.io/substrate/master/node_primitives/index.html)
- [`node_rpc`](https://paritytech.github.io/substrate/master/node_rpc/index.html)
- [`node_template`](https://paritytech.github.io/substrate/master/node_template/index.html)
- [`node_template_runtime`](https://paritytech.github.io/substrate/master/node_template_runtime/index.html)