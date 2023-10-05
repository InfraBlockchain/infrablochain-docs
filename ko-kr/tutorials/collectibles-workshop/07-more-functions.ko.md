---
title: 더 많은 기능
description:
tutorial:
---

이 팔레트를 더 유용하고 현실적인 간단한 수집품 거래 애플리케이션의 기반으로 만들기 위해 몇 가지 기본 기능을 추가하여 다음 작업을 수행할 수 있도록 합시다:

- 계정 간 수집품 이전하기.
- 수집품의 가격 설정하기.
- 수집품 구매하기.

## 이전하기

사용자가 수집품을 한 계정에서 다른 계정으로 이전할 수 있도록 하려면 공개적으로 호출 가능한 함수를 생성해야 합니다.
이 워크샵에서는 공개 함수인 `transfer`가 내부 함수인 `do_transfer`를 호출하여 체크를 수행하고 변경 사항을 스토리지에 기록하도록 할 것입니다.
내부 함수에는 다음과 같은 체크가 포함되어 수집품을 이전할 수 있는지 여부를 결정합니다:

- 이전 가능한 수집품은 존재해야 합니다.
- 수집품은 현재 소유자에게 이전할 수 없습니다.
- 수집품은 이미 최대 허용 수집품 수를 가진 계정에게 이전할 수 없습니다.

수집품이 존재하고 현재 소유자에게 이전되지 않는 경우, `do_transfer` 내부 함수는 `OwnerOfCollectibles` 스토리지 맵을 업데이트하여 새로운 소유자를 반영하고 기본 데이터베이스에 변경 사항을 기록합니다.
오류를 처리하고 이벤트를 발생시키기 위해 다음과 같은 것들도 필요합니다:

- 체크가 실패할 경우 `NoCollectible`, `NotOwner`, `TransferToSelf` 오류.
- 이전이 성공적으로 이루어질 경우 `TransferSucceeded` 이벤트.

공개적으로 호출 가능한 함수는 단순히 호출자의 원본을 확인하고 `do_transfer` 내부 함수를 호출합니다.

1. 오류 열거형에 `NoCollectible`, `NotOwner`, `TransferToSelf` 변형을 추가하세요.
   
	 ```rust
	 // 팔레트 오류 메시지.
	 #[pallet::error]
	 pub enum Error<T> {
		 /// 각 수집품은 고유한 식별자를 가져야 합니다.
		 DuplicateCollectible,
		 /// 계정은 `MaximumOwned` 상수를 초과할 수 없습니다.
		 MaximumCollectiblesOwned,
		 /// 수집품의 총 공급은 u64 제한을 초과할 수 없습니다.
		 BoundsOverflow,
		 /// 수집품이 존재하지 않습니다.
		 NoCollectible,
		 /// 소유자가 아닙니다.
		 NotOwner,
		 /// 자신에게 수집품을 이전하려고 합니다.
		 TransferToSelf,
	 }
   ```

2. 팔레트에 `TransferSucceeded` 이벤트를 추가하세요.
   
	 ```rust
	 // 팔레트 이벤트.
	 #[pallet::event]
	 #[pallet::generate_deposit(pub(super) fn deposit_event)]
	 pub enum Event<T: Config> {
		  /// 새로운 수집품이 성공적으로 생성되었습니다.
			CollectibleCreated { collectible: [u8; 16], owner: T::AccountId },
			/// 수집품이 성공적으로 이전되었습니다.
			TransferSucceeded { from: T::AccountId, to: T::AccountId, collectible: [u8; 16] },
	 }
	 ```

