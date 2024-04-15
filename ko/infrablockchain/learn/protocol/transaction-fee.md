---
title: 트랜잭션 수수료 모델
description: 인프라블록체인만의 트랜잭션 수수료 모델에 대한 전반적인 내용을 다룹니다.
keywords:
  - 트랜잭션 수수료
  - 시스템 토큰
  - 시스템 토큰 가중치
---

## 시작하기 전에

- [트랜잭션 수수료에 대해 배우기](../substrate/learn/frame/tx-weights-fees.md)를 통해 **_인프라블록체인(InfraBlockchain)_** 이 기본적으로 적용하고 있는 트랜잭션 수수료 모델에 대해 배웁니다.

## 트랜잭션 수수료 모델

**_인프라블록체인(InfraBlockchain)_** 에는 다음과 같은 트랜잭션 수수료 모델이 있습니다:

- 동적인 트랜잭션 수수료 모델

- 트랜잭션 수수료 테이블에 기반한 트랜잭션 수수료 모델

## 동적인 트랜잭션 수수료 모델

**동적인 트랜잭션 수수료 모델** 은 실제 블록체인의 자원(e.g cpu, 네트워크 대역폭, 저장 공간)을 사용한 양만큼 부과되는 트랜잭션 수수료 부과 모델입니다.

### 트랜잭션 수수료 산정 공식

트랜잭션 실행에 대한 최종 수수료는 다음과 같이 계산됩니다:

    ```
    * INCLUSION_FEE = BASE_FEE + LENGTH_FEE + [TARGETED_FEE_ADJUSTMENT * WEIGHT_FEE];

    * FINAL_FEE = INCLUSION_FEE + TIP;
    ```

## 트랜잭션 수수료 테이블에 기반한 트랜잭션 수수료 모델

블록 생성자(밸리데이터)는 오퍼레이션당 기본 고정된 트랜잭션 수수료를 설정할 수 있습니다.

> 트랜잭션 수수료 테이블 예시

- 파라체인-1000 번의 `balances` 팔렛의 `transfer` 트랜잭션 수수료를 '1_000' 으로 고정
- 파라체인-2000 번의 `did` 팔렛의 `create_did` 트랜잭션 수수료를 '0' 으로 고정

| 파라체인 식별자 | 트랜잭션 이름 | 팔렛 이름 | 수수료 |
| --------------- | ------------- | --------- | ------ |
| 1000            | transfer      | balances  | 1_000  |
| 2000            | create_did    | did       | 0      |

멀티체인 아키텍처인 **_인프라블록체인(InfraBlockchain)_** 에서는 **_인프라릴레이체인(InfraRelayChain)_** 밸리데이터들의 거버넌스에 의해 각 파라체인별로 부과되는 트랜잭션 수수료를 트랜잭션 수수료 테이블로 관리할 수 있습니다.

- 트랜잭션에는 `CallMetadata` 라는 메타데이터가 포함되어 있습니다.

  ```rust
  pub struct CallMetadata {
      pub function_name: &str,
      pub pallet_name: &str
  }
  ```

- 트랜잭션 수수료를 책정할 때 **트랜잭션 수수료 테이블** 을 먼저 참조하고, 값이 있을 시 **동적인 트랜잭션 수수료** 를 덮어씁니다.

  ```rust
  let actual_fee: BalanceOf<T> =
      // 트랜잭션 수수료 테이블
      if let Some(fee) = T::FeeTableProvider::get_fee_from_fee_table(metadata) {
          fee.into()
      } else {
          // 동적인 트랜잭션 수수료
          pallet_transaction_payment::Pallet::<T>::compute_actual_fee(
              len as u32, info, post_info, tip,
          )
      };
  ```

## 시스템 토큰을 이용한 트랜잭션 수수료 부과

