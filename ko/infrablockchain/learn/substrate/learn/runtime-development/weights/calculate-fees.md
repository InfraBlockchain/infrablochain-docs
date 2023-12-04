---
title: 수수료 계산하기
description:
keywords:
  - 가중치
  - 수수료
  - 실행 시간
  - FRAME v1
---

이 안내서는 `pallet_transaction_payment`의 런타임 구현에 대한 `WeightToFee`를 사용자 정의하는 과정을 안내합니다.
수수료는 세 가지 구성 요소로 나뉩니다:

- **바이트 수수료** - 트랜잭션의 길이에 비례하는 수수료입니다.
  비례 상수는 Transaction Payment Pallet의 매개변수입니다.
- **가중치 수수료** - [트랜잭션 가중치](../../frame/tx-weights-fees.md)에서 계산된 수수료입니다.
  변환은 선형일 필요는 없지만, 일반적으로 선형입니다.
  동일한 변환 함수가 런타임의 모든 팔렛의 모든 트랜잭션에 적용됩니다.
- **수수료 곱셈기** - 계산된 수수료에 대한 곱셈기로, 체인이 진행됨에 따라 변경될 수 있습니다.

FRAME은 트랜잭션 실행을 위한 수수료를 계산하고 수집하기 위한 [Transaction Payment Pallet](https://paritytech.github.io/substrate/master/pallet_transaction_payment/index.html)을 제공합니다.
수수료를 더 정확하게 부과하기 위해 수수료 계산 방식을 수정하는 것이 유용할 수 있습니다.

## 목표

런타임에서 수수료 계산 방식을 수정하기 위해 `WeightToFee`를 사용자 정의합니다.

## 사용 사례

[`IdentityFee`](https://paritytech.github.io/substrate/master/frame_support/weights/struct.IdentityFee.html) 대신에 수수료를 계산하는 방식을 수정합니다. 이는 수수료의 한 단위를 한 단위의 가중치에 매핑합니다.

## 절차

### 1. `LinearWeightToFee` 구조체 작성

`runtime/src/lib.rs`에서 `WeightToFeePolynomial`을 구현하는 `LinearWeightToFee`라는 구조체를 작성합니다.
`WeightToFeeCoefficient` 정수의 smallvec을 반환해야 합니다.

`runtime/src/lib.rs`

```rust
pub struct LinearWeightToFee<C>(sp_std::marker::PhantomData<C>);

impl<C> WeightToFeePolynomial for LinearWeightToFee<C>
where
	C: Get<Balance>,
{
	type Balance = Balance;

	fn polynomial() -> WeightToFeeCoefficients<Self::Balance> {
		let coefficient = WeightToFeeCoefficient {
			coeff_integer: C::get(),
			coeff_frac: Perbill::zero(),
			negative: false,
			degree: 1,
		};

		smallvec!(coefficient)
	}
}
```

### 2. 런타임에서 `pallet_transaction_payment` 구성

디스패치 가중치를 `LinearWeightToFee`로 변환하여 청구 가능한 수수료로 변환합니다 (`IdentityFee<Balance>;` 대신):

`runtime/src/lib.rs`

```rust
parameter_types! {
    // LinearWeightToFee 변환에 사용됩니다.
	pub const FeeWeightRatio: u128 = 1_000;
	// 바이트 수수료를 설정합니다. 모든 구성에서 사용됩니다.
	pub const TransactionByteFee: u128 = 1;
}

impl pallet_transaction_payment::Config for Runtime {
	type OnChargeTransaction = CurrencyAdapter<Balances, ()>;
	type TransactionByteFee = TransactionByteFee;

	// 디스패치 가중치를 청구 가능한 수수료로 변환합니다.
	type WeightToFee = LinearWeightToFee<FeeWeightRatio>;

	type FeeMultiplierUpdate = ();
}
```

## 관련 자료

- [가중치](../../basic/glossary.md#가증치)
- [벤치마크 추가](./add-benchmarks.md)
- [사용자 정의 가중치 사용](./use-custom-weights.md)
- [트랜잭션 가중치와 수수료](../../frame/tx-weights-fees.md)
- [`WeightToFeeCoefficients`](https://paritytech.github.io/substrate/master/frame_support/weights/type.WeightToFeeCoefficients.html)
- [`WeightToFeePolynomial`](https://paritytech.github.io/substrate/master/frame_support/weights/trait.WeightToFeePolynomial.html)