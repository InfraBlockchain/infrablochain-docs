---
title: 릴레이 체인에 연결하기
difficulty: 1
keywords:
  - 콜레이터
  - 파라체인
  - 파라스레드
  - 업그레이드
  - 커뮬러스
  - 스토리지
  - 마이그레이션
  - paraid
  - 등록
  - 론칭
---

탈중앙화된 릴레이 체인에 파라체인을 연결하는 방법에 대해 설명하는 가이드입니다.
이 가이드는 테스트 또는 프로덕션 네트워크에 파라체인을 배포하고 로컬 테스트 환경에서 유사한 단계를 이미 수행한 경우에 대한 빠른 참조용으로 제공됩니다.
이 가이드를 사용하기 전에 이 가이드 이전에 로컬 릴레이 체인을 설정하고 파라체인을 연결하는 단계를 따라야 합니다.

이 가이드는 다음과 같은 주제를 강조합니다:

- 파라체인에 대한 고유 식별자 획득 방법
- 파라체인 등록 방법
- 파라체인 슬롯 획득 방법

릴레이 체인에서 슬롯을 보호하기 전에 파라체인에 대한 몇 가지 정보를 준비하고 이 정보를 연결하려는 릴레이 체인에 제공해야 합니다.

릴레이 체인에 파라체인을 연결하기 위해 파라체인을 준비하려면 다음 단계를 성공적으로 완료해야 합니다:

- 연결하려는 특정 릴레이 체인에서 고유 식별자를 예약합니다.
- WebAssembly 런타임 바이너리를 컴파일합니다.
- 파라체인을 위한 체인 스펙에서 제네시스 상태를 생성합니다.

특정 릴레이 체인에 대한 파라체인을 등록하는 이러한 단계를 완료한 후에는 네트워크에 파라체인을 추가하기 위해 해당 릴레이 체인에서 슬롯을 요청할 수 있습니다.
테스트 목적으로는 Sudo 팔레트를 사용하여 릴레이 체인에서 슬롯을 얻을 수 있습니다.
프로덕션 네트워크에서는 경매 및 크라우드론을 통해 슬롯을 얻을 수 있습니다.
릴레이 체인에서 슬롯을 보호한 후에는 파라체인이 블록을 생성할 수 있습니다.

## 고유 식별자 예약

파라체인이나 파라스레드와 관련된 작업을 수행하기 전에 특정 릴레이 체인에 대한 고유 식별자가 있어야 합니다.
예를 들어, WebAssembly blob을 로드하거나 제네시스 상태를 등록하거나 다른 파라체인과의 메시징 채널을 생성하거나 크라우드론을 시작하려면 고유 식별자가 필요합니다.

식별자를 예약하려면 다음 단계를 수행하세요:

