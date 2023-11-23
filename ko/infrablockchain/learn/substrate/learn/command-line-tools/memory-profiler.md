---
title: 메모리 프로파일러
description: 메모리 프로파일러 프로그램에 대한 명령 줄 참조 정보입니다.
keywords:
---

메모리 프로파일링을 통해 Substrate 기반 클라이언트에서 블록체인 애플리케이션의 메모리 할당 및 동작을 시간에 따라 이해할 수 있습니다.
이는 메모리가 할당된 방식과 할당된 객체의 수를 결합하여 메서드 호출을 식별합니다.
또한 프로파일링은 메모리 누수를 분석하고 메모리 소비가 발생하는 위치를 식별하며, 임시 할당을 정의하고 애플리케이션 내에서 과도한 메모리 단편화를 조사하는 데 사용할 수 있습니다.

우리가 추천하는 프로파일러는 [koute의 메모리 프로파일러](https://github.com/koute/memory-profiler)입니다.

## 설치

프로파일러의 미리 컴파일된 바이너리 릴리스를 [릴리스](https://github.com/koute/memory-profiler/releases)에서 다운로드할 수 있습니다.
마지막으로 테스트한 버전은 0.6.1이지만, 그 이상의 버전도 대부분 작동할 것입니다.

다음은 명령 줄에서 다운로드하고 압축을 푸는 방법입니다:

```bash
$ curl -L https://github.com/koute/memory-profiler/releases/download/0.6.1/memory-profiler-x86_64-unknown-linux-gnu.tgz -o memory-profiler-x86_64-unknown-linux-gnu.tgz
$ tar -xf memory-profiler-x86_64-unknown-linux-gnu.tgz
```

이렇게 하면 세 개의 파일이 압축이 풀립니다. 우리는 두 개의 파일에만 관심이 있습니다:

- `libmemory_profiler.so` - 이것은 Substrate에 연결할 메모리 프로파일러 자체입니다.
- `memory-profiler-cli` - 이것은 프로파일링 데이터를 분석하는 데 사용할 프로그램입니다.

또한 [소스에서 프로파일러를 컴파일](https://github.com/koute/memory-profiler#building)할 수도 있습니다.

먼저 다음이 설치되어 있는지 확인해야 합니다:

- GCC 툴체인
- Rust nightly (버전 `nightly-2021-06-08`을 테스트했습니다)
- [Yarn](https://yarnpkg.com) 패키지 관리자 (GUI 빌드에 사용)

그런 다음 다음과 같이 프로파일러를 빌드할 수 있어야 합니다:

```bash
$ git clone https://github.com/koute/memory-profiler
$ cd memory-profiler
$ cargo build --release -p memory-profiler
$ cargo build --release -p memory-profiler-cli
```

필요한 바이너리 파일은 `target/release/libmemory_profiler.so`와 `target/release/memory-profiler-cli`에 있습니다.

## 명령 줄에서 프로파일링이 활성화된 노드 시작

명령 줄에서 Substrate을 수동으로 시작하는 경우 프로파일러를 연결하는 것은 몇 가지 추가 환경 변수를 설정한 다음 일반적으로 실행하는 것으로 줄어듭니다.

먼저 로깅을 활성화하고 프로파일링 로그를 출력할 위치와 프로파일링 데이터를 수집할 위치를 지정하고 필요한 경우 임시 할당을 제거하도록 프로파일러에 지시합니다:

```bash
$ export MEMORY_PROFILER_LOG=info
$ export MEMORY_PROFILER_LOGFILE=profiling_%e_%t.log
$ export MEMORY_PROFILER_OUTPUT=profiling_%e_%t.dat

# 필요에 따라 선택 사항입니다. 정확히 어떤 측면을 프로파일링하고 얼마 동안 프로파일링할지에 따라 다릅니다.
$ export MEMORY_PROFILER_CULL_TEMPORARY_ALLOCATIONS=1
```

그런 다음 프로파일러가 연결된 Substrate을 시작할 수 있습니다:

```bash
$ LD_PRELOAD=/path/to/libmemory_profiler.so ./target/release/substrate
```

`LD_PRELOAD` 환경 변수를 설정하면 Linux의 동적 링커가 Substrate을 시작하기 직전에 메모리 프로파일러를 주입하도록 지시하므로, 프로파일러는 시스템의 메모리 할당 루틴에 연결되어 Substrate이 수행하는 모든 메모리 할당을 추적할 수 있습니다.

## 원격 프로파일링이 활성화된 노드 시작

Substrate 기반 노드를 원격으로 실행하는 경우 아마도 `systemd`를 사용하여 관리하고 있을 것입니다.
이러한 경우 프로파일링을 설정하는 방법은 다음과 같습니다.

프로파일러의 미리 컴파일된 바이너리 파일을 이미 다운로드하거나 소스에서 컴파일하여 현재 디렉토리에 있는 것으로 가정합니다.

먼저, 메모리 프로파일러를 전역적으로 액세스할 수 있는 위치로 복사하고 로그를 작성하고 프로파일링 데이터를 수집할 수 있는 위치를 설정합니다.

```bash
$ sudo mkdir -p /opt/memory-profiler/bin
$ sudo cp libmemory_profiler.so /opt/memory-profiler/bin/
$ sudo mkdir /opt/memory-profiler/logs
$ sudo chmod 0777 /opt/memory-profiler/logs
```

그런 다음 프로파일러 자체를 구성하는 모든 환경 변수가 있는 파일을 설정합니다.

```bash
$ echo "MEMORY_PROFILER_OUTPUT=/opt/memory-profiler/logs/profiling_%e_%t_%p.dat" | sudo tee /opt/memory-profiler/env
$ echo "MEMORY_PROFILER_LOGFILE=/opt/memory-profiler/logs/profiling_%e_%t_%p.txt" | sudo tee -a /opt/memory-profiler/env
$ echo "MEMORY_PROFILER_LOG=info" | sudo tee -a /opt/memory-profiler/env
$ echo "MEMORY_PROFILER_CULL_TEMPORARY_ALLOCATIONS=1" | sudo tee -a /opt/memory-profiler/env
$ echo "LD_PRELOAD=/opt/memory-profiler/bin/libmemory_profiler.so" | sudo tee -a /opt/memory-profiler/env
```

이제 노드의 `systemd` 유닛 파일을 열고 `[Service]` 섹션에 다음을 추가합니다:

```text
[Service]
EnvironmentFile=/opt/memory-profiler/env
```

이미 `[Service]` 섹션이 있는 경우 다른 `[Service]` 섹션을 추가하지 마세요. 그냥 `EnvironmentFile` 키를 추가하면 됩니다.
이미 `EnvironmentFile` 키가 하나 있는 경우 대체하지 마세요. 두 번째 키를 추가하면 `systemd`가 둘 다 적용합니다.

이제 `systemd` 데몬을 다시 로드할 수 있습니다:

```bash
$ sudo systemctl daemon-reload
```

그런 다음 프로파일링을 시작하기 위해 서비스를 다시 시작합니다:

```bash
$ sudo systemctl restart kusama
```

프로파일링 데이터는 `/opt/memory-profiler/logs`에 수집됩니다. 메모리 프로파일러를 비활성화하려면 유닛 파일에 추가한 `EnvironmentFile` 키를 삭제하고 서비스를 다시 시작하면 됩니다.

## 프로파일러 구성

프로파일러를 구성하기 위해 설정할 수 있는 [다른 환경 변수](https://github.com/koute/memory-profiler#environment-variables-used-by-libmemory_profilerso)도 있습니다.
이미 변경해야 할 환경 변수를 제외하고는 일반적인 상황에서는 변경할 필요가 없습니다.

추가 고려해야 할 구성 옵션 중 하나는 `MEMORY_PROFILER_CULL_TEMPORARY_ALLOCATIONS`입니다.
이 옵션은 프로파일러가 짧은 수명을 가진 할당을 수집할지 여부를 제어합니다.

기본적으로 프로파일러는 프로파일링 대상 애플리케이션이 수행하는 **모든** 할당을 수집합니다.
이는 매우 많은 데이터이며 초당 _메가바이트_ 수준일 수 있습니다. 이는 짧은 시간 동안만 프로파일링하려는 경우나 특정한 임시 할당을 진단하려는 경우에는 좋지만, 더 오래 프로파일링을 실행하려는 경우에는 문제가 될 수 있습니다.

여기서 `MEMORY_PROFILER_CULL_TEMPORARY_ALLOCATIONS` 옵션이 필요합니다. `1`로 설정하여 켜면 프로파일러는 실제로는 매우 짧은 수명을 가진 할당을 생략하고 디스크에 기록하지 않습니다. 이렇게 하면 생성되는 데이터 양이 크게 줄어들어 일반적으로 초당 _킬로바이트_ 수준으로 줄어들게 되므로 프로파일링을 여러 일 동안 실행할 수 있습니다.

## 분석

프로파일링 데이터를 수집했으므로 이제 분석할 수 있습니다.

`memory-profiler-cli`와 수집한 `.dat` 파일이 동일한 디렉토리에 있는 경우 GUI를 로드할 수 있습니다:

```bash
$ ./memory-profiler-cli server *.dat
```

하드웨어 및 로드하려는 데이터 양에 따라 시간이 다소 걸릴 수 있습니다.
최종적으로 다음과 같은 내용이 출력되어야 합니다:

```text
[2020-05-06T08:59:20Z INFO  cli_core::loader] Loaded data in 315s 820
[2020-05-06T08:59:20Z INFO  actix_server::builder] Starting 8 workers
[2020-05-06T08:59:20Z INFO  actix_server::builder] Starting server on 127.0.0.1:8080
```

이제 웹 브라우저를 열고 `http://localhost:8080/`에서 GUI에 액세스할 수 있습니다.

![웹 UI의 그래프](/media/images/docs/reference/mem-profiling-memory-graph.png)

또한 데이터를 다른 형식으로 내보내거나 프로그래밍적으로 검사하려는 경우 [REST API](https://github.com/koute/memory-profiler#rest-api-exposed-by-memory-profiler-cli-server)를 사용할 수 있습니다.

## 기타 팁

- 프로파일러가 생성한 로그를 항상 확인하고 `WRN` 또는 `ERR` 로그가 있는지 확인하는 것이 좋습니다.

- 다음과 같은 오류 또는 경고 메시지를 볼 수 있습니다. 이는 실행 중인 Linux 배포판에 따라 다를 수 있습니다:

  ```text
  The perf_event_open syscall failed for PID 0: Operation not permitted (os error 1)`
  ```

  이는 일반적으로 해롭지 않습니다. 이로 인해 프로파일링할 때 CPU 사용량이 높아질 수 있습니다.
  다음과 같이 이를 피할 수 있습니다:

  ```bash
  $ echo "-1" | sudo tee /proc/sys/kernel/perf_event_paranoid
  ```

  그러나 이는 일부 보안 문제가 있을 수 있습니다.
  자세한 내용은 `man perf_event_open`을 참조하십시오.

- 분석 중에는 전체 데이터 파일을 메모리에 로드해야 합니다.
  충분한 RAM이 없는 경우 큰 파일을 로드하려고 하면 분석기가 메모리 부족으로 인해 충돌할 수 있습니다.