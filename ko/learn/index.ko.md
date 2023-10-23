---
title: 배우기
description: Substrate 기반 블록체인과 Substrate 런타임 개발의 핵심 원칙과 독특한 기능을 설명합니다.
섹션: 문서
keywords:
  - 블록체인
  - 합의
  - Substrate
  - 아키텍처
---

Substrate는 Rust 기반 라이브러리와 도구를 사용하여 모듈식이고 확장 가능한 구성 요소로 애플리케이션별 블록체인을 구축할 수 있도록 하는 소프트웨어 개발 키트(SDK)입니다.
이 섹션에서는 Substrate 개발 환경의 핵심 원칙과 독특한 기능을 많이 설명합니다.

이러한 주제들은 Substrate 기반 블록체인을 구축할 때 가능한 옵션과 Substrate가 특정 프로젝트 요구사항이나 비즈니스 모델에 가장 적합한 블록체인을 구축하는 데 어떻게 도움을 줄 수 있는지를 학습하는 데 도움이 되도록 설계되었습니다.

- [Substrate에 오신 것을 환영합니다](/learn/welcome-to-substrate/)는 Substrate로 개발하는 것이 다른 대부분의 블록체인 및 스마트 컨트랙트 플랫폼에서 제공하지 못하는 주요 이점을 강조합니다.

- [블록체인 기본 개념](/learn/blockchain-basics/)은 블록체인 개발과 관련된 복잡성에 대한 맥락을 제공하고 일반적인 블록체인 개념, 구성 요소 및 용어를 소개합니다.

- [아키텍처와 Rust 라이브러리](/learn/architecture/)는 Substrate 아키텍처와 핵심 Rust 라이브러리 간의 관계를 설명합니다.

- [네트워크와 노드](/learn/networks-and-nodes/)는 Substrate로 구축할 수 있는 다양한 유형의 네트워크 토폴로지와 노드의 역할을 정의합니다.

- [무엇을 구축할 수 있을까요](/learn/what-can-you-build/)는 다양한 개발 옵션의 장점과 한계를 소개하고 어떤 접근 방식을 선택해야 하는지에 대해 설명합니다.

- [런타임 개발](/learn/runtime-development/)은 Substrate 런타임의 중요성을 강조하고 Substrate 런타임 개발에 필요한 핵심 애플리케이션 인터페이스와 기본 요소를 소개합니다.

- [트랜잭션과 블록 기본 개념](/learn/transaction-types/)은 트랜잭션 유형과 블록을 구성하는 구성 요소를 소개합니다.

- [트랜잭션 라이프사이클](/learn/transaction-lifecycle/)은 트랜잭션이 어떻게 받아들여지고 대기열에 추가되며 실행되어 블록에 포함되는지 설명합니다.

- [상태 전이와 저장소](/learn/state-transitions-and-storage/)는 런타임에서 처리되는 상태 변경이 트라이 데이터 구조와 Key-Value 데이터베이스를 사용하여 저장되고 관리되는 방식을 설명합니다.

- [계정, 주소 및 키](/learn/accounts-addresses-keys/)는 계정, 주소 및 키 간의 관계와 사용 방법을 설명합니다.

- [Substrate용 Rust](/learn/rust-basics/)는 Substrate 기반 블록체인을 구축하기 위해 가장 익숙해져야 할 특정 Rust 기능(트레이트, 제네릭, 연관 타입, 매크로 등)을 강조합니다.

- [오프체인 작업](/learn/offchain-operations/)은 일부 작업을 오프체인으로 처리해야 하는 이유와 그 오프체인 작업을 수행하는 대체 방법에 대해 탐구합니다.

- [Substrate Connect에서 라이트 클라이언트](/learn/light-clients-in-substrate-connect/)는 Substrate Connect를 사용하여 라이트 클라이언트를 애플리케이션에 통합하고 어떤 Substrate 기반 체인과도 상호작용할 수 있도록 하는 방법을 설명합니다.

- [암호화](/learn/cryptography)는 Substrate에서 암호화에 사용되는 해시 알고리즘과 서명 체계에 대한 개요를 제공합니다.

- [합의](/learn/consensus/)는 가장 일반적인 합의 모델과 Substrate 블록체인에 구현할 수 있는 합의 유형을 설명합니다.

- [교차 합의 메시징](/learn/xcm-communication/)은 교차 합의 통신과 교차 합의 메시징(XCM) 형식에 대한 개요를 제공합니다.

이 소개 섹션에서 제공된 정보를 소화한 후에는 직접 사용자 정의 블록체인 솔루션을 설계, 구축 및 테스트할 준비가 될 것입니다.