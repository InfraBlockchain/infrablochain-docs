---
title: Rust 문제 해결 방법
description: Rust 및 Substrate 개발 환경 문제를 진단하고 해결합니다.
keywords:
  - Rust
  - 툴체인
  - 컴파일러
  - rustup
---

만약 [Substrate 노드 템플릿](https://github.com/substrate-developer-hub/substrate-node-template)을 컴파일하는데 실패한다면, 문제는 개발 환경에서 Rust가 어떻게 구성되어 있는지에 의해 발생한 것입니다.
이 섹션에서는 구성 문제를 진단하고 해결하는 방법을 제안합니다.

## 현재 구성 확인하기

현재 사용 중인 Rust 툴체인에 대한 정보를 확인하려면 다음 명령을 실행하세요:

```bash
rustup show
```

이 명령은 다음과 같은 출력을 표시합니다(Ubuntu 예시):

```text
Default host: x86_64-unknown-linux-gnu
rustup home:  /home/user/.rustup

installed toolchains
--------------------

stable-x86_64-unknown-linux-gnu (default)
nightly-2020-10-06-x86_64-unknown-linux-gnu
nightly-x86_64-unknown-linux-gnu

installed targets for active toolchain
--------------------------------------

wasm32-unknown-unknown
x86_64-unknown-linux-gnu

active toolchain
----------------

stable-x86_64-unknown-linux-gnu (default)
rustc 1.50.0 (cb75ad5db 2021-02-10)
```

이 예시에서는 기본 툴체인이 x86_64 아키텍처에서 실행되는 Linux용 `stable` 릴리스 채널에서 가져온 것입니다.
샘플 출력은 또한 `nightly-x86_64-unknown-linux-gnu` 툴체인이 설치되어 있으며, 두 개의 타겟이 설치되어 있는 것을 나타냅니다:

- `x86_64-unknown-linux-gnu`: Linux용 네이티브 Rust 타겟.
- `wasm32-unknown-unknown`: WebAssembly 타겟.

이 환경에는 `nightly-2020-10-06-x86_64-unknown-linux-gnu` 툴체인도 설치되어 있지만, 이 툴체인은 명시적으로 명령줄 옵션으로 지정하지 않는 한 사용되지 않습니다.
특정 툴체인을 명령줄 옵션으로 지정하는 예시는 [특정 nightly 버전 지정](#특정-nightly-버전-지정)을 참조하세요.

## WebAssembly을 위해 nightly 릴리스 채널 사용하기

Substrate은 블록체인 런타임을 생성하기 위해 [WebAssembly](https://webassembly.org) (Wasm)을 사용합니다.
Rust 컴파일러를 [`nightly` 빌드](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html)를 사용하도록 구성해야 Substrate 런타임 코드를 Wasm 타겟으로 컴파일할 수 있습니다.

## 툴체인 업데이트하기

일반적으로, Rust `stable` 및 `nightly` 빌드의 최신 버전을 항상 사용해야 합니다. Substrate의 변경 사항은 종종 Rust `nightly` 컴파일러 빌드의 상위 변경 사항에 의존하기 때문입니다.
Rust 컴파일러가 항상 최신 상태인지 확인하기 위해 다음 명령을 실행해야 합니다:

```bash
rustup update
rustup update nightly
rustup target add wasm32-unknown-unknown --toolchain nightly
```

`rustup update` 명령은 `nightly` 및 `stable` 툴체인을 모두 가장 최신 릴리스로 업데이트합니다.
`nightly` 툴체인을 업데이트한 후에도 WebAssembly 타겟을 컴파일할 수 없는 경우, 이전 버전의 툴체인으로 롤백하고 해당 버전을 명령줄 옵션으로 지정할 수 있습니다.
`nightly` 툴체인의 이전 버전을 얻고 명령줄 옵션으로 사용할 버전을 지정하는 자세한 정보는 [툴체인 다운그레이드](#Rust-nightly-다운그레이드)를 참조하세요.

## 특정 nightly 툴체인 사용하기

Rust와 다른 종속성을 업데이트하면서 빌드가 컴퓨터에서 작동하는지 보장하려면, 사용 중인 Substrate 버전과 호환되는 특정 Rust `nightly` 툴체인을 사용해야 합니다.
프로젝트에 대해 특정 `nightly` 툴체인 버전을 식별하고 전달하는 방법은 다양할 수 있습니다.
예를 들어, Polkadot은 [릴리스 노트](https://github.com/paritytech/polkadot/releases)에서 이 정보를 게시합니다.

사용할 특정 `nightly` 툴체인 버전을 식별한 후, 다음과 유사한 명령을 실행하여 개발 환경에 설치할 수 있습니다:

```bash
rustup install nightly-<yyyy-MM-dd>
```

예를 들어:

```bash
rustup install nightly-2022-02-16
```

특정 버전의 nightly 툴체인을 설치한 후, 다음과 유사한 명령을 실행하여 WebAssembly 타겟을 사용하도록 구성할 수 있습니다:

```bash
rustup target add wasm32-unknown-unknown --toolchain nightly-<yyyy-MM-dd>
```

예를 들어:

```bash
rustup target add wasm32-unknown-unknown --toolchain nightly-2022-02-16
```

### 환경 변수로 툴체인 지정하기

`WASM_BUILD_TOOLCHAIN` 환경 변수를 설정하여 WebAssembly 컴파일을 위해 사용할 `nightly` 툴체인의 버전을 지정할 수 있습니다. 예를 들어:

```bash
WASM_BUILD_TOOLCHAIN=nightly-<yyyy-MM-dd> cargo build --release
```

이 명령은 지정된 nightly 툴체인을 사용하여 _런타임_을 빌드합니다. 프로젝트의 나머지 부분은 _기본_ 툴체인, 즉 설치된 최신 `stable` 툴체인을 사용하여 컴파일됩니다.

### nightly 툴체인 다운그레이드하기

컴퓨터가 최신 Rust `nightly` 툴체인을 사용하도록 구성되어 있고 특정 nightly 버전으로 다운그레이드하려면, 먼저 최신 `nightly` 툴체인을 제거해야 합니다.
예를 들어, 최신 `nightly` 툴체인을 제거한 후 특정 버전의 `nightly` 툴체인을 사용하려면 다음과 유사한 명령을 실행하세요:

```sh
rustup uninstall nightly
rustup install nightly-<yyyy-MM-dd>
rustup target add wasm32-unknown-unknown --toolchain nightly-<yyyy-MM-dd>
```

## PATH가 올바르게 설정되었는지 확인하기

Rust를 설치한 후에 명령이 작동하지 않고 `command not found: rustup`과 같은 오류가 표시된다면, PATH가 올바르게 구성되었는지 확인하세요.

현재 `rustup` 설치 프로그램은 기본적으로 bash 프로필(mac의 경우)에 설치합니다. 다른 쉘을 사용하는 경우 프로필(예: `.zshrc`)에 다음 줄을 추가해야 합니다:

```bash
source "$HOME/.cargo/env"
```

## M1 macOS 사용자를 위한 cmake 또는 protobuf 설치

현재, M1 칩을 사용하는 macOS 컴퓨터에 미리 설치된 패키지를 사용하여 Substrate 노드를 컴파일하는 데 문제가 있습니다.

```sh
error: failed to run custom build command for prost-build v0.10.4
```

이 오류가 발생하는 경우 두 가지 해결 방법이 있습니다.

- 다음 명령을 실행하여 `cmake`를 설치하세요:

```bash
brew install cmake
```

- 다음 명령 세트를 실행하여 올바른 미리 컴파일된 `protoc`를 설치하세요:

```bash
git clone https://github.com/protocolbuffers/protobuf.git
cd protobuf

brew install autoconf
brew install automake
brew install Libtool

autoreconf -i
./autogen.sh
./configure
make
make check
sudo make install

export PATH=/opt/usr/local/bin:$PATH
```
