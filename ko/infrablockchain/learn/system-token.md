---
title: 시스템 토큰
description: 이 튜토리얼은 인프라 블록체인의 법정 화폐 기반의 토큰인 시스템 토큰에 대해 배웁니다. 
keywords:
  - 시스템 토큰
---

# System Token

System token은 Infrablockspace만의 고유한 feature 중 하나입니다.

System token의 특징
- System token은 법정화폐에 연동된(Fiat-pegged) token입니다. 실제 화폐가 담보된 양 만큼만 체인 상에서 발행될 수 있습니다.
- 따라서 System token은 별도의 tokenomics를 가지고 있지 않습니다.
- 사용자는 System token을 활용해 변동없는 가스 비용을 지불할 수 있으며 PoT(page 링크 ** )의 Vote로 사용할 수 있습니다.

## Lifecycle of a System Token

Infrablockspace는 System token을 가스비로 사용하는 차별화된 블록체인 시스템을 제공합니다. 
아래는 시스템 토큰의 등록, 사용 및 폐기과정에 대해 개발자들이 쉽게 이해하고 개발할 수 있도록 정리한 내용입니다.
해당 내용에 대한 실습은 [how-to-pay-transaction-fee](../tutorials/how-to-pay-transaction-fee.md)에서 진행할 수 있습니다.

1. Bootstrap 과정
- Test Token 활용: Bootstrap 중에는 가스비로 사용 가능한 "test token"이 존재합니다.
- 첫 System token 승인: 첫 시스템 토큰이 승인 및 유통된 후 bootstrap이 완료됩니다. [System token 승인 과정 링크]
- Test Token 소각: Bootstrap 완료 후 test token은 모두 소각되며, 더 이상 가스비로 사용할 수 없습니다(sufficient = false).

2. 시스템 토큰 등록
- 토큰 생성 및 발행: 특정 체인에서 토큰을 생성 및 발행합니다.
- Governance 제출: 토큰의 오너는 해당 토큰을 System token으로 등록하기 위해 relay chain 거버넌스를 제출합니다.
- Systen Token 투표: Relay chain의 validator들은 거버넌스에 참여하며, 2/3 이상이 동의할 경우 해당 시스템 토큰이 등록됩니다.
- Gas fee 사용: 해당 토큰이 System token으로 등록되면, 토큰을 발행한 chain에서 System token을 gas fee로 사용할 수 있게 됩니다.

3. 다른 parachain에서 시스템 토큰 사용
- Wrapped System token 사용 승인: 다른 체인에서 시스템 토큰을 가스비로 사용하고자 할 때 해당 토큰의 "wrapped system token" 사용에 대한 안건을 거버넌스에 제출해야 합니다.
- 거버넌스 승인: 해당 안건이 거버넌스에 승인되면, 해당 체인에서는 wrapped system token을 가스비로 사용할 수 있게 됩니다.

4. 시스템 토큰 Suspend 및 Deregister
- 문제 발생 시 조치: Parachain이나 system token 등에 문제가 발생하면, 생태계에 해를 끼치지 않도록 system token 혹은 wrapped system token을 suspend(일시 사용 금지) 또는 deregister(영구 삭제)할 수 있습니다.
- 거버넌스 승인 필요: 해당 시스템 토큰의 사용 중지는 거버넌스의 승인을 받아야 하며, 승인된 후에는 해당 system token 및 wrapped system token은 가스비로 사용될 수 없습니다.


## System Token을 활용한 가스비 계산

### Gas Fee Calculation Logic

extrinsic 실행에 대한 final fee는 아래와 같이 계산됩니다. 
```
inclusion_fee = base_fee + length_fee + [targeted_fee_adjustment * weight_fee];
final_fee = inclusion_fee + tip;
```

pallet-assets의 BalanceToAssetBalance을 통해 System token fee를 얼마나 내야 하는지 결정됩니다. 
BalanceToAssetBalance에서 추가된 logic은 다음과 같습니다.
```
system_token_fee = final_fee * para_fee_rate / (system_token_weight * default_para_fee_rate)
```

