---
title: 스마트 계약 개발하기
description: ink! 프로그래밍 언어를 사용하여 Substrate 기반 네트워크에 스마트 계약을 생성하고 배포하는 방법을 안내합니다.
keywords:
  - 스마트 계약
---

**스마트 계약 개발하기** 튜토리얼은 [ink! 프로그래밍 언어](https://use.ink)를 사용하여 Substrate 기반 블록체인에서 실행되는 스마트 계약을 구축하는 방법을 안내합니다.

이 섹션의 튜토리얼은 사전 구성된 [`substrate-contracts-node`](https://github.com/paritytech/substrate-contracts-node)를 사용하여 스마트 계약 작성의 기본 사항에 집중할 수 있도록 도와줍니다.

만약 [표준 노드 템플릿](https://github.com/substrate-developer-hub/substrate-node-template)을 사용한다면, [Contracts pallet](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/contracts)을 수동으로 추가하고 개발 환경을 변경해야 합니다.
두 노드의 런타임 코드를 비교하여 두 노드 간의 차이점에 대한 추가 정보를 확인할 수 있습니다.

- [첫 번째 계약 준비하기](/tutorials/smart-contracts/prepare-your-first-contract/)는 ink! 프로그래밍 언어를 사용하여 개발 환경을 업데이트하고 스마트 계약 프로젝트를 생성하는 방법을 설명합니다.
- [스마트 계약 개발하기](/tutorials/smart-contracts/develop-a-smart-contract/)은 스마트 계약을 사용하여 간단한 값들을 저장, 증가, 검색하는 방법을 보여줍니다.
- [값 저장을 위한 맵 사용하기](/tutorials/smart-contracts/use-maps-for-storing-values/)은 이전 튜토리얼을 확장하여 맵을 사용하여 스마트 계약에서 값들을 저장하고 검색하는 방법을 설명합니다.
- [토큰 계약 만들기](/tutorials/smart-contracts/build-a-token-contract/)는 ERC-20 토큰을 전송하기 위한 간단한 스마트 계약을 만드는 방법을 보여줍니다.
- [스마트 계약 문제 해결하기](/tutorials/smart-contracts/troubleshoot-smart-contracts/)은 스마트 계약 작성 및 배포 시 발생할 수 있는 몇 가지 일반적인 문제들에 대해 설명합니다.