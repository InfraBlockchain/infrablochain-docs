---
title: 애플리케이션 개발
description: Substrate 메타데이터와 RPC 라이브러리가 애플리케이션 인터페이스 구축에 어떻게 사용되는지 살펴봅니다.
keywords:
  - 프론트엔드
  - 애플리케이션
  - dApp
---

블록체인 개발자로서, 프론트엔드 애플리케이션을 구축하는 데 참여하지 않을 수도 있습니다. 그러나 블록체인에서 실행되는 대부분의 애플리케이션은 블록체인이 저장하는 데이터에 사용자 또는 다른 프로그램이 접근하고 수정할 수 있도록 프론트엔드 또는 사용자 인터페이스 클라이언트의 형태를 필요로 합니다.

예를 들어, 사용자가 트랜잭션을 제출하거나 글을 게시하거나 자산을 확인하거나 이전 활동을 추적할 수 있는 브라우저 기반, 모바일 또는 데스크톱 애플리케이션을 개발할 수 있습니다.
해당 애플리케이션의 백엔드는 블록체인의 런타임 로직으로 구성되지만, 런타임 기능을 사용자에게 제공하는 것은 프론트엔드 클라이언트입니다.

다른 사람들에게 유용한 블록체인 서비스를 제공하기 위해서는, 블록체인에 계속 저장되는 정보를 사용자가 볼 수 있거나 상호작용하거나 업데이트할 수 있는 클라이언트 애플리케이션을 제공해야 합니다.

해당 문서에서는 클라이언트 애플리케이션이 이러한 정보를 사용할 수 있도록 런타임(_블록체인 실행 환경_) 메타데이터를 가져오는 방법을 알아보고, 노출된 정보의 예시를 살펴보며, 이 정보를 사용하는 도구와 라이브러리를 탐색해보겠습니다.

## 런타임 정보를 메타데이터로 공개하기

Substrate 기반 블록체인에 저장되어 있는 정보와 상호작용하기 위해서는 체인에 연결하는 방법과 런타임이 외부에 공개하는 기능에 접근하는 방법을 알아야 합니다.
일반적으로 이 상호작용은 원격 RPC 를 통해 관심 있는 정보를 요청하거나 검색하는 것을 포함하고 있습니다.
그러나 애플리케이션 개발자로서, 일반적으로 다음과 같은 런타임 로직에 대한 상세 정보를 알아야 합니다:

- 애플리케이션이 연결하는 런타임의 버전.
- 런타임이 지원하는 API.
- 특정 런타임에 구현된 팔렛(pallet)들.
- 특정 런타임에 정의된 모든 함수와 그 함수들의 타입 시그니처.
- 특정 런타임에 정의된 모든 커스텀 타입들.
- 런타임이 사용자가 설정할 수 있도록 공개하는 모든 매개변수들.

Substrate는 모듈화되어 있고 조립 가능한 프레임워크를 제공하기 때문에, 미리 정의된 속성 스키마는 없습니다.
대신, 각 런타임은 고유한 속성으로 구성되며, 이러한 속성(함수와 타입 포함)은 업그레이드로 시간이 지남에 따라 변경될 수 있습니다.
이러한 런타임의 모든 정보는 런타임 **메타데이터** 스키마를 통해 공개됩니다.
런타임의 메타데이터는 특정 버전의 런타임에 정의된 모든 팔렛과 타입에 대한 정보를 나타냅니다. 
각 팔렛에 대해, 메타데이터에는 해당 팔렛의 스토리지 항목, 함수, 이벤트, 오류 및 상수에 대한 정보가 포함됩니다.
메타데이터에는 런타임에 포함된 커스텀 타입의 정의도 포함됩니다.

메타데이터는 런타임의 완전한 목록을 제공하기 때문에, 클라이언트 애플리케이션이 노드와 상호작용하고 응답을 구문 분석하고 메시지 페이로드를 형식화할 수 있도록 하는 열쇠 역할을 합니다.

## 메타데이터 생성하기

