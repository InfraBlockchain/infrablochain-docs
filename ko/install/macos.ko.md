---
title: macOS 개발 환경
description: macOS에서 Substrate를 위한 로컬 개발 환경을 설정하세요.
keywords:
---

Rust를 설치하고 Apple macOS 컴퓨터(Intel 또는 Apple M1 프로세서)에서 Substrate 개발 환경을 설정할 수 있습니다.

## 시작하기 전에

macOS에서 Rust를 설치하고 개발 환경을 설정하기 전에, 컴퓨터가 다음의 기본 요구 사항을 충족하는지 확인하세요:

- 운영 체제 버전이 10.7 Lion 이상이어야 합니다.
- 프로세서 속도가 최소 2Ghz 이상이어야 하며, 3Ghz 이상을 권장합니다.
- 최소 8GB RAM, 16GB 이상을 권장합니다.
- 최소 10GB의 사용 가능한 저장 공간이 있어야 합니다.
- 고속 인터넷 연결이 필요합니다.

### Apple Silicon 지원

빌드 프로세스를 시작하기 전에 Protobuf를 설치해야 합니다. 다음 명령을 실행하여 설치하세요:

`brew install protobuf`

### Homebrew 설치

대부분의 경우, macOS 컴퓨터에서 패키지를 설치하고 관리하기 위해 Homebrew를 사용해야 합니다.
로컬 컴퓨터에 Homebrew가 이미 설치되어 있지 않은 경우, 계속하기 전에 Homebrew를 다운로드하고 설치해야 합니다.

Homebrew를 설치하려면 다음 단계를 따르세요:

1. Terminal 애플리케이션을 엽니다.

1. 다음 명령을 실행하여 Homebrew를 다운로드하고 설치하세요:

   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
   ```

1. 다음 명령을 실행하여 Homebrew가 성공적으로 설치되었는지 확인하세요:

   ```bash
   brew --version
   ```

   명령을 실행하면 다음과 유사한 출력이 표시됩니다:

   ```bash
   Homebrew 3.3.1
   Homebrew/homebrew-core (git revision c6c488fbc0f; last commit 2021-10-30)
   Homebrew/homebrew-cask (git revision 66bab33b26; last commit 2021-10-30)
   ```

## 설치

블록체인은 공개/개인 키 쌍의 생성 및 트랜잭션 서명의 유효성 검사를 지원하기 위해 표준 암호화를 필요로 합니다. 따라서 `openssl`과 같은 암호화를 제공하는 패키지가 필요합니다.

macOS에서 `openssl`과 Rust 툴체인을 설치하려면 다음 단계를 따르세요:

1. Terminal 애플리케이션을 엽니다.

1. 다음 명령을 실행하여 최신 버전의 Homebrew가 설치되었는지 확인하세요:

   ```bash
   brew update
   ```

1. 다음 명령을 실행하여 `openssl` 패키지를 설치하세요:

   ```bash
   brew install openssl
   ```

1. 다음 명령을 실행하여 `rustup` 설치 프로그램을 다운로드하고 Rust를 설치하세요:

   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```

1. 표시된 프롬프트에 따라 기본 설치를 진행하세요.

1. 다음 명령을 실행하여 현재 쉘에 Cargo를 포함시키세요:

   ```bash
   source ~/.cargo/env
   ```

1. 다음 명령을 실행하여 설치가 정상적으로 완료되었는지 확인하세요:

   ```bash
   rustc --version
   ```

1. 다음 명령을 실행하여 Rust 툴체인을 최신 안정 버전으로 설정하세요:

   ```bash
   rustup default stable
   rustup update
   rustup target add wasm32-unknown-unknown
   ```

1. 다음 명령을 실행하여 `nightly` 릴리스와 `nightly` WebAssembly (wasm) 타겟을 개발 환경에 추가하세요:

   ```bash
   rustup update nightly
   rustup target add wasm32-unknown-unknown --toolchain nightly
   ```

