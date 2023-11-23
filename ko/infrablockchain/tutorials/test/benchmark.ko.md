---
title: 벤치마크
description: 런타임 로직의 함수를 실행하는 데 필요한 계산 리소스를 추정하기 위해 사용할 수 있는 벤치마킹 프레임워크를 설명합니다.
keywords:
  - 가중치
  - 리소스 소비
  - 실행 시간
---

Substrate와 FRAME은 블록체인에 대한 사용자 정의 로직을 개발하기 위한 유연한 프레임워크를 제공합니다.
이 유연성은 복잡하고 상호작용하는 팔렛을 설계하고 정교한 런타임 로직을 구현할 수 있도록 합니다.
그러나 팔렛의 함수에 할당할 적절한 [가중치](/ko/substrate/reference/glossary.ko.md/#weight)를 결정하는 것은 어려운 작업일 수 있습니다.
벤치마킹을 사용하면 런타임에서 다른 조건에서 다른 함수를 실행하는 데 걸리는 시간을 측정할 수 있습니다.
벤치마킹을 사용하여 함수 호출에 정확한 가중치를 할당하면, 악의적인 사용자에 의한 블록체인의 과부하로 인해 블록을 생성하지 못하거나 서비스 거부 (DoS) 공격에 취약해지는 것을 방지할 수 있습니다.

## 팔렛에 대한 벤치마크의 필요성

다른 함수를 실행하는 데 필요한 계산 리소스, 런타임 함수인 `on_initialize` 및 `verify_unsigned`를 포함하여 이해하는 것은 런타임의 안전성을 유지하고 리소스가 있는 경우에만 트랜잭션을 포함하거나 제외할 수 있도록 하는 데 중요합니다.

리소스가 있는 경우 트랜잭션을 포함하거나 제외할 수 있는 능력은 런타임이 서비스 중단 없이 블록을 계속 생성하고 가져올 수 있도록 합니다.
예를 들어, 특히 집중적인 계산을 필요로 하는 함수 호출이 있는 경우, 해당 호출을 실행하는 데 걸리는 시간이 블록 생성 또는 가져오기에 허용된 최대 시간을 초과할 수 있으므로 블록 처리 과정을 방해하거나 블록체인의 진행을 중지시킬 수 있습니다.
벤치마킹을 통해 다른 함수에 필요한 실행 시간이 합리적인 범위 내에 있는지 확인할 수 있습니다.

마찬가지로, 악의적인 사용자는 집중적인 계산을 필요로 하는 함수 호출을 반복적으로 실행하거나 필요한 계산을 정확하게 반영하지 않는 함수 호출을 실행하여 네트워크 서비스를 방해하려고 할 수 있습니다.
함수 호출을 실행하는 데 필요한 비용이 계산에 정확하게 반영되지 않는 경우, 악의적인 사용자가 네트워크를 공격하는 것을 방지할 동기가 없습니다.
벤치마킹을 통해 트랜잭션 실행에 연관된 가중치를 평가할 수 있으므로, 적절한 트랜잭션 수수료를 결정하는 데도 도움이 됩니다.
벤치마크를 기반으로 트랜잭션 실행에 소비되는 리소스를 나타내는 수수료를 설정할 수 있습니다.

## 선형 모델 개발

벤치마킹은 다음 단계를 수행해야 합니다.

- 특정 함수에 대한 특정 코드 경로를 실행하는 사용자 정의 벤치마킹 로직을 작성합니다.
- 특정 하드웨어 및 특정 런타임 구성을 사용하여 WebAssembly 실행 환경에서 벤치마킹 로직을 실행합니다.
- 함수 실행에 영향을 미칠 수 있는 가능한 값의 제어 범위에서 벤치마킹 로직을 실행합니다.
- 함수의 각 구성 요소에 대해 벤치마크를 여러 번 실행하여 이상치를 분리하고 제거합니다.

벤치마킹 로직을 실행하여 생성된 결과를 통해 벤치마킹 도구는 함수의 선형 모델을 생성합니다.
함수에 대한 선형 모델을 사용하면 특정 코드 경로를 실행하는 데 걸리는 시간을 추정하고 실제로 런타임에서 상당한 리소스를 소비하지 않고도 정보를 기반으로 결정할 수 있습니다.
벤치마킹은 모든 트랜잭션이 선형 복잡성을 가진다고 가정합니다. 더 높은 복잡성을 가진 함수는 런타임 상태나 입력이 너무 복잡해지면 이러한 함수의 가중치가 폭발할 수 있기 때문에 런타임에 위험하다고 간주됩니다.

## 벤치마킹과 가중치

[트랜잭션, 가중치 및 수수료](/ko/substrate/learn/tx-weights-fees.ko.md)에서 설명한 대로, Substrate 기반 체인은 블록의 트랜잭션을 실행하는 데 걸리는 시간을 나타내는 **가중치** 개념을 사용합니다.
특정 호출을 실행하는 데 필요한 시간은 다음과 같은 여러 요소에 따라 달라집니다.

- 계산 복잡성.
- 저장소 복잡성.
- 필요한 데이터베이스 읽기 및 쓰기 작업.
- 사용된 하드웨어.

트랜잭션에 적절한 가중치를 계산하려면, 벤치마크 매개변수를 사용하여 다른 하드웨어에서 다른 변수 값 및 여러 번 반복하여 함수 호출을 실행하는 데 걸리는 시간을 측정할 수 있습니다.
그런 다음 벤치마킹 테스트의 결과를 사용하여 각 함수 호출 및 각 코드 경로를 실행하는 데 필요한 리소스를 나타내는 근사적인 최악의 경우 가중치를 설정할 수 있습니다.
수수료는 최악의 경우 가중치를 기반으로 합니다.
실제 호출이 최악의 경우보다 성능이 우수한 경우, 가중치가 조정되고 초과 수수료가 반환될 수 있습니다.

가중치는 특정 물리적인 기계의 계산 시간을 기반으로 하는 일반적인 측정 단위이므로, 벤치마킹에 사용된 특정 하드웨어에 따라 함수의 가중치가 변경될 수 있습니다.

각 런타임 함수의 예상 가중치를 모델링함으로써 블록체인은 일정 기간 동안 얼마나 많은 트랜잭션 또는 시스템 수준 호출을 실행할 수 있는지 계산할 수 있습니다.

FRAME 내에서 디스패치할 수 있는 각 함수 호출은 해당 함수의 `#[weight]` 주석을 가져야 합니다. 벤치마킹 프레임워크는 이러한 공식을 자동으로 생성하는 파일을 생성합니다.

## 벤치마킹 도구

[벤치마킹 프레임워크](https://github.com/InfraBlockchain/infrablockspace-sdk/tree/master/substrate/frame/benchmarking)는 런타임의 함수에 대한 벤치마크를 추가, 테스트, 실행 및 분석하는 데 도움이 되는 도구를 제공합니다.
함수 호출을 실행하는 데 걸리는 시간을 결정하는 데 도움이 되는 벤치마킹 도구는 다음과 같습니다.

- [벤치마크 매크로](https://github.com/InfraBlockchain/infrablockspace-sdk/blob/master/substrate/frame/benchmarking/src/lib.rs)는 런타임 벤치마크를 작성, 테스트 및 추가하는 데 도움이 됩니다.
- [선형 회귀 분석 함수](https://github.com/InfraBlockchain/infrablockspace-sdk/blob/master/substrate/frame/benchmarking/src/analysis.rs)는 벤치마크 데이터를 처리하는 데 사용됩니다.
- [Command-line interface (CLI)](https://github.com/InfraBlockchain/infrablockspace-sdk/tree/master/substrate/utils/frame/benchmarking-cli)를 사용하여 노드에서 벤치마크를 실행할 수 있습니다.

노드를 컴파일할 때 벤치마킹 파이프라인은 기본적으로 비활성화됩니다.
벤치마크를 실행하려면 `runtime-benchmarks` Rust 기능 플래그를 사용하여 노드를 컴파일해야 합니다.

## 벤치마크 작성

런타임 벤치마크를 작성하는 것은 팔렛의 단위 테스트를 작성하는 것과 유사합니다.
단위 테스트와 마찬가지로 벤치마크는 코드의 특정 논리적 경로를 실행해야 합니다.
단위 테스트에서는 코드를 특정 성공 및 실패 결과에 대해 확인합니다.
벤치마크에서는 **가장 계산적으로 집중적인** 경로를 실행하려고 합니다.

벤치마크를 작성할 때 함수의 복잡성에 영향을 줄 수 있는 특정 조건 (예: 저장소 또는 런타임 상태)을 고려해야 합니다.
예를 들어, `for` 루프에서 더 많은 반복을 트리거하는 경우 데이터베이스 읽기 및 쓰기 작업의 수가 증가할 수 있으므로, 이 조건을 트리거하는 벤치마크를 설정하여 함수의 성능을 더 정확하게 나타낼 수 있습니다.

함수가 사용자 입력이나 다른 조건에 따라 다른 코드 경로를 실행하는 경우, 가장 계산적으로 집중적인 경로가 어떤 것인지 알 수 없을 수 있습니다.
코드의 복잡성이 관리하기 어려워질 수 있는 위치를 파악하기 위해 각 가능한 실행 경로에 대한 벤치마크를 생성해야 합니다.
벤치마크를 사용하여 사용자가 팔렛과 상호작용하는 방식을 제어하기 위해 벡터의 요소 수를 제한하거나 `for` 루프의 반복 횟수를 제한하는 등의 경계를 강제하는 코드의 위치를 식별할 수 있습니다.

모든 미리 구축된 [FRAME 팔렛](https://github.com/InfraBlockchain/infrablockspace-sdk/tree/master/substrate/frame)에서 종단간 벤치마크 예제를 찾을 수 있습니다.

## 벤치마크 테스트

팔렛을 단위 테스트하는 데 사용한 것과 동일한 모의 런타임을 사용하여 벤치마크를 테스트할 수 있습니다.
`benchmarking.rs` 모듈에서 사용하는 벤치마크 매크로는 자동으로 테스트 함수를 생성합니다.
예를 들어:

```rust
fn test_benchmark_[벤치마크_이름]<T>::() -> Result<(), &'static str>
```

벤치마크 함수를 단위 테스트에 추가하고 함수의 결과가 `Ok(())`인지 확인할 수 있습니다.

### 블록 확인

일반적으로 벤치마크가 `Ok(())`를 반환했는지만 확인하면 됩니다. 이 결과는 함수가 성공적으로 실행되었음을 나타냅니다.
그러나 벤치마크에 최종 상태와 같은 최종 조건을 확인하려는 경우 `verify` 블록을 선택적으로 포함할 수 있습니다.
추가 `verify` 블록은 최종 벤치마킹 프로세스의 결과에 영향을 주지 않습니다.

### 벤치마크가 포함된 단위 테스트 실행

벤치마킹 테스트를 실행하려면 테스트할 패키지를 지정하고 `runtime-benchmarks` 기능을 활성화해야 합니다.
예를 들어, 다음 명령을 실행하여 Balances 팔렛에 대한 벤치마크를 테스트할 수 있습니다.

```bash
cargo test --package pallet-balances --features runtime-benchmarks
```

## 벤치마크 추가

각 팔렛에 포함된 벤치마크는 자동으로 노드에 추가되지 않습니다.
이러한 벤치마크를 실행하려면 `frame_benchmarking::Benchmark` 트레이트를 구현해야 합니다.
이를 수행하는 방법에 대한 예제는 [Substrate 노드](https://github.com/InfraBlockchain/infrablockspace-sdk/blob/master/substrate/bin/node/runtime/src/lib.rs)에서 확인할 수 있습니다.

이미 노드에 일부 벤치마크가 설정되어 있는 경우, 팔렛을 `define_benchmarks!` 매크로에 추가하기만 하면 됩니다.

```rust
#[cfg(feature = "runtime-benchmarks")]
mod benches {
	define_benchmarks!(
		[frame_benchmarking, BaselineBench::<Runtime>]
		[pallet_assets, Assets]
		[pallet_babe, Babe]
    ...
    [pallet_mycustom, MyCustom]
    ...
```

팔렛을 추가한 후에는 `runtime-benchmarks` 기능 플래그를 사용하여 노드 바이너리를 컴파일합니다.
예를 들어:

```bash
cd bin/node/cli
cargo build --profile=production --features runtime-benchmarks
```

`production` 프로필은 다양한 컴파일러 최적화를 적용합니다.
이러한 최적화는 컴파일 프로세스를 _매우_ 느리게 만듭니다.
테스트 중이고 최종 숫자가 필요하지 않은 경우, `production` 프로필 대신 `--release` 커맨드 라인 옵션을 사용하십시오.

## 벤치마크 실행

벤치마크가 활성화된 노드 바이너리를 컴파일한 후에는 벤치마크를 실행해야 합니다.
노드를 컴파일할 때 `production` 프로필을 사용한 경우, 다음 명령을 실행하여 사용 가능한 벤치마크를 나열할 수 있습니다.

```bash
./target/production/node-template benchmark pallet --list
```

### 모든 팔렛의 모든 함수에 대한 벤치마크 실행

런타임의 모든 벤치마크를 실행하려면 다음과 유사한 명령을 실행할 수 있습니다.

```bash
./target/production/node-template benchmark pallet \
    --chain dev \
    --execution=wasm \
    --wasm-execution=compiled \
    --pallet "*" \
    --extrinsic "*" \
    --steps 50 \
    --repeat 20 \
    --output pallets/all-weight.rs
```

이 명령은 `all-weight.rs`라는 파일을 생성하여 런타임에 대한 `WeightInfo` 트레이트를 구현합니다.

### 특정 팔렛의 특정 함수에 대한 벤치마크 실행

특정 팔렛의 특정 함수에 대한 벤치마크를 실행하려면 다음과 유사한 명령을 실행할 수 있습니다.

```bash
./target/production/node-template benchmark pallet \
    --chain dev \
    --execution=wasm \
    --wasm-execution=compiled \
    --pallet pallet_balances \
    --extrinsic transfer \
    --steps 50 \
    --repeat 20 \
    --output pallets/transfer-weight.rs
```

이 명령은 `pallet_balances` 팔렛을 위한 `transfer-weight.rs`와 같은 선택한 팔렛을 위한 출력 파일을 생성하며, `WeightInfo` 트레이트를 구현합니다.

### 템플릿을 사용하여 벤치마크 서식 지정

벤치마크 명령줄 인터페이스는 최종 출력 파일을 서식 지정하기 위해 Handlebars 템플릿을 사용합니다.
기본값 대신 사용자 정의 템플릿을 지정하려면 `--template` 명령줄 옵션을 선택적으로 전달할 수 있습니다.
템플릿 내에서는 벤치마킹 명령줄 인터페이스의 `TemplateData` 구조체에서 제공하는 모든 데이터에 액세스할 수 있습니다.

출력 생성에 포함된 몇 가지 사용자 정의 Handlebars 도우미가 있습니다.

- `underscore`: 문자열의 오른쪽에서부터 3자리마다 밑줄을 추가합니다.
  주로 큰 숫자를 구분하기 위해 사용됩니다.
- `join`: 문자열 배열을 템플릿에 공백으로 구분된 문자열로 결합합니다.
  주로 CLI에 전달된 모든 인수를 결합하는 데 사용됩니다.

`benchmark` 하위 명령어의 전체 목록을 보려면 다음을 실행합니다.

```bash
./target/production/node-template benchmark --help
```

`benchmark pallet` 하위 명령어의 사용 가능한 옵션 목록을 보려면 다음을 실행합니다.

```bash
./target/production/node-template benchmark pallet --help
```

## 다음 단계로 넘어가기

- [frame-benchmarking README](https://github.com/InfraBlockchain/infrablockspace-sdk/blob/master/substrate/frame/benchmarking/README.md)
- [Substrate 세미나: Substrate 팔렛의 벤치마킹](https://www.youtube.com/watch?v=Qa6sTyUqgek)
- [How-to: 벤치마크 추가](/ko/substrate/reference/how-to-guides/weights/add-benchmarks.ko.md)
- [Command reference: node-template benchmark](/ko/substrate/reference/command-line-tools/node-template.ko.md/#benchmark)