3. 수집품 이전을 가능하게 하는 내부 함수를 생성하세요.
   
	 ```rust
	 // 스토리지 업데이트하여 수집품 이전하기
	 pub fn do_transfer(
		 collectible_id: [u8; 16],
		 to: T::AccountId,
	 ) -> DispatchResult {
		 // 수집품 가져오기
		 let mut collectible = CollectibleMap::<T>::get(&collectible_id).ok_or(Error::<T>::NoCollectible)?;
		 let from = collectible.owner;
		 
		 ensure!(from != to, Error::<T>::TransferToSelf);
		 let mut from_owned = OwnerOfCollectibles::<T>::get(&from);
		 
		 // 소유한 수집품 목록에서 수집품 제거하기.
		 if let Some(ind) = from_owned.iter().position(|&id| id == collectible_id) {
			 from_owned.swap_remove(ind);
			} else {
			  return Err(Error::<T>::NoCollectible.into())
			}
				// 소유한 수집품 목록에 수집품 추가하기.
				let mut to_owned = OwnerOfCollectibles::<T>::get(&to);
				to_owned.try_push(collectible_id).map_err(|_id| Error::<T>::MaximumCollectiblesOwned)?;
				
				// 이전이 성공적으로 이루어졌으므로 소유자를 업데이트하고 가격을 `None`으로 재설정합니다.
				collectible.owner = to.clone();
				collectible.price = None;

				// 업데이트를 스토리지에 기록합니다.
				CollectibleMap::<T>::insert(&collectible_id, collectible);
				OwnerOfCollectibles::<T>::insert(&to, to_owned);
				OwnerOfCollectibles::<T>::insert(&from, from_owned);
				
				Self::deposit_event(Event::TransferSucceeded { from, to, collectible: collectible_id });
			Ok(())
		}
	```

1. 호출 가능한 함수를 생성하세요.
   
	 ```rust
	 /// 수집품을 다른 계정으로 이전합니다.
	 /// 수집품을 보유한 어떤 계정이든 다른 계정으로 보낼 수 있습니다.
	 /// 이전은 수집품의 가격을 재설정하여 판매하지 않음으로 표시합니다.
	 #[pallet::weight(0)]
	 pub fn transfer(
		  origin: OriginFor<T>,
			to: T::AccountId,
			unique_id: [u8; 16],
	 ) -> DispatchResult {
		// 호출자가 서명된 원본인지 확인합니다.
		let from = ensure_signed(origin)?;
		let collectible = CollectibleMap::<T>::get(&unique_id).ok_or(Error::<T>::NoCollectible)?;
		ensure!(collectible.owner == from, Error::<T>::NotOwner);
		Self::do_transfer(unique_id, to)?;
		Ok(())
	 }
   ```

1. 변경 사항을 저장하세요.

2. 다음 명령을 실행하여 프로그램이 오류 없이 컴파일되는지 확인하세요:
   
   ```bash
   cargo build --package collectibles
   ```

## 가격 설정하기

수집품 소유자가 소유한 수집품의 가격을 설정할 수 있도록 하기 위해 팔레트는 Collectible 데이터 구조의 가격 필드를 업데이트하고 이벤트를 발생시키는 함수를 제공해야 합니다.
이 함수에서는 `CollectibleMap` 스토리지 항목의 `get` 및 `insert` 메서드를 사용하여 Collectible 객체를 수정하고 업데이트할 수 있습니다.

이전 함수와 마찬가지로, 호출자가 새로운 가격을 스토리지에 기록하기 전에 몇 가지 체크를 수행하도록 하려고 합니다:

- 호출자는 서명된 원본이어야 합니다.
- 수집품은 이미 존재해야 합니다.
- 호출자는 수집품의 소유자여야 합니다.

체크가 통과되면 함수는 새로운 가격을 스토리지에 기록하고 `PriceSet` 이벤트를 발생시킵니다.

1. `PriceSet` 이벤트를 팔레트에 추가하세요.
   
	 ```rust
	 // 팔레트 이벤트
	 #[pallet::event]
	 #[pallet::generate_deposit(pub(super) fn deposit_event)]
	 pub enum Event<T: Config> {
		/// 새로운 수집품이 성공적으로 생성되었습니다.
		CollectibleCreated { collectible: [u8; 16], owner: T::AccountId },
     	/// 수집품이 성공적으로 이전되었습니다.
     	TransferSucceeded { from: T::AccountId, to: T::AccountId, collectible: [u8; 16] },
		/// 수집품의 가격이 성공적으로 설정되었습니다.
		PriceSet { collectible: [u8; 16], price: Option<BalanceOf<T>> },
		}
	 ```

