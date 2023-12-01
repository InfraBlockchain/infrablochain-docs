---
title: 런타임에 팔레트 추가하기
description: Substrate 노드 템플릿을 위한 런타임에 간단한 팔레트를 추가하는 기본 단계를 보여줍니다.
keywords:
  - 런타임
  - FRAME
  - 팔레트
  - 의존성
  - 닉네임
  - 설정
---

[로컬 블록체인 만들기](../build-a-blockchain/build-local-blockchain.md)에서 보았듯이, [Substrate 노드 템플릿](https://github.com/substrate-developer-hub/substrate-node-template)은 사용자 정의 블록체인을 구축하기 위한 시작점으로 몇 가지 기본 FRAME 개발 모듈인 **팔레트**를 포함한 작동하는 **런타임**을 제공합니다.

이 튜토리얼에서는 노드 템플릿의 런타임에 새로운 팔레트를 추가하는 기본 단계를 소개합니다.
새로운 FRAME 팔레트를 런타임에 추가하려는 경우 언제든지 비슷한 단계를 수행해야 합니다.
그러나 각 팔레트는 팔레트가 구현하는 기능을 수행하기 위해 필요한 특정 구성 설정(예: 팔레트에서 필요한 특정 매개변수와 유형)이 필요합니다.
이 튜토리얼에서는 노드 템플릿의 런타임에 [Nicks 팔레트](https://paritytech.github.io/substrate/master/pallet_nicks/index.html)를 추가하므로 Nicks 팔레트에 특정한 설정을 구성하는 방법을 알 수 있습니다.
Nicks 팔레트는 블록체인 사용자가 자신이 제어하는 계정에 대해 닉네임을 예약하기 위해 예치금을 지불할 수 있도록 하는 기능을 구현합니다. 다음과 같은 기능을 구현합니다.

- `set_name` 함수는 이름이 이미 사용 중이지 않은 경우 예치금을 수집하고 계정의 이름을 설정합니다.
- `clear_name` 함수는 계정과 연결된 이름을 제거하고 예치금을 반환합니다.
- `kill_name` 함수는 예치금을 반환하지 않고 계정 이름을 강제로 제거합니다.

이 튜토리얼은 더 복잡한 구성 설정을 가진 팔레트를 추가하는 방법, 사용자 정의 팔레트를 생성하는 방법 및 팔레트를 게시하는 방법을 설명하는 고급 튜토리얼로 가는 중간 단계입니다.

## 시작하기 전에

시작하기 전에 다음을 확인하세요.

- [Rust 및 Rust 툴체인](../install/README.md)을 설치하여 Substrate 개발 환경을 구성했는지 확인하세요.

- [로컬 블록체인 만들기](../build-a-blockchain/build-local-blockchain.md) 튜토리얼을 완료하고 개발자 허브에서 Substrate 노드 템플릿을 로컬에 설치했는지 확인하세요.

- 소프트웨어 개발 및 명령 줄 인터페이스 사용에 대해 일반적으로 알고 있는지 확인하세요.

- 블록체인 및 스마트 컨트랙트 플랫폼에 대해 일반적으로 알고 있는지 확인하세요.

## 튜토리얼 목표

이 튜토리얼을 완료함으로써 Nicks 팔레트를 사용하여 다음 목표를 달성할 수 있습니다.

- 새로운 팔레트를 포함하도록 런타임 종속성을 업데이트하는 방법을 배웁니다.

- 팔레트별 Rust 특성을 구성하는 방법을 배웁니다.

- 프론트엔드 템플릿을 사용하여 새로운 팔레트와 상호작용하면서 런타임의 변경 사항을 확인합니다.

## Nicks 팔레트 종속성 추가하기

새로운 팔레트를 사용하기 전에, 컴파일러가 런타임 바이너리를 빌드하는 데 사용하는 구성 파일에 대한 정보를 추가해야 합니다.

Rust 프로그램의 경우, `Cargo.toml` 파일을 사용하여 결과 바이너리를 컴파일하는 데 사용되는 구성 설정 및 종속성을 정의합니다.
Substrate 런타임은 표준 라이브러리 Rust 함수를 포함하는 네이티브 플랫폼 바이너리와 표준 Rust 라이브러리를 포함하지 않는 [WebAssembly (Wasm)](https://webassembly.org/) 바이너리로 컴파일되므로 `Cargo.toml` 파일은 두 가지 중요한 정보를 제어합니다.

- 런타임에 대한 종속성으로 가져올 팔레트의 위치 및 버전을 포함합니다.
- 네이티브 Rust 바이너리를 컴파일할 때 활성화해야 하는 각 팔레트의 기능을 지정합니다. 각 팔레트에서 표준(`std`) 기능 세트를 활성화함으로써 런타임을 컴파일할 때 웹 어셈블리 바이너리를 빌드할 때 누락되는 함수, 유형 및 기본 요소를 포함할 수 있습니다.

`Cargo.toml` 파일에서 종속성을 추가하는 방법에 대한 자세한 내용은 Cargo 문서의 [Dependencies](https://doc.rust-lang.org/cargo/guide/dependencies.html)를 참조하세요.
의존 패키지의 기능을 활성화하고 관리하는 방법에 대한 자세한 내용은 Cargo 문서의 [Features](https://doc.rust-lang.org/cargo/reference/features.html)를 참조하세요.

Nicks 팔레트의 종속성을 런타임에 추가하려면 다음 단계를 수행하세요.

1. 터미널 쉘을 열고 노드 템플릿의 루트 디렉토리로 이동하세요.

2. 텍스트 편집기에서 `runtime/Cargo.toml` 구성 파일을 엽니다.

3. [dependencies] 섹션을 찾고 다른 팔레트가 어떻게 가져오는지 확인하세요.

4. 기존 팔레트 종속성 설명을 복사하고 팔레트 이름을 `pallet-nicks`로 바꿔 팔레트를 노드 템플릿 런타임에서 사용할 수 있도록 합니다.

   예를 들어, 다음과 유사한 줄을 추가하세요.

   ```toml
   pallet-nicks = { version = "4.0.0-dev", default-features = false, git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-v1.0.0" }
   ```

   이 줄은 `pallet-nicks` 크레이트를 종속성으로 가져오고 다음을 지정합니다.

   - 가져올 크레이트의 버전을 식별하는 데 사용되는 버전.
   - 표준 Rust 라이브러리를 사용하여 런타임을 컴파일할 때 팔레트 기능을 포함하는 기본 동작.
   - `pallet-nicks` 크레이트를 검색하기 위한 저장소 위치.
   - 크레이트를 검색하기 위해 사용할 브랜치. Nicks 팔레트에 대해 다른 팔레트에서 사용하는 **버전** 및 **브랜치** 정보와 동일한 정보를 사용해야 합니다.
   
   이러한 세부 정보는 노드 템플릿의 특정 버전에 포함된 팔레트의 모든 팔레트에 대해 동일해야 합니다.

5. `features` 목록에 `pallet-nicks/std` 기능을 추가하세요.

   ```toml
   [features]
   default = ["std"]
   std = [
      ...
      "pallet-aura/std",
      "pallet-balances/std",
      "pallet-nicks/std",
      ...
   ]
   ```

   이 섹션은 이 런타임을 컴파일하는 데 사용되는 기본 기능 세트가 `std` 기능 세트임을 지정합니다.
   `std` 기능 세트를 사용하여 런타임을 컴파일할 때 종속성으로 나열된 모든 팔레트의 `std` 기능이 활성화됩니다.
   표준 Rust 라이브러리를 사용하여 플랫폼 네이티브 바이너리로 컴파일되는 런타임이 어떻게 구성되는지에 대한 자세한 내용은 [Build process](../../build/build-process.md)를 참조하세요.

   `Cargo.toml` 파일의 `features` 섹션을 업데이트하는 것을 잊으면 런타임 바이너리를 컴파일할 때 `cannot find function` 오류가 발생할 수 있습니다.

6. 다음 명령을 실행하여 새로운 종속성이 올바르게 해결되는지 확인하세요.

   ```bash
   cargo check -p node-template-runtime --release
   ```

## Balances 구성 검토하기

각 팔레트에는 `Config`라는 [Rust **trait**](https://doc.rust-lang.org/book/ch10-02-traits.html)가 있습니다.
`Config` trait은 팔레트가 기능을 수행하는 데 필요한 매개변수와 유형을 식별하는 데 사용됩니다.

팔레트를 추가하는 데 필요한 대부분의 팔레트별 코드는 `Config` trait을 사용하여 구현됩니다.
팔레트를 추가하기 위해 어떤 것을 구현해야 하는지는 해당 팔레트의 Rust 문서나 팔레트의 소스 코드를 참조하여 확인할 수 있습니다.
예를 들어, `nicks` 팔레트에 대해 구현해야 하는 내용을 확인하려면 [`pallet_nicks::Config`](https://paritytech.github.io/substrate/master/pallet_nicks/pallet/trait.Config.html)의 Rust 문서나 [Nicks 팔레트 소스 코드](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/nicks/src/lib.rs)를 참조할 수 있습니다.

이 튜토리얼에서는 Balances 팔레트의 `Config` trait을 구현하는 방법을 알아보기 위해 Balances 팔레트를 사용합니다.

Balances 팔레트의 `Config` trait을 검토하려면:

1. 텍스트 편집기에서 `runtime/src/lib.rs` 파일을 엽니다.

2. Balances 팔레트를 찾고 다음과 같은 구현 (`impl`) 코드 블록이 있는지 확인하세요.

   ```text
   pub type Balance = u128;

   // ...

   /// Existential deposit.
   pub const EXISTENTIAL_DEPOSIT: u128 = 500;

   impl pallet_balances::Config for Runtime {
      type MaxLocks = ConstU32<50>;
      type MaxReserves = ();
      type ReserveIdentifier = [u8; 8];
      /// The type for recording an account's balance.
      type Balance = Balance;
      /// The ubiquitous event type.
      type RuntimeEvent = RuntimeEvent;
      /// The empty value, (), is used to specify a no-op callback function.
      type DustRemoval = ();
      /// Set the minimum balanced required for an account to exist on-chain
      type ExistentialDeposit = ConstU128<EXISTENTIAL_DEPOSIT>;
      /// The FRAME runtime system is used to track the accounts that hold balances.
      type AccountStore = System;
      /// Weight information is supplied to the Balances pallet by the node template runtime.
      type WeightInfo = pallet_balances::weights::SubstrateWeight<Runtime>;
   }
   ```

   이 예제에서 볼 수 있듯이, `impl pallet_balances::Config` 블록을 사용하여 Balances 팔레트 `Config` trait에서 지정된 유형과 매개변수를 구성할 수 있습니다.
   예를 들어, 이 `impl` 블록은 Balances 팔레트에서 잔액을 추적하기 위해 `u128` 유형을 사용하도록 구성합니다.

## Nicks 구성 구현하기

Balances 팔레트에 대한 `Config` trait이 어떻게 구현되는지 예제로 확인했으므로 이제 Nicks 팔레트에 대한 `Config` trait을 구현할 준비가 되었습니다.

런타임에 `nicks` 팔레트를 구현하려면 다음 단계를 수행하세요.

1. 텍스트 편집기에서 `runtime/src/lib.rs` 파일을 엽니다.

2. Balances 코드 블록의 마지막 줄을 찾으세요.

3. 다음과 같은 Nicks 팔레트 코드 블록을 추가하세요.

   ```rust
   impl pallet_nicks::Config for Runtime {
    // Balances 팔레트는 ReservableCurrency trait을 구현합니다.
    // `Balances`는 `construct_runtime!` 매크로에서 정의됩니다.
    type Currency = Balances;

    // ReservationFee를 값으로 설정합니다.
    type ReservationFee = ConstU128<100>;

    // 예치금이 손실될 때 아무 조치도 취하지 않습니다.
    type Slashed = ();

    // Nick 팔레트 관리자로 FRAME System Root origin을 구성합니다.
    // https://paritytech.github.io/substrate/master/frame_system/enum.RawOrigin.html#variant.Root
    type ForceOrigin = frame_system::EnsureRoot<AccountId>;

    // 닉네임의 MinLength를 원하는 값으로 설정합니다.
    type MinLength = ConstU32<8>;

    // 닉네임의 MaxLength를 원하는 값으로 설정합니다.
    type MaxLength = ConstU32<32>;

    // The ubiquitous event type.
    type RuntimeEvent = RuntimeEvent;
   }
   ```

4. Nicks를 `construct_runtime!` 매크로에 추가하세요.

   예를 들어:

   ```rust
   construct_runtime!(
   pub enum Runtime where
       Block = Block,
       NodeBlock = opaque::Block,
       UncheckedExtrinsic = UncheckedExtrinsic
     {
       /* --snip-- */
       Balances: pallet_balances,

       /*** 이 줄을 추가하세요 ***/
       Nicks: pallet_nicks,
     }
   );
   ```

5. 변경 사항을 저장하고 파일을 닫으세요.

6. 다음 명령을 실행하여 새로운 종속성이 올바르게 해결되는지 확인하세요.

   ```bash
   cargo check -p node-template-runtime --release
   ```

   오류가 없으면 컴파일할 준비가 되었습니다.

7. 다음 명령을 실행하여 릴리스 모드로 노드를 컴파일하세요.

   ```bash
   cargo build --release
   ```

## 블록체인 노드 시작하기

노드가 컴파일되면, Nicks 팔레트로 기능이 향상된 노드를 시작하고 프론트엔드 템플릿을 사용하여 상호작용할 수 있습니다.

로컬 Substrate 노드를 시작하려면 다음을 수행하세요.

1. 필요한 경우 터미널 쉘을 엽니다.

2. Substrate 노드 템플릿의 루트 디렉토리로 이동하세요.

3. 다음 명령을 실행하여 개발 모드에서 노드를 시작하세요.

   ```bash
   ./target/release/node-template --dev
   ```

   이 경우 `--dev` 옵션은 노드가 미리 정의된 `development` 체인 스펙을 사용하여 개발자 모드에서 실행되도록 지정합니다.
   기본적으로 이 옵션은 노드를 중지할 때(예: Control-c를 누르면) 모든 활성 데이터(키, 블록체인 데이터베이스, 네트워킹 정보 등)를 삭제합니다.
   `--dev` 옵션을 사용하면 노드를 중지하고 다시 시작할 때마다 깨끗한 작업 상태를 유지할 수 있습니다.

4. 터미널에 표시된 출력을 확인하여 노드가 성공적으로 실행되었는지 확인하세요.

   `finalized` 다음의 숫자가 증가하는 것을 확인하면 블록체인이 새로운 블록을 생성하고 그들이 설명하는 상태에 대해 합의에 도달하고 있다는 것을 의미합니다.

5. 노드 출력이 표시되는 터미널을 열어둡니다.

## 프론트엔드 템플릿 시작하기

이제 런타임에 새로운 팔레트를 추가했으므로 [Substrate 프론트엔드 템플릿](../build-a-blockchain/build-local-blockchain.md#프론트엔드-템플릿-설치하기)을 사용하여 노드 템플릿과 상호작용하고 Nicks 팔레트에 액세스할 수 있습니다.

프론트엔드 템플릿을 시작하려면 다음을 수행하세요.

1. 컴퓨터에서 새로운 터미널 쉘을 엽니다.

2. 새 터미널에서 프론트엔드 템플릿을 설치한 루트 디렉토리로 이동하세요.

3. 다음 명령을 실행하여 프론트엔드 템플릿의 웹 서버를 시작하세요.

   ```bash
   yarn start
   ```

4. 브라우저에서 `http://localhost:8000/`을 열어 프론트엔드 템플릿을 확인하세요.

## Nicks 팔레트를 사용하여 닉네임 설정하기

프론트엔드 템플릿을 시작한 후에는 방금 런타임에 추가한 Nicks 팔레트와 상호작용하기 위해 프론트엔드 템플릿을 사용할 수 있습니다.

계정에 닉네임을 설정하려면 다음을 수행하세요.

1. 계정 선택 목록을 확인하여 현재 Alice 계정이 선택되어 있는지 확인하세요.

2. Pallet Interactor 구성 요소에서 **Extrinsic**가 선택되어 있는지 확인하세요.

3. 팔레트 목록에서 **nicks**를 선택하세요.

4. nicks 팔레트에서 호출할 함수로 [**setName**](https://paritytech.github.io/substrate/master/pallet_nicks/pallet/enum.Call.html#variant.set_name)을 선택하세요.

5. `MinNickLength` (8자)보다 길고 `MaxNickLength` (32자)보다 길지 않은 **name**을 입력하세요.

   ![팔레트와 호출할 함수 선택](/media/images/docs/tutorials/add-a-pallet/set-name-function.png)

6. **Signed**를 클릭하여 함수를 실행하세요.

7. 호출의 상태가 준비에서 InBlock으로 변경되고 최종적으로 Finalized로 변경되는 것과 Nicks 팔레트에서 발생한 [이벤트](https://paritytech.github.io/substrate/master/pallet_nicks/pallet/enum.Event.html)를 확인하세요.

   ![Alice의 닉네임 업데이트 성공](/media/images/docs/tutorials/add-a-pallet/set-name-result.png)

## Nicks 팔레트를 사용하여 계정에 대한 정보 조회하기

다음으로, Nicks 팔레트의 런타임 스토리지에서 Alice의 닉네임 값을 읽기 위해 쿼리 기능을 사용할 수 있습니다.

Alice에 대한 저장된 정보를 반환하려면 다음을 수행하세요.

1. Pallet Interactor 구성 요소에서 **Query**를 상호작용 유형으로 선택하세요.

2. 팔레트 목록에서 **nicks**를 선택하세요.

3. 호출할 함수로 [**nameOf**](https://paritytech.github.io/substrate/master/pallet_nicks/pallet/enum.Call.html#variant.set_name)을 선택하세요.

4. AccountId 필드에 **alice** 계정의 주소를 복사하여 붙여넣은 다음 **Query**를 클릭하세요.

   ![이름 읽기](/media/images/docs/tutorials/add-a-pallet/Alice-query-result.png)

   반환 유형은 두 개의 값을 포함하는 튜플입니다.

   - Alice 계정에 대한 16진수로 인코딩된 닉네임 `53756273747261746520737570657273746172202d20416c696365`입니다. 16진수로 인코딩된 값을 문자열로 변환하면 `setName` 함수에 지정한 이름을 볼 수 있습니다.
   - Alice의 계정에서 닉네임을 보호하기 위해 예약된 금액(`100`)입니다.

   Bob의 계정에 대해 Nicks 팔레트에 대한 `nameOf`를 쿼리하면 `None`이 반환됩니다. 이는 Bob이 닉네임을 예약하기 위해 `setName` 함수를 호출하지 않았기 때문입니다.

## 추가 기능 탐색하기

이 튜토리얼은 런타임에 간단한 팔레트를 추가하는 방법을 보여주며, 미리 정의된 프론트엔드 템플릿을 사용하여 새로운 팔레트와 상호작용하는 방법을 보여줍니다.
이 경우 런타임에 `nicks` 팔레트를 추가하고 프론트엔드 템플릿을 사용하여 `set_name` 및 `nameOf` 함수를 호출했습니다.
`nicks` 팔레트는 `clear_name` 함수와 `kill_name` 함수라는 두 가지 추가 기능을 제공하여 계정 소유자가 예약된 이름을 제거하거나 루트 수준 사용자가 계정 이름을 강제로 제거할 수 있도록 합니다.
Nicks 및 Sudo 팔레트를 통해 노출된 추가 기능(예: Sudo 팔레트 및 오리진 계정 사용)을 탐색하여 추가 기능에 대해 자세히 알아볼 수 있습니다.
그러나 이러한 기능은 이 튜토리얼의 목적 범위를 벗어납니다.
Nicks 및 Sudo 팔레트를 통해 노출된 추가 기능을 탐색하려면 [다음 단계](#다음-단계)로 이동하여 호출에 대한 오리진 지정을 선택하세요.

## 다음 단계로 넘어가기

Substrate 개발에 대해 더 알아보기 위해 여러 [튜토리얼](../../tutorials/README.md)을 참조할 수 있습니다.

- [호출에 대한 오리진 지정](../build-application-logic/specify-the-origin-for-a-call.md)은 다른 발신 계정을 사용하여 함수를 호출하는 방법을 탐색합니다.
- [스마트 컨트랙트 개발](../smart-contracts/README.md)은 ink!를 사용하여 스마트 컨트랙트를 구축하는 방법을 안내합니다.
- [사용자 정의 팔레트에서 매크로 사용하기](../build-application-logic/use-macros-in-a-custom-pallet.md)는 매크로를 사용하여 자체 팔레트를 만드는 방법을 보여줍니다.