1. 웹 브라우저에서 [Polkadot/Substrate Portal](https://polkadot.js.org/apps/)을 열고 실행 중인 파라체인 노드의 WebSocket 포트에 연결합니다.

2. **Network**를 클릭하고 **Parachains**를 선택합니다.

3. **Parathreads**를 클릭한 다음 **+ParaID**를 클릭합니다.
   
   ![`ParaID` 예약](/media/images/docs/tutorials/parachains/paraid-reserve.png)

4. 연결하려는 릴레이 체인에 대한 **ParaID**를 예약하기 위해 트랜잭션을 제출합니다. 
   
   이 트랜잭션에는 예치금이 필요합니다.
   식별자를 예약하는 데 사용하는 계정은 트랜잭션에 대한 요금이 청구되며 해당 식별자와 관련된 파라스레드의 원본 계정이 됩니다.

   식별자는 등록 중인 릴레이 체인에서만 사용할 수 있으며 추가 단계를 완료하기 위해 식별자를 지정해야 합니다.

## 파라체인 스펙 사용자 정의

파라체인의 체인 스펙을 구성하여 예약한 식별자를 사용하도록 설정해야 합니다.
이 작업을 수행하려면 다음을 수행해야 합니다:

- 체인 스펙을 일반 텍스트 JSON 파일로 생성합니다.
- 텍스트 편집기에서 체인 스펙을 수정합니다.
- 수정된 체인 스펙을 원시(raw) 형식으로 생성합니다.

사용자 정의 체인 스펙을 생성하려면 다음 단계를 수행하세요:

1. 다음과 유사한 명령을 실행하여 일반 체인 스펙을 생성합니다:
   
   ```text
   ./target/release/parachain-template-node build-spec --disable-default-bootnode > rococo-local-parachain-plain.json
   ```

   이 예제의 명령은 `rococo-local`이 `node/chan_spec.rs` 파일에서 등록한 릴레이 체인을 가정합니다.

2. 텍스트 편집기에서 일반 텍스트 체인 스펙 파일을 열고 `ParaID`를 이전에 예약한 `ParaID`로 설정합니다.
   
   예를 들어, `rococo-local-parachain-plain.json` 파일을 열고 두 개의 필드를 수정합니다:
   
   ```text
  // --snip--
  "para_id": <your-reserved-identifier> ,
  // --snip--
      "parachainInfo": {
        "parachainId": <your-reserved-identifier> 
      },
  // --snip--
  ```

3. 다음과 유사한 명령을 실행하여 수정된 체인 스펙 파일에서 원시(raw) 체인 스펙을 생성합니다:
   
   ```text
   ./target/release/parachain-template-node build-spec \
     --chain rococo-local-parachain-plain.json \
     --raw \
     --disable-default-bootnode > rococo-local-parachain-raw.json
     ```

## 원시(raw) 체인 스펙 저장 및 배포

다른 사람들이 네트워크에 연결할 수 있도록 하려면 네트워크에 대한 결정론적 런타임 빌드를 빌드하고 배포해야 합니다.
결정론적 런타임을 빌드하는 방법에 대한 자세한 내용은 [결정론적 런타임 빌드](/build/build-a-deterministic-runtime/)를 참조하십시오.

일반적으로 체인 스펙은 노드의 코드베이스에 게시되는 `/chain-specs` 폴더에 있습니다.
예를 들어:

- Polkadot은 [node/service/chain-specs](https://github.com/paritytech/polkadot-sdk/tree/master/polkadot/node/service/chain-specs)에서 이러한 **릴레이 체인** 체인 스펙을 포함합니다.
- Cumulus는 [chain-specs](https://github.com/paritytech/polkadot-sdk/tree/master/cumulus/polkadot-parachain/src/chain_spec)에서 이러한 **파라체인** 체인 스펙을 포함합니다.

계속하기 전에 원시(raw) 체인 스펙을 소스에 커밋하는 것이 좋은 관행입니다.

## WebAssembly 런타임 유효성 검사 함수 획득

릴레이 체인은 파라체인 블록을 유효성 검사하기 위해 파라체인에 특화된 런타임 유효성 검사 로직도 필요합니다.
파라체인 콜레이터 노드에서 `export-genesis-wasm` 명령을 실행하여 이 WebAssembly blob을 생성할 수 있습니다.
예를 들어:

```bash
./target/release/parachain-template-node export-genesis-wasm --chain rococo-local-parachain-raw.json > para-wasm
```

## 파라체인 제네시스 상태 생성

파라체인을 등록하려면 릴레이 체인이 파라체인에 대한 [제네시스 상태](/build/chain-spec#the-genesis-state)를 알아야 합니다.
파라체인 콜레이터 노드에서 `export-genesis-state` 명령을 실행하여 파라체인에 대한 16진수로 인코딩된 제네시스 상태를 생성할 수 있습니다.
예를 들어:

```bash
./target/release/parachain-template-node export-genesis-state --chain parachain-raw.json > para-genesis
```

## 콜레이터 시작

이제 다음과 유사한 명령으로 콜레이터 노드를 시작할 준비가 되었습니다:

```text
parachain-collator \
--alice \
--collator \
--force-authoring \
--chain parachain-raw.json \
--base-path /tmp/parachain/alice \
--port 40333 \
--rpc-port 8844 \
-- \
--execution wasm \
--chain <relay-chain-spec-json> \
--port 30343 \
--rpc-port 9977
```

이 명령에서 `--` 인자 이전에 전달된 인수는 파라체인 콜레이터를 위한 것입니다.
`--` 이후의 인수는 내장된 릴레이 체인 노드를 위한 것입니다.
릴레이 체인 스펙은 등록한 릴레이 체인의 체인 스펙이어야 함에 유의하세요. 

콜레이터가 실행되고 릴레이 체인 노드와 피어링을 시작해야 합니다.
하지만 파라체인은 아직 파라체인 블록을 생성하지 않을 것입니다.
콜레이터가 릴레이 체인에 등록된 후에 블록 생성이 시작됩니다.

## 파라체인 등록

대상 릴레이 체인 및 해당 권한에 따라 등록하는 옵션이 있습니다.
테스트를 위해 Sudo 트랜잭션을 사용하거나 프로덕션을 위해 [경매](https://wiki.polkadot.network/docs/learn-auction) 및 [크라우드론](https://wiki.polkadot.network/docs/learn-crowdloans)을 사용할 수 있습니다.

이 가이드에서는 Sudo 트랜잭션 사용만 다룹니다.

### 등록 예치금 계산

옵션으로 Polkadot 런타임에 대한 예치금 계산 정확한 공식을 계산할 수 있습니다:

```rust
pub const fn deposit(items: u32, bytes: u32) -> Balance {}
```

Polkadot 저장소에서 각 릴레이 체인에 대한 이 함수를 찾을 수 있습니다.
예를 들어:

- [Kusama](https://github.com/paritytech/polkadot-sdk/blob/master/polkadot/runtime/kusama/constants/src/lib.rs)
- [Polkadot](https://github.com/paritytech/polkadot-sdk/blob/master/polkadot/runtime/polkadot/constants/src/lib.rs)
- [Rococo](https://github.com/paritytech/polkadot-sdk/blob/master/polkadot/runtime/rococo/constants/src/lib.rs)
- [Westend](https://github.com/paritytech/polkadot-sdk/blob/master/polkadot/runtime/westend/constants/src/lib.rs)

### Sudo를 사용하여 등록

`sudo` 호출을 사용하여 릴레이 체인에 파라체인을 등록합니다.

1. 브라우저에서 [Polkadot/Substrate Portal](https://polkadot.js.org/apps/#/explorer)을 열고 대상 릴레이 체인에 연결합니다.

2. **Developer**를 클릭하고 **Sudo**를 선택합니다.

5. **paraSudoWrapper**를 선택한 다음 **sudoScheduleParaInitialize(id, genesis)**를 선택하여 다음 릴레이 체인 세션 시작 시 예약한 식별자를 초기화합니다.

   트랜잭션 매개변수에 대해 다음을 수행합니다:

   - `id`: 예약한 `ParaId`를 입력합니다. 

   - `genesisHead`: **파일 업로드**를 클릭하고 파라체인을 위해 내보낸 제네시스 상태를 업로드합니다. 
     이 튜토리얼에서는 `para-2000-genesis` 파일을 선택합니다.
  
   - `validationCode`: **파일 업로드**를 클릭하고 파라체인을 위해 내보낸 WebAssembly 런타임을 업로드합니다.
     이 튜토리얼에서는 `para-2000-wasm` 파일을 선택합니다.
     
   - `paraKind`: **예**를 선택합니다.
  
   ![등록을 위한 매개변수 설정](/media/images/docs/tutorials/parachains/register-with-sudo.png)

1. 트랜잭션이 성공적으로 완료되었는지 확인하기 위해 **Network**를 클릭하고 **Explorer**를 선택하여 최근 이벤트 목록에서 `sudo.Sudid` _및_ `paras.PvfCheckAccepted` 이벤트를 확인합니다.

### 슬롯 임대를 사용하여 등록

슬롯 임대를 강제로 하려면 Sudo 트랜잭션을 사용할 수도 있습니다.

슬롯 임대를 강제로 하려면 다음을 수행합니다:

1. 브라우저에서 [Polkadot/Substrate Portal](https://polkadot.js.org/apps/#/explorer)을 열고 대상 릴레이 체인에 연결합니다.

2. **Developer**를 클릭하고 **Sudo**를 선택합니다.

3. **slots**를 선택한 다음 **forceLease(para, leaser, amount, period_begin, period_end)**를 선택합니다.
   
   ![슬롯 임대를 위한 Sudo 트랜잭션](/media/images/docs/tutorials/parachains/forceLease.png)

   트랜잭션 매개변수에 대해 다음을 수행합니다:
   
   - `period_begin`: 시작할 슬롯을 지정합니다. 예를 들어, 테스트 환경에서 활성 슬롯 `0`을 선택합니다.
  
   - `period_end`: 파라체인 테스트에 할당한 시간을 초과하는 슬롯을 지정합니다.
     온보딩 및 오프보딩을 테스트하려면 간격이 있는 슬롯 임대를 선택해야 합니다.
    
   파라체인이 온보딩되고 블록 생성이 시작되면 파라체인에 대한 다음과 유사한 정보가 표시됩니다:
   
   ![파라체인에 대한 정보 보기](/media/images/docs/tutorials/parachains/parachain-active-lease.png)

## 블록 생성 및 최종화

등록이 성공적으로 완료되고 새로운 릴레이 체인 에포크가 시작되면 콜레이터는 파라체인 블록 생성을 시작해야 합니다.

> 새로운 에포크가 시작되기까지 시간이 걸릴 수 있습니다.

Polkadot/substrate Portal에서 **Network**를 클릭하고 **Parachains**를 선택하여 등록된 파라체인 및 해당 최신 헤드 데이터를 추적할 수 있습니다.

## 예제

- [로컬 릴레이 체인 준비](/tutorials/build-a-parachain/prepare-a-local-relay-chain)
- [로컬 파라체인 연결](/tutorials/build-a-parachain/connect-a-local-parachain/)
- [테스트넷 슬롯 획득](/tutorials/build-a-parachain/acquire-a-testnet-slot/)