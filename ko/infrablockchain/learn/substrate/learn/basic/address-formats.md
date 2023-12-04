---
title: 주소 형식
description: Substrate 기반 체인의 계정에 대한 기본 SS58 주소 형식에 대한 기술적 사양을 제공합니다.
keywords:
  - 주소 형식
  - 계정 형식
  - 네트워크별 계정
  - SS58
---

기본 Substrate 주소 형식은 SS58입니다. SS58 인코딩된 주소 형식은 비트코인 Base-58-check 형식을 기반으로 하지만, Substrate 기반 체인에 맞게 몇 가지 수정이 되었습니다. Substrate 기반 체인에서 다른 주소 형식을 사용할 수도 있습니다.

하지만, SS58 주소 형식은 어떤 Substrate 체인에서도 특정 계정을 식별할 수 있는 base-58 인코딩된 값을 제공합니다.
다른 체인은 계정을 식별하는 다른 방법을 가질 수 있기 때문에, SS58 주소는 확장 가능하도록 설계되었습니다.

## 기본 형식

SS58 주소 형식의 구현은 [Ss58Codec](https://paritytech.github.io/substrate/master/sp_core/crypto/trait.Ss58Codec.html)에서 찾을 수 있습니다.

주소의 기본 형식은 다음과 같이 설명할 수 있습니다:

```text
base58encode ( concat ( <주소-유형>, <주소>, <체크섬> ) )
```

주소는 주소 유형, 인코딩된 주소, 체크섬으로 구성된 연결된 바이트 시퀀스입니다. 이 시퀀스는 base-58 인코더에 전달됩니다.
`base58encode` 함수는 비트코인 및 IPFS 사양과 정확히 동일하게 구현되며, 두 구현 모두와 동일한 알파벳을 사용합니다.
Base-58 알파벳은 인쇄 시 모호하게 보일 수 있는 문자들을 제거합니다. 예를 들어:

- 알파벳이 아닌 문자 (+와 /)
- 숫자 0
- 대문자 i (I)
- 대문자 o (O)
- 소문자 l (l)

## 주소 유형

SS58 주소 형식의 `주소-유형`은 주소 바이트 뒤를 설명하는 하나 이상의 바이트입니다.

현재 유효한 값은 다음과 같습니다:

- `00000000b..=00111111b` (0에서 63까지 포함)

  간단한 계정/주소/네트워크 식별자입니다.
  이 바이트는 해당 식별자로 직접 해석될 수 있습니다.

- `01000000b..=01111111b` (64에서 127까지 포함)

  전체 주소/주소/네트워크 식별자입니다.
  이 바이트의 하위 6비트는 14비트 식별자 값의 상위 6비트로 취급되어야 하며, 하위 8비트는 다음 바이트에서 정의됩니다.
  이는 2\*\*14 (16,383)까지의 모든 식별자에 대해 작동합니다.

- `10000000b..=11111111b` (128에서 255까지 포함)

  미래의 주소 형식 확장을 위해 예약되어 있습니다.
  후자(42) 주소는 고정 길이 주소를 지원하는 모든 Substrate 네트워크에서 유효한 것으로 의도되어 있습니다.
  그러나 실제 네트워크에서는 키 재사용으로 인해 발생할 수 있는 문제를 피하기 위해 네트워크별 버전이 바람직할 수 있습니다. Substrate 노드는 기본적으로 주소 유형 42로 키를 출력합니다.
  그러나 Polkadot 생태계와 같은 대체 노드 구현을 가진 Substrate 기반 체인은 다른 주소 유형을 기본값으로 설정할 수 있습니다.

## 바이트 단위 주소 길이

체크섬을 포함한 페이로드의 바이트 단위 길이에 따라 16가지 다른 주소 형식이 있습니다.

| 총 길이 | 유형 | 원시(raw) 계정 | 체크섬 |
| ------- | ---- | --------- | ------ |
| 3       | 1    | 1         | 1      |
| 4       | 1    | 2         | 1      |
| 5       | 1    | 2         | 2      |
| 6       | 1    | 4         | 1      |
| 7       | 1    | 4         | 2      |
| 8       | 1    | 4         | 3      |
| 9       | 1    | 4         | 4      |
| 10      | 1    | 8         | 1      |
| 11      | 1    | 8         | 2      |
| 12      | 1    | 8         | 3      |
| 13      | 1    | 8         | 4      |
| 14      | 1    | 8         | 5      |
| 15      | 1    | 8         | 6      |
| 16      | 1    | 8         | 7      |
| 17      | 1    | 8         | 8      |
| 35      | 1    | 32        | 2      |

## 체크섬 유형

Substrate에서는 여러 가지 체크섬 전략이 있으며, 길이와 유효성 보장이 다릅니다.
SS58 및 AccountID로 알려진 체크섬 원본과 1바이트에서 8바이트까지 다양한 체크섬 길이가 있습니다.

Substrate의 모든 경우에는 Blake2b-512 (Spec, Wiki) 해시 함수가 사용됩니다 (OID 1.3.6.1.4.1.1722.12.2.1.16).
다른 변형은 해시 함수에 사용되는 원본과 출력에서 가져올 바이트 수를 선택합니다.

사용되는 바이트는 항상 가장 왼쪽의 바이트입니다.
사용할 입력은 base-58 함수에 입력으로 사용되는 SS58 바이트 시리즈의 체크섬이 아닌 부분입니다. 예를 들어 `concat( <주소-유형>, <주소> )`와 같습니다.
해싱 원본에 최종적으로 사용되기 위해 입력 앞에 0x53533538505245 (문자열 SS58PRE) 컨텍스트 접두사가 추가됩니다.

더 많은 체크섬 바이트를 사용하는 장점은 입력 오류 및 인덱스 변경에 대한 보호 수준을 더 많은 바이트가 제공하며, 이는 몇 개의 추가 문자로 텍스트 주소를 넓히는 비용으로 이어집니다.
계정 ID 형식의 경우, 이는 무시할 만한 것이므로 1바이트 대체가 제공되지 않습니다.
더 짧은 계정 인덱스 형식의 경우, 추가 바이트는 최종 주소의 상당 부분을 나타내므로 최적의 트레이드오프를 결정하기 위해 상위 스택(사용자 자신이 아닐 수도 있음)에서 결정하는 것으로 남겨집니다.

## 주소 유형 및 네트워크 레지스트리

[SS58 레지스트리](https://github.com/paritytech/ss58-registry)는 모든 주소 유형 식별자와 이들이 Substrate 기반 네트워크에 어떻게 매핑되는지에 대한 권위 있는 목록입니다.

## 주소 및 네트워크 식별자 인코딩

값 64까지의 식별자는 간단한 주소 형식을 사용하여 표현할 수 있습니다.
간단한 주소 형식의 경우, 네트워크 식별자 값의 가장 낮은 유효 바이트는 인코딩된 주소의 첫 번째 바이트로 표현됩니다.

64에서 16,383 사이의 식별자의 경우, 전체 주소 형식을 사용해야 합니다.

전체 주소 인코딩은 SCALE 인코딩이 리틀 엔디언으로 작동하기 때문에 첫 번째 두 비트를 01 접두사로 사용해야 하는 특별한 처리가 필요합니다.
네트워크 식별자를 인코딩하기 위해 전체 주소 형식은 첫 번째 두 바이트를 16비트 시퀀스로 취급하고, 이 시퀀스의 첫 번째 두 비트를 01 접두사를 위해 무시합니다.
나머지 14비트는 리틀 엔디언으로 네트워크 식별자 값을 인코딩하며, 두 개의 높은 순서 비트가 0임을 가정합니다.
이로써 하위 바이트가 두 바이트 사이의 경계를 가로지르도록 효과적으로 구현됩니다.

예를 들어, 14비트 식별자 `0b00HHHHHH_MMLLLLLL`은 두 바이트로 다음과 같이 표현됩니다:

```text
0b01LLLLLL
0bHHHHHHMM
```

16384 이상의 식별자는 현재 지원되지 않습니다.

## 주소 유효성 검사

`subkey inspect` 명령어나 Polkadot-JS API를 사용하여 값이 유효한 SS58 주소인지 확인할 수 있습니다.

### subkey 사용

`subkey inspect` 명령어의 기본 구문은 다음과 같습니다:

```text
subkey inspect [flags] [options] uri
```

`uri` 명령행 인수에는 비밀 시드 구문, 16진수로 인코딩된 개인 키 또는 SS58 주소를 지정할 수 있습니다.
입력이 유효한 주소인 경우, `subkey` 프로그램은 해당하는 16진수로 인코딩된 공개 키, 계정 식별자 및 SS58 주소를 표시합니다.
예를 들어, 비밀 시드 구문에서 파생된 공개 키를 검사하려면 다음과 유사한 명령을 실행할 수 있습니다:

```bash
subkey inspect "caution juice atom organ advance problem want pledge someone senior holiday very"
```

명령은 다음과 유사한 출력을 표시합니다:

```text
Secret phrase `caution juice atom organ advance problem want pledge someone senior holiday very` is account:
  Secret seed:       0xc8fa03532fb22ee1f7f6908b9c02b4e72483f0dbd66e4cd456b8f34c6230b849
  Public key (hex):  0xd6a3105d6768e956e9e5d41050ac29843f98561410d3a47f9dd5b3b227ab8746
  Public key (SS58): 5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR
  Account ID:        0xd6a3105d6768e956e9e5d41050ac29843f98561410d3a47f9dd5b3b227ab8746
  SS58 Address:      5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR
```

`subkey` 프로그램은 주소가 공개/비공개 키 쌍을 기반으로 한다고 가정합니다.
주소를 검사하면 명령은 32바이트 계정 식별자를 반환합니다.
그러나 Substrate 기반 네트워크의 모든 주소가 키를 기반으로 하는 것은 아닙니다.

지정한 옵션과 입력에 따라 명령 출력에는 주소가 인코딩된 네트워크도 표시될 수 있습니다.
예를 들어:

```bash
subkey inspect "12bzRJfh7arnnfPPUZHeJUaE62QLEwhK48QnH9LXeK2m1iZU"
```

명령은 다음과 유사한 출력을 표시합니다:

```text
Public Key URI `12bzRJfh7arnnfPPUZHeJUaE62QLEwhK48QnH9LXeK2m1iZU` is account:
  Network ID/Version: polkadot
  Public key (hex):   0x46ebddef8cd9bb167dc30878d7113b7e168e6f0646beffd77d69d39bad76b47a
  Account ID:         0x46ebddef8cd9bb167dc30878d7113b7e168e6f0646beffd77d69d39bad76b47a
  Public key (SS58):  12bzRJfh7arnnfPPUZHeJUaE62QLEwhK48QnH9LXeK2m1iZU
  SS58 Address:       12bzRJfh7arnnfPPUZHeJUaE62QLEwhK48QnH9LXeK2m1iZU
```

### Polkadot-JS API 사용

JavaScript 또는 TypeScript 프로젝트에서 주소를 확인하려면 Polkadot-JS API에 내장된 함수를 사용할 수 있습니다.
예를 들어:

```bash
// Polkadot.js API 종속성 가져오기.
const { decodeAddress, encodeAddress } = require('@polkadot/keyring')
const { hexToU8a, isHex } = require('@polkadot/util')

// 테스트할 주소 지정.
const address = '<addressToTest>'

// 주소 확인.
const isValidSubstrateAddress = () => {
  try {
    encodeAddress(isHex(address) ? hexToU8a(address) : decodeAddress(address))

    return true
  } catch (error) {
    return false
  }
}

// 쿼리 결과.
const isValid = isValidSubstrateAddress()
console.log(isValid)
```

함수가 `true`를 반환하면 지정한 주소가 유효한 주소입니다.

### 다른 SS58 구현

Substrate SS58 주소의 인코딩 및 디코딩을 지원하는 다른 언어 및 라이브러리도 구현되어 있습니다.

- Crystal: [`wyhaines/base58.cr`](https://github.com/wyhaines/base58.cr)
- Go: [`itering/subscan`](https://github.com/subscan-explorer/subscan-essentials/blob/master/util/ss58/ss58.go)
- Python: [`polkascan/py-scale-codec`](https://github.com/polkascan/py-scale-codec/blob/master/scalecodec/utils/ss58.py)
- TypeScript: [`subsquid/squid-sdk`](https://github.com/subsquid/squid-sdk/tree/master/substrate/ss58-codec)