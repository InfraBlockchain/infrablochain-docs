---
title: 런타임에 컬렉터블 추가하기
description:
tutorial:
---

이제 기본 기능을 갖춘 디지털 컬렉터블 트랜잭션을 위한 사용자 정의 팔레트를 갖게 되었습니다.
다음 단계는 이 팔레트를 블록체인에서 기능을 노출하기 위해 런타임에 추가하는 것입니다.
하지만 그 전에, [workspace에 팔레트 추가](/tutorials/collectibles-workshop/03-create-pallet/#add-the-pallet-to-the-workspace)에서 지시한 대로 노드의 매니페스트에 컬렉터블 팔레트가 포함되었는지 확인해보세요.

workspace의 매니페스트를 확인하려면 다음을 수행하세요:

1. 필요한 경우 새 터미널을 엽니다.
   
2. workspace의 `workshop-node-template` 디렉토리로 이동합니다.

3. 노드 workspace의 Cargo.toml 파일의 내용을 확인합니다.
   
   ```bash
   more Cargo.toml
   ```

   컬렉터블 팔레트가 workspace의 구성원으로 나열되어 있는지 확인할 수 있어야 합니다.
   예를 들어:

   ```toml
   [workspace]
   members = [
        "node",
        "pallets/collectibles",
        "pallets/template",
        "runtime",
   ]
   [profile.release]
   panic = "unwind"
   ```

   컬렉터블 팔레트가 포함되어 있다면, 런타임을 업데이트할 준비가 되었습니다.

## 런타임 파일 업데이트하기

컬렉터블 팔레트를 런타임에 추가하려면 다음을 수행하세요:

1. 코드 편집기에서 `runtime/Cargo.toml` 파일을 열고 컬렉터블 팔레트를 로컬 종속성 및 런타임의 표준 기능에 추가합니다.
   
   예를 들어:

   ```toml
   # Local Dependencies
   pallet-template = { version = "4.0.0-dev", default-features = false, path = "../pallets/template" }
   collectibles = { default-features = false, path = "../pallets/collectibles" }

   [features]
   default = ["std"]
   std = [
   ...
        "collectibles/std",
   ...
   ```

1. 변경 사항을 저장하세요.
   
1. `runtime/src/lib.rs` 파일을 엽니다.

1. 컬렉터블 팔레트를 런타임에 가져옵니다.

   ```rust
   /// 컬렉터블 팔레트를 가져옵니다.
   pub use collectibles;
   ```

1. 컬렉터블 팔레트를 위한 configurate 트레잇을 구현합니다.
   
   ```rust
   impl collectibles::Config for Runtime {
        type RuntimeEvent = RuntimeEvent;
        type Currency = Balances;
        type CollectionRandomness = RandomnessCollectiveFlip;
        type MaximumOwned = frame_support::pallet_prelude::ConstU32<100>;
   }
   ```

1. `construct_runtime!` 매크로에 팔레트를 추가합니다.
   
   ```rust
   construct_runtime!(
        pub struct Runtime
        where
            Block = Block,
            NodeBlock = opaque::Block,
            UncheckedExtrinsic = UncheckedExtrinsic,
        {
            System: frame_system,
            RandomnessCollectiveFlip: pallet_randomness_collective_flip,
            Timestamp: pallet_timestamp,
            Aura: pallet_aura,
            Grandpa: pallet_grandpa,
            Balances: pallet_balances,
            TransactionPayment: pallet_transaction_payment,
            Sudo: pallet_sudo,
            TemplateModule: pallet_template,
            Collectibles: collectibles,
        }
   );
   ```

1. 다음 명령을 실행하여 업데이트된 런타임으로 블록체인 노드를 컴파일합니다.
   
   ```bash
   cargo build --release
   ```

   노드가 컴파일되면 사용자 정의 팔레트가 준비된 상태입니다.

## 컬렉터블 팔레트에 접근하기

새로 컴파일된 노드를 갖게 되었으므로, 블록체인을 다시 시작하고 새로운 팔레트를 사용할 수 있습니다.

1. 다음 명령을 실행하여 블록체인 노드를 시작합니다.
   
   ```bash
   ./target/release/node-template --dev
   ```

2. [Polkadot/Substrate Portal](https://polkadot.js.org/apps/#/explorer)을 열고 로컬 **Development** 노드에 연결합니다.
   
   ![로컬 노드에 연결](/media/images/docs/tutorials/collectibles-workshop/connect-to-local-endpoint.png)

1. **Developer**를 클릭하고 **Extrinsics**를 선택합니다.

2. **Collectibles** 팔레트를 선택하고 호출 가능한 함수 목록을 확인합니다.
   
   ![컬렉터블 팔레트의 호출 가능한 함수들](/media/images/docs/tutorials/collectibles-workshop/collectibles-pallet.png)

3. **createCollectible** 함수를 선택하고 **Submit Transaction**을 클릭한 다음 **Sign and Submit**을 클릭합니다.

4. **Network**를 클릭하고 **Explorer**를 선택하여 새로운 컬렉터블을 생성하는 이벤트를 확인합니다.
   
   ![CollectibleCreated 이벤트](/media/images/docs/tutorials/collectibles-workshop/create-collectible-event.png)

   Polkadot/Substrate Portal을 사용하여 함수, 이벤트 및 오류가 예상대로 작동하는지 테스트할 수 있습니다.
   그러나 사용자를 유치하기 위해 애플리케이션을 개발할 때, 다른 사용자가 생성하고 판매를 위해 올린 컬렉터블을 둘러보고 입찰할 수 있는 사용자 정의 인터페이스를 개발하는 것이 좋습니다.
   매력적인 인터페이스를 구축하는 것은 별도의 예술이므로, 이 워크샵에서는 애플리케이션이 어떻게 보일지 또는 제공해야 할 사용자 경험에 대해 자세히 다루지 않을 것입니다.
   대부분의 경우, 블록체인에서 실행되는 애플리케이션의 프론트엔드를 구축하기 위해 TypeScript와 React, Vue, Angular와 같은 웹 개발 프레임워크를 사용하는 것이 좋습니다.