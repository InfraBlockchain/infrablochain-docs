---
title: 기본 저장소 마이그레이션
description: 특정 팔레트의 저장소를 수정하고 새로운 저장소 레이아웃으로 마이그레이션하기 위한 준비 방법을 설명합니다.
keywords:
  - 저장소 마이그레이션
  - 런타임
  - 업그레이드
---

이 가이드는 FRAME [Nicks 팔레트](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/nicks)를 수정하여 특정 팔레트에 대한 저장소 마이그레이션을 수행하는 방법을 설명합니다.
이 튜토리얼에서는 옵션 필드를 포함하는 마지막 이름을 포함하는 저장소 맵을 수정하고, 이후 런타임 업그레이드로 트리거할 수 있는 마이그레이션 함수를 작성합니다.
이러한 유형의 간단한 저장소 마이그레이션은 변경 사항이 특정 팔레트와 개별 저장소 항목에 제한되는 경우에 사용할 수 있습니다.
더 복잡한 데이터 마이그레이션에 대해서는 이 튜토리얼에서 설명하는 것보다 더 복잡한 마이그레이션 함수를 작성하고 마이그레이션을 테스트하기 위해 추가 도구를 사용해야 합니다.

## Nicks 팔레트를 로컬에 추가하기

[FRAME의 Nicks 팔레트](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/nicks)에서 변경 사항을 가지고 올 것입니다. [런타임에 팔레트 추가하기](/tutorials/build-application-logic/add-a-pallet/) 튜토리얼에서는 노드 템플릿에 Nicks 팔레트를 추가하는 방법을 보여줍니다.

