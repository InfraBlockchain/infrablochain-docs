---
title: 오프체인 워커 추가
description: 팔렛을 수정하여 오프체인 워커를 포함시키고, 오프체인 워커에서 온체인 상태를 업데이트하기 위해 트랜잭션을 제출하는 방법을 설명합니다.
keywords:
  - 오프체인 워커
  - OCW
  - 트랜잭션
  - 오프체인 데이터
  - 서명된 트랜잭션
  - 서명되지 않은 트랜잭션
  - 서명된 페이로드
---

이 튜토리얼은 팔렛을 수정하여 오프체인 워커를 포함시키고, 오프체인 워커가 온체인 상태를 업데이트하는 트랜잭션을 제출할 수 있도록 팔렛과 런타임을 구성하는 방법을 설명합니다.

## 오프체인 워커 사용하기

오프체인 워커를 사용하여 장기 실행 계산 또는 오프라인 소스에서 데이터를 가져오는 경우, 해당 작업의 결과를 온체인에 저장하고자 할 것입니다.
그러나 오프체인 스토리지는 온체인 리소스와 별개이며, 오프체인 워커가 처리한 데이터를 직접 온체인 스토리지에 저장할 수는 없습니다.
오프체인 워커에서 처리한 데이터를 온체인 상태의 일부로 저장하려면, 오프체인 워커 스토리지에서 데이터를 온체인 스토리지 시스템으로 전송하는 트랜잭션을 생성해야 합니다.

이 튜토리얼은 오프체인 데이터를 온체인에 저장하기 위해 서명된 또는 서명되지 않은 트랜잭션을 제출할 수 있는 오프체인 워커를 생성하는 방법을 설명합니다.
일반적으로 서명된 트랜잭션은 보안성이 높지만, 호출하는 계정이 트랜잭션 수수료를 처리해야 합니다.
예를 들어:

- 트랜잭션 호출자 계정을 기록하고 호출자 계정에서 트랜잭션 수수료를 차감하려면 **서명된 트랜잭션**을 사용하세요.
- 트랜잭션 호출자를 기록하려고 하지만 호출자가 트랜잭션 수수료 지불에 대한 책임을 지지 않으려면 **서명된 페이로드를 포함한 서명되지 않은 트랜잭션**을 사용하세요.

## 서명되지 않은 트랜잭션 사용하기

