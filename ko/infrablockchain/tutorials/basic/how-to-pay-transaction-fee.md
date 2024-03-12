---
title: 시스템 토큰 등록 및 수수료 사용해보기
description: 이 튜토리얼은 토큰 생성부터 시스템 토큰 등록을 위한 거버넌스, 시스템 토큰 사용까지의 일련의 과정에 대해 배웁니다.
keywords:
  - Assets
  - System token manager
  - Governance
  - Transaction fee
---

이 튜토리얼은 멀티체인 환경에서 발행한 토큰을 시스템 토큰으로 등록하고 가스비로 사용하는 방법에 대해 배워보겠습니다.

## 튜토리얼 목표

이 튜토리얼을 완료함으로써 다음 목표를 달성할 수 있습니다:

- 좀비넷을 활용하여 `릴레이체인 <> 파라체인` 테스트 네트워크를 구축할 수 있습니다.
- 파라체인에서 토큰을 발행하고 릴레이체인에서 해당 토큰을 시스템 토큰으로 등록하기 위한 거버넌스를 발의할 수 있습니다.
- 거버넌스가 통과된 후, 파라체인에서 발행한 시스템 토큰으로 가스비를 낼 수 있습니다.

## 시작하기 전에

시작하기 전에 다음을 확인하세요:

- [좀비넷을 이용한 파라체인 네트워크 시뮬레이션 하기](./test/simulate-parachains.md)

## 파라체인에 토큰 생성 및 발행하기

시스템 토큰을 릴레이체인에 등록하기 전에 파라체인에 토큰을 생성해야합니다. *이때, 파라체인 토큰 생성 및 발행에 필요한 가스비를 위해 테스트 시스템 토큰 혹은 시스템 토큰을 보유하고 있어야 합니다.*

파라체인에 토큰 생성 및 발행 절차는 다음과 같습니다:

