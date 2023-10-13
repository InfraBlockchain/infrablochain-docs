---
title: Rust 툴체인
description:
keywords:
---

<!-- TODO 하드웨어 사양이 낮은 경우 빌드 시간에 대한 기대 관리. 실제 개발 환경 요구 사항은 노드 실행에만 필요한 것보다 높음을 정의 -->

Rust는 복잡한 시스템을 구축하기 위한 풍부한 기능을 제공하는 현대적이고 안정적이며 성능 우수한 프로그래밍 언어입니다.
이 언어는 또한 활발한 개발자 커뮤니티와 **크레이트**라고 불리는 공유 라이브러리의 성장하는 생태계를 가지고 있습니다.

## Rust 배우기

Rust는 Substrate 기반 블록체인을 구축하는 데 사용되는 핵심 언어이므로, Substrate 개발을 하려면 Rust 프로그래밍 언어, 컴파일러 및 툴체인 관리에 익숙해져야 합니다.

Rust를 시작하는 경우, [The Rust Programming Language](https://doc.rust-lang.org/book/)을 즐겨찾기에 추가하고 Rust 웹사이트의 다른 [Learn Rust](https://www.rust-lang.org/learn) 자료를 참조합니다.
그러나 개발 환경을 준비하는 동안 알아두어야 할 몇 가지 중요한 사항이 있습니다.

## Rust 툴체인에 대해

Rust **툴체인**의 핵심 도구는 `rustc` 컴파일러, `cargo` 빌드 및 패키지 매니저, 그리고 `rustup` 툴체인 관리자입니다.
특정 시점에는 여러 버전의 Rust가 사용 가능합니다.
예를 들어, Stable 버전, Beta 버전 및 Nightly 빌드를 위한 릴리스 채널이 있습니다.
`rustup` 프로그램을 사용하여 환경에서 사용 가능한 빌드 및 툴체인 프로그램의 버전을 관리합니다.

`rustc` 컴파일러를 사용하여 다양한 아키텍처에 대한 바이너리 파일을 빌드할 수 있습니다. 이를 **Target**이라고 합니다.
target은 컴파일러가 생성해야 하는 출력 종류를 지정하는 문자열 입니다.
이 기능은 Substrate가 네이티브 Rust 바이너리 파일과 WebAssembly target으로 컴파일되기 때문에 중요합니다.

WebAssembly는 모든 최신 컴퓨터 하드웨어에서 실행되고 인터넷에 접속하는 모든 브라우저에서 실행될 수 있는 바이너리 형식입니다.
WebAssembly (Wasm) 타겟을 사용하면 Substrate가 블록체인 런타임을 생성할 수 있습니다.
이러한 바이너리 파일이 어떻게 사용되는지에 대한 자세한 정보는 [빌드 프로세스](/build/build-process/)를 참조하십시오.