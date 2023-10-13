---
title: 경매와 크라우드론
description:
keywords:
  - 파라체인
  - 슬롯
  - 대출
  - 크라우드론
  - 경매
---

Polkadot이나 Kusama와 같은 _제품 네트워크_ 에 파라체인을 출시할 준비가 되었나요?
[파라체인 슬롯 경매](https://wiki.polkadot.network/docs/learn-auction)에서 슬롯을 획득해야 합니다.

> 제품 네트워크로 진행하기 전에 먼저 [Rococo 테스트넷](/tutorials/build-a-parachain/acquire-a-testnet-slot/)에서 모든 것을 시도해보세요!

대부분에게 슬롯을 위해 잠금된 DOT 또는 KSM의 총액은 상당히 크기 때문에, [크라우드론](https://wiki.polkadot.network/docs/learn-crowdloans)은 일반적으로 경매에 참여하기 위해 커뮤니티가 기여하는 풀에 사용됩니다.

위의 두 문서를 읽은 후에는 아래에서 개발자를 위한 크라우드론 및 입찰 구성 방법을 확인하세요.

### 파라스레드로 등록하기

모든 파라체인은 먼저 파라스레드로 등록해야 합니다.
다음과 같은 항목이 필요합니다:

- `ParaID`를 등록하기 위한 예치금 (네트워크에 따라 다름)

- 고유한 `ParaID`를 예약합니다. 이는 다음 사용 가능한 ID에 할당됩니다.
  이 정수는 [시스템 파라체인](https://wiki.polkadot.network/docs/learn-common-goods#system-level-chains)을 위해 `0-999`가 예약되고, [공공 유틸리티 파라체인](https://wiki.polkadot.network/docs/learn-common-goods#public-utility-chains)을 위해 `1000-1999`가 예약되므로 `2000`보다 큰 값이어야 합니다.

- 파라체인 genesis 상태입니다.
  [로컬 릴레이 체인 준비](/tutorials/build-a-parachain/prepare-a-local-relay-chain/)의 genesis 상태 내보내기 과정을 참조하세요.

- 파라체인 Wasm 런타임입니다.
  [로컬 릴레이 체인 준비](/tutorials/build-a-parachain/prepare-a-local-relay-chain/)의 Wasm 런타임 내보내기 과정을 참조하세요.

절차는 다음과 같습니다:

- **_올바른 릴레이 체인_** [여기](https://polkadot.js.org/apps/#/parachains/parathreads)에서 Polkadot-JS Apps 파라스레드 섹션으로 이동합니다.

- 다음 사용 가능한 `ParaID`를 예약합니다.

  ![paraid-reserve.png](/media/images/docs/tutorials/parachains/paraid-reserve.png)

- `ParaID`를 성공적으로 예약한 후, 이제 **파라스레드**로 등록할 수 있습니다.

  ![register-parathread.png](/media/images/docs/tutorials/parachains/register-parathread.png)

- extrinsic이 성공하면 `registrar.Registered` 이벤트가 발생하는 것을 볼 수 있습니다.

  ![parathread-register-success.png](/media/images/docs/tutorials/parachains/parathread-register-success.png)

- Polkadot-JS Apps의 [파라체인 -> 파라스레드](https://polkadot.js.org/apps/#/parachains/parathreads) 페이지에서 파라스레드 등록이 **Onboarding**되었음을 확인할 수 있습니다:

  ![parathread-onboarding.png](/media/images/docs/tutorials/parachains/parathread-onboarding.png)

extrinsic이 성공한 후, 체인이 파라스레드로 완전히 등록되기까지 [**2개의 세션**](#relevant-settings)이 소요됩니다.

### 파라체인 슬롯 경매

파라스레드는 파라체인 슬롯 경매에서 승리함으로써 파라체인으로 변환될 수 있습니다.
이로써 이제 파라체인은 슬롯 기간 동안 항상 블록 데이터가 해시되고 릴레이 체인 블록에 포함됩니다(검증의 증명, 또는 **PoV**로 불림).

**완전히 등록된 파라스레드만이 파라체인 슬롯 경매에 입찰할 자격이 있습니다.**
Common goods 체인은 경매를 우회하고 체인 내 거버넌스 결정을 통해 파라체인이 될 수 있습니다.

#### 관련 설정

> **일부 _중요한_ 설정은 네트워크에 따라 다릅니다! 이는 일부 파라체인 작업의 시간 동작에 영향을 미치며 변경될 수 있습니다!**

- 세션 길이
- 임대 기간 길이
- 종료 기간
- 현재 임대 기간 인덱스 = 현재 블록 번호 / 14400

파라체인/파라스레드의 상태 전이는 온보딩, 오프보딩, 업그레이드, 다운그레이드 등을 포함하여 적어도 2개의 세션이 소요됩니다.

이러한 설정은 대상 릴레이 체인에서 현재 실행 중인 [Polkadot 위키](https://wiki.polkadot.network/docs/learn-crowdloans#starting-a-crowdloan-campaign) 및 런타임 [소스](https://github.com/paritytech/polkadot-sdk/tree/master/polkadot/runtime)에서 찾을 수 있습니다.
이러한 매개변수를 확정하기 전에 이러한 매개변수를 이해하는 것이 _절대적으로 필수_ 입니다.

#### 입찰

완전히 등록된 파라스레드를 가진 누구나 자신의 `ParaID`를 위해 파라체인 슬롯을 얻기 위해 입찰할 수 있습니다.
슬롯 경매에 참여하는 다른 모든 사람들보다 높은 가격을 제시해야 합니다.

이를 위해 Polkadot-JS 앱 [Network -> Parachains -> Auctions](https://polkadot.js.org/apps/#/parachains/auctions) 페이지에서 수행할 수 있습니다(올바른 네트워크에 접속되어 있는지 확인하세요).

`ParaID`, 입찰하려는 금액 및 입찰하려는 슬롯을 선택하세요:

![parachain-bid.png](/media/images/docs/tutorials/parachains/parachain-bid.png)

### 크라우드론

크라우드론 슬롯 경매에서 승리하기 위해 커뮤니티의 지원을 활용할 수도 있습니다.
이 경우, 크라우드론 캠페인을 시작하여 지지자들이 경매에서 승리하기 위해 프로젝트에 토큰을 대출할 수 있도록 할 수 있습니다.
이러한 토큰은 경매 과정에서 보관되며, 승리한 경우 슬롯 임대 기간 동안 보관됩니다.

테스트넷 파라체인 슬롯에 대해서는 이 작업을 수행하지 않을 것이지만, 메인넷 파라체인 슬롯에 대해서는 이 옵션을 고려할 수 있습니다.

#### 크라우드론 캠페인 시작하기

다음에서는 크라우드론을 시작하기 위해 extrinsic을 제출할 준비를 합니다.

![parachain-bid.png](/media/images/docs/tutorials/parachains/parachain-crowdloan.png)

매개변수에 대한 주의 사항:

- `파라체인 ID`: 등록한 `ParaID`에 대해서만 크라우드론 캠페인을 생성할 수 있습니다.

- `크라우드펀드 상한`: 크라우드론 캠페인이 모을 수 있는 최대 금액입니다.
  이는 외부 지원을 받기 위한 책임 있는 조치입니다.

- `종료 블록`: 크라우드론을 종료하려는 시점입니다.
  3일 후에 경매가 시작되고 5일 동안 진행된다면, 크라우드론을 10일 후에 종료하고자 할 것입니다. 이렇게 하면 경매 과정 전체 동안 크라우드론이 활성 상태임을 확실하게 할 수 있습니다.
  반면에 크라우드론 기간을 너무 길게 설정하면 커뮤니티 자금의 잠금 기간을 불필요하게 연장시키고 참여를 저해할 수 있습니다.

- `첫 번째 기간` / `마지막 기간`: 입찰하려는 슬롯 기간입니다.
  현재 경매에서 3-6번 슬롯이 열려 있다면, 이 값은 그 사이의 어떤 숫자든 될 수 있습니다.

- 기여를 받지 않은 경우 크라우드론을 취소할 수 있습니다.

extrinsic이 성공하면 [Network -> Parachains -> Crowdloan 페이지](https://polkadot.js.org/apps/#/parachains/crowdloan)에서 새로운 크라우드론 항목을 확인할 수 있습니다:

![crowdloan-success.png](/media/images/docs/tutorials/parachains/crowdloan-success.png)

#### 크라우드론 캠페인 자금 조달하기

자금을 조달하려는 모든 계정은 자신의 캠페인을 지원할 수 있으며, 이는 이 캠페인을 시작한 계정과 동일한 계정도 포함됩니다.

위에서 언급한 [크라우드론 페이지](https://polkadot.js.org/apps/#/parachains/crowdloan)로 이동하여 지원하려는 캠페인에서 **+ Contribute**를 선택하세요.

다음과 유사한 팝업 extrinsic이 표시됩니다:

![crowdloan-contribute.png](/media/images/docs/tutorials/parachains/crowdloan-contribute.png)

지원하려는 금액을 입력하고 트랜잭션을 제출하세요.