서명되지 않은 트랜잭션을 **서명된 페이로드 없이** 제출하는 것도 가능합니다. 이는 트랜잭션 호출자를 기록하지 않으려는 경우에 사용될 수 있습니다.
그러나 서명되지 않은 트랜잭션을 사용하여 체인 상태를 수정하는 것은 상당한 위험이 따릅니다.
서명되지 않은 트랜잭션은 악의적인 사용자가 악용할 수 있는 잠재적인 공격 경로를 제공합니다.
오프체인 워커가 서명되지 않은 트랜잭션을 전송할 수 있도록 허용하는 경우, 트랜잭션이 인가되었는지 확인하는 로직을 포함해야 합니다.
서명되지 않은 트랜잭션이 온체인 상태를 확인하여 검증되는 방법에 대한 예제는 [`enact_authorized_upgrade`](https://github.com/paritytech/polkadot-sdk/blob/master/cumulus/pallets/parachain-system/src/lib.rs) 호출의 `ValidateUnsigned` 구현을 참조하세요.
이 예제에서는 주어진 코드 해시가 이전에 인가되었는지 확인하여 서명되지 않은 트랜잭션을 검증합니다.

또한, 서명된 페이로드를 가진 서명되지 않은 트랜잭션도 악용될 수 있습니다. 오프체인 워커는 트랜잭션의 유효성을 확인하기 위해 엄격한 로직을 구현하지 않는 한 신뢰할 수 있는 소스로 가정할 수 없습니다.
대부분의 경우, 저장소에 쓰기 전에 트랜잭션이 오프체인 워커에 의해 제출되었는지 확인하는 것만으로는 네트워크를 보호하기에 충분하지 않습니다.
오프체인 워커를 보호하기 위해 엄격한 로직을 구현하지 않는 한, 오프체인 워커를 신뢰할 수 있다고 가정해서는 안 됩니다.
대신 오프체인 워커의 프로세스에 대한 액세스와 수행할 수 있는 작업을 제한하는 제한적인 권한을 명시적으로 설정해야 합니다.

서명되지 않은 트랜잭션은 사실상 런타임으로의 **개방된 문**입니다.
트랜잭션이 실행되는 조건을 신중히 고려한 후에만 사용해야 합니다.
보호장치 없이는 악의적인 사용자가 오프체인 워커로 위장하고 런타임 스토리지에 액세스할 수 있습니다.

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [Rust 및 Rust 툴체인](/install/)을 설치하여 Substrate 개발 환경을 구성했습니다.

- [로컬 블록체인 만들기](/tutorials/build-a-blockchain/build-local-blockchain/) 튜토리얼을 완료하고 개발자 허브에서 Substrate 노드 템플릿을 로컬에 설치했습니다.

- FRAME 매크로를 사용하는 방법과 팔렛의 로직을 편집하는 방법에 익숙합니다.

- 런타임에서 팔렛의 구성 트레이트를 수정하는 방법에 익숙합니다.

## 튜토리얼 목표

이 튜토리얼을 완료하면 다음을 수행할 수 있습니다:

- 서명되지 않은 트랜잭션 사용 시 발생할 수 있는 위험을 식별합니다.
- 팔렛에 오프체인 워커 함수를 추가합니다.
- 팔렛과 런타임을 구성하여 오프체인 워커가 서명된 트랜잭션을 제출할 수 있도록 합니다.
- 팔렛과 런타임을 구성하여 오프체인 워커가 서명되지 않은 트랜잭션을 제출할 수 있도록 합니다.
- 팔렛과 런타임을 구성하여 오프체인 워커가 서명된 페이로드를 포함한 서명되지 않은 트랜잭션을 제출할 수 있도록 합니다.

## 서명된 트랜잭션

서명된 트랜잭션을 제출하려면 팔렛과 런타임을 구성하여 오프체인 워커가 사용할 수 있는 최소한 하나의 계정을 활성화해야 합니다.
팔렛을 오프체인 워커를 사용하고 서명된 트랜잭션을 제출할 수 있도록 구성하는 데는 다음 단계가 필요합니다:

- [팔렛에서 오프체인 워커 구성](#팔렛에서-오프체인-워커-구성)
- [런타임에서 팔렛과 필요한 트레이트 구현](#런타임에서-팔렛-구현)
- [트랜잭션 서명을 위한 계정 추가](#트랜잭션-서명을-위한-계정-추가)

### 팔렛에서 오프체인 워커 구성

오프체인 워커가 서명된 트랜잭션을 전송할 수 있도록 하려면:

1. 팔렛의 `src/lib.rs` 파일을 텍스트 편집기로 엽니다.
2. `#[pallet::hooks]` 매크로와 오프체인 워커의 진입점을 코드에 추가합니다.

   예시:

   ```rust
   #[pallet::hooks]
   impl<T: Config> Hooks<BlockNumberFor<T>> for Pallet<T> {
   	/// 오프체인 워커 진입점.
   	///
   	/// `fn offchain_worker`를 구현함으로써 새로운 오프체인 워커를 선언합니다.
   	/// 이 함수는 노드가 완전히 동기화되고 새로운 최상의 블록이 성공적으로 가져올 때 호출됩니다.
   	/// 모든 블록에서 오프체인 워커가 실행되는 것은 보장되지 않으며, 일부 블록이 건너뛰어지거나 워커가 두 번 실행되는(재조직) 경우가 있을 수 있으므로 이를 처리할 수 있는 코드여야 합니다.
   	fn offchain_worker(block_number: T::BlockNumber) {
   		log::info!("팔렛-ocw에서 안녕하세요.");
   		// 오프체인 워커에 의해 호출되는 코드의 진입점
   	}
   	// ...
   }
   ```

3. `offchain_worker` 함수의 로직을 추가합니다.
4. 팔렛의 `Config` 트레이트에 [`CreateSignedTransaction`](https://paritytech.github.io/substrate/master/frame_system/offchain/trait.CreateSignedTransaction.html)을 추가합니다.
   예시:

   ```rust
   /// 이 팔렛의 구성 트레이트
   #[pallet::config]
   pub trait Config: CreateSignedTransaction<Call<Self>> + frame_system::Config {
   	// ...
   }
   ```

5. 팔렛의 `Config` 트레이트에 `AuthorityId` 타입을 추가합니다.

   ```rust
   #[pallet::config]
   pub trait Config: CreateSignedTransaction<Call<Self>> + frame_system::Config {
   	// ...
   type AuthorityId: AppCrypto<Self::Public, Self::Signature>;
   }
   ```

6. 트랜잭션 서명을 위해 팔렛이 사용할 수 있는 계정을 보유한 `crypto` 모듈을 추가합니다.

   ```rust
   use sp_core::{crypto::KeyTypeId};

   // ...

   pub const KEY_TYPE: KeyTypeId = KeyTypeId(*b"demo");

   // ...

   pub mod crypto {
   	use super::KEY_TYPE;
   	use sp_core::sr25519::Signature as Sr25519Signature;
   	use sp_runtime::{
   		app_crypto::{app_crypto, sr25519},
   		traits::Verify, MultiSignature, MultiSigner
   	};
   	app_crypto!(sr25519, KEY_TYPE);

   	pub struct TestAuthId;

   	// 런타임을 위해 구현됨
   	impl frame_system::offchain::AppCrypto<MultiSigner, MultiSignature> for TestAuthId {
   	type RuntimeAppPublic = Public;
   	type GenericSignature = sp_core::sr25519::Signature;
   	type GenericPublic = sp_core::sr25519::Public;
   	}
   }
   ```

   [`app_crypto` 매크로](https://paritytech.github.io/substrate/master/sp_application_crypto/macro.app_crypto.html)는 `KEY_TYPE`로 식별되는 `sr25519` 서명을 가진 계정을 선언합니다.
   이 예시에서 `KEY_TYPE`은 `demo`입니다.
   이 매크로는 새로운 계정을 생성하지 않습니다.
   이 매크로는 팔렛에서 사용할 수 있는 `crypto` 계정이 있는 것을 선언하는 것입니다.

7. 오프체인 워커가 사용할 서명된 트랜잭션을 온체인 스토리지에 전송하기 위해 사용할 계정을 초기화합니다.

   ```rust
   fn offchain_worker(block_number: T::BlockNumber) {
   	let signer = Signer::<T, T::AuthorityId>::all_accounts();

   	// ...
   }
   ```

   이 코드는 팔렛이 소유하는 모든 서명자를 검색할 수 있도록 합니다.

8. `send_signed_transaction()`을 사용하여 서명된 트랜잭션 호출을 생성합니다.

   ```rust
   fn offchain_worker(block_number: T::BlockNumber) {
   	let signer = Signer::<T, T::AuthorityId>::all_accounts();

   	// `send_signed_transaction`의 연관 타입을 사용하여 방금 생성한 호출을 나타내는 트랜잭션을 생성하고 제출합니다.
   	// `send_signed_transaction()`의 반환 타입은 `Option<(Account<T>, Result<(), ()>)>`입니다. 다음과 같습니다:
   	//	 - `None`: 트랜잭션을 전송할 수 있는 계정이 없음
   	//	 - `Some((account, Ok(())))`: 트랜잭션이 성공적으로 전송됨
   	//	 - `Some((account, Err(())))`: 트랜잭션 전송 중 오류 발생
   	let results = signer.send_signed_transaction(|_account| {
   		Call::on_chain_call { key: val }
   	});

   	// ...
   }
   ```

9. 트랜잭션이 온체인에 성공적으로 제출되었는지 확인하고 반환된 `results`를 통해 적절한 오류 처리를 수행합니다.

   ```rust
   fn offchain_worker(block_number: T::BlockNumber) {
   	// ...

   	for (acc, res) in &results {
   		match res {
   			Ok(()) => log::info!("[{:?}]: 트랜잭션 제출 성공.", acc.id),
   			Err(e) => log::error!("[{:?}]: 트랜잭션 제출 실패. 이유: {:?}", acc.id, e),
   		}
   	}

   	Ok(())
   }
   ```

### 런타임에서 팔렛 구현

1. 텍스트 편집기에서 노드 템플릿의 `runtime/src/lib.rs` 파일을 엽니다.

2. 팔렛을 위한 구성에 `AuthorityId`를 추가하고, `crypto` 모듈에서 `TestAuthId`를 사용하도록 런타임 구성을 수정합니다.

   ```rust
   impl pallet_your_ocw_pallet::Config for Runtime {
	   // ...
	   type AuthorityId = pallet_your_ocw_pallet::crypto::TestAuthId;
   }
   ```

3. 런타임에서 `CreateSignedTransaction` 트레이트를 구현합니다.

   팔렛에서 `CreateSignedTransaction` 트레이트를 구현했으므로, 런타임에서도 해당 트레이트를 구현해야 합니다.

   [`CreateSignedTransaction`](https://paritytech.github.io/substrate/master/frame_system/offchain/trait.CreateSignedTransaction.html)을 살펴보면 런타임을 위해 `create_transaction()` 함수만 구현하면 되는 것을 알 수 있습니다.
   예시:

   ```rust
   use codec::Encode;
   use sp_runtime::{generic::Era, SaturatedConversion};
 
   // ...

   impl<LocalCall> frame_system::offchain::CreateSignedTransaction<LocalCall> for Runtime
   where
	    RuntimeCall: From<LocalCall>,
   {
	    fn create_transaction<C: frame_system::offchain::AppCrypto<Self::Public, Self::Signature>>(
   		   call: RuntimeCall,
	       public: <Signature as Verify>::Signer,
		     account: AccountId,
		     nonce: Nonce,
	     ) -> Option<(RuntimeCall, <UncheckedExtrinsic as traits::Extrinsic>::SignaturePayload)> {
		     let tip = 0;
		     // take the biggest period possible.
		     let period =
			      BlockHashCount::get().checked_next_power_of_two().map(|c| c / 2).unwrap_or(2) as u64;
		     let current_block = System::block_number()
			      .saturated_into::<u64>()
			      // The `System::block_number` is initialized with `n+1`,
			      // so the actual block number is `n`.
			      .saturating_sub(1);
		     let era = Era::mortal(period, current_block);
		     let extra = (
			      frame_system::CheckNonZeroSender::<Runtime>::new(),
			      frame_system::CheckSpecVersion::<Runtime>::new(),
			      frame_system::CheckTxVersion::<Runtime>::new(),
			      frame_system::CheckGenesis::<Runtime>::new(),
			      frame_system::CheckEra::<Runtime>::from(era),
			      frame_system::CheckNonce::<Runtime>::from(nonce),
			      frame_system::CheckWeight::<Runtime>::new(),
			      pallet_transaction_payment::ChargeTransactionPayment::<Runtime>::from(tip),
		     );
		     let raw_payload = SignedPayload::new(call, extra)
			      .map_err(|e| {
				       log::warn!("서명된 페이로드를 생성할 수 없습니다: {:?}", e);
			      })
			      .ok()?;
		     let signature = raw_payload.using_encoded(|payload| C::sign(payload, public))?;
		     let address = account;
		     let (call, extra, _) = raw_payload.deconstruct();
		     Some((call, (sp_runtime::MultiAddress::Id(address), signature, extra)))
	   }
   }
   ```

   이 코드 조각은 길지만, 기본적으로 다음 주요 단계를 보여줍니다:

   - `extra`의 `SignedExtra` 유형을 생성하고 준비하고, 다양한 체커를 설정합니다.
   - 전달된 `call`과 `extra`를 기반으로 원시 페이로드를 생성합니다.
   - 계정 공개 키로 원시 페이로드에 서명합니다.
   - 모든 데이터를 번들로 묶어 호출, 호출자, 서명 및 서명 확장 데이터를 포함하는 튜플을 반환합니다.

   이 코드 예제는 [Substrate 코드베이스](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/bin/node/runtime/src/lib.rs#L1239-L1279)에서 확인할 수 있습니다.

4. 런타임에서 `SigningTypes` 및 `SendTransactionTypes`를 구현하여 서명된 또는 서명되지 않은 트랜잭션을 제출할 수 있도록 지원합니다.

   ```rust
   impl frame_system::offchain::SigningTypes for Runtime {
        type Public = <Signature as traits::Verify>::Signer;
        type Signature = Signature;
   }

   impl<C> frame_system::offchain::SendTransactionTypes<C> for Runtime
   where
        RuntimeCall: From<C>,
   {
        type Extrinsic = UncheckedExtrinsic;
        type OverarchingCall = RuntimeCall;
   }
   ```

   이 구현 예제는 [Substrate 코드베이스](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/bin/node/runtime/src/lib.rs#L1280-L1292)에서 확인할 수 있습니다.

### 트랜잭션 서명을 위한 계정 추가

이제 팔렛의 오프체인 워커가 서명된 트랜잭션을 제출할 수 있도록 준비되었습니다.
팔렛을 준비하는 데는 다음 단계가 필요합니다:

- `--dev` 명령줄 옵션을 사용하여 개발 모드에서 노드를 실행하는 경우, 개발 계정을 위한 계정 키를 수동으로 생성하고 삽입합니다. 이를 위해 `node/src/service.rs` 파일을 수정합니다.

   ```rust
   pub fn new_partial(config: &Configuration) -> Result <SomeStruct, SomeError> {

   //...

  if config.offchain_worker.enabled {
  // 오프체인 워커가 트랜잭션을 서명하는 데 사용할 수 있도록 편의를 위해 트랜잭션 서명에 사용할 시드를 초기화합니다.
  // 학습자가 트랜잭션을 제출할 수 있도록 노드를 실행하기만 하면 트랜잭션을 볼 수 있습니다.
  // 일반적으로 이러한 키는 RPC 호출을 통해 `author_insertKey`에 삽입되어야 합니다.
   	sp_keystore::SyncCryptoStore::sr25519_generate_new(
   		&*keystore,
   		node_template_runtime::pallet_your_ocw_pallet::KEY_TYPE,
   		Some("//Alice"),
   	).expect("Creating key with account Alice should succeed.");
   	}
}
   ```

   이 예시에서는 `Alice` 계정의 키를 `KEY_TYPE`에 정의된 팔렛의 키스토어에 추가합니다.
   작동하는 예제는 [service.rs](https://github.com/jimmychu0807/substrate-offchain-worker-demo/blob/v2.0.0/node/src/service.rs#L87-L105) 파일을 참조하세요.

- 다른 계정을 사용하는 경우, `subkey`와 같은 도구를 사용하여 오프체인 워커가 사용할 계정을 생성한 다음, 해당 키를 노드 키스토어에 추가할 수 있습니다.

   - 체인 사양 파일의 구성을 수정합니다.
   - `author_insertKey` RPC 메서드를 사용하여 매개변수를 전달합니다.

   예를 들어, [Polkadot/Substrate Portal](https://polkadot.js.org/apps/#/rpc), Polkadot-JS API 또는 `curl` 명령을 사용하여 `author_insertKey` 메서드를 선택하고, 키 유형, 비밀 문구 및 공개 키 매개변수를 지정할 수 있습니다.
   
   ![`author_insertKey` 메서드를 사용하여 계정 삽입](/media/images/docs/author_insertKey.png)
   
   이 예제에서 `demo` 키 유형은 팔렛에서 선언한 `KEY_TYPE`과 일치합니다.

이제 팔렛의 오프체인 워커가 서명된 트랜잭션을 온체인에 제출할 수 있습니다.

## 서명되지 않은 트랜잭션

Substrate에서는 기본적으로 모든 서명되지 않은 트랜잭션을 거부합니다.
Substrate에서 특정 서명되지 않은 트랜잭션을 허용하려면 팔렛에 `ValidateUnsigned` 트레이트를 구현해야 합니다.

서명되지 않은 트랜잭션을 제출하려면 `ValidateUnsigned` 트레이트를 구현해야 하지만, 이 체크만으로 **오프체인 워커만** 트랜잭션을 제출할 수 있다는 것을 보장할 수는 없습니다.
이러한 트랜잭션은 항상 악용 가능한 잠재적인 공격 경로를 제공하며, 오프체인 워커가 신뢰할 수 있는 소스로 가정할 수 없습니다.

서명되지 않은 트랜잭션이 오프체인 워커에 의해 제출된 것인지 확인하기 위해 항상 추가적인 보호장치 또는 검증 로직을 구현해야 합니다.

오프체인 워커가 서명되지 않은 트랜잭션을 전송할 수 있도록 허용하려면 다음 단계를 수행하세요.

### 팔렛 구성

오프체인 워커가 서명되지 않은 트랜잭션을 전송할 수 있도록 하려면:

1. 팔렛의 `src/lib.rs` 파일을 텍스트 편집기로 엽니다.
2. [`validate_unsigned`](https://paritytech.github.io/substrate/master/frame_support/attr.pallet.html#validate-unsigned-palletvalidate_unsigned-optional) 매크로를 추가합니다.
   
	 예시:

   ```rust
   #[pallet::validate_unsigned]
   impl<T: Config> ValidateUnsigned for Pallet<T> {
   	type Call = Call<T>;

   		/// 이 모듈에 대한 서명되지 않은 호출을 검증합니다.
   		///
   		/// 기본적으로 서명되지 않은 트랜잭션은 허용되지 않지만, 여기에서 유효성 검사기를 구현함으로써 특정 호출(오프체인 워커가 생성한 호출)을 화이트리스트에 추가하고 유효한 것으로 표시할 수 있습니다.
   		fn validate_unsigned(source: TransactionSource, call: &Self::Call) -> TransactionValidity {
   		//...
   		}
   }
   ```

1. 다음과 같이 트레이트를 구현합니다.

   ```rust
   fn validate_unsigned(source: TransactionSource, call: &Self::Call) -> TransactionValidity {
   	let valid_tx = |provide| ValidTransaction::with_tag_prefix("my-pallet")
   		.priority(UNSIGNED_TXS_PRIORITY) // 이 줄 이전에 `UNSIGNED_TXS_PRIORITY`를 정의하세요
   		.and_provides([&provide])
   		.longevity(3)
   		.propagate(true)
   		.build();
   	// ...
   }
   ```

2. 호출되는 익스트린식을 확인하여 호출이 허용되는지 확인하고, 호출이 허용되는 경우 `ValidTransaction`을 반환하거나 호출이 허용되지 않는 경우 `TransactionValidityError`를 반환합니다.
   
	 예시:

   ```rust
   fn validate_unsigned(source: TransactionSource, call: &Self::Call) -> TransactionValidity {
   	// ...
   	match call {
   		RuntimeCall::my_unsigned_tx { key: value } => valid_tx(b"my_unsigned_tx".to_vec()),
   		_ => InvalidTransaction::Call.into(),
   	}
   }
   ```

   이 예시에서는 사용자가 특정 `my_unsigned_tx` 함수를 서명 없이 호출할 수 있습니다.
	 다른 함수를 호출하는 경우 서명된 트랜잭션이 필요합니다.

	 팔렛에서 `ValidateUnsigned`을 구현하는 예제는 [offchain-worker](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/offchain-worker/src/lib.rs#L301-L329)의 코드를 참조하세요.

3. `#[pallet::hooks]` 매크로와 `offchain_worker` 함수를 추가하여 서명되지 않은 트랜잭션을 전송합니다.

   ```rust
   #[pallet::hooks]
   impl<T: Config> Hooks<BlockNumberFor<T>> for Pallet<T> {
   	/// 오프체인 워커 진입점.
   	fn offchain_worker(block_number: T::BlockNumber) {
   		let value: u64 = 10;
   		// 온체인 익스트린식과 필요한 매개변수를 포함한 호출입니다.
   		let call = RuntimeCall::unsigned_extrinsic1 { key: value };

   		// `submit_unsigned_transaction`의 반환 타입은 `Result<(), ()>`입니다.
   		//	 참조: https://paritytech.github.io/substrate/master/frame_system/offchain/struct.SubmitTransaction.html
   		SubmitTransaction::<T, Call<T>>::submit_unsigned_transaction(call.into())
   			.map_err(|_| {
   			log::error!("offchain_unsigned_tx에서 실패했습니다.");
   		});
   	}
   }
   ```

   이 코드는 `let call = ...` 줄에서 호출을 준비하고, [`SubmitTransaction::submit_unsigned_transaction`](https://paritytech.github.io/substrate/master/frame_system/offchain/struct.SubmitTransaction.html)을 사용하여 트랜잭션을 제출하고, 콜백 함수에서 필요한 오류 처리를 수행합니다.

### 런타임 구성

1. 런타임에서 팔렛에 `ValidateUnsigned` 트레이트를 활성화하기 위해 `construct_runtime` 매크로에 `ValidateUnsigned` 타입을 추가합니다.

   예시:

   ```rust
   construct_runtime!(
   	pub enum Runtime where
   		Block = Block,
   		NodeBlock = opaque::Block,
   		UncheckedExtrinsic = UncheckedExtrinsic
   	{
   		// ...
   		OcwPallet: pallet_ocw::{Pallet, Call, Storage, Event<T>, ValidateUnsigned},
   	}
   );
   ```

1. [서명된 트랜잭션 제출](#서명된-트랜잭션-제출)에서 설명한 대로 런타임에 `ValidateUnsigned` 타입을 추가합니다.

   전체 예시는 [offchain-worker](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/examples/offchain-worker) 예제 팔렛을 참조하세요.

## 서명된 페이로드

서명된 페이로드를 사용하여 서명되지 않은 트랜잭션을 전송하는 방법은 서명되지 않은 트랜잭션을 전송하는 방법과 유사합니다.
트랜잭션을 전송하려면 다음 단계를 수행해야 합니다.

- 팔렛에 `ValidateUnsigned` 트레이트를 구현합니다.
- 런타임에서 이 팔렛에 `ValidateUnsigned` 타입을 추가합니다.
- 서명할 데이터 구조, 즉 서명된 페이로드를 준비하기 위해 [`SignedPayload`](https://paritytech.github.io/substrate/master/frame_system/offchain/trait.SignedPayload.html) 트레이트를 구현합니다.
- 서명된 페이로드로 트랜잭션을 전송합니다.

자세한 내용은 서명되지 않은 트랜잭션을 전송하는 방법에 대한 섹션을 참조하세요.

서명되지 않은 트랜잭션은 항상 잠재적인 공격 경로를 제공하며, 오프체인 워커가 신뢰할 수 있는 소스로 가정할 수 없으므로 트랜잭션을 제출하기 전에 추가적인 보호장치 또는 검증 로직을 구현해야 합니다.

서명되지 않은 트랜잭션을 전송하는 방법과 서명된 페이로드를 포함한 서명되지 않은 트랜잭션을 전송하는 방법의 차이점은 다음 코드 예제에서 설명합니다.

데이터 구조를 서명할 수 있도록 하려면:

1. [`SignedPayload`](https://paritytech.github.io/substrate/master/frame_system/offchain/trait.SignedPayload.html)을 구현합니다.

   예시:

   ```rust
   #[derive(Encode, Decode, Clone, PartialEq, Eq, RuntimeDebug, scale_info::TypeInfo)]
   pub struct Payload<Public> {
   	number: u64,
   	public: Public,
   }

   impl<T: SigningTypes> SignedPayload<T> for Payload<T::Public> {
   	fn public(&self) -> T::Public {
   	self.public.clone()
   }
   }
   ```

  서명된 페이로드의 예제는 [offchain-worker](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/offchain-worker/src/lib.rs#L348-L361)의 코드를 참조하세요.

1. `offchain_worker` 함수에서 서명자를 호출한 다음 트랜잭션을 전송하는 함수를 호출합니다.

   ```rust
   #[pallet::hooks]
   impl<T: Config> Hooks<BlockNumberFor<T>> for Pallet<T> {
   	/// 오프체인 워커 진입점.
   	fn offchain_worker(block_number: T::BlockNumber) {
   		let value: u64 = 10;

   		// 페이로드에 서명하기 위해 서명자를 검색합니다.
   		let signer = Signer::<T, T::AuthorityId>::any_account();

   		// `send_unsigned_transaction`의 반환 타입은 `Option<(Account<T>, Result<(), ()>)>`입니다.
   		//	 반환된 결과는 다음과 같습니다:
   		//	 - `None`: 트랜잭션을 전송할 수 있는 계정이 없음
   		//	 - `Some((account, Ok(())))`: 트랜잭션이 성공적으로 전송됨
   		//	 - `Some((account, Err(())))`: 트랜잭션 전송 중 오류 발생
   		if let Some((_, res)) = signer.send_unsigned_transaction(
   			// 이 줄은 페이로드를 준비하고 반환합니다.
   			|acct| Payload { number, public: acct.public.clone() },
   			|payload, signature| RuntimeCall::some_extrinsics { payload, signature },
   		) {
   			match res {
   				Ok(()) => log::info!("서명된 페이로드를 포함한 서명되지 않은 트랜잭션 성공적으로 전송."),
   				Err(()) => log::error!("서명된 페이로드를 포함한 서명되지 않은 트랜잭션 전송 실패."),
   			};
   		} else {
   			// `None`인 경우: 트랜잭션을 전송할 수 있는 계정이 없음
   			log::error!("로컬 계정을 찾을 수 없음");
   		}
   	}
   }
   ```

   이 코드는 `signer`를 검색한 다음 `send_unsigned_transaction()`을 두 개의 함수 클로저와 함께 호출합니다.
   첫 번째 함수 클로저는 사용할 페이로드를 반환하고, 두 번째 함수 클로저는 페이로드와 서명이 포함된 온체인 호출을 반환합니다.
   이 호출은 `Option<(Account<T>, Result<(), ()>)>` 결과 타입을 반환하여 다음 결과를 처리할 수 있습니다:

   - 트랜잭션을 전송할 수 있는 계정이 없는 경우 `None`입니다.
   - 트랜잭션이 성공적으로 전송된 경우 `Some((account, Ok(())))`입니다.
   - 트랜잭션 전송 중 오류가 발생한 경우 `Some((account, Err(())))`입니다.

2. 제공된 서명이 페이로드를 서명한 공개 키와 일치하는지 확인합니다.

   ```rust
   #[pallet::validate_unsigned]
   impl<T: Config> ValidateUnsigned for Pallet<T> {
   	type Call = Call<T>;



fn validate_unsigned(_source: TransactionSource, call: &Self::Call) -> TransactionValidity {
    let valid_tx = |provide| ValidTransaction::with_tag_prefix("ocw-demo")
        .priority(UNSIGNED_TXS_PRIORITY)
        .and_provides([&provide])
        .longevity(3)
        .propagate(true)
        .build();

    match call {
        RuntimeCall::unsigned_extrinsic_with_signed_payload {
        ref payload,
        ref signature
        } => {
        if !SignedPayload::<T>::verify::<T::AuthorityId>(payload, signature.clone()) {
            return InvalidTransaction::BadProof.into();
        }
        valid_tx(b"unsigned_extrinsic_with_signed_payload".to_vec())
        },
        _ => InvalidTransaction::Call.into(),
    }
}
}

이 예제는 [`SignedPayload`](https://paritytech.github.io/substrate/master/frame_system/offchain/trait.SignedPayload.html)을 사용하여 페이로드의 공개 키가 제공된 시그니처와 동일한지 확인합니다.
그러나 이 예제의 코드는 제공된 `signature`가 `payload` 내에 포함된 `public` 키에 대해 유효한지만 확인합니다.
이 검사는 사인한 페이로드를 사용하여 상태를 수정하는 권한이 있는지 여부를 확인하지 않습니다.
이 간단한 검사는 권한이 없는 사용자가 사인된 페이로드를 사용하여 상태를 수정하는 것을 방지하지 않습니다.

이 코드의 작동 예제는 [오프체인 함수 호출](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/offchain-worker/src/lib.rs#L508-L536)와 [`ValidateUnsigned`](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/examples/offchain-worker/src/lib.rs#L305-L329)의 구현을 참조하십시오.

## 다음으로 어디로 가야 할까요?

이 자습서에서는 오프체인 워커를 사용하여 온체인 저장소에 트랜잭션을 전송하는 간단한 예제를 제공합니다.
더 자세한 내용은 다음 리소스를 참조하십시오:

- [오프체인 작업](/learn/offchain-operations/)
- [오프체인 워커 예제](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame/examples/offchain-worker)
- [오프체인 워커 데모](https://github.com/jimmychu0807/substrate-offchain-worker-demo/tree/v2.0.0/pallets/ocw)