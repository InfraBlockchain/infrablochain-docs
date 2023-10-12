---
title: 키 서명과 네트워크 작업
description:
keywords:
---

[계정, 주소 및 키](/learn/accounts-addresses-keys/)에서는 공개 및 개인 키에 대한 논의가 서로 다른 네트워크를 이용하는 사용자와 관련된 계정 및 주소에 초점을 맞추었습니다.
그러나 키와 다른 키 서명은 Substrate 노드를 배포하고 특정 노드 작업을 수행하는 데에도 기본적입니다.
이 섹션에서는 암호화 방식과 각각의 노드 구성 요소에서 어디에 사용되는지에 대해 다시 설명합니다.

## 디지털 서명 방식

대부분의 디지털 서명 방식은 다음과 같은 기능을 제공합니다:

- **키 생성**. 서명 방식은 모든 가능한 개인 키 집합에서 임의의 개인 키를 생성하는 방법과 해당 개인 키의 진정성을 확인할 수 있는 공개 키를 제공해야 합니다.
- **메시지 서명**. 서명 방식은 주어진 메시지와 개인 키에 대한 서명을 생성하는 방법을 제공해야 합니다.
- **서명 검증**. 서명 방식은 메시지, 공개 키 및 서명을 기반으로 메시지의 진정성을 수용하거나 거부하는 방법을 제공해야 합니다.

서명 방식은 이러한 작업을 수행하기 위해 다른 알고리즘을 사용합니다.
사용되는 수학에 관계없이, 모든 서명 방식은 두 가지 주요 결과를 달성하기 위해 설계되었습니다:

- 주어진 메시지와 개인 키에 대해 생성된 서명의 검증은 해당하는 공개 키를 사용하여 확인됩니다.
- 개인 키 없이 유효한 서명을 생성하는 것은 계산적으로 불가능하므로 메시지의 무결성을 보장할 수 있습니다. 

다음 서명 방식은 Substrate 기반 체인에서 지원됩니다:

| 방식    | 설명                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| :------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ed25519 | Ed25519 서명 방식은 Schnorr 서명의 변형된 형태이자 SHA-512 및 Curve25519을 사용하는 Edwards-curve Digital Signature Algorithm (EdDSA) 입니다. 이 서명 방식은 ECDSA 서명 방식보다 보안성이 높고 메시지에 대한 서명 속도가 훨씬 빠릅니다.                                                                                                                                                                                                                                           |
| sr25519 | Sr25519 서명 방식은 Substrate의 기본 서명 방식입니다. 이 서명 방식은 Ristretto point compression을 사용하는 Schnorr 서명을 사용하는 Schnorrkel 변형에 기반합니다. Sr25519 서명 방식은 블록체인 환경에서 특히 유용한 계층적 결정론적 키 파생, 다중 서명 서명 및 검증 가능한 난수 함수와 같은 추가 기능을 지원합니다.                                                                                                                                                                 |
| ecdsa   | 타원 곡선 디지털 서명 알고리즘 (ECDSA)은 Secp256k1 타원 곡선 암호를 사용하는 Digital Signature Algorithm (DSA)의 변형입니다. 이 서명 방식은 Schnorr 서명을 보호하는 특허 때문에 초기에 Bitcoin 및 Ethereum에서 사용되었습니다. ECDSA 서명 방식은 임계 서명과 같은 일부 고급 암호 기술을 복잡하게 만듭니다.                                                                                                                                                 |

## 세션 키 및 타입

세션 키는 검증자가 합의 관련 메시지에 서명하기 위해 사용하는 개인 온라인 키입니다.
세션 키는 검증자가 특정 네트워크 작업을 수행할 수 있도록 온라인으로 사용 가능해야 합니다.

