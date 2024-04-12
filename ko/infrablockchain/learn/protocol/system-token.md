---
title: 시스템 토큰
description: 시스템 토큰에 대한 전반적인 내용을 다룹니다.
keywords:
  - 시스템 토큰
---

**_인프라블록체인(InfraBlockchain)_** 에는 자체적으로 발행되는 네이티브 가상화폐 토큰(e.g 비트코인의 BITCOIN, 이더리움의 ETHER)이 없고 법정화폐와 연동된 **시스템 토큰(System Token)** 을 발행하여 트랜잭션 수수료로 사용됩니다. 또한 멀티체인기반으로 이루어져있어 다른 네트워크에서 발행된 시스템 토큰을 가져와 트랜잭션 수수료로 사용될 수 있습니다. 이를 위해 시스템 토큰의 식별 체계는 멀티체인에 맞게 설계되어 있습니다:

## 시스템 토큰 식별자

멀티체인상에 존재하는 모든 주체(e.g 토큰, 계정 등)은 상대적인 위치를 갖고 있습니다. 예를 들면, 파라체인 A 에서 바라본 (릴레이 체인에 존재하는) 토큰 X 의 위치와 릴레이 체인에서 바라본 토큰 X의 위치는 서로 다르게 표현됩니다. 이를 반영하여 각 체인에서 발행된 시스템 토큰 위치도 이러한 개념을 반영하여 표현됩니다.

```rust
pub struct MultiLocation {
	parents: u8,
	interior: Junctions,
}
```

`parents`: 상위 폴더와 같은 개념으로 서로 다른 컨센서스를 갖는 네트워크의 뎁스(depth)를 의미합니다. 예를 들면, 위의 토큰 X 의 위치는 파라체인내에서는 `parents = 1` 이지만 릴레이 체인 입장에서는 `parents = 0` 이 됩니다. 인프라 블록체인에서 릴레이 체인은 파라체인보다 항상 상위 개념으로 존재합니다.

`interior`: 하위 폴더와 같은 개념으로 같은 네트워크 상에서의 뎁스를 의미합니다. 예를 들면, 토큰 X 의 위치는 `interior = PalletInstance(0) -> GeneralIndex(0)` 식으로 표현될 수 있습니다.

![SystemTokenId](/media/images/docs/infrablockchain/learn/protocol/system_token_id.png)

토큰 X의 상대적 위치를 종합해보면,

- 릴레이 체인:
  ```rust
  MultiLocation {
  	parents: 0,
  	interior: Junctions::X2(PalletInstance(0), GeneralIndex(0))
  };
  ```
- 파라체인:
  ```rust
  MultiLocation {
  	parents: 1,
  	interior: Junctions::X2(PalletInstance(1), GeneralIndex(0))
  };
  ```

## 시스템 토큰의 특징

- 시스템 토큰은 법정화폐에 연동된(Fiat-pegged) 토큰이며 실제 화폐가 담보된 양 만큼만 체인 상에 발행될 수 있습니다.

- 시스템 토큰은 오라클에 의한 실시간 환율정보에 기반하여 법정 화폐별로 다른 가치를 반영하고 있습니다.

- 사용자는 시스템 토큰을 사용하여 트랜잭션 수수료를 지불할 수 있으며, 이 수수료는 [밸리데이터에 대한 투표 형태](./proof-of-transaction.md)로 변환됩니다.

## 랩드(Wrapped) 시스템 토큰

![Wrapped System Token](/media/images/docs/infrablockchain/learn/protocol/wrapped.png)

릴레이 체인 밸리데이터에 의해 채택된 시스템 토큰에 대한 wrapped 를 의미합니다. Wrapped 를 사용하기 위해서는 릴레이 체인 거버넌스의 승인을 받아야 하며 승인이 되면 토큰을 사용하려고 하는 네트워크에 해당 시스템 토큰의 메타데이터(e.g 소수점자리, 토큰 심볼, 토큰 이름 등)를 포함한 wrapped 토큰이 생성됩니다.

### 외부 토큰(Foreign Asset)

Wrapped 에 대한 등록을 마친 후 해당 토큰을 XCM 을 통해 받게되면 `ForeignAsset` 으로 저장됩니다. `ForeignAsset` 은 자신이 생성한 토큰이 아닌 외부 토큰을 관리하는 모듈이며 `SystemTokenId(=MultiLocation)` 를 Id 로 토큰을 관리하게 됩니다. 또한 해당 모듈에는 시스템 토큰 뿐만 아니라 외부의 어떤 자산이든 들어올 수 있습니다.

## 오라클

![오라클 노드](/media/images/docs/infrablockchain/learn/protocol/oracle.png)

시스템 토큰은 법정 화폐기반 토큰으로 환율에 기반하여 서로 다른 가치를 가지고 있습니다. 환율 정보는 오프체인 데이터이며 릴레이 체인 거버넌스에 의해 선택된 오라클이 정보를 제공합니다. 특정 주기마다 현재 유통되어 있는 시스템 토큰 법정 화폐에 한해 환율 정보를 제공하며 이때마다 각 [시스템 토큰의 가중치](./transaction-fee.md#시스템-토큰-가중치)를 다시 계산해주어 반영됩니다.

인프라블록체인의 시스템 체인인 _Asset Hub_ 에서 오라클 노드가 기준이 되는 토큰 대비 환율 정보를 주기적으로(일반적으로는 하루 단위) 업데이트하여 릴레이 체인에게 전달하게 됩니다. 환율이 업데이트될때마다 관련된 통화를 사용하고 있는 체인에 업데이트 됩니다.

예를 들면, 현재 기준이 되는 통화가 `USD` 이고 `KRW` 관련 시스템 토큰을 사용하고 있는 체인이 릴레이 체인과 파라체인 A,B 라고 하면, `USD` 대비 환율 정보를 매일 업데이트하여 각 체인에 업데이트 정보를 전달 후 업데이트하게 됩니다. 시스템 토큰과 관련된 모든 사항은 릴레이 체인의 [`SystemTokenManager`](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/master/infrablockchain/runtime/parachains/src/system_token_manager.rs) 팔렛에서 관리합니다.

## 다음으로 넘어가기

- [트랜잭션 증명(Proof-of-Transaction, PoT)](./proof-of-transaction.md)
- [SystemTokenManager 모듈](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/master/infrablockchain/runtime/parachains/src/system_token_manager.rs)
