---
title: 이벤트와 오류
description: 런타임에서 이벤트와 오류를 발생시키는 방법을 설명합니다.
keywords:
---

팔렛은 런타임에서 변경 사항이나 조건에 대한 알림을 사용자, 체인 익스플로러 또는 dApp과 같은 외부 개체에게 보내고자 할 때 이벤트를 발생시킬 수 있습니다.

사용자 정의 팔렛에서는 다음을 정의할 수 있습니다:

- 발생시키고자 하는 이벤트의 유형
- 이벤트에 포함되는 정보
- 이벤트가 발생하는 시점

## 이벤트 선언하기

이벤트는 `#[pallet::event]` 매크로를 사용하여 생성됩니다.
예를 들어:

```rust
#[pallet::event]
#[pallet::generate_deposit(pub(super) fn deposit_event)]
pub enum Event<T: Config> {
	/// 값을 설정합니다.
	ValueSet { value: u32, who: T::AccountId },
}
```

그런 다음, 런타임에서 이벤트를 집계하기 위해 `RuntimeEvent` 타입이 필요합니다.

```rust
#[pallet::config]
	pub trait Config: frame_system::Config {
		/// 전체적인 이벤트 유형입니다.
		type RuntimeEvent: From<Event<Self>> + IsType<<Self as frame_system::Config>::RuntimeEvent>;
	}
```

## 런타임에 이벤트 노출하기

팔렛에서 정의한 모든 이벤트는 `/runtime/src/lib.rs` 파일에서 런타임에 노출되어야 합니다.

이벤트를 런타임에 노출하려면 다음을 수행합니다:

1. 텍스트 편집기에서 `/runtime/src/lib.rs` 파일을 엽니다.
2. 팔렛의 configuration 트레이트에서 `RuntimeEvent` 타입을 구현합니다:

   ```rust
   impl template::Config for Runtime {
   	 type RuntimeEvent = RuntimeEvent;
   }
   ```

3. `construct_runtime!` 매크로에 `RuntimeEvent` 타입을 추가합니다:

   ```rust
   construct_runtime!(
   	 pub enum Runtime where
    	 Block = Block,
   	   NodeBlock = opaque::Block,
   	   UncheckedExtrinsic = UncheckedExtrinsic
   	 {
       // --snip--
   	   TemplateModule: template::{Pallet, Call, Storage, Event<T>},
   	   //--add-this------------------------------------->
   		 }
   );
   ```

   이 예제에서 이벤트는 일반적인 타입이며 `<T>` 매개변수가 필요합니다.
   이벤트가 일반적인 타입을 사용하지 않는 경우 `<T>` 매개변수는 필요하지 않습니다.

## 이벤트 발생시키기

Substrate는 매크로를 사용하여 이벤트를 발생시키는 기본 구현을 제공합니다.
이벤트를 발생시키는 구조는 다음과 같습니다:

```rust
// 1. Event enum을 선언할 때 `generate_deposit` 속성을 사용합니다.
#[pallet::event]
	#[pallet::generate_deposit(pub(super) fn deposit_event)] // <------ 여기 ----
	#[pallet::metadata(...)]
	pub enum Event<T: Config> {
		// --snip--
	}

// 2. 디스패처 함수 내에서 `deposit_event`를 사용합니다.
#[pallet::call]
	impl<T: Config> Pallet<T> {
		#[pallet::weight(1_000)]
		pub(super) fn set_value(
			origin: OriginFor<T>,
			value: u64,
		) -> DispatchResultWithPostInfo {
			let sender = ensure_signed(origin)?;
			// --snip--
			Self::deposit_event(RawEvent::ValueSet(value, sender));
		}
	}
```

