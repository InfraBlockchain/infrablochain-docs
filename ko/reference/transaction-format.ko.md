---
title: 트랜잭션 형식
description: Substrate에서 서명된 및 서명되지 않은 트랜잭션의 형식을 설명합니다.
keywords:
---

이 문서에서는 Substrate에서 서명된 및 서명되지 않은 트랜잭션의 데이터 구조에 대해 자세히 설명합니다.
이는 트랜잭션 풀이 들어오는 트랜잭션을 확인하는 방법을 이해하는 데 특히 유용합니다.
파라체인 빌더는 트랜잭션이 어떻게 포맷되는지를 사용자 정의하는 데 유용하며, 선택한 형식을 준수해야 하는 클라이언트 애플리케이션을 작성하는 데도 유용합니다.

Extrinsic은 일반적으로 서명, 일부 유효성 검사를 통과한 Extrinsic인지 여부를 나타내는 데이터 및 해당 Extrinsic 대상으로 하는 팔렛 및 호출에 대한 참조를 포함합니다.
이 형식은 응용 프로그램이 Extrinsic의 요구 사항이 충족되고 올바르게 구성되었는지 확인할 수 있는 방법을 제공합니다.

- Unchecked: 트랜잭션 풀에서 수락되기 전에 일부 유효성 검사를 필요로 하는 서명된 트랜잭션입니다.
  확인되지 않은 Extrinsic에는 보내는 데이터에 대한 서명과 추가 데이터가 포함됩니다.
- Checked: 서명 검증이 필요하지 않은 고유한 Extrinsic입니다.
  대신, Extrinsic이 어디에서 왔는지와 추가 데이터를 포함합니다.
- Opaque: 아직 형식에 커밋되지 않은 Extrinsic에 사용됩니다. 그러나 여전히 디코딩할 수 있습니다.

