---
title: Transaction Fee Model
description: Covers the overall content regarding InfraBlockchain's unique transaction fee model.
keywords:
  - Transaction Fees
  - System Token
  - System Token Weight
---

## Before Starting

Learn about the basic transaction fee model applied by **_InfraBlockchain_** through [Learning About Transaction Fees](../substrate/learn/frame/tx-weights-fees.md).

## Transaction Fee Model

**_InfraBlockchain_** employs the following transaction fee models:

- Dynamic transaction fee model

- Transaction fee model based on the transaction fee table

## Dynamic Transaction Fee Model

**Dynamic transaction fees** are charged based on the actual amount of blockchain resources used (e.g., CPU, network bandwidth, storage space).

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

- When determining transaction fees, the **transaction fee table** is first referenced and, if a value is present, it overrides the **dynamic transaction fee**.

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

InfraBlockchain is a blockchain that imposes transaction fees based on [System Token](./system-token.md) linked to fiat currency, without its own cryptocurrency. It follows the fee imposition method of [Substrate](https://substrate.io) based blockchains but also applies its unique transaction fee model.

The [BalanceToAssetBalance](https://paritytech.github.io/polkadot-sdk/master/pallet_assets/struct.BalanceToAssetBalance.html) of the [Asset pallet](https://paritytech.github.io/substrate/master/pallet_assets/index.html) determines how much system token is required for fees. The transaction fee in System Token is proportionate to the Parachain Fee Rate (PARA_FEE_RATE) and the System Token Weight (SYSTEM_TOKEN_WEIGHT).

    ```
    * TRANSACTION_FEE = FEE_ADJUSTMENT * ACUTAL_FEE * PARA_FEE_RATE / (SYTEM_TOKEN_WEIGHT * DEFAULT_PARA_FEE_RATE)

`FEE_ADJUSTMENT`: A value used to adjust the benchmarking standard applied to existing Substrate-based chains.

`ACTUAL_FEE`: The fee imposed when actual computing resources are used compared to the benchmarking standard.

`PARA_FEE_RATE`: The fee rate imposed for each parachain.

`SYSTEM_TOKEN_WEIGHT`: The weight of the system token held for each system token.

`DEFAULT_PARA_FEE_RATE`: A value used to adjust the fee rate for each parachain.

### Parachain Fee Rate

**Parachain Fee Rate** is a value multiplied uniformly within the **_InfraBlockchain_** multi-chain architecture to impose different transaction fees for each parachain.

- **InfraRelayChain** governance can change the **Parachain Fee Rate**.

- In general, the **Parachain Fee Rate** value is set to 1,000,000 (10^6) to account for decimal places, to adjust the value to 1, and to divide by `DEFAULT_PARA_FEE_RATE` to make it 1.

Depending on the Parachain Fee Rate, different transaction fees can be measured for each parachain. This allows certain parachains to pay more or less in fees for the same transaction.

### System Token Weight

System Token Weight is a value that reflects the difference in exchange rates between fiat currencies. It is used to calculate transaction fees and transaction votes (Transaction-as-a-Vote). In general, the value is designed to ensure that the same transaction fee is paid regardless of which system token is used, considering the final converted value.

- For example, if the value charged as a transaction fee is 1_000, the number of tokens to be paid when paying in USD or KRW will be different (e.g., USD will pay 100 tokens, while KRW will pay 1300 tokens), but the same 1_000 value will be paid.

- System Token Weight is calculated considering the decimal places and exchange rate information of the base system token.

```
SYSTEM_TOKEN_WEIGHT = BASE_WEIGHT \* DECIMAL_RELATIVE_TO_BASE / EXCHANGE_RATE_RELATIVE_TO_BASE
```

`BASE_WEIGHT`: Weight of the base system token.

`DECIMAL_RELATIVE_TO_BASE`: Decimal places relative to the base system token.

`EXCHANGE_RATE_RELATIVE_TO_BASE`: Exchange rate relative to the base system token.

For example, let's consider the current scenario where the base system token is `USD`.

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

In this situation, let's assume that a `transfer` transaction is made. _Here, we do not consider `PARA_FEE_RATE`._

```text
* SYSTEM_TOKEN_FEE = ACTUAL_FEE / SYSTEM_TOKEN_WEIGHT
* ACTUAL_FEE = 100_000_000(10^8)

• When paying in USD
> 10^8 / 10^6 = 100 USD tokens = 0.01 dollar(decimal: 4) = 13원

• When paying in KRW
> 10^8 / 769 * 10^3 = 130 KRW tokens(decimal: 1) = 13원

Therefore, for the same transaction, both USD and KRW will pay a fee of 13 won.
```

## Delegatable Transaction Fee Payment

From the perspective of a blockchain service provider, it is not ideal to require users of the service to pay transaction fees. Therefore, **_InfraBlockchain_** allows for the existence of **fee payers** who can optionally pay transaction fees on behalf of users.

- The existing Substrate-based transaction structure has been modified to include the signature of the **fee payer**.

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

## Next Steps

- [System-Token-Tx-Payment](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/master/substrate/frame/transaction-payment/system-token-tx-payment/src/lib.rs)
