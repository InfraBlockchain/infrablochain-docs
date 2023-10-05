---
title: 작업 환경 준비하기
description:
tutorial: 1
---

이 워크샵에서 최대한의 효과를 얻으려면 작업 개발 환경을 갖추어야 합니다.
필요한 모든 것을 갖고 있는지 확인하기 위해 사전 점검 목록을 검토하세요.
누락된 항목을 발견하면 해당 링크를 클릭하세요.

[Substrate Playground](https://substrate.io/developers/playground/)을 사용하면 로컬 작업 환경을 설정하지 않고도 이 워크샵을 완료할 수 있습니다.
그러나 Substrate Playground을 사용하면 세션을 종료할 때 변경 사항이 저장되지 않습니다.
Substrate Playground을 사용하는 경우 이 체크리스트를 건너뛰고 [Get oriented](/tutorials/collectibles-workshop/02-orientation/)로 바로 이동할 수 있지만, 편집한 파일의 사본을 저장하는 것을 잊지 마세요!

이 워크샵의 지침은 로컬 환경에서 작업하는 것을 가정합니다.

## 지원되는 운영 체제

로컬 개발 환경을 설정하려면 다음 중 하나의 지원되는 운영 체제가 필요합니다:

- [ ] 리눅스 배포판
- [ ] macOS
- [ ] Windows Subsystem for Linux

## Rust 프로그래밍 언어와 도구 체인

Substrate은 현대적인 타입 안정성 프로그래밍 언어인 Rust로 개발되었습니다.
Rust 컴파일러는 코드에 오류가 들어가는 가능성을 최소화하고 대부분의 운영 체제와 WebAssembly 대상에서 실행되는 이진 파일을 생성합니다.

- [ ] Rust가 로컬에 설치되어 있거나 Substrate Playground을 통해 브라우저에서 사용 가능합니다.

  확실하지 않은 경우 터미널을 열고 `rustup show`를 실행하세요.
  컴퓨터에 Rust가 설치되어 있지 않은 경우, [Install](/install)에서 운영 체제에 맞는 지침을 따르세요.
  설치 지침의 마지막 단계는 기본 노드 템플릿이 컴파일되는지 확인하는 것입니다.

- [ ] Substrate 노드가 로컬에서 컴파일되거나 Substrate Playground을 통해 브라우저에서 사용 가능합니다.

Rust에 익숙하지 않은 경우, 이 워크샵은 Rust를 _배우는 것_에 대한 것이 아님을 염두에 두세요.
그러나 몇 가지 중요한 개념에 대한 간단한 소개를 보려면 [Detour: Learn Rust for Substrate](/tutorials/collectibles-workshop/detours/learn-rust/)를 참조하세요.

## 코드 편집기

파일을 수정하기 위해 편집기가 필요합니다.
이상적으로는 구문 강조, 자동 코드 완성 및 디버깅 기능을 제공하는 통합 개발 환경(IDE)을 선택해야 합니다.
선호하는 IDE가 없는 경우 Visual Studio Code를 추천합니다.

- [ ] [Visual Studio Code](https://code.visualstudio.com/download)

<!--다른 일반적인 코드 편집기는 다음과 같습니다:

- [Sublime Text](https://www.sublimetext.com/)
- [Vim](https://www.vim.org/)
- [Atom](https://atom.io/)
-->

## 브라우저

Substrate 블록체인과 상호작용하고 Substrate 컬렉터블 애플리케이션을 구축하는 동안 작업물을 테스트하기 위해 Substrate 노드에 연결할 수 있는 브라우저 기반 애플리케이션이 필요합니다.
이 워크샵에서는 Chrome 또는 Chromium 기반 브라우저를 사용하여 [Polkadot/Substrate Portal](https://polkadot.js.org/apps/)에서 노드에 연결할 수 있습니다.

- [ ] [Google Chrome](https://www.google.com/chrome/) 또는 [Brave](https://brave.com/download/), [Microsoft Edge](https://www.microsoft.com/en-us/edge?form=MA13FJ&exp=e00), [Opera](https://www.opera.com/download), [Vivaldi](https://vivaldi.com/download/)와 같은 Chromium 기반 브라우저.

보안 또는 개인 정보 보호를 위해 Firefox와 같은 보다 제한적인 브라우저를 사용하는 경우, Polkadot/Substrate Portal과 노드 간의 연결이 차단될 수 있습니다.

브라우저가 연결을 차단하는 경우 [polkadot-js/apps](https://github.com/polkadot-js/apps) 저장소를 복제하고 로컬에서 실행하세요.
Polkadot/Substrate Portal을 로컬에서 실행하려면 [Detour: Set up Polkadot/Substrate Portal](/tutorials/collectibles-workshop/detours/set-up-app-locally/)를 참조하세요.

## 프론트엔드 라이브러리

Substrate 컬렉터블을 사용자에게 제공하기 위해 최소한 기본적인 사용자 인터페이스를 구축할 도구가 필요합니다.

- [ ] [Node.js](https://nodejs.org/en/download/)와 패키지 관리자 `npm`
- [ ] [Yarn 패키지 관리자](https://yarnpkg.com/)
- [ ] [TypeScript](https://www.typescriptlang.org/)
- [ ] React, Vue, Bootstrap, Angular와 같은 기본적인 UI/UX 프레임워크.

프론트엔드 라이브러리를 선택하는 데 도움이 필요한 경우 [Detour: Select front-end tools](/tutorials/collectibles-workshop/detours/select-ui-tools/)를 참조하세요.

## 노드 템플릿

이 워크샵을 완료하려면 Substrate 노드에 액세스할 수 있어야 합니다.
로컬 개발 환경을 설정하는 경우, [Quick start](/quickstart/)에서 설명하는 대로 개발자 허브 [substrate-node-template](https://github.com/substrate-developer-hub/substrate-node-template/tags/)을 다운로드하고 컴파일할 수 있습니다.

`substrate-node-template` 저장소는 주요 Substrate `node-template` 이진 파일의 스냅샷을 제공하며, 기능적인 노드와 핵심 기능 세트로 시작하는 데 필요한 모든 것을 포함합니다.