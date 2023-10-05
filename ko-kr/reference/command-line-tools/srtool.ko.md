---
title: srtool
description: Substrate WebAssembly 런타임을 빌드하기 위한 Docker 컨테이너와 명령줄 인터페이스에 대해 설명합니다.
keywords:
---

Substrate 런타임 도구 상자(`srtool`)의 핵심 구성 요소는 Substrate WebAssembly 런타임을 결정론적인 방식으로 빌드할 수 있게 해주는 Docker 컨테이너입니다.
이 도구를 사용하면 동일한 소스 코드가 항상 동일한 WebAssembly blob을 재현한다는 것을 보장할 수 있습니다.
또한 이 도구를 사용하여 Substrate 기반 체인의 런타임을 검사하고 감사하며, WebAssembly 런타임 빌드를 CI/CD 파이프라인에 통합할 수도 있습니다.

## Docker 컨테이너 사용하기

`srtool`은 Docker 컨테이너이므로 사용하려면 빌드 환경에 Docker가 설치되어 있어야 합니다.
그러나 `srtool`을 사용하여 체인을 빌드하는 데 Docker를 사용하는 방법에 대해 알 필요는 없습니다. `srtool-cli` 명령줄 인터페이스를 사용하여 Docker 이미지와 작업할 수 있습니다.

`srtool-cli` 패키지는 Rust로 작성된 명령줄 유틸리티로, 컴퓨터에 `srtool`이라는 실행 가능한 프로그램을 설치합니다.
이 프로그램은 `srtool` Docker 컨테이너와 상호 작용을 단순화합니다.
`srtool` Docker 이미지를 사용하는 도구는 다음과 같은 도구와 도움 프로그램을 포함하고 있습니다:

