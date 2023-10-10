---
title: 체인 스펙을 사용자 정의하기
description:
keywords:
---

Substrate 노드를 만든 후에는 많은 피어와 함께 네트워크를 시작하고 싶을 것입니다!
이 가이드는 다른 노드가 당신의 네트워크를 _명시적으로_ 발견하고 피어링할 수 있도록 체인 스펙 파일을 균일하게 생성하고 배포하는 한 가지 방법을 보여줍니다.

이 가이드에서는 다음을 설명합니다:

- `chain-spec.json`을 생성하고 다른 노드가 공통 네트워크에 참여할 수 있도록 포함하는 방법
- 노드 소스 코드를 편집하지 않고 기존의 일반 체인 스펙을 수정하는 방법

## 체인 스펙 생성하기

**일반 체인 스펙 파일**

1. 노드의 작업 디렉토리에서 다음 명령을 사용하여 일반 체인 스펙을 생성합니다:

   ```bash
   ./target/release/node-template build-spec > chain-spec-plain.json
   ```

   이제 당신의 `chain_spec.rs` 파일에서 설정된 _기본_ 네트워크를 위한 **일반 체인 스펙** 파일을 생성했습니다.
   이 파일은 다른 노드에 전달할 수 있습니다.

1. 일반 체인 스펙 수정하기 (선택 사항):

   이 선택 사항은 그렇지 않으면 노드의 소스를 수정하여 _새 네트워크_에서 실행해야 하는 네트워크의 _기존_ 일반 체인 스펙을 활용할 수 있습니다.
   예를 들어, 이는 [로컬 릴레이 체인 준비](/tutorials/build-a-parachain/prepare-a-local-relay-chain/)에서 Polkadot의 소스를 사용자 정의하지 않고 사용자 정의 _릴레이 체인_을 생성하려는 경우에 매우 유용합니다.

   여기서는 _동일한_ 체인 스펙을 사용하지만 부트노드를 비활성화하기 위해 플래그를 전달합니다. 이렇게 하면 이러한 노드가 다른 새 네트워크를 위해 다른 노드가 될 수 있는 _새 네트워크_가 필요합니다.

   ```bash
   ./target/release/node-template build-spec --chain chain-spec-plain.json --raw --disable-default-bootnode > no-bootnodes-chain-spec-plain.json
   ```

   이 `no-bootnodes-chain-spec-plain.json`은 SCALE 저장소 인코딩된 배포 가능한 원시(raw) 체인 스펙을 생성하는 데 사용될 수 있습니다.

**원시(raw) 체인 스펙 파일**

1. 원시(raw) 체인 스펙 생성하기.

   일반 사양이 준비되면 다음을 실행하여 최종 원시(raw) 체인 스펙을 생성할 수 있습니다:

   ```bash
   ./target/release/node-template build-spec --chain chain-spec-plain.json --raw > chain-spec.json
   ```

   원시(raw) 체인 스펙은 노드에 전달될 때 항상 사용되어야 합니다.

## 원시(raw) 체인 스펙 게시하기

Rust로 빌드된 WebAssembly 대상은 최적화되어 있기 때문에 바이너리는 결정론적으로 재현할 수 없습니다.
각 네트워크 참가자가 일반 및/또는 원시(raw) 체인 스펙을 생성한다면, 결과적인 Wasm 블롭의 차이로 인해 합의가 깨질 수 있습니다.

일반적으로는 노드의 소스 코드 자체에 체인 스펙 파일을 포함하여 노드를 동일한 방식으로 빌드할 수 있도록하는 것이 좋습니다. 이렇게 하면 다른 genesis 블롭과 비교하여 결정론적이지 않음을 확인하기 쉬워집니다.
Polkadot, Kusama, Rococo 등의 네트워크 체인 스펙 파일은 [여기에서](https://github.com/paritytech/polkadot-sdk/tree/master/polkadot/node/service/chain-specs) 찾을 수 있으며, `.gitignore` 파일도 함께 제공되어 이러한 `!/*.json` 파일을 실수로 변경하지 않도록 합니다. 이렇게 하면 노드 소프트웨어를 더 발전시키고 [런타임 업그레이드](/tutorials/build-a-blockchain/upgrade-a-running-network/)를 수행하는 동안 안정성을 유지할 수 있습니다.

## 새 노드 시작하기

노드 바이너리를 게시하거나 사용자가 직접 빌드한 다음 당신의 네트워크에 참여하려는 경우, 필요한 것은 _동일한_ 원시(raw) 체인 스펙 파일과 다음 명령을 사용하여 바이너리를 실행하는 것입니다:

```bash
# 바이너리 이름 `node-template`
# `chain-spec.json`은 공식 공통 소스에서 얻음
node-template --chain chain-spec.json
```

이는 _기본_ 네트워크로 간단히 구성할 수도 있습니다.
참고로, [Polkadot](https://github.com/paritytech/polkadot/commits/master/cli/src/command.rs)은 다양한 네트워크에 대한 체인 스펙을 사용하는 기본 명령을 구현하는 방법을 볼 수 있습니다. [여기에서](https://github.com/paritytech/polkadot-sdk/tree/master/polkadot/node/service/chain-specs) 이러한 체인 스펙을 찾을 수 있습니다.

## 예시

- [신뢰할 수 있는 노드 추가](/tutorials/build-a-blockchain/add-trusted-nodes#add-keys-to-keystore)
- [Polkadot과 유사한 네트워크 체인 스펙](https://github.com/paritytech/polkadot-sdk/tree/master/polkadot/node/service/chain-specs)
- [다양한 네트워크에 대한 Polkadot 명령](https://github.com/paritytech/polkadot/commits/master/cli/src/command.rs)