---
title: Transaction Fee Model
description: Discusses the unique transaction fee model of the InfraBlockchain.
keywords:
  - Transaction Fees
  - System Token
  - System Token Weight
---

## Before Starting

- Learn about the basic transaction fee model applied by **_InfraBlockchain_** through [Learning About Transaction Fees](../substrate/learn/frame/tx-weights-fees.md).

## Transaction Fee Model

**_InfraBlockchain_** employs the following transaction fee models:

- Dynamic transaction fee model
- Transaction fee model based on the transaction fee table

## Dynamic Transaction Fee Model

_Dynamic transaction fees_ are charged based on the actual amount of blockchain resources used (e.g., CPU, network bandwidth, storage space).

### Transaction Fee Calculation Formula

The final fee for a transaction is calculated as follows:

    ```
    * INCLUSION_FEE = BASE_FEE + LENGTH_FEE + [TARGETED_FEE_ADJUSTMENT * WEIGHT_FEE];
    * FINAL_FEE = INCLUSION_FEE + TIP;
    ```

## Transaction Fee Model Based on Fee Table

Block producers (validators) can set a fixed transaction fee per operation.

> Example of Transaction Fee Table

- Fix the `transfer` transaction fee of the `balances` pallet in parachain-1000 at '1_000'
- Fix the `create_did` transaction fee of the `did` pallet in parachain-2000 at '0'

| Parachain Identifier | Transaction Name | Pallet Name | Fee   |
| -------------------- | ---------------- | ----------- | ----- |
| 1000                 | transfer         | balances    | 1_000 |
| 2000                 | create_did       | did         | 0     |

In the multi-chain architecture of **_InfraBlockchain_**, transaction fees imposed for each parachain can be managed through the transaction fee table by the governance of **_InfraRelayChain_** validators.

- Transactions include metadata known as `CallMetadata`.

  ```rust
  pub struct CallMetadata {
      pub function_name: &str,
      pub pallet_name: &str
  }
  ```

- When determining transaction fees, the _transaction fee table_ is first referenced and, if a value is present, it overrides the _dynamic transaction fee_.

  ```rust
  let actual_fee: BalanceOf<T> =
      // Transaction fee table
      if let Some(fee) = T::FeeTableProvider::get_fee_from_fee_table(metadata) {
          fee.into()
      } else {
          // Dynamic transaction fee
          pallet_transaction_payment::Pallet::<T>::compute_actual_fee(
              len as u32, info, post_info, tip,
          )
      };
  ```

## Imposing Transaction Fees Using System Token

InfraBlockchain is a blockchain that imposes transaction fees based on _[System Token](./system-token.md)_ linked to fiat currency, without its own cryptocurrency. It follows the fee imposition method of _[Substrate](https://substrate.io)_ based blockchains but also applies its unique transaction fee model.

The BalanceToAssetBalance of the Asset pallet determines how much system token is required for fees. The transaction fee in System Token is proportionate to the _Parachain Fee Rate (PARA_FEE_RATE)_ and the _System Token Weight (SYSTEM_TOKEN_WEIGHT)_.

    ```
    * SYSTEM_TOKEN_FEE = ACTUAL_FEE * PARA_FEE_RATE / (SYTEM_TOKEN_WEIGHT * DEFAULT_PARA_FEE_RATE)
    ```

    `ACTUAL_FEE`: Fee resulting from actual computing resources used compared to benchmark
    `PARA_FEE_RATE`: Fee rate imposed by each parachain
    `SYSTEM_TOKEN_WEIGHT`: Weight held by each system token
    `DEFAULT_PARA_FEE_RATE`: Value for adjusting the parachain fee rate

### Parachain Fee Rate

_Parachain Fee Rate_ is a value uniformly multiplied to differently impose transaction fees for each parachain in the multi-chain **_InfraBlockchain_**.

- _Parachain Fee Rate_ can be changed by the governance of **_InfraRelayChain_**.
- Normally, the _Parachain Fee Rate_ value is 1, and considering decimals, it is set to 1,000,000 (10^6). To produce a value of 1, it is divided by `DEFAULT_PARA_FEE_RATE (10^6)` as in the formula.
- Depending on the _Parachain Fee Rate_, transaction fees can be measured differently for each parachain. Thus, the same transaction could incur more/less fees on certain parachains compared to others.

### System Token Weight

Generally, the fee for the same transaction should have the same converted value regardless of System Token used, and the _System Token Weight_ is designed to consider this. _For example, if a transaction fee is valued at `1,000`, paying in `USD` or `KRW` would differ in amount (e.g., 1 USD token might equate to 1.3 KRW tokens) but will equate to the same value of `1,000`._

- _System Token Weight_ is calculated considering the decimal places and exchange rate information relative to the _Base System Token (BASE_SYSTEM_TOKEN)_.

  ```
  SYSTEM_TOKEN_WEIGHT = BASE_WEIGHT * DECIMAL_RELATIVE_TO_BASE / EXCHANGE_RATE_RELATIVE_TO_BASE
  ```

  `BASE_WEIGHT`: Weight of the base system token
  `DECIMAL_RELATIVE_TO_BASE`: Decimal places relative to the base system token
  `EXCHANGE_RATE_RELATIVE_TO_BASE`: Exchange rate relative to the base system token

- Let's look at an example. In the current scenario, the _Base System Token_ is `USD`.

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

  Assuming a `transfer` transaction fee. (ignore the _PARA_FEE_RATE_ here.)

  ```text
  * SYSTEM_TOKEN_FEE = ACTUAL_FEE / SYSTEM_TOKEN_WEIGHT
  * ACTUAL_FEE = 100_000_000 (10^8)

  • Paying in USD
  > 10^8 / 10^6 = 100 USD tokens = 0.01 dollar (decimal: 4) = 13 KRW

  • Paying in KRW
  > 10^8 / 769 * 10^3 = 130 KRW tokens (decimal: 1) = 13 KRW

  Thus, for the same transaction, both USD and KRW pay a fee value of 13 KRW.
  ```

## Delegatable Transaction Fee Payment

From the perspective of a blockchain service provider, having users pay transaction fees can negatively impact the user experience. Therefore, **_InfraBlockchain_** allows for an optional _fee payer_ who can pay transaction fees on behalf of users.

- It operates by adding the signature of the _fee payer_ to the existing Substrate-based transaction structure.

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

## Moving Forward

- [System-Token-Tx-Payment](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/master/substrate/frame/transaction-payment/system-token-tx-payment/src/lib.rs)
