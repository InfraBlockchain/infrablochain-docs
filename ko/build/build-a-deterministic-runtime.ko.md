타이틀: 결정론적 런타임 빌드
설명: Substrate 기반 체인을 위한 WebAssembly 런타임을 빌드하기 위해 Substrate 런타임 도구 (srtool)와 Docker를 사용하는 방법을 설명합니다.
키워드:

기본적으로 Rust 컴파일러는 최적화된 WebAssembly 이진 파일을 생성합니다.
이러한 이진 파일은 로컬 개발을 수행하는 독립적인 환경에서는 문제가 없습니다.
그러나 컴파일러가 기본적으로 빌드하는 WebAssembly 이진 파일은 결정론적으로 재현 가능하지 않습니다.
컴파일러가 WebAssembly 런타임을 생성할 때마다 약간 다른 WebAssembly 바이트 코드를 생성할 수 있습니다.
이는 모든 노드가 정확히 동일한 체인 사양 파일을 사용해야 하는 블록체인 네트워크에서 문제가 됩니다.

결정론적으로 재현 가능하지 않은 빌드를 사용하면 다른 문제가 발생할 수도 있습니다.
예를 들어, 블록체인의 빌드 프로세스를 자동화하려면 항상 동일한 코드가 동일한 결과를 생성하도록 보장해야 합니다.
결정론적인 빌드 없이는 매번 푸시할 때마다 WebAssembly 런타임을 컴파일하면 일관되지 않고 예측할 수 없는 결과가 생성되어 자동화와 통합하기 어렵고 CI/CD 파이프라인을 계속해서 깨뜨리게 됩니다.
결정론적인 빌드 - 항상 정확히 동일한 바이트 코드로 컴파일되는 코드 - 는 또한 WebAssembly 런타임을 검사, 감사 및 독립적으로 검증할 수 있도록 합니다.

## WebAssembly 런타임을 위한 도구

결정론적인 방식으로 WebAssembly 런타임을 컴파일하는 데 도움이 되는 도구는 Polkadot, Kusama 및 기타 Substrate 기반 체인에 대한 런타임을 생성하는 데 사용되는 동일한 도구입니다.
이 도구는 Substrate 런타임 도구 또는 `srtool`로 통칭되며, 동일한 소스 코드가 항상 동일한 WebAssembly blob으로 컴파일되도록 보장합니다.

Substrate 런타임 도구 (`srtool`)의 핵심 구성 요소는 Docker 컨테이너입니다.
이 컨테이너는 Docker 이미지의 일부로 실행됩니다.
`srtool` Docker 이미지의 이름은 이미지에 포함된 코드를 컴파일하는 데 사용된 Rust 컴파일러의 버전을 지정합니다.
예를 들어, 이미지 `paritytech/srtool:1.62.0`는 이미지에 포함된 코드가 `rustc` 컴파일러의 버전 `1.62.0`으로 컴파일된 것을 나타냅니다.

## Docker 컨테이너 사용하기

`srtool`은 Docker 컨테이너이므로 사용하려면 빌드 환경에 Docker가 설치되어 있어야 합니다.
그러나 `srtool`을 사용하여 Substrate 기반 체인을 빌드하는 데 Docker를 사용하는 방법에 대해 알 필요는 없습니다. `srtool-cli` 명령줄 인터페이스를 사용하여 Docker 이미지와 작업할 수 있습니다.

`srtool-cli` 패키지는 Rust로 작성된 명령줄 유틸리티로, 컴퓨터에 `srtool`이라는 실행 가능한 프로그램을 설치합니다.
이 프로그램은 `srtool` Docker 컨테이너와의 상호작용을 간소화합니다.
시간이 지남에 따라 `srtool` 이미지 주변의 도구들은 다음과 같은 도구와 도움 프로그램을 포함하도록 확장되었습니다:

