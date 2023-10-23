---
title: 사용자 정의 팔레트에서 매크로 사용하기
description: FRAME 매크로의 뼈대를 사용하여 Substrate 런타임에 대한 사용자 정의 팔레트를 생성하는 방법을 설명합니다.
keywords:
---

이 튜토리얼은 [FRAME](/reference/frame-macros/) 개발 환경의 일부인 **매크로**를 사용하여 Substrate 런타임에 대한 사용자 정의 팔레트를 생성하는 방법을 설명합니다.

이 튜토리얼에서는 간단한 **존재 증명** 애플리케이션을 구축합니다. 존재 증명은 블록체인에 객체에 대한 정보를 저장함으로써 디지털 객체의 진위성과 소유권을 검증하는 접근 방식입니다.
블록체인은 객체와 관련된 타임스탬프와 계정을 연결하기 때문에 블록체인 기록은 특정한 객체가 특정한 날짜와 시간에 존재했음을 "증명"할 수 있습니다.
또한 해당 날짜와 시간에 기록의 소유자가 누구였는지도 확인할 수 있습니다.

## 디지털 객체와 해시

블록체인에 전체 파일을 저장하는 대신, 해당 파일의 [암호 해시](https://en.wikipedia.org/wiki/Cryptographic_hash_function)만 저장하는 것이 훨씬 효율적일 수 있습니다.
이를 "디지털 지문"이라고도 합니다.
해시는 작고 고유한 해시 값으로 임의 크기의 파일을 효율적으로 저장할 수 있게 해줍니다.
파일에 대한 어떠한 변경도 다른 해시를 결과로 만들기 때문에 사용자는 해시를 계산하고 그 해시를 체인에 저장된 해시와 비교함으로써 파일의 유효성을 증명할 수 있습니다.

![파일 해시](/media/images/docs/tutorials/custom-pallet/file-hash.png)

## 디지털 객체와 계정 서명

블록체인은 [공개 키 암호](https://en.wikipedia.org/wiki/Public-key_cryptography)를 사용하여 디지털 신원을 계정에 매핑합니다.
블록체인은 디지털 객체의 해시를 저장하는 데 사용하는 계정을 트랜잭션의 일부로 기록합니다.
계정 정보가 트랜잭션의 일부로 저장되기 때문에 해당 계정의 개인 키를 소유한 사용자는 나중에 파일을 업로드한 사람으로서 소유권을 증명할 수 있습니다.

## 이 튜토리얼을 완료하는 데 필요한 시간은 얼마나 걸리나요?

이 튜토리얼은 Rust 코드를 컴파일하는 것을 요구하며, 약 1~2시간 정도 소요됩니다.

## 시작하기 전에

이 튜토리얼에서는 작동하는 코드를 다운로드하고 사용합니다. 시작하기 전에 다음을 확인하세요:

- [Rust와 Rust 툴체인](/install/)을 설치하여 Substrate 개발 환경을 구성했는지 확인하세요.

- [로컬 블록체인 생성](/tutorials/build-a-blockchain/build-local-blockchain/)을 완료하고 Substrate 노드 템플릿을 로컬에 설치했는지 확인하세요.

- [네트워크 시뮬레이션](/tutorials/build-a-blockchain/simulate-network/)에서 설명한 대로 미리 정의된 계정을 사용하여 단일 컴퓨터에서 노드를 시작했는지 확인하세요.

- 소프트웨어 개발에 대해 일반적으로 알고 있으며 명령 줄 인터페이스를 사용할 수 있는지 확인하세요.

## 튜토리얼 목표

이 튜토리얼을 완료함으로써 다음 목표를 달성할 수 있습니다:

- 사용자 정의 팔레트의 기본 구조를 배우십시오.

- Rust 매크로가 필요한 코드 작성을 간소화하는 방법에 대한 예제를 확인하십시오.

- 사용자 정의 팔레트가 포함된 블록체인 노드를 시작하십시오.

- 존재 증명 팔레트를 노출하는 프론트엔드 코드를 추가하십시오.

## 애플리케이션 설계

존재 증명 애플리케이션은 다음과 같은 호출 가능한 함수를 노출합니다:

- `create_claim()`은 사용자가 해시를 업로드하여 파일의 존재를 주장할 수 있게 합니다.

- `revoke_claim()`은 현재 소유자가 소유권을 취소할 수 있게 합니다.

## 사용자 정의 팔레트 빌드

Substrate 노드 템플릿에는 FRAME 기반 런타임이 있습니다.
[런타임 개발](/learn/runtime-development)에서 배운 대로, FRAME은 팔레트라고 불리는 모듈을 조합하여 Substrate 런타임을 구축할 수 있는 코드 라이브러리입니다.
팔레트는 블록체인이 수행할 수 있는 작업을 정의하는 특수한 논리적 단위로 생각할 수 있습니다.
Substrate은 FRAME 기반 런타임에서 사용할 수 있는 사전 구축된 팔레트를 제공합니다.

![런타임 구성](/media/images/docs/tutorials/custom-pallet/frame-runtime.png)

이 튜토리얼에서는 사용자 정의 팔레트를 생성하여 사용자 정의 블록체인에 포함시키는 방법을 보여줍니다.

### 팔레트에 대한 스캐폴딩 설정

이 튜토리얼에서는 처음부터 사용자 정의 팔레트를 생성하는 방법을 보여줍니다.
따라서 첫 번째 단계는 노드 템플릿 디렉토리의 일부 파일과 내용을 제거하는 것입니다.

1. 터미널 셸을 열고 노드 템플릿의 루트 디렉토리로 이동합니다.

2. 다음 명령을 실행하여 `pallets/template/src` 디렉토리로 이동합니다.

   ```bash
   cd pallets/template/src
   ```

3. 다음 파일을 제거합니다.

   ```bash
   benchmarking.rs
   mock.rs
   tests.rs
   ```

4. 텍스트 편집기에서 `lib.rs` 파일을 엽니다.

   이 파일에는 새 팔레트의 템플릿으로 사용할 수 있는 코드가 포함되어 있습니다.
   이 튜토리얼에서는 템플릿 코드를 사용하지 않습니다.
   그러나 삭제하기 전에 템플릿 코드를 검토하여 제공되는 내용을 확인할 수 있습니다.

5. `lib.rs` 파일의 모든 줄을 삭제합니다.

6. 다음 코드를 추가하여 네이티브 Rust 바이너리(`std`)와 WebAssembly(`no_std`) 바이너리를 빌드하는 데 필요한 매크로를 추가합니다.

   ```rust
   #![cfg_attr(not(feature = "std"), no_std)]
   ```

   런타임에서 사용하는 모든 팔레트는 `no_std` 기능으로 컴파일되도록 설정해야 합니다.

7. 다음 코드를 복사하여 사용자 정의 팔레트에 필요한 스켈레톤 팔레트 종속성 및 [매크로](/reference/frame-macros)를 추가합니다.

   ```rust
   // 팔레트 항목을 다른 크레이트 네임스페이스에서 액세스할 수 있도록 다시 내보냅니다.
   pub use pallet::*;

   #[frame_support::pallet]
   pub mod pallet {
     use frame_support::pallet_prelude::*;
     use frame_system::pallet_prelude::*;

     #[pallet::pallet]
     #[pallet::generate_store(pub(super) trait Store)]
     pub struct Pallet<T>(_);

     #[pallet::config]  // <-- Step 2. 이 코드 블록은 이후에 대체됩니다.
     #[pallet::event]   // <-- Step 3. 이 코드 블록은 이후에 대체됩니다.
     #[pallet::error]   // <-- Step 4. 이 코드 블록은 이후에 대체됩니다.
     #[pallet::storage] // <-- Step 5. 이 코드 블록은 이후에 대체됩니다.
     #[pallet::call]    // <-- Step 6. 이 코드 블록은 이후에 대체됩니다.
   }
   ```

   이제 이벤트, 오류, 스토리지 및 호출 가능한 함수에 대한 자리 표시자가 포함된 프레임워크가 있습니다.

8. 변경 사항을 저장합니다.

## 팔레트가 이벤트를 발생하도록 구성

모든 팔레트에는 `Config`라는 [Rust "트레이트"](https://doc.rust-lang.org/book/ch10-02-traits.html)가 있습니다. 이 트레이트를 사용하여 팔레트에 필요한 특정 설정을 구성합니다.
이 튜토리얼에서는 팔레트가 이벤트를 발생시킬 수 있도록 구성 설정을 정의합니다.

존재 증명 팔레트의 `Config` 트레이트를 정의하려면 다음 단계를 수행합니다:

1. 텍스트 편집기에서 `pallets/template/src/lib.rs` 파일을 엽니다.

2. `#[pallet::config]` 줄을 다음 코드 블록으로 대체합니다:

   ```rust
   /// 팔레트를 구성하는 데 필요한 매개변수와 종속되는 유형을 지정하여 팔레트를 구성합니다.
   #[pallet::config]
   pub trait Config: frame_system::Config {
     /// 이 팔레트가 이벤트를 발생시키기 때문에 이 팔레트는 런타임의 이벤트 정의에 의존합니다.
     type RuntimeEvent: From<Event<Self>> + IsType<<Self as frame_system::Config>::RuntimeEvent>;
   }
   ```

3. 변경 사항을 저장합니다.

## 팔레트 이벤트 구현

팔레트가 이벤트를 발생시킬 수 있도록 구성했으므로 이제 해당 이벤트를 정의할 수 있습니다.
[애플리케이션 설계](#애플리케이션-설계)에서 설명한 대로, 존재 증명 팔레트는 다음 조건에서 이벤트를 발생시킵니다:

- 블록체인에 새로운 클레임이 추가될 때.
- 클레임이 소유자에 의해 취소될 때.

각 이벤트는 이벤트를 트리거한 사람을 나타내는 `AccountId`와 저장되거나 제거되는 존재 증명 클레임(`Hash`)을 표시합니다.

팔레트 이벤트를 구현하려면 다음 단계를 수행합니다:

1. 텍스트 편집기에서 `pallets/template/src/lib.rs` 파일을 엽니다.

2. `#[pallet::event]` 줄을 다음 코드 블록으로 대체합니다:

   ```rust
   // 팔레트는 중요한 변경 사항이 발생할 때 사용자에게 알리기 위해 이벤트를 사용합니다.
   // 이벤트 문서는 매개변수에 대한 설명적인 이름을 제공하는 배열로 끝나야 합니다.
   #[pallet::event]
   #[pallet::generate_deposit(pub(super) fn deposit_event)]
   pub enum Event<T: Config> {
     /// 클레임이 생성될 때 발생하는 이벤트입니다.
     ClaimCreated { who: T::AccountId, claim: T::Hash },
     /// 소유자에 의해 클레임이 취소될 때 발생하는 이벤트입니다.
     ClaimRevoked { who: T::AccountId, claim: T::Hash },
   }
   ```

3. 변경 사항을 저장합니다.

## 팔레트 오류 포함

정의한 이벤트는 팔레트 호출이 성공적으로 완료된 경우를 나타냅니다.
오류는 호출이 실패한 이유와 실패한 호출에 대한 정보를 나타냅니다.
이 튜토리얼에서는 다음과 같은 오류 조건을 정의합니다:

- 이미 클레임이 있는 상태에서 클레임을 추가하려고 할 때.

- 존재하지 않는 클레임을 취소하려고 할 때.

- 다른 계정이 소유한 클레임을 취소하려고 할 때.

존재 증명 팔레트의 오류를 구현하려면 다음 단계를 수행합니다:

1. 텍스트 편집기에서 `pallets/template/src/lib.rs` 파일을 엽니다.

2. `#[pallet::error]` 줄을 다음 코드 블록으로 대체합니다:

   ```rust
   #[pallet::error]
   pub enum Error<T> {
     /// 클레임이 이미 존재합니다.
     AlreadyClaimed,
     /// 클레임이 존재하지 않으므로 취소할 수 없습니다.
     NoSuchClaim,
     /// 클레임이 다른 계정에 소유되어 있으므로 호출자는 취소할 수 없습니다.
     NotClaimOwner,
   }
   ```

3. 변경 사항을 저장합니다.

## 저장된 항목에 대한 스토리지 맵 구현

존재 증명 팔레트에서 블록체인에 새로운 클레임을 추가하려면 스토리지 메커니즘이 필요합니다.
이 요구 사항을 해결하기 위해 각 클레임이 소유자와 클레임이 생성된 블록 번호를 가리키는 Key-Value 맵을 생성할 수 있습니다.
이러한 Key-Value 맵을 생성하기 위해 FRAME [`StorageMap`](https://paritytech.github.io/substrate/master/frame_support/pallet_prelude/struct.StorageMap.html)을 사용할 수 있습니다.

존재 증명 팔레트에 대한 스토리지를 구현하려면 다음 단계를 수행합니다:

1. 텍스트 편집기에서 `pallets/template/src/lib.rs` 파일을 엽니다.

2. `#[pallet::storage]` 줄을 다음 코드 블록으로 대체합니다:

   ```rust
   #[pallet::storage]
   pub(super) type Claims<T: Config> = StorageMap<_, Blake2_128Concat, T::Hash, (T::AccountId, T::BlockNumber)>;
   ```

3. 변경 사항을 저장합니다.

## 호출 가능한 함수 구현

존재 증명 팔레트는 사용자에게 다음과 같은 두 가지 호출 가능한 함수를 노출합니다:

- `create_claim()`은 사용자가 해시와 함께 파일의 존재를 주장할 수 있게 합니다.

- `revoke_claim()`은 클레임의 소유자가 클레임을 취소할 수 있게 합니다.

이러한 함수는 `StorageMap`을 사용하여 다음 논리를 구현합니다:

- 클레임이 이미 스토리지에 있는 경우 이미 소유자가 있으므로 클레임을 다시 주장할 수 없습니다.
- 클레임이 스토리지에 존재하지 않는 경우 클레임을 주장하고 스토리지에 기록할 수 있습니다.

존재 증명 팔레트에서 이 논리를 구현하려면 다음 단계를 수행합니다:

1. 텍스트 편집기에서 `pallets/template/src/lib.rs` 파일을 엽니다.

2. `#[pallet::call]` 줄을 다음 코드 블록으로 대체합니다. `revoke_claim` 함수를 직접 구현해 보세요. 함수 시그니처만 복사하고 내용은 복사하지 마세요. `Claims::<T>::get` 및 `Claims::<T>::remove`를 사용하여 클레임을 가져오거나 제거해야 합니다.

   ```rust
   // 호출 가능한 함수는 사용자가 팔레트와 상호 작용하고 상태 변경을 수행할 수 있게 합니다.
   // 이러한 함수는 종종 트랜잭션과 비교되는 "엑스트린식"으로 구현됩니다.
   // 호출 가능한 함수는 가중치로 주석이 달려 있어야 하며 DispatchResult를 반환해야 합니다.
   #[pallet::call]
   impl<T: Config> Pallet<T> {
     #[pallet::weight(0)]
     #[pallet::call_index(1)]
     pub fn create_claim(origin: OriginFor<T>, claim: T::Hash) -> DispatchResult {
       // 외부에서 서명된 extrinsic인지 확인하고 서명한 사람을 가져옵니다.
       // 이 함수는 extrinsic이 서명되지 않은 경우 오류를 반환합니다.
       let sender = ensure_signed(origin)?;

       // 지정된 클레임이 이미 저장되어 있는지 확인합니다.
       ensure!(!Claims::<T>::contains_key(&claim), Error::<T>::AlreadyClaimed);

       // FRAME System 팔레트에서 블록 번호를 가져옵니다.
       let current_block = <frame_system::Pallet<T>>::block_number();

       // 클레임을 발신자와 블록 번호와 함께 저장합니다.
       Claims::<T>::insert(&claim, (&sender, current_block));

       // 클레임이 생성되었다는 이벤트를 발생시킵니다.
       Self::deposit_event(Event::ClaimCreated { who: sender, claim });

       Ok(())
     }

     #[pallet::weight(0)]
     #[pallet::call_index(2)]
     pub fn revoke_claim(origin: OriginFor<T>, claim: T::Hash) -> DispatchResult {
       // 외부에서 서명된 extrinsic인지 확인하고 서명한 사람을 가져옵니다.
       // 이 함수는 extrinsic이 서명되지 않은 경우 오류를 반환합니다.
       let sender = ensure_signed(origin)?;

       // 클레임의 소유자를 가져옵니다. 클레임이 없으면 오류를 반환합니다.
       let (owner, _) = Claims::<T>::get(&claim).ok_or(Error::<T>::NoSuchClaim)?;

       // 현재 호출자가 클레임 소유자인지 확인합니다.
       ensure!(sender == owner, Error::<T>::NotClaimOwner);

       // 스토리지에서 클레임을 제거합니다.
       Claims::<T>::remove(&claim);

       // 클레임이 취소되었다는 이벤트를 발생시킵니다.
       Self::deposit_event(Event::ClaimRevoked { who: sender, claim });
       Ok(())
     }
   }
   ```

3. 변경 사항을 저장하고 파일을 닫습니다.

4. 다음 명령을 실행하여 코드가 컴파일되는지 확인합니다.

   ```bash
   cargo check -p node-template-runtime --release
   ```

   `[-p](https://doc.rust-lang.org/cargo/commands/cargo-check.html#options) node-template-runtime` 지시문은 cargo에게 `node_template_runtime` 패키지만 확인하도록 지시합니다.

   막힌 경우 노드 템플릿 [해결 방법](https://github.com/substrate-developer-hub/substrate-node-template/blob/tutorials/solutions/proof-of-existence/pallets/template/src/lib.rs)을 참조할 수 있습니다.

## 새 팔레트로 런타임 빌드

존재 증명 팔레트의 모든 부분을 `pallets/template/lib.rs` 파일로 복사한 후, 런타임을 컴파일하고 노드를 시작할 준비가 되었습니다.

업데이트된 Substrate 노드를 컴파일하고 시작하려면 다음 단계를 수행합니다:

1. 터미널 셸을 엽니다.

2. 노드 템플릿의 루트 디렉토리로 이동합니다.

3. 다음 명령을 실행하여 노드 템플릿을 컴파일합니다.

   ```bash
   cargo build --release
   ```

4. 다음 명령을 실행하여 업데이트된 Substrate 노드를 개발 모드로 시작합니다.

   ```bash
   ./target/release/node-template --dev
   ```

   `--dev` 옵션은 미리 정의된 `development` 체인 스펙을 사용하여 노드를 시작합니다.
   `--dev` 옵션을 사용하면 노드를 중지하고 다시 시작할 때마다 깨끗한 작업 상태를 얻을 수 있습니다.

5. 노드가 블록을 생성하는지 확인합니다.

## 블록체인과 상호 작용하기

사용자 정의 존재 증명 팔레트가 포함된 새로운 블록체인이 실행 중이므로, 블록체인과 상호 작용하여 기능이 예상대로 작동하는지 확인할 수 있습니다!

이를 위해 Substrate 기반 블록체인과 상호 작용할 수 있는 개발자 도구인 Polkadot JS Apps를 사용할 것입니다.

기본적으로 블록체인은 `ws://127.0.0.1:9944`에서 실행 중이므로 다음 링크를 사용하여 연결할 수 있습니다:

https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/

Substrate 블록체인이 실행 중이고 Polkadot JS Apps가 연결되어 있는 경우, 왼쪽 상단 모서리에서 블록 번호가 증가하는 것을 확인할 수 있어야 합니다:

![Polkadot JS Explorer](/media/images/docs/tutorials/custom-pallet/poe-explorer.png)

### 클레임 제출

프론트엔드를 사용하여 증명 증명 팔레트를 테스트하려면 다음 단계를 수행합니다:

1. ["개발자 > 엑스트린식"](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/extrinsics) 탭으로 이동합니다.

   ![엑스트린식 탭](/media/images/docs/tutorials/custom-pallet/poe-extrinsics-tab.png)

2. 엑스트린식 페이지를 조정하여 "ALICE"를 계정으로 선택하고 "templateModule > createClaim"을 엑스트린식으로 선택합니다.

   ![클레임 생성](/media/images/docs/tutorials/custom-pallet/poe-create-claim.png)

3. 그런 다음 "파일 해싱"을 토글하여 해싱할 파일을 선택할 수 있습니다.

   ![파일 해싱](/media/images/docs/tutorials/custom-pallet/poe-hash-file.png)

4. 오른쪽 하단의 "트랜잭션 제출"을 클릭한 다음 팝업에서 "서명 및 제출"을 클릭합니다.

   ![엑스트린식 제출](/media/images/docs/tutorials/custom-pallet/poe-submit.png)

5. 모든 것이 성공적으로 완료되었다면 초록색 엑스트린식 성공 알림이 표시됩니다!

   ![엑스트린식 성공](/media/images/docs/tutorials/custom-pallet/poe-success.png)

### 클레임 읽기

이 튜토리얼의 마지막 단계는 블록체인에 저장된 클레임을 확인하는 것입니다.

1. ["개발자 > 체인 상태"](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/chainstate) 탭으로 이동합니다.

   ![체인 상태](/media/images/docs/tutorials/custom-pallet/poe-chain-state.png)

1. 상태 쿼리를 "templateModule > claims"로 조정합니다.

1. 해시 입력의 "옵션 포함"을 해제하여 입력을 비워둡니다.

   이렇게 하면 한 번에 하나가 아닌 모든 클레임을 볼 수 있습니다.

   ![모든 클레임 쿼리](/media/images/docs/tutorials/custom-pallet/poe-claims.png)

1. "+" 버튼을 눌러 쿼리를 실행합니다.

   ![쿼리 결과](/media/images/docs/tutorials/custom-pallet/poe-query.png)

   이제 클레임이 블록체인에 저장되고 클레임이 생성된 블록 번호와 소유자 주소에 대한 데이터가 표시됩니다!

## 다음 단계로 넘어가기

이 튜토리얼에서는 다음과 같은 내용을 포함하여 새로운 사용자 정의 팔레트를 생성하는 기본 사항을 배웠습니다:

- 사용자 정의 팔레트에 이벤트, 오류, 스토리지 및 호출 가능한 함수를 추가하는 방법을 배웠습니다.

- 사용자 정의 팔레트를 런타임에 통합하는 방법을 배웠습니다.

- 사용자 정의 팔레트가 포함된 노드를 컴파일하고 시작하는 방법을 배웠습니다.

- 사용자 정의 블록체인과 상호 작용하기 위해 Polkadot JS Apps 개발자 도구를 사용하는 방법을 배웠습니다.

이 튜토리얼은 코드에 너무 깊이 들어가지 않고 기본 사항만 다루었습니다.
그러나 완전히 사용자 정의된 블록체인을 구축하기 위해 더 많은 작업을 수행할 수 있습니다.
사용자 정의 팔레트를 사용하면 블록체인이 지원해야 하는 기능을 노출할 수 있습니다.

존재 증명 체인에 대한 이해를 완성하기 위해 다음을 시도해 보세요:

- "ALICE" 또는 "BOB" 계정으로 동일한 파일을 다시 주장합니다.
  - 오류가 발생해야 합니다!
- "ALICE" 및/또는 "BOB" 계정으로 다른 파일을 주장합니다.
- 해당 클레임 소유자 계정으로 클레임을 취소합니다.
- 스토리지를 읽어 클레임 목록을 확인합니다.

사용자 정의 팔레트를 생성함으로써 가능한 작업에 대해 자세히 알아보려면
FRAME 문서와 [how-to 가이드](/reference/how-to-guides)를 참조하세요.