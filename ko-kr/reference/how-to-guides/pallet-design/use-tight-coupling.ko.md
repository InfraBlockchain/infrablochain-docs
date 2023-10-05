---
title: 타이트한 팔레트 결합 사용하기
description:
keywords:
---

타이트한 팔레트 결합은 기존 팔레트에서 타입과 메소드를 재사용하는 팔레트를 작성하는 기술입니다.

공통 타입과 메소드에 접근해야하는 별개의 팔레트로 일부 런타임 로직을 분리하는 데 유용합니다.
팔레트 간의 타이트한 결합과 루즈한 결합에 대해 자세히 알아보려면 [팔레트 결합](/build/pallet-coupling)을 참조하십시오.

이 안내서에서는 기존 팔레트의 메소드를 자신의 팔레트 내에서 사용하는 방법을 보여줍니다.

## 시작하기 전에

이 안내서는 다른 팔레트와 결합하려는 팔레트가 있다고 가정합니다.
작업할 팔레트가 아직 없다면 [템플릿 팔레트](https://github.com/substrate-developer-hub/substrate-node-template/blob/main/pallets/template/src/lib.rs)를 사용하십시오.

1. 워크스페이스 구성

   로컬 워크스페이스에 있는 `special-pallet`이라는 팔레트를 사용하려고 가정합니다.
   팔레트의 `Cargo.toml` 파일에서 경로를 제공하여 팔레트를 가져옵니다:

   ```toml
   special-pallet = { path = '../special-pallet', default-features = false }
   ```

1. 팔레트에 트레이트 바운드 추가

   이제 팔레트의 구성 트레이트 주위에 트레이트 바운드를 생성하기만 하면 됩니다:

   ```rust
   pub trait Config: frame_system::Config + special_pallet::Config {
       // --snip--
   }
   ```

1. 게터 함수 사용

   `special-pallet`에서 메소드를 사용하려면 다음과 같이 호출하면 됩니다:

   ```rust
   // `special-pallet` 팔레트에서 멤버 가져오기
   let who = special_pallet::Pallet::<T>::get();
   ```

   위의 코드 조각은 `special-pallet`에 `get()`이라는 메소드가 있고, 이 메소드가 계정 ID의 벡터를 반환한다고 가정합니다.

## 예시

- FRAME의 [Bounties](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/bounties) 및 [Tipping](https://github.com/paritytech/polkadot-sdk/blob/master/substrate/frame/tips) 팔레트와 Treasury 팔레트

## 자원

- [팔레트 결합](/build/pallet-coupling)
- [팔레트 간 루즈한 결합 사용하기](/reference/how-to-guides/pallet-design/use-loose-coupling/)