- [srtool-cli](https://github.com/chevdor/srtool-cli)는 srtool Docker 이미지를 가져오고 이미지 및 상호작용에 사용되는 도구에 대한 정보를 얻으며 `srtool` Docker 컨테이너를 사용하여 런타임을 빌드하는 명령줄 인터페이스를 제공합니다.

- [subwasm](https://github.com/chevdor/subwasm)은 srtool을 사용하여 빌드된 메타데이터 및 WebAssembly 런타임을 사용하는 명령줄 옵션을 제공합니다. `subwasm` 프로그램은 `srtool` 이미지에서 작업을 수행하는 데에도 내부적으로 사용됩니다.

- [srtool-actions](https://github.com/chevdor/srtool-actions)은 `srtool` 이미지를 사용하여 생성된 빌드를 GitHub CI/CD 파이프라인과 통합하기 위한 GitHub 액션을 제공합니다.
- [srtool-app](https://gitlab.com/chevdor/srtool-app)은 `srtool` Docker 이미지를 사용하여 런타임을 빌드하기 위한 간단한 그래픽 사용자 인터페이스를 제공합니다.

## 환경 준비하기

`srtool` Docker 컨테이너에서 코드를 실행하는 Docker 이미지를 사용하려면 Docker 계정과 Docker 명령줄 또는 데스크톱 도구가 설치되어 있어야 합니다.
또한 사용하려는 특정 명령줄 도구로 개발 환경을 준비해야 합니다.
최소한 `srtool-cli` 프로그램을 설치하여 간단한 명령줄 인터페이스를 사용하여 Docker 이미지와 작업할 수 있도록 해야 합니다.

환경을 준비하려면 다음을 수행하세요:

1. Substrate 개발 환경에서 터미널 쉘을 엽니다.

2. 다음 명령을 실행하여 Docker가 설치되어 있는지 확인합니다:

   ```bash
   docker --version
   ```

   Docker가 설치되어 있다면 버전 정보가 표시됩니다.
   예를 들어:

   ```text
   Docker version 20.10.17, build 100c701
   ```

3. 다음 명령을 실행하여 `srtool` 명령줄 인터페이스를 설치합니다:

   ```bash
   cargo install --git https://github.com/chevdor/srtool-cli
   ```

4. 다음 명령을 실행하여 `srtool` 명령줄 인터페이스의 사용법 정보를 확인합니다:

   ```bash
   srtool help
   ```

5. 다음 명령을 실행하여 최신 `srtool` Docker 이미지를 다운로드합니다:

   ```bash
   srtool pull
   ```

## 결정론적인 빌드 시작하기

환경을 준비한 후에는 `srtool` Docker 이미지를 사용하여 WebAssembly 런타임을 컴파일을 시작할 수 있습니다.

런타임을 빌드하려면 다음을 수행하세요:

1. Substrate 개발 환경에서 터미널 쉘을 엽니다.

2. 다음과 유사한 명령을 실행하여 프로젝트의 런타임을 컴파일합니다:

   ```bash
   srtool build --app --package node-template-runtime --runtime-dir runtime
   ```

   - `--package`에 지정하는 이름은 런타임의 `Cargo.toml` 파일에 정의된 이름이어야 합니다.

   - `--runtime-dir`에 지정하는 경로는 런타임의 `Cargo.toml` 파일의 경로여야 합니다.
     런타임의 `Cargo.toml` 파일이 `runtime` 하위 디렉토리에 위치한 경우 (예: runtime/kusama), `--runtime-dir` 명령줄 옵션을 생략할 수 있습니다.

## 워크플로 액션 추가하기

Substrate 기반 프로젝트에 GitHub 저장소를 사용하는 경우, WebAssembly 런타임을 자동으로 컴파일하기 위한 GitHub 워크플로우를 설정할 수 있습니다.

런타임을 빌드하기 위한 워크플로우를 추가하려면 다음을 수행하세요:

1. Substrate 저장소에서 `.github/workflows` 디렉토리를 생성합니다.

1. `.github/workflows` 디렉토리에서 **파일 추가**를 클릭한 다음 **새 파일 생성**을 선택합니다.
1. 이전 단계에서 생성한 파일에 `srtools-actions` 저장소의 `basic.yml` 예제에서 샘플 GitHub 액션을 복사하여 붙여넣습니다.

1. 샘플 액션의 설정을 자신의 체인에 맞게 수정합니다.

   예를 들어, 다음 설정을 수정합니다:

   - 체인의 이름
   - 런타임 패키지의 이름
   - 런타임의 위치

1. Substrate 저장소에서 액션 파일의 이름을 입력합니다.

1. **새 파일 커밋**을 클릭합니다.

## Docker Hub에서 다운로드하기

Substrate 런타임 도구를 사용하려면 Docker 계정과 Docker가 빌드 환경에 설치되어 있어야 합니다.
Docker Hub에 로그인하면 `paritytech/srtool` 컨테이너를 검색하고 Rust 컴파일러 버전과 빌드 스크립트 버전을 식별하는 태그 이름을 가진 해당 이미지를 찾을 수 있습니다.

`srtool-cli` 또는 `srtool-app`을 사용하지 않고 `paritytech/srtool` 컨테이너와 작업하려는 경우 Docker Hub에서 `paritytech/srtool` 컨테이너 이미지를 직접 가져올 수 있습니다.

Docker Hub에서 이미지를 가져오려면 다음을 수행하세요:

1. Docker Hub에 로그인합니다.

2. 검색 필드에 `paritytech/srtool`을 입력하고 Enter 키를 누릅니다.

3. **paritytech/srtool**을 클릭한 다음 **Tags**를 클릭합니다.

4. 가져올 이미지에 대한 명령을 복사합니다.
5. 로컬 컴퓨터의 터미널 쉘을 엽니다.
6. Docker Hub에서 복사한 명령을 붙여넣습니다.

   예를 들어, 다음과 유사한 명령을 실행할 수 있습니다:

   ```bash
   docker pull paritytech/srtool:1.62.0
   ```

   이 명령은 이미지를 다운로드하고 풀어냅니다.

### 이미지의 네이밍 규칙

`srtool` 이미지에는 `latest` 태그가 없는 다른 Docker 이미지와 달리 `latest` 태그가 없습니다.
Docker Hub에서 이미지를 직접 다운로드하는 경우 설치된 Rust 컴파일러 버전과 호환되는 이미지를 선택해야 합니다.
`paritytech/srtool` Docker 이미지의 네이밍 규칙은 이미지에 포함된 코드를 컴파일하는 데 사용된 Rust 컴파일러의 버전을 지정합니다.
또한 컴파일러 버전과 빌드 스크립트 버전을 모두 지정하는 이미지도 있습니다.
예를 들어, `paritytech/srtool:1.62.0-0.9.19`라는 이미지는 `rustc` 컴파일러의 버전 `1.62.0`으로 컴파일된 코드를 빌드 스크립트의 버전 `0.9.19`을 사용하여 컴파일한 것을 나타냅니다.

컴파일러 버전만 지정하는 이미지는 항상 소프트웨어의 최신 버전을 포함합니다.
대부분의 경우, 그것이 사용해야 할 이미지입니다.

### 컴파일러 버전

Docker Hub에서 이미지를 직접 다운로드하는 경우 설치된 Rust 컴파일러 버전과 호환되는 이미지를 선택하기 위해 먼저 사용 중인 Rust 컴파일러 버전을 확인해야 합니다.
사용 중인 Rust 컴파일러 버전을 확인하려면 다음 명령을 실행하여 설치된 버전을 확인할 수 있습니다:

```bash
rustc --version
```