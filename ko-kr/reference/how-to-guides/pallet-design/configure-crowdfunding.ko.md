크라우드펀딩 구성하기
FRAME 팔렛을 사용하여 크라우드펀딩 캠페인을 만드는 방법을 알려드립니다.
키워드:
  - 팔렛 디자인
  - 중급
  - 런타임
  - 자식 트라이

이 안내서에서는 여러 토큰 계정을 제어하고 자식 스토리지에 데이터를 저장하는 팔렛을 구축하는 방법을 보여줍니다.
이 구조는 크라우드펀딩 앱을 구축하는 데 유용합니다.

이 안내서에서는 작은 Merkle proof를 사용하여 기여자가 기여한 것을 증명할 수 있는 자식 스토리지 트라이의 사용에 초점을 맞출 것입니다.
간단한 기여 증명을 할 수 있다면 사용자는 크라우드론에 참여하여 보상을 받을 수 있습니다.

이 안내서에서는 각기 다른 크라우드펀드 캠페인을 위해 새로운 자식 트라이를 생성하는 방법을 보여줍니다.
사용자는 크라우드펀드를 시작할 수 있으며, 크라우드펀드의 목표 금액, 종료 시간 및 목표가 달성되면 풀된 자금을 받을 수 있는 수혜자를 지정할 수 있습니다.
펀드가 성공하지 못하면 원래 소유자에게 자금이 반환될 수 있습니다.

## 시작하기 전에

이 안내서를 따라가고 자체 팔렛에 크라우드펀딩 기능을 구현하려면 팔렛에 다음 종속성을 포함하십시오:

```rust
use frame_support::{
	ensure,
	pallet_prelude::*,
	sp_runtime::traits::{AccountIdConversion, Hash, Saturating, Zero},
	storage::child,
	traits::{Currency, ExistenceRequirement, Get, ReservableCurrency, WithdrawReasons},
	PalletId,
};
use frame_system::{pallet_prelude::*, ensure_signed};
use super::*;
```

이 안내서는 팔렛 로직에 따라 자체 오류와 이벤트를 생성하는 방법을 알고 있다고 가정합니다.

## 팔렛 구성하기

1. 팔렛의 구성 특성을 선언합니다.

   `Event` 타입 외에도, 이 팔렛은 다음 특성이 필요합니다:

   - **`Currency`** - 크라우드펀드에 사용될 통화입니다.
   - **`SubmissionDeposit`** - 크라우드펀드 소유자가 보증금으로 보유해야 할 금액입니다.
   - **`MinContribution`** - 크라우드펀드에 기여할 수 있는 최소 금액입니다.

   다음 타입으로 특성을 확장합니다:

   ```rust
   /// 팔렛의 구성 특성.
   #[pallet::config]
   pub trait Config: frame_system::Config {
   	type RuntimeEvent: From<Event<Self>> + IsType<<Self as frame_system::Config>::RuntimeEvent>;
   	type Currency: ReservableCurrency<Self::AccountId>;
   	type SubmissionDeposit: Get<BalanceOf<Self>>;
   	type MinContribution: Get<BalanceOf<Self>>;
   	type RetirementPeriod: Get<Self::BlockNumber>;
   }
   ```

   그리고 다음 타입 별칭을 추가합니다:

   ```rust
   pub type FundIndex = u32;
   type AccountIdOf<T> = <T as frame_system::Config>::AccountId;
   type BalanceOf<T> = <<T as Config>::Currency as Currency<AccountIdOf<T>>>::Balance;
   ```

1. 크라우드펀드 구성 정보를 저장하는 구조체를 생성합니다.

   ```rust
   #[derive(Encode, Decode, Default, PartialEq, Eq, TypeInfo)]
   #[cfg_attr(feature = "std", derive(Debug))]
   pub struct FundInfo<AccountId, Balance, BlockNumber> {
   	/// 캠페인이 성공하면 자금을 받을 계정입니다.
   	beneficiary: AccountId,
   	/// 예치금액입니다.
   	deposit: Balance,
   	/// 모금된 총 금액입니다.
   	raised: Balance,
   	/// 자금 조달이 성공해야 하는 블록 번호입니다.
   	end: BlockNumber,
   	/// `raised`의 상한입니다.
   	goal: Balance,
   }
   ```

   그리고 이를 더 쉽게 사용하기 위한 새로운 별칭을 생성합니다:

   ```rust
   type FundInfoOf<T> =
   	FundInfo<AccountIdOf<T>, BalanceOf<T>, <T as frame_system::Config>::BlockNumber>;
   ```

1. 스토리지 아이템을 선언합니다.

   스토리지 아이템은 어떤 사용자가 어떤 펀드에 얼마나 기여했는지 추적합니다.
   다음 타입을 정의하여 스토리지 아이템을 선언합니다:

   ```rust
   #[pallet::storage]
   #[pallet::getter(fn funds)]
   /// 모든 펀드에 대한 정보입니다.
   pub(super) type Funds<T: Config> = StorageMap<
   	_,
   	Blake2_128Concat,
   	FundIndex,
   	FundInfoOf<T>,
   	OptionQuery,
   >;

   #[pallet::storage]
   #[pallet::getter(fn fund_count)]
   /// 지금까지 할당된 펀드의 총 수입니다.
   pub(super) type FundCount<T: Config> = StorageValue<_, FundIndex, ValueQuery>;
   ```

## 자식 트라이 API 도우미 함수 작성하기

팔렛의 디스패처에게 자금 풀의 계정 ID를 제공하는 함수를 작성합니다.

