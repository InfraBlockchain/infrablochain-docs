---
title: 코드 탐색하기
description: 노드 템플릿의 내용을 자세히 살펴보세요.
keywords:
---

[노드 시작하기](/quick-start/start-a-node/)에서 개발 모드에서 로컬 Substrate 노드를 컴파일하고 시작했습니다.
이 특정 노드인 `substrate-node-template`은 시작하기 위해 몇 가지 공통 모듈만 간단히 제공하는 간소화된 환경을 제공합니다.
세부 사항에 너무 깊게 들어가지 않고도, 노드 템플릿 코드의 기본 구성 요소를 탐색함으로써 많은 것을 배울 수 있습니다.

## 노드 템플릿에 대해

노드 템플릿에는 P2P 네트워킹, 간단한 합의 메커니즘, 트랜잭션 처리와 같은 기본적인 블록체인 요소가 포함되어 있습니다.
노드 템플릿에는 또한 계정, 잔액, 트랜잭션 수수료 작업을 다루고 관리자 작업을 수행하는 데 필요한 몇 가지 기본 기능도 포함되어 있습니다.
이러한 핵심 기능 세트는 특정 기능을 구현하는 여러 사전 정의 모듈인 **팔레트**(pallets)를 통해 제공됩니다.

예를 들어, 다음과 같은 핵심 모듈이 노드 템플릿에 사전 정의되어 있습니다:

- `pallet_balances`: 계정 자산 및 계정 간 이체를 관리합니다.
- `pallet_transaction_payment`: 수행된 트랜잭션의 수수료를 관리합니다.
- `pallet_sudo`: 관리자 권한이 필요한 작업을 수행합니다.

노드 템플릿은 또한 사용자 정의 팔레트를 구현하는 방법을 보여주는 시작용 `pallet_template`도 제공합니다.

이제 노드 템플릿에 포함된 기능을 개요로 알게 되었으니, `substrate-node-template` 디렉토리와 하위 디렉토리의 코드를 자세히 살펴보겠습니다.

## 매니페스트 파일

Substrate는 Rust 기반 프레임워크이기 때문에 각 패키지에는 패키지를 컴파일하는 데 필요한 정보를 담고 있는 매니페스트 파일인 `Cargo.toml` 파일이 있습니다.
`substrate-node-template`의 루트 디렉토리에 위치한 `Cargo.toml` 파일을 열면, 노드 템플릿 워크스페이스를 구성하는 멤버 패키지를 설명하는 내용을 볼 수 있습니다.
예를 들면:

```toml
[workspace]
members = [
    "node",
    "pallets/template",
    "runtime",
]
[profile.release]
panic = "unwind"
```

이 매니페스트에서는 노드 템플릿 워크스페이스에 세 개의 패키지가 포함되어 있는 것을 알 수 있습니다:

- `node` 패키지는 P2P 네트워킹, 블록 생성, 블록 최종화, 트랜잭션 풀 관리 등 많은 핵심 블록체인 서비스를 위한 Rust 모듈을 제공합니다.
- `pallets` 하위 디렉토리의 `template` 패키지는 사용자 정의 모듈을 구축할 때 기능을 구현하는 방법을 보여주는 시작용 템플릿입니다.
- `runtime` 패키지는 노드 템플릿에 포함된 계정, 잔액, 트랜잭션 수수료 및 기타 기능을 처리하는 모든 응용 로직을 제공합니다.

각 멤버 패키지는 자체의 매니페스트인 `Cargo.toml` 파일을 가지고 있으며, 해당 멤버 패키지를 컴파일하는 데 필요한 종속성 및 구성 설정과 같은 패키지별 정보를 포함하고 있습니다.
예를 들어, 워크스페이스의 `node` 멤버를 위한 `Cargo.toml` 파일은 패키지의 이름이 `node-template`이며, 노드 템플릿이 필수적인 블록체인 서비스를 제공할 수 있도록 하는 핵심 라이브러리와 기본 요소를 나열하고 있습니다.
라이브러리와 기본 요소에 대해서는 [아키텍처 및 Rust 라이브러리](/learn/architecture)에서 자세히 알아보게 될 것입니다.
지금은 각 패키지에 대한 종속성 및 기타 중요한 정보를 설명하는 매니페스트의 중요성을 이해하는 것으로 충분합니다.

`runtime/Cargo.toml` 파일과 `pallets/template/Cargo.toml` 파일을 열면 다른 라이브러리와 기본 요소가 종속성으로 나열되어 있지만, 이러한 패키지를 컴파일하는 데 필요한 내용을 대략적으로 파악할 수 있습니다.
예를 들어, 런타임에 대한 매니페스트는 `frame_system`, `frame_support` 및 이전에 언급한 `pallet_balances`, `pallet_transaction_payment`, `pallet_sudo` 모듈을 포함하는 기본 런타임을 나타냅니다.

