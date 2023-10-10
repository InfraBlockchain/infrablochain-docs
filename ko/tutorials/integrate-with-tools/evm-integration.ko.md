---
title: 이더리움 통합
description:
keywords:
---

Frontier 프로젝트의 크레이트를 사용하고 EVM과 이더리움 팔레트를 런타임에 추가함으로써, Solidity 기반 스마트 컨트랙트의 실행을 지원하고 이더리움 기반 계정을 갖춘 Substrate 기반 블록체인을 구축할 수 있습니다.

이더리움 가상 머신(EVM)은 이더리움 네트워크 참여자가 데이터를 저장하고 데이터의 상태에 대해 합의할 수 있도록 하는 구성 요소를 갖춘 가상 컴퓨터입니다.
Substrate 기반 블록체인에서 EVM의 핵심 역할은 **EVM 팔레트**에 구현되어 있습니다.
EVM 팔레트는 Solidity와 같은 고급 언어로 작성된 스마트 컨트랙트를 위해 EVM 바이트코드를 실행하는 역할을 담당합니다.
다음 다이어그램은 EVM 팔레트와 이더리움 RPC 호출이 Substrate 런타임에 통합되는 방식을 간략히 보여줍니다.

![이더리움 호환 런타임 아키텍처](/media/images/docs/tutorials/evm-ethereum/pallet-evm.png)

EVM 팔레트 외에도, 이더리움 팔레트는 이더리움 형식의 블록, 트랜잭션 영수증 및 트랜잭션 상태를 저장하는 역할을 담당합니다.
사용자가 원시(raw) 이더리움 트랜잭션을 제출하면, 런타임의 `pallet_ethereum`에서 `transact` 함수를 호출하여 트랜잭션을 Substrate 트랜잭션으로 변환합니다.

![이더리움 팔레트](/media/images/docs/tutorials/evm-ethereum/pallet-ethereum.png)

이더리움 계정과 Substrate 계정은 단일 개인 키를 사용하여 직접 호환되지 않습니다.
이더리움 계정과 키를 Substrate 계정과 키에 매핑하는 방법에 대한 정보는 Moonbeam 문서의 [통합 계정](https://docs.moonbeam.network/learn/unified-accounts/#substrate-evm-compatible-blockchain)을 참조하십시오.

## 이더리움 특정 런타임 API 및 RPC

런타임은 쿼리할 수 있는 모든 이더리움 형식의 정보를 저장합니다.
노드 RPC 서버 및 런타임 API 및 RPC 클라이언트 호출을 사용하여 해당 정보를 런타임에 호출하고 검색할 수 있습니다.

![이더리움 형식의 정보에 접근하기 위한 원격 프로시저 호출](/media/images/docs/tutorials/evm-ethereum/rpc.png)

## Frontier 블록 가져오기

![블록 가져오기 과정](/media/images/docs/tutorials/evm-ethereum/block-import.png)

## 다음으로 어디로 가야 할까요

- [Moonbeam: 이더리움 호환성](https://docs.moonbeam.network/learn/features/eth-compatibility/)
- [이더리움 가상 머신 (EVM)](https://ethereum.org/en/developers/docs/evm/)
- [EVM 계정에 접근하기](/tutorials/integrate-with-tools/access-evm-accounts/)
- [Substrate EVM 유틸리티](https://github.com/paritytech/frontier/blob/master/template/utils/README.md#substrate-evm-utilities)
- [Frontier rpc 및 rpc-core](https://github.com/paritytech/frontier/tree/master/client/)
- [Frontier 합의](https://github.com/paritytech/frontier/tree/master/primitives/consensus)