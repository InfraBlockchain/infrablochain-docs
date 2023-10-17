---
title: EVM 계정에 접근하기
description: Substrate 블록체인 노드를 통해 이더리움 기반 계정과 스마트 계약에 접근하는 방법을 설명합니다.
keywords:
  - 이더리움
  - 계약
  - 솔리디티
  - EVM
  - ethersjs
  - web3js
  - hardhat
  - truffle
  - ganache
---

이 튜토리얼은 [Frontier](https://github.com/paritytech/frontier) 프로젝트의 크레이트를 사용하여 이더리움 기반 계정에 접근하고 솔리디티 기반 스마트 계약을 실행할 수 있는 **이더리움 호환** 블록체인을 구축하는 방법을 설명합니다.
Frontier 프로젝트의 두 가지 주요 목표는 다음과 같습니다:

- 로컬 Substrate 노드를 사용하여 수정하지 않은 상태로 이더리움 탈중앙화 앱을 실행합니다.
- 이더리움 메인 네트워크에서 상태를 가져옵니다.

이 튜토리얼은 작동 환경을 제공하기 위해 미리 정의된 노드 템플릿을 사용합니다.
이 템플릿은 [Frontier 릴리스 가이드](https://github.com/paritytech/frontier/blob/master/docs/node-template-release.md)의 지침을 사용하여 생성되었습니다.

자체적으로 독립적인 템플릿을 생성하려면 [node-template-release.sh](https://github.com/paritytech/frontier/blob/master/.maintain/node-template-release.sh) 템플릿 생성 스크립트를 사용할 수 있습니다.
[frontier](https://github.com/paritytech/frontier) 저장소에서 직접 노드를 빌드하거나 템플릿 생성 스크립트를 사용하여 자체적으로 노드를 빌드하는 경우, Frontier는 자체 버전의 Substrate 크레이트를 사용하므로 `Cargo` 파일의 종속성을 프로젝트의 종속성과 일치하도록 업데이트해야 할 수도 있습니다.

## 시작하기 전에

이 튜토리얼을 시도하기 전에 다음 Substrate 튜토리얼을 완료해야 합니다:

- [로컬 블록체인 구축](/tutorials/build-a-blockchain/build-local-blockchain/)
- [런타임에 팔렛 추가](/tutorials/build-application-logic/add-a-pallet/)
- [사용자 정의 팔렛에서 매크로 사용](/tutorials/build-application-logic/use-macros-in-a-custom-pallet/)

튜토리얼을 통해 다음 작업을 수행하는 방법에 익숙해져야 합니다:

- Substrate 블록체인 노드 실행하기.
- 런타임에서 팔렛 추가, 제거 및 구성하기.
- Polkadot-JS 또는 다른 프론트엔드를 사용하여 노드에 연결하여 트랜잭션 제출하기.

이 튜토리얼을 시작하기 전에 다음 사항에도 익숙해져야 합니다:

- 이더리움 핵심 개념과 용어
- 이더리움 가상 머신 (EVM) 기본 개념
- 탈중앙화 앱과 스마트 계약
- 팔렛 설계 원칙

## 제네시스 구성

`frontier-node-template`의 개발용 [체인 사양](https://github.com/substrate-developer-hub/frontier-node-template/blob/main/node/src/chain_spec.rs)은 `alice` 계정을 위한 EVM 계정이 미리 구성된 제네시스 블록을 정의합니다.
개발 모드에서 이 노드를 시작하면 `alice`의 EVM 계정에 기본 이더 양이 제공됩니다.
이 계정을 사용하여 EVM 계정 세부 정보를 보고 이더리움 스마트 계약을 호출할 수 있습니다.
노드를 시작한 후 [Polkadot-JS 애플리케이션](https://polkadot.js.org/apps/#?rpc=ws://127.0.0.1:9944)을 사용하여 `alice`의 EVM 계정 세부 정보를 확인할 수 있습니다.

## Frontier 노드 컴파일하기

[Frontier 노드 템플릿](https://github.com/substrate-developer-hub/frontier-node-template)은 Substrate에서 바로 개발을 시작할 수 있는 작동하는 개발 환경을 제공합니다.

Frontier 노드 템플릿을 컴파일하려면 다음 단계를 따르세요:

1. 컴퓨터에서 터미널 쉘을 엽니다.

1. 다음 명령을 실행하여 노드 템플릿 저장소를 복제합니다:

   ```bash
   git clone https://github.com/substrate-developer-hub/frontier-node-template.git
   ```

1. 다음 명령을 실행하여 노드 템플릿 디렉토리의 루트로 이동합니다:

   ```bash
   cd frontier-node-template
   ```

1. 다음 명령을 실행하여 노드 템플릿을 컴파일합니다:

   ```bash
   cargo build --release
   ```

## 노드에 연결하기

노드를 컴파일한 후, 미리 구성된 EVM 계정을 탐색하기 위해 노드를 시작해야 합니다.

로컬 Substrate 노드를 시작하려면 다음 단계를 따르세요:

1. 필요한 경우 로컬 컴퓨터에서 터미널 쉘을 엽니다.

1. `frontier-node-template`을 컴파일한 루트 디렉토리로 이동합니다.

1. 다음 명령을 실행하여 개발 모드에서 노드를 시작합니다:

   ```bash
   ./target/release/frontier-template-node --dev
   ```

   `--dev` 명령줄 옵션은 노드가 미리 정의된 `development` 체인 사양을 사용하여 실행되도록 지정합니다. 이 사양에는 `alice`와 테스트용 다른 계정이 포함되어 있습니다.

1. 터미널에 표시된 출력을 확인하여 노드가 성공적으로 실행되었는지 확인합니다.

   터미널에는 다음과 유사한 출력이 표시됩니다:

   ```text
   2022-07-08 10:06:42 Frontier Node
   2022-07-08 10:06:42 ✌️  version 0.0.0-1b6bff4-x86_64-macos
   2022-07-08 10:06:42 ❤️  by Substrate DevHub <https://github.com/substrate-developer-hub>, 2021-2022
   2022-07-08 10:06:42 📋 Chain specification: Development
   2022-07-08 10:06:42 🏷  Node name: flippant-boat-0444
   2022-07-08 10:06:42 👤 Role: AUTHORITY
   ...
   ```

1. [Polkadot-JS 애플리케이션](https://polkadot.js.org/apps/#?rpc=ws://127.0.0.1:9944)을 사용하여 로컬 노드에 연결합니다.

1. **Settings**를 클릭한 다음 **Developer**를 클릭합니다.
  
   ![Developer settings](/media/images/docs/tutorials/evm-ethereum/settings-developer.png)

1. 다음 계정 정보를 정의하여 EVM `Account` 유형을 생성하고 계정이 트랜잭션을 보내고 블록을 검사할 수 있도록 활성화합니다.

   트랜잭션을 보내려면 `Address`와 `LookupSource` 설정을 정의해야 합니다.
   <br>
   블록을 검사하려면 `Transaction`과 `Signature` 설정을 정의해야 합니다.

   ```json
   {
      "Address": "MultiAddress",
      "LookupSource": "MultiAddress",
      "Account": {
         "nonce": "U256",
         "balance": "U256"
      },
      "Transaction": {
         "nonce": "U256",
         "action": "String",
         "gas_price": "u64",
         "gas_limit": "u64",
         "value": "U256",
         "input": "Vec<u8>",
         "signature": "Signature"
      },
      "Signature": {
         "v": "u64",
         "r": "H256",
         "s": "H256"
      }
   }
   ```

1. **Save**를 클릭합니다.

## RPC를 사용하여 잔액 조회하기

EVM 계정에 대한 설정을 구성한 후, Polkadot-JS 애플리케이션을 사용하여 `alice`의 EVM 계정에 대한 정보를 확인할 수 있습니다.

1. 노드가 실행 중이고 [Polkadot-JS 애플리케이션](https://polkadot.js.org/apps/#?rpc=ws://127.0.0.1:9944)이 노드에 연결되어 있는지 확인합니다.

1. **Developer**를 클릭한 다음 **RPC calls**를 선택합니다.

1. **Submission** 탭에서 호출할 엔드포인트로 **eth**를 선택합니다.

1. 호출할 함수 목록에서 **getBalance(address, number)**를 선택합니다.

1. 주소에 `alice` 계정을 위한 EVM 계정 식별자를 지정합니다.
  
   미리 정의된 계정 주소는 `0xd43593c715fdd31c61141abd04a99fd6822c8558`입니다.
   이 주소는 [Substrate EVM 유틸리티](https://github.com/substrate-developer-hub/frontier-node-template/tree/main/utils/README.md#--evm-address-address)를 사용하여 `alice` 계정의 공개 키에서 계산된 주소입니다.

1. **Submit RPC call**을 클릭합니다.
  
   호출은 다음과 유사한 출력을 반환해야 합니다:
  
   ```text
   2: eth.getBalance: U256
   340,282,366,920,938,463,463,374,607,431,768,210,955
   ```

## 스마트 계약 배포하기

이더리움 주소의 잔액을 조회하는 방법을 확인했으므로, 이더리움 스마트 계약을 배포하고 호출하여 관련 기능을 테스트해볼 수 있습니다.
이 튜토리얼에서는 [Truffle](https://www.trufflesuite.com/truffle) 샘플 계약을 사용하여 [ERC-20 토큰](https://github.com/substrate-developer-hub/frontier-node-template/blob/main/examples/contract-erc20/truffle/contracts/MyToken.sol)을 정의하는 계약을 배포합니다.
Polkadot JS SDK와 [Typescript](https://github.com/substrate-developer-hub/frontier-node-template/tree/main/examples/contract-erc20)를 사용하여 ERC-20 토큰 계약을 생성할 수도 있습니다.

1. ERC-20 계약을 생성합니다.
  
   편의를 위해 [MyToken.json](https://github.com/substrate-developer-hub/frontier-node-template/blob/main/examples/contract-erc20/truffle/contracts/MyToken.json)의 컴파일된 `bytecode`를 사용하여 Substrate 블록체인에 계약을 배포합니다.

1. 노드가 실행 중이고 [Polkadot-JS 애플리케이션](https://polkadot.js.org/apps/#?rpc=ws://127.0.0.1:9944)이 노드에 연결되어 있는지 확인합니다.

1. **Developer**를 클릭한 다음 **Extrinsics**를 선택합니다.

1. **ALICE** 개발 계정을 트랜잭션을 제출하는 데 사용할 계정으로 선택합니다.

1. **evm**을 선택합니다.

1. **create** 함수를 선택합니다.

1. 함수에 대한 매개변수를 구성합니다.

   | 이에 대해 | 이렇게 지정
   | -------- | ------------
   | `source` | 0xd43593c715fdd31c61141abd04a99fd6822c8558
   | `init` | `MyToken.json`의 raw `bytecode` 16진수 값
   | `value` | 0
   | `gasLimit` | 4294967295
   | `maxFeePerGas` | 100000000

   선택적인 매개변수는 비워둘 수 있습니다.
   `nonce`의 값은 `0x0`부터 시작하여 소스 계정의 알려진 nonce를 증가시킵니다.
   선택한 함수에 따라 사용하지 않는 매개변수를 제거해야 할 수도 있습니다.

1. **Submit Transaction**을 클릭합니다.

1. 트랜잭션을 승인하기 위해 **Sign and Submit**을 클릭합니다.

## 스마트 계약 보기

트랜잭션을 제출한 후, 계약이 네트워크에 배포되고 [Polkadot-JS 애플리케이션](https://polkadot.js.org/apps/#?rpc=ws://127.0.0.1:9944)을 사용하여 계약에 대한 정보를 확인할 수 있습니다.

1. 노드가 실행 중이고 [Polkadot-JS 애플리케이션](https://polkadot.js.org/apps/#?rpc=ws://127.0.0.1:9944)이 노드에 연결되어 있는지 확인합니다.

1. **Network**를 클릭한 다음 **Explorer**를 선택합니다.

1. **evm.Created** 이벤트를 클릭하여 새로 생성된 계약의 주소가 `0x8a50db1e0f9452cfd91be8dc004ceb11cb08832f`인지 확인합니다.

   ![Successful contract created event](/media/images/docs/tutorials/evm-ethereum/evm-created.png)

   브라우저의 개발자 도구 콘솔에서도 트랜잭션에 대한 세부 정보를 확인할 수 있습니다.

   EVM 계약 주소는 계약 생성자의 계정 식별자와 nonce에 따라 결정되므로, 계약이 배포된 주소는 잘 알려진 `alice` 계정의 계정 식별자 `0xd43593c715fdd31c61141abd04a99fd6822c8558`와 nonce `0x0`을 사용하여 계산됩니다.

1. **Developer**를 클릭한 다음 **Chain State**를 선택합니다.

1. 상태로 **evm**을 선택하고 **accountCodes**를 선택합니다.

1. `alice` 계정을 위한 계정 식별자 `0xd43593c715fdd31c61141abd04a99fd6822c8558`를 지정하고 계정 코드가 비어 있는지 확인합니다 (`0x`).

1. `alice` 개발 계정을 사용하여 배포한 계약 주소 `0x8a50db1e0f9452cfd91be8dc004ceb11cb08832f`를 지정하고 계약 계정 코드가 Solidity 계약의 바이트코드인지 확인합니다.

## 계정 스토리지 보기

배포한 ERC-20 계약은 [OpenZeppelin ERC-20 구현](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol)을 기반으로 합니다.
이 계약에는 최대 토큰 수를 생성하고 계약 생성자와 연결된 계정에 저장하는 생성자가 포함되어 있습니다.

스마트 계약과 관련된 계정 스토리지를 조회하려면 다음 단계를 따르세요:

1. 상태로 **evm**을 선택하고 **accountStorages**를 선택합니다.

1. 첫 번째 매개변수로 ERC-20 계약 주소 `0x8a50db1e0f9452cfd91be8dc004ceb11cb08832f`를 지정합니다.

1. 두 번째 매개변수로 읽을 스토리지 슬롯을 지정합니다. `0x045c0350b9cf0df39c4b40400c965118df2dca5ce0fbcf0de4aafc099aea4a14`입니다.

   슬롯은 슬롯 0과 `alice` 계정의 계정 식별자 `0xd43593c715fdd31c61141abd04a99fd6822c8558`을 기반으로 [Substrate EVM 유틸리티](https://github.com/substrate-developer-hub/frontier-node-template/tree/main/utils/README.md#--erc20-slot-slot-address)를 사용하여 계산된 것입니다.

   반환된 값은 `0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff`입니다.

   계약을 배포한 후 `alice` 계정의 잔액을 확인하면 계정에서 수수료가 인출되었으며 `getBalance(address, number)` 호출은 다음과 유사한 값을 반환합니다:

   ```text
   340,282,366,920,938,463,463,374,603,530,233,757,803
   ```

## 토큰 전송하기

지금까지 `alice` 개발 계정만 사용했습니다.
다음 단계는 배포한 계약을 사용하여 토큰을 다른 계정으로 전송하는 것입니다.

1. 노드가 실행 중이고 [Polkadot-JS 애플리케이션](https://polkadot.js.org/apps/#?rpc=ws://127.0.0.1:9944)이 노드에 연결되어 있는지 확인합니다.

1. **Developer**를 클릭한 다음 **Extrinsics**를 선택합니다.

1. **ALICE** 개발 계정을 트랜잭션을 제출하는 데 사용할 계정으로 선택합니다.

1. **evm**을 선택합니다.

1. ERC-20 계약에서 `transfer(address, uint256)` 함수를 호출하기 위해 **call**을 선택합니다.

1. 함수에 대한 매개변수를 구성합니다.

   | 이에 대해 | 이렇게 지정
   | -------- | ------------
   | `source` | 0xd43593c715fdd31c61141abd04a99fd6822c8558
   | `target` | 0x8a50db1e0f9452cfd91be8dc004ceb11cb08832f
   | `input` | 0xa9059cbb0000000000000000000000008eaf04151687736326c9fea17e25fc528761369300000000000000000000000000000000000000000000000000000000000000dd
   | `value` | 0
   | `gasLimit` | 4294967295
   | `maxFeePerGas` | 100000000

   `source`는 토큰을 보유한 계정을 나타냅니다.
   이 경우 `source`는 계약 생성자인 `alice`의 EVM 계정입니다.
   `target`은 `alice`에서 `bob`으로 토큰을 전송하기 위한 계약 주소입니다.

   `input` 매개변수는 전송을 수행하는 EVM ABI로 인코딩된 함수 호출을 지정합니다. (`0xa9059cbb`) 및 함수에서 필요한 인수입니다.
   이 함수의 인수는 `bob` EVM 계정 식별자 (`0x8eaf04151687736326c9fea17e25fc5287613693`)와 전송할 토큰 수 (221 또는 16진수로 `0xdd`)입니다.

   이 튜토리얼에서는 [Remix 웹 IDE](http://remix.ethereum.org)를 사용하여 입력 값을 계산했습니다.

1. **Submit Transaction**을 클릭합니다.

1. 트랜잭션을 승인하기 위해 **Sign and Submit**을 클릭합니다.

## 토큰 전송 확인하기

트랜잭션을 제출한 후, [Polkadot-JS 애플리케이션](https://polkadot.js.org/apps/#?rpc=ws://127.0.0.1:9944)을 사용하여 토큰 전송을 확인할 수 있습니다.

1. 노드가 실행 중이고 [Polkadot-JS 애플리케이션](https://polkadot.js.org/apps/#?rpc=ws://127.0.0.1:9944)이 노드에 연결되어 있는지 확인합니다.

1. **Network**를 클릭한 다음 **Explorer**를 선택합니다.

1. **evm.Executed** 이벤트를 클릭하여 실행된 계약의 주소가 `0x8a50db1e0f9452cfd91be8dc004ceb11cb08832f`인지 확인합니다.

   ![Successful execution event](/media/images/docs/tutorials/evm-ethereum/evm-executed.png)

1. **Developer**를 클릭한 다음 **Chain State**를 선택합니다.

1. 상태로 **evm**을 선택하고 **accountStorages**를 선택합니다.

1. `alice` 개발 계정을 사용하여 배포한 계약 주소 `0x8a50db1e0f9452cfd91be8dc004ceb11cb08832f`의 스토리지를 확인하고 스토리지 슬롯 `0x045c0350b9cf0df39c4b40400c965118df2dca5ce0fbcf0de4aafc099aea4a14`를 확인합니다.

   ```text
   0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff22
   ```
  
   계약을 배포한 후 `alice` 계정의 잔액을 확인하면 계정에서 수수료가 인출되었으며 `getBalance(address, number)` 호출은 다음과 유사한 값을 반환합니다:
  
   ```text
   340,282,366,920,938,463,463,374,603,530,233,366,411
   ```

다음 단계로 넘어가기

- [Moonbeam: 이더리움 호환성](https://docs.moonbeam.network/learn/features/eth-compatibility/)
- [이더리움 통합](/tutorials/integrate-with-tools/evm-integration/)
- [EVM ABI 사양](https://solidity.readthedocs.io/en/latest/abi-spec.html)
- [ERC-20 토큰 표준](https://ethereum.org/en/developers/docs/standards/tokens/erc-20/)