1. [인프라블록체인 익스플로러](https://portal.infrablockspace.net/#/explorer/)를 열고 파라체인 엔드포인트에 연결합니다.

2. [Developer > Extrinsic] 에서 [Assets](https://github.com/InfraBlockchain/infrablockchain-substrate/tree/master/substrate/frame/assets) 팔렛의 [create](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/599828207489db1d2b4633473c15c9be9dd97253/substrate/frame/assets/src/lib.rs#L625)를 선택한 후 토큰을 생성합니다.

   - id: 토큰을 식별할 값. 여기서는 1을 넣어줍니다. 
   - admin: 토큰 관리 계정. 여기서는 `alice` 를 지정합니다.
   - minBalance: 토큰 최소 예치 단위. 여기서는 1 을 넣어줍니다. 

    ![토큰 생성하기](/media/images/docs/infrablockchain/tutorials/create_token.png)

3. Assets 팔렛의 [mint](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/599828207489db1d2b4633473c15c9be9dd97253/substrate/frame/assets/src/lib.rs#L801C7-L801C14)를 선택한 후 토큰을 발행합니다. 

   - id: 토큰을 식별할 값. 여기서는 전에 생성했던 토큰 식별자를 넣어줍니다.
   - beneficiary: 발행된 토큰을 받을 계정. 여기서는 `alice` 에게 발행해줍니다.
   - amount: 발행할 토큰의 양. 여기서는 1000000000000 을 넣어줍니다. 
  
    ![토큰 발행하기](/media/images/docs/infrablockchain/tutorials/mint_token.png)

## 릴레이체인에 `register_system_token` 거버넌스 등록하기

### 거버넌스에 올릴 `register_system_token`을 위한 preimage 준비

1. [인프라블록체인 익스플로러](https://portal.infrablockspace.net/#/explorer/)를 열고 릴레이체인 엔드포인트에 연결합니다.

2. `register_system_token`을 거버넌스 투표로 올리기 위해 preimage에 먼저 등록을 해야합니다.
  
3. [Governance > Preimages > Add preimage]를 누릅니다.
(preimage는 누구나 등록이 가능합니다.)

![preimage_button](/media/images/docs/infrablockchain/tutorials/preimage_button.png)

4. 파라체인에서 생성했던 토큰 정보를 토대로 `register_system_token`에 대한 preimage를 생성합니다.

![시스템 토큰 등록하기(아래 계속)](/media/images/docs/infrablockchain/tutorials/register_system_token1.png)

![시스템 토큰 등록하기](/media/images/docs/infrablockchain/tutorials/register_system_token2.png)

- `systemTokenType`: 시스템 토큰의 종류. 시스템 토큰 종류에 대한 자세한 내용은 [다음](../learn/protocol/system-token.md)을 참고하세요. 여기서는 `Original` 을 선택합니다. 
  ```text
  - paraId: 토큰이 생성된 파라체인 식별자
  - palletId: 토큰이 생성된 *Assets 팔렛* 의 식별자 
  - assetId: 토큰 식별자
  ```   
- `systemTokenWeight`: 토큰의 가중치. 여기서는 1_000_000(default)로 설정해줍니다.
- `wrappedForRelayChain`: 릴레이체인에 사용할 wrapped 시스템 토큰. 
- `systemTokenMetadata`: 시스템 토큰에 대한 메타데이터.
- `assetMetadata`: 토큰에 대한 메타데이터.

5. 다음과 같이 해시값과 함께 정상적으로 등록이 됩니다.

![preimage 결과](/media/images/docs/infrablockchain/tutorials/preimage_result.png)

### 등록된 preimage를 거버넌스에 등록하기

`register_system_token` 에 대한 preimage를 성공적으로 등록했다면 이제 거버넌스에 등록할 수 있습니다.

1. [인프라블록체인 익스플로러](https://portal.infrablockspace.net/#/explorer/)를 열고 릴레이체인 엔드포인트에 연결합니다.

2. 익스플로러 탭에서 [Developer > Extrinsic]를 누른 후,
[*Council*](https://github.com/InfraBlockchain/infrablockchain-substrate/tree/master/substrate/frame/collective) 팔렛의 [propose](https://github.com/InfraBlockchain/infrablockchain-substrate/blob/599828207489db1d2b4633473c15c9be9dd97253/substrate/frame/collective/src/lib.rs#L519)를 통해 등록했던 preimage에 대한 안건을 council 거버넌스에 올립니다. 
(주의: 릴레이체인 밸리데이터만 council propose를 올리고 투표할 수 있습니다.)

  ```text
  - threshold: 안건을 종료하기 위한 최소 투표수
  - Legacy-hash: 등록한 preimage에 대한 hash
  - lengthBound: preimage의 길이(Byte) 제한. 
  ```

![거버넌스 제안](/media/images/docs/infrablockchain/tutorials/council_propose.png)

## 거버넌스 통과시키기

거버넌스에 정상적으로 등록됐다면 [Governance > Council > Motion] 에 해당 안건이 아래와 같이 올라와있습니다. 

![거버넌스 투표](/media/images/docs/infrablockchain/tutorials/governance_voting.png)

위 화면에서 Vote를 누르고, Council을 구성하는 밸리데이터들(alice_stash, bob_stash)로 투표를 해줍니다.

threshold 인원 이상 투표를 했다면 투표를 바로 종료시킬 수 있고, 정족수가 동의(60% 이상)했다면 해당 안건은 바로 통과됩니다.

![투표 종료](/media/images/docs/infrablockchain/tutorials/vote_close.png)

안건이 정상적으로 통과됐고, 파라체인에서도 `sufficient`가 `true`로 바뀐 것을 확인할 수 있습니다.

![거버넌스 실행](/media/images/docs/infrablockchain/tutorials/enact_motion.png)

![파라체인 토큰 상태 변화](/media/images/docs/infrablockchain/tutorials/parachain_sufficient_true.png)

## 시스템 토큰 가스비로 사용하기

이제 파라체인에서는 처음에 발행한 토큰이 시스템 토큰이 되었고,
해당 토큰으로 트랜잭션 가스비를 지불할 수 있습니다. 

아래처럼 Assets 의 `transfer` 트랜잭션을 호출해 보겠습니다:

![parachain_asset_transfer](/media/images/docs/infrablockchain/tutorials/parachain_asset_transfer.png)

트랜잭션의 extra 정보에서 `include an optional asset id` 토글을 누르면 등록된 시스템 토큰 정보를 입력해줄 수 있는 칸이 나오게 됩니다. 

![시스템 토큰](/media/images/docs/infrablockchain/tutorials/system_token_id.png)

이 시스템 토큰을 활용해 가스비를 낼 수 있는 걸 확인할 수 있습니다.

![시스템 토큰 가스비](/media/images/docs/infrablockchain/tutorials/system_token_paid.png)