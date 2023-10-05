---
title: 콜레이터 선택
description:
keywords:
  - 콜레이터
  - 파라체인
  - 커뮬러스
  - 디자인
  - 선택
  - 경제
  - 게임 이론
---

이 가이드는 제품 런칭을 준비하는 팀들에게 유용하며, 콜레이터 선택에 대한 다양한 옵션을 탐색합니다.

파라체인 네트워크의 콜레이터가 설정되어 있어야 하는데, 이는 검열을 방지하기 위해 _일부_ 중립적인 콜레이터가 존재해야 함을 의미합니다. 하지만 반드시 다수의 콜레이터가 필요한 것은 아닙니다.
또한 너무 많은 콜레이터를 가지는 것은 네트워크 속도를 늦출 수 있으므로 피해야 합니다.
이 가이드는 파라체인 네트워크를 설계할 때 고려해야 할 사항을 안내합니다.

## 콜레이터 선택

콜레이터 선택 방법은 자유롭게 선택할 수 있습니다.
주로 지분 투표 또는 위원회나 민주주의와 같은 다른 출처를 통해 콜레이터를 직접 지정하는 방법이 일반적입니다.
두 경우 모두, 필요에 가장 적합한 로직을 구현하기 위해 팔렛을 생성하세요.

### 지분 투표

Cumulus [`collator-selection` 팔렛](https://github.com/paritytech/polkadot-sdk/blob/master/cumulus/pallets/collator-selection/src/lib.rs)은 지분 투표를 구현하는 실용적인 예입니다.

### 온체인 거버넌스 사용

특정 출처를 구현하여 해당 출처의 구성원이 콜레이터가 될 수 있도록 합니다. 민주주의 팔렛을 사용하여 이러한 구성원을 선출하고, 콜레이터 선택을 다루는 팔렛에서 이를 정의하세요:

```rust
    /// 이 팔렛의 구성 특성.
	#[pallet::config]
	pub trait Config: frame_system::Config {
        // --snip-- //
        type MySpecialOrigin: EnsureOrigin<Self::RuntimeOrigin>;
    }
    // --snip-- //
    #[pallet::call]
	impl<T: Config> Pallet<T> {
		/// 일부 set-collator 디스패처.
		#[pallet::weight(some_weight)]
		pub fn set_collator( origin: OriginFor<T>) -> DispatchResultWithPostInfo {
            T::MySpecialOrigin::ensure_origin(origin)?;
            // --snip-- //
        }
```

콜레이터에 대한 인센티브를 구현하는 다양한 방법도 있습니다.
[이 예시](https://github.com/PureStake/moonbeam/blob/master/pallets/parachain-staking/src/lib.rs)를 살펴보세요.

## 예시

- [콜레이터 선택과 거래 수수료를 이용한 인센티브 구현](https://github.com/paritytech/polkadot-sdk/blob/master/cumulus/pallets/collator-selection/src/lib.rs)인 Cumulus 구현.
- [인플레이션적 통화 정책 스테이킹 방식을 이용한 콜레이터 선택](https://github.com/PureStake/moonbeam/blob/master/pallets/parachain-staking/src/lib.rs)인 Moonbeam 구현.

## 자료

- [파라체인 DevOps 최적 사례](https://gist.github.com/lovelaced/cddc1c7234b883ee37e71cf4a1d63cac)