---
title: 특정 노드 승인
description: 승인된 노드 및 제한된 액세스 권한이 있는 네트워크를 구성합니다.
keywords:
  - 허가된
  - 비공개
  - 승인
  - 제한된 액세스
---

[신뢰할 수 있는 노드 추가](/tutorials/build-a-blockchain/add-trusted-nodes/)에서 알려드린대로 알려진 유효성 검사 노드의 집합을 갖는 간단한 네트워크를 구축하는 방법을 보았습니다.
이 튜토리얼은 **허가된 네트워크**의 단순화된 버전을 설명한 것입니다.
허가된 네트워크에서는 **승인된 노드**만 특정 네트워크 활동을 수행할 수 있습니다.
예를 들어, 일부 노드에게 블록 유효성 검사 권한을 부여하고 다른 노드에게 트랜잭션 전파 권한을 부여할 수 있습니다.

특정 권한이 부여된 노드가 있는 블록체인은 **공개** 또는 **허가되지 않은** 블록체인과 다릅니다.
허가되지 않은 블록체인에서는 적절한 하드웨어에서 노드 소프트웨어를 실행함으로써 누구나 네트워크에 참여할 수 있습니다.
일반적으로 허가되지 않은 블록체인은 네트워크의 분산을 더 크게 제공합니다.
그러나 허가된 블록체인을 만드는 것이 적절한 경우가 있는데, 예를 들어 다음과 같은 프로젝트에는 허가된 블록체인이 적합할 수 있습니다.

- 비공개 기업 또는 비영리 단체와 같은 개인 또는 협회 네트워크를 위해.
- 의료, 금융 또는 기업 간 원장과 같은 엄격한 규제 환경에서.
- 대규모로 사전 공개 블록체인 네트워크 테스트에 사용할 수 있습니다.