이 가이드에서는 팔레트의 코드를 수정하기 때문에 팔레트의 코드를 가져와서 로컬에 추가할 것입니다. 로컬에 추가하는 방법에 대한 예제는 [여기](https://github.com/substrate-developer-hub/substrate-node-template/commit/022b6da0d1d55f54de3568e97aa5fe45a7975fa5)에서 확인할 수 있습니다.

테스트를 위해 노드를 시작하고 Nicks 팔레트 내에서 extrinsic `setName`을 사용하여 닉네임을 설정할 수 있습니다.

## 저장소 구조체 생성 및 저장소 항목 업데이트

기본적으로 Nicks 팔레트는 저장소 맵을 사용하여 닉네임을 저장하는 `BoundedVec`을 포함하는 조회 테이블을 제공합니다.
예를 들어, 기본 저장소 정의는 다음과 같습니다:

```rust
/// 이름을 위한 조회 테이블.
	#[pallet::storage]
	pub(super) type NameOf<T: Config> =
		StorageMap<_, Twox64Concat, T::AccountId, (BoundedVec<u8, T::MaxLength>, BalanceOf<T>)>;
```
저장소를 업데이트하여 옵션 필드를 포함하는 새로운 구조체 `Nickname`를 만들고 이전 및 새로운 저장소 항목인 이름과 성을 관리합니다:

```rust
    #[derive(Encode, Decode, Default, TypeInfo, MaxEncodedLen, PartialEqNoBound, RuntimeDebug)]
	#[scale_info(skip_type_params(T))]
	#[codec(mel_bound())]
	pub struct Nickname<T: Config> {
		pub first: BoundedVec<u8, T::MaxLength>,
		pub last: Option<BoundedVec<u8, T::MaxLength>>,
	}
```
이제 저장소에 저장된 데이터를 변경하기 위해 `NameOf` StorageMap을 `BoundedVec` 대신 Nickname 구조체를 저장하도록 업데이트합니다.

```rust
    #[pallet::storage]
	pub(super) type NameOf<T: Config> =
		StorageMap<_, Twox64Concat, T::AccountId, (Nickname<T>, BalanceOf<T>)>;
```

## 함수 업데이트

새로운 데이터 구조체를 추가하고 저장소에 이름과 선택적인 성을 모두 포함하도록 수정한 후에는 Nicks 팔레트 함수를 업데이트하여 새로운 `last: Option<BoundedVec<u8>>` 매개변수 선언을 포함해야 합니다.
대부분의 경우, 저장소 항목을 수정할 때 함수를 업데이트하려면 변경 사항을 처리하기 위해 일부 로직을 추가해야 합니다. 예를 들어, 매개변수 이름을 수정하거나 새 변수를 추가해야 할 수 있습니다.

이 경우에는 `set_name` 및 `force_name` 함수에서 대부분의 변경 사항이 필요합니다.
예를 들어, `set_name` 함수를 수정하여 `bounded_name`을 `bounded_first`로 변경하고 다음과 같은 코드로 `bounded_last` 선언을 추가할 수 있습니다:

```rust
//--snip
pub fn set_name(origin,
    first: Vec<u8>,
    last: Option<Vec<u8>>)  -> DispatchResult{
```

또한, 모든 저장소 쓰기를 `Nickname` 구조체로 업데이트합니다:
```rust
//--snip
pub fn set_name(origin,
    first: Vec<u8>,
    last: Option<Vec<u8>>)  -> DispatchResult{
    //--snip

    let bounded_first: BoundedVec<_, _> =
        first.try_into().map_err(|_| Error::<T>::TooLong)?;
    ensure!(bounded_first.len() >= T::MinLength::get() as usize, Error::<T>::TooShort);

    let mut bounded_last: BoundedVec<_, _> = Default::default();
    if let Some(last) = last {
        bounded_last= last.try_into().map_err(|_| Error::<T>::TooLong)?;
        ensure!(bounded_last.len() >= T::MinLength::get() as usize, Error::<T>::TooShort);
    }
    let bounded_last: Option<BoundedVec<u8, T::MaxLength>> = Some(bounded_last);

    //--snip
    <NameOf<T>>::insert(&sender, (Nickname{first: bounded_first, last: bounded_last}, deposit));
    }
```
완전한 extrinsic 업데이트 예제는 [여기](https://github.com/substrate-developer-hub/substrate-node-template/commit/a9ee9b2b9096c2b85ecb4448366df2b8502e7aa7)에서 확인할 수 있습니다.

## 저장소 버전 추가

`pallet::pallet` 매크로는 `traits::GetStorageVersion`을 구현하지만 현재 저장소 버전은 매크로에 전달되어야 합니다. 이를 위해 `pallet::storage_version` 매크로를 사용할 수 있습니다.
```rust
    /// 현재 저장소 버전, 새 버전으로 2를 설정합니다.
	const STORAGE_VERSION: StorageVersion = StorageVersion::new(2);


    #[pallet::pallet]
	#[pallet::storage_version(STORAGE_VERSION)]
	pub struct Pallet<T>(_);
```

## 마이그레이션 모듈 선언

마이그레이션 모듈은 두 부분으로 구성되어야 합니다:

- 마이그레이션할 이전 저장소를 나타내는 모듈
- 마이그레이션 함수로 가중치를 반환하는 함수

src/pallets/nicks/migration.rs에 새 파일을 생성합니다.

이 모듈의 구조는 다음과 같습니다:

```rust
pub mod migration {
  use super::*;

  pub mod v1 {...} // V1 저장소 형식만 포함

  pub fn migrate_to_v2<T: Config>() -> Weight {...} // 저장소를 V2 형식으로 변환하는 체크 및 변환 로직
}
```

## `migrate_to_v2` 작성

이 함수가 수행해야 할 작업에 대한 개요입니다:

- 마이그레이션이 필요한지 확인하기 위해 저장소 버전을 확인합니다(좋은 습관).
- 저장소 값을 새로운 저장소 형식으로 변환합니다.
- 저장소 버전을 업데이트합니다.
- 마이그레이션에 소비된 가중치를 반환합니다.

### 저장소 버전 확인

마이그레이션 로직을 `migrate_to_v2` 주위에 구성합니다. 저장소 마이그레이션이 필요하지 않은 경우 0을 반환합니다:

```rust
    let onchain_version =  Pallet::<T>::on_chain_storage_version();
    if onchain_version < 2 {

    }
    else {
        // 여기서는 아무 작업도 수행하지 않습니다.
		Weight::zero()
    }
```

### 저장소 값을 변환

[`translate storage 메서드`][translate-storage-rustdocs]를 사용하여 저장소 값을 새 형식으로 변환합니다. 기존 저장소의 `nick` 값은 공백으로 구분된 문자열일 수 있으므로, `' '`에서 분할하여 새로운 `last` 저장소 항목에 그 이후의 모든 것을 배치합니다. 그렇지 않은 경우 `last`는 `None` 값을 가집니다:

```rust
        // 이전 형식에서 새 형식으로 저장소 값을 변환합니다.
        NameOf::<T>::translate::<(Vec<u8>, BalanceOf<T>), _>(
            |k: T::AccountId, (nick, deposit): (Vec<u8>, BalanceOf<T>)| {
                info!(target: LOG_TARGET, "     Migrated nickname for {:?}...", k);

                // nick을 ' ' (<공백>)로 분할합니다.
                match nick.iter().rposition(|&x| x == b" "[0]) {
                    Some(ndx) => {
                        let bounded_first: BoundedVec<_, _> = nick[0..ndx].to_vec().try_into().unwrap();
                        let bounded_last: BoundedVec<_, _> = nick[ndx + 1..].to_vec().try_into().unwrap();
                        Some((Nickname {
                            first: bounded_first,
                            last: Some(bounded_last)
                        }, deposit))
                },
                    None => {
                        let bounded_name: BoundedVec<_, _> = nick.to_vec().try_into().unwrap();
                        Some((Nickname { first: bounded_name, last: None }, deposit))
                    }
                }
            }
        );
```

### 저장소 버전 업데이트

```rust
    // 저장소 버전 업데이트.
	StorageVersion::new(2).put::<Pallet::<T>>();
```

### 소비된 가중치 반환

이를 위해 저장소 읽기 및 쓰기 수를 계산하고 해당하는 가중치를 반환합니다:

```rust
let count = NameOf::<T>::iter().count();
T::DbWeight::get().reads_writes(count as Weight + 1, count as Weight + 1)
```

### 7. `on_runtime_upgrade`에서 `migrate_to_v2` 사용
pallet lib.rs에서 mod migration을 선언합니다.
```rust
mod migration;
```

그리고 팔레트의 함수로 돌아가서 `on_runtime_upgrade`에서 `migrate_to_v2` 함수를 지정합니다. 이렇게 하면 런타임 업그레이드 시에 어떤 작업이 수행되어야 하는지 표현할 수 있습니다:

```rust
    #[pallet::hooks]
	impl<T: Config> Hooks<BlockNumberFor<T>> for Pallet<T> {
		fn on_runtime_upgrade() -> frame_support::weights::Weight {
			migration::migrate_to_v2::<T>()
		}
	}
```

전체 마이그레이션 코드 예제는 [여기](https://github.com/substrate-developer-hub/substrate-node-template/commit/cfbe01dd4be358d0df45a81b87b6ba7393e20368)에서 확인할 수 있습니다.

## 단위 테스트 업데이트

런타임 마이그레이션 모듈을 작성할 때는 저장소 항목을 망가뜨리는 중대한 문제를 방지하기 위해 테스트하는 것이 중요합니다.

Nicks 팔레트의 테스트는 다음과 같습니다:

- `fn kill_name_should_work()`
- `fn force_name_should_work()`
- `fn normal_operation_should_work()`
- `fn error_catching_should_work()`

이러한 테스트를 새로운 코드와 함께 작동하도록 업데이트해야 합니다. 예를 들어:
```rust
    #[test]
	fn normal_operation_should_work() {
		new_test_ext().execute_with(|| {
			assert_ok!(Nicks::set_name(RuntimeOrigin::signed(1), b"Gav".to_vec(), None));
			assert_eq!(Balances::reserved_balance(1), 2);
			assert_eq!(Balances::free_balance(1), 8);
			assert_eq!(<NameOf<Test>>::get(1).unwrap().0.first, b"Gav".to_vec());

			assert_ok!(Nicks::set_name(RuntimeOrigin::signed(1), b"Gavin".to_vec(), None));
			assert_eq!(Balances::reserved_balance(1), 2);
			assert_eq!(Balances::free_balance(1), 8);
			assert_eq!(<NameOf<Test>>::get(1).unwrap().0.first, b"Gavin".to_vec());

			assert_ok!(Nicks::clear_name(RuntimeOrigin::signed(1)));
			assert_eq!(Balances::reserved_balance(1), 0);
			assert_eq!(Balances::free_balance(1), 10);
		});
	}
```
전체 테스트 수정 예제는 [여기](https://github.com/substrate-developer-hub/substrate-node-template/commit/fa021b11878e8621bb455d4638e1821b681c085e)에서 확인할 수 있습니다.

## 예제

- [Nicks 팔레트 마이그레이션](https://github.com/substrate-developer-hub/substrate-node-template/tree/alexd10s/how-to-storage-migration-example)

## 자료

#### Rust 문서

- [Rust의 Option](https://doc.rust-lang.org/std/option/)
- [`frame_support::storage::migration`](https://paritytech.github.io/substrate/master/frame_support/storage/migration/index.html) 유틸리티 문서

[translate-storage-rustdocs]: https://paritytech.github.io/substrate/master/frame_support/storage/types/struct.StorageMap.html#method.translate
[nicks-migration-htg-diff]: https://github.com/substrate-developer-hub/migration-example/pull/2/files