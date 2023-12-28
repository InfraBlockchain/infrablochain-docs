---
title: 체인 컨테이너화
description: 체인 컨테이너화하는 방법에 대해 설명합니다.
keywords:
  - 체인
  - 컨테이너화
---

## 컨테이너화
컨테이너화는 애플리케이션의 코드를 모든 인프라에서 실행하는 데 필요한 모든 파일 및 라이브러리와 함께 번들로 제공하는 소프트웨어 배포 프로세스입니다. **_인프라 블록체인(InfraBlockchain)_** 에 속해있는 모든 체인들 또한, 컨테이너화가 가능하며 관련 방법은 아래에 나와있습니다.

## 컨테이너 이미지 만들기

도커 컨테이너 이미지를 만드는 방법에 대해 알아보겠습니다.

컨테이너 이미지를 만들기 위해서는 Dockerfile이 필요합니다.
여기서는 릴레이체인의 컨테이너 이미지를 만드는 것에 대해 설명합니다.
다른 체인의 컨테이너 이미지를 만드시길 원하실 경우 아래 링크에 접속한 다음 원하시는 체인 레포지토리에서 확인하시길 바랍니다.

https://github.com/InfraBlockchain/infrablockchain-substrate

```docker
FROM ubuntu:jammy AS builder

# The node will be built in this directory
WORKDIR /infra-relay-chain

RUN apt -y update && \
  apt install -y --no-install-recommends \
  software-properties-common llvm curl git file binutils binutils-dev \
  make cmake ca-certificates clang g++ zip dpkg-dev openssl gettext \
  build-essential pkg-config libssl-dev libudev-dev time clang protobuf-compiler

# install rustup
RUN curl https://sh.rustup.rs -sSf | sh -s -- -y

# rustup directory
ENV PATH /root/.cargo/bin:$PATH

# setup rust nightly channel, pinning specific version as newer versions have a regression
RUN rustup install nightly-2023-05-21

RUN rustup default nightly-2023-05-21

# install wasm toolchain for substrate
RUN rustup target add wasm32-unknown-unknown --toolchain nightly-2023-02-20

#compiler ENV
ENV CC clang
ENV CXX g++

# Copy code to build directory, instead of only using .dockerignore, we copy elements
# explicitly. This lets us cache build results while iterating on scripts.
COPY . .

RUN echo 'Building in release mode.' ; \
    cargo build --release ; \
    mv /infra-relay-chain/target/release/infra-relaychain /infra-relay-chain/target/; 

# Final stage. Copy the node executable and the script
FROM ubuntu:jammy

WORKDIR /infra-relay-chain

COPY --from=builder /infra-relay-chain/target/infrablockspace .

# curl is required for uploading to keystore
# note: `subkey insert` is a potential alternarve to curl
RUN apt -y update \
  && apt install -y --no-install-recommends curl \
  && rm -rf /var/lib/apt/lists/*

# expose node ports
EXPOSE 30333 9933 9944

ENV RUST_BACKTRACE 1

ENTRYPOINT ["./infrablockspace"]
CMD []
```

관련 파일 작성이 완료가 되었다면, 아래 명령어를 통해 container image를 생성해줍니다.

```bash
docker build -t <tag name>:<version> .
```

생성된 이미지를 활용할 방법은 deploy 부분에 나와있습니다.

