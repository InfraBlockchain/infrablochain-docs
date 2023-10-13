---
title: 런타임 수정하기
description: 기본 노드 템플릿을 수정하여 사용자 정의 런타임을 만드는 간단한 변경 사항을 만들어보세요.
keywords:
---

[코드 탐색](/quick-start/explore-the-code/)에서 기본 노드 템플릿을 구성하는 매니페스트 파일과 Rust 모듈에 대해 배웠습니다.
이제 런타임 소스 코드가 어떻게 구성되어 있는지 대략적으로 알았으니, 몇 가지 간단한 변경 사항을 통해 런타임을 사용자 정의하는 방법을 알아보겠습니다.

이 간단한 데모에서 다음과 같은 작업을 수행할 것입니다:

- 사용하고 싶은 기능을 가진 팔레트를 추가합니다.
- 일부 상수 값을 변경합니다.
- 런타임 버전을 업데이트합니다.
- 변경 사항을 포함한 런타임을 다시 컴파일합니다.
- 체인에 저장된 런타임을 업데이트하기 위해 트랜잭션을 제출합니다.

또한 Polkadot-JS API를 사용하는 다른 응용 프로그램을 보고, 호스팅된 버전의 해당 응용 프로그램을 사용하여 체인 상태를 확인하고 트랜잭션을 제출하는 방법을 알아볼 것입니다.

## 시작하기 전에

`--dev` 커맨드 라인 옵션을 사용하여 개발 모드에서 노드를 실행하면 첫 번째 블록과 함께 깨끗한 상태로 시작됩니다.
런타임을 수정하고 업데이트하는 방법을 가장 잘 설명하기 위해, 기본 노드 템플릿을 기본 런타임으로 다시 시작하여 블록을 생성하도록 해보겠습니다.

기본 런타임으로 노드를 다시 시작하려면 다음 단계를 따르세요:

1. 컴퓨터에서 터미널 쉘을 엽니다.

2. Substrate 노드 템플릿을 컴파일한 루트 디렉토리로 이동합니다.
3. 다음 명령을 실행하여 개발 모드에서 로컬 노드를 시작합니다:

   ```bash
   cargo run --release -- --dev
   ```

노드를 시작한 후, Polkadot-JS API를 사용하여 노드에 연결할 수 있습니다.

실행 중인 노드에 연결하려면 다음 단계를 따르세요:

