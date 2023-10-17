---
title: Windows 개발 환경
description: Windows에서 Substrate를 위한 로컬 개발 환경을 설정합니다.
keywords:
---

일반적으로 macOS나 Linux와 같은 UNIX 기반 운영 체제는 Substrate 기반 블록체인을 구축하기 위한 개발 환경을 제공하는 데 더 적합합니다.
Substrate [튜토리얼](/tutorials/) 및 [How-to 가이드](/reference/how-to-guides/)의 모든 코드 예제와 명령 줄 지침은 터미널에서 UNIX 호환 명령을 사용하여 Substrate와 상호 작용하는 방법을 보여줍니다.

그러나 로컬 컴퓨터가 UNIX 기반 운영 체제 대신 Microsoft Windows를 사용하는 경우, Substrate 기반 블록체인을 구축하기 위한 적합한 개발 환경으로 구성하기 위해 추가 소프트웨어를 설치할 수 있습니다.
Microsoft Windows에서 개발 환경을 준비하려면 Windows Subsystem for Linux (WSL)을 사용하여 UNIX 운영 환경을 에뮬레이션할 수 있습니다.

## 시작하기 전에

Microsoft Windows에 설치하기 전에 다음 기본 요구 사항을 확인하세요:

- 지원되는 버전의 Microsoft Windows 운영 체제가 실행되는 컴퓨터가 있어야 합니다.
- Windows 데스크톱 운영 체제를 사용하는 컴퓨터에 Windows Subsystem for Linux를 설치하려면 Microsoft Windows 10, 버전 2004 이상 또는 Microsoft Windows 11이 실행되어야 합니다.
- Windows 서버 운영 체제를 사용하는 컴퓨터에 Windows Subsystem for Linux를 설치하려면 Microsoft Windows Server 2019 이상이 실행되어야 합니다.
- 로컬 컴퓨터에서 좋은 인터넷 연결과 셸 터미널에 액세스할 수 있어야 합니다.

## Windows Subsystem for Linux 설치하기

Windows Subsystem for Linux (WSL)을 사용하면 Windows 운영 체제를 사용하는 컴퓨터에서 Linux 환경을 에뮬레이션할 수 있습니다.
Substrate 개발에 대한 이 접근 방식의 주요 장점은 Substrate 문서에서 설명하는 모든 코드와 명령 줄 예제를 사용할 수 있다는 것입니다.
예를 들어, `ls`나 `ps`와 같은 일반적인 명령을 수정하지 않고 실행할 수 있습니다.
Windows Subsystem for Linux를 사용하면 가상 머신 이미지나 듀얼 부팅 운영 체제를 구성할 필요가 없습니다.

Windows Subsystem for Linux을 사용하여 개발 환경을 준비하려면 다음 단계를 따르세요:

