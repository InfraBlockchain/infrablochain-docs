---
title: 디버그
description:
keywords:
---

디버깅은 소프트웨어 개발의 모든 분야에서 필수적이며, 블록체인도 예외는 아닙니다. Rust 디버깅을 위해 사용되는 대부분의 도구는 Substrate에도 적용됩니다.

## 로깅 유틸리티

Rust의 로깅 API를 사용하여 런타임을 디버깅할 수 있습니다. 이를 위해 [`debug`](https://docs.rs/log/0.4.14/log/macro.debug.html)와 [`info`](https://docs.rs/log/0.4.14/log/macro.info.html)를 포함한 여러 매크로를 사용할 수 있습니다.

예를 들어, 팔렛의 `Cargo.toml` 파일을 [`log` 크레이트](https://crates.io/crates/log)로 업데이트한 후에는 `log::info!`를 사용하여 콘솔에 로그를 남길 수 있습니다:<!-- markdown-link-check-disable-line -->

```rust
pub fn do_something(origin) -> DispatchResult {

	let who = ensure_signed(origin)?;
	let my_val: u32 = 777;

	Something::put(my_val);

	log::info!("called by {:?}", who);

	Self::deposit_event(RawEvent::SomethingStored(my_val, who));
	Ok(())
}
```

## Printable trait

Printable trait은 `no_std`와 `std`에서 런타임에서 출력하는 방법을 제공합니다. `print` 함수는 [`Printable` trait](https://paritytech.github.io/substrate/master/sp_runtime/traits/trait.Printable.html)를 구현한 모든 타입과 함께 작동합니다. Substrate는 기본적으로 몇 가지 타입 (`u8`, `u32`, `u64`, `usize`, `&[u8]`, `&str`)에 대해 이 trait을 구현합니다. 또한 사용자 정의 타입에 대해서도 구현할 수 있습니다. 다음은 노드 템플릿을 기반으로 한 팔렛의 `Error` 타입에 대한 구현 예시입니다.

```rust
use sp_runtime::traits::Printable;
use sp_runtime::print;
```

```rust
#[frame_support::pallet]
pub mod pallet {
	// 팔렛의 에러들
	#[pallet::error]
	pub enum Error<T> {
		/// 값이 None인 경우
		NoneValue,
		/// 값이 최대치에 도달하여 더 이상 증가할 수 없는 경우
		StorageOverflow,
	}

	impl<T: Config> Printable for Error<T> {
		fn print(&self) {
			match self {
				Error::NoneValue => "잘못된 값".print(),
				Error::StorageOverflow => "값이 초과되어 오버플로우 발생".print(),
				_ => "잘못된 에러 케이스".print(),
			}
		}
	}
}
```

```rust
/// 매개변수를 받지 않고, 스토리지 값을 증가시키려고 시도하고, 오류가 발생할 수 있음
pub fn cause_error(origin) -> dispatch::DispatchResult {
	// 서명된 것인지 확인하고 서명자를 가져옴. ensure_root와 ensure_none도 참고
	let _who = ensure_signed(origin)?;

	print!("내 테스트 메시지");

	match Something::get() {
		None => {
			print(Error::<T>::NoneValue);
			Err(Error::<T>::NoneValue)?
		}
		Some(old) => {
			let new = old.checked_add(1).ok_or(
				{
					print(Error::<T>::StorageOverflow);
					Error::<T>::StorageOverflow
				})?;
			Something::put(new);
			Ok(())
		},
	}
}
```

RUST_LOG 환경 변수를 사용하여 노드 바이너리를 실행하면 값들이 출력됩니다.

```sh
RUST_LOG=runtime=debug ./target/release/node-template --dev
```

값들은 런타임 함수가 호출될 때마다 터미널이나 표준 출력에 출력됩니다.

```rust
2020-01-01 tokio-blocking-driver DEBUG runtime  내 테스트 메시지  <-- str은 기본적으로 Printable을 구현합니다
2020-01-01 tokio-blocking-driver DEBUG runtime  잘못된 값    <-- NoneValue의 사용자 정의 문자열
2020-01-01 tokio-blocking-driver DEBUG runtime  DispatchError
2020-01-01 tokio-blocking-driver DEBUG runtime  8
2020-01-01 tokio-blocking-driver DEBUG runtime  0                <-- Error enum 정의에서의 인덱스 값
2020-01-01 tokio-blocking-driver DEBUG runtime  NoneValue        <-- 에러의 식별자 이름을 담고 있는 str
```

런타임에 출력 함수를 추가하면 디버그 코드가 포함된 Rust 및 Wasm 바이너리의 크기가 증가하므로 프로덕션에서는 필요하지 않습니다.

## Substrate의 자체 print 함수

레거시 사용 사례에 대해 Substrate는 `Print` 디버깅(또는 추적)을 위한 추가 도구를 제공합니다. [`print` 함수](https://paritytech.github.io/substrate/master/sp_runtime/fn.print.html)를 사용하여 런타임 실행 상태를 로깅할 수 있습니다.

```rust
use sp_runtime::print;

// --snip--
pub fn do_something(origin) -> DispatchResult {
	print!("do_something 실행");

	let who = ensure_signed(origin)?;
	let my_val: u32 = 777;

	Something::put(my_val);

	print!("my_val 저장 후");

	Self::deposit_event(RawEvent::SomethingStored(my_val, who));
	Ok(())
}
// --snip--
```

`RUST_LOG` 환경 변수를 사용하여 체인을 시작하면 출력 로그를 볼 수 있습니다.

```sh
RUST_LOG=runtime=debug ./target/release/node-template --dev
```

값들은 터미널이나 표준 출력에 출력됩니다. 오류가 트리거되면 출력됩니다.

```sh
2020-01-01 00:00:00 tokio-blocking-driver DEBUG runtime  do_something 실행
2020-01-01 00:00:00 tokio-blocking-driver DEBUG runtime  my_val 저장 후
```

## std인 경우

레거시 경우에는 `print` 이상의 작업을 수행하거나, 디버깅 목적으로 Substrate 특정 트레잇을 신경 쓰지 않고 싶을 수 있습니다. 이러한 경우에는 [`if_std!` 매크로](https://paritytech.github.io/substrate/master/sp_std/macro.if_std.html)가 유용합니다.

이 매크로를 사용하는 한 가지 주의점은 내부 코드가 실제로 네이티브 버전의 런타임을 실행할 때만 실행된다는 것입니다.

```rust
use sp_std::if_std; // if_std! 매크로를 스코프에 가져옵니다.
```

`println!` 문은 `if_std` 매크로 내부에 있어야 합니다.

```rust
#[pallet::call]
impl<T: Config<I>, I: 'static> Pallet<T, I> {
		// --snip--
		pub fn do_something(origin) -> DispatchResult {

			let who = ensure_signed(origin)?;
			let my_val: u32 = 777;

			Something::put(my_val);

			if_std! {
				// 이 코드는 `std` 기능이 활성화된 경우에만 컴파일되고 실행됩니다.
				println!("안녕하세요 네이티브 월드!");
				println!("내 값은: {:#?}", my_val);
				println!("호출자 계정은: {:#?}", who);
			}

			Self::deposit_event(RawEvent::SomethingStored(my_val, who));
			Ok(())
		}
		// --snip--
}
```

값들은 런타임 함수가 호출될 때마다 터미널이나 표준 출력에 출력됩니다.

```sh
$		2020-01-01 00:00:00 Substrate Node
		2020-01-01 00:00:00   version x.y.z-x86_64-linux-gnu
		2020-01-01 00:00:00   by Anonymous, 2017, 2020
		2020-01-01 00:00:00 Chain specification: Development
		2020-01-01 00:00:00 Node name: my-node-007
		2020-01-01 00:00:00 Roles: AUTHORITY
		2020-01-01 00:00:00 Imported 999 (0x3d7a…ab6e)
		# --snip--
->		안녕하세요 네이티브 월드!
->		내 값은: 777
->		호출자 계정은: d43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d (5GrwvaEF...)
		# --snip--
		2020-01-01 00:00:00 Imported 1000 (0x3d7a…ab6e)

```
