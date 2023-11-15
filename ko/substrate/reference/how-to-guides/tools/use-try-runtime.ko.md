---
title: try-runtime 사용하기
description: try-runtime 을 사용하여 스토리지 마이그레이션을 테스트하는 방법
keywords:
  - 스토리지 마이그레이션
  - 테스트
  - 런타임
  - 도구
---

`try-runtime` 도구를 사용하면 런타임을 프로덕션에 배포하기 전에 시뮬레이션 환경에서 테스트하고 작업을 확인할 수 있습니다.
이 가이드에서는 `try-runtime` 도구를 런타임에 통합하는 기본 단계를 보여줌으로써 스토리지 마이그레이션을 테스트하는 데 사용할 수 있도록 합니다.

일반적으로 `try-runtime` 도구를 런타임에 추가하는 것은 팔레트를 가져오는 것과 유사합니다.
적절한 종속성을 올바른 위치에 추가하고 런타임 로직을 `try-runtime` 기능을 포함하도록 업데이트하는 것이 포함됩니다.
팔레트와 마찬가지로 종속성을 추가할 때 적절한 태그 또는 브랜치를 사용하여 `try-runtime` 도구에 대한 종속성을 추가하는지 확인하십시오.

## 런타임 종속성 추가

1. 터미널 쉘을 열고 노드 템플릿의 루트 디렉토리로 이동합니다.

2. 텍스트 편집기에서 `runtime/Cargo.toml` 구성 파일을 엽니다.

3. [dependencies] 섹션을 찾고 다른 팔레트가 가져오는 방법을 확인합니다.

4. `frame-try-runtime` 종속성을 추가합니다:

   ```toml
   [dependencies]
   frame-try-runtime = { git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-v1.0.0", optional = true }
   ```

   다른 팔레트가 서로 호환되도록 가져온 팔레트와 동일한 브랜치와 버전 정보를 사용하는지 확인하십시오.
   다른 브랜치에서 팔레트를 사용하면 컴파일러 오류가 발생할 수 있습니다.
   이 예제는 다른 팔레트가 `branch = "polkadot-v1.0.0"`를 사용하는 경우 `Cargo.toml` 파일에 `frame-try-runtime` 팔레트를 추가하는 방법을 보여줍니다.

5. `try-runtime-cli` 종속성을 추가합니다:

   ```toml
   try-runtime-cli = { git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-v1.0.0", optional = true }
   ```

6. 표준 기능 목록에 `frame-try-runtime`을 추가합니다:

   ```toml
   [features]
   default = ["std"]
   std = [
     "codec/std",
     "scale-info/std",
     "frame-try-runtime/std",
     ...
   ]
   ```

7. `[features]` 섹션에 `try-runtime`을 추가하거나 업데이트하여 런타임의 모든 팔레트를 포함하도록 합니다.

   ```toml
   try-runtime = [
     "frame-executive/try-runtime",
     "frame-try-runtime",
     "frame-system/try-runtime",
     "pallet-aura/try-runtime",
     "pallet-balances/try-runtime",
     "pallet-nicks/try-runtime",
     "pallet-grandpa/try-runtime",
     "pallet-randomness-collective-flip/try-runtime",
     "pallet-sudo/try-runtime",
     "pallet-template/try-runtime",
     "pallet-timestamp/try-runtime",
     "pallet-transaction-payment/try-runtime",
   ]
   ```

## 런타임 API에 try-runtime 구현하기

1. try-runtime 기능을 위한 구성 블록을 추가합니다.

   ```rust
   // 런타임 API 구현
   impl_runtime_apis! {
   ...
     /* --snip-- */
     #[cfg(feature = "try-runtime")]
     impl frame_try_runtime::TryRuntime<Block> for Runtime {
         fn on_runtime_upgrade() -> (frame_support::weights::Weight, frame_support::weights::Weight) {
             log::info!("try-runtime::on_runtime_upgrade.");
             // 참고: 의도적인 언랩: 오류를 역방향으로 전파하지 않고 여기에서 백트레이스를 가지려고 합니다.
             // 사전/사후 마이그레이션 검사 중에 오류가 발생하면 여기에서 즉시 중단합니다.
             let weight = Executive::try_runtime_upgrade().map_err(|err|{
                 log::info!("try-runtime::on_runtime_upgrade failed with: {:?}", err);
                 err
             }).unwrap();
             (weight, RuntimeBlockWeights::get().max_block)
         }
         fn execute_block_no_check(block: Block) -> frame_support::weights::Weight {
             Executive::execute_block_no_check(block)
         }
     }
   }
   ```