1. Windows 버전과 빌드 번호를 확인하여 Windows Subsystem for Linux가 기본적으로 활성화되어 있는지 확인합니다.

   Microsoft Windows 10, 버전 2004 (빌드 19041 이상) 또는 Microsoft Windows 11을 사용하는 경우 Windows Subsystem for Linux가 기본적으로 사용 가능하며 다음 단계로 진행할 수 있습니다.

   이전 버전의 Microsoft Windows를 사용하는 경우 [이전 버전에 대한 WSL 수동 설치 단계](https://docs.microsoft.com/en-us/windows/wsl/install-manual)를 참조하세요.
   Microsoft Windows의 이전 버전에 설치하는 경우 Windows 10, 버전 1903 이상이 있는 경우 WSL 2를 다운로드하고 설치할 수 있습니다.

1. 시작 메뉴에서 Windows PowerShell 또는 명령 프롬프트를 선택하고 마우스 오른쪽 버튼을 클릭한 다음 **관리자 권한으로 실행**을 선택합니다.

1. PowerShell 또는 명령 프롬프트 터미널에서 다음 명령을 실행합니다:

   ```bash
   wsl --install
   ```

   이 명령은 Windows 운영 체제의 일부인 필요한 WSL 2 구성 요소를 활성화하고 최신 Linux 커널을 다운로드한 다음 Ubuntu Linux 배포판을 기본적으로 설치합니다.

   다른 Linux 배포판을 확인하려면 다음 명령을 실행하세요:

   ```bash
   wsl --list --online
   ```

1. 배포판이 다운로드되면 터미널을 닫습니다.

1. 시작 메뉴를 클릭하고 **종료 또는 로그아웃**을 선택한 다음 **재시작**을 클릭하여 컴퓨터를 재시작합니다.

   컴퓨터를 재시작해야 Linux 배포판의 설치가 시작됩니다.
   재시작 후 설치가 완료되기까지 몇 분이 걸릴 수 있습니다.

   WSL을 개발 환경으로 설정하는 자세한 내용은 [WSL 개발 환경 설정](https://docs.microsoft.com/en-us/windows/wsl/setup/environment)을 참조하세요.

## 필수 패키지와 Rust 설치하기

WSL에서 Rust 도구 체인을 설치하려면 다음 단계를 따르세요:

1. 시작 메뉴를 클릭한 다음 **Ubuntu**를 선택합니다.

1. UNIX 사용자 이름을 입력하여 사용자 계정을 생성합니다.

1. UNIX 사용자의 암호를 입력하고 확인하기 위해 암호를 다시 입력합니다.

1. 다음 명령을 실행하여 Ubuntu 배포판에 대한 최신 업데이트를 다운로드합니다:

   ```bash
   sudo apt update
   ```

1. 다음 명령을 실행하여 Ubuntu 배포판에 필요한 패키지를 추가합니다:

   ```bash
   sudo apt install --assume-yes git clang curl libssl-dev llvm libudev-dev make protobuf-compiler
   ```

1. `rustup` 설치 프로그램을 다운로드하고 다음 명령을 실행하여 Ubuntu 배포판에 Rust를 설치합니다:

   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```

1. 표시된 프롬프트에 따라 기본 설치를 진행합니다.

1. 다음 명령을 실행하여 현재 셸에 Cargo를 포함시킵니다:

   ```bash
   source ~/.cargo/env
   ```

1. 다음 명령을 실행하여 설치를 확인합니다:

   ```bash
   rustc --version
   ```

1. 다음 명령을 실행하여 Rust 도구 체인을 최신 안정 버전을 사용하는 기본 도구 체인으로 설정합니다:

   ```bash
   rustup default stable
   rustup update
   ```

1. 다음 명령을 실행하여 `nightly` 버전의 도구 체인과 `nightly` WebAssembly (`wasm`) 대상을 개발 환경에 추가합니다:

   ```bash
   rustup update nightly
   rustup target add wasm32-unknown-unknown --toolchain nightly
   ```

1. 다음 명령을 실행하여 개발 환경의 구성을 확인합니다:

   ```bash
   rustup show
   rustup +nightly show
   ```

   이 명령은 다음과 유사한 출력을 표시합니다:

   ```bash
   # rustup show

   active toolchain
   ----------------

   stable-x86_64-unknown-linux-gnu (default)
   rustc 1.61.0 (fe5b13d68 2022-05-18)

   # rustup +nightly show

   active toolchain
   ----------------

   nightly-x86_64-unknown-linux-gnu (overridden by +toolchain on the command line)
   rustc 1.63.0-nightly (e71440575 2022-06-02)
   ```

## Substrate 노드 컴파일하기

Rust가 설치되고 Substrate 개발을 위해 Rust 도구 체인이 구성되었으므로 Substrate 개발 환경을 설정하기 위해 Substrate **노드 템플릿** 파일을 복제하고 Substrate 노드를 컴파일할 준비가 되었습니다.

노드 템플릿은 불필요한 모듈이나 도구 없이 블록체인을 구축하는 데 필요한 모든 일반적인 기능을 포함한 작업 환경을 제공합니다.
실험을 위한 상대적으로 안정적인 작업 환경을 제공하기 위해, 핵심 Substrate 저장소 대신 Substrate 개발자 허브 저장소에서 Substrate 노드 템플릿을 복제하는 것이 권장되는 모범 사례입니다.

Substrate 노드 템플릿을 컴파일하려면 다음 단계를 따르세요:

1. 다음 명령을 실행하여 노드 템플릿 저장소를 복제합니다:

   ```bash
   git clone https://github.com/substrate-developer-hub/substrate-node-template
   ```

   대부분의 경우 최신 코드를 얻기 위해 `main` 브랜치를 복제할 수 있습니다.
   그러나 특정 Polkadot 버전과 호환되는 Substrate 브랜치와 작업하려는 경우 `--branch` 명령 줄 옵션을 사용할 수 있습니다.
   특정 Polkadot 버전과 호환되는 브랜치 목록을 보려면 [Tags](https://github.com/substrate-developer-hub/substrate-node-template/tags)를 클릭하세요.

1. 다음 명령을 실행하여 노드 템플릿 디렉토리의 루트로 이동합니다:

   ```bash
   cd substrate-node-template
   ```

   변경 사항을 저장하고 이 브랜치를 쉽게 식별할 수 있도록 하려면 다음과 유사한 명령을 실행하여 새 브랜치를 생성할 수 있습니다:

   ```bash
   git switch -c my-wip-branch
   ```

1. 다음 명령을 실행하여 노드 템플릿을 컴파일합니다:

   ```bash
   cargo build --release
   ```

   필요한 패키지의 수가 많기 때문에 노드를 컴파일하는 데는 몇 분이 걸릴 수 있습니다.

빌드가 성공적으로 완료되면 로컬 컴퓨터는 Substrate 개발 활동을 위해 준비된 상태입니다.

## 다음 단계로 넘어가기

Substrate 개발자 허브는 커뮤니티에 제공되는 다양한 리소스에 대한 중앙 포털 역할을 합니다.
관심사와 학습 스타일에 따라 선호하는 방법이 다를 수 있습니다.
예를 들어, 소스 코드를 읽는 것을 선호하고 Rust에 익숙한 경우 [Rust API](https://paritytech.github.io/substrate/master)를 파고들며 시작할 수 있습니다.

<!-- TODO NAV.YAML -->
<!-- add these back -->
<!--Substrate 및 Substrate 생태계에 처음 접하는 경우 [탐색](/explore/)을 통해 사용 가능한 리소스와 찾을 수 있는 위치에 대한 더 넓은 이해를 얻을 수 있습니다.-->

더 자세한 내용을 알고 싶다면 다음을 참조하세요.

#### 알려주세요

- [아키텍처](/learn/architecture/)
- [네트워크와 블록체인](/learn/node-and-network-types/)
- [빌드 프로세스](/build/build-process)

#### 안내해주세요

- [로컬 블록체인 구축](/tutorials/build-a-blockchain/build-local-blockchain/)
- [네트워크 시뮬레이션](/tutorials/build-a-blockchain/simulate-network/)
- [신뢰할 수 있는 노드 추가](/tutorials/build-a-blockchain/add-trusted-nodes/)