### Para Fee Rate
para_fee_rate는 infrablockspace 생태계에서 특정 parachain에 더 많은/더 적은 system token fee를 부과하기 위해 일률적으로 곱해지는 값입니다. 
동일한 extrinsic이더라도 특정 parachain은 다른 parachain보다 더 많은/더 적은 fee를 낼 수 있습니다. 
relay chain governance에 의해 para_fee_rate을 변경할 수 있습니다. 
일반적인 경우 para_fee_rate 값은 1이며 소수점을 고려하기 위해 1_000_000(10^6)으로 설정됩니다. 1의 값을 만들어주기 위해 위 식에서처럼 default_para_fee_rate(10^6)로 나눠주어 보정을 해줍니다. 
para_fee_rate가 증가할 경우 더 많은 system token을 지불해야 하고, 감소할 경우 더 적은 system token fee를 지불합니다. 

### System Token Weight란?

일반적으로 동일한 transaction에 대한 gas fee는 어떤 system token을 사용하든 최종 환산된 가치는 동일해야 합니다. 
System token weight는 이를 고려하기 위해 설계되었습니다. System token weight는 기준이 되는 Base System Token(현재 iUSD)과의 Decimal, 환율 정보를 고려하여 계산됩니다. 
iUSD의 System token weight는 1_000_000(10^9)입니다.

- `system token weight`은 기준이 되는 시스템토큰(iUSD)의 `base system token weight`, 달러 대비 환율, 달러 대비 decimal이 고려됩니다.
- `system token weight = (base system token weight) * (달러 대비 decimal) / 달러 대비 환율`
- `달러 대비 decimal = 10^(달러의 decimal - 해당 시스템 토큰의 decimal)`
- `달러 대비 환율 =  (달러 가치 / 해당 시스템 토큰의 가치)`

위 값으로부터, system token transfer(asset transfer)를 위한 fee는 iUSD로 약 0.065달러로 계산됩니다. 

### Systen Token Weight 계산 방법

- infrablocksapce에서 기준이 되는 iUSD의 `system token weight`를 1_000_000(10^6)로 책정합니다. iUSD의 decimal은 4, 달러대비 환율은 당연히 1입니다.
- 예를 들어, iKRW의 system token을 계산해보겠습니다 iKRW의 decimal = 1, 달러대비 환율 = 1300(1 iUSD = 1300 iKRW)일 경우, `system token weight`은 다음과 같습니다.
- `iKRW system token weight = (10^6) * (10^3) / 1300 ~= 769_000`
- 위 식에서와 같이 iKRW의 system token weight는 769_000 입니다.

### System Token을 통한 fee 계산

- 일반적인 경우, 동일한 transaction에 대한 gas fee는 어떤 system token을 사용하든 환산된 가치는 동일합니다.
- 트랜잭션에 대한 가스비의 system token amount는 다음과 같이 계산됩니다. `system token fee amount = final_fee / system token weight`
- 예를 들어, `transfer_call`에 대한 final_fee가 100_000_000(10^8)이고, 두개의 system token iUSD, iKRW이 있다고 가정해봅시다. 각 `system token weight`는 1_000_000, 769_000입니다.
- 따라서, `transfer_call`에 대해 iUSD로는 `100_000_000 / 1_000_000 = 100`만큼, iKRW로는 `100_000_000 / 769_000 = 130`만큼 지불해야 합니다.
- 이를 법정화폐 가치로 계산해보면, iUSD는 decimal을 4만큼 가지기 때문에 0.01 USD를 내는 것이고, iKRW는 decimal이 1이기 때문에 13 KRW을 내야합니다. 
- 환율을 고려하면 0.01 usd = 13 krw 이기 때문에 두 토큰 가치가 동일하게 fee로 사용된 것을 확인할 수 있습니다.
