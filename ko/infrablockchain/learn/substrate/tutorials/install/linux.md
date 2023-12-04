---
title: 리눅스 개발 환경
description: 리눅스에서 Substrate를 위한 로컬 개발 환경을 설정하세요.
keywords:
---

Rust는 대부분의 리눅스 배포판을 지원합니다.
사용하는 특정 배포판과 운영 체제의 버전에 따라 환경에 일부 소프트웨어 종속성을 추가해야 할 수도 있습니다.
일반적으로 개발 환경에는 `clang`와 같은 링커 또는 C 호환 컴파일러와 적절한 통합 개발 환경(IDE)이 포함되어야 합니다.

## 시작하기 전에

사용하는 운영 체제의 문서를 확인하여 설치된 패키지 및 필요한 추가 패키지를 다운로드하고 설치하는 방법에 대한 정보를 확인하세요.
예를 들어, Ubuntu를 사용하는 경우 Ubuntu 고급 패키징 도구(`apt`)를 사용하여 `build-essential` 패키지를 설치할 수 있습니다:

```bash
sudo apt install build-essential
```

Rust를 설치하기 전에 다음 패키지가 필요합니다:

```text
clang curl git make
```

블록체인은 공개/개인 키 쌍 생성 및 트랜잭션 서명 유효성 검사를 지원하기 위해 표준 암호화를 필요로 하므로, `libssl-dev` 또는 `openssl-devel`과 같은 암호화를 제공하는 패키지도 설치해야 합니다.

## 필수 패키지와 Rust 설치하기

리눅스에서 Rust 툴체인을 설치하려면 다음 단계를 따르세요:

1. 컴퓨터에 로그인하고 터미널 쉘을 엽니다.

1. 사용하는 리눅스 배포판에 적합한 패키지 관리 명령을 실행하여 로컬 컴퓨터에 설치된 패키지를 확인하세요.

1. 사용하는 리눅스 배포판에 적합한 패키지 관리 명령을 실행하여 로컬 개발 환경에 누락된 패키지 종속성을 추가하세요.

   예를 들어, Ubuntu Desktop 또는 Ubuntu Server에서는 다음과 유사한 명령을 실행할 수 있습니다:

   ```bash
   sudo apt install --assume-yes git clang curl libssl-dev protobuf-compiler
   ```

   다른 리눅스 운영 체제에 대한 예제를 보려면 탭 제목을 클릭하세요:

   <figure class='tabbed'>

   [[tabbedCode]]
   |```Debian
   | sudo apt install --assume-yes git clang curl libssl-dev llvm libudev-dev make protobuf-compiler

   [[tabbedCode]]
   |```Arch
   | pacman -Syu --needed --noconfirm curl git clang make protobuf

   [[tabbedCode]]
   | ```fedora
   | sudo dnf update
   | sudo dnf install clang curl git openssl-devel make protobuf-compiler

   [[tabbedCode]]
   | ```opensuse
   | sudo zypper install clang curl git openssl-devel llvm-devel libudev-devel make protobuf

   </figure>

   다른 배포판은 다른 패키지 관리자를 사용하고 패키지를 다른 방식으로 번들로 제공할 수 있습니다.
   예를 들어, 설치 선택에 따라 Ubuntu Desktop과 Ubuntu Server는 다른 패키지와 요구 사항을 가질 수 있습니다.
   그러나 명령줄 예제에 나열된 패키지는 Debian, Linux Mint, MX Linux, Elementary OS를 포함한 많은 일반적인 리눅스 배포판에 적용될 수 있습니다.

