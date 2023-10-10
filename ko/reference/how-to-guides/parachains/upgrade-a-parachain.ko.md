---
title: 파라체인 업그레이드
description:
keywords:
  - 콜레이터
  - 파라체인
  - 업그레이드
  - 커뮬러스
  - 스토리지
  - 마이그레이션
---

이 가이드의 목표는 파라체인 개발자들이 런타임 업그레이드가 성공하는지 보장할 수 있도록 돕는 것입니다.

이 가이드에서는 다음을 설명합니다:

- 파라체인 런타임 수정하는 방법 (팔렛 추가/제거)
- 파라체인 스토리지 마이그레이션하는 방법

파라체인에서의 런타임 업그레이드는 _매우_ 엄격한 요구사항을 가지며, 이를 위해 중계 체인과 협력해야 하는 약간 다른 흐름이 필요합니다.
이로 인해 상태 전이 조정의 매우 제한된 성격과 블록 포함을 위한 **빠르고 간결한** 조정이 필요합니다.

## 계속하기 전에

파라체인의 런타임을 업그레이드하기 전에 다음을 확인하세요:

- 중계 체인에 연결되지 않은 Substrate 노드에서 [런타임 업그레이드](/maintain/runtime-upgrades)를 수행하는 방법에 익숙합니다.
- [로컬 중계 체인 준비](/tutorials/build-a-parachain/prepare-a-local-relay-chain) 단계를 따라하고, 파라체인 콜레이터 노드와 중계 체인 간의 상호작용에 익숙합니다.
- 테스트를 위해 `polkadot-launch`에 액세스할 수 있습니다.

## 업그레이드 방법 선택하기

기존 Substrate 체인에 대규모 상태가 있고 이를 다른 스토리지 형식으로 마이그레이션하는 경우, 모든 런타임 마이그레이션을 한 블록 내에서 실행하는 것이 불가능할 수 있습니다.
이 문제를 해결하기 위해 몇 가지 전략을 사용할 수 있습니다:

1. 마이그레이션할 스토리지 항목의 양이 두 개 또는 세 개의 블록 내에서 처리될 수 있는 경우, [스케줄러 팔렛](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/scheduler)을 사용하여 마이그레이션을 실행하여 블록 프로듀서에 관계없이 실행되도록 할 수 있습니다.

1. 버전별 스토리지를 사용하고, 아직 업그레이드되지 않은 스토리지 값에 액세스할 때에만 마이그레이션을 실행합니다.
   이는 사용자 간의 트랜잭션 수수료에 차이를 일으킬 수 있으며, 더 복잡한 런타임 코드로 이어질 수도 있습니다.
   그러나 적절하게 계량화된 경우 (가중치가 적절하게 벤치마킹되었을 경우) 이 접근 방식은 마이그레이션의 최소한의 다운타임을 보장합니다.

1. 마이그레이션을 여러 블록에 분할해야 하는 경우, 온체인 또는 오프체인으로 수행할 수 있습니다:

   - 온체인 멀티블록 마이그레이션은 시간에 따라 변경 사항을 대기열에 넣거나 스케줄러 팔렛을 사용하여 스토리지 청크를 한 번에 마이그레이션하는 사용자 정의 팔렛 로직이 필요합니다.

   - 런타임에 마이그레이션 코드를 추가하는 대신, 마이그레이션을 수동으로 오프체인에서 생성하고, 루트 권한을 가진 오리진을 통해 필요한 스토리지 항목을 추가하고 제거하기 위해 여러 `system.setStorage` 호출을 사용할 수 있습니다.
     만약 만들 수 있는 트랜잭션의 수에 제한이 있다면, 스케줄러를 통해 시간이 지남에 따라 여러 트랜잭션을 일괄 처리할 수 있습니다.

마이그레이션 전략이 확립되면, 실제로 진행하기 전에 비생산 테스트넷에서 마이그레이션을 테스트하여 작동하는지 확인해야 합니다.

제한된 네트워크에서 테스트하는 것은 많은 콜레이터와 검증자, 대역폭 및 지연 시간과 같은 제약 조건을 가진 실제 네트워크에서의 잠재적인 실패에 대비하는 데 도움이 됩니다.
테스트를 위해 실제 네트워크를 가능한 한 정확하게 모사할수록 런타임 업그레이드가 성공할 확신을 가질 수 있습니다.

## 업그레이드 플로우 승인 -> 실행하기

파라체인을 업그레이드하려면, 업그레이드가 발생하기 전에 중계 체인에게 체인의 런타임 업그레이드에 대해 알려주어야 합니다.
[커뮬러스](https://github.com/paritytech/polkadot-sdk/tree/master/cumulus) 라이브러리는 다음을 통해 중계 체인에게 예정된 업그레이드에 대해 알리는 데 도움을 줍니다:

1. **[`authorize_upgrade`](https://paritytech.github.io/cumulus/cumulus_pallet_parachain_system/pallet/struct.Pallet.html#method.authorize_upgrade)**를 사용하여 업그레이드의 해시를 제공하고 승인합니다.
1. **[`enact_authorized_upgrade`](https://paritytech.github.io/cumulus/cumulus_pallet_parachain_system/pallet/struct.Pallet.html#method.enact_authorized_upgrade)**를 사용하여 업그레이드를 위한 실제 코드를 제공합니다.

이 두 함수를 호출하면, 중계 체인에게 새로운 업그레이드가 예정되었음을 알릴 수 있습니다.

## 예시

- [Substrate 마이그레이션 예시 저장소](https://github.com/apopiak/substrate-migrations)
- [스테이킹 팔렛 마이그레이션 로직](https://github.com/paritytech/substrate/blob/6be513d663836c5c5b8a436f5712402a1c5365a3/frame/staking/src/lib.rs#L757)

## 자료

- [런타임 업그레이드](/maintain/runtime-upgrades)
- [Substrate에서 포크하기](https://github.com/maxsam4/fork-off-substrate)
- [`try-runtime`](/reference/command-line-tools/try-runtime)
- [`try-runtime` 비디오 워크샵](https://www.crowdcast.io/e/substrate-seminar/41)