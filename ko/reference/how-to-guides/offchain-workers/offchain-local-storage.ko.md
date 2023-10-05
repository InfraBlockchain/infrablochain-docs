---
title: 오프체인 로컬 스토리지
description:
keywords:
  - 오프체인 워커
  - ocw
  - 로컬
  - 스토리지
---

이 가이드는 오프체인 워커를 사용하여 검색된 데이터를 로컬 스토리지에 저장하여 나중에 액세스하는 방법을 가르칠 것입니다.

지난 섹션에서 오프체인 워커(OCW)는 블록체인 상태를 직접 수정할 수 없으므로 계산된 결과를 체인에 다시 저장하기 위해 트랜잭션을 제출해야 한다고 언급했습니다.
그러나 데이터가 체인에 저장하기에 적합하지 않지만 나중에 액세스할 수 있도록 어딘가에 저장해야 하는 경우도 있습니다.
이에는 일시적인 데이터나 계산이 완료된 후에 폐기할 수 있는 중간 계산이 포함됩니다.

이 가이드에서는 오프체인 워커에게 데이터를 전체 블록체인 네트워크 사이에서 전달하지 않고 로컬 스토리지에 데이터를 작성하도록 지시합니다.
여러 OCW에서 일관되게 액세스할 수 있는 값을 가지기 위해 **스토리지 락(Storage Lock)** 개념이 소개됩니다.
OCW는 각 블록 생성의 끝에서 비동기적으로 실행되며 실행 시간에 제한이 없으므로 언제든지 여러 OCW 인스턴스가 초기화될 수 있습니다.

