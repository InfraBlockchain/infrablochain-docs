---
title: 아카이브
description: 아카이브 프로그램에 대한 명령 줄 참조 정보입니다.
keywords:
---

`아카이브` 프로그램은 Substrate 기반 체인의 모든 블록, 상태 및 트랜잭션 데이터를 색인화하고 관계형 SQL 데이터베이스에 색인화된 데이터를 저장하는 데 사용됩니다.
`아카이브` 프로그램에 의해 생성된 데이터베이스는 실행 중인 Substrate 블록체인의 모든 데이터를 반영합니다.
데이터를 아카이브한 후에는 데이터베이스 도구를 사용하여 블록체인 상태에 대한 SQL 데이터베이스에서 정보를 조회하고 검색할 수 있습니다.
Substrate 아카이브 데이터베이스에 대해 실행할 수 있는 쿼리 예제는 [유용한 쿼리](https://github.com/paritytech/substrate-archive/wiki/Useful-Queries)를 참조하십시오.

## 시작하기 전에

Substrate 기반 체인을 위한 데이터베이스를 생성하기 전에 필요한 파일로 환경을 준비해야 합니다:

- Substrate 노드를 실행하는 컴퓨터에 PostgreSQL이 설치되어 있어야 합니다.

  PostgreSQL [Downloads](https://www.postgresql.org/download/) 페이지에서 다양한 플랫폼용 PostgreSQL 패키지를 다운로드할 수 있습니다.

  플랫폼에 따라 로컬 패키지 관리자를 사용하여 PostgreSQL을 설치할 수도 있습니다.
  예를 들어, macOS 컴퓨터에서는 터미널에서 `brew install postgresql` 명령을 실행하여 PostgreSQL 패키지를 설치할 수 있습니다.

- PostgreSQL이 설치된 컴퓨터에 RabbitMQ 또는 Docker Compose가 설치되어 있어야 합니다.

  RabbitMQ 또는 Docker를 설치하는 방법은 플랫폼에 따라 지침과 시스템 요구 사항이 다를 수 있습니다.
  [RabbitMQ](https://www.rabbitmq.com/) 또는 [Docker](https://www.docker.com/) 사용에 대한 정보는 [Setup](https://github.com/paritytech/substrate-archive/wiki/1-Setup) `substrate-archive` 위키 페이지를 참조하십시오.

- Substrate 체인은 백엔드 데이터베이스로 RocksDB를 사용해야 합니다.

## 설치 및 구성

`substrate-archive-cli` 프로그램을 설치하려면 다음 단계를 따르십시오:

1. 컴퓨터에서 터미널 쉘을 엽니다.

2. 다음 명령을 실행하여 `substrate-archive` 저장소를 복제합니다:

   ```
   git clone https://github.com/paritytech/substrate-archive.git
   ```

3. 다음 명령을 실행하여 `substrate-archive` 저장소의 루트 디렉토리로 이동합니다:

   ```
   cd substrate-archive
   ```

4. Substrate 노드에서 PostgreSQL 데이터베이스 (`postgres`)와 PostgreSQL 관리 프로세스 (`pgadmin`)를 시작합니다.

   Docker Compose를 사용하는 경우 `docker-compose up -d` 명령을 실행하여 서비스를 자동으로 시작할 수 있습니다.

5. `pruning`을 아카이브로 설정하여 Substrate 노드를 시작합니다.

   예:

   ```
   ./target/release/node-template --pruning=archive
   ```

6. 현재 DB를 확인합니다:
   `psql -U postgres -hlocalhost -p6432`

7. `substrate-archive/src`에서 `DATABASE_URL=postgres://postgres:123@localhost:6432/local_chain_db sqlx` 명령을 실행하여 데이터베이스를 생성합니다.

8. `CHAIN_DATA_DB="<your_path>"`를 설정합니다.

9. `archive.conf` 파일을 설정합니다:

   - 기본 경로를 기본 DB로 설정합니다.
   - rocksdb가 있는 위치를 지정합니다. CHAIN_DATA_DB를 사용하여 상태를 지정합니다.
   - 보조 DB는 최적화입니다.
   - PostgreSQL URL은 (프로덕션 환경에서인 경우) 변수로 설정합니다.

10. (선택 사항) 로깅 및 디버깅을 설정합니다.

11. 노드 템플릿을 실행합니다. `--release --dev base-path=/tmp/dir --pruning=archive`로 실행하는지 확인합니다.

12. 노드 템플릿으로 트랜잭션을 만듭니다.

13. 대상 체인에 대한 `substrate-archive` 노드를 시작합니다:
    `cargo run --release -- -c archive-conf.toml --chain=polkadot`

14. 웹 브라우저를 열고 Postgres 관리 콘솔에 로그인합니다.

   - 기본 URL:  localhost:16543
   - 기본 사용자 이름: pgadmin4@pgadmin.org
   - 기본 비밀번호: admin

15. 쿼리를 시작할 수 있는 참조를 확인합니다.