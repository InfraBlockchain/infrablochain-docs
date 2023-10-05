제목: 솔로 체인을 변환하기
설명:
키워드:
  - 콜레이터
  - 파라체인
  - 파라스레드
  - 업그레이드
  - 커뮬러스
  - 스토리지
  - 마이그레이션
  - 파라이드
  - 등록
  - 론칭
  - 변환

이 안내서는 솔로 체인에 Cumulus를 추가하고 릴레이 체인을 사용하여 최종성을 제공하는 방법을 설명합니다.

다음을 배울 수 있습니다:

- 솔로 체인의 보안 부트스트랩
- 공통 릴레이 체인에 있는 파라체인과 XCMP 액세스

Cumulus의 통합을 통해 Substrate 체인의 최종성을 Polkadot과 같은 릴레이 체인과 결합할 수 있습니다.
이 안내서는 _실행 중인 솔로 체인_을 마이그레이션하는 방법을 알려주지 않으며, 다른 Substrate 솔로 체인에서 일반적으로 사용되는 GRANDPA 대신 Cumulus를 사용하여 합의를 위한 노드의 _코드베이스_를 변환하는 데 필요한 단계만을 설명합니다.

## 파라체인과 솔로 체인 노드 템플릿

Substrate 파라체인 템플릿은 Substrate 노드 템플릿과 유사합니다.
두 템플릿은 `node`, `runtime`, `pallets` 디렉토리와 많은 사전 정의된 팔렛과 트레이트를 포함합니다.
두 템플릿을 모두 사용하여 대부분의 [튜토리얼](/tutorials/)을 따를 수 있습니다.
그러나 노드와 파라체인 템플릿 사이에는 몇 가지 중요한 차이점이 있으므로 처음부터 주의해야 합니다.

### 파라체인 정보 팔렛

기본적으로 파라체인 템플릿 [런타임](https://github.com/substrate-developer-hub/substrate-parachain-template/blob/main/runtime/Cargo.toml)에는 [`parachain-info` 팔렛](https://paritytech.github.io/cumulus/parachain_info/pallet/index.html)을 포함한 여러 파라체인 특정 팔렛이 포함되어 있습니다.
이 팔렛은 고유한 파라체인 식별자를 파라체인 런타임에 주입하는 데 사용됩니다.
이 정보를 통해 런타임은 어떤 크로스 체인 메시지가 해당 파라체인을 위해 의도된 것인지를 알 수 있습니다.

### 블록 유효성 검사 매크로

각 파라체인은 해당 파라체인이 등록된 릴레이 체인에게 웹어셈블리 블롭 형태의 `validate_block` 함수를 제공해야 합니다.
이 함수는 파라체인에만 필요하므로 기본적으로 노드 템플릿에 포함되어 있지 않습니다.
그러나 파라체인 템플릿은 다음 코드를 런타임 로직의 맨 아래에 추가하여 Substrate 런타임을 위한 이 함수를 생성합니다:

```rust
cumulus_pallet_parachain_system::register_validate_block!(
  Runtime = Runtime,
  BlockExecutor = cumulus_pallet_aura_ext::BlockExecutor::<Runtime, Executive>,
  CheckInherents = CheckInherents,
);
```

### 최종성은 릴레이 체인에 의존합니다

파라체인 템플릿에는 블록 최종화 메커니즘이 포함되어 있지 않습니다. 이는 파라체인이 릴레이 체인에서 제공하는 최종성을 사용하기 위해 고안되었기 때문입니다.
릴레이 체인 최종화는 Polkadot 및 기타 릴레이 체인의 아키텍처에서 기본 개념입니다.
반면 Substrate 노드 템플릿 및 다른 많은 Substrate 기반 체인은 자체 블록 최종화 메커니즘을 구현하며, 일반적으로 GRANDPA 팔렛과 관련 API를 사용합니다.

### 콜레이터 서비스

콜레이터 서비스인 [`node/src/service.rs`](https://github.com/substrate-developer-hub/substrate-parachain-template/blob/main/node/src/service.rs)는 노드 템플릿의 동일한 이름을 가진 [`node/src/service.rs`](https://github.com/substrate-developer-hub/substrate-node-template/blob/main/node/src/service.rs)와 완전히 다릅니다.
콜레이터 서비스는 표준 Substrate 노드에서 필요하지 않은 파라체인 특정 작업을 제공하기 위한 래퍼로 명시적으로 설계되었습니다.

기존의 Substrate 체인을 파라체인으로 변환하려는 경우, 시작점으로 [`node/src/service.rs`](https://github.com/substrate-developer-hub/substrate-parachain-template/blob/main/node/src/service.rs)를 파라체인 템플릿에서 복사해야 합니다.