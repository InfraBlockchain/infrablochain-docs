---
title: 노드 메트릭 모니터링
description: 관찰 도구를 사용하여 Substrate 노드에 대한 정보를 캡처하고 확인합니다.
keywords:
  - 관찰성
  - 메트릭
  - 노드 작업
---

Substrate은 네트워크 작업에 대한 메트릭을 노출합니다.
예를 들어, 노드가 연결된 피어 수, 노드가 사용하는 메모리 양, 생성되는 블록 수 등에 대한 정보를 수집할 수 있습니다.
Substrate 노드가 노출하는 메트릭을 캡처하고 시각화하기 위해 [Prometheus](https://prometheus.io/)와 [Grafana](https://grafana.com/)와 같은 도구를 구성하고 사용할 수 있습니다.
이 튜토리얼에서는 Prometheus를 사용하여 데이터 샘플을 수집하고, Grafana를 사용하여 데이터 샘플을 사용하여 노드 메트릭을 시각화하는 그래프와 대시보드를 생성하는 방법을 보여줍니다.

Substrate, Prometheus 및 Grafana 간의 상호 작용을 구성하여 노드 작업에 대한 정보를 표시하는 방법에 대한 단순화된 개요를 다음 다이어그램에서 제공합니다.

![Prometheus와 Grafana를 사용하여 노드 메트릭 시각화하기](/media/images/docs/tutorials/monitor-node-metrics/node-metrics.png)

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [Rust 및 Rust 툴체인](/install/)을 설치하여 Substrate 개발 환경을 구성했는지 확인하세요.

- [로컬 블록체인 구축](/tutorials/build-a-blockchain/build-local-blockchain/) 및 [네트워크 시뮬레이션](/tutorials/build-a-blockchain/simulate-network/)을 포함한 이전 튜토리얼 중 일부를 완료했는지 확인하세요.

## 튜토리얼 목표

이 튜토리얼을 완료함으로써 다음 목표를 달성할 수 있습니다:

- Prometheus와 Grafana를 설치합니다.
- Prometheus를 구성하여 Substrate 노드의 시계열 데이터를 캡처합니다.
- Grafana를 구성하여 Prometheus 엔드포인트를 사용하여 수집된 노드 메트릭을 시각화합니다.

## Prometheus와 Grafana 설치하기

테스트 및 데모 목적으로, 도구를 직접 빌드하거나 Docker 이미지를 사용하는 대신 Prometheus와 Grafana의 컴파일된 `bin` 프로그램을 다운로드하는 것이 좋습니다.
아키텍처에 맞는 바이너리를 다운로드하기 위해 다음 링크를 사용하세요.
이 튜토리얼에서는 작업 디렉토리에서 컴파일된 바이너리를 사용한다고 가정합니다.

이 튜토리얼에서 도구를 설치하려면 다음 단계를 따르세요:

1. 컴퓨터에서 브라우저를 엽니다.

2. [Prometheus 다운로드](https://prometheus.io/download/)에서 적절한 사전 컴파일된 바이너리를 다운로드합니다.

3. 컴퓨터의 터미널 쉘을 열고 다운로드한 파일에서 내용을 추출하기 위해 다음과 유사한 명령을 실행합니다.

   예를 들어, macOS에서는 다음과 유사한 명령을 실행할 수 있습니다:

   ```text
   gunzip prometheus-2.38.0.darwin-amd64.tar.gz && tar -xvf prometheus-2.38.0.darwin-amd64.tar
   ```

4. [Grafana OSS 다운로드](https://grafana.com/grafana/download?edition=oss)로 이동합니다.

5. 아키텍처에 맞는 사전 컴파일된 바이너리를 선택합니다.

6. 컴퓨터의 터미널 쉘을 열고 아키텍처에 맞는 명령을 실행하여 설치합니다.

   예를 들어, Homebrew가 설치된 macOS에서는 다음 명령을 실행할 수 있습니다:

   ```text
   brew update
   brew install grafana
   ```

## Substrate 노드 시작하기

Substrate은 포트 `9615`에서 [Prometheus 노출 형식](https://prometheus.io/docs/concepts/data_model/)으로 메트릭을 제공하는 엔드포인트를 노출합니다.
`--prometheus-port` 명령줄 옵션을 사용하여 포트를 변경하고, `--prometheus-external` 명령줄 옵션을 사용하여 로컬 호스트 이외의 인터페이스에서 액세스할 수 있도록 설정할 수 있습니다.
간단하게 설명하기 위해, 이 튜토리얼에서는 Substrate 노드, Prometheus 인스턴스 및 Grafana 서비스가 모두 로컬에서 기본 포트를 사용하여 실행되고 있다고 가정합니다.

1. 컴퓨터에서 터미널 쉘을 엽니다.

2. 필요한 경우 노드 템플릿 디렉토리의 루트로 이동합니다. 다음 명령을 실행하여 이동할 수 있습니다:

   ```bash
   cd substrate-node-template
   ```

3. 다음 명령을 실행하여 개발 모드에서 노드 템플릿을 시작합니다:

   ```bash
   ./target/release/node-template --dev
   ```

## Prometheus 엔드포인트 구성하기

Prometheus 다운로드를 추출할 때 생성된 디렉토리에는 `prometheus.yml` 구성 파일이 포함되어 있습니다.
이 파일을 수정하거나 사용자 정의 구성 파일을 생성하여 Prometheus를 구성하여 기본 Prometheus 포트 엔드포인트(포트 `9615`)에서 데이터를 가져올 수 있습니다.
`--prometheus-port <포트 번호>` 명령줄 옵션을 사용하여 지정한 포트로 변경할 수도 있습니다.

Substrate 노출된 엔드포인트를 Prometheus 대상 목록에 추가하려면 다음 단계를 수행하세요:

1. 컴퓨터에서 새로운 터미널 쉘을 엽니다.

2. Prometheus 작업 디렉토리의 루트로 이동합니다.

3. 텍스트 편집기에서 `prometheus.yml` 구성 파일을 엽니다.

4. `scrape_config` 엔드포인트로 `substrate_node`를 추가합니다.

   예를 들어, 다음과 유사한 섹션을 추가합니다:

   ```yml
    # A scrape configuration containing exactly one endpoint to scrape:
    scrape_configs:
      - job_name: "substrate_node"

        scrape_interval: 5s

        static_configs:
          - targets: ["localhost:9615"]
   ```

    이 설정은 `substrate_node` 작업에 대한 스크래핑 대상의 전역 기본값을 재정의합니다.
    블록 시간보다 작은 값을 `scrape_interval`로 설정하여 모든 블록에 대한 데이터 포인트가 있는지 확인하는 것이 중요합니다.
    예를 들어, Polkadot 블록 시간은 6초이므로 `scrape_interval`을 5초로 설정합니다.

1. 수정된 `prometheus.yml` 구성 파일로 Prometheus 인스턴스를 시작합니다.

   예를 들어, 현재 Prometheus 작업 디렉토리에 있고 기본 구성 파일 이름을 사용하는 경우 다음 명령을 실행하여 Prometheus를 시작합니다:

   ```bash
   ./prometheus --config.file prometheus.yml
   ```

   이 프로세스를 실행한 상태로 둡니다.

2. 새로운 터미널 쉘을 열고 다음 명령을 실행하여 Substrate 노드에 대한 메트릭을 검색합니다:

   ```bash
   curl localhost:9615/metrics
   ```

   이 명령은 다음과 유사한 출력을 반환합니다(일부 예시만 표시됨):

   ```text
   # HELP substrate_block_height Block height info of the chain
   # TYPE substrate_block_height gauge
   substrate_block_height{status="best",chain="dev"} 16
   substrate_block_height{status="finalized",chain="dev"} 14
   substrate_block_height{status="sync_target",chain="dev"} 16
   # HELP substrate_block_verification_and_import_time Time taken to verify and import blocks
   # TYPE substrate_block_verification_and_import_time histogram
   substrate_block_verification_and_import_time_bucket{chain="dev",le="0.005"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="0.01"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="0.025"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="0.05"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="0.1"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="0.25"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="0.5"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="1"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="2.5"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="5"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="10"} 0
   substrate_block_verification_and_import_time_bucket{chain="dev",le="+Inf"} 0
   substrate_block_verification_and_import_time_sum{chain="dev"} 0
   substrate_block_verification_and_import_time_count{chain="dev"} 0
   # HELP substrate_build_info A metric with a constant '1' value labeled by name, version
   # TYPE substrate_build_info gauge
   substrate_build_info{name="ruddy-afternoon-1788",version="4.0.0-dev-6a8b2b12371",chain="dev"} 1
   # HELP substrate_database_cache_bytes RocksDB cache size in bytes
   # TYPE substrate_database_cache_bytes gauge
   substrate_database_cache_bytes{chain="dev"} 0
   # HELP substrate_finality_grandpa_precommits_total Total number of GRANDPA precommits cast locally.
   # TYPE substrate_finality_grandpa_precommits_total counter
   substrate_finality_grandpa_precommits_total{chain="dev"} 76
   # HELP substrate_finality_grandpa_prevotes_total Total number of GRANDPA prevotes cast locally.
   # TYPE substrate_finality_grandpa_prevotes_total counter
   substrate_finality_grandpa_prevotes_total{chain="dev"} 77
   # HELP substrate_finality_grandpa_round Highest completed GRANDPA round.
   # TYPE substrate_finality_grandpa_round gauge
   substrate_finality_grandpa_round{chain="dev"} 76
   ...
   ```

   또는 동일한 엔드포인트를 브라우저에서 열어 사용 가능한 모든 메트릭 데이터를 확인할 수 있습니다.
   예를 들어, 기본 Prometheus 포트를 사용하는 경우 브라우저에서 [`http://localhost:9615/metrics`](http://localhost:9615/metrics)를 엽니다. <!-- markdown-link-check-disable-line -->

## Grafana 데이터 소스 구성하기

아키텍처에 맞는 명령을 실행하여 Grafana를 설치한 후, 로컬 컴퓨터에서 서비스를 시작하여 사용할 수 있습니다.
서비스를 시작하는 데 사용되는 명령은 로컬 시스템 아키텍처 및 패키지 관리자에 따라 다릅니다.
예를 들어, macOS와 Homebrew를 사용하는 경우 다음 명령을 실행하여 Grafana를 시작할 수 있습니다:

```bash
brew services start grafana
```

다른 운영 체제에서 Grafana를 시작하는 방법에 대한 자세한 내용은 해당하는 [Grafana 문서](https://grafana.com/docs/grafana/v9.0/)를 참조하세요.

Grafana를 시작한 후, 브라우저에서 Grafana로 이동할 수 있습니다.

1. 브라우저를 열고 Grafana가 사용하는 포트로 이동합니다.

   기본적으로 Grafana는 http://localhost:3000을 사용하지만, 호스트 또는 포트를 다른 값으로 구성한 경우 해당 값을 사용합니다.<!-- markdown-link-check-disable-line -->

2. 기본 `admin` 사용자 이름과 비밀번호 `admin`을 사용하여 로그인한 다음 **Log in**을 클릭합니다.

3. 환영 페이지에서 **Configuration** 메뉴 아래에 있는 **Data Sources**를 클릭합니다.

1. **Prometheus**를 클릭하여 Prometheus 엔드포인트를 Substrate 노드 메트릭의 데이터 소스로 구성합니다.

   Substrate 노드와 Prometheus 인스턴스가 모두 실행 중인 경우, Grafana를 기본 포트 `http://localhost:9090` 또는 사용자 정의한 포트 정보를 구성한 경우 해당 포트에 Prometheus를 찾도록 구성합니다.

   `prometheus.yml` 파일에서 설정한 Prometheus 포트를 지정하지 않아야 합니다.
   해당 포트는 노드가 데이터를 게시하는 포트입니다.

2. **Save & Test**를 클릭하여 데이터 소스가 올바르게 설정되었는지 확인합니다.

   데이터 소스가 작동하는 경우, 노드 메트릭을 표시할 대시보드를 구성할 준비가 되었습니다.

## 템플릿 대시보드 가져오기

시작하기 위해 기본 대시보드가 필요한 경우, [Substrate 대시보드 템플릿](/assets/tutorials/monitor-node/substrate-node-template-metrics.json)을 가져와 노드에 대한 기본 정보를 얻을 수 있습니다.

대시보드 템플릿을 가져오려면 다음 단계를 수행하세요:

1. Grafana 환영 페이지에서 **Dashboards**를 클릭합니다.

1. 왼쪽 탐색에서 **Dashboards**를 클릭하고 **Browse**를 선택합니다.

2. 검색 옵션에 대한 **New**를 클릭하고 **Import**를 선택합니다.

3. [Substrate 대시보드 템플릿](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/tutorials/monitor-node/substrate-node-template-metrics.json)을 복사하여 **Import via panel json** 텍스트 상자에 붙여넣습니다.

4. **Load**를 클릭합니다.

5. 대시보드의 이름, 폴더 및 고유 식별자를 검토하고 필요한 경우 수정합니다.

6. **Prometheus (default)**를 선택한 다음 **Import**를 클릭합니다.

   ![Substrate 대시보드 템플릿](/media/images/docs/tutorials/monitor-node-metrics/grafana-template-dashboard.png)

   [Substrate 대시보드 템플릿](https://grafana.com/grafana/dashboards/13759/)은 Substrate 기반 체인에서 사용할 수 있으며, Grafana Labs 대시보드 갤러리에서도 다운로드할 수 있습니다.

   사용자 정의 대시보드를 만들고 싶다면 [Prometheus 문서의 Grafana 섹션](https://prometheus.io/docs/visualization/grafana/)을 참조하세요.

   사용자 정의 대시보드를 만든 경우, [Grafana 대시보드](https://grafana.com/grafana/dashboards)에 업로드하는 것을 고려하세요.
   [Awesome Substrate](https://github.com/substrate-developer-hub/awesome-substrate) 리포지토리에 대시보드가 있는지 나열하여 Substrate 빌더 커뮤니티에 알릴 수 있습니다.

## 다음 단계

- [신뢰할 수 있는 노드 추가](/tutorials/build-a-blockchain/add-trusted-nodes)
- [노드 모니터링](https://wiki.polkadot.network/docs/en/maintain-guides-how-to-monitor-your-node)
- [Substrate Prometheus Exporter](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/utils/prometheus)
- [Polkadot 네트워크 대시보드](https://github.com/w3f/polkadot-dashboard)