---
title: 사용자 정의 저장소 항목 추가
description:
tutorial: 1
---

`collectibles` 팔레트가 유용하려면 생성된 수집품의 수와 각 수집품을 소유한 사람에 대한 정보를 저장해야 합니다.
저장할 정보를 결정한 후, 어떻게 저장할지 결정해야 합니다: 단일 값으로 저장할지 아니면 저장소 맵으로 저장할지.
이 워크샵에서는 상태를 추적하기 위해 세 가지 사용자 정의 저장소 항목을 생성합니다:

- 단일 값인 `CollectiblesCount`는 팔레트 내의 총 수집품 수를 추적합니다.
- Key-Value 쌍인 `CollectiblesMap`은 각 수집품에 연결된 속성을 해당 수집품의 고유 식별자에 매핑합니다.
- Key-Value 쌍인 `OwnerOfCollectibles`은 수집품을 소유한 사용자 계정에 매핑합니다.

Substrate가 사용하는 저장소 아키텍처와 추상화에 대해 자세히 알아보려면 [상태 전이와 저장소](/learn/state-transitions-and-storage/)를 참조하세요.

## 단일 값 저장

FRAME 저장소 모듈은 런타임에서 단일 값을 저장하기 위한 `StorageValue` 트레이트를 제공합니다.

이 워크샵에서는 `CollectiblesCount`에 대해 `StorageValue`를 사용하여 팔레트 내의 총 수집품 수를 추적합니다. `StorageValue`는 64비트 부호 없는 정수(u64) 값을 추적하며, 새로운 수집품을 생성할 때마다 증가하며 최대 18_446_744_073_709_551_615개의 고유한 수집품을 생성할 수 있습니다.

```rust
#[pallet::storage]
pub(super) type CollectiblesCount<T: Config> = StorageValue<_, u64, ValueQuery>;
```

이 선언에서 `ValueQuery`는 저장소에 값이 없을 때 쿼리가 반환해야 하는 값을 지정합니다.
쿼리가 반환해야 하는 값에는 세 가지 가능한 설정이 있습니다: `OptionQuery`, `ResultQuery`, 또는 `ValueQuery`.
여기서는 `ValueQuery`를 사용하여 저장소에 값이 없을 때(예: 네트워크를 처음 시작할 때) 쿼리가 `None` 값인 `OptionQuery` 대신 0인 `ValueQuery` 값을 반환하도록 합니다.

## 수집품과 속성을 매핑

FRAME 저장소 모듈은 런타임에서 단일 키 맵을 저장하기 위한 `StorageMap` 트레이트를 제공합니다.
`CollectiblesMap`이라는 StorageMap은 각 수집품을 해당 수집품의 고유한 정보에 매핑합니다.
`CollectiblesMap` 맵의 키는 수집품의 `unique_id`입니다.

```rust
/// Collectible 구조체를 unique_id에 매핑합니다.
#[pallet::storage]
pub(super) type CollectibleMap<T: Config> = StorageMap<_, Twox64Concat, [u8; 16], Collectible<T>>;
```

이 선언에서 `Twox64Concat`은 이 저장소 값을 생성하기 위해 사용할 해시 알고리즘을 지정합니다.
해시 알고리즘을 지정할 수 있도록 함으로써 저장소 맵은 저장할 정보의 보안 수준을 제어할 수 있습니다.
예를 들어, 수집품에 대한 정보를 저장할 때는 보다 성능이 우수하지만 보안 수준이 낮은 해시 알고리즘을 선택하고, 보다 민감한 정보를 저장할 때는 보다 보안 수준이 높지만 성능이 낮은 해시 알고리즘을 선택할 수 있습니다.
Substrate가 지원하는 해시 알고리즘과 제공하는 보안 수준에 대한 정보는 [해시 알고리즘](/build/runtime-storage/#hashing-algorithms)을 참조하세요.

## 소유자와 수집품을 매핑

`OwnerOfCollectibles`라는 StorageMap은 사용자 계정을 해당 사용자가 소유한 수집품에 매핑합니다.

이 저장소 맵의 키는 사용자 계정인 `T::AccountID`입니다.
이 저장소 맵의 값은 각 사용자 계정이 소유한 각 수집품의 `unique_id`를 가진 `BoundedVec` 데이터 유형입니다.
이 맵을 사용하면 `unique_id`가 `collectiblesMap` 맵의 키로 사용되므로 각 개별 수집품에 대한 정보를 쉽게 조회할 수 있습니다.
BoundedVec를 사용함으로써 각 저장소 항목이 최대 길이를 가지도록 할 수 있으며, 이는 런타임 내에서 제한을 관리하는 데 중요합니다.

```rust
/// 각 계정이 소유한 수집품을 추적합니다.
#[pallet::storage]
pub(super) type OwnerOfCollectibles<T: Config> = StorageMap<
	_,
	Twox64Concat,
	T::AccountId,
	BoundedVec<[u8; 16], T::MaximumOwned>,
	ValueQuery,
>;
```

코드가 컴파일되려면 `Collectible` 데이터 구조체에 다음 파생 매크로를 주석으로 추가해야 합니다:

```rust
#[scale_info(skip_type_params(T))]
```

다음 명령을 실행하여 프로그램이 컴파일되는지 확인하세요:

```bash
cargo build --package collectibles
```

현재는 컴파일러 경고를 무시해도 됩니다.