## 핵심 클라이언트 소스 코드

Substrate의 가장 중요한 측면 중 하나는 노드가 **코어 클라이언트**(core client)와 **런타임**(runtime) 두 가지 주요 부분으로 구성된다는 것입니다.
노드 템플릿은 또한 `node/src` 디렉토리에 있는 코어 클라이언트 서비스를 위한 별도의 패키지와 `runtime/src` 디렉토리에 있는 런타임을 위한 별도의 패키지로 구성되어 있습니다.

기본적으로 `node/src` 디렉토리에는 다음과 같은 Rust 모듈이 포함되어 있습니다:

- `benchmarking.rs`
- `chain_spec.rs`
- `cli.rs`
- `command.rs`
- `lib.rs`
- `main.rs`
- `rpc.rs`
- `service.rs`

대부분의 코어 클라이언트 서비스는 `node/src/service.rs` Rust 모듈에 캡슐화되어 있습니다.
이 파일이나 `node/src` 디렉토리의 다른 Rust 모듈을 수정해야 할 일은 드물지만, 수정할 가능성이 있는 파일은 `chain_spec.rs` 파일입니다.
`chain_spec.rs` 파일은 개발 및 로컬 테스트넷 체인의 기본 구성, 기본 사전 자금이 제공된 개발 계정 및 블록 생성 권한이 사전 구성된 노드에 대한 정보를 포함합니다.
사용자 정의 체인을 생성하는 경우, 이 파일을 사용하여 노드가 연결되는 네트워크와 로컬 노드가 통신하는 다른 노드를 식별합니다.

## 기본 노드 템플릿 런타임

Substrate는 모듈식이고 유연한 블록체인 구축 프레임워크를 제공하기 때문에 워크스페이스의 모든 패키지를 변경할 수 있습니다.
그러나 대부분의 응용 프로그램 개발 작업은 런타임과 런타임을 구성하는 모듈인 팔레트에서 수행됩니다.
자신의 프로젝트에 맞게 런타임을 사용자 정의하기 전에, 기본 노드 템플릿에 있는 내용을 약간 살펴보는 것이 좋습니다.

### 기본 매니페스트

이미 다음과 같이 런타임의 기본 종속성과 기능을 나열하는 런타임의 기본 매니페스트를 보았습니다:

```rust
pallet-balances = { version = "4.0.0-dev", default-features = false, git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-vX.Y.Z" }

pallet-sudo = { version = "4.0.0-dev", default-features = false, git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-vX.Y.Z" }

pallet-transaction-payment = { version = "4.0.0-dev", default-features = false, git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-vX.Y.Z" }
```

또한 `frame-system`, `frame-support`, `frame-executive`와 같은 핵심 패키지에 대한 종속성도 있습니다.
이러한 핵심 서비스에 대해서는 [핵심 FRAME 서비스](/learn/runtime-development/#core-frame-services)에서 자세히 알아보게 될 것입니다.
지금은 이러한 모듈과 기타 모듈이 노드 템플릿의 런타임을 컴파일하는 데 필요하다는 점에 주목해주세요.

### 기본 소스 코드

런타임의 주요 소스 코드는 `runtime/src/lib.rs` 파일에 있습니다.
이 파일을 코드 편집기에서 열면 처음에는 복잡해 보일 수 있습니다.
이 파일에 대한 세부 사항은 문서의 다른 부분에서 다루는 뉘앙스가 있지만, 본질적으로 소스 코드는 다음과 같은 작업을 수행합니다:

- `frame_system` 및 `frame_support` 핵심 서비스를 가져옵니다.
- 런타임의 버전 정보를 지정합니다.
- 포함할 팔레트를 선언합니다.
- 포함된 각 팔레트의 유형과 매개변수를 선언합니다.
- 포함된 각 팔레트에 대한 상수 및 변수 값을 설정합니다.
- 포함된 각 팔레트에 대한 `Config` 트레이트를 구현합니다.
- 포함된 팔레트로부터 런타임을 구성합니다.
- 팔레트 성능을 평가하기 위한 벤치마킹 프레임워크를 준비합니다.
- 코어 클라이언트가 런타임에 호출할 수 있도록 인터페이스를 구현합니다.

런타임을 구성하는 방법, 벤치마킹 정의, 런타임 인터페이스 사용에 대해서는 [빌드](/build/) 및 [테스트](/test/) 섹션의 주제에서 자세히 알아보게 될 것입니다.
지금은 런타임이 어떻게 구성되고 기본 팔레트가 `Config` 트레이트를 사용하여 구현되는지에 대한 일반적인 감각만 가지고 있으면 됩니다.