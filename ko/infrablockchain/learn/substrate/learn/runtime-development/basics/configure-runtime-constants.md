---
title: 런타임 상수 구성
description:
keywords:
  - 기초
  - 런타임
  - 상수 구성
---

런타임에서 상수 값을 선언하는 것은 고정된 값 또는 어떤 요소에 따라 동적으로 변경되는 값들을 정의하는 유용한 도구입니다.
이 가이드에서는 `u32` 값을 스토리지에서 재설정하는 데 사용되는 팔레트 상수를 생성하는 방법을 보여줍니다.
이 값은 `SingleValue`라고 부르며, `add_value`라는 메서드를 사용하여 수정할 수도 있습니다.

## 팔레트의 타입, 이벤트 및 오류 구성

1. 팔레트에서 상수를 정의하세요.

   - `MaxAddend`는 메타데이터에 표시되는 값입니다.
   - `ClearFrequency`는 블록 번호를 추적하고 `SingleValue`를 재설정하는 데 사용됩니다.

   ```rust
   #[pallet::config]
   pub trait Config: frame_system::Config {
   	type RuntimeEvent: From<Event<Self>> + IsType<<Self as frame_system::Config>::RuntimeEvent>;

   	#[pallet::constant] // 상수를 메타데이터에 넣습니다.
   	/// 호출당 추가되는 최대 금액입니다.
   	type MaxAddend: Get<u32>;

   	/// 저장된 값이 삭제되는 빈도입니다.
   	type ClearFrequency: Get<Self::BlockNumber>;
   }
   ```

1. 스토리지 항목과 이벤트를 선언하세요.

   저장 속성 매크로를 사용하여 매 블록 주기마다 수정되는 값인 `SingleValue`를 선언하세요.

   ```rust
   #[pallet::storage]
   #[pallet::getter(fn single_value)]
   pub(super) type SingleValue<T: Config> = StorageValue<_, u32, ValueQuery>;
   ```

1. 팔레트의 이벤트를 정의하세요.

   ```rust
   #[pallet::event]
   #[pallet::generate_deposit(pub(super) fn deposit_event)]
   pub enum Event<T: Config> {
   	/// 값이 추가되었습니다. 매개변수는 (초기 금액, 추가된 금액, 최종 금액)입니다.
   	Added(u32, u32, u32),
   	/// 값이 지워졌습니다. 매개변수는 지워지기 전의 값입니다.
   	Cleared(u32)
   }
   ```

1. 오버플로우를 처리하는 오류를 추가하세요.

   ```rust
   #[pallet::error]
   pub enum Error<T> {
   	/// 연산이 오버플로우를 발생시킵니다.
   	Overflow
   }
   ```

## 팔레트 메서드와 런타임 상수 생성

1. `on_finalize`를 구성하세요.

   `on_finalize` 함수에서 `ClearFrequency` 개수의 블록마다 `SingleValue`를 0으로 설정합니다.
   이 로직을 `#[pallet::hooks]` 속성 아래에 지정하세요.

   ```rust
   #[pallet::hooks]
   impl<T: Config> Hooks<BlockNumberFor<T>> for Pallet<T> {
   	fn on_finalize(n: T::BlockNumber) {
   		if (n % T::ClearFrequency::get()).is_zero() {
   			let current_value = <SingleValue<T>>::get();
   			<SingleValue<T>>::put(0u32);
   			Self::deposit_event(Event::Cleared(current_value));
   		}
   	}
   }
   ```

1. 사용자가 값을 지정할 수 있는 메서드를 생성하세요.

   `add_value` 메서드는 각 호출이 `MaxAddend` 값보다 작게 추가될 때까지 `SingleValue`를 증가시킵니다.

   이 메서드에서는 다음을 수행해야 합니다.

   - 체크를 포함합니다.
   - 이전 값 추적합니다.
   - 오버플로우를 확인합니다.
   - `SingleValue`를 업데이트합니다.

   ```rust
   // 런타임 외부에서 호출 가능한 엑스트린시스.
   #[pallet::call]
   impl<T: Config> Pallet<T> {
   	#[pallet::weight(1_000)]
   	fn add_value(
   		origin: OriginFor<T>,
   		val_to_add: u32
   	) -> DispatchResultWithPostInfo {
   		let _ = ensure_signed(origin)?;
   		ensure!(val_to_add <= T::MaxAddend::get(), "value must be <= maximum add amount constant");
   		// 이전 값 가져오기
   		let c_val = SingleValue::<T>::get();
   		// 새 값 추가 시 오버플로우 확인
   		let result = c_val.checked_add(val_to_add).ok_or(Error::<T>::Overflow)?;
   		<SingleValue<T>>::put(result);
   		Self::deposit_event(Event::Added(c_val, val_to_add, result));
   		Ok(().into())
   	}
   }
   ```

1. 런타임에 대한 상수 값을 제공하세요.

   `runtime/src/lib.rs`에서 팔레트의 `MaxAddend`와 `ClearFrequency`에 대한 런타임 구현의 값을 선언하세요.

   ```rust
   parameter_types! {
   	pub const MaxAddend: u32 = 1738;
   	pub const ClearFrequency: u32 = 10;
   }

   impl constant_config::Config for Runtime {
   	type RuntimeEvent = RuntimeEvent;
   	type MaxAddend = MaxAddend;
   	type ClearFrequency = ClearFrequency;
   }
   ```

## 예제

- [`constant-config` 예제 팔레트](https://github.com/substrate-developer-hub/substrate-how-to-guides/blob/main/example-code/template-node/pallets/configurable-constant/src/lib.rs)

## 자원

- [실행 중인 네트워크 업그레이드](/tutorials/build-a-blockchain/upgrade-a-running-network/)
- [Get trait](https://paritytech.github.io/substrate/master/frame_support/traits/trait.Get.html)
- [Extra_constants](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#extra-constants-palletextra_constants-optional)