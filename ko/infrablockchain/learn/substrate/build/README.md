---
title: 빌드
description: Substrate 노드가 구성되고 컴파일되는 방법에 대한 세부 정보를 제공합니다.
keywords:
---

이 섹션에서는 런타임 로직을 구성하는 데 사용되는 코드에 대해 더 자세히 살펴보고, 노드를 구축하고 상호 작용하는 데 사용할 수 있는 라이브러리와 도구, 그리고 Substrate 노드를 구축하기 위해 로직이 컴파일되는 방법을 자세히 살펴봅니다.

- [빌드할 것을 결정하기](./decide-what-to-build.md) 는 Substrate 이용하여 빌드할 것에 대한 방향성에 대해 설명합니다.
- [빌드 프로세스](./build-process.md)는 Rust 코드가 Rust 바이너리와 WebAssembly 타겟으로 컴파일되는 세부 사항과 이 두 가지 타겟이 노드 작업을 최적화하는 데 사용되는 방법에 대해 자세히 설명합니다.
- [결정론적(Deterministic) 런타임 빌드](./build-a-deterministic-runtime.md)은 Substrate 기반 체인을 위한 WebAssembly 런타임을 구축하기 위해 Substrate 런타임 도구 (`srtool`)와 Docker를 사용하는 방법을 설명합니다.
- [체인 스펙](./chain-spec.md)은 체인 스펙의 사용에 대해 논의하며, 수정할 수 있는 내용과 수정할 수 없는 내용, 그리고 커스텀 체인 스펙을 배포하는 방법에 대해 설명합니다.
- [Genesis 구성](./genesis-configuration.md)은 genesis 구성의 주요 요소에 대해 설명합니다.
- [어플리케이션 개발](./application-development.md)은 블록체인에서 실행되는 어플리케이션을 구축하기 위한 도구로서 메타데이터와 프론트엔드 라이브러리의 역할을 소개합니다.
- [RPC](./remote-procedure-calls.md)은 원격 프로시저 호출과 RPC 메서드를 사용하여 Substrate 노드와 상호 작용하는 방법을 요약합니다.
- [문제 해결](./troubleshoot-your-code.md)은 문제 해결과 최적의 방법을 따르기 위한 일반적인 코딩 기술과 Substrate에 특화된 코딩 기술을 강조합니다.