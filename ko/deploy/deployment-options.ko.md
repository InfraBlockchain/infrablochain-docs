---
title: 배포 옵션과 도구
description:
keywords:
---

[공통 배포 대상](/deploy/prepare-to-deploy#common-deployment-targets)에서 볼 수 있듯이, 일반적인 배포 시나리오가 여러 가지 있습니다.
제공하는 서비스의 노드를 관리하는 방법은 물리적 하드웨어에 배포하는지, 클라우드 공급자를 사용하는지, Kubernetes 클러스터를 구성하는지에 따라 많이 달라집니다.
이 섹션에서는 다양한 환경에서 노드를 배포하고 관리하기 위한 가장 일반적인 옵션과 도구에 대해 설명합니다.

## Linux 서버

리눅스 운영 체제의 배포 또는 가상 머신에 배포하는 경우, 대부분의 서비스를 관리하기 위해 `systemd`를 사용합니다.
`systemd`를 사용하여 프로세스가 활성화되고 실행되도록하고, 서비스 재시작 정책을 설정하고, 호스트에서 실행할 사용자 계정을 지정하고, 메모리 사용량 및 기타 속성을 제한하는 시스템 매개변수를 구성할 수 있습니다.

다음 예제는 Alice와 연결된 계정 및 로컬 사용자 이름 `infrablockspace`을 사용하여 로컬 개발 체인을 실행하는 노드의 `systemd` 파일을 구성하는 방법을 보여줍니다:

```text
[Unit]
Description=InfraBlockspace Validator

[Service]
User=infrablockspace
ExecStart=/home/infrablockspace/infrablockspace  --dev --alice
Restart=always
RestartSec=90

[Install]
WantedBy=multi-user.target
```

데모 목적으로 이 파일의 이름은 `infrablockspace.service`이며 `/etc/systemd/system` 디렉토리에 위치시킵니다.
다음 명령을 실행하여 서비스를 활성화할 수 있습니다:

```bash
systemctl enable infrablockspace
```

활성화된 후 서비스를 시작하려면 다음 명령을 실행할 수 있습니다:

```bash
systemctl start infrablockspace
```

### 환경 변수 사용

`systemd` 구성에서 일부 설정을 제거하려면 예를 들어 `--dev` 및 `--alice` 커맨드 옵션을 구성하는 대신 **환경 변수** 파일에 해당 설정을 구성할 수 있습니다.
환경 변수 파일을 사용하면 각 서버에 대한 적절한 설정을 구성할 수 있지만, 여전히 `systemd` 명령을 사용하여 서비스를 관리할 수 있습니다.
이 예제에서는 `/etc/default/infrablockspace`에 로컬 호스트를 위한 새로운 환경 변수 파일을 만들고 다음과 같이 구성합니다:

```text
START_OPTIONS="--dev --alice"
```

그런 다음 `systemd` 서비스를 다음과 같이 수정합니다:

```text
[Unit]
Description=InfraBlockspace Validator

[Service]
User=infrablockspace
EnvironmentFile=/etc/default/infrablockspace
ExecStart=/home/infrablockspace/infrablockspace  $START_OPTIONS
Restart=always
RestartSec=90

[Install]
WantedBy=multi-user.target
```

이 기법을 사용하여 여러 변수와 함께 구성 세부 정보를 `systemd` 파일에서 추상화할 수 있습니다.

### 로컬 로깅

기본적으로 `systemd` 서비스는 로컬 `syslog` 파일(`/var/log/syslog` 또는 `/var/log/messages`)에 출력을 작성합니다.
또한 `journalctl` 명령을 사용하여 이 출력을 볼 수도 있습니다.
예를 들어, `infrablockspace` 프로세스의 가장 최근 출력을 보려면 다음 명령을 실행할 수 있습니다:

```bash
journalctl -u infrablockspace -f
```

2일 전보다 오래된 로그를 제거하려면 다음과 유사한 명령을 실행할 수 있습니다:

```bash
journalctl -u infrablockspace --vacuum-time=2d
```

과거 1G의 데이터만 유지하려면 다음을 실행합니다:

```bash
journalctl --vacuum-size=1G
```

### 원격 로깅

배포에 여러 호스트가 포함된 경우 Loki 또는 Elasticsearch를 사용하여 여러 소스에서 데이터를 집계할 수 있습니다.

#### Loki

원격 `loki` 인스턴스에 로그를 기록하려면 다음 단계를 수행합니다:

1. 각 서버에 `promtail` 서버 패키지를 설치합니다.
2. 서버가 로그를 원격 호스트로 보낼 수 있도록 서버 및 클라이언트 정보를 지정하는 구성 파일을 작성합니다.

   예를 들면 다음과 같습니다:

   ```yaml
   # promtail 서버 구성
   server:
     http_listen_port: 9080
     grpc_listen_port: 0
     log_level: info
   positions:
     filename: /var/lib/promtail/positions.yaml

   # loki 서버
   clients:
     - url: http://myloki.mycompany.com/loki/api/v1/push
       backoff_config:
         min_period: 1m
         max_period: 1h
         max_retries: 10000
   scrape_configs:
     - job_name: journald
       journal:
         max_age: 1m
         path: /var/log/journal
         labels:
           job: journald
       pipeline_stages:
         - match:
           selector: '{job="journald"}'
           stages:
             - multiline:
                 firstline: '^\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}\.\d{3}'
                 max_lines: 2500
             - regex:
                 expression: '(?P<date>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}\.\d{3})\s+(?P<level>(TRACE|DEBUG|INFO|WARN|ERROR))\s+(?P<worker>([^\s]+))\s+(?P<target>[\w-]+):?:?(?P<subtarget>[\w-]+)?:[\s]?(?P<chaintype>\[[\w-]+\]+)?[\s]?(?P<message>.+)'
             - labels:
                 level:
                 target:
                 subtarget:
   ```

#### Elasticsearch

원격 Elasticsearch 클러스터에 로그를 기록하려면 다음 단계를 수행합니다:

1. `logstash` 패키지를 설치합니다.
2. 서버 및 클라이언트 정보를 지정하는 구성 파일을 작성하여 각 서버가 로그를 원격 호스트로 보낼 수 있도록합니다.
   다음은 예제 구성 파일입니다:

   ```text
   nput {
     journald {
       path      => "/var/log/journal"
       seekto => "tail"
       thisboot => true
       filter    => {
           "_SYSTEMD_UNIT" => "infrablockspace.service"
       }
       type => "systemd"
     }
   }

   filter {
     date {
       match => ["timestamp", "YYYY-mm-dd HH:MM:ss.SSS"]
       target => "@timestamp"
     }
     mutate {
       rename => [ "MESSAGE", "message" ]
       remove_field => [ "cursor", "PRIORITY", "SYSLOG_FACILITY", "SYSLOG_IDENTIFIER", "_BOOT_ID", "_CAP_EFFECTIVE", "_CMDLINE", "_COMM", "_EXE", "_GID", "_HOSTNAME", "_MACHINE_ID", "_PID", "_SELINUX_CONTEXT", "_STREAM_ID", "_SYSTEMD_CGROUP", "_SYSTEMD_INVOCATION_ID", "_SYSTEMD_SLICE", "_SYSTEMD_UNIT", "_TRANSPORT", "_UID" ]
     }
     if ([message] =~ ".*TRACE .*") { drop{ } }
     grok {
        match => { "message" => "%{NOTSPACE:thread} %{LOGLEVEL:log-level} %{NOTSPACE:namespace} %{GREEDYDATA:message}" }
     }
   }

   output {
      elasticsearch {
        hosts => ["https://myelasticsearch.mycompany.com:9243"]
        user => "username"
        password => "password"
        index => "logstash-infrablockspace-%{+YYYY.MM.dd}"
      }
   }
   ```

### 로깅 커맨드 옵션

노드를 시작할 때 로그 레벨과 로그 활동을 기록할 대상 구성을 지정하기 위해 커맨드 옵션을 사용할 수 있습니다.
모든 대상 구성은 기본적으로 `info` 로깅 레벨로 설정됩니다.
`--log` 또는 `-l` 커맨드 옵션을 사용하여 개별 구성 요소의 로그 레벨을 조정할 수 있습니다.
예를 들어, afg 및 sync 구성 요소의 로깅 레벨을 변경하려면 다음과 같이 입력합니다:

```bash
--log afg=trace,sync=debug
```

모든 구성 요소의 로깅 레벨을 `debug`로 변경하려면 다음과 같이 입력합니다:

```bash
-ldebug.
```

가장 적은 상세도에서 가장 자세한 상세도까지 유효한 로그 레벨은 `error`, `warn`, `info`, `debug`, `trace`입니다.

로그 기록 대상은 다음과 같습니다:

```text
afg
aura
babe
beefy
db
gossip
header
peerset
pow
rpc
runtime
runtime::contracts
sc_offchain
slots
state-db
state_tracing
sub-libp2p
sync
telemetry
tracing
trie
txpool
```

## 클라우드 프로비저닝(Cloud Provisioning)

클라우드 공급자에서 노드를 프로비저닝하는 여러 가지 옵션이 있습니다.
클라우드 리소스를 사용하여 배포하는 도구 중 일부는 공급자별로 특정되어 있고, 일부 도구는 공급자에 독립적입니다.

AWS, Microsoft Azure 또는 Google Cloud에서 배포하는 데 가장 일반적으로 사용되는 공급자별 도구는 다음과 같습니다:

- [Amazon Cloud Formation](https://aws.amazon.com/cloudformation/)
- [Azure Resource Manager](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/overview)
- [Google Cloud Deployment Manager](https://cloud.google.com/deployment-manager/docs)

이러한 공급자별 배포 도구는 사용하기 쉽고, 샘플 코드, 문서 및 지원과 같은 중요한 리소스를 제공합니다.
그러나 여러 공급자를 사용하는 경우 각각의 스크립팅 형식과 구성 요구 사항이 다르기 때문에 기본 인프라에 심지어 기본적인 변경을 하려면 모든 공급자에 대해 코드의 여러 섹션을 변경해야 할 수도 있습니다.

공급자별 도구 대신 [Terraform](https://www.terraform.io/)은 인프라 프로비저닝에 대한 더 일반적인 솔루션을 제공합니다.
Terraform을 사용하면 변경을 한 번만 지정하고 여러 공급자에 변경을 적용할 수 있습니다.

### Terraform

Terraform은 [HashiCorp Configuration Language (HCL)](https://developer.hashicorp.com/terraform/language)을 사용하여 주요한 세 가지 공급자인 AWS, Azure 및 GCP를 포함한 2000개 이상의 다양한 클라우드 리소스 [공급자](https://registry.terraform.io/browse/providers)를 지원합니다.

구성 언어를 사용하면 제공자에 관계없이 개발, 테스트 및 프로덕션 환경에서 동일한 코드를 사용하여 구성 세부 정보를 추상화하고 인프라에 대한 모든 변경을 소스 코드 버전 관리를 통해 관리할 수 있습니다.
Terraform은 또한 공통 언어를 사용하여 독립적인 리소스를 인프라에 통합할 수 있도록 지원합니다.
예를 들어, 단일 구성 파일을 사용하여 RPC 노드와 프론트엔드 로드 밸런서를 함께 배포할 수 있습니다.

배포를 위해 호스트를 준비한 후에는 Terraform을 사용하여 호스트를 이미지에서 필요한 소프트웨어로 사전 구성하거나 기본 이미지를 사전 구성하는 스크립트를 실행할 수 있습니다.

여러 공급자와 함께 Terraform을 사용하는 예제는 [polkadot-validator-setup](https://github.com/w3f/polkadot-validator-setup)에서 찾을 수 있습니다.

### Ansible

기본 호스트가 배포된 후에는 필요한 소프트웨어 구성 요소, 구성 파일 및 시스템 설정을 구성해야 합니다.
Terraform 또는 클라우드 공급자 도구 외에도 Ansible은 인프라 배포를 자동화하는 또 다른 유연한 방법을 제공합니다.

Ansible은 **playbook**을 사용하여 시스템 구성 요소를 오케스트레이션, 구성, 관리 및 배포합니다.
playbook과 **role**의 조합을 사용하여 그룹화된 노드에 대한 특정 구성 또는 동작을 구현할 수 있습니다.

블록체인 노드를 배포할 때 Ansible을 사용하면 호스트를 설명하고 호스트를 역할에 따라 그룹화하는 인벤토리를 정의할 수 있습니다. 예를 들어, 호스트를 validator, collator 또는 rpc 노드로 식별하는 그룹으로 그룹화할 수 있습니다.
그런 다음 인벤토리에서 호스트와 그룹을 링크하고 각 호스트에서 실행할 역할을 실행하는 플레이북을 호출할 수 있습니다.

Ansible을 사용하는 예제는 [ansible-galaxy](https://github.com/paritytech/ansible-galaxy) 및 [node role](https://github.com/paritytech/ansible-galaxy/tree/main/roles/node)에서 찾을 수 있습니다.

## Kubernetes

Kubernetes 클러스터에서는 이전에 Kubernetes 구성을 관리한 경험이 있는 경우에만 배포해야 합니다.
Kubernetes에서 Substrate 기반 노드를 관리하기 위한 주요 도구는 노드를 배포하는 데 사용할 수 있는 **helm 차트**와 Kubernetes 클러스터에서 테스트 네트워크를 배포 및 유지 관리하는 데 사용할 수 있는 **테스트넷 매니저**입니다.
이러한 도구를 사용하기 전에 Kubernetes 클러스터에 액세스 권한, `kubectl`의 로컬 복사본 및 Helm이 설치되어 있어야 합니다.

### Helm 차트

[Parity Helm 차트](https://github.com/paritytech/helm-charts)는 Substrate 및 InfraBlockspace 구성 요소를 정의, 설치, 관리 및 업그레이드하는 helm 차트의 모음입니다.
이 차트 컬렉션에서 [node](https://github.com/paritytech/helm-charts/tree/main/charts/node) 차트는 substrate 또는 infrablockspace 노드 바이너리를 배포하는 데 사용됩니다.
차트의 모든 매개변수는 [node 차트 README.md](https://github.com/paritytech/helm-charts/tree/main/charts/node)에 문서화되어 있습니다.

알아야 할 가장 중요한 매개변수는 다음과 같습니다:

| 옵션                  | 설명                                                        |
| :---------------------- | :------------------------------------------------------------ |
| node.chain              | 연결할 네트워크입니다.                                        |
| node.command            | 사용할 바이너리입니다.                                                |
| node.flags              | 컨테이너 내에서 바이너리와 함께 사용할 커맨드 옵션입니다. |
| node.customChainspecUrl | 사용자 정의 체인 사양 URL입니다.                               |

시작할 수 있는 `values.yml` 구성 파일 예제도 사용할 수 있습니다.

다음 예제는 두 개의 validator와 두 개의 전체 노드를 가진 Kubernetes에서 `rococo-local` 테스트 네트워크 체인을 배포하는 방법을 보여줍니다.

helm 차트를 사용하여 `rococo-local` 체인을 배포하려면 다음 단계를 수행합니다:

1. Kubernetes 클러스터에 액세스할 수 있고 Helm 클라이언트가 설치되어 있는지 확인합니다.
2. 다음 명령을 실행하여 Parity 차트 저장소를 Helm에 추가합니다:

   ```bash
   helm repo add parity https://paritytech.github.io/helm-charts/
   ```

3. 다음 명령을 실행하여 node 차트를 설치합니다:

   ```bash
   helm install infrablockspace-node parity/node
   ```

4. 다음 명령을 실행하여 Alice 계정과 사용자 정의 노드 키를 사용하여 validator 노드를 배포합니다:

   ```bash
   helm install rococo-alice parity/node --set node.role="validator" \
      --set node.customNodeKey="91cb59d86820419075b08e3043cd802ba3506388d8b161d2d4acd203af5194c1" \
      --set node.chain=rococo-local \
      --set node.perNodeServices.relayP2pService.enabled=true \
      --set node.perNodeServices.relayP2pService.port=30333 \
      --set node.flags="--alice --rpc-external --ws-external --rpc-cors all --rpc-methods=unsafe"
   ```

   이 명령은 `alice` 노드를 상태 서비스로 배포하며 예제 사용자 정의 노드 키와 다른 모든 호스트에 대한 부팅 노드로 사용할 서비스를 가지고 있습니다.

5. 다음 명령을 실행하여 Bob 계정과 `alice`를 부팅 노드로 사용하여 validator 노드를 배포합니다:

   ```bash
   helm install rococo-bob parity/node --set node.role="validator" \
      --set node.chain=rococo-local \
      --set node.flags="--bob --bootnodes '/dns4/rococo-alice-node-0-relay-chain-p2p/tcp/30333/p2p/12D3KooWMeR4iQLRBNq87ViDf9W7f6cc9ydAPJgmq48rAH116WoC'"
   ```

   두 개의 validator가 실행되면 체인이 블록을 생성하기 시작해야 합니다.

6. 다음 명령을 실행하여 두 개의 전체 노드를 배포합니다:

   ```bash
   helm install rococo-pool parity/node --set node.chain=rococo-local \
      --set node.replicas=2 \
      --set node.flags="--bootnodes '/dns4/rococo-alice-node-0-relay-chain-p2p/tcp/30333/p2p/12D3KooWMeR4iQLRBNq87ViDf9W7f6cc9ydAPJgmq48rAH116WoC'"
   ```

   이 단계를 완료하면 두 개의 validator와 두 개의 전체 노드가 있는 작동하는 `rococo-local` 테스트 체인이 있게 됩니다.

다음 도구는 helm 릴리스를 관리하는 데 유용한 도구입니다.

- [Helmfile](https://github.com/roboll/helmfile)
- [Terraform Helm 공급자](https://registry.terraform.io/providers/hashicorp/helm/latest/docs)
- [Flux CD](https://fluxcd.io/)
- [ArgoCD](https://argo-cd.readthedocs.io/en/stable/)

## Docker

네트워크의 가상 머신으로 노드를 배포하는 경우 Docker 이미지를 사용하여 다른 유형의 노드에 대한 노드 구성을 준비할 수 있습니다.
예를 들어, validator 노드와 RPC 제공자 노드에 대한 Docker 이미지를 준비하고 각 유형의 여러 노드를 별도의 가상 머신을 구성하지 않고 배포할 수 있습니다.
외부 노드 운영자는 제공하는 Docker 이미지를 사용하여 필요할 때마다 새로운 노드를 배포할 수 있습니다.

### 샘플 Dockerfile

다음 샘플 Dockerfile은 공격 표면을 최소화하고 보안을 유지하는 안전한 방식으로 Docker 이미지를 빌드하는 최선의 방법을 보여줍니다.
이 예제는 공식 InfraBlockspace 이미지를 만드는 데 사용되는 Dockerfile과 유사한 버전입니다.
추가 정보는 Docker 문서의 [Dockerfile 작성을 위한 최상의 방법](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)을 참조할 수도 있습니다.

```dockerfile
# 이것은 노드 템플릿의 예제 빌드 단계입니다. 여기서는 임시 이미지에서 바이너리를 작성합니다.

# 이것은 substrate 노드를 빌드하기 위한 기본 이미지입니다.
FROM docker.io/paritytech/ci-linux:production as builder

WORKDIR /node-template
COPY . .
RUN cargo build --locked --release

# 이것은 2단계입니다: 매우 작은 이미지로 바이너리를 복사합니다."
FROM docker.io/library/ubuntu:20.04
LABEL description="Substrate Node Template을 위한 멀티스테이지 Docker 이미지" \
  image.type="builder" \
  image.authors="you@email.com" \
  image.vendor="Substrate Developer Hub" \
  image.description="Substrate Node Template을 위한 멀티스테이지 Docker 이미지" \
  image.source="https://github.com/substrate-developer-hub/substrate-node-template" \
  image.documentation="https://github.com/substrate-developer-hub/substrate-node-template"

# 노드 바이너리를 복사합니다.
COPY --from=builder /node-template/target/release/node-template /usr/local/bin

RUN useradd -m -u 1000 -U -s /bin/sh -d /node-dev node-dev && \
  mkdir -p /chain-data /node-dev/.local/share && \
  chown -R node-dev:node-dev /chain-data && \
  ln -s /chain-data /node-dev/.local/share/node-template && \
  # unclutter and minimize the attack surface
  rm -rf /usr/bin /usr/sbin && \
  # check if executable works in this container
  /usr/local/bin/node-template --version

USER node-dev

EXPOSE 30333 9933 9944 9615
VOLUME ["/chain-data"]

ENTRYPOINT ["/usr/local/bin/node-template"]
```

### 자동화된 빌드 파이프라인

다음 샘플 [GitHub 액션](https://github.com/substrate-developer-hub/substrate-node-template/blob/main/.github/workflows/release.yml)은 Docker 이미지를 빌드하고 DockerHub에 게시합니다.
대부분의 경우, 이 작업은 수동 워크플로우나 새로운 릴리스가 게시될 때 트리거됩니다.

Docker 이미지를 안전하게 게시하려면 [암호화된 비밀](https://docs.github.com/en/actions/security-guides/encrypted-secrets)로 GitHub 저장소 또는 조직에 비밀을 추가해야 합니다.
또한 GitHub 비밀에 DockerHub 계정의 자격 증명을 저장해야 합니다.
대신 GitHub 이미지 레지스트리(예: GitHub 이미지 레지스트리)를 사용하려는 경우 _Build and push Docker images_ 단계를 수정할 수 있습니다.

```yaml
# 이 작업이 실행될 때
name: Build & Publish Docker Image

on:
  # push 이벤트에서 워크플로우를 트리거하지만 main 브랜치에서만 트리거합니다.
  # push:
  # branches: [ main ]

  # Actions 탭에서 이 워크플로우를 수동으로 실행할 수 있습니다.
  workflow_dispatch:

# 작업은 순차적으로 또는 병렬로 실행될 수 있는 하나 이상의 작업으로 구성됩니다.
jobs:
  build:
    # 작업이 실행될 수 있는 실행기의 유형
    runs-on: ubuntu-22.04

    # 단계는 작업의 일부로 실행될 일련의 작업을 나타냅니다.
    steps:
      # 저장소를 $GITHUB_WORKSPACE에 체크아웃하여 작업이 액세스할 수 있도록합니다.
      - name: Check out the repo
        uses: actions/checkout@v2.5.0

      # Docker Hub의 자격 증명을 사용하여 Docker Hub에 로그인합니다.
      - name: Log in to Docker Hub
        uses: docker/login-action@v2.1.0
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}

      # rev를 사용하기 위해 커밋 짧은 해시를 가져옵니다.
      - name: Calculate rev hash
        id: rev
        run: echo "value=$(git rev-parse --short HEAD)" >> $GITHUB_OUTPUT

      # 2개의 이미지를 빌드하고 푸시합니다. 버전 태그와 최신 태그로 하나씩 빌드합니다.
      - name: Build and push Docker images
        uses: docker/build-push-action@v3.2.0
        with:
          context: .
          push: true
          tags: ${{ env.DOCKER_REPO }}:v${{ steps.rev.outputs.value }}, ${{ secrets.DOCKER_REPO }}:latest
```