1. 다음 명령을 실행하여 개발 환경의 구성을 확인하세요:

   ```bash
   rustup show
   rustup +nightly show
   ```

   명령을 실행하면 다음과 유사한 출력이 표시됩니다:

   ```bash
   # rustup show

   active toolchain
   ----------------

   stable-x86_64-apple-darwin (default)
   rustc 1.61.0 (fe5b13d68 2022-05-18)

   # rustup +nightly show

   active toolchain
   ----------------

   nightly-x86_64-apple-darwin (overridden by +toolchain on the command line)
   rustc 1.63.0-nightly (e71440575 2022-06-02)
   ```

   1. 다음 명령을 사용하여 `cmake`을 설치하세요:

   ```
   brew install cmake
   ```

## Substrate 노드 컴파일

Rust가 설치되고 Substrate 개발을 위해 Rust 툴체인이 구성되었으므로, Substrate **노드 템플릿** 파일을 Clone하고 Substrate 노드를 컴파일하여 개발 환경을 설정할 준비가 되었습니다.

노드 템플릿은 불필요한 모듈이나 도구 없이 블록체인을 구축하는 데 필요한 가장 일반적인 기능을 모두 포함한 작업 환경을 제공합니다.
실험을 위한 상대적으로 안정적인 작업 환경을 제공하기 위해, 권장하는 모범 사례는 핵심 Substrate 레포지토리가 아닌 Substrate development hub 레포지토리에서 Substrate 노드 템플릿을 Clone하는 것입니다.

Substrate 노드 템플릿을 컴파일하려면 다음 단계를 따르세요:

1. 다음 명령을 실행하여 노드 템플릿 저장소를 복제하세요:

   ```bash
   git clone https://github.com/substrate-developer-hub/substrate-node-template
   ```

   대부분의 경우, 최신 코드를 얻기 위해 `main` 브랜치를 복제할 수 있습니다.
   그러나 특정 Polkadot 버전과 호환되는 Substrate 브랜치와 작업하려는 경우 `--branch` 명령줄 옵션을 사용할 수 있습니다.
   [Tags](https://github.com/substrate-developer-hub/substrate-node-template/tags)를 클릭하여 특정 Polkadot 버전과 호환되는 브랜치 목록을 확인할 수 있습니다.

1. 다음 명령을 실행하여 노드 템플릿 디렉토리의 루트로 이동하세요:

   ```bash
   cd substrate-node-template
   ```

   변경 사항을 저장하고 이 브랜치를 쉽게 식별할 수 있도록 하려면 다음과 유사한 명령을 실행하여 새 브랜치를 생성할 수 있습니다:

   ```bash
   git switch -c my-wip-branch
   ```

1. 다음 명령을 실행하여 노드 템플릿을 컴파일하세요:

   ```bash
   cargo build --release
   ```

   필요한 패키지의 수가 많기 때문에, 노드를 컴파일하는 데는 몇 분 정도 소요될 수 있습니다.

빌드가 성공적으로 완료되면 로컬 컴퓨터는 Substrate 개발 활동을 위해 준비된 상태입니다.

## 다음 단계로 넘어가기

Substrate 개발자 허브는 커뮤니티에게 제공되는 다양한 리소스에 대한 중앙 포털 역할을 합니다.
관심사와 학습 스타일에 따라 선호하는 방법이 다를 수 있습니다.
예를 들어, Rust에 익숙하고 소스 코드를 읽는 것을 선호하는 경우 [Rust API](https://paritytech.github.io/substrate/master)를 참조하는 것이 좋습니다.

<!-- TODO NAV.YAML -->
<!-- add these back -->
<!--Substrate 및 Substrate 생태계에 처음 접하는 경우, [탐색](/explore/)을 통해 사용 가능한 리소스와 찾을 수 있는 위치에 대한 더 넓은 정보를 얻을 수 있습니다.-->

더 자세히 알아볼 수 있는 몇 가지 추가적인 제안을 제공합니다.

#### 알려주세요

- [아키텍처](/learn/architecture/)
- [네트워크와 블록체인](/learn/node-and-network-types/)
- [빌드 프로세스](/build/build-process)

#### 안내해주세요

- [로컬 블록체인 구축](/tutorials/build-a-blockchain/build-local-blockchain/)
- [네트워크 시뮬레이션](/tutorials/build-a-blockchain/simulate-network/)
- [신뢰할 수 있는 노드 추가](/tutorials/build-a-blockchain/add-trusted-nodes/)