## 노드 종속성 추가

1. 텍스트 편집기에서 `node/Cargo.toml` 구성 파일을 엽니다.

2. [dependencies] 섹션을 찾고 다른 팔레트가 가져오는 방법을 확인합니다.

3. `frame-try-runtime` 종속성을 추가합니다:

   ```toml
   frame-try-runtime = { git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-v1.0.0", optional = true }
   ```

4. `try-runtime-cli` 종속성을 추가합니다:

   ```toml
   try-runtime-cli = { git = "https://github.com/paritytech/polkadot-sdk.git", branch = "polkadot-v1.0.0", optional = true }
   ```

5. `[features]` 섹션에 `cli`와 `try-runtime`을 추가하거나 업데이트합니다.

   ```toml
   [features]
   ...
   cli = [ "try-runtime-cli" ]
   try-runtime = [ "node-template-runtime/try-runtime", "try-runtime-cli" ]
   ...
   ```

## 명령줄 하위 명령 추가

1. 텍스트 편집기에서 `node/src/cli.rs` 파일을 엽니다.

```rust
/* --snip-- */
/// 런타임 상태에 대해 명령을 시도합니다.
#[cfg(feature = "try-runtime")]
TryRuntime(try_runtime_cli::TryRuntimeCmd),

/// 런타임 상태에 대해 명령을 시도합니다. 참고: `try-runtime` 기능이 활성화되어 있어야 합니다.
#[cfg(not(feature = "try-runtime"))]
TryRuntime,
/* --snip-- */
```

## 명령 추가

1. 텍스트 편집기에서 `node/src/commands.rs` 파일을 엽니다.

2. 명령을 추가합니다.

   ```rust
   /* --snip-- */
   #[cfg(feature = "try-runtime")]
   Some(Subcommand::TryRuntime(cmd)) => {
     let runner = cli.create_runner(cmd)?;
     runner.async_run(|config| {
       // `async_run`을 수행하기 위해서는 런타임 또는 태스크 관리자만 필요합니다.
       let registry = config.prometheus_config.as_ref().map(|cfg| &cfg.registry)let task_manager = sc_service::TaskManager::new(
         config.tokio_handle.clone(),
         registry,
       ).map_err(|e| sc_cli::Error::Service(sc_service::Error::Prometheus(e)))?;
       
       Ok((cmd.run::<Block, service::ExecutorDispatch>(config), task_manager))
     })
   },
   
   #[cfg(not(feature = "try-runtime"))]
   Some(Subcommand::TryRuntime) => {
     Err("TryRuntime wasn't enabled when building the node. \
     You can enable it with `--features try-runtime`.".into())
   },
   /* --snip-- */
   ```

작업 공간에서 사용자 정의 팔레트를 사용하는 경우 작업 공간의 `pallets/pallet_name/Cargo.toml` 파일의 종속성에 `try-runtime`을 포함했는지 확인하십시오.

## try-runtime 사용하기

try-runtime 도구를 사용하는 방법은 유닛 테스트를 작성하는 것과 유사합니다.

`try-runtime`을 사용하려면 다음을 수행합니다:

1. Externalities 인스턴스를 생성합니다.

2. 인스턴스에서 `execute_with`를 호출합니다.

<!--
## 예제

## 자원
-->

## 다음으로 이동할 위치

- [Command-line reference: try-runtime](/reference/command-line-tools/try-runtime/)
- [TryRuntime API](https://crates.parity.io/frame_try_runtime/trait.TryRuntime.html)