2. 가격을 설정하는 호출 가능한 함수를 추가하세요.
   
    ```rust
    /// 수집품 가격을 업데이트하고 스토리지에 기록합니다.
    #[pallet::weight(0)]
    pub fn set_price(
        origin: OriginFor<T>,
        unique_id: [u8; 16],
        new_price: Option<BalanceOf<T>>,
    ) -> DispatchResult {
        // 호출자가 서명된 원본인지 확인합니다.
        let sender = ensure_signed(origin)?;
        // 수집품이 존재하고 호출자가 소유자인지 확인합니다.
        let mut collectible = CollectibleMap::<T>::get(&unique_id).ok_or(Error::<T>::NoCollectible)?;
        ensure!(collectible.owner == sender, Error::<T>::NotOwner);
        // 스토리지에 가격을 설정합니다.
        collectible.price = new_price;
        CollectibleMap::<T>::insert(&unique_id, collectible);

        // "PriceSet" 이벤트를 발생시킵니다.
        Self::deposit_event(Event::PriceSet { collectible: unique_id, price: new_price });
        Ok(())
    }
    ```

2. 다음 명령을 실행하여 프로그램이 오류 없이 컴파일되는지 확인하세요:
   
   ```bash
   cargo build --package collectibles
   ```

## 수집품 구매하기

사용자가 수집품을 구매할 수 있도록 하려면 호출 가능한 다른 함수인 `buy_collectible`를 노출해야 합니다. 이 함수는 이전 호출 가능한 함수와 마찬가지로 내부 함수를 사용하여 변경 사항을 스토리지에 기록하기 전에 체크를 수행합니다.

이 워크샵에서 내부 함수는 `do_buy_collectible`이며, 수집품 구매 시도가 성공할지 여부를 결정하는 데 대부분의 작업을 수행합니다.
예를 들어, `do_buy_collectible` 내부 함수는 다음을 확인합니다:

- 제안된 구매 가격이 수집품 소유자가 설정한 가격보다 크거나 같은지 확인하고, 제안된 가격이 너무 낮으면 `BidPriceTooLow` 오류를 반환합니다.
- 수집품이 판매 중인지 확인하고, 수집품 가격이 `None`인 경우 `NotForSale` 오류를 반환합니다.
- 구매자 계정에 수집품 가격을 지불할 수 있는 충분한 잔액이 있는지 확인합니다.
- 구매자 계정이 이미 너무 많은 수집품을 소유하여 다른 수집품을 받을 수 없는지 확인합니다.

모든 체크가 통과되면 `do_buy_collectible` 내부 함수는 계정 잔액을 업데이트하고 `Currency` 트레이트의 transfer 메서드를 사용하여 수집품의 소유권을 이전합니다.

대부분의 작업이 내부 함수에서 수행되므로 공개된 `buy_collectible` 함수는 호출자의 계정을 확인하고 `do_buy_collectible` 함수를 호출하기만 하면 됩니다.