이러한 키는 자금을 제어하는 데 사용되지 않으며 의도된 목적에만 사용되어야 하며
정기적으로 변경될 수 있습니다.
세션 키를 생성하려면 검증자 노드 운영자는 컨트롤러 계정을 사용하여 세션의 공개 키로 서명된 인증서를 생성해야 합니다.
인증서는 해당 키가 검증자의 스테이킹 계정 및 노미네이터(nominators) 를 대신하여 작동한다는 것을 증명합니다.
세션 키를 생성한 후, 검증자 노드 운영자는 이 키가 컨트롤러 키를 대표한다는 것을 체인에 알리기 위해 세션 인증서를 체인의 트랜잭션으로 게시합니다.
대부분의 경우, 노드 운영자는 [Session](https://paritytech.github.io/substrate/master/pallet_session/index.html) 팔레트를 사용하여 세션 키를 관리합니다.

[`SessionKeys`](https://paritytech.github.io/substrate/master/sp_session/index.html)
트레이트는 일반적인 색인 가능한 타입이며 런타임에서 원하는 만큼 여러 개의 세션 키를 선언할 수 있습니다.
기본 Substrate 노드 템플릿은 네 개의 세션 키를 사용합니다.
다른 체인은 해당 체인의 검증자가 수행해야 하는 작업에 따라 더 많거나 더 적을 수 있습니다.

실제로 검증자는 모든 세션 공개 키를 단일 객체로 결합하고 컨트롤러 계정으로 공개 키 집합에 서명한 후 키를 등록하기 위해 트랜잭션을 제출합니다.
이 체인상 등록은 검증자 _노드_ 를 자금을 보유한 _계정_ 과 연결합니다.
따라서 세션 키 객체와 관련된 계정은 노드의 동작에 따라 보상을 받거나 벌점을 받을 수 있습니다.

런타임은 `impl_opaque_keys!` 매크로를 사용하여 어떤 세션 키가 구현되는지 선언합니다:

```rust
impl_opaque_keys! {
    pub struct SessionKeys {
        pub grandpa: Grandpa,
        pub babe: Babe,
        pub im_online: ImOnline,
        pub authority_discovery: AuthorityDiscovery,
    }
}
```

Infrablockspace 는 다음과 같은 세션 키를 사용합니다:

| 이름                 | 유형    |
| :------------------- | :------ |
| Authority discovery  | sr25519 |
| GRANDPA              | ed25519 |
| BABE                 | sr25519 |
| I'm online           | sr25519 |
| Parachain assignment | sr25519 |
| Parachain validator  | ed25519 |

BABE는 Verifiable Random Function에서 사용할 수 있는 키와 디지털 서명에 적합한 키가 필요합니다.
Sr25519 키는 두 가지 기능을 모두 갖고 있으므로 BABE에 사용됩니다.

## 커맨드 인터페이스

`infrablockspace keys` 또는 `subkey` 명령을 사용하여 키를 생성하고 검사할 수 있습니다.

두 가지 중요한 하위 명령은 다음과 같습니다:

- `generate`는 새로운 무작위 계정을 생성하고 개인 키를 표준 출력에 인쇄하거나 키를 파일에 저장합니다.
- `inspect`는 비밀 문구 또는 시드를 전달하여 계정에 대한 계정 및 주소 정보를 확인합니다.

일부 중요한 옵션은 다음과 같습니다:

- `--network`는 키가 사용될 네트워크를 지정합니다 (기본값은 `substrate`입니다).
- `--scheme`은 키의 서명 방식을 지정합니다 (기본값은 `sr25519`입니다).

예를 들어, 다음 명령을 실행하여 infrablockspace 임의 키를 **생성**할 수 있습니다:

```bash
infrablockspace key generate -n infrablockspace
```

다음과 유사한 출력이 표시됩니다:

```text
Secret phrase:       settle whisper usual blast device source region pumpkin ugly beyond promote cluster
  Network ID:        infrablockspace
  Secret seed:       0x2e6371e04b45f16cd5c2d66fc47c8ad7f2881215287c374abfa0e07fd003cb01
  Public key (hex):  0x9e65e97bd8ba80095440a68d1be71adff107c73627c8b85d29669721e02e2b24
  Account ID:        0x9e65e97bd8ba80095440a68d1be71adff107c73627c8b85d29669721e02e2b24
  Public key (SS58): 14agqii5GAiM5z4yzGhJdyWQ3a6HeY2oXvLdCrdhFXRnQ77D
  SS58 Address:      14agqii5GAiM5z4yzGhJdyWQ3a6HeY2oXvLdCrdhFXRnQ77D
```

비밀 문구로 키를 생성한 경우, 다음과 유사한 명령을 실행하여 키에 대한 계정 및 주소 정보를 **검사**할 수 있습니다:

```bash
./infrablockspace key inspect -n infrablockspace "settle whisper usual blast device source region pumpkin ugly beyond promote cluster"
```

다음과 유사한 출력이 표시됩니다:

```text
Secret phrase:       settle whisper usual blast device source region pumpkin ugly beyond promote cluster
  Network ID:        infrablockspace
  Secret seed:       0x2e6371e04b45f16cd5c2d66fc47c8ad7f2881215287c374abfa0e07fd003cb01
  Public key (hex):  0x9e65e97bd8ba80095440a68d1be71adff107c73627c8b85d29669721e02e2b24
  Account ID:        0x9e65e97bd8ba80095440a68d1be71adff107c73627c8b85d29669721e02e2b24
  Public key (SS58): 14agqii5GAiM5z4yzGhJdyWQ3a6HeY2oXvLdCrdhFXRnQ77D
  SS58 Address:      14agqii5GAiM5z4yzGhJdyWQ3a6HeY2oXvLdCrdhFXRnQ77D
```

`//Stash//0`로 생성된 키를 검사하려면 다음과 유사한 명령을 실행해야 합니다:

```bash
infrablockspace key inspect -n infrablockspace "settle whisper usual blast device source region pumpkin ugly beyond promote cluster//Stash//0"
```

다음과 유사한 출력이 표시됩니다:

```text
Secret Key URI `settle whisper usual blast device source region pumpkin ugly beyond promote cluster//Stash//0` is account:
  Network ID:        infrablockspace
 Secret seed:       0xe9437b365161e8228e8abd53d64e6b31058dcddcd0b96f895045ecc41579ee3e
  Public key (hex):  0xd8ed7b942f6e590b06e99951ac10e3312f65f01df5b3f250b70374fc2da1046d
  Account ID:        0xd8ed7b942f6e590b06e99951ac10e3312f65f01df5b3f250b70374fc2da1046d
  Public key (SS58): 15uRtdeE4MyMHV9LP1UHKqTx4f8Qa8uVZUpxWWw8VKSroucK
  SS58 Address:      15uRtdeE4MyMHV9LP1UHKqTx4f8Qa8uVZUpxWWw8VKSroucK
```

subkey 명령과 커맨드 옵션에 대한 자세한 내용은 [subkey](/reference/command-line-tools/subkey/)를 참조하십시오.