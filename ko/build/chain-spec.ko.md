---
title: 체인 스펙
description: Substrate 기반 네트워크에서 체인 스펙의 역할, 노드를 시작할 때 사용할 체인 스펙을 지정하는 방법, 그리고 체인 스펙을 커스텀하고 배포하는 방법에 대해 설명합니다.
keywords:
  - genesis
  - 구성
  - 체인 스펙
---

Substrate에서 **체인 스펙**은 Substrate 기반 블록체인 네트워크를 설명하는 정보의 모음입니다.
예를 들어, 체인 스펙은 블록체인 노드가 연결하는 네트워크, 초기 통신을 수행하는 다른 노드, 그리고 노드가 블록을 생성하기 위해 동의해야 하는 genesis(genesis state)를 식별합니다.

체인 스펙은 [`ChainSpec` 구조체](https://paritytech.github.io/substrate/master/sc_service/struct.GenericChainSpec.html)을 사용하여 정의됩니다.
`ChainSpec` 구조체는 체인에 필요한 정보를 두 부분으로 나누어 제공합니다:

- **외부 노드**가 네트워크 참여자와 통신하고 텔레메트리 엔드포인트로 데이터를 전송하는 데 사용되는 정보를 포함하는 클라이언트 사양입니다.
  이러한 체인 스펙 설정 중 많은 부분은 노드를 시작할 때 command-line 옵션으로 재정의하거나 블록체인이 시작된 후에 변경할 수 있습니다.

- 네트워크의 모든 노드가 동의해야 하는 **genesis(genesis state)**입니다.
  genesis는 블록체인이 처음 시작될 때 설정되어야 하며, 블록체인을 완전히 새로 시작하지 않고는 변경할 수 없습니다.

## 외부 노드 설정 커스텀하기

외부 노드에서 체인 스펙은 다음과 같은 정보를 제어합니다:

- 노드가 통신하는 부트 노드.

- 노드가 텔레메트리(telemetry) 데이터를 전송하는 서버 엔드포인트.

- 노드가 연결하는 네트워크의 사람과 기계가 읽을 수 있는 이름.

Substrate 프레임워크는 확장 가능하므로, 체인 스펙을 커스텀하여 추가 정보를 포함시킬 수도 있습니다.
예를 들어, 외부 노드를 구성하여 특정 블록에 특정 높이에서 연결하도록 할 수 있어, genesis에서 새로운 노드를 동기화할 때 장거리 공격을 방지할 수 있습니다.

참고로, genesis 이후에도 외부 노드 설정을 커스텀할 수 있습니다.
하지만 노드는 동일한 `protocolId`를 사용하는 피어만 추가합니다.

## genesis 구성 커스텀하기

네트워크의 모든 노드는 후속 블록에 동의하기 전에 genesis 상태에 동의해야 합니다.
체인 스펙의 genesis 부분에 구성된 정보는 genesis 블록을 생성하는 데 사용됩니다.
이 정보는 command-line 옵션으로 재정의할 수 없으며, 블록체인을 시작할 때 설정되어야 합니다.
하지만 체인 스펙의 genesis 부분에 일부 정보를 구성할 수 있습니다.
예를 들어, 초기 토큰 보유자 잔액, 초기 거버넌스 위원회(Council)에 속하는 계정, `sudo` 키를 제어하는 관리 계정 등의 정보를 체인 스펙의 genesis 부분에 정의할 수 있습니다.

Substrate 노드는 체인의 런타임 로직을 위한 컴파일된 WebAssembly도 포함하므로, 체인 스펙에 초기 런타임도 제공되어야 합니다.

## 체인 스펙 정보 저장하기

체인 스펙의 정보는 Rust 코드나 JSON 파일로 저장할 수 있습니다.
Substrate 노드는 일반적으로 하나 이상의 하드코딩된 체인 스펙을 포함합니다.
노드 바이너리에 직접 Rust 코드로 이 정보를 포함시키면, 노드 운영자가 추가 정보를 제공하지 않아도 노드가 적어도 하나의 체인에 연결할 수 있습니다.
메인 네트워크를 정의하기 위해 블록체인을 구축하는 경우, 이 메인 네트워크 사양은 일반적으로 외부 노드에 하드코딩됩니다.

또는 `build-spec` 하위 명령을 사용하여 체인 스펙을 JSON 파일로 직렬화(serialize) 할 수 있습니다.
테스트 네트워크나 개인 체인을 시작할 때, 노드 바이너리와 함께 JSON으로 인코딩된 체인 스펙을 배포하는 것이 일반적입니다.

## 노드 시작에 체인 스펙 제공하기

노드를 시작할 때마다 노드가 사용할 체인 스펙을 제공해야 합니다.
가장 간단한 경우, 노드는 노드 바이너리에 하드코딩된 기본 체인 스펙을 사용합니다.
노드를 시작할 때 `--chain` command-line 옵션을 사용하여 대체 하드코딩된 체인 스펙을 선택할 수 있습니다.
예를 들어, `--chain local`을 command-line 옵션으로 지정하여 노드가 "local" 과 관련된 체인 스펙을 사용하도록 지시할 수 있습니다.

하드코딩된 체인 스펙으로 노드를 시작하고 싶지 않은 경우, JSON 파일로 제공할 수 있습니다.
예를 들어, `--chain=someCustomSpec.json`을 command-line 옵션으로 지정하여 노드가 `someCustomSpec.json` 파일의 체인 스펙을 사용하도록 지시할 수 있습니다.
JSON 파일을 지정하면, 노드는 제공된 JSON 체인 스펙을 역직렬화(deserialize) 하고 사용합니다.

## 런타임을 위한 스토리지 항목 선언하기

대부분의 경우, Substrate 런타임은 genesis에서 구성해야 할 일부 스토리지 항목을 필요로 합니다.
예를 들어, FRAME으로 런타임을 개발하는 경우, 런타임에서 `Config` 트레이트로 선언된 모든 스토리지 항목은 genesis에서 구성되어야 합니다.
이러한 스토리지 값은 체인 스펙의 genesis 부분에 구성됩니다.
팔렛에서 스토리지 항목에 초기 값을 설정하는 방법에 대한 정보는 [Genesis configuration](/build/genesis-configuration/)을 참조하십시오.

### 커스텀 체인 스펙 생성하기

개발, 테스트 또는 데모 목적으로 일회성 네트워크를 생성하려는 경우, 완전히 커스텀된 체인 스펙이 필요할 수 있습니다.
완전히 커스텀된 체인 스펙을 생성하려면, 기본 체인 스펙을 JSON 형식으로 내보내고, 그 후 JSON 파일의 필드를 편집하면 됩니다.
예를 들어, 다음과 같이 `build-spec` 하위 명령을 사용하여 체인 스펙을 JSON 파일로 내보낼 수 있습니다:

```bash
substrate build-spec > myCustomSpec.json
```

체인 스펙을 내보낸 후에는 텍스트 편집기에서 필드를 수정할 수 있습니다.
예를 들어, 네트워크 이름, 부트노드, 토큰 잔액과 같은 genesis 스토리지 항목을 변경하고 싶을 수 있습니다.
JSON 파일을 편집한 후에는 수정된 JSON을 사용하여 노드를 시작할 수 있습니다.
예를 들어:

```bash
substrate --chain=myCustomSpec.json
```

## 원시(Raw) 체인 스펙

Substrate 노드는 런타임 업그레이드를 지원합니다.
런타임 업그레이드를 통해 블록체인의 런타임은 체인이 시작될 때와 다를 수 있습니다.
체인 스펙은 노드의 런타임이 이해할 수 있는 형식으로 구조화된 정보를 포함합니다.
예를 들어, Substrate 노드 템플릿의 기본 체인 스펙에서 다음과 같은 일부분을 살펴보십시오:

```json
"sudo": {
  "key": "5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY"
}
```

JSON 파일에서 이 키와 관련된 값은 사람이 읽을 수 있는 텍스트입니다.
하지만 이 정보는 Substrate가 사용하는 기본 저장소 구조에 이 형식으로 저장할 수 없습니다.
노드의 genesis 스토리지를 초기화하기 위해 체인 스펙을 사용하려면, 사람이 읽을 수 있는 키를 실제 저장소 키로 변환하여 값이 [스토리지](/learn/state-transitions-and-storage/)에 저장될 수 있도록 해야 합니다.
이 변환은 간단하지만, 체인 스펙이 노드 런타임이 읽을 수 있는 형식으로 인코딩되어 있어야 한다는 점에 유의해야 합니다.

업그레이드된 런타임을 가진 노드가 genesis부터 체인을 동기화할 수 있도록 하기 위해, 사람이 읽을 수 있는 체인 스펙은 **원시(raw)** 형식으로 인코딩됩니다.
원시(Raw) 형식을 사용하면 런타임 업그레이드 후에도 모든 노드가 체인을 동기화하는 데 사용할 수 있는 체인 스펙을 배포할 수 있습니다.

Substrate 기반 노드는 `--raw` command-line 옵션을 지원하여 원시 체인 스펙을 생성할 수 있습니다.
예를 들어, 다음 명령을 실행하여 사람이 읽을 수 있는 `myCustomSpec.json` 파일에 대한 원시 체인 스펙을 생성할 수 있습니다:

```bash
substrate build-spec --chain=myCustomSpec.json --raw > customSpecRaw.json
```

원시(Raw) 형식으로 변환된 후, `sudo key` 스니펫은 다음과 같이 보입니다:

```json
"0x50a63a871aced22e88ee6466fe5aa5d9": "0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d",
```

## 다음 단계

- [신뢰할 수 있는 노드 추가](/tutorials/build-a-blockchain/add-trusted-nodes/)
- [가이드: genesis 상태 구성](/reference/how-to-guides/basics/configure-genesis-state/)
- [가이드: 체인 스펙 커스텀](/reference/how-to-guides/basics/customize-a-chain-specification/)
- [노드 템플릿 체인 스펙](https://github.com/substrate-developer-hub/substrate-node-template/blob/master/node/src/chain_spec.rs)
- [ChainSpec 구조체](https://paritytech.github.io/substrate/master/sc_service/struct.GenericChainSpec.html)
- [ProtocolId 구조체](https://paritytech.github.io/substrate/master/sc_network/config/struct.ProtocolId.html)