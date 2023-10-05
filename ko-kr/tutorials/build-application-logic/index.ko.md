---
title: 애플리케이션 로직 구축
description: Substrate 런타임 환경을 사용자 정의하기 위해 팔레트를 추가하는 방법을 보여줍니다.
keywords:
---

**애플리케이션 로직 구축** 튜토리얼은 팔레트를 사용하여 런타임을 사용자 정의하는 방법에 중점을 둡니다. 이 튜토리얼에서는 런타임에 간단한 팔레트와 복잡한 팔레트를 추가하는 방법, 그리고 팔레트를 스마트 계약과 함께 사용하는 방법을 배울 수 있습니다.
다음과 같은 내용을 배우게 됩니다:

- [런타임에 팔레트 추가하기](/tutorials/build-application-logic/add-a-pallet/)는 노드 템플릿 런타임에 간단한 미리 정의된 팔레트를 추가하는 공통 단계를 소개합니다.
- [매크로를 사용한 사용자 정의 팔레트](/tutorials/build-application-logic/use-macros-in-a-custom-pallet)는 매크로를 사용하여 사용자 정의 팔레트를 만드는 방법을 설명합니다.
- [호출의 원본 지정하기](/tutorials/build-application-logic/specify-the-origin-for-a-call)는 함수 호출의 원본으로 사용할 계정을 지정하는 방법을 보여줍니다.
- [오프체인 워커 추가하기](/tutorials/build-application-logic/add-offchain-workers/)는 팔레트를 수정하여 오프체인 워커를 포함시키고, 오프체인 워커가 체인 상태를 업데이트하는 트랜잭션을 제출할 수 있도록 팔레트와 런타임을 구성하는 방법을 설명합니다.
- [사용자 정의 팔레트 게시하기](/tutorials/build-application-logic/publish-custom-pallets/)는 사용자 정의 팔레트와 크레이트를 게시하여 커뮤니티에서 사용할 수 있도록 하는 방법을 설명합니다.

블록체인의 스마트 계약 개발을 실험하려면 표준 노드 템플릿 대신 미리 구성된 [substrate-contracts-node](https://github.com/paritytech/substrate-contracts-node)를 사용해야 합니다.
스마트 계약을 지원하는 팔레트와 현재의 노드 템플릿 간에 호환성 문제가 있습니다.
이러한 호환성 문제를 해결하려면 표준 노드 템플릿의 구성을 포함하여 모든 크레이트의 이전 버전을 사용하고 여러 파일을 수정해야 합니다.
스마트 계약 개발에 바로 뛰어들기 위해 [스마트 계약 개발](/tutorials/smart-contracts/) 튜토리얼을 참조하십시오.

<!--
- [계약 팔레트 구성하기](/tutorials/build-application-logic/contracts-pallet/)는 스마트 계약과 함께 작동하도록 복잡한 팔레트를 구성하는 방법을 보여줍니다.

-->