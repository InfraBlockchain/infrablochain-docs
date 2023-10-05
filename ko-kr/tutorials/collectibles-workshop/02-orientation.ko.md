---
title: 방향 잡기
description:
tutorial: 1
code: |

answer: |
---

[작업 환경 준비](/tutorials/collectibles-workshop/01-prepare/)에서 체크리스트를 완료했다면 Substrate 노드를 성공적으로 컴파일했습니다.
이 특정 노드인 substrate-node-template은 몇 가지 일반적인 모듈로 사전 구성되어 있어 시작하기 위한 간단한 작업 환경을 제공합니다.

## 노드 템플릿에 대해

노드 템플릿에는 P2P 네트워킹, 블록 생성, 합의, 최종 블록에 대한 블록 가져오기와 같은 기본적인 블록체인 요소가 포함되어 있습니다.
노드 템플릿에는 또한 계정 및 트랜잭션 작업과 관리 작업 수행을 위한 몇 가지 기본 기능도 포함되어 있습니다.
이러한 핵심 기능 세트는 특정 기능을 구현하는 미리 정의된 모듈인 팔렛(pallets)으로 제공됩니다.

다음과 같은 핵심 모듈이 노드 템플릿에 미리 정의되어 있습니다:

- `pallet_balances`: 계정 자산 및 계정 간 이체 관리.
- `pallet_transaction_payment`: 수행된 트랜잭션에 대한 트랜잭션 수수료 관리.
- `pallet_sudo`: 관리 권한이 필요한 작업 수행.

노드 템플릿은 또한 사용자 정의 팔렛에서 기능을 구현하는 방법을 보여주는 시작용 `pallet_template`도 제공합니다.
이 워크샵에서 할 작업은 `pallet_template`에서 볼 수 있는 것과 유사하지만, 구축 중인 애플리케이션에 특화된 로직을 구현할 것입니다.

## 작업 공간 이름 바꾸기

기본 Substrate 노드 템플릿과 노드를 사용자 정의하기 위해 작업 디렉토리의 이름을 변경하고 변경 사항에 대한 별도의 브랜치를 생성할 수 있습니다.

사용자 정의 작업 공간을 준비하려면 다음을 수행하세요:

1. 로컬 컴퓨터에서 터미널을 열고 `substrate-node-template` 루트 디렉토리가 포함된 디렉토리로 이동합니다.

2. 다음 명령을 실행하여 `substrate-node-template` 루트 디렉토리의 이름을 변경합니다:
   
   ```bash
   mv substrate-node-template workshop-node-template
   ```

   이후 단계에서 `workshop-node-template` 디렉토리는 노드의 루트 디렉토리를 가리키는 데 사용됩니다.

3. 다음 명령을 실행하여 노드의 루트 디렉토리로 이동합니다:
   
   ```bash
   cd workshop-node-template
   ```

4. 다음과 유사한 명령을 실행하여 작업 공간에 대한 사용자 정의 브랜치를 생성합니다:
   
   ```bash
   git switch -c build-substrate-workshop
   ```

   작업 공간 폴더와 저장소 브랜치의 이름만 변경하므로 노드를 다시 컴파일할 필요는 없습니다.
   컴파일을 통해 빌드를 확인하려면 다음 명령을 실행하세요:

   ```bash
   cargo build --release
   ```

## 개발 모드에서 노드 시작하기

작업 공간의 이름을 변경한 후 개발 모드에서 노드를 시작하고 백그라운드에서 실행할 수 있습니다.
개발 모드에서는 개발 환경에 미리 정의된 몇 가지 기본 설정을 사용합니다.
개발 환경의 기본 설정 중 일부는 다음과 같습니다:

- Alice, Bob, Charlie, Dave를 포함한 몇 가지 미리 정의된 계정.
- Alice와 Bob 계정의 초기 계정 잔액.
- Alice 계정에 대한 루트 수준 권한.
- Alice 계정을 사용하여 블록 생성 및 유효성 검사를 위한 미리 정의된 권한.
  
기본적으로 개발 모드는 체인 데이터를 임시 데이터베이스에 저장하고 노드를 중지할 때 해당 데이터를 삭제합니다.

로컬 Substrate 노드를 시작하려면 다음을 수행하세요:

1. 필요한 경우 터미널 쉘을 엽니다.

2. `workshop-node-template` 루트 디렉토리로 이동합니다.

3. 다음 명령을 실행하여 개발 모드에서 노드를 시작합니다:
   
   ```bash
   ./target/release/node-template --dev
   ```
   
   개발 모드에서는 체인이 블록을 최종화하기 위해 피어 컴퓨터를 필요로하지 않습니다.
   초기화 메시지 몇 개 이후에 블록이 제안되고 봉인되며 최종화되는 것을 확인할 수 있습니다.
   예를 들면 다음과 같습니다:

   ```text
   2022-10-12 13:07:18 Substrate Node    
   2022-10-12 13:07:18 ✌️  version 4.0.0-dev-6c701fa6109    
   2022-10-12 13:07:18 ❤️  by Substrate DevHub <https://github.com/substrate-developer-hub>, 2017-2022    
   2022-10-12 13:07:18 📋 Chain specification: Development    
   2022-10-12 13:07:18 🏷  Node name: cooperative-clocks-6375    
   2022-10-12 13:07:18 👤 Role: AUTHORITY    
   ...
   2022-10-12 13:07:24 🙌 Starting consensus session on top of parent 0x72d1778c51e4c624e39d20bd3fac5d2c800a424fbd3bde08f4863bdf90966bf9    
   2022-10-12 13:07:24 🎁 Prepared block for proposing at 1 (1 ms) [hash: 0xdc23986196af67f6b07a78c50cd7b42c24b2e8da2d9f8ebf81c2540c358ada06; parent_hash: 0x72d1…6bf9; extrinsics (1): [0xd047…45da]]    
   2022-10-12 13:07:24 🔖 Pre-sealed block for proposal at 1. Hash now 0x0e2c8238d12d4babc83881384f6f178aa2996df5083387070c3d726bb7b10aec, previously 0xdc23986196af67f6b07a78c50cd7b42c24b2e8da2d9f8ebf81c2540c358ada06.    
   2022-10-12 13:07:24 ✨ Imported #1 (0x0e2c…0aec)
   ```

   이제 블록을 생성하는 실행 중인 블록체인이 있습니다.
   이를 백그라운드에서 실행하거나 원하는 때에 중지하고 다시 시작할 수 있습니다.
   
   - 노드를 중지하려면 Control-c를 누르세요.
   - 노드를 다시 시작하려면 `./target/release/node-template --dev`를 실행하세요.

   다음 단계는 이 블록체인이 하는 작업을 사용자 정의하는 것입니다.

   [작업 환경 준비](/tutorials/collectibles-workshop/01-prepare/)에서 배운 대로 Substrate는 Rust 프로그래밍 언어를 사용하여 구축됩니다.
   따라서 블록체인이 하는 작업을 사용자 정의하기 위해서는 Rust에서 작업해야 합니다.
   
   Rust에 익숙하다면 [새 팔렛 만들기](/tutorials/collectibles-workshop/03-create-pallet/)로 바로 이동하여 작성을 시작할 수 있습니다.
   Rust에 익숙하지 않다면 코드 작성을 시작하기 전에 [Detour: Learn Rust for Substrate](/tutorials/collectibles-workshop/detours/learn-rust/)에서 간단한 소개를 살펴보는 것이 좋습니다.