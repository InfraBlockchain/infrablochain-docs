---
title: 첫 번째 계약 준비하기
description: ink! 스마트 컨트랙트 언어를 사용하여 간단한 스마트 컨트랙트를 작성하고 테스트하세요.
keywords:
---

[블록체인 기본](../../learn/basic/blockchain-basics.md)에서 배운 대로, 탈중앙화 애플리케이션은 대부분 **스마트 컨트랙트**으로 작성됩니다.

Substrate는 주로 맞춤형 블록체인을 구축하기 위한 프레임워크와 도구이지만, 스마트 컨트랙트 플랫폼으로도 사용할 수 있습니다.

이 튜토리얼에서는 Substrate 기반 체인에서 기본적인 스마트 컨트랙트를 작성하는 방법을 보여줍니다.

이 튜토리얼에서는 Rust 기반 스마트 컨트랙트를 작성하기 위한 프로그래밍 언어로 ink!를 사용하는 방법을 알아봅니다.

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- 컴퓨터에 좋은 인터넷 연결과 로컬 컴퓨터에서 셸 터미널에 액세스할 수 있어야 합니다.

- 소프트웨어 개발과 명령 줄 인터페이스 사용에 대해 일반적으로 알고 있어야 합니다.

- 블록체인과 스마트 컨트랙트 플랫폼에 대해 일반적으로 알고 있어야 합니다.

- [설치](../install/README.md)에 설명된 대로 Rust를 설치하고 개발 환경을 설정했는지 확인하세요.

## 튜토리얼 목표

이 튜토리얼을 완료함으로써 다음 목표를 달성할 수 있습니다:

- 스마트 컨트랙트 프로젝트를 생성하는 방법을 배웁니다.

- ink! 스마트 컨트랙트 언어를 사용하여 스마트 컨트랙트를 작성하고 테스트합니다.

- 로컬 Substrate 노드에 스마트 컨트랙트를 배포합니다.

- `cargo-contract` CLI를 사용하여 스마트 컨트랙트와 상호 작용합니다.

## Rust 환경 업데이트하기

이 튜토리얼에서는 Substrate 개발 환경에 일부 Rust 소스 코드를 추가해야 합니다.

개발 환경을 업데이트하려면 다음 단계를 따르세요:

1. 컴퓨터에서 터미널 셸을 엽니다.

2. 다음 명령을 실행하여 Rust 환경을 업데이트합니다:

   ```bash
   rustup component add rust-src
   ```

3. 다음 명령을 실행하여 WebAssembly 타겟이 설치되었는지 확인합니다:

   ```bash
   rustup target add wasm32-unknown-unknown --toolchain nightly
   ```

   타겟이 설치되어 있고 최신 상태인 경우, 다음과 유사한 출력이 표시됩니다:

   ```text
   info: component 'rust-std' for target 'wasm32-unknown-unknown' is up to date
   ```

## `cargo-contract` CLI 도구 설치하기

`cargo-contract`는 ink! 계약을 빌드, 배포 및 상호 작용하는 데 사용하는 명령 줄 도구입니다.

Rust 외에도 `cargo-contract`를 설치하려면 C++17을 지원하는 C++ 컴파일러가 필요합니다.

최신 버전의 `gcc`, `clang` 및 Visual Studio 2019+가 작동할 것입니다.

1. `rust-src` 컴파일러 구성 요소를 추가합니다:

```bash
rustup component add rust-src
```

2. 최신 버전의 `cargo-contract`를 설치합니다:

```bash
cargo install --force --locked cargo-contract --version 2.0.0-rc
```

3. 다음 명령을 실행하여 설치를 확인하고 사용 가능한 명령을 살펴봅니다:

```bash
cargo contract --help
```

## Substrate Contracts 노드 설치하기

