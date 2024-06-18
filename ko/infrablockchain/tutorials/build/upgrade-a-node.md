---
title: Asynchronous Backing 적용하기
description: Inclusion 과 Backing 을 분리하여 파라체인 throughput 을 증가시킬 수 있는 async backing 을 적용하는 방법에 대해 설명합니다.
keywords:
  - Async Backing
---

## 개요 

기존 릴레이 체인의 파라체인 블록은 *Inclusion* 과 *Backing* 파이프라인이 서로 의존적(_a.k.a Synchronous Backing_) 으로 진행되어 파라체인 블록이 생성되고 다음 블록이 생성되는 시간에 큰 딜레이(e.g 파라체인 블록 생성 시간 = 12초)가 있었습니다. 이를 보안하기 위해 두 파이프라인을 분리하여 비동기적으로 파라체인 블록을 검증할 수 있는 프로토콜인 **Asynchronous Backing** 을 적용하여 기존 파라체인 노드를 업그레이드 하는 방법에 대해 설명합니다.

## 파라체인 업데이트

### Phase 1 - 파라체인 런타임 업데이트

- `ConsensusHook` 타입 

  ```rust
  // runtime/src/lib.rs

  pub const UNINCLUDED_SEGMENT_CAPACITY: u32 = 3;

  pub const BLOCK_PROCESSING_VELOCITY: u32 = 1;

  pub const RELAY_CHAIN_SLOT_DURATION_MILLIS: u32 = 6000;

  ```

  `UNINCLUDED_SEGMENT_CAPACITY` : 릴레이 체인에 의해 확정되지 않았지만 생성할 수 있는 파라체인 블록 수. 여기서는 `3` 으로 설정해줍니다. `3` 의 의미는 확정되지 않은 블록을 최대 3개 까지 생성할 수 있다는 의미입니다. 

  `BLOCK_PROCESSING_VELOCITY` : 하나의 릴레이 체인 블록에 생성할 수 있는 파라체인 블록 수. 여기서는 `1` 로 설정해줍니다. `1` 의 의미는 릴레이 체인 블록에 anchoring 할 수 있는 파라체인 블록이 최대 2 개라는 의미입니다.

- `cumulus_pallet_parachain_system` 설정

  ```rust
  impl cumulus_pallet_parachain_system::Config for Runtime {
    ...
    type CheckAssociatedRelayNumber = RelayNumberMonotonicallyIncreases;
    type ConsensusHook = ConsensusHook;
  }
  ```
  `CheckAssociatedRelayNumber` : 릴레이 체인 블록 정책입니다. 여기서는 `RelayNumberMonotonicallyIncreases` 로 설정해줍니다. 해당 타입의 의미는 파라체인이 바라보는 릴레이 체인 블록이 전 블록이 바롸봤던 릴레이 체인 블록 넘버와 같거나 커야함을 의미합니다. 

  `ConsensusHook` : 여기서는 위에서 정의했던 `ConsensusHook` 으로 설정해줍니다.

- `pallet_aura` 설정

```rust
impl pallet_aura::Config for Runtime {
    ...
    type AllowMultipleBlocksPerSlot = ConstBool<true>;
    type SlotDuration = ConstU64<6_000>;
}

// impl_runtime_apis!

impl sp_consensus_aura::AuraApi<Block, AuraId> for Runtime {
        fn slot_duration() -> sp_consensus_aura::SlotDuration {
            sp_consensus_aura::SlotDuration::from_millis(6_000)
        }

        ...
}
```

`AllowMultipleBlocksPerSlot` : 하나의 slot 에 여러 개의 블록 허용 여부를 나타냅니다. 여기서는 `true` 로 변경해줍니다.

`SlotDuration` : 한 슬롯의 기간을 의미. 여기서는 `6_000` 으로 설정하여 한 슬롯의 기간이 6초임을 나타내줍니다.

- `Cargo.toml` 에 `cumulus_primitives_aura` 추가

  ```rust
  // impl_runtime_apis!

  impl cumulus_primitives_aura::AuraUnincludedSegmentApi<Block> for Runtime {
    fn can_build_upon(
        included_hash: <Block as BlockT>::Hash,
        slot: cumulus_primitives_aura::Slot,
    ) -> bool {
        ConsensusHook::can_build_upon(included_hash, slot)
    }
  }
  ```

- `Maximum_Block_Weight` 를 2 초로 증가시키기 

  ```rust
  pub const MAXIMUM_BLOCK_WEIGHT: Weight = Weight::from_parts(
    WEIGHT_REF_TIME_PER_SECOND.saturating_mul(2),
    cumulus_primitives_core::relay_chain::MAX_POV_SIZE as u64,
  );
  ```

- `pallet_timestamp` 의 `MINIMUM_PERIOD` 을 `0` 으로 설정

  ```rust
  impl pallet_timestamp::Config for Runtime {
    ...
    type MinimumPeriod = ConstU64<0>;
  }
  ```

### Phase 2 - 파라체인 노드 업데이트

- `infra_parachain/src/service.rs` 파일에서 `basic_aura` 를 `lookahead_aura` 로 교체합니다. Node 의 param 부분에 정의되어있는 `authoring_duration` 을 `500` 에서 `Duration::from_millis(1500)` 으로 변경해줍니다. 그 후 `basic_aura::run` 에서 `aura::run` 으로 변경해줍니다. 

- `infra_parachain/src/command.rs` 로 가서 커맨드에 맞는 `start_*` 로 변경해줍니다(e.g `start_asset_hub_lookahead_node`)

```rust
use cumulus_client_consensus_aura::collators::lookahead::{self as aura, Params as AuraParams};

let params = AuraParams {
    ... 
    authoring_duration: Duration::from_millis(1500),
};

aura::run(params).await;
```

## 참고

- [Asynchronous Backing Wiki](https://wiki.polkadot.network/docs/learn-async-backing)
