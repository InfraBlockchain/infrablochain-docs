---
title: 단위 테스트
description: 런타임 로직에 대한 기본 단위 테스트를 설명합니다.
keywords:
---

런타임 로직을 구축하는 동안 로직이 예상대로 작동하는지 정기적으로 테스트하고 싶을 것입니다.
Rust에서 제공하는 [단위 테스트 프레임워크](https://doc.rust-lang.org/rust-by-example/testing/unit_testing.html)를 사용하여 런타임에 대한 단위 테스트를 생성할 수 있습니다.
하나 이상의 단위 테스트를 생성한 후 `cargo test` 명령을 사용하여 테스트를 실행할 수 있습니다.
예를 들어, 다음 명령을 실행하여 런타임에 대해 생성한 모든 테스트를 실행할 수 있습니다:

```shell
cargo test
```

Rust cargo test 명령과 테스트 프레임워크를 사용하는 더 많은 정보를 얻으려면 다음 명령을 실행하세요:

```shell
cargo help test
```

## 모의 런타임에서 팔레트 로그 테스트

Rust 테스트 프레임워크로 수행할 수 있는 단위 테스트 외에도, 모의 런타임 환경을 구성하여 런타임의 로직을 확인할 수 있습니다.
`Test` 구성 타입은 모의 런타임에서 사용되는 팔레트 구성 특성 각각에 대한 Rust enum으로 정의됩니다.

```rust
frame_support::construct_runtime!(
 pub enum Test where
  Block = Block,
  NodeBlock = Block,
  UncheckedExtrinsic = UncheckedExtrinsic,
 {
  System: frame_system::{Pallet, Call, Config, Storage, Event<T>},
  TemplateModule: pallet_template::{Pallet, Call, Storage, Event<T>},
 }
);

impl frame_system::Config for Test {
 // -- 생략 --
 type AccountId = u64;
}
```

`Test`가 `pallet_balances::Config`를 구현한다면, 할당은 `Balance` 타입에 `u64`를 사용할 수 있습니다.
예를 들어:

```rust
impl pallet_balances::Config for Test {
 // -- 생략 --
 type Balance = u64;
}
```

`pallet_balances::Balance`와 `frame_system::AccountId`를 `u64`로 할당함으로써, 테스트 계정과 잔액은 모의 런타임에서 `(AccountId: u64, Balance: u64)` 매핑만 추적하면 됩니다.

## 모의 런타임에서 스토리지 테스트

[`sp-io`](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/master/substrate/primitives/io/src/lib.rs) 크레이트는 모의 환경에서 스토리지를 테스트하기 위해 사용할 수 있는 [`TestExternalities`](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/2e28ab448c5e5e27198ba80b726701479cc982fd/substrate/primitives/state-machine/src/testing.rs#L42C11-L42C11) 구현을 노출합니다.
이는 [`substrate_state_machine`](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/master/substrate/primitives/state-machine/src/lib.rs)에서 인메모리 해시맵 기반의 외부성을 위한 타입 별칭으로 `TestExternalities` 로 참조됩니다.

다음 예제는 `ExtBuilder`라는 구조체를 정의하여 `TestExternalities`의 인스턴스를 빌드하고 블록 번호를 1로 설정하는 방법을 보여줍니다.

```rust
pub struct ExtBuilder;

impl ExtBuilder {
 pub fn build(self) -> sp_io::TestExternalities {
  let mut t = system::GenesisConfig::default().build_storage::<TestRuntime>().unwrap();
  let mut ext = sp_io::TestExternalities::new(t);
  ext.execute_with(|| System::set_block_number(1));
  ext
 }
}
```

단위 테스트에서 테스트 환경을 생성하려면 빌드 메서드를 호출하여 기본 genesis 구성을 사용하여 `TestExternalities`를 생성합니다.

```rust
#[test]
fn fake_test_example() {
 ExtBuilder::default().build_and_execute(|| {
  // ...테스트 로직...
 });
}
```

[Externalities](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/master/substrate/primitives/externalities/src/lib.rs)의 사용자 정의 구현을 통해 외부 노드의 기능에 액세스할 수 있는 런타임 환경을 구축할 수 있습니다.
이와 관련된 또 다른 예는 [`offchain`](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/master/substrate/primitives/core/src/offchain/mod.rs) 모듈에서 찾을 수 있습니다.
`offchain` 모듈은 자체 `Externalities` 구현을 유지합니다.

## 모의 런타임에서 이벤트 테스트

스토리지 외에도 체인에서 발생하는 이벤트를 테스트하는 것도 중요할 수 있습니다.
`deposit_event`를 `generate_deposit` 매크로와 함께 사용하는 경우, 모든 팔레트 이벤트는 `system` / `events` 키 아래에 [`EventRecord`](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/2e28ab448c5e5e27198ba80b726701479cc982fd/substrate/frame/system/src/lib.rs#L725C12-L725C23)로 저장됩니다.

이벤트 레코드는 `System::events()`를 통해 직접 액세스하고 반복할 수 있지만, 테스트에 사용할 수 있는 몇 가지 도우미 메서드도 시스템 팔레트에 정의되어 있습니다. [`assert_last_event`](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/2e28ab448c5e5e27198ba80b726701479cc982fd/substrate/frame/system/src/lib.rs#L1577)와 [`assert_has_event`](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/2e28ab448c5e5e27198ba80b726701479cc982fd/substrate/frame/system/src/lib.rs#L1565)입니다.

```rust
fn fake_test_example() {
 ExtBuilder::default().build_and_execute(|| {
  System::set_block_number(1);
  // ... FakeEvent1을 발생시키는 테스트 로직 ...
  System::assert_has_event(Event::FakeEvent1{}.into())
  System::assert_last_event(Event::FakeEvent2 { data: 7 }.into())
  assert_eq!(System::events().len(), 2);
 });
}
```

주의해야 할 몇 가지 사항은 다음과 같습니다:

- 이벤트는 제네시스 블록에서 발생하지 않으므로, 이 테스트가 통과하려면 블록 번호를 설정해야 합니다.
- 팔레트 이벤트를 인스턴스화한 후 `.into()`를 사용해야 하며, 이렇게 하면 일반 이벤트로 변환됩니다.

### 고급 이벤트 테스트

팔레트에서 이벤트를 테스트할 때, 자주 자신의 팔레트에서 발생한 이벤트만 관심이 있는 경우가 많습니다.
다음 도우미 함수는 이벤트를 필터링하여 사용자 정의 이벤트 유형으로 변환하는 방법을 보여줍니다.
이러한 유형의 도우미 함수는 일반적으로 모의 런타임에서 테스트하기 위한 `mock.rs` 파일에 배치됩니다.

```rust
fn only_example_events() -> Vec<super::Event<Runtime>> {
 System::events()
  .into_iter()
  .map(|r| r.event)
  .filter_map(|e| if let RuntimeEvent::TemplateModule(inner) = e { Some(inner) } else { None })
  .collect::<Vec<_>>();
}
```

또한, 테스트가 시퀀스로 이벤트를 발생시키는 작업을 수행하는 경우, 마지막 확인 이후에 발생한 이벤트만 볼 수 있도록 할 수도 있습니다.
다음 예제는 앞서 언급한 도우미 함수를 활용합니다.

```rust
parameter_types! {
 static ExamplePalletEvents: u32 = 0;
}

fn example_events_since_last_call() -> Vec<super::Event<Runtime>> {
 let events = only_example_events();
 let already_seen = ExamplePalletEvents::get();
 ExamplePalletEvents::set(events.len() as u32);
 events.into_iter().skip(already_seen as usize).collect()
}
```

이렇게 하면 마지막 확인 이후에 발생한 이벤트만 반환합니다.

이전 이벤트 테스트를 이 새로운 함수로 다시 작성하면 다음과 같은 코드가 됩니다.

```rust
fn fake_test_example() {
 ExtBuilder::default().build_and_execute(|| {
  System::set_block_number(1);
  // ... FakeEvent1을 발생시키는 테스트 로직 ...
  assert_eq!(
   example_events_since_last_call(),
   vec![Event::FakeEvent1{}]
  );
  // ... FakeEvent2를 발생시키는 테스트 로직 ...
  assert_eq!(
   example_events_since_last_call(),
   vec![Event::FakeEvent2{}]
  );
 });
}
```

## 제네시스 구성

이전 예제에서 `ExtBuilder::build()` 메서드는 모의 런타임 환경을 구축하기 위해 기본 genesis 구성을 사용했습니다.
많은 경우, 테스트 전에 스토리지를 설정하는 것이 편리합니다.
예를 들어, 테스트 전에 계정 잔액을 미리 설정하고 싶을 수 있습니다.

`frame_system::Config`의 구현에서 `AccountId`와 `Balance` 모두 `u64`로 설정되어 있습니다.
계정 잔액으로 `(AccountId, Balance)` 쌍을 시드로 사용하려면 `balances` 벡터에 `(1, 10)`, `(2, 20)`, `(3, 30)`, `(4, 40)`, `(5, 50)`, `(6, 60)`과 같은 형태로 넣을 수 있습니다.

```rust
impl ExtBuilder {
 pub fn build(self) -> sp_io::TestExternalities {
  let mut t = frame_system::GenesisConfig::default().build_storage::<Test>().unwrap();
  pallet_balances::GenesisConfig::<Test> {
   balances: vec![
    (1, 10),
    (2, 20),
    (3, 30),
    (4, 40),
    (5, 50),
    (6, 60)
   ],
  }
   .assimilate_storage(&mut t)
   .unwrap();

  let mut ext = sp_io::TestExternalities::new(t);
  ext.execute_with(|| System::set_block_number(1));
  ext
 }
}
```

이 예제에서는 계정 1의 잔액이 10, 계정 2의 잔액이 20 등으로 설정됩니다.

팔레트의 제네시스 구성을 정의하는 데 사용되는 정확한 구조는 팔레트의 `GenesisConfig` 구조체 정의에 따라 다릅니다.
예를 들어, Balances 팔레트에서는 다음과 같이 정의됩니다:

```rust
pub struct GenesisConfig<T: Config<I>, I: 'static = ()> {
 pub balances: Vec<(T::AccountId, T::Balance)>,
}
```

## 블록 생성

블록 생성을 시뮬레이션하여 블록 생성 간에 예상한 동작이 유지되는지 확인하는 것이 유용합니다.

이를 수행하는 간단한 방법은 `System` 모듈의 블록 번호를 `on_initialize` 및 `on_finalize` 호출 사이에서 증가시키는 것입니다. 이때 모든 모듈에서 `System::block_number()`를 유일한 입력으로 사용합니다.
런타임 코드에서 스토리지 또는 시스템 모듈 호출을 캐시하는 것이 중요하지만, 테스트 환경 구조는 향후 유지 보수를 용이하게 하기 위해 가독성을 우선시해야 합니다.

```rust
fn run_to_block(n: u64) {
 while System::block_number() < n {
  if System::block_number() > 0 {
   ExamplePallet::on_finalize(System::block_number());
   System::on_finalize(System::block_number());
  }
  System::reset_events();
  System::set_block_number(System::block_number() + 1);
  System::on_initialize(System::block_number());
  ExamplePallet::on_initialize(System::block_number());
 }
}
```

`on_finalize` 및 `on_initialize` 메서드는 각 블록 이전과 이후에 런타임 메서드에 인코딩된 로직을 실행하기 위해 호출됩니다.

그런 다음 다음과 같이 이 함수를 호출합니다.

```rust
#[test]
fn my_runtime_test() {
 with_externalities(&mut new_test_ext(), || {
  assert_ok!(ExamplePallet::start_auction());
  run_to_block(10);
  assert_ok!(ExamplePallet::end_auction());
 });
}
```

## 다음 단계로 넘어가기

<!-- TODO NAV.YAML -->
<!-- add these back -->
<!-- - [팔레트에 대한 테스트 설정](/reference/how-to-guides/testing/) -->