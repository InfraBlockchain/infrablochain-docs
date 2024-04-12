---
title: 시스템 토큰 등록 및 사용
description: 시스템 토큰 등록 절차 및 등록 후 사용 방법에 대한 내용을 다룹니다.
keywords:
---

**_인프라블록체인(InfraBlockchain)_** 은 시스템 토큰을 가스비로 사용하는 차별화된 블록체인 시스템을 제공합니다.
아래는 시스템 토큰의 등록, 사용 및 폐기과정에 대해 개발자들이 쉽게 이해하고 개발할 수 있도록 정리한 내용입니다.
해당 내용에 대한 실습은 [how-to-pay-transaction-fee](./how-to-pay-transaction-fee.md) 에서 진행할 수 있습니다.

## Bootstrap 단계

인프라 블록체인 런타임은 두가지 런타임 상태가 존재합니다. 처음 네트워크를 시작하면 `Bootstrap` 단계로 시스템 토큰이 아직 등록되지 않은 상태입니다. 부트 스트랩 단계에서는 수행할 수 있는 트랜잭션에 제한이 있습니다.

수행할 수 있는 트랜잭션 목록:

- `Assets::create` : 토큰을 생성하는 트랜잭션
- `Assets::set_metadata`: 생성된 토큰에 대한 메타데이터를 설정하는 트랜잭션
- `Assets::mint` : 토큰을 발행하는 트랜잭션(최소한으로 트랜잭션을 수행할 수 있는 토큰이 있어야 등록이 가능합니다)
- `InfraParaCore::request_register_system_token` : 시스템 토큰에 대한 등록을 요청하는 트랜잭션

_표기 `_::_`: `::` 앞은 팔렛 이름이며 뒤는 팔렛 내 트랜잭션 이름입니다._

## 시스템 토큰 등록

### 등록 요청 절차

- **토큰 생성**

  ![create_token](/media/images/docs/infrablockchain/tutorials/create_token.png)

- **메타데이터 설정**

  ![set_metadata](/media/images/docs/infrablockchain/tutorials/set_metadata.png)

- **토큰 발행**

  ![mint_token](/media/images/docs/infrablockchain/tutorials/mint_token.png)

- **시스템 토큰 등록 요청**

  해당 트랜잭션은 릴레이 체인에 의해 승인된 계정 혹은 파라체인 자체 거버넌스로 호출할 수 있습니다.

  ![request_register](/media/images/docs/infrablockchain/tutorials/request_register.png)

  이벤트

  ![request_register_event](/media/images/docs/infrablockchain/tutorials/request_register_event.png)

  _`exp` 는 등록 요청에 대한 만료 시간을 의미합니다._

해당 이벤트까지 확인됐으면 릴레이 체인으로 등록 요청에 대한 정보가 반영되고 릴레이 체인 밸리데이터의 거버넌스에 의해 승인 여부가 결정됩니다.

### 릴레이 체인 거버넌스 절차

**환율정보 가져오기**

모든 시스템 토큰은 법정 화폐기반이며 이에 대한 환율정보가 반영되어야 등록이 진행할 수 있습니다. 먼저 릴레이 체인에서 오라클 노드 쪽으로 `requestExchangeRate` 트랜잭션을 통해 데이터 요청을 보냅니다. 해당 트랜잭션은 거버넌스를 통해 이루어집니다.

![request_exchange_rate](/media/images/docs/infrablockchain/tutorials/request_exchange_rate.png)

해당 트랜잭션은 기준이 되는 시스템 토큰 대비 현재 등록이 완료된 시스템 토큰 혹은 등록 요청된 시스템 토큰의 환율 데이터를 요청하는 트랜잭션입니다. 트랜잭션이 정상적으로 실행되면 릴레이체인은 오라클 노드에 데이터를 요청하게 되고 오라클 노드에서는 다음과 같은 이벤트가 발생합니다.

![request_exchange_rate_event](/media/images/docs/infrablockchain/tutorials/request_exchange_rate_event.png)

