---
title: 인프라 릴레이 체인 구축하기
description: 이 튜토리얼은 인프라 릴레이 체인을 구축하는 방법에 대해 알아봅니다.
keywords:
  - 릴레이 체인
---


이 세션에서는 인프라 릴레이 체인 구축하는 방법 대해 작성한 세션입니다.

먼저 인프라 릴레이 체인 build 하는 방법에 대해 알아보겠습니다.

Build는 다음과 같이 두 가지를 방식이 있습니다.

- binary build
- container image build

두 가지 방식을 어떻게 해야하는지에 대해 알아보겠습니다.

## Binary Build

binary로 빌드하는 것은 아주 쉽습니다

먼저 git clone을 활용해서 infrablockspace 관련 chain repository를 clone을 받아 줍니다.

그런 다음 folder 내부로 들어가줍니다.

repository 내부를 확인해 보면 Makefile이 존재하며 이것을 활용해서 다음 명령어를 실행해주면 binary파일이 생성됩니다.

```bash
make build
```

Makefile을 사용하지 않고 빌드를 원하실 경우 아래 명령어를 통해서 빌드를 진행해줍니다.

```bash
cargo build --release
```

## Container Image Build

container image를 build 하는 방법에 대해 알아보겠습니다.

먼저 Binary Build와 동일하게 git clone을 통해 사용하실 chain repository를 다운로드 받습니다. 만약 이미 repository가 있다면 이 단계는 넘어가줍니다.

그런 다음, 아래 명령어를 통해 container image를 생성해줍니다.

```bash
docker build -t <tag name>:<version> .
```

## Chain 관련 파일 생성 방법

Chain spec 생성은 다음과 같은 방법으로 진행합니다.

<chain type>은 사용하는 chain의 type을 의미합니다. 여기에는 local, lococo 등이 들어갈 수 있습니다.

```bash
target/release/infrablockspace build-spec --chain <chain type> --disable-default-bootnode --raw > <file name>.json
```

wasm 생성은 다음과 같은 방법으로 생성해줍니다.

```bash
target/release/infrablockspace build-spec export-genesis-wasm --chain <chain spec file> <wasm file name>.wasm
```

<chain spec file>은 위에서 생성한 chain spec file을 말합니다. 이를 활용해서  wasm 파일을 생성할 수 있으며 <wasm file name>은 생성할 wasm file의 이름을 나타냅니다.

genesis는 다음 명령어를 통해 생성해줍니다.

```bash
target/release/infrablockspace build-spec export-genesis-state --chain <chain spec file> <genesis state file name>
```

wasm에서와 동일하게 <chain spec file>은 위에서 생성한 chain spec file을 말합니다. 이를 활용해서  genesis 파일을 생성할 수 있으며 <genesis state file name>은 생성할 genesis state file의 이름을 나타냅니다.