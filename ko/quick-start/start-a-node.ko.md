---
title: 노드 시작하기
description: 템플릿에서 Substrate 노드를 시작하세요.
keywords:
---

Substrate 튜토리얼과 가이드를 따라하려면 개발 환경에서 Substrate 노드를 빌드하고 실행해야 합니다.
[Substrate 개발자 허브](https://github.com/substrate-developer-hub/)에서는 작업 환경을 빠르게 설정할 수 있도록 여러분을 위한 _템플릿_을 관리하고 있습니다.
예를 들어, [substrate-node-template](https://github.com/substrate-developer-hub/substrate-node-template/tags/)은 주요 Substrate `node-template` 바이너리의 스냅샷으로, 시작하기 위한 핵심 기능을 포함하고 있습니다.

노드를 시작한 후에는 웹 브라우저와 간단한 애플리케이션을 사용하여 미리 정의된 계정의 잔액을 조회할 수 있습니다.

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- 인터넷 연결이 되어 있고 로컬 컴퓨터에서 대화형 셸 터미널에 접근할 수 있어야 합니다.

- 소프트웨어 개발과 명령 줄 인터페이스 사용에 대해 일반적으로 알고 있어야 합니다.

- Rust 컴파일러와 툴체인이 설치되어 있어야 합니다.

  Rust가 설치되어 있는지 확인하려면 `rustup show` 명령을 실행하세요.
  Rust가 설치되어 있다면, 이 명령은 툴체인과 컴파일러의 버전 정보를 표시합니다.
  Rust가 설치되어 있지 않다면, 명령은 어떤 출력도 반환하지 않습니다.
  Rust를 설치하는 방법에 대한 정보는 [설치](/install)를 참조하세요.

## 노드 템플릿 빌드하기

1. 다음 명령을 실행하여 노드 템플릿 저장소를 복제하세요:

   ```sh
   git clone https://github.com/substrate-developer-hub/substrate-node-template
   ```

   이 명령은 `main` 브랜치를 복제합니다.

   원하는 Polkadot 버전과 호환되는 노드를 생성하려면 `--branch` 명령줄 옵션과 [태그](https://github.com/substrate-developer-hub/substrate-node-template/tags)를 사용할 수도 있습니다.

2. 복제한 디렉토리의 루트로 이동하세요:

   ```sh
   cd substrate-node-template
   ```

3. 다음과 유사한 명령을 실행하여 작업을 저장할 새 브랜치를 생성하세요:

   ```bash
   git switch -c my-learning-branch-yyyy-mm-dd
   ```

   브랜치의 이름은 원하는 식별 정보로 지정할 수 있습니다.
   대부분의 경우, 브랜치를 복제한 날짜인 연도-월-일 정보를 이름에 포함하는 것이 좋습니다.
   예를 들어:

   ```text
   git switch -c my-learning-branch-2023-03-31
   ```

4. 노드 템플릿을 컴파일하세요:

   ```sh
   cargo build --package node-template --release
   ```

   노드를 처음으로 컴파일하는 경우, 완료까지 시간이 소요될 수 있습니다.
   컴파일이 완료되면 다음과 같은 줄이 표시됩니다:

   ```bash
   Finished release [optimized] target(s) in 11m 23s
   ```

## 노드 정보 보기

1. 다음 명령을 실행하여 노드를 사용할 준비가 되었는지 확인하고 사용 가능한 명령 줄 옵션에 대한 정보를 확인하세요:

   ```sh
   ./target/release/node-template --help
   ```

   사용법 정보에서는 다음과 같은 명령 줄 옵션을 사용하여:

   - 노드를 시작할 수 있습니다.
   - 계정과 키를 사용할 수 있습니다.
   - 노드 작업을 수정할 수 있습니다.

1. 다음 명령을 실행하여 미리 정의된 `Alice` 개발 계정에 대한 계정 정보를 확인하세요:

   ```sh
   ./target/release/node-template key inspect //Alice
   ```

   이 명령은 다음과 같은 계정과 주소 정보를 표시합니다:

   ```text
   Secret Key URI `//Alice` is account:
   Network ID:        substrate
   Secret seed:       0xe5be9a5092b81bca64be81d212e7f2f9eba183bb7a90954f7b76361f6edb5c0a
   Public key (hex):  0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d
   Account ID:        0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d
   Public key (SS58): 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
   SS58 Address:      5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
   ```

   `Alice`와 `Bob`과 같은 미리 정의된 개발 계정은 **체인 사양** 파일에 구성되어 있습니다.
   [코드 탐색](/quick-start/explore-the-code/)에서 노드 템플릿 파일에 대해 자세히 알아보고, 특히 체인 사양 파일에 대해서는 [체인 사양](/build/chain-spec/)에서 자세히 알아보세요.
   지금은 간단한 잔액 이체와 같은 테스트를 가능하게 하기 위해 개발 계정이 존재하는 것만 알고 계시면 됩니다.

## 블록체인 시작하기

1. 다음 명령을 실행하여 개발 모드에서 노드를 시작하세요:

   ```sh
   ./target/release/node-template --dev
   ```

   개발 모드에서는 체인이 블록을 최종화하기 위해 다른 피어 컴퓨터를 필요로 하지 않습니다.
   노드가 시작되면 터미널에 수행된 작업에 대한 출력이 표시됩니다.
   블록이 제안되고 최종화되는 메시지가 표시된다면 실행 중인 노드가 있습니다.

   ```text
   ... Idle (0 peers), best: #3 (0xcc78…5cb1), finalized #1 ...
   ... Starting consensus session on top of parent ...
   ... Prepared block for proposing at 4 (0 ms) ...
   ```

## 노드에 연결하기

노드가 실행 중이므로 미리 정의된 `Alice` 계정의 잔액을 확인하기 위해 노드에 연결할 수 있습니다.
이 간단한 애플리케이션에서는 JavaScript와 [Polkadot-JS API](https://polkadot.js.org/docs/api)를 사용하여 블록체인과 상호작용하는 `index.html` HTML 파일을 생성할 수 있습니다.

예를 들어, 이 샘플 [index.html](/assets/quickstart/index.html)은 다음과 같은 작업을 수행하는 JavaScript, Polkadot-JS API, HTML을 사용하는 방법을 보여줍니다:

- 입력으로 계정 주소를 받습니다.
- `onClick` 이벤트를 사용하여 계정 잔액을 조회합니다.
- 계정의 잔액을 출력합니다.

노드에 연결하여 계정 잔액을 확인하려면 다음을 수행하세요:

1. **빠른 시작: 잔액 조회** 애플리케이션의 [샘플 코드](/assets/quickstart/index.html)를 코드 편집기에 복사하여 새 파일에 붙여넣고 파일을 로컬 컴퓨터에 저장하세요.

2. 웹 브라우저에서 `index.html` 파일을 엽니다.

3. `Alice` 계정의 SS58 주소를 입력 필드에 복사하여 붙여넣고 **잔액 조회**를 클릭하세요.

## 노드 중지하기

1. 블록체인 작업을 표시하는 터미널로 이동하세요.

1. `control-c` 키 조합을 눌러 로컬 블록체인을 중지하고 모든 상태를 지웁니다.