이 튜토리얼에서는 [노드 승인 팔렛](https://paritytech.github.io/substrate/master/pallet_node_authorization/index.html)를 사용하여 Substrate로 허가된 네트워크를 어떻게 구축할 수 있는지 설명합니다.

## 노드 승인 및 소유권
`node-authorization` 팔렛은 네트워크의 구성 가능한 노드 집합을 관리할 수 있도록 해주는 미리 구축된 FRAME 팔렛입니다.
각 노드는 피어 식별자(PeerId)로 식별됩니다.
각 피어 식별자는 노드를 주장하는 **하나의 계정 소유자(AccountId)**에 의해 소유됩니다.

노드를 네트워크에 참여할 수 있도록 승인하는 두 가지 방법이 있습니다.

- 피어 식별자를 사슬 사양 파일의 제네시스 구성의 일부로 사전 정의된 노드 목록에 추가함으로써.
이를 위해서는 사슬의 관리 메커니즘에 승인을 받거나 네트워크의 Sudo 팔렛에 대한 루트 계정에 액세스해야 합니다.

- 특정 노드로부터 페어링된 피어 연결을 요청함으로써.
사전 정의된 노드 피어 식별자 또는 각 노드의 공개 및 개인 키에서 생성된 피어 식별자를 사용하여 노드 간 연결을 추가할 수 있습니다.

다음 다이어그램이 제안한 대로, 이 튜토리얼은 사슬 사양에서 Alice와 Bob의 피어 식별자를 정의하고 노드 승인 팔렛을 사용하여 추가 노드를 추가하는 두 가지 승인 방법을 설명합니다.

![피어 식별자를 사용한 노드 승인](/media/images/docs/tutorials/permissioned-network/four-nodes.png)

어떤 사용자든지 `PeerId`의 소유자가 될 수 있습니다.
잘못된 주장을 방지하기 위해 노드 식별자를 노드를 시작하기 전에 주장해야 합니다.
노드를 시작하면 그 `PeerID`가 네트워크에 표시되고 누구든지 나중에 주장할 수 있습니다.

노드 소유자로서 노드의 연결을 추가하고 제거할 수 있습니다.
예를 들어, 사전 정의된 노드와 노드 간 또는 다른 사전 정의되지 않은 노드와 노드 간의 연결을 조작할 수 있습니다.
사전 정의된 노드의 연결을 변경할 수는 없습니다.
그들은 항상 서로 연결할 수 있습니다.

`node-authorization` 팔렛은 [오프체인 워커](/learn/offchain-operations)를 사용하여 노드 연결을 구성합니다.
비권한 노드의 경우 기본적으로 비활성화되어 있으므로 노드를 시작할 때 오프체인 워커를 활성화해야 합니다.

## 시작하기 전에

시작하기 전에 다음을 확인하십시오:

- [Rust 및 Rust 툴체인](/install/)을 설치하여 Substrate 개발 환경을 구성했습니다.

- [로컬 블록체인 빌드](/tutorials/build-a-blockchain/build-local-blockchain/)을 완료하고 Substrate 노드 템플릿을 로컬로 설치했습니다.

- [신뢰할 수 있는 노드 추가](/tutorials/build-a-blockchain/add-trusted-nodes/) 추가 튜토리얼을 완료했습니다.

- Substrate에서 [P2P 네트워킹](https://wiki.polkadot.network/docs/faq#networking)에 대해 일반적으로 알고 있습니다.

## 튜토리얼 목표

이 튜토리얼을 완료함으로써 다음 목표를 달성하게 됩니다:

- 노드 템플릿을 체크아웃하고 컴파일합니다.

- 노드 템플릿 런타임에 노드 승인 팔렛을 추가합니다.

- 여러 노드를 시작하고 새로운 노드를 승인합니다.

## 노드 템플릿 빌드

이전 튜토리얼을 완료한 경우 로컬에 Substrate 노드 템플릿 리포지토리가 있어야 합니다.

1. Substrate 노드 템플릿 리포지토리가 있는 컴퓨터에서 터미널 쉘을 엽니다.

2. 필요한 경우 다음 명령을 실행하여 노드 템플릿 디렉터리의 루트로 이동합니다:

   ```bash
   cd substrate-node-template
   ```

3. 변경 사항을 저장하려면 리포지토리에 작업 브랜치로 전환하려면 다음과 유사한 명령을 실행합니다:

   ```bash
   git switch -c my-wip-branch
   ```

4. 다음 명령을 실행하여 노드 템플릿을 컴파일합니다:

   ```bash
   cargo build --release
   ```

노드 템플릿은 오류 없이 컴파일되어야 합니다.
컴파일 시 문제가 발생하는 경우 [Rust 문제 해결 팁](/install/troubleshoot-rust-issues/)을 시도해 볼 수 있습니다.

## 노드 승인 팔렛 추가

새 팔렛을 사용하기 전에 런타임 바이너리를 빌드하는 컴파일러가 사용하는 구성 파일에 대한 정보를 추가해야 합니다.

Rust 프로그램의 경우 `Cargo.toml` 파일을 사용하여 결과 바이너리를 컴파일하는 데 사용되는 구성 설정 및 종속성을 정의합니다.
Substrate 런타임은 표준 라이브러리 함수를 포함하는 네이티브 Rust 바이너리 및 표준 라이브러리를 포함하지 않는 [WebAssembly (Wasm)](https://webassembly.org/) 바이너리로 컴파일되므로 `Cargo.toml` 파일은 두 가지 중요한 정보를 제어합니다:

- 런타임에 종속성으로 가져올 팔렛 및 팔렛을 가져올 위치와 버전을 포함합니다.

- 네이티브 Rust 바이너리를 컴파일할 때 활성화해야 하는 각 팔렛의 기능을 제어합니다.
  각 팔렛에서 표준 (`std`) 기능 집합을 활성화함으로써 웹어셈블리 바이너리를 빌드할 때 누락되는 함수, 타입 및 기본 요소를 포함하여 런타임을 컴파일할 수 있습니다.

`Cargo.toml` 파일에서 종속성을 추가하는 일반적인 정보는 [의존성(Dependencies)](https://doc.rust-lang.org/cargo/guide/dependencies.html)을 참조하십시오. 종속 패키지의 기능을 활성화하고 관리하는 정보는 Cargo 문서의 [기능(Features)](https://doc.rust-lang.org/cargo/reference/features.html)을 참조하십시오.

### 노드 승인 종속성 추가

`node-authorization` 팔렛을 Substrate 런타임에 추가하려면 다음 단계를 따릅니다:

1. 터미널 쉘을 열고 노드 템플릿의 루트 디렉토리로 이동합니다.

2. 텍스트 편집기에서 `runtime/Cargo.toml` 구성 파일을 엽니다.

3. `[dependencies]` 섹션을 찾아 `pallet-node-authorization` 크레이트를 노드 템플릿 런타임에서 사용할 수 있도록 추가합니다.

   ```toml
   [dependencies]
   pallet-node-authorization = { default-features = false, version = "4.0.0-dev", git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-v1.0.0" }
   ```

   이 줄은 팔렛을 종속성으로 가져오며 크레이트에 대한 다음 구성 세부 정보를 지정합니다.

   - 런타임을 컴파일할 때 팔렛 기능은 기본적으로 활성화되지 않습니다.
   - 크레이트의 버전 식별자.
   - `pallet-node-authorization` 크레이트를 검색하는 리포지토리 위치.
   - 크레이트를 검색하는 브랜치.

   팔렛이 서로 호환되도록 하기 위해 모든 팔렛에 대해 동일한 브랜치 및 버전 정보를 사용해야 합니다. 서로 다른 브랜치의 팔렛을 사용하면 컴파일러 오류가 발생할 수 있습니다. 이 예제는 다른 팔렛이 `branch = "polkadot-v1.0.0"`를 사용하는 경우 `Cargo.toml` 파일에 팔렛을 추가하는 방법을 보여줍니다.

2. 런타임을 컴파일할 때 활성화할 `pallet-node-authorization/std` 기능 목록에 추가합니다.

   ```toml
   [features]
   default = ['std']
   std = [
     ...
     "pallet-node-authorization/std",    # 이 줄 추가
     ...
   ]
   ```

   이 섹션은 이 런타임에 대한 기본 기능 집합을 `std` 기능 집합으로 지정합니다. 런타임을 `std` 기능 집합을 사용하여 컴파일할 때 종속성으로 나열된 모든 팔렛의 `std` 기능이 활성화됩니다. 네이티브 Rust 바이너리 및 표준 라이브러리를 사용하여 런타임이 컴파일되는 방법에 대한 자세한 정보는 [빌드 프로세스(Build process)](https://docs.substrate.io/v3/runtime/basics/build-process)를 참조하십시오.

   `Cargo.toml` 파일의 `features` 섹션을 업데이트하는 것을 잊어버리면 런타임 바이너리를 컴파일할 때 `cannot find function` 오류가 발생할 수 있습니다.

3. 다음 명령을 실행하여 새 종속성이 올바르게 해결되는지 확인합니다.

   ```bash
   cargo check -p node-template-runtime --release
   ```

### 관리 규칙 추가

이 튜토리얼에서는 시뮬레이션을 위해 팔렛을 `EnsureRoot` 권한이 있는 함수를 사용하도록 구성할 수 있으며, 이 함수는 Sudo 팔렛을 사용하여 호출할 수 있습니다. Sudo 팔렛은 기본적으로 노드 템플릿에 포함되어 있으며 루트 수준의 관리 계정을 통해 호출할 수 있게 해줍니다. 실제 환경에서는 더 현실적인 지배 기반 확인을 사용해야 합니다.

런타임에서 `EnsureRoot` 규칙을 활성화하려면 다음 단계를 따르십시오:

1. 텍스트 편집기에서 `runtime/src/lib.rs` 파일을 엽니다.

2. 파일에 다음 라인을 추가합니다:

   ```rust
   use frame_system::EnsureRoot;
   ```

### Config 트레이트 구현

모든 팔렛은 `Config`라는 [Rust 트레이트](https://doc.rust-lang.org/book/ch10-02-traits.html)를 가지고 있습니다. `Config` 트레이트는 팔렛이 필요로 하는 매개변수와 타입을 식별하는 데 사용됩니다.

팔렛을 추가하려면 `Config` 트레이트를 통해 구현해야 할 대부분의 팔렛별 코드가 있습니다. 팔렛의 `Config` 트레이트를 구현하는 데 필요한 내용은 해당 팔렛의 Rust 문서나 팔렛의 소스 코드를 참고하여 확인할 수 있습니다. 예를 들어, `node-authorization` 팔렛에서 `Config` 트레이트를 구현해야 하는 내용을 확인하려면 [`pallet_node_authorization::Config`](https://paritytech.github.io/substrate/master/pallet_node_authorization/pallet/trait.Config.html)에 대한 Rust 문서를 참고할 수 있습니다.

런타임에 `node-authorization` 팔렛을 구현하려면 다음 단계를 따릅니다:

1. 텍스트 편집기에서 `runtime/src/lib.rs` 파일을 엽니다.

2. 다음 코드를 사용하여 팔렛을 위한 `parameter_types` 섹션을 추가합니다.

   ```rust
   parameter_types! {
     pub const MaxWellKnownNodes: u32 = 8;
     pub const MaxPeerIdLength: u32 = 128;
   }
   ```

3. 다음 코드를 사용하여 팔렛의 `Config` 트레이트를 위한 `impl` 섹션을 추가합니다.

   ```rust
   impl pallet_node_authorization::Config for Runtime {
     type RuntimeEvent = RuntimeEvent;
     type MaxWellKnownNodes = MaxWellKnownNodes;
     type MaxPeerIdLength = MaxPeerIdLength;
     type AddOrigin = EnsureRoot<AccountId>;
     type RemoveOrigin = EnsureRoot<AccountId>;
     type SwapOrigin = EnsureRoot<AccountId>;
     type ResetOrigin = EnsureRoot<AccountId>;
     type WeightInfo = ();
   }
   ```

4. 다음 코드를 사용하여 팔렛을 `construct_runtime` 매크로에 추가합니다.

   ```rust
   construct_runtime!(
   pub enum Runtime where
       Block = Block,
       NodeBlock = opaque::Block,
       UncheckedExtrinsic = UncheckedExtrinsic
     {
       /*** 이 줄 추가 ***/
       NodeAuthorization: pallet_node_authorization::{Pallet, Call, Storage, Event<T>, Config<T>},
     }
   );
   ```

5. 변경 사항을 저장하고 파일을 닫습니다.

6. 다음 명령을 실행하여 구성이 컴파일되는지 확인합니다.

   ```bash
   cargo check -p node-template-runtime --release
   ```

### 승인된 노드를 위한 제네시스 스토리지 추가

승인된 노드를 사용하여 네트워크를 시작하려면 피어 식별자와 계정 식별자를 처리하는 몇 가지 추가 구성이 필요합니다.
예를 들어 `PeerId`는 bs58 형식으로 인코딩되기 때문에 `PeerId`를 해독하여 해당 바이트를 가져오려면 `node/Cargo.toml`에 [bs58](https://docs.rs/bs58/) 라이브러리에 대한 새로운 종속성을 추가해야 합니다.
승인된 노드는 미리 정의된 계정과 연결되어 있습니다.

승인된 노드를 위한 제네시스 스토리지를 구성하려면 다음 단계를 따르십시오.

1. 텍스트 편집기에서 `node/Cargo.toml` 파일을 엽니다.

2. `[dependencies]` 섹션을 찾아서 노드 템플릿에 `bs58` 라이브러리를 추가합니다.

   ```toml
   [dependencies]
   bs58 = { version = "0.4.0" }
   ```

3. 변경 사항을 저장하고 파일을 닫습니다.

4. 텍스트 편집기에서 `node/src/chain_spec.rs` 파일을 엽니다.

5. 다음 코드를 사용하여 네트워크에 참여하도록 승인된 노드를 위한 제네시스 스토리지를 추가합니다.

   ```rust
   use sp_core::OpaquePeerId; // 노드 `PeerId`를 나타내기 위해 Vec<u8>을 래핑한 구조체.
   use node_template_runtime::NodeAuthorizationConfig; // 팔렛을 제공하는 제네시스 구성.
   ```

6. FRAME 모듈에 대한 초기 스토리지 상태를 구성하는 `testnet_genesis` 함수를 찾습니다.

   예를 들어:

   ```rust
   /// Configure initial storage state for FRAME modules.
   fn testnet_genesis(
     wasm_binary: &[u8],
     initial_authorities: Vec<(AuraId, GrandpaId)>,
     root_key: AccountId,
     endowed_accounts: Vec<AccountId>,
     _enable_println: bool,
     ) -> GenesisConfig {

   ```

7. `GenesisConfig` 선언 내에서 다음 코드 블록을 추가합니다.

   ```rust
     node_authorization: NodeAuthorizationConfig {
       nodes: vec![
         (
           OpaquePeerId(bs58::decode("12D3KooWBmAwcd4PJNJvfV89HwE48nwkRmAgo8Vy3uQEyNNHBox2").into_vec().unwrap()),
           endowed_accounts[0].clone()
         ),
         (
           OpaquePeerId(bs58::decode("12D3KooWQYV9dGMFoRzNStwpXztXaBUjtPqi6aU76ZgUriHhKust").into_vec().unwrap()),
           endowed_accounts[1].clone()
         ),
       ],
     },
   ```

   이 코드에서 `NodeAuthorizationConfig`는 두 개의 요소로 구성된 튜플의 벡터인 `nodes` 속성을 포함하고 있습니다.
   튜플의 첫 번째 요소는 `OpaquePeerId`입니다.
   `bs58::decode` 작업을 사용하여 사람이 읽을 수 있는 `PeerId` (예: `12D3KooWBmAwcd4PJNJvfV89HwE48nwkRmAgo8Vy3uQEyNNHBox2`)를 바이트로 변환합니다.
   튜플의 두 번째 요소는 이 노드의 소유자를 나타내는 `AccountId`입니다.
   이 예제에서는 사전 정의된 [Alice 및 Bob](/reference/command-line-tools/subkey/#well-known-keys)을 사용하고 있으며, 여기서는 dowed 계정 [0] 및 [1]으로 식별됩니다.

8. 변경 사항을 저장하고 파일을 닫습니다.

### 노드 컴파일 확인

이제 코드 변경을 완료했으므로 노드가 컴파일되는지 확인할 준비가 되었습니다.

노드를 컴파일하려면 다음 단계를 수행하세요.

1. 필요한 경우 `substrate-node-template` 디렉토리의 루트로 이동합니다.

2. 다음 명령을 실행하여 노드를 컴파일합니다.

   ```bash
   cargo build --release
   ```

   문법 오류가 없는 경우 계속 진행할 수 있습니다.
   오류가 발생하는 경우 컴파일 출력에서 지시 사항을 따라 오류를 수정한 다음 `cargo build` 명령을 다시 실행하세요.

## 사용할 계정 키 식별

이미 Genesis 스토리지에 Alice 및 Bob 계정과 관련된 노드를 구성했습니다.
[`subkey`](/reference/command-line-tools/subkey/) 프로그램을 사용하여 미리 정의된 계정과 관련된 키를 검사하고 자체 키를 생성하고 검사할 수 있습니다.
그러나 `subkey generate-node-key` 명령을 실행하면 노드 키와 피어 식별자가 무작위로 생성되어 튜토리얼에서 사용되는 키와 일치하지 않을 것입니다.
이 튜토리얼에서는 미리 정의된 계정과 잘 알려진 노드 키를 사용하므로 각 계정에 대한 다음 키를 사용할 수 있습니다.

### Alice

| Key type | Key value |
| :------- | :-------------------------------------------------------------|
| Node key | c12b6d18942f5ee8528c8e2baf4e147b5c5c18710926ea492d09cbd9f6c9f82a |
| Peer&nbspidentifier&nbspgenerated from the node key | 12D3KooWBmAwcd4PJNJvfV89HwE48nwkRmAgo8Vy3uQEyNNHBox2 |
| Peer&nbspidentifier&nbspdecoded to hex | 0x0024080112201ce5f00ef6e89374afb625f1ae4c1546d31234e87e3c3f51a62b91dd6bfa57df |

### Bob

| Key type | Key value |
| :------- | :-------------------------------------------------------------|
| Node key | 6ce3be907dbcabf20a9a5a60a712b4256a54196000a8ed4050d352bc113f8c58 |
| Peer&nbspidentifier&nbspgenerated from the node key | 12D3KooWQYV9dGMFoRzNStwpXztXaBUjtPqi6aU76ZgUriHhKust |
| Peer&nbspidentifier&nbspdecoded to hex | 0x002408011220dacde7714d8551f674b8bb4b54239383c76a2b286fa436e93b2b7eb226bf4de7 |

The two other development accounts—Charlie and Dave—don't have well-known node keys or peer identifiers defined in the genesis configuration.
For demonstration purposes, you can use the following keys for these accounts.

### Charlie

| Key type | Key value |
| :------- | :-------------------------------------------------------------|
| Node key | 3a9d5b35b9fb4c42aafadeca046f6bf56107bd2579687f069b42646684b94d9e |
| Peer&nbspidentifier&nbspgenerated from the node key | 12D3KooWJvyP3VJYymTqG7eH4PM5rN4T2agk5cdNCfNymAqwqcvZ |
| Peer&nbspidentifier&nbspdecoded to hex | 0x002408011220876a7b4984f98006dc8d666e28b60de307309835d775e7755cc770328cdacf2e |

### Dave

| Key type | Key value |
| :------- | :-------------------------------------------------------------|
| Node key | a99331ff4f0e0a0434a6263da0a5823ea3afcfffe590c9f3014e6cf620f2b19a |
| Peer&nbspidentifier&nbspgenerated from the node key | 12D3KooWPHWFrfaJzxPnqnAYAoRUyAHHKqACmEycGTVmeVhQYuZN |
| Peer&nbspidentifier&nbspdecoded to hex | 0x002408011220c81bc1d7057a1511eb9496f056f6f53cdfe0e14c8bd5ffca47c70a8d76c1326d |

이 튜토리얼에서는 노드 키를 파일로 복사하고 `subkey inspect-node-key`를 사용하여 Charlie와 Dave의 피어 식별자를 확인할 수 있습니다. 예를 들어, 다음과 같이 Charlie를 위한 노드 키를 `charlie-node-key`라는 이름의 파일에 저장할 수 있습니다.

```bash
echo -n "3a9d5b35b9fb4c42aafadeca046f6bf56107bd2579687f069b42646684b94d9e" > charlie-node-key
```

그런 다음 다음 명령을 실행하여 피어 식별자를 확인할 수 있습니다.

```bash
./subkey inspect-node-key --file charlie-node-key
```

이 명령은 Charlie 노드의 피어 식별자를 표시합니다.

```text
12D3KooWJvyP3VJYymTqG7eH4PM5rN4T2agk5cdNCfNymAqwqcvZ
```

자체 계정에 대한 노드 키를 생성하는 경우 필요할 때 `subkey inspect-node-key` 또는 다른 명령에 전달할 수 있도록 노드의 피어 식별자를 파일에 저장하세요.

## 네트워크 노드 시작

미리 정의된 계정의 노드 키와 피어 식별자를 사용하여 권한 부여된 네트워크를 시작하고 다른 노드를 인가할 수 있습니다.

이 튜토리얼의 목적을 위해 네 개의 노드를 시작할 것입니다.
세 개의 노드는 미리 정의된 계정과 관련이 있으며 이 세 노드 모두 블록을 생성하고 유효성을 검사하는 데 사용됩니다.
네 번째 노드는 선택한 노드의 소유자의 승인을 받아 선택한 노드에서 데이터를 읽기만 할 수 있는 **하위 노드**입니다.

### 첫 번째 노드 시작

Alice와 Bob의 잘 알려진 노드 키를 사용하도록 Genesis 스토리지를 구성했으므로 첫 번째 노드를 시작할 때 `--name alice --validator`의 `--alice` 명령 바로 가기를 사용할 수 있습니다.

첫 번째 노드를 시작하려면 다음 단계를 수행하세요.

1. 필요한 경우 컴퓨터에서 터미널 쉘을 엽니다.

2. Substrate 노드 템플릿을 컴파일한 루트 디렉토리로 변경합니다.

3. 다음 명령을 실행하여 첫 번째 노드를 시작합니다.

   ```bash
   ./target/release/node-template \
   --chain=local \
   --base-path /tmp/validator1 \
   --alice \
   --node-key=c12b6d18942f5ee8528c8e2baf4e147b5c5c18710926ea492d09cbd9f6c9f82a \
   --port 30333 \
   --rpc-port 9944
   ```

   이 명령에서는 안전한 네트워크 연결에 사용할 키를 지정하는 `--node-key` 옵션을 사용합니다.
   이 키는 위의 섹션에서 표시된 대로 인간이 읽을 수 있는 PeerId를 내부적으로 생성하는 데도 사용됩니다.

   다른 튜토리얼에서 본 것처럼 사용된 명령줄 옵션은 다음과 같습니다.

   - 로컬 테스트넷용 `--chain=local`입니다 (개발 플래그인 `--dev`와는 다릅니다!).
   - 노드를 `alice`로 명명하고 노드를 블록 생성자로 만들기 위해 `--alice`입니다.
   - 피어 간 통신용 포트를 지정하기 위한 `--port`입니다.
   - WebSocket 연결용 수신 대기 포트를 지정하기 위한 `--rpc-port`입니다.

### 두 번째 노드 시작

두 번째 노드를 시작하려면 두 번째 노드를 시작하는 데 `--name bob --validator`의 `--bob` 명령 바로 가기를 사용할 수 있습니다.

두 번째 노드를 시작하려면 다음 단계를 수행하세요.

1. 컴퓨터에서 **새로운** 터미널 쉘을 엽니다.

2. Substrate 노드 템플릿을 컴파일한 루트 디렉토리로 변경합니다.

3. 다음 명령을 실행하여 두 번째 노드를 시작합니다.

   ```bash
   ./target/release/node-template \
   --chain=local \
   --base-path /tmp/validator2 \
   --bob \
   --node-key=6ce3be907dbcabf20a9a5a60a712b4256a54196000a8ed4050d352bc113f8c58 \
   --bootnodes /ip4/127.0.0.1/tcp/30333/p2p/12D3KooWBmAwcd4PJNJvfV89HwE48nwkRmAgo8Vy3uQEyNNHBox2 \
   --port 30334 \
   --rpc-port 9945
   ```

   이후에 두 노드가 시작되면 두 터미널 로그에서 새로운 블록이 생성되고 최종화됨을 볼 수 있어야 합니다.

### 잘 알려진 노드 목록에 세 번째 노드 추가

세 번째 노드를 시작할 때 `--name charlie` 명령을 사용할 수 있습니다.
`node-authorization` 팔레트는 [오프체인 워커](/learn/offchain-operations)를 사용하여 노드 연결을 구성합니다.
세 번째 노드는 잘 알려진 노드가 아니며 네트워크에서 네 번째 노드를 읽기 전용 하위 노드로 구성해야 하므로 오프체인 워커를 활성화하는 명령줄 옵션을 포함해야 합니다.

세 번째 노드를 시작하려면 다음 단계를 수행하세요.

1. 컴퓨터에서 **새로운** 터미널 쉘을 엽니다.

2. Substrate 노드 템플릿을 컴파일한 루트 디렉토리로 변경합니다.

3. 다음 명령을 실행하여 세 번째 노드를 시작합니다.

   ```bash
   ./target/release/node-template \
   --chain=local \
   --base-path /tmp/validator3 \
   --name charlie  \
   --node-key=3a9d5b35b9fb4c42aafadeca046f6bf56107bd2579687f069b42646684b94d9e \
   --port 30335 \
   --rpc-port=9946 \
   --offchain-worker always
   ```

   이 노드를 시작한 후에는 노드에 **연결된 피어가 없음**을 볼 수 있어야 합니다.
   이것은 권한이 있는 네트워크이므로 명시적으로 연결 권한을 부여해야 합니다.
   Alice와 Bob 노드는 genesis `chain_spec.rs` 파일에서 구성되었습니다.
   다른 모든 노드는 Sudo 팔레트를 호출하여 수동으로 추가해야 합니다.

### 세 번째 노드에 대한 액세스 권한 부여

이 튜토리얼에서는 지배권을 위해 Sudo 팔레트를 사용합니다.
따라서 Sudo 팔레트를 사용하여 `node-authorization` 팔레트에서 제공하는 `addWellKnownNode` 함수를 호출하여 세 번째 노드를 추가할 수 있습니다.
단순하게 하기 위해 Sudo 팔레트에 액세스하려면 Polkadot/Substrate Portal 응용 프로그램을 사용할 수 있습니다.

1. 브라우저에서 [Polkadot/Substrate Portal](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/explorer)를 엽니다.

2. **개발자**를 클릭하고 **Sudo**를 선택합니다.

3. **nodeAuthorization**을 선택하고 **addWellKnownNode(node, owner)**를 선택합니다.

4. 필수 0x 접두사 이후에 Charlie가 소유한 노드에 대한 16진수로 인코딩된 피어 식별자를 복사하여 붙여넣습니다.

5. 노드 소유자로 **Charlie**를 선택합니다.
   
6. **Sudo 제출**을 클릭합니다.
   
   ![세 번째 노드 추가를 위한 Sudo 트랜잭션 제출](/media/images/docs/tutorials/permissioned-network/sudo-add-well-known-node.png)

7. 권한 부여 트랜잭션에서 Alice 개발 계정이 기본 루트 관리 계정으로 사용되며, **서명 및 제출**을 클릭합니다.
   
   ![트랜잭션 검증 및 권한 부여](/media/images/docs/tutorials/permissioned-network/sudo-to-add-node.png)
   
8. **네트워크**를 클릭하고 **탐색기**를 선택하여 최근 트랜잭션을 확인합니다.
   
   ![Sudo 이벤트 확인](/media/images/docs/tutorials/permissioned-network/sudo-events-add-node.png)

   트랜잭션이 블록에 포함된 후에는 `charlie` 노드가 `alice`와 `bob` 노드에 연결되고 블록 동기화를 시작해야 합니다.
   이 세 노드는 로컬 네트워크에서 기본적으로 활성화된 [mDNS](https://paritytech.github.io/substrate/master/sc_network/index.html) 검색 메커니즘을 사용하여 서로를 찾을 수 있어야 합니다. 로컬 방화벽도 mDNS를 허용하도록 구성되어 있어야 합니다.
   
   노드가 동일한 로컬 네트워크에 없는 경우 `--no-mdns` 명령줄 옵션을 사용하여 비활성화해야 합니다.

### 하위 노드에서의 연결 허용

이 네트워크의 네 번째 노드는 발행자로 사용되지 않으며 잘 알려진 노드 목록에 추가되지 않을 것입니다.
이 네 번째 노드는 `dave` 사용자 계정이 소유하지만 `charlie` 노드의 하위 노드입니다.
하위 노드는 `charlie` 상위 노드에 연결하여만 네트워크에 액세스할 수 있습니다.
하위 노드를 연결하는 것을 승인하는 데 부모 노드가 책임이 있으며 하위 노드를 제거하거나 감사할 필요가 있는 경우 부모 노드의 소유자가 액세스를 제어해야 합니다.

이것이 _권한 있는 네트워크_ 이므로 Dave의 노드에서 올 수 있도록 Charlie는 자신의 노드를 구성해야 합니다.
Polkadot/Substrate Portal 응용 프로그램을 사용하여 이 권한을 부여할 수 있습니다.

하위 노드가 네트워크에 액세스하도록 허용하려면 다음 단계를 수행하세요.

1. 브라우저에서 [Polkadot/Substrate Portal](https://polkadot.js.org/apps/#/explorer)를 엽니다.

2. **개발자**를 클릭하고 **Extrinsics**를 선택합니다.

3. **nodeAuthorization**을 선택하고 **addConnections(node, connections)**를 선택합니다.

4. Charlie가 소유한 노드에 대한 16진수로 인코딩된 피어 식별자를 필수 0x 접두사 이후에 복사하여 붙여넣습니다.

5. 연결 매개변수에 필수 0x 접두사 이후에 Dave가 소유한 노드에 대한 16진수로 인코딩된 피어 식별자를 복사하여 붙여넣은 다음 **트랜잭션 제출**을 클릭합니다.
   
6. 트랜잭션 세부 정보를 검토한 후 **서명 및 제출**을 클릭합니다.
   
   ![charlie_add_connections](/media/images/docs/tutorials/permissioned-network/charlie_add_connections.png)

### 하위 노드 요청

하위 노드를 시작하기 전에 노드 소유자는 자신의 노드 피어 식별자를 요청해야 합니다.
Dave 계정이 소유한 노드를 요청하기 위해 Polkadot/Substrate Portal 응용 프로그램을 사용할 수 있습니다.

1. 브라우저에서 [Polkadot/Substrate Portal](https://polkadot.js.org/apps/#/explorer)를 엽니다.

2. **개발자**를 클릭하고 **Extrinsics**를 선택합니다.

3. **nodeAuthorization**을 선택하고 **claimNode(node)**를 선택합니다.

4. Dave가 소유한 노드에 대한 16진수로 인코딩된 피어 식별자를 필수 0x 접두사 이후에 복사하여 붙여넣은 다음 **트랜잭션 제출**을 클릭합니다.
   
5. 트랜잭션 세부 정보를 검토한 후 **서명 및 제출**을 클릭합니다.

   ![dave_claim_node](/media/images/docs/tutorials/permissioned-network/dave_claim_node.png)

### 하위 노드 시작

노드 피어 식별자를 요청한 후 하위 노드를 시작할 준비가 되었습니다.

하위 노드를 시작하려면 다음 단계를 수행하세요:

1. 컴퓨터에서 새로운 터미널 창을 엽니다.

2. Substrate 노드 템플릿을 컴파일한 루트 디렉토리로 이동합니다.

3. 다음 명령을 실행하여 하위 노드를 시작합니다:
   
   ```bash
   ./target/release/node-template \
   --chain=local \
   --base-path /tmp/validator4 \
   --name dave \
   --node-key=a99331ff4f0e0a0434a6263da0a5823ea3afcfffe590c9f3014e6cf620f2b19a \
   --port 30336 \
   --rpc-port 9947 \
   --offchain-worker always
   ```

### 하위 노드로부터의 연결 허용

이제 네 개의 노드를 가진 네트워크를 보유하고 있습니다.
그러나 하위 노드가 참여하도록 허용하려면 Charlie가 소유한 상위 노드에서 연결을 허용하도록 구성해야 합니다.
Dave가 소유한 노드에서 연결을 허용하도록 구성한 방법과 유사한 단계입니다.

1. 브라우저에서 [Polkadot/Substrate Portal](https://polkadot.js.org/apps/#/explorer)를 엽니다.

2. **개발자**를 클릭하고 **Extrinsics**를 선택합니다.

3. **nodeAuthorization**을 선택하고 **addConnections(node, connections)**를 선택합니다.

4. Dave가 소유한 노드에 대한 16진수로 인코딩된 피어 식별자를 필수 0x 접두사 이후에 복사하여 붙여넣습니다. 

5. 연결 매개변수에 필수 0x 접두사 이후에 Charlie가 소유한 노드에 대한 16진수로 인코딩된 피어 식별자를 복사하여 붙여넣은 다음 **트랜잭션 제출**을 클릭합니다.
   
6. 트랜잭션 세부 정보를 검토한 후 **서명 및 제출**을 클릭합니다.
   
   ![dave_add_connections](/media/images/docs/tutorials/permissioned-network/dave_add_connections.png)
   
   이제 하위 노드가 Charlie의 소유한 노드에만 연결되어 있고 체인에서 블록을 동기화하고 있는 것을 볼 수 있어야 합니다.
   하위 노드가 바로 피어 노드에 연결되지 않으면 하위 노드를 중지하고 다시 시작해 보세요.

## 트랜잭션 제출에 필요한 키

다음 사항을 주목해야 합니다. 다른 노드의 동작에 영향을 주는 트랜잭션을 서명하고 제출하는 데 어떤 계정이든 사용할 수 있습니다.
그러나 자신이 소유하지 않은 노드에 영향을 미치는 트랜잭션을 서명하고 제출하려면 다음과 같은 조건을 충족해야 합니다:

- 트랜잭션은 _체인 상 데이터_를 참조해야 합니다.
- 필요한 원천(origin)을 가진 계정의 _서명 키(signing key)_가 keystore에 있어야 합니다.

이 튜토리얼에서는 모든 노드가 개발 계정 서명 키에 액세스할 수 있습니다.
따라서 여러분은 Charlie 또는 Dave를 대신하여 연결된 노드에 영향을 미치는 트랜잭션을 서명하고 제출할 수 있었습니다.
실제 응용 프로그램을 위한 허가된 네트워크를 구축하는 경우, 노드 운영자들은 대부분 자신의 노드 키에만 액세스할 수 있으며, 노드 소유 계정은 서명 키를 통제하는 데 필요한 노드에 영향을 미치는 트랜잭션을 서명하고 제출해야 할 것입니다.

## 다음 단계

이 튜토리얼에서는 일부 노드가 제한된 권한과 네트워크 리소스에 제한된 액세스를 갖는 네트워크를 구축하는 기본 사항을 배웠습니다.
이 튜토리얼에서 소개된 주제에 대해 더 자세히 알아보려면 다음 리소스를 참조하십시오:

- [계정, 주소 및 키](/learn/accounts-addresses-keys)
- [노드 권한 부여 팔레트](https://paritytech.github.io/substrate/master/pallet_node_authorization/index.html#)
- [노드 권한 부여 소스 코드](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/node-authorization/src/lib.rs)
- [노드 메트릭스 모니터링](/tutorials/build-a-blockchain/monitor-node-metrics/)