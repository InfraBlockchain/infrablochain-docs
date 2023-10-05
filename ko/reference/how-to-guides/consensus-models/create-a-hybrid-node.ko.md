---
title: 하이브리드 노드 생성하기
description: SHA3 작업 증명을 사용하여 블록 생성을 지시하고 [Grandpa](https://paritytech.github.io/substrate/master/sc_consensus_grandpa/index.html) 최종성을 제공하는 Substrate 기반 노드를 생성하는 방법을 설명합니다.
keywords:
  - 합의
  - 작업 증명
  - PoW
---

이 가이드는 SHA3 작업 증명을 사용하여 블록 생성을 지시하고 [Grandpa](https://paritytech.github.io/substrate/master/sc_consensus_grandpa/index.html) 최종성 기능을 제공하는 Substrate 기반 노드를 생성하는 방법을 보여줍니다. 최소한의 작업 증명 합의는 런타임 외부에 위치합니다.

Grandpa 최종성은 [Grandpa API](https://paritytech.github.io/substrate/master/sc_consensus_grandpa/trait.GrandpaApi.html)를 사용하여 런타임으로부터 권한 집합을 가져옵니다. 따라서 이 가이드를 구현하는 노드를 성공적으로 컴파일하려면 이 API를 제공하는 런타임이 필요합니다.

## 사용 사례

Substrate 체인의 합의 메커니즘을 사용자 정의합니다.

## 단계 미리보기

1. 블록 가져오기 파이프라인 구성하기
1. 가져오기 대기열 생성하기
1. 작업 증명 블록 생성 작업 생성하기
1. Grandpa 작업 생성하기

## 블록 가져오기 파이프라인 구성하기

먼저 Grandpa를 위한 블록 가져오기를 생성합니다. 블록 가져오기 자체 외에도 `grandpa_link`를 얻습니다. 이 링크는 블록 가져오기가 Grandpa 투표를 실제로 전달하는 백그라운드 작업과 통신하기 위한 채널입니다. [Grandpa 프로토콜의 세부 정보](https://research.web3.foundation/en/latest/polkadot/finality.html)는 이 가이드의 범위를 벗어납니다.

`node/src/service.rs`에서 Grandpa 블록 가져오기를 생성합니다:

```rust
let (grandpa_block_import, grandpa_link) = sc_finality_grandpa::block_import(
	client.clone(),
	&(client.clone() as std::sync::Arc<_>),
	select_chain.clone(),
)?;
```

Grandpa 블록 가져오기가 생성되었으므로 이제 PoW 블록 가져오기를 생성할 수 있습니다. Pow 블록 가져오기는 블록 가져오기의 가장 바깥쪽 레이어이며 grandpa 블록 가져오기를 감싸고 있습니다.

```rust
let pow_block_import = sc_consensus_pow::PowBlockImport::new(
	grandpa_block_import,
	client.clone(),
	sha3pow::MinimalSha3Algorithm,
	0, // 블록 0부터 상속성 확인
	select_chain.clone(),
	inherent_data_providers.clone(),
	can_author_with,
);
```

## 가져오기 대기열 생성하기

블록 가져오기가 설정되었으므로 이제 가져오기 대기열을 생성할 수 있습니다. PoW의 [`import_queue` 도우미 함수](https://paritytech.github.io/substrate/master/sc_consensus_pow/fn.import_queue.html)를 사용하여 대기열을 만듭니다. PoW가 가장 바깥쪽 레이어이므로 전체 블록 가져오기 파이프라인을 `pow_block_import`로 참조합니다.

```rust
let import_queue = sc_consensus_pow::import_queue(
	Box::new(pow_block_import.clone()),
	None,
	sha3pow::MinimalSha3Algorithm,
	inherent_data_providers.clone(),
	&task_manager.spawn_handle(),
	config.prometheus_registry(),
)?;
```

## 작업 증명 작성 작업 생성하기

일반적으로 "마이너"라고 불리는 권한을 가진 노드는 작업 증명 컨텍스트에서 생성되는 마이닝 워커를 실행해야 합니다. 이 워커는 작업 관리자에 의해 생성됩니다.

```rust
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
	// 블록을 실제로 빌드하는 데 걸리는 시간 (즉, extrinsic 실행)
	Duration::from_secs(10),
	can_author_with,
);

task_manager
	.spawn_essential_handle()
	.spawn_blocking("pow", worker_task);
```

## Grandpa 작업 생성하기

Grandpa는 CPU 집약적이지 않으므로 표준 `async` 워커를 사용하여 Grandpa 투표를 수신하고 전달하는 작업을 수행합니다. 먼저 Grandpa [`Config`](https://paritytech.github.io/substrate/master/sc_consensus_grandpa/struct.Config.html)를 생성합니다.

```rust
let grandpa_config = sc_finality_grandpa::Config {
	gossip_duration: Duration::from_millis(333),
	justification_period: 512,
	name: None,
	observer_enabled: false,
	keystore: Some(keystore_container.sync_keystore()),
	is_authority,
};
```

이 구성을 사용하여 [`GrandpaParams`](https://paritytech.github.io/substrate/master/sc_consensus_grandpa/struct.GrandpaParams.html)의 인스턴스를 생성할 수 있습니다.

```rust
let grandpa_config = sc_finality_grandpa::GrandpaParams {
	config: grandpa_config,
	link: grandpa_link,
	network,
	telemetry_on_connect: telemetry_connection_notifier.map(|x| x.on_connect_stream()),
	voting_rule: sc_finality_grandpa::VotingRulesBuilder::default().build(),
	prometheus_registry,
	shared_voter_state: sc_finality_grandpa::SharedVoterState::empty(),
};
```

매개변수가 설정되었으므로 이제 작성 작업을 생성하고 실행할 수 있습니다.

```rust
task_manager.spawn_essential_handle().spawn_blocking(
	"grandpa-voter",
	sc_finality_grandpa::run_grandpa_voter(grandpa_config)?
);
```

## 예제

- [하이브리드 합의](https://github.com/substrate-developer-hub/recipes/blob/master/nodes/hybrid-consensus/src/service.rs)

## 자원

- [POW 알고리즘](https://paritytech.github.io/substrate/master/sc_consensus_pow/trait.PowAlgorithm.html) 트레이트
- [`PowBlockimport`](https://paritytech.github.io/substrate/master/sc_consensus_pow/struct.PowBlockImport.html)
- [상속성](https://paritytech.github.io/substrate/master/sp_inherents/index.html)