---
title: 기본 함수 생성하기
description:
tutorial:
---

이제 몇 가지 사용자 정의 타입과 저장소 항목을 포함한 팔레트의 기본 구조가 준비되었습니다.
이제 사용자 정의 타입을 사용하고 생성한 저장소 항목에 변경 사항을 기록하는 함수를 작성해야 합니다.
코드를 모듈화하고 유연하게 유지하기 위해 대부분의 로직은 내부 도우미 함수에 정의되어 있으며, 블록체인과 상호작용할 수 있는 호출 가능한 함수는 하나뿐입니다.
내부 함수를 사용하는 장점 중 하나는 내부 함수는 런타임 개발자로서만 접근할 수 있으므로 수행되는 작업에 대해 인증 또는 권한 부여를 할 필요가 없다는 것입니다.

이 워크샵에서 사용자에게 노출해야 하는 유일한 함수는 `create_collectible` 함수입니다.
이 함수를 사용하면 사용자가 `CollectibleMap`에 저장되고 `OwnerOfCollectibles` 맵에 추가되는 새로운 고유한 수집품을 생성할 수 있습니다.
높은 수준에서 다음 작업을 수행하는 함수를 작성해야 합니다.

- 각 수집품에 대해 고유한 식별자를 생성하고 중복을 허용하지 않습니다.
- 각 계정이 소유할 수 있는 수집품의 수를 제한하여 각 사용자가 사용할 수 있는 저장소를 관리합니다.
- 수집품의 총 수가 `u64` 데이터 타입이 허용하는 최대값을 초과하지 않도록 합니다.
- 사용자가 새로운 수집품을 생성할 수 있도록 합니다.

새로운 수집품을 생성하는 데 필요한 기본 기능 외에도, 팔레트는 기본 이벤트 및 오류 처리 기능을 제공해야 합니다.
이를 위해 다음과 같은 작업을 수행해야 합니다.

- 문제가 발생한 경우 무엇이 잘못되었는지를 보고하기 위해 사용자 정의 오류 메시지를 생성합니다.
- 새로운 수집품이 성공적으로 생성되었음을 알리기 위해 사용자 정의 이벤트를 생성합니다.

오류와 이벤트는 상당히 직관적이므로, 먼저 이러한 선언부터 시작합시다.

## 사용자 정의 오류 추가하기

`create_collectible` 함수에서 처리해야 할 몇 가지 잠재적인 오류는 다음과 같습니다.

- `DuplicateCollectible` - 생성하려는 수집품이 이미 존재하는 경우 발생합니다.
- `MaximumCollectiblesOwned` - 계정이 단일 계정이 보유할 수 있는 최대 수집품 수를 초과한 경우 발생합니다.
- `BoundsOverflow` - 수집품의 공급이 `u64` 제한을 초과하는 경우 발생합니다.

런타임에 오류 처리를 추가하려면 다음 단계를 수행하세요.

1. 코드 편집기에서 `collectibles` 팔레트의 `src/lib.rs` 파일을 엽니다.

2. 이전에 정의한 저장소 매크로 다음에 `#[pallet::error]` 매크로를 추가합니다.

	 ```rust
	 #[pallet::error]
	 ```

1. `create_collectibles` 함수가 반환할 수 있는 잠재적인 오류에 대한 열거형 데이터 타입을 정의합니다.

	 ```rust
	 pub enum Error<T> {
		/// 각 수집품은 고유한 식별자를 가져야 합니다.
		DuplicateCollectible,
		/// 계정은 `MaximumOwned` 상수를 초과할 수 없습니다.
		MaximumCollectiblesOwned,
		/// 수집품의 총 공급은 `u64` 제한을 초과할 수 없습니다.
		BoundsOverflow,
	}
	```

1. 변경 사항을 저장합니다.

1. 다음 명령을 실행하여 프로그램이 컴파일되는지 확인합니다.

   ```bash
   cargo build --package collectibles
   ```

   현재는 컴파일러 경고를 무시해도 됩니다.

## 이벤트 추가하기