1. `impl<T: Config> Pallet<T>` 내부에 다음을 작성합니다:

   ```rust
   pub fn fund_account_id(index: FundIndex) -> T::AccountId {
   	const PALLET_ID: ModuleId = ModuleId(*b"ex/cfund");
   	PALLET_ID.into_sub_account(index)
   }
   ```

1. 고유한 [ChildInfo](https://paritytech.github.io/substrate/master/sp_storage/enum.ChildInfo.html) ID를 생성합니다:

   ```rust
   pub fn id_from_index(index: FundIndex) -> child::ChildInfo {
   		let mut buf = Vec::new();
   		buf.extend_from_slice(b"crowdfnd");
   		buf.extend_from_slice(&index.to_le_bytes()[..]);

   		child::ChildInfo::new_default(T::Hashing::hash(&buf[..]).as_ref())
   }
   ```

1. 다음과 같은 도우미 함수를 작성합니다. 이 함수들은 [Child API](https://paritytech.github.io/substrate/master/frame_support/storage/child/index.html)를 사용합니다:

   - **`pub fn contribution_put`**: [`put`](https://paritytech.github.io/substrate/master/frame_support/storage/child/fn.put.html)를 사용하여 관련된 자식 트라이에 기여를 기록합니다.

   - **`pub fn contribution_get`**: [`get`](https://paritytech.github.io/substrate/master/frame_support/storage/child/fn.get.html)을 사용하여 관련된 자식 트라이에서 기여를 조회합니다.

   - **`pub fn contribution_kil`**: [`kill`](https://paritytech.github.io/substrate/master/frame_support/storage/child/fn.kill.html)을 사용하여 관련된 자식 트라이에서 기여를 제거합니다.

   - **`pub fn crowdfund_kill`**: [`kill_storage`](https://paritytech.github.io/substrate/master/frame_support/storage/child/fn.kill_storage.html)를 사용하여 관련된 자식 트라이에서 모든 기여자 정보를 한 번에 제거합니다.

## 디스패처 함수 작성하기

이 팔렛의 디스패처를 작성하는 방법에 대해 다음 단계를 안내합니다.
디스패처의 로직 내에서 여러 검사를 수행한 후, 각 함수는 연관된 메서드를 사용하여 `Funds<T>` 스토리지 맵을 변경합니다.
팔렛의 `create` 함수는 또한 2단계에서 생성한 `FundInfo` 구조체를 사용합니다.

1. 새로운 펀드 생성하기

   `fn create`:

   - [`T::Currency::withdraw`](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/currency/trait.Currency.html#tymethod.withdraw)를 사용하여 불균형 변수를 생성합니다.
   - `FundInfo` 구조체를 사용하여 `Funds` 스토리지 아이템을 업데이트합니다.
   - `Created` 이벤트를 생성합니다.

1. 기존 펀드에 기여하기

   `fn contribute`:

   - `ensure!`를 사용하여 사전 안전성 검사를 수행합니다.
   - `T::Currency::transfer`를 사용하여 기여를 펀드에 추가합니다.
   - 도우미 함수 `contribution_put`을 사용하여 자식 트라이에 기여를 기록합니다.
   - `Contributed` 이벤트를 생성합니다.

1. 기여자의 전체 잔액을 펀드에서 인출하기

   `fn withdraw`:

   - `ensure!`를 사용하여 사전 안전성 검사를 수행합니다.
   - [`T::Currency::resolve_into_existing`](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/currency/trait.Currency.html#method.resolve_into_existing)를 사용하여 자금을 반환합니다.
   - 새로운 잔액을 계산하고 자식 트라이 도우미 함수 `funds`, `contribution_get`, `contribution_kill`을 사용하여 스토리지를 업데이트합니다.
   - `Withdrew` 이벤트를 생성합니다.

1. 퇴직 기간이 만료된 후 전체 크라우드펀드를 해산하기

   `fn dissolve`:

   - `ensure!`를 사용하여 사전 안전성 검사를 수행합니다.
   - [`T::Currency::resolve_creating`](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/currency/trait.Currency.html#method.resolve_creating)을 사용하여 해산자가 자금을 수집할 수 있도록 합니다.
   - 자식 트라이 도우미 함수 `crowdfund_kill`을 사용하여 스토리지에서 기여자 정보를 제거합니다.
   - `Dissolved` 이벤트를 생성합니다.

1. 성공한 크라우드펀드의 수혜자에게 지급하기

   `fn dispense`:

   - 수혜자와 호출자 모두 자금을 수집하기 위해 [`T::Currency::resolve_creating`](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/currency/trait.Currency.html#method.resolve_creating)를 사용합니다.
   - 이 함수를 호출한 계정에 초기 예치금을 제공하여 스토리지를 정리하는 인센티브를 제공합니다.
   - `<Funds<T>>::remove(index);`와 `Self::crowdfund_kill(index);`를 사용하여 펀드를 스토리지에서 제거합니다. 이를 통해 모든 기여자를 한 번에 스토리지에서 제거합니다.

## 예제

- [`pallet_simple_crowdfund`](https://github.com/substrate-developer-hub/substrate-how-to-guides/blob/main/example-code/template-node/pallets/simple-crowdfund/src/lib.rs#L1)

## 자원

- [Currency Imbalance trait](https://paritytech.github.io/substrate/master/frame_support/traits/tokens/imbalance/trait.Imbalance.html)
- [Child trie API](https://paritytech.github.io/substrate/master/frame_support/storage/child/index.html)
- [`extend_from_slice`](https://paritytech.github.io/substrate/master/frame_support/dispatch/struct.Vec.html#method.extend_from_slice)
- [`using_encode`](https://paritytech.github.io/substrate/master/frame_support/pallet_prelude/trait.Encode.html#method.using_encoded)