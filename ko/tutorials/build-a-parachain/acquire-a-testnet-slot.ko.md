---
title: 테스트넷 슬롯 획득하기
description: 파라체인을 위한 공개 테스트 네트워크에서 슬롯을 획득하는 방법을 안내합니다.
keywords:
  - rococo
  - 테스트넷
  - faucet
  - 릴레이 체인
---

이 튜토리얼은 [Rococo](https://wiki.polkadot.network/docs/build-pdk#rococo-testnet) 테스트넷과 같은 공개 테스트 네트워크에서 파라체인을 배포하는 방법을 보여줍니다.
공개 테스트 네트워크는 개인 네트워크보다 진입 장벽이 높지만, 파라체인 프로젝트를 프로덕션 네트워크로 이동하기 위한 중요한 단계입니다.

## 시작하기 전에

Rococo는 공유 테스트 네트워크이므로, 이 튜토리얼에서는 로컬 테스트와는 다른 추가 단계가 필요합니다.
시작하기 전에 다음을 확인하세요:

- [신뢰할 수 있는 노드 추가](/tutorials/build-a-blockchain/add-trusted-nodes/)에서 설명하는 대로 체인 사양 파일을 생성하고 수정하는 방법을 알고 있어야 합니다.

- [신뢰할 수 있는 노드 추가](/tutorials/build-a-blockchain/add-trusted-nodes/)에서 설명하는 대로 키를 생성하고 저장하는 방법을 알고 있어야 합니다.

- 로컬 컴퓨터에서 [로컬 릴레이 체인 준비](/tutorials/build-a-parachain/prepare-a-local-relay-chain/) 및 [로컬 파라체인 연결](/tutorials/build-a-parachain/connect-a-local-parachain/) 튜토리얼을 완료했어야 합니다.

## 계정 및 토큰 준비하기

Rococo에서 어떤 작업을 수행하려면 ROC 토큰이 필요하며, 토큰을 저장하기 위해서는 Substrate 호환 디지털 화폐 지갑에 액세스해야 합니다.
개발용 키와 계정은 공개 환경에서는 사용할 수 없습니다.
하드웨어 지갑과 브라우저 기반 애플리케이션을 포함해 다양한 디지털 화폐 보관 옵션이 있으며, 일부는 다른 것보다 신뢰성이 높을 수 있습니다.
선택하기 전에 직접 조사를 해야 합니다.

그러나 [Polkadot/Substrate 포털](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frococo-rpc.polkadot.io#/explorer)을 사용하여 테스트 목적으로 시작할 수 있습니다.

계정을 준비하려면 다음을 수행하세요:

1. [Polkadot/Substrate 포털](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frococo-rpc.polkadot.io#/explorer)을 Rococo 네트워크에 연결합니다.

2. **Accounts**를 클릭하고 선택합니다.

3. **Add Account**를 클릭합니다.

4. 비밀 시드 구문을 복사하여 안전한 위치에 보관한 다음 **Next**를 클릭합니다.

5. 계정 이름과 비밀번호를 입력한 다음 **Next**를 클릭합니다.

6. **Save**를 클릭합니다.

7. [Rococo Element 채널](https://matrix.to/#/#rococo-faucet:matrix.org)에 가입하고, `!drip`와 Rococo의 공개 주소를 포함한 메시지를 보내서 지갑에 100 ROC을 받습니다.

   예를 들어, 다음과 유사한 메시지를 보냅니다:

   ```text
   !drip 5CVYesFxbDBU5rkZXYTAA6BnADbCoSpQkvexBQZvbtvyGTP1
   ```

## 파라체인 식별자 예약하기

Rococo에서 파라스레드로 등록하기 전에 파라체인 식별자를 예약해야 합니다.
이 단계는 로컬 릴레이 체인에서 식별자를 예약하는 [로컬 파라체인 연결](/tutorials/build-a-parachain/connect-a-local-parachain/) 튜토리얼과 유사합니다.
하지만, 공개 테스트넷에서는 다음으로 사용 가능한 식별자가 할당됩니다.

식별자를 예약하려면 다음을 수행하세요:

1. [Polkadot/Substrate 포털](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frococo-rpc.polkadot.io#/explorer)을 Rococo 네트워크에 연결합니다.

2. **Network**를 클릭하고 **Parachains**를 선택합니다. 
   
   ![네트워크 메뉴에서 Parachains 선택](/media/images/docs/tutorials/parachains/rococo-select-parachains.png)
   
3. **Parathreads**를 클릭한 다음 **ParaId**를 클릭합니다.

4. 트랜잭션을 검토하고 할당된 파라체인 식별자를 확인한 다음 **Submit**을 클릭합니다.
   
5. 신원을 인증하기 위해 비밀번호를 입력한 다음 **Sign and Submit**을 클릭합니다.
   
   ![트랜잭션 인증](/media/images/docs/tutorials/parachains/auth-rococo-id.png)

6. **Network**를 클릭하고 **Explorer**를 선택하여 최근 `registrar.Reserved` 이벤트 목록을 확인합니다.
   
   ![Rococo 네트워크에 대한 예약된 식별자](/media/images/docs/tutorials/parachains/rococo-reserved-id-event.png)

## 체인 사양 파일 수정하기

파라체인을 등록하기 위해 필요한 파일은 연결할 올바른 릴레이 체인과 할당된 파라체인 식별자를 지정해야 합니다.
이러한 변경 사항을 반영하기 위해 파라체인을 위한 체인 사양 파일을 빌드하고 수정해야 합니다.
이 튜토리얼에서는 릴레이 체인을 `rococo-local` 대신 `rococo`로, 파라체인 식별자를 `4105`로 설정합니다.

체인 사양을 수정하려면 다음을 수행하세요:

1. 다음 명령을 실행하여 파라체인 템플릿 노드를 위한 일반 텍스트 체인 사양을 생성합니다:
   
   ```bash
   ./target/release/parachain-template-node build-spec --disable-default-bootnode > plain-parachain-chainspec.json
   ```

2. 텍스트 편집기에서 파라체인 템플릿 노드를 위한 일반 텍스트 체인 사양을 엽니다.

3. `relay-chain`을 `rococo`로, `para_id`를 할당된 식별자로 설정합니다.
   
   예를 들어, 할당된 식별자가 `4105`인 경우 `para_id` 필드를 `4105`로 설정합니다:
   
   ```json
   ...
   "relay_chain": "rococo",
   "para_id": 4105,
   "codeSubstitutes": {},
   "genesis": {
      ...
   }
   ```
   
4. 이전에 예약한 파라체인 식별자를 `parachainId`로 설정합니다.
      
   ```json
   ...
      "parachainSystem": null,
      "parachainInfo": {
        "parachainId": 4105
      },
   ...
   ```

5. 계정의 공개 키를 세션 키 섹션에 추가합니다. 각 구성된 세션 키는 실행 중인 콜레이터가 필요합니다.

   ```json
   ...
      "session": {
        "keys": [
         [
           "5CVYesFxbDBU5rkZXYTAA6BnADbCoSpQkvexBQZvbtvyGTP1",
           "5CVYesFxbDBU5rkZXYTAA6BnADbCoSpQkvexBQZvbtvyGTP1",
           {
            "aura": "5CVYesFxbDBU5rkZXYTAA6BnADbCoSpQkvexBQZvbtvyGTP1"
           }
         ],
        ]
      }
    ...
    ```

6. 변경 사항을 저장하고 일반 텍스트 체인 사양 파일을 닫습니다.
   
7. 다음 명령을 실행하여 수정된 체인 사양 파일로부터 원시 체인 사양 파일을 생성합니다:
   
   ```bash
   ./target/release/parachain-template-node build-spec --chain plain-parachain-chainspec.json --disable-default-bootnode --raw > raw-parachain-chainspec.json
   ```

## 필요한 파일 내보내기

등록할 파라체인 콜레이터를 준비하기 위해 다음을 수행합니다:

1. 다음과 유사한 명령을 실행하여 파라체인을 위한 WebAssembly 런타임을 내보냅니다:

   ```bash
   ./target/release/parachain-template-node export-genesis-wasm --chain raw-parachain-chainspec.json para-4105-wasm
   ```

2. 다음과 유사한 명령을 실행하여 파라체인 제네시스 상태를 내보냅니다:
   
   ```bash
   ./target/release/parachain-template-node export-genesis-state --chain raw-parachain-chainspec.json para-4105-genesis-state
   ```

## 콜레이터 노드 시작하기

파라체인 노드의 포트는 릴레이 체인 노드와 피어링하기 위해 공개적으로 접근 가능하고 검색 가능해야 합니다.
`--port` 커맨드 라인 옵션을 사용하여 사용할 포트를 지정할 수 있습니다.
예를 들어, 다음과 유사한 명령을 사용하여 콜레이터를 시작할 수 있습니다:

```bash
./target/release/parachain-template-node --collator \
  --chain raw-parachain-chainspec.json \
  --base-path /tmp/parachain/pubs-demo \
  --port 50333 \
  --rpc-port 8855 \
  -- \
  --execution wasm \
  --chain rococo \
  --port 50343 \
  --rpc-port 9988
  ```

이 예제에서 첫 번째 `--port` 설정은 콜레이터 노드의 포트를 지정하고, 두 번째 `--port`는 내장된 릴레이 체인 노드에 대한 포트를 지정합니다.
첫 번째 `--rpc-port` 설정은 Polkadot-JS API 호출이나 Polkadot/Substrate 포털 애플리케이션을 사용하여 콜레이터에 연결할 수 있는 포트를 지정합니다.
두 번째 `--rpc-port`는 Polkadot-JS API나 Polkadot/Substrate 포털 애플리케이션을 사용하여 내장된 릴레이 체인에 연결할 수 있는 포트를 지정합니다.

## 파라스레드로 등록하기

파라체인으로 슬롯을 임대하기 전에 Rococo에서 파라스레드로 등록해야 합니다.

파라스레드로 등록하려면 다음을 수행하세요:

1. [Polkadot/Substrate 포털](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frococo-rpc.polkadot.io#/explorer)을 Rococo 네트워크에 연결합니다.

2. **Network**를 클릭하고 **Parachains**를 선택합니다. 

3. **Parathread**를 클릭한 다음 **ParaThread**를 클릭합니다.

   ![등록하려면 ParaThread를 클릭합니다](/media/images/docs/tutorials/parachains/rococo-parathread.png)

4. 파라체인 소유자와 파라체인 식별자를 확인하고, WebAssembly 유효성 검사 함수와 파라체인 제네시스 상태를 포함한 파일을 업로드한 다음 **Submit**을 클릭합니다.

5. 신원을 인증하기 위해 비밀번호를 입력한 다음 **Sign and Submit**을 클릭합니다.

6. 파라스레드 등록이 **Onboarding** 상태인지 Parathreads 목록에서 확인합니다:
   
   ![파라스레드 온보딩](/media/images/docs/tutorials/parachains/rococo-onboarding.png)

   또한, Network Explorer에서 `registrar.Registered` 이벤트를 확인하여 등록 요청을 확인할 수 있습니다.
      
   등록 요청을 제출한 후, 파라스레드는 두 개의 [세션](https://wiki.polkadot.network/docs/glossary#session)을 거쳐 온보딩을 완료합니다.

## 파라체인 슬롯 요청하기

파라체인이 파라스레드로 활성화된 후, 관련 프로젝트 팀은 Rococo에서 **영구적인** 또는 **임시적인 파라체인 슬롯**을 요청해야 합니다.

- **영구적인 슬롯**은 일반적으로 슬롯 임대 경매를 성공적으로 완료하고 Polkadot에 슬롯을 배포한 팀에게 할당됩니다.
  
  영구적인 슬롯을 통해 해당 팀은 최신 Polkadot 기능과의 호환성을 테스트하기 위해 라이브 공개 환경에서 코드베이스를 계속해서 테스트할 수 있습니다.
  영구적인 슬롯은 제한된 수만 사용할 수 있습니다.

- **임시적인 슬롯**은 연속적인 라운드 로빈 방식으로 동적으로 할당되는 파라체인 슬롯입니다.
  
  임대 기간의 시작마다, 릴레이 체인 구성에서 정의한 최대치까지 일정 수의 파라스레드가 자동으로 일정 기간 동안 파라체인으로 업그레이드됩니다.
  종료된 임대 기간 동안 활성화된 파라체인은 다음 기간을 위해 자동으로 파라스레드로 다운그레이드되어 슬롯을 다른 파라체인이 사용할 수 있도록 합니다.
  동적 할당을 통한 임시적인 슬롯은 Polkadot에 파라체인 슬롯이 없는 팀이 더 자주 런타임을 실제 네트워크 환경에서 테스트할 수 있도록 합니다.

### 슬롯 요청 제출하기
  
Rococo 런타임은 슬롯을 할당하기 위해 `sudo` 액세스가 필요합니다.
예를 들어, Rococo 런타임은 슬롯을 할당하기 위해 사용되는 계정이 최상위 권한을 갖도록 지정합니다:

```text
AssignSlotOrigin = EnsureRoot<Self::AccountId>;
```

슬롯 할당은 최종적으로 Rococo 거버넌스를 통해 커뮤니티 기반으로 이루어질 예정입니다.
하지만, 현재 Rococo `sudo` 키는 Parity Technologies에서 제어하고 있습니다.
따라서 슬롯 할당을 받기 위해 [Rococo 슬롯 요청](https://github.com/paritytech/subport/issues/new?assignees=&labels=Rococo&template=rococo.yaml)을 제출해야 합니다.
슬롯이 할당되면 알림을 받고 연결할 준비가 됩니다.

### 관리 계정을 사용하여 슬롯 할당하기 

`AssignSlotOrigin` 원본을 갖는 계정이 있는 경우, 해당 계정을 사용하여 Rococo 네트워크에서 임시적인 슬롯을 할당할 수 있습니다.

임시적인 슬롯을 할당하려면 다음을 수행하세요:

1. [Polkadot/Substrate 포털](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frococo-rpc.polkadot.io#/explorer)을 Rococo 네트워크에 연결합니다.

2. **Developer**를 클릭하고 [**Extrinsics**](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frococo-rpc.polkadot.io#/extrinsics)를 선택합니다.
   
3. 트랜잭션을 제출할 계정을 선택합니다.
   
4. `assignedSlots` 팔레트를 선택합니다.

5. `assignTempParachainSlot` 함수를 선택합니다.
   
6. 할당된 파라체인 식별자를 입력합니다.
   
7. `LeasePeriodStart`에 `Current`를 선택합니다.
   
   현재 슬롯이 가득 찬 경우, 다음으로 사용 가능한 슬롯이 할당됩니다.

8. **Submit Transaction**을 클릭합니다.
   
9. 신원을 인증하기 위해 비밀번호를 입력한 다음 **Sign and Submit**을 클릭합니다.
   
   계정에 충분한 권한이 없는 경우, 트랜잭션이 `BadOrigin` 오류로 실패합니다.

### 임대 기간

Rococo에서 할당된 파라체인 슬롯의 현재 임대 기간 및 슬롯 가용성 설정은 다음과 같습니다:

- **영구적인 슬롯 임대 기간**: 1년 (365일)
- **임시적인 슬롯 임대 기간**: 3일
- **영구적인 슬롯 최대 개수**: 최대 25개의 영구적인 슬롯
- **임시적인 슬롯 최대 개수**: 최대 20개의 임시적인 슬롯
- **임대 기간 당 할당된 임시적인 슬롯 최대 개수**: 3일 동안 최대 5개의 임시적인 슬롯

이러한 설정은 커뮤니티의 요구에 따라 변경될 수 있습니다.

## 파라체인 테스트하기

슬롯이 할당되고 활성화된 후, Rococo 테스트넷에서 파라체인을 테스트할 수 있습니다.
임시적인 슬롯 임대 기간이 종료되면 파라체인은 자동으로 파라스레드로 다운그레이드됩니다.
등록 및 승인된 슬롯은 자동으로 라운드 로빈 방식으로 순환되므로, 때때로 파라체인으로 다시 온라인 상태가 될 것입니다.

## 다음 단계

Rococo와 같은 테스트넷에서 파라체인을 철저히 테스트하고 검증한 후, [Kusama](https://kusama.network/)와 같은 프로덕션 네트워크에 참여하고 [슬롯 경매](https://guide.kusama.network/docs/learn-auction/)를 통해 파라체인 슬롯을 확보할 수 있습니다.