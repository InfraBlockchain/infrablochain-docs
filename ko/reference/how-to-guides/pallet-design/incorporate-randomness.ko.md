---
title: 무작위성 적용하기
description: 온체인 무작위성 기술과 도구에 대한 자세한 설명입니다.
keywords:
  - 팔렛 디자인
  - 중급
  - 런타임
  - 무작위성
---

무작위성은 컴퓨터 프로그램에서 여러 가지 응용에 사용됩니다. 예를 들어, 게임 응용, NFT 생성 및 블록 작성자 선택에는 모두 어느 정도의 무작위성이 필요합니다.

결정론적인 컴퓨터에서 진정한 무작위성을 얻는 것은 어렵습니다.
특히 블록체인의 경우 네트워크의 모든 노드가 체인의 상태에 대해 동의해야 합니다.
FRAME은 런타임 엔지니어에게 [온체인 무작위성](/build/randomness/)을 제공하며, [무작위성 특성](https://paritytech.github.io/substrate/master/frame_support/traits/trait.Randomness.html)을 사용합니다.

이 가이드에서는 FRAME의 무작위성 특성을 `random` 메서드와 nonce를 주제로 사용하는 방법에 대해 설명합니다.
또한, "random" 타입을 노출하는 팔렛의 구성 특성에 `RandomCollectiveFlip` 팔렛을 할당하여 무작위성 값에 엔트로피를 추가하는 방법도 설명합니다.

## `Randomness` 가져오기

1. 사용하려는 팔렛에서 `frame_support`에서 [`Randomness`](https://paritytech.github.io/substrate/master/frame_support/traits/trait.Randomness.html) 트레이트를 가져옵니다:

   ```rust
   use frame_support::traits::Randomness;
   ```

2. 팔렛의 `Config` 트레이트에 포함시킵니다:

   ```rust
   #[pallet::config]
   pub trait frame_system::Config {
   	type MyRandomness: Randomness<Self::Hash, Self::BlockNumber>;
   }
   ```

   `Randomness` 트레이트는 `Output` 및 `BlockNumber` 타입의 일반적인 반환을 지정합니다.
   팔렛에서 이 트레이트 요구 사항을 충족하기 위해 `frame_system`에서 [`BlockNumber`](https://paritytech.github.io/substrate/master/frame_system/pallet/trait.Config.html#associatedtype.BlockNumber)와 [`Hash`](https://paritytech.github.io/substrate/master/frame_system/pallet/trait.Config.html#associatedtype.Hash)를 사용하세요.

   [이 트레이트의 문서](https://paritytech.github.io/substrate/master/frame_support/traits/trait.Randomness.html)에 명시된 대로, 이 트레이트은 예측하기 어려웠던 무작위성을 제공할 수 있지만, 최근에는 예측하기 쉬워진 무작위성을 제공할 수 있습니다.

## nonce 생성 및 무작위성 구현에 사용하기

`frame_support::traits::Randomness::random(subject: &[u8])` 메서드의 주제로 nonce를 사용합니다.

1. 팔렛에 nonce를 포함시키기 위해 두 가지 단계가 있습니다:

   - **`Nonce` 스토리지 아이템 생성하기.** 스토리지 아이템은 `u32` 또는 `u64` 타입일 수 있습니다.

   - **개인용 nonce 함수 생성하기.** 이 함수는 사용될 때마다 nonce를 증가시킵니다.

   `increment_nonce()` 개인 함수는 nonce를 반환하고 업데이트하는 방식으로 구현할 수 있습니다.
   예를 들면:

   ```rust
   fn get_and_increment_nonce() -> Vec<u8> {
   	let nonce = Nonce::<T>::get();
   	Nonce::<T>::put(nonce.wrapping_add(1));
   	nonce.encode()
   }
   ```

   wrapping 및 encoding 메서드에 대한 자세한 내용은 러스트 문서의 [`wrapping_add`](https://doc.rust-lang.org/std/intrinsics/fn.wrapping_add.html) 및 [`encode`](https://paritytech.github.io/substrate/master/frame_support/dispatch/trait.Encode.html#method.encode)를 참조하세요.

2. 디스패처에서 `Randomness`를 사용합니다.

   nonce를 사용하여 `Randomness`가 노출하는 `random()` 메서드를 호출할 수 있습니다.
   아래 코드 스니펫은 관련 이벤트 및 스토리지 아이템이 구현되었다고 가정한 가짜 예제입니다:

   ```rust
   #[pallet::weight(100)]
   pub fn create_unique(
   	origin: OriginFor<T>)
   	-> DispatchResultWithPostInfo {
   	// 이 디스패처를 호출하는 계정.
   	let sender = ensure_signed(origin)?;
   		// 무작위 값.
   		let nonce = Self::get_and_increment_nonce();
   		let (randomValue, _) = T::MyRandomness::random(&nonce);
   	// 무작위 값을 스토리지에 기록합니다.
   	<MyStorageItem<T>>::put(randomValue);
   	Self::deposit_event(Event::UniqueCreated(randomValue));
   }
   ```

3. 팔렛의 런타임 구현 업데이트하기.

   팔렛의 `Config` 트레이트에 타입을 추가했으므로, `Config`를 사용하여 `Randomness` 트레이트에 의해 파생된 무작위성을 더욱 향상시킬 수 있습니다.
   이는 [Randomness Collective Flip 팔렛](https://paritytech.github.io/substrate/master/pallet_insecure_randomness_collective_flip/index.html)을 사용하여 수행됩니다.

   `Randomness` 트레이트과 함께 이 팔렛을 사용하면 `random()`에서 처리되는 엔트로피가 크게 향상됩니다.

   `runtime/src/lib.rs`에서 `pallet_random_collective_flip`이 `construct_runtime`에서 `RandomCollectiveFlip`으로 인스턴스화된다고 가정하면, 다음과 같이 노출된 타입을 지정합니다:

   ```rust
   impl my_pallet::Config for Runtime{
   	type Event;
   	type MyRandomness = RandomCollectiveFlip;
   }
   ```

## 예제

- [BABE에서 사용된 무작위성](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/babe/src/randomness.rs)
- [FRAME의 Lottery 팔렛](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/lottery/src/lib.rs#L120)

## 관련 자료

- [검증 가능한 무작위 함수](https://en.wikipedia.org/wiki/Verifiable_random_function)