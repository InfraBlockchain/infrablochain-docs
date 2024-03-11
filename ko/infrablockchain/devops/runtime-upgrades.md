---
title: 런타임 업그레이드
description: 런타임 버전 및 스토리지 마이그레이션은 Substrate 기반 네트워크의 포크 없는 업그레이드를 지원하는 방법을 설명합니다.
keywords:
---

포크 없는 런타임 업그레이드는 블록체인 개발을 위한 Substrate 프레임워크의 핵심 기능입니다. 코드 베이스를 포크하지 않고 런타임 로직을 업데이트할 수 있는 능력은 블록체인이 시간이 지남에 따라 진화하고 개선될 수 있도록 합니다. 이 기능은 블록체인의 런타임 상태 요소인 런타임 WebAssembly 블롭의 정의를 블록체인의 런타임 상태에 포함시킴으로써 가능해집니다.

런타임이 블록체인 상태의 일부이기 때문에 네트워크 관리자는 신뢰할 수 있는 탈중앙화된 합의 메커니즘을 활용하여 런타임을 안전하게 개선할 수 있습니다.

런타임 개발을 위한 FRAME 시스템에서는 시스템 라이브러리가 런타임 정의를 업데이트하는 데 사용되는 `set_code` 호출을 정의합니다. [실행 중인 네트워크 업그레이드](/tutorials/build-a-blockchain/upgrade-a-running-network/) 튜토리얼에서는 노드를 종료하거나 작업을 중단하지 않고 런타임을 업그레이드하는 두 가지 방법을 보여줍니다. 그러나 튜토리얼의 두 가지 업그레이드 모두 런타임에 기능을 추가하는 것을 보여주는 것이지, 기존 런타임 상태를 업데이트하는 것은 아닙니다. 런타임 업그레이드가 기존 상태의 변경을 필요로 하는 경우 스토리지 마이그레이션이 필요할 가능성이 높습니다.

### 런타임 버전 관리

[빌드 프로세스](/main-docs/build/build-process/)에서는 노드를 컴파일하면 플랫폼별 바이너리와 WebAssembly 바이너리가 생성되며, 블록 생성 프로세스의 다른 지점에서 어떤 바이너리를 사용할지 선택할 수 있는 실행 전략 명령줄 옵션을 통해 제어할 수 있다는 것을 배웠습니다. 런타임 실행 환경과 통신하기 위해 사용되는 구성 요소는 **실행자(executor)** 라고 합니다. 사용자 정의 시나리오에서 기본 실행 전략을 재정의할 수는 있지만 대부분의 경우 실행자는 네이티브 런타임과 WebAssembly 런타임의 다음 정보를 평가하여 사용할 바이너리를 선택합니다.

- `spec_name`
- `spec_version`
- `authoring_version`