네트워크 상에서 데이터를 전송하는 데 필요한 대역폭을 최소화하기 위해, 메타데이터 스키마는 [SCALE 코덱 라이브러리](../learn/frame/scale-codec.md)를 사용하여 인코딩됩니다.
이 인코딩은 [`scale-info`](https://docs.rs/scale-info/latest/scale_info/) 를 사용하여 컴파일할 때 자동으로 수행됩니다.

메타데이터를 생성하는 과정은 다음 단계로 이루어집니다:

- 런타임 로직의 팔렛은 메타데이터에 인코딩되어야 하는 모든 호출 가능한 함수, 타입, 매개변수 및 문서를 공개합니다.

- `scale-info` 는 런타임의 팔렛에 대한 타입 정보를 수집하고, 특정 런타임에 존재하는 팔렛과 각 팔렛의 관련된 타입에 대한 레지스트리를 구축합니다.
  타입 정보는 모든 타입에 대한 인코딩 및 디코딩을 가능하게 하는 충분한 세부 정보를 제공합니다.

- [`frame-metadata`](https://github.com/paritytech/frame-metadata) 는 `scale-info` 가 제공하는 레지스트리를 기반으로 런타임의 구조를 설명합니다.

- Substrate 노드는 `state_getMetadata` RPC 메서드를 제공하여 현재 런타임의 모든 타입에 대한 완전한 설명을 16진수로 인코딩된 SCALE 바이트 벡터로 반환합니다.

다음 다이어그램은 런타임 로직이 컴파일되고 RPC 요청으로 노드에 연결되어 메타데이터가 생성되는 과정을 간략히 보여줍니다.

![런타임 컴파일로 메타데이터 생성](/media/images/docs/infrablockchain/learn/substrate/build/metadata.png)

## 런타임의 메타데이터 가져오기

런타임의 메타데이터를 가져오는 방법은 여러 가지가 있습니다.
예를 들어, 다음 중 하나를 수행할 수 있습니다:

- [Polkadot/Substrate Portal](https://polkadot.js.org/apps/#/rpc)을 사용하여 노드에 연결하고 **state** 엔드포인트와 **getMetadata** 메서드를 선택하여 메타데이터를 JSON 형식으로 받아낼 수 있습니다.
- 명령줄에서 `polkadot-js-api`를 사용하여 `state_getMetadata` RPC 메서드를 호출하여 메타데이터를 16진수로 인코딩된 SCALE 바이트 벡터로 받아낼 수 있습니다.
- `subxt metadata` 명령을 사용하여 메타데이터를 JSON, 16진수 또는 원시 바이트(raw bytes) 로 다운로드할 수 있습니다.
- `sidecar` API와 `/runtime/metadata` 엔드포인트를 사용하여 노드에 연결하고 메타데이터를 JSON 형식으로 받아낼 수 있습니다.

메타데이터가 제공하는 타입 정보를 통해 애플리케이션은 서로 다른 런타임 버전, 트랜잭션, 이벤트, 타입 그리고 스토리지를 갖고 있는 체인들과 인터랙션을 할 수 있습니다.
메타데이터는 또한 라이브러리가 주어진 Substrate 노드와 통신하기 위해 필요한 거의 모든 코드를 생성할 수 있도록 합니다. 이를 통해 `subxt`와 같은 라이브러리는 대상 체인에 특화된 프론트엔드 인터페이스를 생성할 수 있습니다.

## 클라이언트 애플리케이션과 메타데이터

클라이언트 애플리케이션은 메타데이터를 사용하여 노드와 상호작용하고 응답을 구문 분석하며 노드에 전송되는 메시지 페이로드(e.g 트랜잭션) 를 생성합니다. 
메타데이터를 사용하기 위해서는 클라이언트 애플리케이션에서 [SCALE 코덱 라이브러리](../learn/frame/scale-codec.md)를 사용하여 RPC 페이로드를 인코딩하고 디코딩해야 합니다.
메타데이터는 모든 타입이 어떻게 디코딩되어야 하는지 공개하기 때문에, 애플리케이션은 수동으로 인코딩 및 디코딩하지 않고도 애플리케이션 정보를 전송, 검색 및 처리할 수 있습니다.

## 메타데이터 형식

SCALE로 인코딩된 바이트는 `frame-metadata`와 [`parity-scale-codec`](https://github.com/paritytech/parity-scale-codec) 라이브러리를 사용하여 디코딩할 수 있지만, `subxt` 및 Polkadot-JS API와 같은 다른 도구를 사용하여 원시 데이터(raw data)를 사람이 읽을 수 있는 JSON 형식으로 변환할 수도 있습니다.

`state_getMetadata` RPC 호출로 반환되는 메타데이터에 포함된 타입 및 타입 정의는 런타임의 메타데이터 버전에 따라 다릅니다.
일반적으로 메타데이터에는 다음과 같은 정보가 포함됩니다:

- 메타데이터 파일을 메타데이터로 식별하는 상수.
- 런타임에서 사용하는 메타데이터 형식의 버전.
- `scale-info`에서 생성된 런타임에서 사용되는 모든 타입에 대한 정의.
- `construct_runtime` 에서 정의된 순서대로 런타임에 포함된 모든 팔렛에 대한 정보.

다음 예시는 디코딩되고 JSON으로 변환된 메타데이터의 압축된 주석 부분을 보여줍니다:

```json
[
  1635018093,
  {
    "V14": {
      "types": {
        "types": [
          {
            // 타입 인덱스
          }
        ]
      },
      "pallets": [
        {
          // 팔렛 인덱스 및 각 팔렛이 공개하는 메타데이터
        }
      ],
      "extrinsic": {
        "ty": 126, // 익스트린식 형식을 정의하는 타입 인덱스 식별자
        "version": 4, // 익스트린식을 인코딩 및 디코딩하는 데 사용되는 트랜잭션 버전
        "signed_extensions": [
          {
            // 서명된 확장 인덱스
          }
        ]
      },
      "ty": 141 // 시스템 팔렛의 타입 ID
    }
  }
]
```

상수 `1635018093`은 메타데이터 파일을 메타데이터 파일로 식별하는 매직 넘버입니다.
나머지 메타데이터는 `types`, `pallets` 및 `extrinsic` 섹션으로 나누어집니다.
`types` 섹션에는 타입 인덱스에 대한 인덱스가 포함되어 있으며, 각 타입에 대한 타입 시그니처에 대한 정보가 제공됩니다.
`pallets` 섹션에는 런타임에 포함된 각 팔렛에 대한 정보가 포함되어 있습니다.
`extrinsic` 섹션에는 런타임이 사용하는 타입 식별자와 트랜잭션 형식 버전이 포함됩니다.
특정 익스트린식 버전은 특히 [서명된 트랜잭션](../learn/basic/transaction-types.md)을 고려할 때 서로 다른 형식을 가질 수 있습니다.

### 팔렛

다음은 `pallets` 배열의 단일 요소의 압축된 주석 예시입니다:

```json
{
  "name": "Sudo",        // 팔렛의 이름
  "storage": {           // 팔렛의 스토리지 정보
      "prefix": "Sudo",  // 팔렛 스토리지 항목의 데이터베이스 접두사
      "entries": [
        {
          "name": "Key",
          "modifier": "Optional",
          "ty": {
             "Plain": 0
          },
          "default": [
             0
          ],
          "docs": [
             "sudo 키의 `AccountId`입니다."
          ]
        }
      ]
  },
  "calls": {       // 팔렛 호출 타입
      "ty": 117    // 타입 인덱스 식별자
  },
  "event": {       // 팔렛 이벤트 타입
      "ty": 42     // 타입 인덱스 식별자
  },
  "constants": [], // 팔렛 상수
  "error": {       // 팔렛 오류 타입
      "ty": 124    // 타입 인덱스 식별자
          },
  "index": 8       // 런타임에서 팔렛의 인덱스 식별자
},
```

각 요소에는 해당 팔렛을 나타내는 이름과 스토리지, 호출, 이벤트 및 오류에 대한 정보가 포함됩니다.
호출, 이벤트 및 오류의 정의에 대한 자세한 내용은 타입 인덱스 식별자를 참조하여 확인할 수 있습니다.
각 항목의 타입 인덱스 식별자는 해당 항목의 타입 정보에 액세스하는 데 사용되는 `u32` 정수입니다.
예를 들어, Sudo 팔렛의 호출에 대한 타입 인덱스 식별자는 117입니다.
메타데이터의 `types` 섹션에서 해당 타입 식별자에 대한 정보를 보면, 각 호출에 대한 문서를 포함한 사용 가능한 호출에 대한 정보를 제공합니다.

예를 들어, 다음은 Sudo 팔렛의 호출에 대한 압축된 예시 일부입니다:

```json
    {
      "id": 117,
      "type": {
          "path": [
              "pallet_sudo",
              "pallet",
              "Call"
          ],
          "params": [
            {
              "name": "T",
              "type": null
            }
          ],
          "def": {
              "variant": {
                  "variants": [
                    {
                      "name": "sudo",
                      "fields": [
                        {
                          "name": "call",
                          "type": 114,
                          "typeName": "Box<<T as Config>::RuntimeCall>"
                        }
                  ],
                      "index": 0,
                      "docs": [
                        "sudo 키를 인증하고 `Root` 출처로 함수 호출을 디스패치합니다.",
                      ]
                    },
                    {
                      "name": "sudo_unchecked_weight",
                      "fields": [
                        {
                          "name": "call",
                          "type": 114,
                          "typeName": "Box<<T as Config>::RuntimeCall>"
                        },
                        {
                          "name": "weight",
                          "type": 8,
                          "typeName": "Weight"
                        }
                      ],
                      "index": 1,
                      "docs": [
                        "sudo 키를 인증하고 `Root` 출처로 함수 호출을 디스패치합니다.",
                      ]
                    },
                    {
                      "name": "set_key",
                      "fields": [
                        {
                          "name": "new",
                          "type": 103,
                          "typeName": "AccountIdLookupOf<T>"
                        }
                      ],
                      "index": 2,
                      "docs": [
                        "현재 sudo 키를 인증하고 주어진 AccountId(`new`)를 새로운 sudo로 설정합니다.",
                      ]
                    },
                    {
                      "name": "sudo_as",
                      "fields": [
                        {
                          "name": "who",
                          "type": 103,
                          "typeName": "AccountIdLookupOf<T>"
                        },
                        {
                          "name": "call",
                          "type": 114,
                          "typeName": "Box<<T as Config>::RuntimeCall>"
                        }
                      ],
                      "index": 3,
                      "docs": [
                        "sudo 키를 인증하고 주어진 계정에서 `Signed` 출처로 함수 호출을 디스패치합니다.",
                        "계정에서.",
                      ]
                    }
                  ]
                }
              },
            },
```

각 필드에 대해 스토리지 메타데이터, 호출 메타데이터, 이벤트 메타데이터 및 오류 메타데이터에 대한 다음과 같은 타입 정보 및 메타데이터에 액세스할 수 있습니다:

- 스토리지 메타데이터는 특정 스토리지 항목에 대한 정보를 애플리케이션이 가져올 수 있도록 하는 정보를 제공합니다.
- 호출 메타데이터에는 `#[pallet]` 매크로에 의해 정의된 런타임 호출에 대한 정보가 포함되며, 호출 이름, 인수 및 문서를 포함합니다.
- 이벤트 메타데이터는 `#[pallet::event]` 매크로에 의해 생성된 메타데이터를 제공하며, 각 팔렛 이벤트의 이름, 인수 및 문서를 포함합니다.
- 상수 메타데이터는 `#[pallet::constant]` 매크로에 의해 생성된 메타데이터를 제공하며, 상수의 이름, 타입 및 16진수로 인코딩된 값이 포함됩니다.
- 오류 메타데이터는 `#[pallet::error]` 매크로에 의해 생성된 메타데이터를 제공하며, 각 팔렛 오류의 이름과 문서를 포함합니다.

타입 식별자가 시간이 지남에 따라 변경될 수 있으므로 타입 식별자에 의존하는 것을 피해야 합니다.

### 익스트린식(Extrinsic)

런타임에서 생성된 익스트린식 메타데이터는 트랜잭션이 어떻게 포맷되는지에 대한 유용한 정보를 제공합니다.
디코딩되면, 메타데이터에는 트랜잭션 버전과 서명된 확장의 목록이 포함됩니다.
예를 들어:

```json
    "extrinsic": {
        "ty": 126,
        "version": 4,
        "signed_extensions": [
          {
            "identifier": "CheckNonZeroSender",
            "ty": 132,
            "additional_signed": 41
          },
          {
            "identifier": "CheckSpecVersion",
            "ty": 133,
            "additional_signed": 4
          },
          {
            "identifier": "CheckTxVersion",
            "ty": 134,
            "additional_signed": 4
          },
          {
            "identifier": "CheckGenesis",
            "ty": 135,
            "additional_signed": 11
          },
          {
            "identifier": "CheckMortality",
            "ty": 136,
            "additional_signed": 11
          },
          {
            "identifier": "CheckNonce",
            "ty": 138,
            "additional_signed": 41
          },
          {
            "identifier": "CheckWeight",
            "ty": 139,
            "additional_signed": 41
          },
          {
            "identifier": "ChargeTransactionPayment",
            "ty": 140,
            "additional_signed": 41
          }
        ]
      },
      "ty": 141
    }
  }
]
```

타입 시스템은 복합적입니다.
각 타입 식별자에는 특정 타입 또는 관련된 기본 타입에 대한 정보를 제공하는 다른 타입 식별자에 대한 참조가 포함됩니다.
예를 들어, `BitVec<Order, Store>` 타입을 인코딩할 수 있지만, 올바르게 디코딩하려면 `Order` 및 `Store` 타입에 사용되는 타입을 알아야 합니다.
`Order` 및 `Store`에 대한 타입 정보를 찾으려면 디코딩된 JSON의 경로를 사용하여 해당 타입 식별자를 찾을 수 있습니다.

## RPC API

Substrate에는 다음과 같은 API가 제공됩니다:

- [`AuthorApiServer`](https://paritytech.github.io/substrate/master/sc_rpc/author/trait.AuthorApiServer.html): 익스트린식 제출과 세션 키 검증을 포함한 풀노드(full node)로의 호출을 수행하는 API입니다.
- [`ChainApiServer`](https://paritytech.github.io/substrate/master/sc_rpc/chain/trait.ChainApiServer.html): 블록 헤더 및 확정(finality) 정보를 검색하는 API입니다.
- [`OffchainApiServer`](https://paritytech.github.io/substrate/master/sc_rpc/offchain/trait.OffchainApiServer.html): 오프체인 워커용 RPC 호출을 수행하는 API입니다.
- [`StateApiServer`](https://paritytech.github.io/substrate/master/sc_rpc/state/trait.StateApiServer.html): 런타임 버전, 스토리지 항목 및 증명과 같은 온체인 상태에 대한 정보를 쿼리하는 API입니다.
- [`SystemApiServer`](https://paritytech.github.io/substrate/master/sc_rpc/system/trait.SystemApiServer.html): 연결된 피어 및 노드 역할과 같은 네트워크 상태에 대한 정보를 검색하는 API입니다.

## 노드에 연결하기

일반적으로 애플리케이션은 공개된(open) HTTP 또는 WebSocket 포트를 통해 JSON-RPC 메서드를 사용하여 Substrate 노드에 연결합니다.
대부분의 애플리케이션은 WebSocket 포트를 사용하는 것이 일반적입니다. 하나의 연결로 노드와의 여러 메시지를 보내고 받을 수 있기 때문입니다.
HTTP 연결의 경우 애플리케이션은 한 번에 하나의 메시지만 보내고 받을 수 있습니다.
HTTP를 사용하여 노드에 연결하는 가장 일반적인 이유는 오프체인 워커를 사용하여 데이터를 가져오고자 하는 경우입니다.
오프체인 워커 사용에 대한 자세한 내용은 [오프체인 작업](../learn/basic/offchain-operations.md)을 참조하십시오.

RPC를 사용하여 연결하는 대신, [Substrate Connect](https://substrate.io/developers/substrate-connect/)와 라이트 클라이언트(Light Client) 노드를 사용하여 Substrate 기반 블록체인에 연결할 수도 있습니다.
Substrate Connect는 브라우저에서 실행되며, 애플리케이션이 자체 라이트 클라이언트 노드를 생성하고 노출된 JSON-RPC 엔드포인트에 직접 연결할 수 있도록 합니다.
Substrate Connect가 내장된 애플리케이션은 브라우저 내부 로컬 메모리를 활용하여 라이트 클라이언트 노드와 연결합니다.

## 프론트엔드 애플리케이션 구축하기

다음 라이브러리는 [JSON-RPC API](https://github.com/paritytech/jsonrpsee)를 사용하여 애플리케이션이 Substrate 노드와 상호작용할 수 있도록 합니다:

| 이름                                                             | 설명                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | 언어       |
| :--------------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------- |
| [Chain API](https://github.com/paritytech/capi)                  | Substrate 기반 체인과 상호 작용하기 위한 TypeScript 툴킷을 제공합니다. 해당 툴킷에는 FRAME 유틸리티, 기능적 효과 시스템 및 성능 또는 안전성을 저해하지 않고 최종 사용자를 위한 다단계, 다중 체인 상호 작용을 용이하게 하는 API가 포함되어 있습니다.                                                                                                                                                                                                                         |
| [Polkadot JS API](https://polkadot.js.org/docs/api)              | Substrate 기반 체인과 상호 작용할 때 노드 변경에 동적으로 적응할 수 있는 애플리케이션을 구축하기 위한 JavaScript 라이브러리를 제공합니다. 해당 라이브러리는 React 와 같은 인기 있는 프론트엔드 프레임워크와 함께 사용할 수 있습니다.                                                                                                                                                                                                                                                                                       | Javascript |
| [Polkadot JS extension](https://polkadot.js.org/docs/extension/) | Polkadot JS API로 작성된 브라우저 확장 프로그램 및 제공자와 상호 작용하기 위한 API를 제공합니다.                                                                                                                                                                                                                                                                                                                                                                                                                             | Javascript |
| [Substrate Connect](/learn/light-clients-in-substrate-connect/)  | 브라우저 내 라이트 클라이언트 노드를 사용하여 Substrate 기반 체인에 직접 연결하는 애플리케이션을 구축하기 위한 라이브러리 및 브라우저 확장 프로그램을 제공합니다. Substrate Connect를 사용하면 사용자가 여러 체인과 상호 작용하기 위해 애플리케이션을 사용하는 경우 단일 경험을 제공할 수 있습니다.                                                                                                                                                                                            | Javascript |
| [`subxt`](https://github.com/paritytech/subxt/)                  | 대상 체인의 메타데이터를 기반으로 노드의 RPC API와 상호 작용하기 위한 정적으로 타입 지정된 Rust 인터페이스를 생성하는 라이브러리입니다. `subxt` (submit extrinsic) 라이브러리를 사용하면 노드와 생성된 인터페이스 간에 안전한 통신이 필요한 하위 수준 애플리케이션 (예: 브라우저가 아닌 그래픽 사용자 인터페이스, 체인별 CLI 또는 트랜잭션에 잘못된 입력을 생성하지 못하도록 하거나 존재하지 않는 호출을 제출하지 못하도록 하는 사용자 지향 애플리케이션)을 구축할 수 있습니다. | Rust       |
| [`txwrapper`](https://github.com/paritytech/txwrapper)           | 오프라인에서 서명된 Substrate 트랜잭션을 생성하기 위한 JavaScript 라이브러리를 제공합니다. 이 라이브러리를 사용하면 오프라인에서 서명된 트랜잭션을 생성하는 스크립트를 작성할 수 있으며, 이후에 노드에 제출할 수 있습니다. 이 기능은 특히 테스트 및 트랜잭션 디코딩에 유용합니다.                                                                                                                                                                                                                                 | Javascript |

JSON-RPC API 및 최신 인터페이스 사양에 대한 자세한 정보는 [JSON-RPC 사양](https://paritytech.github.io/json-rpc-interface-spec/)을 참조하십시오.

## 다음 단계로 넘어가기

- [Substrate Connect](https://github.com/paritytech/substrate-connect)
- [프론트엔드 템플릿 설치](/tutorials/build-a-blockchain/build-local-blockchain/#install-the-front-end-template)
- [메타데이터 QR 코드 생성](https://github.com/paritytech/metadata-portal)
- [하위 호환성 있는 메타데이터 가져오기 (desub)](https://github.com/paritytech/desub)