이 함수의 기본 동작은 FRAME 시스템에서 [`deposit_event`](https://paritytech.github.io/substrate/master/frame_system/pallet/struct.Pallet.html#method.deposit_event)를 호출하여 이벤트를 스토리지에 기록하는 것입니다.

이 함수는 해당 블록의 시스템 팔렛의 런타임 스토리지에 이벤트를 배치합니다.
새로운 블록이 시작될 때, 시스템 팔렛은 이전 블록에서 저장된 모든 이벤트를 자동으로 제거합니다.

기본 구현을 사용하여 예치된 이벤트는 [Polkadot-JS API](https://github.com/polkadot-js/api)와 같은 하위 라이브러리에서 직접 지원됩니다.
그러나 이벤트를 다르게 처리하고 싶다면 자체 `deposit_event` 함수를 구현할 수도 있습니다.

## 지원되는 타입

이벤트는 [SCALE 코덱](/reference/scale-codec)를 사용하여 타입 인코딩을 지원하는 모든 타입을 발생시킬 수 있습니다.

`AccountId`나 `Balances`와 같은 런타임 제네릭 타입을 사용하려는 경우, 해당 타입을 정의하기 위해 [`where 절`](https://doc.rust-lang.org/rust-by-example/generics/where.html)을 포함해야 합니다.
위의 예제에서 보여진 대로 정의합니다.

## 이벤트 수신하기

Substrate RPC는 이벤트를 직접 쿼리할 수 있는 엔드포인트를 제공하지 않습니다.
기본 구현을 사용한 경우, System 팔렛의 스토리지를 쿼리하여 현재 블록의 이벤트 목록을 볼 수 있습니다.
그렇지 않은 경우, [Polkadot-JS API](https://github.com/polkadot-js/api)는 런타임 이벤트에 대한 WebSocket 구독을 지원합니다.

## 오류

런타임 코드는 명시적으로 모든 오류 상황을 처리해야 합니다.
런타임 코드의 함수는 컴파일러를 [패닉](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html)시키지 않는 오류가 없는 함수여야 합니다.
Rust에서 패닉이 발생하지 않는 코드를 작성하는 일반적인 관용구는 [`Result` 타입](https://paritytech.github.io/substrate/master/frame_support/dispatch/result/enum.Result.html)을 반환하는 함수를 작성하는 것입니다.
`Result` enum 타입은 함수가 성공적으로 실행되지 못했음을 패닉하지 않고 알릴 수 있는 `Err` 변형을 가지고 있습니다.
FRAME 개발 환경에서 런타임으로 디스패치될 수 있는 함수 호출은, 함수가 오류를 만났을 경우 [`DispatchError` 변형](https://paritytech.github.io/substrate/master/frame_support/dispatch/enum.DispatchError.html)일 수 있는 [`DispatchResult` 타입](https://paritytech.github.io/substrate/master/frame_support/dispatch/type.DispatchResult.html)을 반환해야 합니다.

각 FRAME 팔렛은 `#[pallet::error]` 매크로를 사용하여 사용자 정의 `DispatchError`를 정의할 수 있습니다.
예를 들어:

```rust
#[pallet::error]
pub enum Error<T> {
		/// Error names should be descriptive.
		InvalidParameter,
		/// Errors should have helpful documentation associated with them.
		OutOfSpace,
	}
```

FRAME Support 모듈에는 전처리 조건을 확인하고 오류를 발생시킬 수 있는 유용한 [`ensure!` 매크로](https://paritytech.github.io/substrate/master/frame_support/macro.ensure.html)도 포함되어 있습니다.

```rust
frame_support::ensure!(param < T::MaxVal::get(), Error::<T>::InvalidParameter);
```

## 다음으로 이동할 곳

- [Frame 매크로](/reference/frame-macros)
- [Polkadot-JS API](https://github.com/polkadot-js/api).
- [`construct_runtime!` 매크로](https://paritytech.github.io/substrate/master/frame_support/macro.construct_runtime.html)
- [`#[frame_support::pallet]` 매크로](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html)
- [`[pallet::error]` 매크로](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#error-palleterror-optional)