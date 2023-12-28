---
title: Containerization of Chains
description: This article explains how to containerize chains.
keywords:
  - chain
  - containerization
---

## Containerization
Containerization is a software distribution process that bundles an application's code with all the necessary files and libraries required to run it on any infrastructure. **_InfraBlockchain_** also supports containerization for all chains within it, and the relevant methods are described below.

## Creating a Container Image

Let's explore how to create a Docker container image.

To create a container image, you need a Dockerfile. Here, we'll explain how to create a container image for the relay chain. If you want to create a container image for a different chain, please visit the link below and check the desired chain repository.

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
    mv /infra-relay-chain/target/release/infrablockspace /infra-relay-chain/target/; 

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

Once you have completed writing the relevant files, use the following command to generate the container image:

```bash
docker build -t <tag name>:<version> .
```

The ways to utilize the generated image are mentioned in the deployment section.