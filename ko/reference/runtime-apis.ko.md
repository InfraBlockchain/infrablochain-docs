---
title: 런타임 API
description: 외부 노드 서비스와의 통신을 가능하게 하는 런타임 인터페이스를 강조합니다.
keywords:
---

[아키텍처](/learn/architecture)에서 설명한 대로, Substrate 노드는 외부 노드 서비스와 런타임으로 구성되며, 이 책임의 분리는 Substrate 기반 체인을 설계하고 업그레이드 가능한 로직을 구축하는 데 중요한 개념입니다.
그러나 외부 노드 서비스와 런타임은 데이터 읽기 및 쓰기, 상태 전이 등 많은 중요한 작업을 완료하기 위해 서로 통신해야 합니다.
외부 노드 서비스는 특정 작업을 수행하기 위해 런타임 응용 프로그래밍 인터페이스를 호출하여 런타임과 통신합니다.
기본적으로 Substrate 런타임은 외부 노드 서비스가 호출할 수 있는 다음 특성을 제공합니다:

- [`AccountNonceApi`](https://paritytech.github.io/substrate/master/frame_system_rpc_runtime_api/trait.AccountNonceApi.html)
- [`AuraApi`](https://paritytech.github.io/substrate/master/sp_consensus_aura/trait.AuraApi.html)
- [`Benchmark`](https://paritytech.github.io/substrate/master/frame_benchmarking/trait.Benchmark.html)
- [`BlockBuilder`](https://paritytech.github.io/substrate/master/sp_block_builder/trait.BlockBuilder.html)
- [`GrandpaApi`](https://paritytech.github.io/substrate/master/sp_consensus_grandpa/trait.GrandpaApi.html)
- [`NominationPoolsApi`](https://paritytech.github.io/substrate/master/pallet_nomination_pools_runtime_api/trait.NominationPoolsApi.html)
- [`OffchainWorkerApi`](https://paritytech.github.io/substrate/master/sp_offchain/trait.OffchainWorkerApi.html)
- [`SessionKeys`](https://paritytech.github.io/substrate/master/sp_session/trait.SessionKeys.html)
- [`TaggedTransactionQueue`](https://paritytech.github.io/substrate/master/sp_transaction_pool/runtime_api/trait.TaggedTransactionQueue.html)
- [`TransactionPaymentApi`](https://paritytech.github.io/substrate/master/pallet_transaction_payment_rpc_runtime_api/trait.TransactionPaymentApi.html)

## AccountNonceApi

`AccountNonceApi`를 사용하여 지정된 계정 식별자에 대한 nonce를 가져옵니다.
각 계정의 nonce는 해당 계정이 트랜잭션을 완료하는 데 사용될 때마다 증가합니다.
따라서 nonce는 때로는 트랜잭션 인덱스로도 불리기도 합니다.

이 API는 다음 메서드를 제공합니다:

- `account_nonce` : 지정된 AccountId에 대한 현재 계정 nonce를 가져옵니다.
- `account_nonce_with_context` : 지정된 AccountId와 실행 컨텍스트에 대한 현재 계정 nonce를 가져옵니다.

## AuraApi

`AuraApi`를 사용하여 슬롯 기반 합의를 사용하는 블록 작성을 관리합니다.
대부분의 합의 관련 작업은 외부 노드 서비스에서 처리하지만, 런타임은 상태 전이 로직의 일부인 합의 관련 작업을 위해 이 API를 제공해야 합니다.

이 API는 권한 기반의 라운드 로빈 스케줄링 ([Aura](/reference/glossary/#aura))을 위한 다음 메서드를 제공합니다:

- `slot_duration` : Aura 합의에 대한 슬롯 기간을 가져옵니다.
- `slot_duration_with_context` : 지정된 실행 컨텍스트 내에서 Aura 합의에 대한 슬롯 기간을 가져옵니다.
- `authorities` : Aura 합의에 대한 권한 집합을 가져옵니다.
- `authorities_with_context` : 지정된 실행 컨텍스트 내에서 Aura 합의에 대한 권한 집합을 가져옵니다.

## Benchmark

`Benchmark` API를 사용하여 FRAME 런타임에서 [벤치마킹](/test/benchmark/) 함수 실행에 필요한 정보를 제공합니다.

이 API는 다음 메서드를 제공합니다:

- `benchmark_metadata` : 이 런타임에서 사용 가능한 벤치마크 메타데이터를 가져옵니다.
- `benchmark_metadata_with_context` : 지정된 실행 컨텍스트 내에서 사용 가능한 벤치마크 메타데이터를 가져옵니다.
- `dispatch_benchmark` : 지정된 벤치마크를 디스패치합니다.
- `dispatch_benchmark_with_context` : 지정된 벤치마크를 지정된 실행 컨텍스트 내에서 디스패치합니다.

## BlockBuilder

`BlockBuilder` API를 사용하여 블록을 구축하고 완료하는 데 필요한 기능을 제공합니다.
런타임은 트랜잭션의 유효성을 확인하고 블록을 구성하기 위해 트랜잭션을 실행하는 책임이 있습니다.
외부 노드에 대해서는 트랜잭션은 불투명한 벡터 배열 (Vec<u8>)입니다.

이 API는 다음 메서드를 제공합니다:

- `apply_extrinsic` : 지정된 외부 트랜잭션을 현재 블록에 포함합니다.
  이 메서드는 또한 트랜잭션이 블록에 포함되었는지 여부를 나타내는 결과를 반환합니다.
- `apply_extrinsic_with_context` : 지정된 외부 트랜잭션을 현재 블록과 지정된 실행 컨텍스트에 포함합니다.
  이 메서드는 또한 트랜잭션이 블록에 포함되었는지 여부를 나타내는 결과를 반환합니다.
- `finalize_block` : 현재 블록의 구축을 완료합니다.
- `finalize_block_with_context` : 지정된 실행 컨텍스트 내에서 현재 블록의 구축을 완료합니다.
- `inherent_extrinsics` : 현재 블록에 내재된 트랜잭션을 포함합니다.
  내재 트랜잭션 유형은 체인마다 다릅니다.
- `inherent_extrinsics_with_context` : 현재 블록과 지정된 실행 컨텍스트에 내재된 트랜잭션을 포함합니다.
  내재 트랜잭션 유형은 체인마다 다릅니다.
- `check_inherents` : 내재 트랜잭션이 유효한지 확인합니다.
- `check_inherents_with_context` : 지정된 실행 컨텍스트 내에서 내재 트랜잭션이 유효한지 확인합니다.

## GrandpaApi

`GrandpaApi`를 사용하여 GRANDPA 최종화 프로토콜에서 권한 집합 변경을 런타임에 통합합니다.
GRANDPA 최종화 프로토콜은 일정한 블록 수의 지연을 지정하여 권한 집합의 변경을 신호로 보냅니다.
지정된 블록 수가 최종화된 후에 변경 사항이 자동으로 런타임에 적용됩니다.

이 API는 다음 메서드를 제공합니다:

- `grandpa_authorities` : GRANDPA 최종화를 위한 현재 권한과 가중치를 가져옵니다.
- `grandpa_authorities_with_context` : 지정된 실행 컨텍스트 내에서 GRANDPA 최종화를 위한 현재 권한과 가중치를 가져옵니다.
- `current_set_id` : 현재 GRANDPA 권한 집합 식별자를 가져옵니다.
- `current_set_id_with_context` : 지정된 실행 컨텍스트 내에서 현재 GRANDPA 권한 집합 식별자를 가져옵니다.

`GrandpaApi`는 또한 잘못된 동작에 대한 증거 및 관련 키 소유 증명을 보고하기 위해 트랜잭션을 제출하는 메서드도 제공합니다.
이러한 메서드에 대한 정보는 [`GrandpaApi`](https://paritytech.github.io/substrate/master/sp_consensus_grandpa/trait.GrandpaApi.html)를 참조하세요.

## NominationPoolsApi

`NominationPoolsApi`를 사용하여 노미네이션 풀 및 노미네이션 풀 멤버에 대한 정보를 얻을 수 있습니다.

이 API는 다음 메서드를 제공합니다:

- `pending_rewards` : 지정된 AccountId의 노미네이션 풀 멤버에 대한 보류 중인 보상을 가져옵니다.
- `pending_rewards_with_context` : 지정된 AccountId와 실행 컨텍스트 내에서 노미네이션 풀 멤버에 대한 보류 중인 보상을 가져옵니다.

## OffchainWorkerApi

`OffchainWorkerApi`를 사용하여 [오프체인 워커 작업](/learn/offchain-operations/)을 시작합니다.

이 API는 다음 메서드를 제공합니다:

- `offchain_worker` : 지정된 블록 헤더에 대한 오프체인 작업을 시작합니다.
- `offchain_worker_with_context` : 지정된 블록 헤더와 실행 컨텍스트에 대한 오프체인 작업을 시작합니다.

## SessionKeys

`SessionKeys` API를 사용하여 [세션 키](/learn/accounts-addresses-keys/)를 생성하고 디코딩합니다.
(https://paritytech.github.io/substrate/master/sp_session/trait.SessionKeys.html)

이 API는 다음 메서드를 제공합니다:

- `generate_session_keys` : 세트의 세션 키를 생성합니다.
  지정된 시드를 사용하여 키를 생성하는 경우 시드는 유효한 `utf8` 문자열이어야 합니다.
  생성된 키를 런타임 외부성이 노출하는 키스토어에 저장해야 합니다.
  이 메서드는 연결된 SCALE 인코딩 값으로 된 공개 키를 반환합니다.

- `generate_session_keys_with_context` : 지정된 실행 컨텍스트 내에서 세트의 세션 키를 생성합니다.
  지정된 시드를 사용하여 키를 생성하는 경우 시드는 유효한 `utf8` 문자열이어야 합니다.
  생성된 키를 런타임 외부성이 노출하는 키스토어에 저장해야 합니다.
  이 메서드는 연결된 SCALE 인코딩 값으로 된 공개 키를 반환합니다.

- `decode_session_keys` : 지정된 공개 세션 키를 디코딩합니다.
  이 메서드는 원시 공개 키 목록과 키 유형을 반환합니다.

- `decode_session_keys_with_context` : 지정된 실행 컨텍스트 내에서 지정된 공개 세션 키를 디코딩합니다.
  이 메서드는 원시 공개 키 목록과 키 유형을 반환합니다.

## TaggedTransactionQueue

`TaggedTransactionQueue` API를 사용하여 트랜잭션 큐에서 트랜잭션을 유효성 검사합니다.

이 API는 다음 메서드를 제공합니다:

- `validate_transaction` : `block_hash` 매개변수로 지정된 상태에 대해 지정된 트랜잭션이 유효한 트랜잭션인지 확인합니다.

- `validate_transaction_with_context` : `block_hash` 매개변수로 지정된 상태에 대해 지정된 트랜잭션이 유효한 트랜잭션인지 확인합니다. 지정된 실행 컨텍스트 내에서 확인합니다.

## TransactionPaymentApi

`TransactionPaymentApi`를 사용하여 트랜잭션 및 트랜잭션 수수료에 대한 정보를 런타임에 쿼리합니다.

이 API는 다음 메서드를 제공합니다:

- `query_info` : 런타임에 디스패치된 지정된 트랜잭션에 대한 정보를 반환합니다.
- `query_info_with_context` : 지정된 실행 컨텍스트 내에서 런타임에 디스패치된 지정된 트랜잭션에 대한 정보를 반환합니다.
- `query_fee_details` : 지정된 트랜잭션에 대한 트랜잭션 수수료에 대한 정보를 반환합니다.
- `query_fee_details_with_context` : 지정된 실행 컨텍스트 내에서 지정된 트랜잭션에 대한 트랜잭션 수수료에 대한 정보를 반환합니다.

## 다음으로 어디로 가야 할까요

- [런타임 개발](/learn/runtime-development/)
- [FRAME 매크로](/reference/frame-macros)
- [impl_runtime_apis](https://paritytech.github.io/substrate/master/sp_api/macro.impl_runtime_apis.html)