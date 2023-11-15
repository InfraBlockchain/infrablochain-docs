---
title: 작업 증명을 사용하는 체인 구성
description:
keywords:
  - 노드
  - 클라이언트
  - 합의
  - 작업 증명
---

`basic-pow` 노드는 Substrate 기반 블록체인에 사용자 정의 합의 엔진을 추가하는 방법을 보여줍니다.
이 예제에서는 노드가 블록체인에 대한 합의를 도달하기 위해 최소한의 작업 증명 합의 엔진을 사용합니다.
이 가이드에서는 합의 엔진과 함께 작업하는 몇 가지 핵심 원칙을 소개합니다.

## 사용 사례

- 작업 증명 합의 엔진을 사용하는 체인을 시작합니다.
- 권한(Aura) 기반 합의 엔진에서 작업 증명(PoW) 합의 엔진으로 체인을 업그레이드합니다.

## 미리보기 단계

1. 작업 증명 합의를 사용하는 전체 노드를 정의합니다.
1. 데이터 제공자를 위한 가져오기 대기열을 생성합니다.
1. `proposer` 및 `worker` 함수를 정의합니다.
1. 경량 클라이언트 서비스를 정의합니다.

## sc_consensus_pow 및 sc_service를 사용하여 전체 노드 정의

`src/service.rs`에서 [`PartialComponents`](https://paritytech.github.io/substrate/master/sc_service/struct.PartialComponents.html) 및 [`PowBlockImport`](https://paritytech.github.io/substrate/master/sc_consensus_pow/struct.PowBlockImport.html)를 정의하는 `new_full` 함수를 작성합니다:

```rust
let pow_block_import = sc_consensus_pow::PowBlockImport::new(
  client.clone(),
  client.clone(),
  sha3pow::MinimalSha3Algorithm,
  0,                              // 블록 0부터 inherents 확인
  select_chain.clone(),
  inherent_data_providers.clone(),
  can_author_with,
);

let import_queue = sc_consensus_pow::import_queue(
  Box::new(pow_block_import.clone()),
  None,
  sha3pow::MinimalSha3Algorithm,  // 클라이언트에 대한 참조를 제공합니다.
  inherent_data_providers.clone(),
  &task_manager.spawn_handle(),
  config.prometheus_registry(),
)?;
```

## 가져오기 대기열 생성

POW 시스템의 제공자를 정의하는 함수에서 [`InherentDataProviders`](https://paritytech.github.io/substrate/master/sc_consensus_aura/struct.InherentDataProvider.html)를 사용하여 노드의 inherents를 정의합니다:

```rust
pub fn build_inherent_data_providers() -> Result<InherentDataProviders, ServiceError> {
  let providers = InherentDataProviders::new();

  providers
    .register_provider(sp_timestamp::InherentDataProvider)
    .map_err(Into::into)
    .map_err(sp_consensus::error::Error::InherentData)?;

    Ok(providers)
}
```

## proposer 및 worker 정의

`new_full` 함수에서 `proposer`를 정의합니다:

```rust
let proposer = sc_basic_authorship::ProposerFactory::new(
    task_manager.spawn_handle(),
    client.clone(),
    transaction_pool,
    prometheus_registry.as_ref(),
);

let (_worker, worker_task) = sc_consensus_pow::start_mining_worker(
    Box::new(pow_block_import),
    client,
    select_chain,
    MinimalSha3Algorithm,
    proposer,
    network.clone(),
    None,
    inherent_data_providers,
    // 새 블록을 채굴하기 전에 대기하는 시간
    Duration::from_secs(10),
    // 블록을 실제로 빌드하는 데 걸리는 시간 (예: extrinsics 실행)
    Duration::from_secs(10),
    can_author_with,
);
```

작업 관리자가 실행하도록 합니다:

```rust
task_manager
    .spawn_essential_handle()
    .spawn_blocking("pow", worker_task);
```

## 예제

- [기본 POW 예제 노드](https://github.com/substrate-developer-hub/substrate-how-to-guides/tree/main/example-code/consensus-nodes/POW)

## 자원

- [Partial components](https://paritytech.github.io/substrate/master/sc_service/struct.PartialComponents.html)
- [Pow block import](https://paritytech.github.io/substrate/master/sc_consensus_pow/struct.PowBlockImport.html)
- [Inherent data providers 생성](https://paritytech.github.io/substrate/master/sp_inherents/trait.CreateInherentDataProviders.html)
- [Pow algorithm](https://paritytech.github.io/substrate/master/sc_consensus_pow/trait.PowAlgorithm.html)