1. `rustup` 설치 프로그램을 다운로드하고 다음 명령을 실행하여 Rust를 설치하세요:

   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```

1. 표시된 프롬프트에 따라 기본 설치를 진행하세요.

1. 다음 명령을 실행하여 현재 쉘에 Cargo를 포함시킵니다:

   ```bash
   source $HOME/.cargo/env
   ```

1. 다음 명령을 실행하여 설치를 확인하세요:

   ```bash
   rustc --version
   ```

1. 다음 명령을 실행하여 Rust 툴체인을 최신 안정 버전으로 설정하세요:

   ```bash
   rustup default stable
   rustup update
   ```

1. 다음 명령을 실행하여 `nightly` 릴리스와 `nightly` WebAssembly (wasm) 대상을 개발 환경에 추가하세요:

   ```bash
   rustup update nightly
   rustup target add wasm32-unknown-unknown --toolchain nightly
   ```

1. 다음 명령을 실행하여 개발 환경의 구성을 확인하세요:

   ```bash
   rustup show
   rustup +nightly show
   ```

   명령은 다음과 유사한 출력을 표시합니다:

   ```bash
   # rustup show

   active toolchain
   ----------------

   stable-x86_64-unknown-linux-gnu (default)
   rustc 1.62.1 (e092d0b6b 2022-07-16)

   # rustup +nightly show

   active toolchain
   ----------------

   nightly-x86_64-unknown-linux-gnu (overridden by +toolchain on the command line)
   rustc 1.65.0-nightly (34a6cae28 2022-08-09)
   ```

## Substrate 노드 컴파일하기

이제 Rust가 설치되고 Substrate 개발을 위해 Rust 툴체인이 구성되었으므로, Substrate **노드 템플릿** 을 Clone하고  Substrate 노드를 컴파일하여 개발 환경을 완성할 준비가 되었습니다.

노드 템플릿은 불필요한 모듈이나 도구 없이 블록체인을 구축하는 데 필요한 가장 일반적인 기능을 모두 포함한 작업 환경을 제공합니다.
실험에 안정적인 상대적으로 안정된 작업 환경을 제공하기 위해, 코어 Substrate 레포지토리 대신 Substrate development hub 레포지토리에서 Substrate 노드 템플릿을 Clone하는 것이 권장되는 모범 사례입니다.

Substrate 노드 템플릿을 컴파일하려면 다음 단계를 따르세요:

1. 다음 명령을 실행하여 노드 템플릿 저장소를 복제하세요:

   ```bash
   git clone https://github.com/substrate-developer-hub/substrate-node-template
   ```

   대부분의 경우 최신 코드를 얻기 위해 `main` 브랜치를 복제할 수 있습니다.
   그러나 특정 Polkadot 버전과 호환되는 Substrate 브랜치와 작업하려는 경우 `--branch` 명령줄 옵션을 사용할 수 있습니다.
   특정 Polkadot 버전과 호환되는 브랜치 목록을 보려면 [Tags](https://github.com/substrate-developer-hub/substrate-node-template/tags)를 클릭하세요.

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

   필요한 패키지의 수가 많기 때문에, 노드를 컴파일하는 데는 몇 분 정도 걸릴 수 있습니다.

빌드가 성공적으로 완료되면 로컬 컴퓨터는 Substrate 개발 활동을 위해 준비된 상태입니다.

## 다음 단계로 넘어가기

Substrate 개발자 허브는 커뮤니티에 제공되는 다양한 리소스에 대한 중앙 포털 역할을 합니다.
관심사와 학습 스타일에 따라 선호하는 방법이 다를 수 있습니다.
예를 들어, 소스 코드를 읽는 것을 선호하고 Rust에 익숙한 경우 [Rust API](https://paritytech.github.io/substrate/master)를 살펴보는 것으로 시작할 수 있습니다.

#### 알려주세요

- [아키텍처](../../learn/basic/architecture.md)
- [네트워크와 블록체인](../../learn/basic/networks-and-nodes.md)
- [빌드 프로세스](../../build/build-process.md)

#### 안내해주세요

- [로컬 블록체인 구축](../build-a-blockchain/build-local-blockchain.md)
- [네트워크 시뮬레이션](../build-a-blockchain/simulate-network.md)
- [신뢰할 수 있는 노드 추가](../build-a-blockchain/add-trusted-nodes.md)

<!-- TODO NAV.YAML -->
<!-- add these back -->
<!--Substrate와 Substrate 생태계에 처음 접하는 경우, [탐색](/main-docs/explore/)을 통해 사용 가능한 리소스와 찾을 수 있는 위치에 대한 넓은 시야를 얻을 수 있습니다.-->