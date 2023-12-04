---
title: 개발자 도구
description:
keywords:
---

Rust로 코드를 작성할 예정이므로 개발 환경에 Rust와 함께 작업하기 위한 적절한 확장 프로그램과 플러그인이 설치되어 있는지 확인하세요.
Substrate 노드 템플릿을 사용하면 런타임 개발을 위해 특별히 설계된 핵심 기능과 도구가 포함되어 있는 것을 알 수 있습니다.
또한 개발 환경을 보완하거나 특정 작업을 처리하기 위해 설치할 수 있는 다양한 도구도 많이 있습니다.

Substrate 기반 블록체인 개발을 시작할 때 유용한 몇 가지 도구는 다음과 같습니다:

- [Polkadot-JS API](https://polkadot.js.org/docs/api)

  Polkadot-JS API는 JavaScript를 사용하여 Substrate 기반 체인을 쿼리하고 상호작용할 수 있는 라이브러리를 제공합니다.
  `@polkadot/api` 패키지를 JavaScript 또는 TypeScript 작업 환경에 추가할 수 있습니다.

  API에서 구성하는 대부분의 인터페이스는 실행 중인 노드에 연결하여 동적으로 생성됩니다.
  노드의 구성에 따라 인터페이스가 결정되므로 API를 사용하여 다양한 기능을 구현한 커스텀 체인과 작업할 수 있습니다.
  API를 사용하려면 연결할 체인의 URL을 식별해야 합니다.
  체인의 노드에 연결한 후, API는 해당 체인에 대한 정보와 기능을 수집하고, 그 정보를 기반으로 API에 메서드를 추가합니다.

- [프론트엔드 템플릿](https://github.com/substrate-developer-hub/substrate-front-end-template)
  Substrate 프론트엔드 템플릿은 최소한의 구성으로 Substrate 노드 백엔드에 연결할 수 있는 미리 구현된 프론트엔드 애플리케이션을 제공합니다.
  이 템플릿을 사용하면 커스텀 사용자 인터페이스를 구축하지 않고도 Substrate 노드의 기본 기능을 실험할 수 있습니다.
  이 템플릿은 Create React App 프로젝트와 Polkadot-JS API를 사용하여 구축되었습니다.

- [트랜잭션 제출 명령줄(command-line) 인터페이스](https://github.com/paritytech/subxt)

  `subxt-cli`는 실행 중인 노드에 연결하여 Substrate 기반 체인의 완전한 구성 정보인 [메타데이터](/reference/glossary/#metadata)를 다운로드하는 데 사용할 수 있는 명령줄 프로그램입니다.
  Polkadot-JS API와 유사하게, `subxt-cli` 프로그램으로 다운로드할 수 있는 메타데이터는 해당 체인과 상호작용할 수 있는 정보를 제공합니다.
  `subxt-cli` 프로그램을 사용하여 체인에 대한 정보를 사람이 읽을 수 있는 형식으로 표시할 수도 있습니다.

- [sidecar](https://github.com/paritytech/substrate-api-sidecar)

  @substrate/api-sidecar 패키지는 [FRAME](../../learn/basic/glossary.md#팔렛) 개발 프레임워크를 사용하여 구축된 Substrate 노드에 연결하고 상호작용할 수 있는 RESTful 서비스입니다.
  서비스가 지원하는 엔드포인트에 대한 정보는 [Substrate API Sidecar](https://paritytech.github.io/substrate-api-sidecar/dist/)를 참조하세요.

또한 [Awesome Substrate](https://github.com/substrate-developer-hub/awesome-substrate)에 나열된 리소스와 커뮤니티 프로젝트를 탐색해 볼 수도 있습니다.

가장 일반적으로 사용되는 도구에 대한 개요는 [command-line tools](../../learn/command-line-tools/README.md)를 참조하세요.

## 다음으로 어디로 가야 할까요

- [Command-line tools](../../learn/command-line-tools/README.md)
- [node-template](../../learn/command-line-tools/node-template.md)
- [subkey](../../learn/command-line-tools/subkey.md)
- [try-runtime](../../learn/command-line-tools/try-runtime.md)