이 튜토리얼을 간단하게 만들기 위해 [여기](https://github.com/paritytech/substrate-contracts-node/releases)에서
Linux 또는 macOS용 미리 컴파일된 Substrate 노드를 [다운로드](https://github.com/paritytech/substrate-contracts-node/releases)할 수 있습니다.

미리 컴파일된 이진 파일에는 기본적으로 스마트 컨트랙트를 위한 FRAME 팔렛이 포함되어 있습니다.

macOS 또는 Linux에서 계약 노드를 설치하려면 다음 단계를 따르세요:

1. [릴리스](https://github.com/paritytech/substrate-contracts-node/releases) 페이지를 엽니다.

2. 로컬 컴퓨터에 해당하는 압축 아카이브를 다운로드합니다.

3. 다운로드한 파일을 열고 작업 디렉토리에 내용을 추출합니다.

미리 컴파일된 노드를 다운로드할 수 없는 경우 다음과 유사한 명령을 사용하여 로컬에서 컴파일할 수 있습니다.
[릴리스](https://github.com/paritytech/substrate-contracts-node/releases) 페이지에서 최신 태그를 찾을 수 있습니다:

```bash
cargo install contracts-node --git https://github.com/paritytech/substrate-contracts-node.git --tag <latest-tag> --force --locked
```

사용할 최신 태그는 [Tags](https://github.com/paritytech/substrate-contracts-node/tags) 페이지에서 확인할 수 있습니다.

`substrate-contracts-node --version`을 실행하여 설치를 확인할 수 있습니다.

## 새로운 스마트 컨트랙트 프로젝트 생성하기

이제 ink! 스마트 컨트랙트 프로젝트를 개발하기 위해 준비가 되었습니다.

ink! 프로젝트 파일을 생성하려면 다음 단계를 따르세요:

1. 컴퓨터에서 터미널 셸을 엽니다.

2. 다음 명령을 실행하여 `flipper`라는 새 프로젝트 폴더를 생성합니다:

   ```bash
   cargo contract new flipper
   ```

3. 다음 명령을 실행하여 새 프로젝트 폴더로 이동합니다:

   ```bash
   cd flipper/
   ```

4. 다음 명령을 실행하여 디렉토리의 모든 내용을 나열합니다:

   ```bash
   ls -al
   ```

   디렉토리에 다음 파일이 포함되어 있는지 확인해야 합니다:

   ```text
   -rwxr-xr-x   1 dev-doc  staff   285 Mar  4 14:49 .gitignore
   -rwxr-xr-x   1 dev-doc  staff  1023 Mar  4 14:49 Cargo.toml
   -rwxr-xr-x   1 dev-doc  staff  2262 Mar  4 14:49 lib.rs
   ```

Rust 프로젝트와 마찬가지로 `Cargo.toml` 파일은 패키지 종속성 및 구성 정보를 제공하는 데 사용됩니다.

`lib.rs` 파일은 스마트 컨트랙트 비즈니스 로직에 사용됩니다.

### 기본 프로젝트 파일 살펴보기

기본적으로 ink! 프로젝트를 생성하면 매우 간단한 계약을 위한 일부 템플릿 소스 코드가 생성됩니다.

이 계약에는 `flip()` 함수와 `get()` 함수가 있습니다. `flip()` 함수는 부울 변수를 true에서 false로 변경하고, `get()` 함수는 현재 부울 값에 대한 값을 가져옵니다.

`lib.rs` 파일에는 계약이 예상대로 작동하는지 테스트하는 데 사용되는 두 개의 함수도 포함되어 있습니다.

튜토리얼을 진행하면서 시작 코드의 다른 부분을 수정하게 될 것입니다.
튜토리얼이 끝나면 더 고급스러운 스마트 컨트랙트를 갖게 될 것입니다. [여기](https://use.ink/examples/smart-contracts/)에서 더 많은 예제를 볼 수 있습니다.

기본 프로젝트 파일을 살펴보려면 다음 단계를 따르세요:

1. 필요한 경우 컴퓨터에서 터미널 셸을 엽니다.

2. `flipper` 스마트 컨트랙트를 위한 프로젝트 폴더로 이동합니다.

3. 텍스트 편집기에서 `Cargo.toml` 파일을 열고 계약에 대한 종속성을 검토합니다.

4. 텍스트 편집기에서 `lib.rs` 파일을 열고 계약에 대해 정의된 매크로, 생성자 및 함수를 검토합니다.

   - `#[ink::contract]` 매크로는 스마트 컨트랙트 로직의 진입점을 정의합니다.
   - `#[ink(storage)]` 매크로는 계약에 대한 단일 부울 값을 저장하는 구조체를 정의합니다.
   - `new` 및 `default` 함수는 부울 값을 false로 초기화합니다.
   - `#[ink(message)]` 매크로에는 데이터 저장을 위한 상태를 변경하는 `flip` 함수가 있습니다.
   - `#[ink(message)]` 매크로에는 데이터 저장을 위한 현재 상태를 가져오는 `get` 함수가 있습니다.

### 기본 계약 테스트하기

`lib.rs` 소스 코드 파일의 맨 아래에는 계약의 기능을 확인하기 위한 간단한 테스트 케이스가 있습니다.
이는 `#[ink(test)]` 매크로를 사용하여 주석 처리되어 있습니다. **오프체인 테스트 환경**을 사용하여 코드가 예상대로 작동하는지 테스트할 수 있습니다.

계약을 테스트하려면 다음 단계를 따르세요:

1. 필요한 경우 컴퓨터에서 터미널 셸을 엽니다.

2. `flipper` 계약을 위한 프로젝트 폴더에 있는지 확인합니다.

3. 다음 명령을 실행하여 `flipper` 계약의 기본 테스트를 실행합니다:

   ```bash
   cargo test
   ```

   이 명령은 프로그램을 컴파일하고 다음과 유사한 출력을 표시하여 테스트가 성공적으로 완료되었음을 나타냅니다:

   ```text
   running 2 tests
   test flipper::tests::it_works ... ok
   test flipper::tests::default_works ... ok

   test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
   ```

### 계약 빌드하기

기본 계약을 테스트한 후에는 이 프로젝트를 WebAssembly로 컴파일할 준비가 되었습니다.

이 스마트 컨트랙트를 위한 WebAssembly를 빌드하려면 다음 단계를 따르세요:

1. 필요한 경우 컴퓨터에서 터미널 셸을 엽니다.

2. `flipper` 프로젝트 폴더에 있는지 확인합니다.

3. 다음 명령을 실행하여 `flipper` 스마트 컨트랙트를 빌드합니다:

   ```bash
   cargo contract build
   ```

   이 명령은 `flipper` 프로젝트를 위한 WebAssembly 이진 파일, 계약의 Application Binary Interface (ABI)를 포함하는 메타데이터 파일, 계약을 배포하는 데 사용하는 `.contract` 파일을 빌드합니다.

   예를 들어, 다음과 유사한 출력이 표시됩니다:

   ```text
   Original wasm size: 35.5K, Optimized: 11.9K

   The contract was built in DEBUG mode.

   Your contract artifacts are ready. You can find them in:
   /Users/dev-doc/flipper/target/ink

   - flipper.contract (code + metadata)
   - flipper.wasm (the contract's code)
   - flipper.json (the contract's metadata)
   ```

   `.contract` 파일에는 비즈니스 로직과 메타데이터가 모두 포함되어 있습니다. 이 파일은 도구 (예: UI)에서 계약을 배포할 때 예상되는 파일입니다.

   `.json` 파일에는 호출할 수 있는 함수 (생성자 및 메시지) 및 발생하는 이벤트와 표시할 수 있는 모든 문서와 같은 함수에 대한 정보가 포함됩니다. 이 섹션에는 함수 이름의 4바이트 해시를 포함하는 `selector` 필드도 포함되어 있으며, 이는 계약 호출을 올바른 함수로 라우팅하는 데 사용됩니다.

   `storage` 섹션에서는 계약에서 관리하는 모든 저장소 항목과 해당 항목에 액세스하는 방법을 정의합니다.

   `types` 섹션에서는 계약에서 사용하는 사용자 정의 데이터 유형을 제공합니다.

## Substrate Contracts 노드 시작하기

[substrate-contracts-node를 성공적으로 설치](./prepare-your-first-contract.md#substrate-contracts-노드-설치하기)한 경우 로컬 노드를 시작할 시간입니다.

1. 다음 명령을 실행하여 로컬 개발 모드에서 계약 노드를 시작합니다:

   ```bash
   substrate-contracts-node --log info,runtime::contracts=debug 2>&1
   ```

   개발에 유용한 추가 로깅입니다.

   다음과 유사한 터미널 출력이 표시됩니다:

   ```text
   2023-01-30 23:08:49.835  INFO main sc_cli::runner: Substrate Contracts Node
   2023-01-30 23:08:49.836  INFO main sc_cli::runner: ✌️  version 0.23.0-87a3d76c880
   2023-01-30 23:08:49.836  INFO main sc_cli::runner: ❤️  by Parity Technologies <admin@parity.io>, 2021-2023
   2023-01-30 23:08:49.836  INFO main sc_cli::runner: 📋 Chain specification: Development
   2023-01-30 23:08:49.836  INFO main sc_cli::runner: 🏷  Node name: profuse-grandmother-6287
   2023-01-30 23:08:49.836  INFO main sc_cli::runner: 👤 Role: AUTHORITY
   2023-01-30 23:08:49.836  INFO main sc_cli::runner: 💾 Database: ParityDb at /tmp/substrateCu3FVo/chains/dev/paritydb/full
   2023-01-30 23:08:49.836  INFO main sc_cli::runner: ⛓  Native runtime: substrate-contracts-node-100 (substrate-contracts-node-1.tx1.au1)
   2023-01-30 23:08:54.570  INFO main sc_service::client::client: 🔨 Initializing Genesis block/state (state: 0x27d2…a1d8, header-hash: 0x6a05…1669)
   2023-01-30 23:08:54.573  INFO main sub-libp2p: 🏷  Local node identity is: 12D3KooWG4h1FpwAhybzyMxoEGQgY8SbrLb4F5FB6mCBZCY6u7W1
   2023-01-30 23:08:58.643  INFO main sc_service::builder: 📦 Highest known block at #0
   2023-01-30 23:08:58.643  INFO tokio-runtime-worker substrate_prometheus_endpoint: 〽️ Prometheus exporter started at 127.0.0.1:9615
   2023-01-30 23:08:58.644  INFO                 main sc_rpc_server: Running JSON-RPC HTTP server: addr=127.0.0.1:9933, allowed origins=None
   2023-01-30 23:08:58.644  INFO                 main sc_rpc_server: Running JSON-RPC WS server: addr=127.0.0.1:9944, allowed origins=None
   2023-01-30 23:09:03.645  INFO tokio-runtime-worker substrate: 💤 Idle (0 peers), best: #0 (0x6a05…1669), finalized #0 (0x6a05…1669), ⬇ 0 ⬆ 0
   2023-01-30 23:09:08.646  INFO tokio-runtime-worker substrate: 💤 Idle (0 peers), best: #0 (0x6a05…1669), finalized #0 (0x6a05…1669), ⬇ 0 ⬆ 0
   ```

   `substrate-contracts-node`은 `Manual Seal`을 사용하는 것이기 때문에 Extrinsic을 보내지 않는 한 블록이 생성되지 않습니다.

## 계약 배포하기

이 시점에서 다음 단계를 완료했습니다:

- 로컬 개발을 위한 패키지를 설치했습니다.
- `flipper` 스마트 컨트랙트를 위한 WebAssembly 이진 파일을 생성했습니다.
- 계약 노드를 개발 모드로 시작했습니다.

다음 단계는 Substrate 체인에 `flipper` 계약을 배포하는 것입니다.

그러나 Substrate에 스마트 컨트랙트를 배포하는 방법은 전통적인 스마트 컨트랙트 플랫폼과 약간 다릅니다.

대부분의 스마트 컨트랙트 플랫폼에서는 변경 사항을 적용할 때마다 완전히 새로운 스마트 컨트랙트 소스 코드 덩어리를 배포해야 합니다.

예를 들어, 표준 ERC20 토큰은 Ethereum에 수천 번 배포되었습니다.

변경 사항이 최소한이거나 초기 구성 설정에 영향을 미치는 경우에도 코드의 전체 재배포가 필요합니다.

각 스마트 컨트랙트 인스턴스는 실제로 코드가 변경되지 않았더라도 전체 계약 소스 코드에 해당하는 블록체인 리소스를 사용합니다.

Substrate에서는 계약 배포 프로세스가 두 단계로 나뉩니다:

- 계약 코드를 블록체인에 업로드합니다.
- 계약의 인스턴스를 생성합니다.

이러한 패턴을 사용하면 ERC20 표준과 같은 스마트 컨트랙트 코드를 한 번 블록체인에 저장한 다음 필요한 만큼 인스턴스를 생성할 수 있습니다.

동일한 소스 코드를 반복적으로 다시로드할 필요가 없으므로 스마트 컨트랙트가 블록체인에서 불필요한 리소스를 사용하지 않습니다.

### ink! 계약 코드 업로드하기

이 튜토리얼에서는 `cargo-contract` CLI 도구를 사용하여 Substrate 체인에 `flipper` 계약을 `upload`하고 `instantiate`합니다.

1. `substrate-contracts-node --log info,runtime::contracts=debug 2>&1`를 사용하여 노드를 시작합니다.

2. `flipper` 프로젝트 폴더로 이동합니다.

3. `cargo contract build`를 사용하여 계약을 빌드합니다.

4. 다음 명령을 사용하여 계약을 업로드하고 인스턴스를 생성합니다:

   ```bash
   cargo contract instantiate --constructor new --args "false" --suri //Alice --salt $(date +%s)
   ```

   명령에 대한 몇 가지 주의 사항:
   - `instantiate` 명령은 `upload`와 `instantiate` 단계를 모두 수행합니다.

   - 이 경우에는 `new()`를 사용하는 계약 생성자를 지정해야 합니다.

   - 이 경우에는 `false`를 인수로 지정해야 합니다.

   - 이 경우에는 계약을 업로드하고 인스턴스화하는 계정을 지정해야 합니다. 여기서는 기본 개발 계정인 `//Alice`입니다.

   - 개발 중에는 동일한 계약을 여러 번 업로드하고 인스턴스화할 수 있으므로 현재 시간을 사용하여 `salt`를 지정합니다. 이는 선택 사항입니다.

   명령을 실행한 후에는 가스 추정치에 만족하는지 확인하는 메시지가 표시됩니다:

   ```text
   Dry-running new (skip with --skip-dry-run)
      Success! Gas required estimated at Weight(ref_time: 328660939, proof_size: 0)
  Confirm transaction details: (skip with --skip-confirm)
   Constructor new
          Args false
     Gas limit Weight(ref_time: 328660939, proof_size: 0)
  Submit? (Y/n):
        Events
         Event Balances ➜ Withdraw
           who: 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
           amount: 98.986123μUNIT
         Event System ➜ NewAccount
           account: 5GRAVvuSXx8pCpRUDHzK6S1r2FjadahRQ6NEgAVooQ2bB8r5
         ... 생략 ...
         Event TransactionPayment ➜ TransactionFeePaid
           who: 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
           actual_fee: 98.986123μUNIT
           tip: 0UNIT
         Event System ➜ ExtrinsicSuccess
           dispatch_info: DispatchInfo { weight: Weight { ref_time: 2827629132, proof_size: 0 }, class: Normal, pays_fee: Yes }

      Contract 5GRAVvuSXx8pCpRUDHzK6S1r2FjadahRQ6NEgAVooQ2bB8r5
   ```

   `Contract` 주소를 기록해두어 계약을 `call`할 때 사용할 수 있도록 해야 합니다.


## 배포된 ink! 계약 호출하기

`cargo-contract`를 사용하여 `upload`와 `instantiate`뿐만 아니라 `call`도 할 수 있습니다!

### `get()` 메시지

계약을 초기화할 때 `flipper`의 초기 값을 `false`로 설정했습니다. 이를 확인하기 위해 `get()` 메시지를 호출할 수 있습니다.

블록체인 상태에서 읽기만 하는 경우 (새로운 데이터를 작성하지 않는 경우) `--dry-run` 플래그를 사용하여 Extrinsic을 제출하지 않도록 할 수 있습니다.

```bash
cargo contract call --contract $INSTANTIATED_CONTRACT_ADDRESS --message get --suri //Alice --dry-run
```

명령에 대한 몇 가지 주의 사항:
  - `--contract` 플래그를 사용하여 호출할 계약의 주소를 지정해야 합니다.

  - 이는 `cargo contract instantiate` 명령의 출력 로그에서 찾을 수 있습니다.

  - 호출하는 계약 메시지를 지정해야 합니다. 이 경우 `get()`입니다.

  - 계약을 호출하는 계정을 지정해야 합니다. 이 경우 기본 개발 계정인 `//Alice`입니다.

  - Extrinsic을 체인에 제출하지 않기 위해 `--dry-run`을 지정합니다.

명령을 실행한 후에는 다음과 유사한 출력이 표시됩니다:

```text
Result Success!
Reverted false
    Data Tuple(Tuple { ident: Some("Ok"), values: [Bool(false)] })
```

여기서 우리가 관심 있는 것은 `value`입니다. 예상대로 `false`입니다.

### `flip()` 메시지

`flip()` 메시지는 저장소 값을 `false`에서 `true`로 또는 그 반대로 변경합니다.

`flip()` 메시지를 호출하려면 블록체인의 상태를 변경하기 때문에 Extrinsic을 제출해야 합니다.

다음 명령을 사용할 수 있습니다:

```bash
cargo contract call --contract $INSTANTIATED_CONTRACT_ADDRESS --message flip --suri //Alice
```

메시지를 `flip`으로 변경하고 `--dry-run` 플래그를 제거했습니다.

명령을 실행한 후에는 다음과 유사한 출력이 표시됩니다:

```text
Dry-running flip (skip with --skip-dry-run)
    Success! Gas required estimated at Weight(ref_time: 8013742080, proof_size: 262144)
Confirm transaction details: (skip with --skip-confirm)
     Message flip
        Args
   Gas limit Weight(ref_time: 8013742080, proof_size: 262144)
Submit? (Y/n):
      Events
       Event Balances ➜ Withdraw
         who: 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
         amount: 98.974156μUNIT
       Event Contracts ➜ Called
         caller: 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
         contract: 5GQwxP5VTVHwJaRpoQsK5Fzs5cERYBzYhgik8SX7VAnvvbZS
       Event TransactionPayment ➜ TransactionFeePaid
         who: 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
         actual_fee: 98.974156μUNIT
         tip: 0UNIT
       Event System ➜ ExtrinsicSuccess
         dispatch_info: DispatchInfo { weight: Weight { ref_time: 1410915697, proof_size: 13868 }, class: Normal, pays_fee: Yes }
```

`get()` 메시지를 다시 호출하면 저장소 값이 실제로 뒤집혔음을 확인할 수 있습니다!

```text
Result Success!
Reverted false
    Data Tuple(Tuple { ident: Some("Ok"), values: [Bool(true)] })
```

## 다음 단계로 넘어가기

축하합니다!

이 튜토리얼에서 다음을 배웠습니다:

- ink! 스마트 컨트랙트 언어를 사용하여 새로운 스마트 컨트랙트 프로젝트를 생성하는 방법.

- 간단한 기본 스마트 컨트랙트를 테스트하고 WebAssembly 이진 파일로 빌드하는 방법.

- 계약 노드를 사용하여 작동하는 Substrate 기반 블록체인 노드를 시작하는 방법.

- 로컬 노드에 연결하고 계약을 업로드하고 인스턴스화하여 스마트 컨트랙트와 상호 작용하는 방법.

추가적인 스마트 컨트랙트 튜토리얼은 이 튜토리얼에서 배운 내용을 기반으로 하며 계약 개발의 다른 단계로 나아가게 됩니다.

다음 주제에서 스마트 컨트랙트 개발에 대해 더 자세히 알아볼 수 있습니다:

- [스마트 컨트랙트 개발](./develop-a-smart-contract.md)
- [ERC20 토큰 계약 빌드](./build-a-token-contract.md)
- [스마트 컨트랙트 문제 해결](./troubleshoot-smart-contracts.md)