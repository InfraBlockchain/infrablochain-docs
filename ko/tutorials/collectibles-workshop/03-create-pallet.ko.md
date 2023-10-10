---
title: 새로운 팔레트 생성하기
tutorial: 1
---

이 워크샵에서는 애플리케이션 특정 블록체인을 위한 코드를 포함하는 사용자 정의 Substrate 모듈인 팔레트를 만드는 방법을 배우게 됩니다.
팔레트는 FRAME 라이브러리와 Rust 프로그래밍 언어를 사용하여 구축됩니다.
FRAME에는 응용 프로그램 로직을 재사용 가능한 컨테이너에 쉽게 구성할 수 있도록 하는 특수한 매크로가 많이 포함되어 있습니다.
Substrate 수집품 애플리케이션의 주요 부분에 대한 기본적인 뼈대를 준비하고 새 프로젝트를 생성하는 것으로 시작해 보겠습니다.

## 새 프로젝트 생성하기

이 워크샵은 새로운 모듈(사용자 정의 팔레트)을 생성하는 전체 워크플로우를 보여주기 위한 것이므로 `pallet-template`으로 시작하지 않습니다.
대신, 첫 번째 단계는 새로운 수집품 팔레트를 위한 새로운 Rust 패키지를 생성하는 것입니다.

프로젝트를 생성하려면 다음 단계를 따르세요:

1. 필요한 경우 새 터미널을 엽니다.

2. 작업 공간에서 `workshop-node-template/pallets` 디렉토리로 이동합니다.

3. 다음 명령을 실행하여 `collectibles` 팔레트를 위한 새로운 Rust 프로젝트를 생성합니다:

   ```bash
   cargo new collectibles
   ```

   `cargo`는 새 패키지가 현재 작업 공간에 없으며 이 문제를 해결하는 몇 가지 다른 방법이 있다는 경고를 표시합니다.
   일단은 이 경고를 무시하고 새 프로젝트를 준비하는 작업을 계속할 수 있습니다.

4. 다음 명령을 실행하여 `collectibles` 디렉토리로 이동합니다:

   ```bash
   cd collectibles
   ```

5. `collectibles` 디렉토리의 내용을 확인하고 기본 `Cargo.toml` 및 `src/main.rs` 프로그램을 확인합니다:

   ```text
   ├── Cargo.toml
   └── src
    └── main.rs
   ```

   Rust에서 각 패키지의 `Cargo.toml` 파일은 패키지가 필요로 하는 구성 설정 및 종속성을 정의하는 패키지 매니페스트라고 합니다.
   `workspace-node-template/pallets/collectibles` 폴더의 `Cargo.toml` 파일은 구축 중인 `collectibles` 패키지의 종속성을 정의합니다.

   관례적으로 Substrate의 Rust 프로젝트(팔레트 모듈 포함)의 소스 코드는 일반적으로 `src/lib.rs` 파일에 있습니다.
   Cargo는 새 프로젝트에 대해 기본적으로 `src/main.rs` 파일을 생성합니다.
   워크샵에서는 새 모듈의 주요 소스 파일 이름을 분명하게 하기 위해 main 소스 파일의 이름을 변경해 보겠습니다.

6. 다음 명령을 실행하여 `src/main.rs` 소스 파일의 이름을 변경합니다:

   ```bash
   mv src/main.rs src/lib.rs
   ```

   이제 프로젝트 파일이 준비되었으므로 `cargo`가 경고한 문제를 해결하기 위해 새 프로젝트를 현재 작업 공간에 추가해 보겠습니다.

## 작업 공간에 팔레트 추가하기

기타 Rust 프로그램과 마찬가지로 노드 템플릿에는 `Cargo.toml` 매니페스트 파일이 있습니다.
이 경우 매니페스트 파일은 작업 공간 멤버를 설명합니다.
새 팔레트를 작업 공간에 추가하려면 다음 단계를 따르세요:

1. 작업 공간에서 `workshop-node-template` 디렉토리로 이동합니다.

2. 코드 편집기에서 `Cargo.toml` 파일을 엽니다.

3. 새로운 `pallets/collectibles` 팔레트를 작업 공간의 멤버로 추가합니다.

   ```toml
   [workspace]
   members = [
     "node",
     "pallets/collectibles",
     "pallets/template",
     "runtime",
   ]
   ```

4. 변경 사항을 저장하고 파일을 닫습니다.

## 프로젝트 매니페스트 준비하기

