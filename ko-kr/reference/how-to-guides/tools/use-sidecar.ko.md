---
title: REST 엔드포인트를 사용하여 체인 데이터 가져오기
description:
keywords:
  - 노드
  - 클라이언트
  - 도구
---

이 안내서는 Substrate 블록체인 노드와 상호 작용하기 위해 `sidecar` 서비스에서 제공하는 REST 엔드포인트를 사용하는 방법을 설명합니다.

완료된 경매의 우승자를 찾기 위해서는 경매가 종료된 블록 번호를 알아야 합니다. [Sidecar](https://github.com/paritytech/substrate-api-sidecar)는 상태를 유지하지 않는 API이며 경매 정보는 경매의 마지막 블록에 저장되므로 경매가 종료되면 이벤트와 그 안에 저장된 데이터를 검색하기 위해 블록 번호가 필요합니다.

## 목표

`sidecar`를 사용하여 완료된 파라체인 경매의 우승자 찾기

## 사용 사례

REST 서비스를 사용하여 Substrate 블록체인 노드와 상호 작용하기

## 절차

### 1. /experimental/paras/auctions/current 엔드포인트 활용

데이터베이스에 `finishEnd`, `auctionIndex`, `leasePeriods`를 추적하고 저장합니다:

- `finishEnd`: 이는 경매의 마지막 블록입니다. 이를 저장함으로써 경매가 종료된 블록을 쿼리할 수 있습니다. 해당 블록에서는 리스에 대한 이벤트를 추출할 수 있습니다. (블록 쿼리: GET /blocks/{finishEnd}.)

- `auctionIndex`: 경매의 고유 식별자입니다.

- `leasePeriods`: 특정 `auctionIndex`에 대해 입찰할 수 있는 사용 가능한 리스 기간 인덱스입니다.

### 2. Sidecar를 사용하여 경매 우승자 찾기

`finishEnd` 블록을 저장하고 그 안에 있는 `Leased` 이벤트를 확인함으로써 경매 우승자와 보상된 리스 기간을 확인할 수 있습니다.

필요에 따라 데이터를 형식화합니다. 예를 들면:

```js
auctionIndex: {
    leasePeriods: [
        "11", "12", "13", "14"
    ],
    finishEnd: '200'
}
```

### 3. /blocks/:blockId 엔드포인트 쿼리

이 단계에서는 `finishEnd` 필드에 지정된 블록 높이에 있는 모든 블록을 쿼리하고 `on_initialize` 내부의 모든 이벤트를 검색합니다. 예시 응답은 다음과 같습니다:

```js
{
    authorId: ....,
    extrinsics:....
    ...
    on_initialize: {
        events: [
            {
                "method": {
                    "pallet": "slots",
                    "method": "Leased"
                },
                "data": [
                    '1000', // ParaId
                    '5HpG9w8EBLe5XCrbczpwq5TSXvedjrBGCwqxK1iQ7qUsSWFc', // AccountId
                    '1', // LeasePeriod (리스 기간의 시작)
                    '4', // LeasePeriod (리스 기간의 개수)
                    '10000', // Balance (추가 예약 잔액)
                    '1000000000', // Balance (총 잔액)
                ]
            },
            {
                "method": {
                    "pallet": "auctions",
                    "method": "AuctionClosed"
                },
                "data": [
                    ...
                ]
            }
        ]
    }
}
```

### 4. 데이터 비교

이제 해당 경매의 슬롯을 획득한 `paraIds`를 모두 갖고 있으므로, 이를 `auctionIndex`와 관련된 데이터와 비교할 수 있습니다.
활성 경매 중에 사용 가능한 `leasePeriods`와 `Leased` 이벤트에서 획득하고 표시된 `leasePeriods`를 비교하면 해당 경매의 모든 우승자를 얻을 수 있습니다.

## 예시

- 더 많은 안내서는 [여기](https://github.com/paritytech/substrate-api-sidecar/tree/master/guides)에서 확인할 수 있습니다.

## 자원

- [Sidecar 문서](https://github.com/paritytech/substrate-api-sidecar)
- 사용 가능한 [Substrate Sidecar API 엔드포인트](https://paritytech.github.io/substrate-api-sidecar/dist/)