---
title: 빌드 프로세스
description: Substrate 노드가 플랫폼 네이티브 및 WebAssembly 이진 파일로 컴파일되고, 이진 파일이 Substrate 런타임으로의 호출을 실행하는 방법을 설명합니다.
keywords:
---

[아키텍처](/learn/architecture)에서 Substrate 노드가 외부 노드 호스트와 런타임 실행 환경으로 구성되는 것을 배웠습니다.
이러한 노드 구성 요소는 런타임 API 호출과 호스트 함수 호출을 통해 서로 통신합니다.
이 섹션에서는 Substrate 런타임이 플랫폼 네이티브 실행 파일과 블록체인에 저장되는 WebAssembly (Wasm) 이진 파일로 컴파일되는 방법에 대해 자세히 알아보겠습니다.
이진 파일이 컴파일되는 내부 작업을 살펴본 후, 언제 두 개의 이진 파일이 사용되며, 필요한 경우 실행 전략을 변경하는 방법에 대해 자세히 알아보겠습니다.

## 최적화된 아티팩트 컴파일

Substrate 노드를 컴파일하려면 Substrate 노드 프로젝트의 루트 디렉토리에서 `cargo build --release` 명령을 실행하는 것을 이미 알고 있을 것입니다.
이 명령은 프로젝트를 위한 플랫폼별 실행 파일과 WebAssembly 이진 파일을 모두 빌드하고 **최적화된** 실행 파일 아티팩트를 생성합니다.
최적화된 실행 파일 아티팩트 생성에는 몇 가지 후처리 작업이 포함됩니다.

최적화 과정 중에는 WebAssembly 런타임 이진 파일이 체인의 제네시스 상태에 포함되기 전에 일련의 내부 단계를 거쳐 컴파일되고 압축됩니다.
이 프로세스를 더 잘 이해하기 위해 다음 다이어그램은 이 단계를 요약합니다.

![WebAssembly 컴파일 및 압축 후 체인에 포함](/media/images/docs/node-executable.png)

다음 섹션에서는 빌드 프로세스에 대해 자세히 설명합니다.

### WebAssembly 이진 파일 빌드

`wasm-builder`는 프로젝트의 주요 `cargo` 빌드 프로세스에 WebAssembly 이진 파일 빌드 프로세스를 통합하는 도구입니다.
이 도구는 `substrate-wasm-builder` 크레이트로 게시됩니다.

빌드 프로세스를 시작하면 `cargo`가 프로젝트의 모든 `Cargo.toml`에서 종속성 그래프를 빌드합니다.
그런 다음 런타임 `build.rs` 모듈은 `substrate-wasm-builder` 크레이트를 사용하여 런타임의 Rust 코드를 WebAssembly 이진 파일로 컴파일하여 초기 이진 파일 아티팩트를 생성합니다.

#### WebAssembly에 포함된 기능

기본적으로 `wasm-builder`는 WebAssembly 이진 파일과 플랫폼 네이티브 실행 파일 모두에 프로젝트에서 정의한 모든 기능을 활성화합니다. 다만 `default` 및 `std` 기능은 네이티브 빌드에만 활성화됩니다.

#### 빌드 프로세스를 사용자 정의하기 위한 환경 변수

다음 환경 변수를 사용하여 WebAssembly 이진 파일 빌드 방식을 사용자 정의할 수 있습니다:

| 이 변수를 사용하려면     | 이렇게 하려면                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `SKIP_WASM_BUILD`       | WebAssembly 이진 파일 빌드를 건너뛰기 위해 사용합니다. 이는 네이티브 이진 파일만 다시 컴파일해야 할 때 유용합니다. 그러나 WebAssembly 이진 파일이 존재하지 않으면 이진 파일이 컴파일되지 않습니다. 개별 프로젝트에서 WebAssembly 빌드를 건너뛰려면 환경 변수에 PROJECT_NAME을 포함시키면 됩니다. 예를 들어, cargo 프로젝트 node-runtime의 WebAssembly 이진 파일 빌드를 건너뛰려면 환경 변수 SKIP_NODE_RUNTIME_WASM_BUILD를 사용할 수 있습니다. |
| `WASM_BUILD_TYPE`       | WebAssembly 이진 파일이 `릴리스` 빌드인지 `디버그` 빌드인지 지정합니다. 기본적으로 `cargo` 명령에 지정한 빌드 유형이 사용됩니다.                                                                                                                                                                                                                                                                                                         |
| `FORCE_WASM_BUILD`      | WebAssembly 빌드를 강제로 실행합니다. 이 환경 변수는 거의 사용되지 않습니다. 왜냐하면 `wasm-builder`가 파일 변경을 확인하도록 `cargo`에 지시하기 때문입니다.                                                                                                                                                                                                                                                                                                                 |
| `WASM_BUILD_RUSTFLAGS`  | WebAssembly 이진 파일 빌드 중 `cargo build` 명령에 전달되는 `RUSTFLAGS`를 확장합니다.                                                                                                                                                                                                                                                                                                                                                               |
| `WASM_BUILD_NO_COLOR`   | WebAssembly 빌드의 색상 출력을 비활성화합니다.                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `WASM_TARGET_DIRECTORY` | WebAssembly 이진 파일을 지정된 디렉토리로 복사합니다. 경로는 절대 경로여야 합니다.                                                                                                                                                                                                                                                                                                                                                                          |
| `WASM_BUILD_TOOLCHAIN`  | WebAssembly 이진 파일을 빌드하는 데 사용할 도구 체인을 지정합니다. 형식은 `cargo`에서 사용하는 형식과 동일해야 합니다. 예를 들어, `nightly-2020-02-20`과 같습니다.                                                                                                                                                                                                                                                                                                          |
| `CARGO_NET_OFFLINE`     | 오프라인 환경을 지원하기 위해 일부 또는 모든 프로세스의 네트워크 액세스를 방지합니다.                                                                                                                                                                                                                                                                                                                                                                      |