새 프로젝트가 작업 공간에 포함되어 설정을 시작할 준비가 되었습니다.
이미 프로젝트에 기본 `Cargo.toml` 매니페스트 파일이 있고 이 파일이 패키지 속성과 종속성을 설명한다는 것을 이미 알고 있습니다.
Substrate와 FRAME을 사용하여 블록체인을 개발할 때 패키지 속성과 종속성을 정의하는 것은 특히 중요합니다.
모듈러 개발 환경을 사용하면 코드의 한 부분에서 다른 부분으로 기능을 가져올 수 있으며 이를 통해 공통 기능을 재사용할 수 있습니다.

이 워크샵에서 `collectibles` 모듈은 Substrate 런타임의 일부가 되며, `Cargo.toml` 파일은 이 모듈이 의존하는 일부 모듈을 정의해야 합니다.
예를 들어, `collectibles` 모듈이 필요로 하는 두 가지 핵심 패키지는 `frame_system`과 `frame_support` 모듈입니다:

- `frame_system`은 일반적인 데이터 구조와 원시(raw) 자료형을 다루기 위한 핵심 기능을 제공하여 이를 필요로 하는 모든 팔레트에서 사용할 수 있도록 하여 새로운 팔레트를 런타임에 쉽게 통합하고 서로 상호 작용할 수 있도록 합니다.
- `frame_support`은 런타임으로 디스패치되는 함수 호출을 처리하기 위한 핵심 지원 서비스를 제공하며, 저장소 구조를 정의하고 이벤트와 오류를 준비하고 핵심 유틸리티를 정의합니다.

`frame_system`과 `frame_support` 외에도 `collectibles` 모듈은 블록체인의 네트워크 트래픽을 최소화하기 위해 필요한 유형 인코딩 및 디코딩을 지원하는 패키지가 필요합니다.
SCALE 형식의 인코딩 및 디코딩을 지원하기 위해 `collectibles` 모듈은 `codec` 및 `scale-info` 패키지에 액세스해야 합니다.

`collectibles` 프로젝트의 매니페스트를 업데이트하려면 다음 단계를 따르세요:

1. 코드 편집기에서 `collectibles` 모듈의 기본 `Cargo.toml` 파일을 엽니다.

2. `frame-support`와 `frame-system`을 종속성에 추가합니다.

   ```toml
   [dependencies]
   frame-support = { default-features = false, version = "4.0.0-dev", git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-v1.0.0"}
   frame-system = { default-features = false, version = "4.0.0-dev", git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-v1.0.0" }
   ```

3. `codec`와 `scale-info`를 종속성에 추가합니다.

   ```toml
   codec = { package = "parity-scale-codec", version = "3.0.0", default-features = false, features = ["derive",] }
   scale-info = { version = "2.1.1", default-features = false, features = ["derive"] }
   ```

4. 네이티브 바이너리 타겟을 지원하기 위해 `Cargo.toml` 파일에 이 패키지들의 표준 기능을 추가합니다.

   ```toml
   [features]
   default = ["std"]
   std = [
      "frame-support/std",
      "frame-system/std",
      "codec/std",
      "scale-info/std",
   ]
   ```

   현재 Substrate 런타임은 크로스 플랫폼 WebAssembly 타겟과 플랫폼별 네이티브 바이너리 타겟으로 컴파일됩니다.

## 공통 코드 섹션 준비하기

이제 팔레트에 필요한 최소한의 패키지 종속성이 `Cargo.toml` 매니페스트에 지정되었습니다.
다음 단계는 새로운 팔레트의 뼈대로 사용할 일련의 공통 매크로를 준비하는 것입니다.

1. 텍스트 편집기에서 `src/lib.rs`를 열고 템플릿 `main()` 함수를 삭제합니다.

   이제 Substrate 수집품 팔레트를 생성하기 위한 깨끗한 상태입니다.

2. 다음과 같은 일련의 공통 매크로 선언을 `src/lib.rs` 파일에 추가하여 Substrate 수집품 팔레트의 뼈대를 준비합니다.

   ```rust
   #![cfg_attr(not(feature = "std"), no_std)]
   
   pub use pallet::*;
   
   #[frame_support::pallet(dev_mode)]
   pub mod pallet {
        use frame_support::pallet_prelude::*;
        use frame_system::pallet_prelude::*;

        #[pallet::pallet]
        #[pallet::generate_store(pub(super) trait Store)]
        pub struct Pallet<T>(_);

        #[pallet::config]
        pub trait Config: frame_system::Config {
        }
   }
   ```

3. 변경 사항을 저장하고 파일을 닫습니다.

1. 다음 명령을 실행하여 프로그램이 컴파일되는지 확인합니다.

   ```bash
   cargo build --package collectibles
   ```

   현재는 사용되지 않는 코드에 대한 컴파일러 경고를 무시할 수 있습니다.