환율 데이터를 외부에서 가져오게 되면 릴레이 체인으로 데이터를 제출한 후 다음과 같은 이벤트가 발생합니다.

![exchange_rate_submit_event](/media/images/docs/infrablockchain/tutorials/exchange_rate_submit_event.png)

릴레이 체인에서 해당 데이터를 받으면 각 화페에 해당 하는 환율 데이터를 업데이트하게 되며 다음과 같은 이벤트가 발생합니다.

![exchange_rate_update_event](/media/images/docs/infrablockchain/tutorials/exchange_rate_update_event.png)

**시스템 토큰 등록**

**거버넌스 제출**: 토큰의 오너는 해당 토큰을 시스템 토큰으로 등록하기 위해 릴레이체인 거버넌스를 제출합니다.

![register_system_token](/media/images/docs/infrablockchain/tutorials/register_system_token.png)

_Currency Type 는 릴레이 체인의 토큰을 시스템 토큰으로 등록할 때 넣어주는 옵션입니다_

_Extended Metadata 는 추가적으로 시스템 토큰에 대한 메타데이터를 추가하고 싶을 때 넣어주는 옵션입니다_

**거버넌스 투표**: 릴레이 체인의 밸리데이터들은 거버넌스에 참여하며, 2/3 이상이 동의할 경우 해당 시스템 토큰이 등록됩니다.

승인이 되어 시스템 토큰으로 등록이 되면 해당 토큰을 발행한 체인에서는 다음과 같은 이벤트가 발생하며 시스템 토큰으로서 등록이 완료됩니다.

![register_event](/media/images/docs/infrablockchain/tutorials/register_event.png)

**부트스트랩 종료**

정상적으로 시스템 토큰 등록이 완료되면 부트스트랩 단계를 끝내고 시스템 토큰으로 트랜잭션 수수료를 낼 수 있는 단계입니다. 릴레이 체인에서는 해당 체인에 대한 부트 스트랩단계를 종료할 트랜잭션을 거버넌스에 의해 수행합니다.

![end_bootstrap](/media/images/docs/infrablockchain/tutorials/end_bootstrap.png)

_dest 는 부트 스트랩을 종료시킬 체인을 의미합니다._

정상적으로 트랜잭션이 실행되면 _dest_ 체인에서 다음과 같은 이벤트가 발생합니다:

![end_bootstrap_event](/media/images/docs/infrablockchain/tutorials/end_bootstrap_event.png)

## Wrapped 시스템 토큰 사용

**Wrapped 시스템 토큰 사용 승인**: 다른 체인에서 시스템 토큰을 가스비로 사용하고자 할 때 Wrapped 시스템 토큰 사용에 대한 안건을 거버넌스에 제출해야 합니다.

![register_wrapped](/media/images/docs/infrablockchain/tutorials/register_wrapped.png)

_밑에 para_id 옵션은 wrapped 시스템 토큰을 사용할 체인 id 를 의미합니다_

정상적으로 승인되면 wrapped 를 사용할 체인의 `ForeignAssets` 에 정상적으로 등록되어 있는 것을 확인할 수 있습니다.

_등록이 완료된 후더라도 해당 토큰이 발행된 체인에서 XCM 을 통해 전달받은 후 트랜잭션 수수료로 사용될 수 있습니다._

## 시스템 토큰 사용 중지

- **문제 발생 시 조치**: 파라체인이나 시스템 토큰 등에 문제가 발생하면, 생태계에 해를 끼치지 않도록 시스템 토큰 혹은 랩드 시스템 토큰을 일시적으로 사용 중지시키거나 영구 삭제할 수 있습니다.

  ![suspend](/media/images/docs/infrablockchain/tutorials/suspend.png)

  ![derigster](/media/images/docs/infrablockchain/tutorials/deregister.png)

해당 시스템 토큰의 사용 중지는 거버넌스의 승인을 받아야 하며, 승인된 후에는 해당 시스템 토큰 및 랩드 시스템 토큰은 가스비로 사용될 수 없습니다.