#### WebAssembly 이진 파일 압축 및 압축 해제

`substrate-wasm-builder` 크레이트는 낮은 수준의 프로세스를 사용하여 명령어 시퀀스를 최적화하고 디버깅에 사용되는 코드와 같은 불필요한 코드를 제거하여 압축된 WebAssembly 이진 파일을 생성합니다.
그런 다음 바이너리는 최종 WebAssembly 이진 파일의 크기를 최소화하기 위해 추가로 압축됩니다.
컴파일러는 노드의 `runtime/src/lib.rs` 파일을 처리할 때 생성된 WebAssembly 이진 파일을 포함해야 함을 인식합니다:

```rust
include!(concat!(env!("OUT_DIR"), "/wasm_binary.rs"));
```

이 코드는 압축된 WebAssembly 이진 파일(`WASM_BINARY`)과 컴파일러에 의해 생성된 압축되지 않은 WebAssembly 이진 파일(`WASM_BINARY_BLOATY`)을 컴파일 결과에 포함시키고, 프로젝트의 최종 실행 파일이 생성됩니다.

빌드 프로세스의 각 단계에서 WebAssembly 이진 파일은 이전보다 작은 크기로 압축됩니다.
예를 들어, Polkadot의 각 WebAssembly 이진 파일 아티팩트의 크기를 비교할 수 있습니다:

```bash
.rw-r--r-- 1.2M pep  1 Dec 16:13 │  ├── polkadot_runtime.compact.compressed.wasm
.rw-r--r-- 5.1M pep  1 Dec 16:13 │  ├── polkadot_runtime.compact.wasm
.rwxr-xr-x 5.5M pep  1 Dec 16:13 │  └── polkadot_runtime.wasm
```

체인 업그레이드 및 릴레이 체인 유효성 검사를 위해 항상 완전히 압축된 런타임(`*_runtime.compact.compressed.wasm`) WebAssembly 이진 파일을 사용해야 합니다.
대부분의 경우, 초기 WebAssembly 이진 파일이나 중간 압축 아티팩트를 사용할 필요는 없습니다.

## 실행 전략

네이티브 및 WebAssembly 런타임으로 노드를 컴파일한 후, 명령줄 옵션을 사용하여 노드의 동작 방식을 지정합니다.
노드를 시작하는 데 사용할 수 있는 명령줄 옵션에 대한 자세한 내용은 [node-template](/reference/command-line-tools/node-template) 명령줄 참조를 참조하십시오.

노드를 시작하면 노드 실행 파일은 지정한 명령줄 옵션을 사용하여 체인을 초기화하고 제네시스 블록을 생성합니다.
이 프로세스의 일부로서, 노드는 WebAssembly 런타임을 스토리지 항목 값 및 해당 `:code` 키로 추가합니다.

노드를 시작한 후, 실행 중인 노드는 사용할 런타임을 선택합니다.
기본적으로 노드는 동기화, 새 블록 작성, 블록 가져오기, 오프체인 워커와의 상호작용을 포함한 모든 작업에 WebAssembly 런타임을 항상 사용합니다.

### WebAssembly 런타임 선택

WebAssembly 런타임을 사용하는 것은 중요합니다. 왜냐하면 WebAssembly와 네이티브 런타임은 서로 다를 수 있기 때문입니다.
예를 들어, 런타임에 변경 사항을 가하려면 새로운 WebAssembly blob을 생성하고 WebAssembly 런타임의 새 버전을 사용하도록 체인을 업데이트해야 합니다.
업데이트 후, WebAssembly 런타임은 네이티브 런타임과 다릅니다.
이 차이를 반영하기 위해 모든 실행 전략은 WebAssembly 런타임의 표현을 규범적인 런타임으로 취급합니다.
네이티브 런타임과 WebAssembly 런타임 버전이 다른 경우, 항상 WebAssembly 런타임이 선택됩니다.

