---
title: 빌드
description: Substrate 노드가 구성되고 컴파일되는 방법에 대한 세부 정보를 제공합니다.
keywords:
---

이 섹션에서는 런타임 로직을 구성하는 데 사용되는 코드에 대해 더 자세히 살펴보고, 노드를 구축하고 상호 작용하는 데 사용할 수 있는 라이브러리와 도구, 그리고 Substrate 노드를 구축하기 위해 로직이 컴파일되는 방법을 자세히 살펴봅니다.

- [스마트 컨트랙트](/build/smart-contracts-strategy/)은 Substrate 기반 체인에서 응용 프로그램 개발 방법으로 스마트 컨트랙트를 사용하는 방법을 요약합니다.
- [사용자 정의 팔레트](/build/custom-pallets)은 사용자 정의 팔레트를 구축하기 위한 매크로와 속성을 소개합니다.
- [런타임 스토리지](/build/runtime-storage)는 스토리지 구조와 런타임에 저장된 데이터를 탐색하는 방법에 대해 자세히 설명합니다.
- [트랜잭션, 가중치 및 수수료](/build/tx-weights-fees)는 트랜잭션 실행에 가중치와 수수료의 역할, 그리고 수수료가 어떻게 계산되고 환불되는지에 대한 메커니즘을 설명합니다.
- [팔레트 결합](/build/pallet-coupling)은 팔레트가 런타임에서 얼마나 강하게 또는 약하게 결합될 수 있는지를 설명합니다.
- [이벤트와 오류](/build/events-and-errors)는 런타임에서 이벤트와 오류를 발생시키는 방법을 설명합니다.
- [랜덤성](/build/randomness)은 Substrate 기반 블록체인에서 랜덤성을 포함하는 방법을 제안합니다.
- [특권 호출과 오리진](/build/origins)은 사전 정의된 또는 사용자 정의 오리진을 사용하여 함수 호출의 원천을 식별하는 방법을 설명합니다.
- [원격 프로시저 호출](/build/remote-procedure-calls/)은 원격 프로시저 호출과 RPC 메서드를 사용하여 Substrate 노드와 상호 작용하는 방법을 요약합니다.
- [응용 프로그램 개발](/build/application-development/)은 블록체인에서 실행되는 응용 프로그램을 구축하기 위한 도구로서 메타데이터와 프론트엔드 라이브러리의 역할을 소개합니다.
- [체인 사양](/build/chain-spec)은 체인 사양의 사용에 대해 논의하며, 수정할 수 있는 내용과 수정할 수 없는 내용, 그리고 사용자 정의 체인 사양을 배포하는 방법에 대해 설명합니다.
- [제네시스 구성](/build/genesis-configuration)은 제네시스 구성의 주요 요소에 대해 설명합니다.
- [빌드 프로세스](/build/build-process)는 Rust 코드가 Rust 바이너리와 WebAssembly 타겟으로 컴파일되는 세부 사항과 이 두 가지 타겟이 노드 작업을 최적화하는 데 사용되는 방법에 대해 자세히 설명합니다.
- [결정론적 런타임 빌드](/build/build-a-deterministic-runtime)은 Substrate 기반 체인을 위한 WebAssembly 런타임을 구축하기 위해 Substrate 런타임 도구 (`srtool`)와 Docker를 사용하는 방법을 설명합니다.
- [코드 문제 해결](/build/troubleshoot-your-code)은 문제 해결과 최적의 방법을 따르기 위한 일반적인 코딩 기술과 Substrate에 특화된 코딩 기술을 강조합니다.