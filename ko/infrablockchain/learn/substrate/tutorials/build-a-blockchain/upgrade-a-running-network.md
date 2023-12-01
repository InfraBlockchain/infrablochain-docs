---
title: 실행 중인 네트워크 업그레이드
description: 실행 중인 노드를 업데이트하는 방법을 설명합니다.
keywords:
  - 포크 없는 업그레이드
  - 런타임 업그레이드
---

Substrate 개발 프레임워크는 블록체인의 핵심인 런타임에 대한 **포크 없는 업그레이드**를 지원합니다.
대부분의 블록체인 프로젝트는 새로운 기능 추가 또는 기존 기능 개선을 위해 코드 베이스의 하드 포크를 필요로 합니다.
Substrate를 사용하면 하드 포크 없이도 새로운 런타임 기능(중단 변경 포함)을 배포할 수 있습니다.
런타임의 정의 자체가 Substrate 기반 체인의 상태 중 하나이기 때문에 네트워크 참가자는 트랜잭션에서 [`set_code`](https://paritytech.github.io/substrate/master/frame_system/pallet/enum.Call.html#variant.set_code) 함수를 호출하여 이 값을 업데이트할 수 있습니다.
런타임 상태의 업데이트는 블록체인의 합의 메커니즘과 암호학적 보장을 사용하여 검증되므로 네트워크 참가자는 체인을 포크하거나 새로운 블록체인 클라이언트를 릴리스할 필요 없이 업데이트된 또는 확장된 런타임 로직을 배포할 수 있습니다.

이 튜토리얼은 코드 베이스의 포크를 생성하지 않고 또는 체인의 진행을 중단하지 않고 런타임을 업그레이드하는 방법을 설명합니다.
이 튜토리얼에서는 실행 중인 네트워크 노드의 Substrate 런타임에 다음과 같은 변경 사항을 적용합니다:

<!--

- 스케줄러 팔렛을 런타임에 추가합니다.
- 수정된 런타임을 실행 중인 노드에 업로드하기 위해 트랜잭션을 제출합니다.
- 스케줄러 팔렛을 사용하여 네트워크 계정의 최소 잔액을 증가시킵니다.

-->

- 런타임의 `spec_version`을 증가시킵니다.
- 유틸리티 팔렛을 런타임에 추가합니다.
- 네트워크 계정의 최소 잔액을 증가시킵니다.

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [Rust 및 Rust 툴 체인](../install/README.md)을 설치하여 Substrate 개발 환경을 구성했습니다.

- [로컬 블록체인 구축](./build-local-blockchain.md)을 완료하고 Substrate 노드 템플릿을 로컬에 설치했습니다.

- 새로운 팔렛을 런타임에 추가하는 방법에 대한 소개를 위해 [팔렛 추가](../build-application-logic/add-a-pallet.md)를 검토했습니다.

## 튜토리얼 목표

이 튜토리얼을 완료함으로써 다음 목표를 달성할 수 있습니다:

- Sudo 팔렛을 사용하여 체인 업그레이드에 대한 거버넌스를 시뮬레이션합니다.

- 실행 중인 노드의 런타임을 새로운 팔렛을 포함하도록 업그레이드합니다.

- 실행 중인 노드에 수정된 런타임을 업로드하기 위해 트랜잭션을 제출합니다.

<!--

- 스케줄러 팔렛을 사용하여 런타임 업그레이드를 예약합니다.

-->

## Sudo를 사용하여 업그레이드 승인

일반적으로 런타임 업그레이드는 커뮤니티 구성원이 업그레이드 제안을 승인하거나 거부하기 위해 거버넌스를 통해 관리됩니다.
이 튜토리얼에서는 Sudo 팔렛과 `Root` origin을 사용합니다. 
이 루트 수준의 관리자만 `set_code` 함수를 호출하여 런타임을 업데이트할 수 있습니다.
Sudo 팔렛을 사용하면 루트 수준의 관리자가 루트 수준의 관리 권한을 가진 계정을 지정하여 `set_code` 함수를 호출할 수 있습니다.

노드 템플릿의 체인 스펙 파일은 기본적으로 `alice` 개발 계정이 Sudo 관리 계정으로 지정되어 있습니다.
따라서 이 튜토리얼에서는 `alice` 계정을 사용하여 런타임 업그레이드를 수행합니다.

### 런타임 업그레이드를 위한 리소스 계산

Substrate 런타임으로의 함수 호출은 항상 리소스 사용량을 나타내는 [Weight](../../learn/basic/glossary.md#가중치)와 관련됩니다.
FRAME 시스템 모듈은 이러한 트랜잭션이 사용할 수 있는 블록 길이와 블록 weight에 대한 제한을 설정합니다.
그러나 `set_code` 함수는 의도적으로 블록에 맞을 수 있는 최대 weight를 소비하도록 설계되었습니다.
런타임 업그레이드를 전체 블록에 소비하도록 강제함으로써 동일한 블록에서 실행되는 트랜잭션이 다른 버전의 런타임에서 실행되는 것을 방지합니다.

`set_code` 함수는`Operational` 클래스로 지정 되어있습니다.
해당 클래스로 식별된 함수 호출은 다음과 같은 특징을 가집니다:

- 블록의 가중치 제한을 전체로 소비할 수 있습니다.
- 최대 우선 순위를 부여받습니다.
- 트랜잭션 수수료를 지불할 필요가 없습니다.

### 리소스 계산 관리

이 튜토리얼에서는 `sudo_unchecked_weight` 함수를 사용하여 런타임 업그레이드를 위해 `set_code` 함수를 호출합니다.
`sudo_unchecked_weight` 함수는 `sudo` 함수와 동일하지만 호출에 사용할 weight를 지정하기 위해 추가 매개변수를 지원합니다.
이 매개변수를 사용하여 리소스 계산의 제한을 우회하여 `set_code` 함수를 디스패치하는 호출에 대해 가중치를 0으로 지정할 수 있습니다.
이 설정은 런타임 업그레이드가 얼마나 복잡한 작업이든 트랜잭션이 실패하지 않도록 합니다.

## 유틸리티 팔렛을 런타임에 추가

노드 템플릿은 기본적으로 런타임에 [유틸리티 팔렛](https://paritytech.github.io/substrate/master/pallet_utility/index.html)을 포함하지 않습니다.
런타임 업그레이드를 설명하기 위해 실행 중인 노드에 유틸리티 팔렛을 추가할 수 있습니다.

### 로컬 노드 시작

이 튜토리얼은 실행 중인 노드를 업데이트하는 방법을 설명하기 때문에 첫 번째 단계는 현재 런타임을 사용하여 로컬 노드를 시작하는 것입니다.

현재 런타임으로 노드를 시작하려면 다음 단계를 수행하세요:

1. 컴퓨터에서 터미널 셸을 엽니다.

1. Substrate 노드 템플릿을 컴파일한 루트 디렉토리로 이동합니다.

1. 다음 명령을 실행하여 이전에 컴파일한 로컬 노드를 개발 모드로 시작합니다.

   ```bash
   cargo run --release -- --dev
   ```

   이 노드를 실행한 상태로 둡니다.
   실행 중인 노드를 중지하거나 다시 시작하지 않고 런타임을 업그레이드하기 위해 편집하고 다시 컴파일할 수 있습니다.

1. 브라우저에서 [Polkadot/Substrate 포털](https://polkadot.js.org/apps/#/explorer)을 열고 로컬 노드에 연결합니다.

1. 가장 왼쪽의 드롭다운 메뉴를 클릭하여 네트워크를 선택합니다.

   ![네트워크 선택](/media/images/docs/infrablockchain/learn/substrate/tutorials/polkadot-js-select-network.png)

1. **Development** 아래에서 **Local Node**를 선택한 다음 **Switch**를 클릭합니다.

   ![네트워크 선택](/media/images/docs/infrablockchain/learn/substrate/tutorials/select-local-node.png)
   
   왼쪽 상단에서 노드 템플릿 버전이 기본 버전 100임을 알 수 있습니다.

   ![노드 템플릿 버전](/media/images/docs/infrablockchain/learn/substrate/tutorials/default-version.png)

### 런타임에 유틸리티 팔렛 추가

런타임을 업데이트하여 유틸리티 팔렛을 포함하도록 하려면 다음 단계를 수행하세요:

1. 두 번째 터미널 셸 창이나 탭을 엽니다.

1. Substrate 노드 템플릿을 컴파일한 루트 디렉토리로 이동합니다.

1. `runtime/Cargo.toml` 파일을 텍스트 편집기로 엽니다.

1. `[dependencies]` 섹션을 찾습니다.
   
   예를 들어:

   ```text
   [dependencies]
   codec = { package = "parity-scale-codec", version = "3.0.0", default-features = false, features = ["derive"] }
   scale-info = { version = "2.1.1", default-features = false, features = ["derive"] }
   
   pallet-aura = { version = "4.0.0-dev", default-features = false, git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-v1.0.0" }
   ```

1. 유틸리티 팔렛을 추가합니다.
   
   예를 들어, 다음 필드를 추가합니다:
   
   ```toml
   pallet-utility = {
      version = "4.0.0-dev",
      default-features = false,
      git = "https://github.com/paritytech/polkadot-sdk.git",
      branch = "polkadot-v1.0.0"
   }
   ```

1. `[features]` 섹션과 표준 바이너리의 기본 기능 목록을 찾습니다.
   
   예를 들어:

   ```text
   [features]
   default = ["std"]
   std = [
      "frame-try-runtime?/std",
      "frame-system-benchmarking?/std",
      "frame-benchmarking?/std",
      "codec/std",
      "scale-info/std",
   ```

1. 목록에 유틸리티 팔렛을 추가합니다.
   
   ```toml
   "pallet-utility/std",
   ```

1. 변경 사항을 저장하고 `Cargo.toml` 파일을 닫습니다.


<!--

*** 스케줄러 팔렛이 이 튜토리얼에 다시 추가되면, 유틸리티 팔렛 단계 대신 스케줄러 팔렛 단계로 이동합니다. ***

1. 목록에 스케줄러 팔렛을 추가합니다.
   
   ```toml
   "pallet-scheduler/std",
   ```

1. 스케줄러 팔렛을 종속성으로 추가합니다.
   
   예를 들어, 다음 필드를 포함하는 단일 줄을 추가합니다:
   
   ```toml
   pallet-scheduler = { 
      version = "4.0.0-dev", 
      default-features = false, 
      git = "https://github.com/paritytech/polkadot-sdk.git", 
      branch = "polkadot-v1.0.0" 
   }
   ```

   런타임에 포함된 다른 팔렛들과 동일한 **version** 및 **branch** 정보를 사용하도록 주의하세요.
   이 예제에서 노드 템플릿 런타임의 모든 팔렛은 `version = "4.0.0-dev"` 및 `branch = "polkadot-v1.0.0"`를 사용합니다.
-->

### 다시 컴파일하고 로컬 노드에 연결

1. 첫 번째 터미널에서 로컬 노드가 계속 실행 중인지 확인합니다.
   
1. 런타임 `Cargo.toml` 및 `lib.rs` 파일을 업데이트한 두 번째 터미널에서 다음 명령을 실행하여 런타임을 다시 컴파일합니다.

   ```shell
   cargo build --release --package node-template-runtime
   ```

   `--release` 명령행 옵션은 더 긴 컴파일 시간이 필요합니다.
   그러나 더 작은 빌드 아티팩트를 생성하여 블록체인 네트워크에 제출하기에 더 적합합니다.
   스토리지 최적화는 어떤 블록체인이든 _중요_합니다.
   이 명령으로 빌드 아티팩트가 `target/release` 디렉토리에 출력됩니다.
   WebAssembly 파일의 결과물은 `target/release/wbuild/node-template-runtime` 디렉토리에 있습니다.
   예를 들어, 다음과 같은 WebAssembly 결과물을 볼 수 있어야 합니다:

   ```text
   node_template_runtime.compact.compressed.wasm
   node_template_runtime.compact.wasm
   node_template_runtime.wasm
   ```

## 런타임 업그레이드 실행

이제 수정된 런타임 로직을 설명하는 WebAssembly가 있습니다.
그러나 실행 중인 노드는 아직 업그레이드된 런타임을 사용하지 않습니다.
업그레이드를 완료하려면 업그레이드된 런타임을 사용하도록 노드에 트랜잭션을 제출해야 합니다.

네트워크를 업그레이드하려면 다음 단계를 수행하세요:

1. [Polkadot/Substrate 포털](https://polkadot.js.org/apps/#/explorer)을 브라우저에서 열고 로컬 노드에 연결합니다.

1. **Developer**를 클릭하고 **Extrinsics**를 선택하여 런타임이 새 빌드 아티팩트를 사용하도록 트랜잭션을 제출합니다.

1. 관리자 **Alice** 계정을 선택합니다.

1. **sudo** 팔렛과 **sudoUncheckedWeight(call, weight)** 함수를 선택합니다.
   
1. **system**과 **setCode(code)**를 호출로 사용하도록 설정합니다.

1. **파일 업로드**를 클릭한 다음 생성한 압축된 WebAssembly 파일(`node_template_runtime.compact.compressed.wasm`)을 선택하거나 끌어다 놓습니다.

   예를 들어, `target/release/wbuild/node-template-runtime` 디렉토리로 이동한 다음 `node_template_runtime.compact.compressed.wasm`을 업로드할 파일로 선택합니다.

1. **weight** 매개변수를 기본값인 `0`으로 설정한 채로 두세요.

   ![런타임 업그레이드 설정](/media/images/docs/infrablockchain/learn/substrate/tutorials/set-code-transaction.png)

2. **트랜잭션 제출**을 클릭합니다.

3. 권한을 검토한 다음 **서명 및 제출**을 클릭합니다.

4. **Network**를 클릭하고 **Explorer**를 선택하여 성공적인 `sudo.Sudid` 이벤트가 발생했는지 확인합니다.
   
   ![성공적인 sudo 이벤트](/media/images/docs/infrablockchain/learn/substrate/tutorials/set-code-sudo-event.png)   

   트랜잭션이 블록에 포함된 후, 노드 템플릿 버전 번호는 이제 런타임 버전이 `101`임을 나타냅니다.
   예를 들어:

   ![업데이트된 런타임 버전은 101입니다](/media/images/docs/infrablockchain/learn/substrate/tutorials/runtime-version-101.png)

   로컬 노드가 브라우저에 표시된 내용과 일치하는 블록을 생성하는 경우, 런타임 업그레이드를 성공적으로 완료한 것입니다.

5. **Developer**를 클릭하고 **Extrinsics**를 선택합니다. *다음 익스트린식을 제출*을 클릭하고 목록의 맨 아래로 스크롤합니다. **유틸리티**가 옵션으로 표시되는 것을 볼 수 있습니다.
   
<!--

## 업그레이드 예약

이전 업그레이드 예에서는 `sudo_unchecked_weight` 함수를 사용하여 블록 길이와 가중치를 제한하는 계산 보호장치를 우회하여 `set_code` 함수 호출이 런타임 업그레이드를 완료할 수 있도록 했습니다.
그러나 이제 스케줄러 팔렛을 포함한 노드 템플릿을 구성했으므로 **예약된** 런타임 업그레이드를 수행할 수 있습니다. 
예약된 런타임 업그레이드는 `set_code` 함수 호출이 포함된 트랜잭션이 블록에 포함된 유일한 트랜잭션이 되도록 보장합니다.

참고: 스케줄러 팔렛은 자체 가중치 소비를 계량하기 위해 최근에 수정되었습니다. 
그러나 현재로서는 `set_code` 함수가 블록 전체를 소비하도록 작성되었으므로 스케줄된 작업을 수행하려고 하면 `Exhausted` 디스패치 오류가 발생하여 실패합니다. 
왜냐하면 블록 가중치가 허용되는 가중치를 초과하기 때문입니다.
따라서 현재로서는 예약된 작업으로 런타임을 업그레이드할 수 없습니다.

### 업그레이드할 런타임 준비

이전 업그레이드 예에서는 런타임에 완전히 새로운 팔렛을 추가했습니다.
이 업그레이드 예제는 더 간단하며 이미 존재하는 값만 업데이트하는 것으로 충분합니다.
이 업그레이드 예제에서는 네트워크 계정에 필요한 최소 잔액을 증가시키려고 합니다.
이 최소 잔액은 런타임의 상수이며 기본적으로 노드 템플릿에서는 500의 값을 가집니다.

런타임 업그레이드를 위해 최소 잔액 상수 값을 수정하려면 다음 단계를 수행하세요:

1. 첫 번째 터미널에서 로컬 노드가 계속 실행 중인지 확인합니다.

1. 두 번째 터미널 셸 창이나 탭을 엽니다.

1. Substrate 노드 템플릿을 컴파일한 루트 디렉토리로 이동합니다.

3. `runtime/src/lib.rs` 파일을 텍스트 편집기로 엽니다.

1. 런타임의 `spec_version`을 102로 업데이트합니다.
   
   ```rust
   pub const VERSION: RuntimeVersion = RuntimeVersion {
      spec_name: create_runtime_str!("node-template"),
      impl_name: create_runtime_str!("node-template"),
      authoring_version: 1,
      spec_version: 102,  // 이 값을 증가시킵니다.
      impl_version: 1,
      apis: RUNTIME_API_VERSIONS,
      transaction_version: 1,
      state_version: 1,
   };

1. Balances 팔렛의 EXISTENTIAL_DEPOSIT 값을 업데이트합니다.
   
   ```rust
   pub const EXISTENTIAL_DEPOSIT: u128 = 1000 // 이 값을 업데이트합니다.
   ```
   
   이 변경은 유효한 활성 계정으로 간주되기 위해 계정이 보유해야 하는 최소 잔액을 증가시킵니다.
   이 변경은 500에서 1000 사이의 잔액을 가진 계정을 제거하지 않습니다.
   계정을 제거하려면 스토리지 마이그레이션이 필요합니다.
   데이터 스토리지를 업그레이드하는 방법에 대한 자세한 내용은 [스토리지 마이그레이션](/maintain/runtime-upgrades/#storage-migrations)을 참조하세요.

1. 변경 사항을 저장하고 `runtime/src/lib.rs` 파일을 닫습니다.

1. 다음 명령을 실행하여 업그레이드된 런타임을 빌드합니다:
   
   ```bash
   cargo build --release --package node-template-runtime
   ```
   
   이 명령은 이전 빌드 아티팩트 세트를 덮어쓰는 새로운 빌드 아티팩트 세트를 생성합니다.
   이전의 아티팩트 세트를 저장하려면 노드 템플릿을 컴파일하기 전에 다른 위치로 복사하세요.

1. WebAssembly 빌드 아티팩트가 `target/release/wbuild/node-template-runtime` 디렉토리에 있는지 확인합니다.

### 예약된 업그레이드를 위한 트랜잭션 제출

이제 수정된 런타임 로직을 설명하는 WebAssembly 아티팩트가 있습니다.
스케줄러 팔렛은 `Root` 오리진을 [`ScheduleOrigin`](https://paritytech.github.io/substrate/master/pallet_scheduler/pallet/trait.Config.html#associatedtype.ScheduleOrigin)으로 구성했으므로 `sudo` 함수가 아닌 `schedule` 함수를 호출하는 데 사용할 수 있습니다.

런타임 업그레이드를 예약하려면 다음 단계를 수행하세요:

1. [Polkadot/Substrate 포털](https://polkadot.js.org/apps/#/explorer)을 브라우저에서 열고 로컬 노드에 연결합니다.

2. **Developer**를 클릭하고 **Sudo**를 선택합니다.

3. **scheduler**와 **schedule(when, maybePeriodic, priority, call)**을 선택합니다.

   - **when** 매개변수는 예약된 작업을 수행할 블록 번호를 지정합니다.
   - **maybePeriodic** 매개변수는 선택 사항이므로 기본값(빈 값)을 사용할 수 있습니다.
   - **priority** 매개변수에는 기본값(0)을 사용합니다.
   - **system**과 **setCode(code)**를 호출로 사용하도록 설정합니다.
   - **파일 업로드**를 클릭하고 업그레이드된 런타임을 위해 생성한 압축된 WebAssembly 파일을 선택합니다.

4. 현재 블록 번호를 확인하고 **when** 매개변수를 현재 블록에서 10에서 20 블록 뒤의 블록 번호로 설정한 다음 **Submit Sudo**를 클릭합니다.
   
   ![런타임 업그레이드를 예약하기 위한 설정](/media/images/docs/tutorials/forkless-upgrade/sudo-scheduler-upgrade.png)

5. 권한을 검토하고 **서명 및 제출**을 클릭합니다.

6. 터미널에서 블록 생성을 모니터하거나 [네트워크 탐색기](https://polkadot.js.org/apps/#/explorer?rpc=ws://127.0.0.1:9944)에서 이 예약된 호출이 수행되는 것을 확인합니다.
   
   대상 블록이 체인에 포함된 후, 노드 템플릿 버전 번호는 이제 런타임 버전이 `102`임을 나타냅니다.

-->

1. [Polkadot/Substrate 포털](https://polkadot.js.org/apps/#/explorer?rpc=ws://127.0.0.1:9944)에서 체인 상태를 쿼리하여 상수 값을 확인합니다.
   
   - **Developer**를 클릭하고 **Chain state**를 선택합니다.
   - **Constants**를 클릭합니다.
   - **balances** 팔렛을 선택합니다.
   - 상수 값을 쿼리할 값으로 **existentialDeposit**를 선택합니다.

   ![상수 값 변경 확인](/media/images/docs/infrablockchain/learn/substrate/tutorials/constant-value-lookup.png)
   
## 다음 단계로 넘어가기

- [런타임 버전 101](/assets/tutorials/runtime-upgrade/lib-spec-version-101.rs)
- [런타임 버전 102](/assets/tutorials/runtime-upgrade/lib-spec-version-102.rs)
- [스토리지 마이그레이션](/maintain/runtime-upgrades/#storage-migration)

<!-- - [How-to: Storage migration](/reference/how-to-guides/basics/storage-migration/) -->