WebAssembly 런타임은 블록체인 상태의 일부로 저장되기 때문에 네트워크는 이 바이너리의 표현에 대해 합의해야 합니다.
바이너리에 대한 합의를 도출하기 위해 WebAssembly 런타임을 나타내는 blob은 모든 동기화 노드에서 정확히 동일해야 합니다.

### WebAssembly 실행 환경

WebAssembly 실행 환경은 Rust 실행 환경보다 제한적일 수 있습니다.
예를 들어, WebAssembly 실행 환경은 최대 4GB의 메모리를 가진 32비트 아키텍처입니다.
WebAssembly 런타임에서 실행할 수 있는 로직은 항상 Rust 런타임에서 실행할 수 있습니다.
하지만 Rust 런타임에서 실행할 수 있는 로직을 모두 WebAssembly 런타임에서 실행할 수 있는 것은 아닙니다.
블록 작성 노드는 일반적으로 유효한 블록을 생성하기 위해 WebAssembly 실행 환경을 사용합니다.

### 네이티브 런타임

WebAssembly 런타임이 기본적으로 선택되지만, 모든 또는 특정 작업에 대해 실행되는 런타임을 재정의할 수 있습니다. 이를 위해 **실행 전략**을 명령줄 옵션으로 지정할 수 있습니다.

네이티브 런타임과 WebAssembly 런타임이 동일한 [버전](/maintain/runtime-upgrades/#runtime-versioning)을 공유하는 경우, WebAssembly 런타임 대신 네이티브 런타임을 선택적으로 사용하거나 WebAssembly 런타임과 함께 사용하거나, WebAssembly 런타임 사용이 실패한 경우 대체로 사용할 수 있습니다.
일반적으로, 성능상의 이유나 WebAssembly 런타임보다 제한적인 환경 때문에 네이티브 런타임을 사용하려고 할 때만 네이티브 런타임을 선택하게 됩니다.
예를 들어, 초기 동기화를 위해 네이티브 런타임을 사용하려면 `--execution-syncing native` 또는 `--execution-syncing native-else-wasm` 명령줄 옵션을 사용하여 노드를 시작할 수 있습니다.

모든 또는 특정 작업에 대해 실행 전략을 지정하기 위해 명령줄 옵션을 사용하는 방법에 대한 정보는 [node-template](/reference/command-line-tools/node-template)을 참조하십시오.
실행 전략 변형에 대한 정보는 [ExecutionStrategy](https://paritytech.github.io/substrate/master/sc_cli/arg_enums/enum.ExecutionStrategy.html)을 참조하십시오.

## 네이티브 런타임 없이 WebAssembly 빌드하기

새로운 체인을 시작하려면 WebAssembly 런타임이 필요합니다.
초기 WebAssembly 런타임이 제공된 후, WebAssembly 런타임을 나타내는 blob은 [체인 사양](/build/chain-spec)의 일부로 다른 노드에 전달될 수 있습니다.
일부 특수한 경우에는 네이티브 런타임 없이 WebAssembly 대상을 컴파일하려고 할 수 있습니다.
예를 들어, 포크리스 업그레이드를 준비하기 위해 WebAssembly 런타임을 테스트하는 경우 새로운 WebAssembly 이진 파일만 컴파일하려고 할 수 있습니다.

이는 드물게 사용되는 사용 사례이지만, [build-only-wasm.sh](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/.maintain/build-only-wasm.sh) 스크립트를 사용하여 `no_std` WebAssembly 이진 파일을 네이티브 런타임을 컴파일하지 않고 빌드할 수 있습니다.

또한 `wasm-runtime-overrides` 명령줄 옵션을 사용하여 파일 시스템에서 WebAssembly를 로드할 수 있습니다.

## WebAssembly 없이 Rust 컴파일하기

새로운 WebAssembly 런타임을 빌드하지 않고 노드의 Rust 코드를 컴파일하려면 `SKIP_WASM_BUILD`를 빌드 옵션으로 사용할 수 있습니다.
이 옵션은 WebAssembly를 업데이트할 필요가 없는 경우에 컴파일 시간을 빠르게 하기 위해 주로 사용됩니다.

## 다음 단계

- [Wasm-builder README](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/utils/wasm-builder/README.md)
- [Rust 컴파일 옵션](https://doc.rust-lang.org/cargo/commands/cargo-build.html#compilation-options)
- [토론: 네이티브 런타임 제거](https://github.com/paritytech/substrate/issues/10579)