1. `BidPriceTooLow`와 `NotForSale`을 오류 열거형의 변형에 추가하세요.
   
	 ```rust
	 // 팔레트 오류
	 #[pallet::error]
      pub enum Error<T> {
        /// 각 수집품은 고유한 식별자를 가져야 합니다.
        DuplicateCollectible,
        /// 계정은 `MaximumOwned` 상수를 초과할 수 없습니다.
        MaximumCollectiblesOwned,
        /// 수집품의 총 공급은 u64 제한을 초과할 수 없습니다.
        BoundsOverflow,
        /// 수집품이 존재하지 않습니다.
        NoCollectible,
        // 소유자가 아닙니다.
        NotOwner,
        /// 자신에게 수집품을 이전하려고 합니다.
        TransferToSelf,
		/// 입찰 가격이 요청 가격보다 낮습니다.
		BidPriceTooLow,
		/// 수집품이 판매 중이 아닙니다.
		NotForSale,
      }

2. 팔레트에 `Sold` 이벤트를 추가하세요.
   
	 ```rust
	 // 팔레트 이벤트
   #[pallet::event]
	 #[pallet::generate_deposit(pub(super) fn deposit_event)]
      pub enum Event<T: Config> {
        /// 새로운 수집품이 성공적으로 생성되었습니다.
        CollectibleCreated { collectible: [u8; 16], owner: T::AccountId },
        /// 수집품이 성공적으로 이전되었습니다.
        TransferSucceeded { from: T::AccountId, to: T::AccountId, collectible: [u8; 16] },
        /// 수집품의 가격이 성공적으로 설정되었습니다.
        PriceSet { collectible: [u8; 16], price: Option<BalanceOf<T>> },
        /// 수집품이 성공적으로 판매되었습니다.
		Sold { seller: T::AccountId, buyer: T::AccountId, collectible: [u8; 16], price: BalanceOf<T> },
      }
		```

1. 사용자가 수집품을 구매할 때 호출되는 내부 함수를 생성하세요.
   
	 ```rust
	 // 수집품 구매를 위한 내부 함수
	 pub fn do_buy_collectible(
		  unique_id: [u8; 16],
		  to: T::AccountId,
		  bid_price: BalanceOf<T>,
	 ) -> DispatchResult {
	 
	 // 스토리지 맵에서 수집품 가져오기
	 let mut collectible = CollectibleMap::<T>::get(&unique_id).ok_or(Error::<T>::NoCollectible)?;
	 let from = collectible.owner;
	    ensure!(from != to, Error::<T>::TransferToSelf);
		  let mut from_owned = OwnerOfCollectibles::<T>::get(&from);
	 	 
	 // 소유한 수집품 목록에서 수집품 제거하기.
	 if let Some(ind) = from_owned.iter().position(|&id| id == unique_id) {
			from_owned.swap_remove(ind);
		} else {
			return Err(Error::<T>::NoCollectible.into())
		}
	 // 소유한 수집품 목록에 수집품 추가하기.
	 let mut to_owned = OwnerOfCollectibles::<T>::get(&to);
	 to_owned.try_push(unique_id).map_err(|_id| Error::<T>::MaximumCollectiblesOwned)?;
	 // 잔액 이체로 상태를 변경하므로 이후에는 실패할 수 없습니다.
	 if let Some(price) = collectible.price {
			ensure!(bid_price >= price, Error::<T>::BidPriceTooLow);
			// 구매자에서 판매자로 금액 이체
			T::Currency::transfer(&to, &from, price, frame_support::traits::ExistenceRequirement::KeepAlive)?;
			// 판매 이벤트 발생
			Self::deposit_event(Event::Sold {
				seller: from.clone(),
				buyer: to.clone(),
				collectible: unique_id,
				price,
			});
	 } else {
		  return Err(Error::<T>::NotForSale.into())
   }
	 
	 // 이전이 성공적으로 이루어졌으므로 수집품 소유자를 업데이트하고 가격을 `None`으로 재설정합니다.
	 collectible.owner = to.clone();
	 collectible.price = None;
	 // 업데이트를 스토리지에 기록합니다.
	 CollectibleMap::<T>::insert(&unique_id, collectible);
	 OwnerOfCollectibles::<T>::insert(&to, to_owned);
	 OwnerOfCollectibles::<T>::insert(&from, from_owned);
	 Self::deposit_event(Event::TransferSucceeded { from, to, collectible: unique_id });
	 Ok(())
	 }
   ```

1. 사용자가 수집품을 구매할 수 있는 호출 가능한 함수를 추가하세요.
   
	 ```rust
	 /// 수집품을 구매합니다. 입찰 가격은 수집품 소유자가 설정한 가격보다 크거나 같아야 합니다.
	 #[pallet::weight(0)]
	 pub fn buy_collectible(
		 origin: OriginFor<T>,
		 unique_id: [u8; 16],
		 bid_price: BalanceOf<T>,
	 ) -> DispatchResult {
		 // 호출자가 서명된 원본인지 확인합니다.
		 let buyer = ensure_signed(origin)?;
		 // 판매자에서 구매자로 수집품을 이전합니다.
		 Self::do_buy_collectible(unique_id, buyer, bid_price)?;
	 Ok(())
	 }
	 ```

2. 다음 명령을 실행하여 프로그램이 오류 없이 컴파일되는지 확인하세요:
   
   ```bash
   cargo build --package collectibles
   ```