로컬 스토리지 API는 체인 상의 상응하는 API와 유사하게 `get`, `set`, `mutate`를 사용하여 액세스합니다.
`mutate`는 [**비교 및 교체 패턴**](https://en.wikipedia.org/wiki/Compare-and-swap)을 사용하여 메모리 위치의 내용을 주어진 값과 비교하고, 그 값이 동일한 경우에만 해당 메모리 위치의 내용을 새로운 주어진 값으로 수정합니다.
이는 단일 원자적 작업으로 수행됩니다.
원자성은 새로운 값이 최신 정보를 기반으로 계산되었음을 보장합니다. 값이 그 동안 다른 스레드에 의해 업데이트되었다면 쓰기 작업은 실패합니다.

로컬 스토리지에 저장된 값은 네트워크 간의 합의 메커니즘을 거치지 않았으므로 노드 운영자의 조작에 노출될 수 있다는 점에 유의하십시오.

이 가이드에서는 먼저 스토리지에서 캐시로 작동하는지 확인하기 위해 스토리지를 확인합니다.
캐시된 값이 발견되면 오프체인 워커가 반환되고, 그렇지 않으면 락을 획득하고 고성능 계산을 수행한 후 스토리지에 저장합니다.

## 절차

1. 팔렛의 `offchain_worker` 함수 훅에서 스토리지 참조를 정의합니다.

   ```rust
   fn offchain_worker(block_number: T::BlockNumber) {
     // 로컬 스토리지 값에 대한 참조를 생성합니다.
     // 로컬 스토리지는 모든 오프체인 워커에 공통이므로,
     // 팔렛 이름으로 항목을 먼저 추가하는 것이 좋습니다.
     let storage = StorageValueRef::persistent(b"pallet::my-storage");
   }
   ```

   위의 코드에서는 [`StorageValueRef::persistent()`](https://paritytech.github.io/substrate/master/sp_runtime/offchain/storage/struct.StorageValueRef.html#method.persistent)를 사용하여 영구 로컬 스토리지를 정의하고 `pallet::my-storage` 키로 식별합니다.
   키는 `str` 타입이 아닌 바이트 배열 타입으로 지정됩니다.
   이 로컬 스토리지는 오프체인 워커의 실행 간에 지속되고 공유됩니다.

1. 스토리지에 캐시된 값이 있는지 확인합니다.

   ```rust
   fn offchain_worker(block_number: T::BlockNumber) {
     // ...
     let storage = StorageValueRef::persistent(b"pallet::my-storage");

     if let Ok(Some(res)) = storage.get::<u64>() {
       log::info!("cached result: {:?}", res);
       return Ok(());
     }
   }
   ```

   [`get<T: Decode>()`](https://paritytech.github.io/substrate/master/sp_runtime/offchain/storage/struct.StorageValueRef.html#method.get) 함수를 사용하여 결과를 가져옵니다. 이 함수는 `Result<Option<T>, StorageRetrievalError>` 타입을 반환합니다.
   유효한 값이 있는 경우에만 관심이 있습니다. 그렇다면 `Ok(())`를 반환합니다.
   반환된 값의 타입을 정의해야 합니다.

   `set()`을 사용하여 스토리지에 쓰고, `mutate<T, E, F>()`를 사용하여 스토리지를 원자적으로 읽고 변경합니다.

1. 유효한 값(`None`)이 없거나 `StorageRetrievalError`가 있는 경우, 필요한 결과를 계산하고 스토리지 락을 획득합니다.

   다음과 같이 스토리지 락을 먼저 정의합니다.

   ```rust
   const LOCK_BLOCK_EXPIRATION: u32 = 3; // 블록 번호로 지정
   const LOCK_TIMEOUT_EXPIRATION: u64 = 10000; // 밀리초로 지정

   fn offchain_worker(block_number: T::BlockNumber) {
     // ...
     let storage = StorageValueRef::persistent(b"pallet::my-storage");

     if let Ok(Some(res)) = storage.get::<u64>() {
       log::info!("cached result: {:?}", res);
       return Ok(());
     }

     // 매우 고성능 계산이 여기에 있습니다.
     let val: u64 = 100 + 100;

     // 스토리지 락을 정의합니다.
     let mut lock = StorageLock::<BlockAndTime<Self>>::with_block_and_time_deadline(
       b"pallet::storage-lock",
       LOCK_BLOCK_EXPIRATION,
       Duration::from_millis(LOCK_TIMEOUT_EXPIRATION)
     );
   }
   ```

   위의 코드 스니펫에서는 [블록 번호와 시간 만료가 지정된](https://paritytech.github.io/substrate/master/sp_runtime/offchain/storage_lock/struct.StorageLock.html#method.with_block_and_time_deadline) 스토리지 락이 정의됩니다.
   이 함수는 락 식별자, 블록 번호 만료 및 시간 만료를 입력으로 받습니다.
   위의 락은 지정된 블록 번호나 시간 경과 중 하나가 지나면 만료됩니다.
   [블록 번호만](https://paritytech.github.io/substrate/master/sp_runtime/offchain/storage_lock/struct.StorageLock.html#method.with_block_deadline) 또는 [시간 경과](https://paritytech.github.io/substrate/master/sp_runtime/offchain/storage_lock/struct.StorageLock.html#method.with_deadline)로 만료 기간을 지정할 수도 있습니다.

1. [`.try_lock()`](https://paritytech.github.io/substrate/master/sp_runtime/offchain/storage_lock/struct.StorageLock.html#method.try_lock)을 사용하여 스토리지 락을 획득합니다.

   ```rust
   fn offchain_worker(block_number: T::BlockNumber) {
     // ...

     let mut lock = /* .... */;

     if let Ok(_guard) = lock.try_lock() {
       storage.set(&val);
     }
   }
   ```

   `Result<StorageLockGuard<'a, '_, L>, <L as Lockable>::Deadline>`를 반환합니다.
   여기서의 메커니즘은 스토리지에 쓰기 전에 먼저 락 가드를 소유해야 한다는 것입니다. 락 가드는 한 번에 하나의 프로세스만 소유할 수 있습니다.
   그런 다음 `set()`을 사용하여 값을 스토리지에 씁니다.
   `set()`에 전달되는 값의 데이터 타입은 위의 `get<T>()` 호출에서 지정한 타입과 동일해야 합니다.

1. 마지막으로, 오프체인 워커 함수에서 반환합니다.

   전체 코드는 다음과 유사합니다.

   ```rust
   const LOCK_BLOCK_EXPIRATION: u32 = 3; // 블록 번호로 지정
   const LOCK_TIMEOUT_EXPIRATION: u64 = 10000; // 밀리초로 지정

   fn offchain_worker(block_number: T::BlockNumber) {
     let storage = StorageValueRef::persistent(b"pallet::my-storage");

     if let Ok(Some(res)) = storage.get::<u64>() {
       log::info!("cached result: {:?}", res);
       return Ok(());
     }

     // 매우 고성능 계산이 여기에 있습니다.
     let val: u64 = 100 + 100;

     // 스토리지 락을 정의합니다.
     let mut lock = StorageLock::<BlockAndTime<Self>>::with_block_and_time_deadline(
       b"pallet::storage-lock",
       LOCK_BLOCK_EXPIRATION,
       Duration::from_millis(LOCK_TIMEOUT_EXPIRATION)
     );

     if let Ok(_guard) = lock.try_lock() {
       storage.set(&val);
     }
     Ok(())
   }
   ```

## 예제

- [**Substrate의 오프체인 워커 예제 팔렛**](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/offchain-worker/src/lib.rs#L372-L441)
- [**OCW 팔렛** 데모](https://github.com/jimmychu0807/substrate-offchain-worker-demo/blob/master/pallets/ocw/src/lib.rs#L299-L342)