실행자 프로세스에 이 정보를 제공하기 위해 런타임에는 다음과 유사한 [런타임 버전 구조체](https://paritytech.github.io/substrate/master/sp_version/struct.RuntimeVersion.html)가 포함되어 있습니다.

```rust
pub const VERSION: RuntimeVersion = RuntimeVersion {
  spec_name: create_runtime_str!("node-template"),
  impl_name: create_runtime_str!("node-template"),
  authoring_version: 1,
  spec_version: 1,
  impl_version: 1,
  apis: RUNTIME_API_VERSIONS,
  transaction_version: 1,
};
```

구조체의 매개변수는 다음과 같은 정보를 제공합니다.

| 매개변수            | 제공하는 내용                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `spec_name`         | 서로 다른 Substrate 런타임을 식별하는 식별자입니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `impl_name`         | 사양의 구현 이름입니다. 이는 노드에 대해서는 큰 의미가 없으며 다른 구현 팀의 코드를 구분하기 위한 것입니다.                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `authoring_version` | 작성 노드는 이 값이 네이티브 런타임과 동일하지 않으면 블록을 작성하지 않습니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `spec_version`      | 런타임 사양의 버전입니다. 전체 노드는 웹 어셈블리 런타임 대신 네이티브 런타임을 사용하지 않습니다. `spec_name`, `spec_version`, `authoring_version`이 웹 어셈블리와 네이티브 바이너리 모두에서 동일한 경우에만 업데이트합니다. `spec_version`의 업데이트는 CI 프로세스로 자동화할 수 있으며, [Polkadot 네트워크](https://gitlab.parity.io/parity/mirrors/polkadot/-/blob/master/scripts/ci/gitlab/check_extrinsics_ordering.sh)에서 수행됩니다. 이 매개변수는 일반적으로 `transaction_version`의 업데이트가 있는 경우에 증가합니다. |
| `impl_version`      | 사양의 구현 버전입니다. 노드는 이 값을 무시합니다. 이 값은 코드가 다른지를 나타내기 위해 사용됩니다. `authoring_version`과 `spec_version`이 동일한 경우 코드 자체가 변경되었을 수 있지만 네이티브와 웹 어셈블리 바이너리는 동일한 작업을 수행합니다. 일반적으로 `impl_version`의 변경은 논리를 깨뜨리지 않는 최적화에 의해 발생합니다.                                                                                                                                                                                                    |
| `transaction_version` | 트랜잭션 처리 인터페이스의 버전입니다. 이 매개변수는 하드웨어 지갑이 런타임 트랜잭션을 안전하게 서명할 수 있는지 확인하기 위해 하드웨어 지갑이 펌웨어 업데이트를 동기화하는 데 유용합니다. 이 번호는 `construct_runtime!` 매크로에서 팔렛의 인덱스나 디스패치 가능한 함수의 매개변수 또는 매개변수 유형의 변경이 있는 경우 업데이트해야 합니다. 이 번호가 업데이트되면 `spec_version`도 업데이트해야 합니다. |
| `apis`              | 지원되는 [런타임 API](https://paritytech.github.io/substrate/master/sp_api/macro.impl_runtime_apis.html) 목록과 해당 버전입니다.                                                                                                                                                                                                                                                                                                                                                                                                                                                            |

오케스트레이션 엔진(때로는 실행자라고도 함)은 네이티브 런타임이 WebAssembly 이전에 실행되기 전에 웹 어셈블리와 동일한 합의 기반 논리를 갖고 있는지 확인합니다. 그러나 런타임 버전 관리는 수동으로 설정되기 때문에 런타임 버전이 잘못 표시된 경우 오케스트레이션 엔진이 부적절한 결정을 내릴 수 있습니다.

### 런타임 버전에 접근하기

FRAME 시스템은 `state.getRuntimeVersion` RPC 엔드포인트를 통해 런타임 버전 정보를 노출합니다. 이 엔드포인트는 선택적인 블록 식별자를 허용합니다. 그러나 대부분의 경우 런타임 [메타데이터](/main-docs/build/application-development/#exposing-runtime-information-as-metadata)를 사용하여 런타임이 노출하는 API와 이러한 API와 상호작용하는 방법을 이해하는 데 사용합니다. 런타임 메타데이터는 체인의 [런타임 `spec_version`](https://paritytech.github.io/substrate/master/sp_version/struct.RuntimeVersion.html#structfield.spec_version)이 변경될 때만 변경되어야 합니다.

### 포크 없는 런타임 업그레이드

전통적인 블록체인은 체인의 상태 전이 함수를 업그레이드할 때 [하드 포크](<https://en.wikipedia.org/wiki/Fork_(blockchain)>)가 필요합니다. 하드 포크는 모든 노드 운영자가 노드를 중지하고 최신 실행 파일로 수동으로 업그레이드해야 하는 것을 의미합니다. 분산된 프로덕션 네트워크에서 하드 포크 업그레이드의 조정은 복잡한 과정일 수 있습니다.

런타임 버전 관리 속성은 Substrate 기반 블록체인이 네트워크에서 포크 없이 런타임 로직을 실시간으로 업그레이드할 수 있도록 합니다.

포크 없는 런타임 업그레이드를 수행하기 위해 Substrate는 기존 합의를 깨는 새로운 버전의 로직으로 블록체인에 저장된 WebAssembly 런타임을 업데이트합니다. 이 업그레이드는 합의 프로세스의 일부로 네트워크의 모든 전체 노드에 전파됩니다. WebAssembly 런타임이 업그레이드되면 오케스트레이션 엔진은 네이티브 런타임의 `spec_name`, `spec_version`, 또는 `authoring_version`이 새로운 WebAssembly 런타임과 일치하지 않음을 감지합니다. 결과적으로 오케스트레이션 엔진은 모든 실행 프로세스에서 네이티브 런타임 대신 표준 WebAssembly 런타임을 실행합니다.

## 스토리지 마이그레이션

**스토리지 마이그레이션**은 런타임의 변경에 적응하기 위해 스토리지를 업데이트하는 사용자 정의 일회성 함수입니다. 예를 들어, 런타임 업그레이드가 사용자 잔액을 나타내는 데이터 유형을 **부호 없는** 정수에서 **부호 있는** 정수로 변경하는 경우, 스토리지 마이그레이션은 기존 값을 부호 없는 정수로 읽은 다음 부호 있는 정수로 변환된 업데이트된 값을 다시 기록합니다. 이러한 유형의 데이터 저장 방식 변경을 필요로 하는 경우 이러한 변경을 수행하지 않으면 런타임은 스토리지 값을 올바르게 해석하여 런타임 상태에 포함시킬 수 없으며 정의되지 않은 동작을 유발할 가능성이 높습니다.

### FRAME을 사용한 스토리지 마이그레이션

FRAME 스토리지 마이그레이션은 [`OnRuntimeUpgrade`](https://paritytech.github.io/substrate/master/frame_support/traits/trait.OnRuntimeUpgrade.html) 트레이트를 사용하여 구현됩니다. `OnRuntimeUpgrade` 트레이트는 런타임 업그레이드 이후 즉시 실행되는 로직을 지정할 수 있는 `on_runtime_upgrade`라는 단일 함수를 정의합니다.

### 스토리지 마이그레이션 준비

스토리지 마이그레이션을 준비하는 것은 런타임 업그레이드에 의해 정의된 변경 사항을 이해하는 것을 의미합니다. Substrate 저장소는 이러한 변경 사항을 지정하기 위해 [`E1-runtimemigration`](https://github.com/paritytech/substrate/pulls?q=is%3Apr+label%3AE1-runtimemigration) 레이블을 사용합니다.

### 마이그레이션 작성

각 스토리지 마이그레이션은 요구 사항과 복잡성 수준에 따라 다릅니다. 그러나 스토리지 마이그레이션을 수행해야 할 때 다음 권장 사항을 사용하여 안내할 수 있습니다.

- 재사용 가능한 함수로 마이그레이션을 추출하고 해당 함수에 대한 테스트를 작성합니다.
- 디버깅을 돕기 위해 마이그레이션에 로깅을 포함시킵니다.
- 마이그레이션 코드는 업그레이드된 런타임의 컨텍스트 내에서 실행되므로 [이 예제](https://github.com/hicommonwealth/substrate/blob/5f3933f5735a75d2d438341ec6842f269b886aaa/frame/indices/src/migration.rs#L5-L22)와 같이 폐기된 유형을 포함해야 할 수 있습니다.
- [이 예제](https://github.com/paritytech/substrate/blob/c79b522a11bbc7b3cf2f4a9c0a6627797993cb79/frame/elections-phragmen/src/lib.rs#L119-L157)와 같이 스토리지 버전을 사용하여 마이그레이션을 더 선언적으로 만들어 안전하게 만들 수 있습니다.

### 마이그레이션 순서

기본적으로 FRAME은 `construct_runtime!` 매크로에서 팔렛이 나타나는 순서에 따라 `on_runtime_upgrade` 함수의 실행 순서를 정렬합니다. 업그레이드의 경우 함수는 _역순_으로 실행되며 가장 마지막 팔렛부터 시작합니다. 필요한 경우 사용자 정의 순서를 지정할 수도 있습니다. (예: [여기에서](https://github.com/hicommonwealth/edgeware-node/blob/7b66f4f0a9ec184fdebcccd41533acc728ebe9dc/node/runtime/src/lib.rs#L845-L866) 예제 참조).

FRAME 스토리지 마이그레이션은 다음 순서로 실행됩니다.

1. 사용자 정의 순서를 사용하는 경우 사용자 정의 `on_runtime_upgrade` 함수.
2. 시스템 `frame_system::on_runtime_upgrade` 함수.
3. `construct_runtime!` 매크로에서 마지막 팔렛부터 시작하여 정의된 모든 `on_runtime_upgrade` 함수.

### 마이그레이션 테스트

스토리지 마이그레이션을 테스트하는 것이 중요합니다. 스토리지 마이그레이션을 테스트하기 위해 사용할 수 있는 몇 가지 도구는 다음과 같습니다.

- [Substrate 디버그 키트](https://github.com/paritytech/substrate-debug-kit)에는 라이브 체인 데이터에서 안전하게 스토리지 마이그레이션 단위 테스트를 수행할 수 있는 [원격 externalities](https://github.com/paritytech/substrate-debug-kit/tree/master/remote-externalities) 도구가 포함되어 있습니다.
- [fork-off-substrate](https://github.com/maxsam4/fork-off-substrate) 스크립트를 사용하면 런타임 업그레이드와 스토리지 마이그레이션을 테스트하기 위해 로컬 테스트 체인을 부트스트랩하는 체인 스펙을 쉽게 생성할 수 있습니다.

다음으로 진행할 곳

- [실행 중인 네트워크 업그레이드](/tutorials/build-a-blockchain/upgrade-a-running-network/)
- [Substrate 마이그레이션](https://github.com/apopiak/substrate-migrations)