- [srtool-cli](https://github.com/chevdor/srtool-cli)는 `srtool` Docker 이미지를 가져오고 이미지와 상호 작용하는 데 사용되는 도구에 대한 명령줄 인터페이스를 제공합니다. 또한 `srtool` Docker 컨테이너를 사용하여 런타임을 빌드하는 기능도 제공합니다.

- [subwasm](https://github.com/chevdor/subwasm)은 `srtool`을 사용하여 빌드된 메타데이터와 WebAssembly 런타임을 작업하는 데 사용되는 명령줄 옵션을 제공합니다. `subwasm` 프로그램은 `srtool` 이미지에서 작업을 수행하는 데에도 사용됩니다.

- [srtool-actions](https://github.com/chevdor/srtool-actions)은 `srtool` 이미지로 생성된 빌드를 GitHub CI/CD 파이프라인에 통합하기 위한 Github 액션을 제공합니다.

- [srtool-app](https://gitlab.com/chevdor/srtool-app)은 `srtool` Docker 이미지를 사용하여 런타임을 빌드하기 위한 간단한 그래픽 사용자 인터페이스를 제공합니다.

## srtool-cli 설치하기

`srtool` 명령줄 인터페이스를 사용하여 `srtool` Docker 이미지를 사용하여 WebAssembly 런타임을 빌드할 수 있습니다.
다음 명령을 실행하여 `srtool` 명령줄 인터페이스를 설치할 수 있습니다:

```bash
cargo install --git https://github.com/chevdor/srtool-cli
```

### 기본 명령 사용법

`srtool` 명령을 실행하는 기본 구문은 다음과 같습니다:

```text
srtool [options] [subcommand]
```

### 옵션

`srtool` 명령과 함께 다음 명령줄 옵션을 사용할 수 있습니다.

| 옵션                      | 설명
| :---------------------------| :---------------------
| `-h`,&nbsp;`--help`           | 사용법 정보를 표시합니다.
| `i`,&nbsp;`--image`&nbsp;`<image>` | 대체 이미지를 지정합니다. 기본 `paritytech/srtool` 이미지와 호환되는 이미지를 지정해야 합니다. 다른 이미지를 지정하면 `paritytech/srtool` 이미지와 동일한 결정론적인 결과를 얻을 수 없을 수 있습니다.
| `-n`,&nbsp;`--no-cache`&nbsp; | 로컬 캐시의 태그 값을 사용하지 않도록 지정합니다.
| `-V`,&nbsp;`--version`&nbsp; | 버전 정보를 표시합니다.

### 하위 명령

`srtool` 명령과 함께 다음 하위 명령을 사용할 수 있습니다.

| 명령                    | 설명
|:-------------------------- |:-----------
| `build` | 런타임을 빌드하기 위해 새로운 `srtool` 컨테이너를 시작합니다.
| `help` | `srtool` 또는 지정된 하위 명령에 대한 사용법 정보를 표시합니다.
| `info` | `srtool` 컨테이너와 리포지토리에 대한 정보를 표시합니다.
| `pull` | 다른 작업을 실행하지 않고 `srtool` 이미지를 가져옵니다.
| `version` | `srtool` 컨테이너의 버전 정보를 표시합니다. `srtool-cli` 실행 파일의 버전 정보를 얻으려면 `--version`을 사용하세요.

### 예제

`srtool` Docker 이미지의 버전 정보를 가져오려면 다음 명령을 실행합니다:

```bash
srtool version
```

다음과 유사한 출력이 표시됩니다:

```text
{
  "name": "srtool",
  "version": "0.9.21",
  "rustc": "1.62.0",
  "subwasm": "0.18.0",
  "tera": "0.2.1",
  "toml": "0.2.1"
}
```

`srtool-cli` 실행 파일의 버전 정보를 얻으려면 다음 명령을 실행합니다:

```bash
srtool --version
```

다음과 유사한 출력이 표시됩니다:

```text
srtool-cli 0.8.0
```

## srtool build

`srtool build` 명령을 사용하여 지정한 패키지의 런타임을 빌드하기 위해 새로운 `srtool` 컨테이너를 시작할 수 있습니다.
기본적으로 `srtool build` 명령은 런타임의 `Cargo.toml` 파일이 체인의 이름을 가진 [`runtime`](https://github.com/paritytech/polkadot-sdk/tree/master/polkadot/runtime) 하위 디렉토리에 위치해 있다고 가정합니다.
예를 들어, `srtool build` 명령은 기본적으로 다음 위치를 사용합니다:

- runtime/kusama
- runtime/polkadot
- runtime/rococo
- runtime/westend
  
런타임의 `Cargo.toml` 파일이 다른 위치에 있는 경우, 이를 명령줄 옵션으로 지정할 수 있습니다.

### 기본 사용법

`srtool build` 명령을 실행하는 기본 구문은 다음과 같습니다:

```text
srtool build [options] --package <package> [--runtime-dir <path>] [project-path]
```

### 인수

기본적으로 `srtool build`는 현재 작업 디렉토리에서 실행됩니다. 프로젝트가 현재 작업 디렉토리에 없는 경우, 프로젝트 위치의 경로를 지정할 수 있습니다.

| 인수       | 설명
| :------------- | :-----------
| `project-path` | 런타임을 빌드할 블록체인 프로젝트의 경로를 지정합니다.

### 옵션

`srtool build` 명령과 함께 다음 명령줄 옵션을 사용할 수 있습니다.

| 옵션            | 설명
| :---------------- | :-----------
| `-a`,&nbsp;`--app` | 빌드 중 표준 출력과 JSON 출력을 혼합하여 사용합니다. 이 옵션은 CI에 권장됩니다. JSON 출력은 빌드의 끝에 한 줄로 제공됩니다.
| `--build-opts`&nbsp;`<BUILD_OPTS>` | 사용자 정의 옵션을 `cargo` 빌드 프로세스에 직접 전달할 수 있습니다. 이 명령줄 옵션을 지정하면 Kusama 또는 Polkadot을 빌드하는 데 필요한 자동 옵션은 빌드 프로세스에 전달되지 않습니다. `--build-opts` 명령줄 옵션을 사용할 때 필요한 빌드 옵션을 명시적으로 설정해야 합니다. 일반적으로 이 옵션은 거의 사용되지 않습니다. 이 옵션은 `BUILD_OPTS` 환경 변수를 설정하는 것과 동일합니다.
| `--default-features`&nbsp;`<default-features>` | 자동 기능 감지를 비활성화하지 않고 런타임의 기본 기능 목록을 변경할 수 있습니다. 이 명령줄 옵션은 `DEFAULT_FEATURES` 환경 변수를 설정하는 것과 동일합니다. 이 명령줄 옵션은 `BUILD_OPTS`를 설정하지 않은 경우에만 효과가 있습니다.
| `-h`,&nbsp;`--help` | 사용법 정보를 표시합니다.
| `i`,&nbsp;`--image`&nbsp;`<image>` | 대체 이미지를 지정합니다. 기본 `paritytech/srtool` 이미지와 호환되는 이미지를 지정해야 합니다. 다른 이미지를 지정하면 `paritytech/srtool` 이미지와 동일한 결정론적인 결과를 얻을 수 없을 수 있습니다.
| `-j`,&nbsp;`--json` | JSON 출력을 사용합니다.
| `--no-cache` | 모든 캐싱을 비활성화합니다. 이 옵션을 지정하면 `srtool` 이미지가 빌드 종속성을 위해 Cargo 홈 캐시에 액세스하지 않습니다. 일반적으로 이 옵션은 거의 사용되지 않습니다.
| `-p`,&nbsp;`--package`&nbsp;`<package>` | 빌드할 런타임 패키지의 이름을 지정합니다. 지정하는 이름은 런타임의 `Cargo.toml` 파일에 정의된 이름과 동일해야 합니다. 예를 들어, kusama-runtime, polkadot-runtime 등입니다. 이 옵션은 `PACKAGE` 환경 변수를 설정하는 것과 동일합니다.
| `--profile`&nbsp;`<profile>` | 런타임을 빌드하는 데 사용할 프로파일을 지정합니다. 런타임을 빌드하는 기본 프로파일은 항상 `release`입니다. 이 명령줄 옵션으로 기본값을 재정의할 수 있습니다. 이 옵션은 `PROFILE` 환경 변수를 설정하는 것과 동일합니다.
| `-r`,&nbsp;`--runtime-dir`&nbsp;`<runtime>` | 런타임의 `Cargo.toml` 파일의 위치를 지정합니다. 런타임이 표준 위치에 없는 경우, 이 명령줄 옵션을 사용하여 올바른 위치를 지정할 수 있습니다. 이 옵션은 `RUNTIME_DIR` 환경 변수를 설정하는 것과 동일합니다.
| `-V`,&nbsp;`--version` | 버전 정보를 표시합니다.

### 예제

`cumulus` 리포지토리에서 `parachains/runtimes/assets/asset-hub-westend` 경로에 있는 `Cargo.toml` 파일을 사용하여 Westend 런타임을 빌드하려면 다음 명령을 실행합니다:

```bash
srtool build --app --package westmint-runtime --runtime-dir parachains/runtimes/assets/westmint
```

`srtool build` 명령을 처음 실행할 때는 완료까지 시간이 걸립니다.
런타임이 컴파일되는 동안 진행 상황에 대한 메시지가 표준 출력으로 표시됩니다.
이 예제에서는 `--app` 명령줄 옵션을 사용하므로 JSON 출력이 빌드의 끝에 한 줄로 표시됩니다. 다음은 일부 내용이 생략된 출력 예시입니다:

```text
...
   Compiling cumulus-primitives-parachain-inherent v0.1.0 (/build/primitives/parachain-inherent)
   Compiling cumulus-pallet-parachain-system v0.1.0 (/build/pallets/parachain-system)
    Finished release [optimized] target(s) in 112m 11s
✨ Your Substrate WASM Runtime is ready! ✨
{"gen":"srtool v0.9.21","src":"git","version":"1.0.0","commit":"bd41e3f11887ea2f55fc37be71ff652923388e03","tag":"v0.9.220-rc2","branch":"master","rustc":"rustc 1.62.0 (a8314ef7d 2022-06-27)","pkg":"westmint-runtime","tmsp":"2022-08-22T21:12:18Z","size":"707937","prop":"0x6b8e93443b6660a16f67a6cd34d415af463e2285eda3fd02b9fe052c1ad2ceb9"
... }}}}
```

## srtool help

`srtool help` 명령을 사용하여 `srtool` 또는 지정된 하위 명령에 대한 사용법 메시지를 표시할 수 있습니다.

### 기본 사용법

```text
srtool help [subcommand]
```

### 예제

`build` 하위 명령에 대한 사용법 정보를 표시하려면 다음 명령을 실행합니다:

```bash
subkey help build
```

## srtool info

`srtool info` 명령을 사용하여 `srtool` 컨테이너와 리포지토리에 대한 정보를 표시할 수 있습니다.
기본적으로 `srtool info` 명령은 런타임의 `Cargo.toml` 파일이 체인의 이름을 가진 [`runtime`](https://github.com/paritytech/polkadot-sdk/tree/master/polkadot/runtime) 하위 디렉토리에 위치해 있다고 가정합니다.
예를 들어, `srtool info` 명령은 기본적으로 다음 위치를 사용합니다:

- runtime/kusama
- runtime/polkadot
- runtime/rococo
- runtime/westend
  
런타임의 `Cargo.toml` 파일이 다른 위치에 있는 경우, 이를 명령줄 옵션으로 지정할 수 있습니다.

### 기본 사용법

`srtool info` 명령을 실행하는 기본 구문은 다음과 같습니다:

```text
srtool info [options] --package <package> [--runtime-dir <path>] [project-path]
```

### 인수

기본적으로 `srtool info`는 현재 작업 디렉토리에서 실행됩니다. 
프로젝트가 현재 작업 디렉토리에 없는 경우, 프로젝트 위치의 경로를 지정할 수 있습니다.

| 인수       | 설명
| :-------------- | :-----------
| `project-path` | 현재 작업 디렉토리에 프로젝트가 없는 경우, 블록체인 프로젝트의 경로를 지정합니다.

### 옵션

`srtool info` 명령과 함께 다음 명령줄 옵션을 사용할 수 있습니다.

| 옵션                | 설명
| :-------------------- | -----------
| `-h`,&nbsp;`--help` | 사용법 정보를 표시합니다.
| `i`,&nbsp;`--image`&nbsp;`<image>` | 대체 이미지를 지정합니다. 기본 `paritytech/srtool` 이미지와 호환되는 이미지를 지정해야 합니다. 다른 이미지를 지정하면 `paritytech/srtool` 이미지와 동일한 결정론적인 결과를 얻을 수 없을 수 있습니다.
| `-p`,&nbsp;`--package`&nbsp;`<package>` | 빌드할 런타임 패키지의 이름을 지정합니다. 지정하는 이름은 런타임의 `Cargo.toml` 파일에 정의된 이름과 동일해야 합니다. 예를 들어, kusama-runtime, polkadot-runtime 등입니다. 이 옵션은 `PACKAGE` 환경 변수를 설정하는 것과 동일합니다.
| `-r`,&nbsp;`--runtime-dir`&nbsp;`<runtime>` | 런타임의 `Cargo.toml` 파일의 위치를 지정합니다. 런타임이 표준 위치에 없는 경우, 이 명령줄 옵션을 사용하여 올바른 위치를 지정할 수 있습니다. 이 옵션은 `RUNTIME_DIR` 환경 변수를 설정하는 것과 동일합니다.
| `-V`,&nbsp;`--version` | 버전 정보를 표시합니다.

### 예제

`srtool` 컨테이너와 로컬 `node-template` 리포지토리에 대한 정보를 표시하려면 다음과 유사한 명령을 실행할 수 있습니다:

```bash
srtool info --package node-template-runtime --runtime-dir runtime
```

다음과 유사한 출력이 표시됩니다:

```text
{
  "generator": {
    "name": "srtool",
    "version": "0.9.21"
  },
  "src": "git",
  "version": "4.0.0-dev",
  "git": {
    "commit": "6a8b2b12371395979099d2c79ccc1860531b0449",
    "tag": "",
    "branch": "my-release-branch"
  },
  "rustc": "rustc 1.62.0 (a8314ef7d 2022-06-27)",
  "pkg": "polkadot-runtime",
  "profile": "release"
}
```

## srtool pull

`srtool pull` 명령을 사용하여 `srtool` Docker 이미지의 최신 버전을 확인하고 다운로드할 수 있습니다.

### 기본 사용법

`srtool pull` 명령을 실행하는 기본 구문은 다음과 같습니다:

```text
srtool pull [options]
```

### 옵션

`srtool pull` 명령과 함께 다음 명령줄 옵션을 사용할 수 있습니다.

| 옵션              | 설명
| :-------------------| :-----------
| `-h`,&nbsp;`--help`  | 사용법 정보를 표시합니다.
| `i`,&nbsp;`--image`&nbsp;`<image>` | 대체 이미지를 지정합니다. 기본 `paritytech/srtool` 이미지와 호환되는 이미지를 지정해야 합니다. 다른 이미지를 지정하면 `paritytech/srtool` 이미지와 동일한 결정론적인 결과를 얻을 수 없을 수 있습니다.
| `-V`,&nbsp;`--version` | 버전 정보를 표시합니다.

### 예제

`srtool` 컨테이너와 Docker 이미지의 최신 버전을 확인하려면 다음과 유사한 명령을 실행할 수 있습니다:

```bash
srtool pull
```

이 명령은 Docker Hub에서 `paritytech/srtool` 이미지의 최신 버전을 확인하고 소프트웨어를 다운로드하고 추출하기 시작합니다.
예를 들어,

```text
Found 1.62.0, we will be using paritytech/srtool:1.62.0 for the build
1.62.0: Pulling from paritytech/srtool
405f018f9d1d: Pull complete 
c49473e7f7b3: Pull complete 
7edf98d07029: Pull complete 
85a50724a6fa: Pull complete 
87fb1e3dee5b: Downloading   19.4MB/170.4MB
469075c5d317: Download complete 
533bfa44b64a: Download complete 
...
```

모든 작업이 완료되면 다음과 유사한 출력이 표시됩니다:

```text
Digest: sha256:d5353a63d8fccbef5666e28a8fa0b302d71d4f53cabeb760fe213f3d7df4b8b6
Status: Downloaded newer image for paritytech/srtool:1.62.0
docker.io/paritytech/srtool:1.62.0
```

이미 최신 버전이 로컬에 설치되어 있는 경우, 다음과 유사한 출력이 표시됩니다:

```text
Found 1.62.0, we will be using paritytech/srtool:1.62.0 for the build
1.62.0: Pulling from paritytech/srtool
Digest: sha256:d5353a63d8fccbef5666e28a8fa0b302d71d4f53cabeb760fe213f3d7df4b8b6
Status: Image is up to date for paritytech/srtool:1.62.0
docker.io/paritytech/srtool:1.62.0
```

## srtool version

`srtool version` 명령을 사용하여 `srtool` 컨테이너의 버전 정보를 표시할 수 있습니다. `srtool-cli` 실행 파일의 버전을 얻으려면 `--version`을 사용하세요.

### 기본 사용법

`srtool version` 명령을 실행하는 기본 구문은 다음과 같습니다:

```text
srtool version [options]
```

### 옵션

`srtool version` 명령과 함께 다음 명령줄 옵션을 사용할 수 있습니다.

| 옵션           | 설명
| :----------------| :-------------
| `-h`,&nbsp;`--help` | 사용법 정보를 표시합니다.
| `i`,&nbsp;`--image`&nbsp;`<image>` | 대체 이미지를 지정합니다. 기본 `paritytech/srtool` 이미지와 호환되는 이미지를 지정해야 합니다. 다른 이미지를 지정하면 `paritytech/srtool` 이미지와 동일한 결정론적인 결과를 얻을 수 없을 수 있습니다.
| `-V`,&nbsp;`--version` | 버전 정보를 표시합니다.

### 예제

`srtool` 컨테이너에 대한 정보를 표시하려면 다음 명령을 실행합니다:

```bash
srtool version       
```

다음과 유사한 출력이 표시됩니다:

```text
{
  "name": "srtool",
  "version": "0.9.21",
  "rustc": "1.62.0",
  "subwasm": "0.18.0",
  "tera": "0.2.1",
  "toml": "0.2.1"
}
```

`srtool-cli` 실행 파일의 버전 정보를 표시하려면 다음 명령을 실행합니다:

```bash
srtool version --version
```

다음과 유사한 출력이 표시됩니다:

```text
srtool-cli 0.8.0
```