인프라블록체인은 자체적으로 발행되는 가상화폐 없이 법정화폐와 연결된 [시스템 토큰](./system-token.md) 을 기반으로 트랜잭션 수수료를 부과하는 블록체인입니다. 이에 따라 기본적으로는 [Substrate](https://substrate.io) 기반의 블록체인에서 수수료를 부과하는 방식을 따르지만, 인프라블록체인만의 프로토콜이 적용된 트랜잭션 수수료 모델에 기반하여 수수료가 부과됩니다.

[Asset 팔렛](https://paritytech.github.io/substrate/master/pallet_assets/index.html) 의 [BalanceToAssetBalance](https://paritytech.github.io/polkadot-sdk/master/pallet_assets/struct.BalanceToAssetBalance.html)를 통해 시스템 토큰으로 수수료를 얼마나 내야 하는지가 결정됩니다. 시스템 토큰으로 내야 하는 트랜잭션 수수료는 파라체인별 수수료 비율(PARA_FEE_RATE)과 시스템 토큰 가중치(SYSTEM_TOKEN_WEIGHT)에 비례해서 부과됩니다.

    ```
    * TRANSACTION_FEE = FEE_ADJUSTMENT * ACUTAL_FEE * PARA_FEE_RATE / (SYTEM_TOKEN_WEIGHT * DEFAULT_PARA_FEE_RATE)
    ```

`FEE_ADJUSTMENT`: 기존 Substrate 기반 체인에 적용되어있는 벤치마킹 기준을 조정하기 위한 값

`ACTUAL_FEE`: 벤치마킹 대비 실제 컴퓨팅 리소스가 사용되었을 때 부과되는 수수료

`PARA_FEE_RATE`: 각 파라체인별로 부과되는 수수료 비율

`SYSTEM_TOKEN_WEIGHT`: 시스템 토큰별로 보유하고 있는 가중치

`DEFAULT_PARA_FEE_RATE`: 파라체인별 수수료 비율을 조정해주기 위한 값

### 파라체인별 수수료 비율

**파라체인별 수수료 비율** 은 멀티체인 아키텍처인 **_인프라블록체인(InfraBlockchain)_** 안에서 파라체인별로 트랜잭션 수수료를 다르게 부과하기 위해 일률적으로 곱해지는 값입니다.

- **_인프라릴레이체인(InfraRelayChain)_** 거버넌스에 의해 **파라체인별 수수료 비율** 을 변경할 수 있습니다.
- 일반적인 경우 **파라체인별 수수료 비율** 값은 1이며, 소수점을 고려하기 위해 1_000_000(10^6) 으로 설정됩니다. 1의 값을 만들어주기 위해 위 식에서처럼 `DEFAULT_PARA_FEE_RATE(10^6)`로 나눠주어 보정을 해줍니다.
- **파라체인별 수수료 비율** 에 따라 파라체인별로 트랜잭션 수수료가 다르게 측정될 수 있습니다. 이로 인해 동일한 트랜잭션이라도 특정 파라체인은 다른 파라체인보다 더 많은,혹은 더 적은 수수료를 낼 수 있습니다.

### 시스템 토큰 가중치

**시스템 토큰 가중치** 는 법정 화폐간 환차이를 반영한 값입니다. 트랜잭션 수수료 및 트랜잭션 투표량(Transaction-as-a-Vote)을 계산할 때 사용됩니다. 일반적으로 동일한 트랜잭션에 대한 수수료는 어떤 시스템 토큰을 사용하든 최종 환산된 가치가 동일해야 하기 때문에 **시스템 토큰 가중치** 는 이를 고려하기 위해 설계되었습니다.

_예를 들면, 트랜잭션 수수료로 책정된 가치가 `1_000` 이라면, 해당 금액을 `USD` 로 지불하거나 `KRW` 로 지불하는 경우 내야하는 토큰 개수는 다르겠지만(e.g `USD` 가 1개의 토큰을 낸다면 `KRW`는 1300 개를 지불) 동일한 `1_000` 이라는 가치의 수수료를 낼 것입니다._

- **시스템 토큰 가중치** 는 기준이 되는 시스템 토큰(BASE_SYTEM_TOKEN) 과의 소수점, 환율 정보를 고려하여 계산됩니다.

  ```
  SYSTEM_TOKEN_WEIGHT = BASE_WEIGHT * DECIMAL_RELATIVE_TO_BASE / EXCHANGE_RATE_RELATIVE_TO_BASE
  ```

  `BASE_WEIGHT`: 기준이 되는 시스템 토큰의 가중치

  `DECIMAL_RELATIVE_TO_BASE`: 기준이 되는 시스템 토큰 대비 소수점 자릿수

  `EXCHANGE_RATE_RELATIVE_TO_BASE`: 기준이 되는 시스템 토큰 대비 환율

- 예를 들어보겠습니다. 현재 시나리오에서 기준이 되는 시스템 토큰 은 `USD` 입니다.

  ```toml
  iUSD = {
      "decimals": 4,
      "system_token_weight": 1_000_000,
      "exchange_rate": 1_000_000,
  }

  iKRW = {
      "decimals": 1,
      "system_token_weight": x,
      "exchange_rate": 1_300_000_000
  }

  x = 1_000_000 * 10^(4-1) / (1_300_000_000 / 1_000_000) = 769_000
  ```

  이 상황에서 `transfer` 트랜잭션에 대한 수수료를 낸다고 가정해봅시다. _여기서는 `PARA_FEE_RATE` 를 고려하지 않습니다._

  ```text
  * SYSTEM_TOKEN_FEE = ACTUAL_FEE / SYSTEM_TOKEN_WEIGHT
  * ACTUAL_FEE = 100_000_000(10^8)

  • USD 로 내는 경우
  > 10^8 / 10^6 = 100 USD tokens = 0.01 dollar(decimal: 4) = 13원

  • KRW 로 내는 경우
  > 10^8 / 769 * 10^3 = 130 KRW tokens(decimal: 1) = 13원

  따라서 동일한 트랜잭션에 대해 USD, KRW 모두 한화로 13원 가치의 수수료를 내게 됩니다.
  ```

## 위임 가능한 트랜잭션 수수료 지불

어떤 블록체인 서비스 제공자 입장에서 해당 서비스를 이용하는 사용자들에게 트랜잭션 수수료를 지불하게 하는 것은 사용자 경험 측면에서 좋지 않은 면이 있습니다. 이에 **_인프라블록체인(InfraBlockchain)_** 은 트랜잭션 수수료를 대신 지불해주는 **수수료 대납자(fee payer)** 가 선택적으로 존재할 수 있습니다.

- 기존 Substrate 기반 트랜잭션 구조에 **수수료 대납자** 의 서명을 추가하여 작동하고 있습니다.

  ```rust
  pub struct UncheckedExtrinsic<Address, Call, Signature, Extra>
  where
      Extra: SignedExtension,
  {
      pub signature:
          Option<(Address, Signature, Extra, Option<Vec<UncheckedTxExtension<Address, Signature>>>)>,
      pub function: Call,
  }

  /// This can be extensible
  pub enum UncheckedTxExtension<Address, Signature> {
      FeePayer(Option<(Address, Signature)>),
  }
  ```

## 다음으로 넘어가기

- [System-Token-Tx-Payment](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/master/substrate/frame/transaction-payment/system-token-tx-payment/src/lib.rs)
