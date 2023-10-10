---
title: 오프체인 인덱싱
description:
keywords:
  - 오프체인 워커
  - OCW
  - 인덱싱
---

이 가이드에서는 온체인 외부 호출에서 스토리지에 기록하지 않고 오프체인 워커로 데이터를 전달하는 방법을 안내합니다.

가끔씩 온체인 외부 호출에서 예측 가능한 쓰기 동작을 가진 데이터를 오프체인 워커에 전달해야 할 때가 있습니다.
이 데이터는 오프체인 워커가 읽을 수 있도록 온체인 스토리지에 기록될 수 있지만, 이는 블록체인에 막대한 비용을 발생시킬 수 있습니다.
온체인에서 오프체인으로 데이터를 전달하는 또 다른 방법은 **오프체인 인덱싱**을 통해 노드의 로컬 스토리지에 저장하는 것입니다.

오프체인 인덱싱은 온체인 외부 호출에서 호출되며, 이는 로컬로 기록된 데이터가 네트워크의 모든 노드에서 일관성을 유지할 것으로 예상됨을 의미합니다.

다른 사용 사례는 오프체인 워커가 처리해야 할 대량의 데이터를 온체인에 저장해야 할 때입니다.
이는 너무 비용이 많이 들 수 있습니다.
해결책은 오프체인 인덱싱을 사용하여 해당 데이터의 해시를 온체인에 저장하고, 해당 원시(raw) 데이터를 로컬로 저장하여 나중에 오프체인 워커가 읽을 수 있도록 하는 것입니다.

동일한 외부 호출이 포크된 블록이 있는 경우 여러 번 실행될 수 있다는 점에 유의하십시오.
이로 인해 고유하지 않은 키가 사용되는 경우 데이터가 다른 포크된 블록에 의해 덮어쓰일 수 있으며, 로컬 스토리지의 내용은 노드 간에 다를 수 있습니다.
따라서 개발자는 잠재적인 덮어쓰기를 방지하기 위해 올바른 인덱싱 키를 형성하는 데 주의해야 합니다.

참고: 오프체인 인덱싱 기능을 확인하려면 오프체인 인덱싱 플래그 _ON_ 으로 Substrate 노드를 실행하십시오.
예를 들어: `./target/release/substrate-node --enable-offchain-indexing true`

## 단계

1. 인덱싱에 사용할 고유한 키를 생성합니다.

   팔렛의 `src/lib.rs`에서:

   ```rust
   const ONCHAIN_TX_KEY: &[u8] = b"my_pallet::indexing1";

   #[pallet::call]
   impl<T: Config> Pallet<T> {
   	#[pallet::weight(100)]
   	pub fn extrinsic(origin: OriginFor<T>, number: u64) -> DispatchResult {
   		let who = ensure_signed(origin)?;

   		let key = Self::derived_key(frame_system::Module::<T>::block_number());
   		// ...

   		Ok(())
   	}
   }

   impl<T: Config> Pallet<T> {
   	fn derived_key(block_number: T::BlockNumber) -> Vec<u8> {
   		block_number.using_encoded(|encoded_bn| {
   			ONCHAIN_TX_KEY.clone().into_iter()
   				.chain(b"/".into_iter())
   				.chain(encoded_bn)
   				.copied()
   				.collect::<Vec<u8>>()
   		})
   	}
   }
   ```

   위 코드에서 일반 외부 호출 내에서는 `Self::derived_key()` 도우미 메서드를 호출하여 나중에 인덱싱에 사용할 키를 생성합니다.
   이는 미리 정의된 접두사와 현재 인코딩된 블록 번호를 연결하여 바이트 벡터로 반환합니다.

1. 인덱싱 데이터를 정의하고 오프체인 인덱싱을 사용하여 저장합니다:

   ```rust
   use sp_io::offchain_index;
   const ONCHAIN_TX_KEY: &[u8] = b"my_pallet::indexing1";

   #[derive(Debug, Deserialize, Encode, Decode, Default)]
   struct IndexingData(Vec<u8>, u64);

   #[pallet::call]
   impl<T: Config> Pallet<T> {
   	#[pallet::weight(100)]
   	pub fn extrinsic(origin: OriginFor<T>, number: u64) -> DispatchResult {
   		let who = ensure_signed(origin)?;

   		let key = Self::derived_key(frame_system::Module::<T>::block_number());
   		let data = IndexingData(b"submit_number_unsigned".to_vec(), number);
   		offchain_index::set(&key, &data.encode());
   		Ok(())
   	}
   }

   impl<T: Config> Pallet<T> {
   	// -- 생략 --
   }
   ```

   인덱싱 데이터는 `Encode`, `Decode`, `Deserialize` 트레잇으로 바운드될 수 있는 모든 데이터 유형일 수 있습니다.
   위 코드에서는 데이터를 [`offchain_index::set()`](https://paritytech.github.io/substrate/master/sp_io/offchain_index/fn.set.html) 메서드를 사용하여 오프체인 인덱싱을 통해 저장합니다.

1. `offchain_worker` 훅 메서드를 사용하여 오프체인 워커의 데이터베이스에서 데이터를 읽습니다:

   ```rust
   use sp_runtime::offchain::StorageValueRef;

   #[derive(Debug, Deserialize, Encode, Decode, Default)]
   struct IndexingData(Vec<u8>, u64);

   fn offchain_worker(block_number: T::BlockNumber) {
   	// 오프체인 인덱싱 값을 다시 읽어옵니다. 이는 ocw 로컬 스토리지에서 읽는 것과 정확히 동일합니다.
   	let key = Self::derived_key(block_number);
   	let storage_ref = StorageValueRef::persistent(&key);

   	if let Ok(Some(data)) = storage_ref.get::<IndexingData>() {
   		debug::info!("로컬 스토리지 데이터: {:?}, {:?}",
   			str::from_utf8(&data.0).unwrap_or("에러"), data.1);
   	} else {
   		debug::info!("로컬 스토리지에서 읽는 중 오류가 발생했습니다.");
   	}

   	// -- 생략 --
   }
   ```

   이를 통해 오프체인 워커는 노드의 로컬 스토리지에서 해당 데이터를 읽을 수 있습니다.
   [오프체인 로컬 스토리지](/reference/how-to-guides/offchain-workers/offchain-local-storage/) 가이드에서 이를 수행하는 방법에 대해 설명합니다.

## 관련 자료

- [오프체인 작업](/learn/offchain-operations/)
- [오프체인 로컬 스토리지](/reference/how-to-guides/offchain-workers/offchain-local-storage/)