런타임은 트랜잭션이 성공적으로 실행된 결과를 프론트엔드 애플리케이션에 알리기 위해 이벤트를 발생시킬 수 있습니다.
[Subscan](https://www.subscan.io/)과 [Polkadot/Substrate Portal Explorer](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frpc.polkadot.io#/explorer)와 같은 블록 익스플로러는 완료된 트랜잭션에 대한 이벤트도 표시합니다.

런타임에 `CollectibleCreated` 이벤트를 추가하려면 다음 단계를 수행하세요.

1. 코드 편집기에서 `collectibles` 팔레트의 `src/lib.rs` 파일을 엽니다.

1. `frame_system` 구성에서 `RuntimeEvent`를 팔레트 구성에 추가합니다.

	 ```rust
	 #[pallet::config]
     pub trait Config: frame_system::Config {
		type RuntimeEvent: From<Event<Self>> + IsType<<Self as frame_system::Config>::RuntimeEvent>;
	}
	 ```

2. 이전에 정의한 오류 매크로 다음에 `#[pallet::event]` 매크로를 추가합니다.

	 ```rust
	 #[pallet::event]
	 ```

1. 팔레트에 대한 이벤트를 코드에 추가합니다.

	 ```rust
	 #[pallet::generate_deposit(pub(super) fn deposit_event)]
	 pub enum Event<T: Config> {
		 /// 새로운 수집품이 성공적으로 생성되었습니다.
		 CollectibleCreated { collectible: [u8; 16], owner: T::AccountId },
	 }
	 ```

1. 변경 사항을 저장합니다.

1. 다음 명령을 실행하여 프로그램이 컴파일되는지 확인합니다.

   ```bash
   cargo build --package collectibles
   ```

   현재는 컴파일러 경고를 무시해도 됩니다.

## 내부 및 호출 가능한 함수 추가하기

오류와 이벤트가 해결되었으므로, 이제 수집품을 생성하는 핵심 로직을 작성할 차례입니다.

1. 새로운 수집품에 대한 `unique_id`를 생성하는 내부 함수를 작성합니다.

	 ```rust
	 // 팔레트 내부 함수
	impl<T: Config> Pallet<T> {
		// 고유한 unique_id와 color를 생성하고 반환합니다.
		fn gen_unique_id() -> ([u8; 16], Color) {
			// 난수 생성
			let random = T::CollectionRandomness::random(&b"unique_id"[..]).0;
			
			// 난수 페이로드 생성. 동일한 블록에서 여러 개의 수집품을 생성할 수 있도록 고유성을 유지합니다.
			let unique_payload = (
				random,
				frame_system::Pallet::<T>::extrinsic_index().unwrap_or_default(),frame_system::Pallet::<T>::block_number(),
		);
		
		// 바이트 배열로 변환합니다.
		let encoded_payload = unique_payload.encode();
		let hash = frame_support::Hashable::blake2_128(&encoded_payload);
		
		// Color 생성
		if hash[0] % 2 == 0 {
				(hash, Color::Red)
		} else {
				(hash, Color::Yellow)
			} 
		}
	}
   ```

1. 새로운 수집품을 생성하는 내부 함수를 작성합니다.

	 ```rust
	     // 수집품 생성 함수
		pub fn mint(
			owner: &T::AccountId,
			unique_id: [u8; 16],
			color: Color,
		) -> Result<[u8; 16], DispatchError> {
			// 새로운 객체 생성
			let collectible = Collectible::<T> { unique_id, price: None, color, owner: owner.clone() };
			
			// 저장소 맵에 수집품이 이미 존재하는지 확인합니다.
			ensure!(!CollectibleMap::<T>::contains_key(&collectible.unique_id), Error::<T>::DuplicateCollectible);
			
			// 새로운 수집품을 생성할 수 있는지 확인합니다.
			let count = CollectiblesCount::<T>::get();
			let new_count = count.checked_add(1).ok_or(Error::<T>::BoundsOverflow)?;
			
			// OwnerOfCollectibles 맵에 수집품을 추가합니다.
			OwnerOfCollectibles::<T>::try_append(&owner, collectible.unique_id)
				.map_err(|_| Error::<T>::MaximumCollectiblesOwned)?;
			
			// 새로운 수집품을 저장소에 기록하고 수를 업데이트합니다.
			CollectibleMap::<T>::insert(collectible.unique_id, collectible);
			CollectiblesCount::<T>::put(new_count);
			
			// "CollectibleCreated" 이벤트를 발생시킵니다.
			Self::deposit_event(Event::CollectibleCreated { collectible: unique_id, owner: owner.clone() });
			
			// 성공한 경우 새로운 수집품의 unique_id를 반환합니다.
			Ok(unique_id)
		}
   ```

2. 내부 함수를 사용하는 호출 가능한 함수를 작성합니다.

	 ```rust
	 // 팔레트 호출 가능한 함수
	 #[pallet::call]
	 impl<T: Config> Pallet<T> {
		 
		 /// 새로운 고유한 수집품을 생성합니다.
		 ///
		 /// 실제 수집품 생성은 `mint()` 함수에서 수행됩니다.
		 #[pallet::weight(0)]
		 pub fn create_collectible(origin: OriginFor<T>) -> DispatchResult {
			  // 호출자가 서명된 origin에서 왔는지 확인합니다.
				let sender = ensure_signed(origin)?;
				
				// 도우미 함수를 사용하여 unique_id와 color를 생성합니다.
				let (collectible_gen_unique_id, color) = Self::gen_unique_id();
				
				// 도우미 함수를 호출하여 새로운 수집품을 저장소에 기록합니다.
				Self::mint(&sender, collectible_gen_unique_id, color)?;
				
				Ok(())
			}
	 }
   ```

1. 변경 사항을 저장하고 다음 명령을 실행하여 프로그램이 컴파일되는지 확인합니다.

   ```bash
   cargo build --package collectibles
   ```

  이제 코드가 경고 없이 컴파일되어야 합니다.