1. [Polkadot/Substrate Portal](https://polkadot.js.org/apps/#/explorer)을 크롬 또는 크로미움 기반 브라우저에서 엽니다.

   Firefox와 같은 보다 제한적인 브라우저를 사용하는 경우, Polkadot/Substrate Portal과 노드 간의 연결이 차단될 수 있습니다.

2. 개발 네트워크와 기본 로컬 노드 엔드포인트 `127.0.0.1:9944`에 연결합니다.

   대부분의 경우, [Polkadot/Substrate Portal](https://polkadot.js.org/apps/#/explorer)은 실행 중인 로컬 노드에 대한 연결을 자동으로 초기화합니다.
   필요한 경우 **Unknown**을 클릭하여 네트워크 선택 메뉴를 표시하고, **Development**와 **Local Node**를 선택한 다음 **Switch**를 클릭하세요.

3. 개발 환경에서 노드 템플릿 버전이 기본 버전 100임을 확인하세요.

   ![노드 템플릿 기본 버전](/media/images/docs/quickstart-100.png)

## 팔레트 추가하기

Substrate와 FRAME을 사용하여 개발을 시작하는 가장 일반적인 방법은 기존 라이브러리에서 팔레트를 가져오거나 직접 만들어서 팔레트를 추가하는 것입니다.
팔레트를 처음부터 직접 만드는 것은 어렵지 않지만, 응용 프로그램 로직, 스토리지 요구 사항, 오류 처리 등을 설계하는 데 더 많은 작업이 필요합니다.
간단하게 유지하기 위해, 기존 라이브러리에서 팔레트를 가져와 추가해보겠습니다.

기본 노드 템플릿에는 기본적으로 [Utility 팔레트](https://paritytech.github.io/substrate/master/pallet_utility/index.html)가 포함되어 있지 않습니다.
이 팔레트에 사용하려는 기능이 포함되어 있다면, 기본 런타임에 추가할 수 있습니다.

Utility 팔레트를 추가하려면 다음 단계를 따르세요:

1. 컴퓨터에서 두 번째 터미널 쉘을 열고, 노드 템플릿 루트 디렉토리로 이동합니다.
2. 런타임 매니페스트인 `runtime/Cargo.toml` 파일을 코드 편집기에서 엽니다.

3. `[dependencies]` 섹션을 찾아 Utility 팔레트를 종속성으로 추가합니다.

   예를 들어, 다음과 유사한 한 줄을 추가해야 합니다.

   ```toml
   pallet-utility = { version = "4.0.0-dev", default-features = false, git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-vX.Y.Z" }
   ```

   `branch = "polkadot-vX.Y.Z"`를 다른 팔레트에 사용된 Polkadot 브랜치와 동일한 값으로 바꿔야 합니다.

   `pallet-utility` 종속성의 branch 설정이 다른 팔레트의 branch 설정과 동일하도록 하기 위해 기존 팔레트 종속성을 모델로 복사할 수 있습니다.

4. `[features]` 섹션을 찾아 표준 바이너리의 기본 기능 목록에 Utility 팔레트를 추가합니다.

   예를 들어:

   ```toml
   [features]
   default = ["std"]
   std = [
      ...
      "pallet-utility/std",
      ...
   ]
   ```

   [Rust와 WebAssembly](/build/build-process)에서 표준 및 WebAssembly 바이너리에 대한 기능 빌드에 대해 자세히 알아볼 수 있습니다.

5. 변경 사항을 저장하고 `Cargo.toml` 파일을 닫습니다.

6. 코드 편집기에서 `runtime/src/lib.rs` 파일을 엽니다.

7. Utility 팔레트에 대한 `Config` 트레이트 구현을 추가합니다.

   예를 들어:

   ```rust
   impl pallet_utility::Config for Runtime {
      type RuntimeEvent = RuntimeEvent;
      type RuntimeCall = RuntimeCall;
      type PalletsOrigin = OriginCaller;
      type WeightInfo = pallet_utility::weights::SubstrateWeight<Runtime>;
   }
   ```

   각 팔레트는 필요한 특정 매개변수와 유형을 위한 `Config` 트레이트를 가지고 있습니다.
   팔레트의 구성 요구 사항에 대해 자세히 알아보려면 언제든지 팔레트에 대한 Rust 문서를 참조할 수 있습니다.
   예를 들어, [pallet-utility](https://paritytech.github.io/substrate/master/pallet_utility/index.html)의 Rust 문서를 확인할 수 있습니다.

8. `construct_runtime!` 매크로 내부에 Utility 팔레트를 추가합니다.

   예를 들어:

   ```rust
   construct_runtime!(
     pub struct Runtime
     where
        Block = Block,
        NodeBlock = opaque::Block,
        UncheckedExtrinsic = UncheckedExtrinsic
     {
            System: frame_system,
            RandomnessCollectiveFlip: pallet_randomness_collective_flip,
            Timestamp: pallet_timestamp,
            Aura: pallet_aura,
            ...
            Utility: pallet_utility, // 이 줄을 추가합니다
            ...
     }
   ```

   `construct_runtime` 매크로가 작동하는 방식에 대해 자세히 알아보려면 [FRAME 매크로](/reference/frame-macros/)와 [런타임 구성 매크로](/reference/frame-macros/#runtime-construction-macros)를 참조하세요.

## 상수 값 변경하기

기본 노드 템플릿의 Balances 팔레트는 `EXISTENTIAL_DEPOSIT` 상수를 정의합니다.
`EXISTENTIAL_DEPOSIT`는 유효한 활성 계정으로 간주되기 위해 계정이 가져야 하는 최소 잔액을 나타냅니다.
기본적으로 이 상수는 128비트 부호 없는 정수 형식으로 정의되며 값은 500입니다.
간단하게 유지하기 위해, 이 상수의 값을 500에서 1000으로 변경하겠습니다.

상수 값을 업데이트하려면 다음 단계를 따르세요:

1. 코드 편집기에서 `runtime/src/lib.rs` 파일을 엽니다.

2. Balances 팔레트의 `EXISTENTIAL_DEPOSIT`를 찾습니다.

   ```text
   /// Existential deposit.
   pub const EXISTENTIAL_DEPOSIT: u128 = 500;
   ```

3. `EXISTENTIAL_DEPOSIT`의 값을 업데이트합니다.

   ```rust
   pub const EXISTENTIAL_DEPOSIT: u128 = 1000 // 이 값을 업데이트합니다.
   ```

## 런타임 버전 업데이트하기

기본 노드 템플릿에서는 `VERSION` 상수를 사용하여 기본 런타임 버전을 `spec_version`과 100의 값으로 식별합니다.
기본 런타임에 대한 변경 사항을 알리기 위해 `spec_version`을 100에서 101로 변경하겠습니다.

_빠른 시작_ 에서 기본 런타임에 대한 변경 사항을 수행하는 데는 `spec_version`을 업데이트하는 것이 엄격히 필요하지 않습니다.
그러나 버전을 업데이트하면 포크리스 업그레이드를 수행하는 기본 단계를 확인할 수 있습니다.

런타임 버전을 업데이트하려면 다음 단계를 따르세요:

1. 코드 편집기에서 `runtime/src/lib.rs` 파일을 엽니다.
2. `runtime_version` 매크로를 찾습니다.

   ```text
   #[sp_version::runtime_version]
   pub const VERSION: RuntimeVersion = RuntimeVersion {
        spec_name: create_runtime_str!("node-template"),
        impl_name: create_runtime_str!("node-template"),
        authoring_version: 1,
        spec_version: 100,
        impl_version: 1,
        apis: RUNTIME_API_VERSIONS,
        transaction_version: 1,
        state_version: 1,
   };
   ```

3. `spec_version`을 새로운 런타임 버전으로 지정합니다.

   ```rust
   spec_version: 101,  // spec_version을 100에서 101로 변경합니다
   ```

4. 변경 사항을 저장하고 `runtime/src/lib.rs` 파일을 닫습니다.

이 시점에서 런타임 코드를 수정하고 버전 정보를 변경했습니다.
그러나 실행 중인 노드는 여전히 이전에 컴파일된 런타임 버전을 사용하고 있습니다.
[Polkadot/Substrate Portal](https://polkadot.js.org/apps/#/explorer)을 사용하여 실행 중인 노드에 연결 중인 경우, 노드 템플릿 버전이 여전히 기본 버전 100이고 잔액 상수 `existentialDeposit`에 대한 [체인 상태](https://polkadot.js.org/apps/#/chainstate/constants)도 여전히 500임을 확인할 수 있습니다.

![체인 상태](/media/images/docs/quickstart-org-chainstate.png)

## 런타임 다시 컴파일하기

수정한 런타임을 사용하도록 노드 템플릿을 업데이트하기 전에, 런타임을 다시 컴파일해야 합니다.

런타임 패키지를 다시 컴파일하려면 다음 단계를 따르세요:

1. 두 번째 터미널 쉘을 열고, 노드 템플릿을 컴파일한 루트 디렉토리로 이동합니다.
2. 다음 명령을 실행하여 런타임을 다시 컴파일합니다:

   ```shell
   cargo build --release --package node-template-runtime
   ```

   `--release` 커맨드 라인 옵션은 더 긴 컴파일 시간이 필요합니다.
   그러나 블록체인 네트워크에 제출하기에 더 적합한 더 작은 빌드 아티팩트를 생성합니다.
   스토리지 최적화는 어떤 블록체인에 있어서도 _중요_합니다.
   이 명령을 사용하면 빌드 아티팩트가 `target/release` 디렉토리에 출력됩니다.
   WebAssembly 빌드 아티팩트는 `target/release/wbuild/node-template-runtime` 디렉토리에 있습니다.
   예를 들어, `target/release/wbuild/node-template-runtime` 디렉토리의 내용을 나열하면 다음과 같은 WebAssembly 빌드 아티팩트가 표시됩니다:

   ```text
   node_template_runtime.compact.compressed.wasm
   node_template_runtime.compact.wasm
   node_template_runtime.wasm
   ```

## 트랜잭션 제출하기

이제 수정된 런타임을 설명하는 업데이트된 WebAssembly 객체가 있습니다.
그러나 실행 중인 노드는 아직 업그레이드된 런타임을 사용하지 않고 있습니다.
체인에 저장된 런타임을 업데이트하려면, WebAssembly 객체를 변경하는 트랜잭션을 제출해야 합니다.

런타임을 업데이트하려면 다음 단계를 따르세요:

1. [Polkadot/Substrate Portal](https://polkadot.js.org/apps/#/explorer)에서 **Developer**를 클릭하고 **Extrinsics**를 선택합니다.

2. 관리용 **Alice** 계정을 선택합니다.

3. **sudo** 팔레트와 **sudoUncheckedWeight(call, weight)** 함수를 선택합니다.
4. **system**을 선택하고 **setCode(code)**를 호출하는 호출을 Alice 계정을 사용하여 만듭니다.

5. **파일 업로드**를 클릭한 다음, 업데이트된 런타임을 위해 생성한 압축된 WebAssembly 파일인 `node_template_runtime.compact.compressed.wasm`을 선택하거나 드래그하여 업로드합니다.

   예를 들어, `target/release/wbuild/node-template-runtime` 디렉토리로 이동하고 `node_template_runtime.compact.compressed.wasm`을 업로드할 파일로 선택합니다.

6. **weight** 매개변수를 모두 기본값인 `0`으로 설정합니다.

   ![런타임 업그레이드 설정](/media/images/docs/tutorials/forkless-upgrade/set-code-transaction.png)

7. **트랜잭션 제출**을 클릭합니다.

8. 권한을 검토한 후 **Sign and Submit**을 클릭합니다.

## 수정된 런타임 확인하기

트랜잭션이 블록에 포함되면 수정된 런타임을 사용하는지 확인할 수 있습니다.

변경 사항을 확인하려면 다음 단계를 따르세요:

1. [Polkadot/Substrate Portal](https://polkadot.js.org/apps)에서 **Network**를 클릭하고 **Explorer**를 선택하여 성공적인 `sudo.Sudid` 이벤트가 발생했는지 확인합니다.

   ![성공적인 sudo 이벤트](/media/images/docs/tutorials/forkless-upgrade/set-code-sudo-event.png)

2. 노드 템플릿 버전이 이제 `101`인지 확인합니다.

   예를 들어:

   ![업데이트된 런타임 버전은 101입니다](/media/images/docs/quickstart-101.png)

3. **Developer**를 클릭하고 **Extrinsics**를 선택합니다.
4. **다음 엑스트린식 제출**을 클릭하고 목록의 맨 아래로 스크롤하여 **utility** 팔레트가 옵션으로 사용 가능한지 확인합니다.

   ![Utility 팔레트](/media/images/docs/quickstart-utility-pallet.png)

5. **Developer**를 클릭하고 **Chain state**를 선택한 다음 [Constants](https://polkadot.js.org/apps/#/chainstate/constants?rpc=ws://127.0.0.1:9944)를 클릭합니다.
6. **balances** 팔레트를 선택하고 **existentialDeposit**를 선택한 다음 **+**를 클릭하여 상수 값을 쿼리합니다.

   ![상수 값 변경 확인](/media/images/docs/quickstart-chain-state.png)

## 다음 단계

변경 사항을 확인한 후, 사용자 정의 버전의 노드 템플릿이 실행 중이며 로컬 노드를 수정된 런타임을 사용하도록 성공적으로 업그레이드했음을 알게 되었습니다.

이것은 상당한 성과입니다. 그러나 할 수 있는 작업은 훨씬 많습니다.
개념과 핵심 구성 요소에 대해 자세히 알아보려면 [Learn](/learn/) 섹션에서 주제를 검토하거나, [Build](/build/) 섹션에서 이전에 배운 내용을 기반으로 탐색해보세요.