추가 데이터는 트랜잭션이나 고유한 연결에 첨부할 수 있는 추가 정보입니다.
예를 들어, 트랜잭션의 nonce, 블록 작성자에 대한 팁 또는 Extrinsic의 유효 기간 등이 있습니다.
이 정보는 트랜잭션이 블록에 포함되기 전에 Extrinsic의 유효성과 순서를 결정하는 데 도움이 되는 [특수한 확장](#signed-extensions)에 의해 제공됩니다.

서명된 트랜잭션은 다음과 같이 구성될 수 있습니다:

```rust
node_runtime::UncheckedExtrinsic::new_signed(
		function.clone(),                                      // 일부 호출
		sp_runtime::AccountId32::from(sender.public()).into(), // 일부 보내는 계정
		node_runtime::Signature::Sr25519(signature.clone()),   // 계정의 서명
		extra.clone(),                                         // 서명된 확장
	)
```

## 트랜잭션 구성 방법

Substrate는 트랜잭션 형식을 일반적으로 정의하여 개발자가 유효한 트랜잭션을 정의하는 사용자 정의 방법을 구현할 수 있도록 합니다.
그러나 FRAME으로 빌드된 런타임에서는 (트랜잭션 버전 4를 가정할 때) 다음 인코딩된 데이터를 제출하여 트랜잭션을 구성해야 합니다:

`<서명 계정 ID> + <서명> + <추가 데이터>`

서명된 트랜잭션을 제출할 때 서명은 다음을 서명하여 구성됩니다:

- 팔렛 내의 함수 호출의 실제 호출로 구성됩니다:
  - 팔렛의 인덱스.
  - 팔렛 내의 함수 호출의 인덱스.
  - 대상 함수 호출에 필요한 매개변수.
- 서명된 확장의 확인된 추가 정보:
  - 이 트랜잭션의 Era는 어떻게 되는지, 즉 이 호출이 트랜잭션 풀에서 버려지기 전에 얼마 동안 유지되어야 하는지?
  - nonce, 즉 이 계정에서 이전 트랜잭션이 몇 번 발생했는지?
    이는 재생 공격이나 실수로 중복 제출을 방지하는 데 도움이 됩니다.
  - 이 트랜잭션을 블록에 포함시키기 위해 블록 생성자에게 지불되는 팁 금액.

그런 다음, 서명되지 않은 데이터에는 다음과 같은 추가 데이터가 필요합니다:

- 스펙 버전 및 트랜잭션 버전.
  이를 통해 트랜잭션이 호환되는 런타임에 제출되는지 확인됩니다.
- 제네시스 해시. 이를 통해 트랜잭션이 올바른 체인에 유효한지 확인됩니다.
- 블록 해시. 이는 체크포인트 블록의 해시에 해당하며, 서명된 확장에 의해 제공된 Era 정보의 블록 번호와 비교하여 트랜잭션이 잘못된 포크에서 실행되지 않도록 서명을 확인합니다.

SCALE 인코딩된 데이터는 (즉, (`call`, `extra`, `additional`)) 서명되며, 서명, 추가 데이터 및 호출 데이터가 올바른 순서로 첨부되고 SCALE로 인코딩되어 서명된 페이로드를 확인할 준비가 됩니다.
서명할 페이로드의 크기가 256바이트보다 큰 경우, 일정 크기 이상으로 서명된 데이터의 크기가 커지지 않도록 서명되기 직전에 해시됩니다.

이 프로세스는 다음 단계로 나눌 수 있습니다:

1. 서명되지 않은 페이로드를 구성합니다.
1. 서명 페이로드를 생성합니다.
1. 페이로드를 서명합니다.
1. 서명된 페이로드를 직렬화합니다.
1. 직렬화된 트랜잭션을 제출합니다.

Extrinsic은 다음과 같은 바이트 시퀀스로 인코딩되고 그 후에 16진수로 인코딩됩니다:

`[ 1 ] + [ 2 ] + [ 3 ] + [ 4 ]`

여기서:

- `[1]`은 다음 데이터의 압축 인코딩된 길이를 바이트로 포함합니다. [SCALE](/reference/scale-codec/)을 사용하여 압축 인코딩이 작동하는 방법을 알아보세요.
- `[2]`는 트랜잭션이 서명되었는지 여부를 나타내는 1비트와 인코딩된 트랜잭션 버전 ID(7비트)를 포함하는 `u8`입니다.
- `[3]`은 서명이 있는 경우 이 필드에는 계정 ID, SR25519 서명 및 일부 추가 데이터가 포함됩니다. 서명되지 않은 경우 이 필드에는 0바이트가 포함됩니다.
- `[4]`는 인코딩된 호출 데이터입니다. 이는 호출할 팔렛을 나타내는 1바이트, 해당 팔렛에서 수행할 호출을 나타내는 1바이트 및 해당 호출이 예상하는 인수를 인코딩하는 데 필요한 바이트 수로 구성됩니다.

응용 프로그램이 올바른 방식으로 트랜잭션을 구성하는 방법을 알 수 있는 방법은 [메타데이터 인터페이스](/build/application-development/#metadata-system)에서 제공됩니다.
메타데이터 유형과 트랜잭션 형식을 사용하여 응용 프로그램은 트랜잭션을 올바르게 인코딩하는 방법을 알 수 있습니다.
호출이 서명되지 않아야 하는 경우 `[2]`의 첫 번째 비트가 0이므로 응용 프로그램은 서명을 디코딩하지 않도록 알 수 있습니다.

**Polkadot JS Apps 예시:**

여기에서는 Bob에서 Dave로의 잔액 이체를 자세히 설명하고 수동으로 Extrinsic을 구성하고 제출합니다: Bob이 Dave에게 `42 UNIT`를 보냅니다.

1. `--dev` 모드로 [노드 템플릿](https://github.com/substrate-developer-hub/substrate-node-template)을 시작합니다(설정 방법은 [빠른 시작](/quick-start) 가이드를 참조하세요).
1. <https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9944#/extrinsics>로 이동합니다.
1. `Bob`을 보내는 사람으로 설정하고 `balances` 팔렛과 `transfer(dest, value)` 호출을 선택합니다.
1. `MultiAddress`를 `Id`로 설정하고 `dest`의 `AccountID`를 `Dave`로 설정합니다.
1. `value`를 `42000000000000`으로 설정합니다(이는 노드 템플릿의 [체인 스펙](/build/chain-spec/)에서 정의된 `42 UNIT`입니다).
1. `Submit Transaction` 버튼(오른쪽 하단)을 클릭하고 **_sign and submit_을 선택 취소**하여 기본 `nonce = 0` 및 `Lifetime = 64`로 서명된 트랜잭션을 생성하여 검사합니다.

![Bob에서 Dave로 `transfer` 42 unit](/media/images/docs/reference/apps-extrinsic-transfer-bob-to-dave-42.png)

- 인코딩된 호출 데이터: `0x050300306721211d5404bd9da88e0204360a1a9ab8b87c66c1bc2fcdd37f3c2222cc200b00a014e33226`
- 인코딩된 호출 해시: `0x26c333c22ec93ac431ee348168530b7d77e85d766f130af60890c0fd6ab20d5b`
- 결과 서명된 트랜잭션 호출 해시: `0x450284008eaf04151687736326c9fea17e25fc5287613693c912909cb226aa4794f26a48018eeaeb6a3496444c08b5c3e10e0c5f94776774591504ef4ef26e3873799831285a1a7cbd8ba2babe6fba94ea3585bf20e46c80ce7baeb25b149529ece931478c45020c00050000306721211d5404bd9da88e0204360a1a9ab8b87c66c1bc2fcdd37f3c2222cc200b00a014e33226`

![Bob에서 Dave로 `transfer` 42 unit 서명된 해시](/media/images/docs/reference/apps-extrinsic-transfer-bob-to-dave-42-signed-hash.png)

여기에서 `Signed transaction` 데이터를 복사하여 직접 RPC를 통해 제출하거나 `Developer` -> `Extrinsics` -> `Decode` 섹션에서 검사할 수 있습니다.
이제 이 창을 사용하여 트랜잭션을 제출하고 결과를 확인하겠습니다.

1. `authorize transaction` 카드를 닫습니다.
1. `Submit Transaction` 버튼(오른쪽 하단)을 클릭하고 _sign and submit_을 선택한 상태로 유지합니다.
1. `Developer` -> `RPC Calls` 탭으로 이동합니다.

RPC 탭에서 `author_submitAndWatchExtrinsic` 호출 결과를 다음과 유사한 형식으로 볼 수 있습니다:

```json
{
  dispatchInfo: {
    weight: 159,200,000
    class: Normal
    paysFee: Yes
  }
  events: [
    {
      phase: {
        ApplyExtrinsic: 1
      }
      event: {
        method: Withdraw
        section: balances
        index: 0x0508
        data: {
          who: 5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty
          amount: 125,000,147
        }
      }
      topics: []
    }
    {
      phase: {
        ApplyExtrinsic: 1
      }
      event: {
        method: NewAccount
        section: system
        index: 0x0003
        data: {
          account: 5DAAnrj7VHTznn2AWBemMuyBwZWs6FNFjdyVXUeYum3PTXFy
        }
      }
      topics: []
    }
    {
      phase: {
        ApplyExtrinsic: 1
      }
      event: {
        method: Endowed
        section: balances
        index: 0x0500
        data: {
          account: 5DAAnrj7VHTznn2AWBemMuyBwZWs6FNFjdyVXUeYum3PTXFy
          freeBalance: 42,000,000,000,000
        }
      }
      topics: []
    }
    {
      phase: {
        ApplyExtrinsic: 1
      }
      event: {
        method: Transfer
        section: balances
        index: 0x0502
        data: {
          from: 5FHneW46xGXgs5mUiveU4sbTyGBzmstUspZC92UhjJM694ty
          to: 5DAAnrj7VHTznn2AWBemMuyBwZWs6FNFjdyVXUeYum3PTXFy
          amount: 42,000,000,000,000
        }
      }
      topics: []
    }
    {
      phase: {
        ApplyExtrinsic: 1
      }
      event: {
        method: ExtrinsicSuccess
        section: system
        index: 0x0000
        data: {
          dispatchInfo: {
            weight: 159,200,000
            class: Normal
            paysFee: Yes
          }
        }
      }
      topics: []
    }
  ]
  status: {
    InBlock: 0x501c8f15883bb2b686fb5ea1ca35e99dace8bd6216bfc571a31d7088aea000f7
  }
}
```

1. `Network` -> `Explorer` 탭으로 이동합니다.
1. 카드의 오른쪽 상단에 있는 `<블록 번호>-<Extrinsic 번호>`를 클릭하여 `balances.Transfer` Extrinsic 세부 정보를 엽니다.
1. 트랜잭션의 체인 상 세부 정보를 검사합니다.

![Bob에서 Dave로 `transfer` 42 unit 결과](/media/images/docs/reference/apps-extrinsic-transfer-bob-to-dave-42-result.png)

위의 `Signed transaction` 데이터의 디코딩된 세부 정보를 열기 위해 `#/extrinsics/decode/0x....` 링크를 클릭하고 제출한 것과 동일한 것임을 알 수 있습니다.
따라서 Extrinsic을 제출하기 전이나 후에 이 도구를 사용하여 트랜잭션 호출 데이터를 디코딩하고 검사할 수 있습니다.

## 서명된 확장

Substrate는 [`SignedExtension`](https://paritytech.github.io/substrate/master/sp_runtime/traits/trait.SignedExtension.html) 트레이트에서 제공되는 추가 데이터로 Extrinsic을 확장하는 **서명된 확장(signed extensions)** 개념을 제공합니다.

트랜잭션 큐는 트랜잭션이 블록에 추가되기 전에 계속해서 서명된 확장을 호출하여 트랜잭션이 유효한지 확인합니다.
이는 트랜잭션이 블록에서 실패하지 않도록 하는 유용한 보호 장치입니다.
이들은 트랜잭션 풀을 스팸 및 재생 공격으로부터 보호하기 위해 유효성 검사 로직을 강제로 적용하는 데 자주 사용됩니다.

FRAME에서 기본적으로 서명된 확장은 다음 중 하나의 유형을 포함할 수 있습니다:

- `AccountId`: 보내는 사람의 식별 정보를 인코딩합니다.
- `Call`: 디스패치할 팔렛 호출을 인코딩합니다. 이 데이터는 트랜잭션 수수료를 계산하는 데 사용됩니다.
- `AdditionalSigned`: 서명된 페이로드에 추가 데이터를 처리합니다. 이를 통해 트랜잭션을 디스패치하기 전에 사용자 정의 로직을 추가할 수 있습니다.
- `Pre`: 호출이 디스패치되기 전에 전달할 수 있는 정보를 인코딩합니다.

FRAME의 [system 팔렛](https://paritytech.github.io/substrate/master/frame_system/)은 기본적으로 [유용한 `SignedExtensions`](https://paritytech.github.io/substrate/master/frame_system/index.html#signed-extensions) 세트를 제공합니다.

### 실제 예시

트랜잭션을 유효성 검사하는 데 중요한 서명된 확장은 `CheckSpecVersion`입니다.
이는 트랜잭션에 첨부된 서명된 페이로드로서 발신자가 스펙 버전을 제공할 수 있는 방법을 제공합니다.
스펙 버전은 이미 런타임에서 알려져 있으므로 서명된 확장은 스펙 버전이 일치하는지 간단한 확인을 수행할 수 있습니다.
일치하지 않으면 트랜잭션이 트랜잭션 풀에 넣기 전에 실패합니다.

다른 예로는 트랜잭션 우선 순위를 계산하는 데 사용되는 서명된 확장이 있습니다.
이들은 다음과 같습니다:

- `CheckWeight`: 모든 디스패치 클래스에 대해 우선 순위 값을 `0`으로 설정합니다.
- `ChargeTransactionPayment`: 전체 우선 순위를 계산하고 우선 순위 값을 이에 따라 수정합니다.

우선 순위는 디스패치 클래스와 보내는 사람이 지불할 의도로 하는 단위 가중치당 팁 또는 길이당 팁(더 제한적인 값)에 따라 달라집니다.
팁이 없는 트랜잭션은 우선 순위 계산을 위해 최소 팁 값인 `1`을 사용하여 모든 트랜잭션이 우선 순위 `0`을 가지지 않도록 합니다.
이로 인해 _더 작은_ 트랜잭션이 _더 큰_ 트랜잭션보다 우선되게 됩니다.

## 다음 단계로 넘어가기

트랜잭션이 어떻게 구성되는지 알았으므로, 트랜잭션이 트랜잭션 풀에서 런타임으로 진행되고 블록에 추가되는 과정이나 오프라인으로 트랜잭션을 제출하거나 REST API를 사용하여 트랜잭션을 제출하는 도구를 사용하는 방법을 검토하고 싶을 수 있습니다.

- [트랜잭션 라이프사이클](/learn/transaction-lifecycle/)
- [트랜잭션, 가중치 및 수수료](/build/tx-weights-fees/)
- 오프라인 트랜잭션을 위한 [tx-wrapper](/reference/command-line-tools/tx-wrapper)
- REST 기반 트랜잭션을 위한 [sidecar](/reference/command-line-tools/sidecar)