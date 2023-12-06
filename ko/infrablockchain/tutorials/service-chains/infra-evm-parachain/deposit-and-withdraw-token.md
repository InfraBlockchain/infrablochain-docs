---
title: EVM에 자금 입금 및 인출하기
description: 이 튜토리얼은 InfraEVM 체인에서 자금을 입금 및 인출하는 방법을 설명합니다.
keywords:
  - 파라체인
  - EVM
---

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [InfraEVM](../../../service-chains/infra-evm-parachain.md)

## 주소 변환하기

InfraEVM 파라체인은 두 가지의 주소 체계를 사용하고 있습니다.

- SS58 address
- H160 address

SS58 address는 Substrate 레이어에서 사용하고 있으며 H160 address는 EVM 레이어에서 사용하고 있습니다.

### SS58 Address -> H160 Address

![ss58-to-h160](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/ss58-to-h160.png)

SS58 address에서 H160 address로의 주소 변환은 위 사진과 같이 이루어 집니다.

SS58 address를 디코딩 하여 획득한 32바이트의 버퍼 중 앞의 20바이트만 사용하여 H160 address로 사용합니다.

이를 구현해 놓은 자바스크립트 파일이 [레포지토리](https://github.com/InfraBlockchain/infra-evm-parachain)에 포함되어 있습니다.

SS58 address를 H160 address로 변환하는 방법은 아래와 같습니다.

1. 컴퓨터의 터미널 셸을 엽니다.

2. 다음 명령을 실행하여 InfraEVM 파라체인 저장소를 복제합니다:

   ```bash
   git clone https://github.com/InfraBlockchain/infra-evm-parachain
   ```

   이 명령은 `master` 브랜치를 클론합니다.

3. 다음 명령을 실행하여 노드 템플릿 디렉토리의 루트로 이동합니다:

   ```bash
   cd infra-evm-substrate
   ```

   작업을 포함할 새 브랜치를 만듭니다:

   ```bash
   git switch -c my-learning-branch-yyyy-mm-dd
   ```

   `yyyy-mm-dd`를 원하는 식별 정보로 바꾸세요. 숫자로 된 연도-월-일 형식을 권장합니다. 예를 들어:

   ```bash
   git switch -c my-learning-branch-2023-03-01
   ```

4. 다음 명령을 실행하여 주소 변환을 수행합니다:

   ```bash
   node ./utils --evm-address {ss58-address}
   ```

### H160 Address -> SS58 Address

![h160-to-ss58](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/h160-to-ss58.png)

H160 address에서 SS58 address로의 주소 변환은 위 사진과 같이 이루어 집니다.

`"evm:"` 문자열로 구성된 버퍼와 H160 address로 구성된 버퍼를 합쳐 이를 `blake2` 해시함수로 해시값을 획득한 뒤 획득한 해시값을 SS58 형식으로 인코딩하여 SS58 address를 획득할 수 있습니다.

이를 구현해 놓은 자바스크립트 파일이 [레포지토리](https://github.com/InfraBlockchain/infra-evm-parachain)에 포함되어 있습니다.

H160 address를 SS58 address로 변환하는 방법은 아래와 같습니다.

1. 컴퓨터의 터미널 셸을 엽니다.

2. 다음 명령을 실행하여 InfraEVM 파라체인 저장소를 복제합니다:

   ```bash
   git clone https://github.com/InfraBlockchain/infra-evm-parachain
   ```

   이 명령은 `master` 브랜치를 클론합니다.

3. 다음 명령을 실행하여 노드 템플릿 디렉토리의 루트로 이동합니다:

   ```bash
   cd infra-evm-substrate
   ```

   작업을 포함할 새 브랜치를 만듭니다:

   ```bash
   git switch -c my-learning-branch-yyyy-mm-dd
   ```

   `yyyy-mm-dd`를 원하는 식별 정보로 바꾸세요. 숫자로 된 연도-월-일 형식을 권장합니다. 예를 들어:

   ```bash
   git switch -c my-learning-branch-2023-03-01
   ```

4. 다음 명령을 실행하여 주소 변환을 수행합니다:

   ```bash
   node ./utils --ss58-address {evm-address}
   ```

## Substrate에서 EVM으로 자산 이동하기

1. 위에서 설명한 H160 address를 SS58 address로 변경하는 방법을 사용하여 EVM에서 사용할 H160 address와 매핑되어 있는 SS58 address를 획득합니다.

   ```bash
   node ./utils --ss58-address {evm-address}
   ```

2. 1번의 결과로 획득한 SS58 address로 자산을 이동시킵니다.

   ![transfer-asset](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/transfer-asset.png)

   기본적으로 InfraEVM 파라체인에서는 99번 asset을 EVM의 네이티브 토큰으로 사용하도록 연동되어 있습니다.

3. 메타마스크 등의 EVM 월렛에서 반영되었음을 확인합니다.

   ![metamask-balance](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/metamask-balance.png)

## EVM에서 Substrate로 자산 이동하기

1. 위에서 설명한 SS58 address를 H160 address로 변경하는 방법을 사용하여 Substrate에서 사용할 SS58 address와 매핑되어 있는 H160 address를 획득합니다.

   ```bash
   node ./utils --evm-address {ss58-address}
   ```

2. 1번의 결과로 획득한 EVM address로 자산을 이동시킵니다.

   ![transfer-asset](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/metamask-transfer.png)

3. [_인프라 블록체인(InfraBlockchain)_ 익스플로러](https://portal.infrablockspace.net) 에 접속하여 아래 과정을 따릅니다.

- `개발자` - `익스트린식` - `evm` 팔레트의 `withdraw` 익스트린식을 선택합니다.

  아래와 같이 구성하고 익스트린식을 발생시킵니다.

  ![withdraw](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/withdraw.png)

  1번 과정에서 주소 변환할 때 사용하였던 SS58 계정이 익스트린식을 실행시켜야 합니다.

4. 이벤트를 확인하여 정상적으로 자산이 입금되었는지 확인합니다.

   ![withdraw-success](/media/images/docs/infrablockchain/tutorials/service-chains/infra-evm-parachain/withdraw-success.png)

## 다음 단계로 넘어가기

- [ERC20 토큰 컨트랙트 배포하기](./deploy-erc20-contract.md)
