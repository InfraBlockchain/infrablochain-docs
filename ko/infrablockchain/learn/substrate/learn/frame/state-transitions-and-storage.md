---
title: 상태 전이와 스토리지
description:
keywords:
---

Substrate는 데이터베이스 기반의 단순한 Key-Value 데이터 스토리지를 사용하여 수정된 Merkle 트리로 구현됩니다.
Substrate의 모든 [상위 수준 스토리지 추상화](./runtime-storage.md)는 이 단순한 Key-Value 스토리지 위에 구축되었습니다.

## Key-Value 데이터베이스

Substrate는 빠른 저장 환경을 위한 영속적인 Key-Value 스토리지인 [RocksDB](https://rocksdb.org/)를 사용하여 스토리지 데이터베이스를 구현합니다. 또한 실험적인 [Parity DB](https://github.com/paritytech/parity-db)도 지원합니다.

DB는 Substrate의 모든 영속적인 스토리지를 필요로하는 구성 요소에 사용됩니다. 예를 들어:

- Substrate 클라이언트
- Substrate 라이트 클라이언트
- 오프 체인 워커

## Trie 추상화

단순한 Key-Value 스토리지를 사용하는 장점 중 하나는 이를 기반으로 스토리지 구조를 쉽게 추상화할 수 있다는 것입니다.

Substrate는 수정 가능하고 루트 해시를 효율적으로 다시 계산할 수 있는 Base-16 Modified Merkle Patricia 트리("trie")를 [`paritytech/trie`](https://github.com/paritytech/trie)에서 사용하여 제공합니다.

Trie는 역사적인 블록 상태를 효율적으로 저장하고 공유할 수 있습니다. Trie 루트는 Trie 내부의 데이터를 나타냅니다. 즉, 서로 다른 데이터를 가진 두 개의 Trie는 항상 다른 루트를 가지게 됩니다.
따라서 두 개의 블록체인 노드는 단순히 Trie 루트를 비교함으로써 동일한 상태를 가지고 있는지 쉽게 확인할 수 있습니다.

Trie 데이터에 접근하는 것은 비용이 많이 듭니다.
각 읽기 작업은 O(log N) 시간이 걸리며, 여기서 N은 Trie에 저장된 요소의 수입니다. 이를 완화하기 위해 Key-Value 캐시를 사용합니다.

모든 Trie 노드는 DB에 저장되며, Trie 상태의 일부는 가지를 자를 수 있습니다. 즉, 비아카이브 노드에 대해 가지를 스토리지에서 삭제할 수 있습니다.
성능상의 이유로 [참조 카운팅](http://en.wikipedia.org/wiki/Reference_counting)을 사용하지 않습니다.

### 상태 Trie

Substrate 기반 체인은 각 블록 헤더에 배치된 상태 Trie라는 단일한 주요 Trie를 가지고 있습니다.
이는 블록체인의 상태를 쉽게 확인하고 라이트 클라이언트가 증명을 검증하는 기반을 제공하기 위해 사용됩니다.

이 Trie는 포크가 아닌 정규 체인의 내용만 저장합니다.
비정규적인 모든 것을 메모리에서 참조 카운트된 상태로 유지하는 별도의 [`state_db` 레이어](https://paritytech.github.io/substrate/master/sc_state_db/index.html)가 있습니다.

### 자식 Trie

Substrate는 런타임에서 사용할 수 있는 자체 루트 해시를 가진 새로운 자식 Trie를 생성하기 위한 API도 제공합니다.

자식 Trie는 주요 상태 Trie와 동일하지만, 자식 Trie의 루트는 블록 헤더 대신 주요 Trie의 노드로 저장되고 업데이트됩니다.
자식 Trie의 헤더는 주요 상태 Trie의 일부이므로, 자식 Trie를 포함한 전체 노드 상태를 검증하는 것은 여전히 쉽습니다.

자식 Trie는 자체 독립적인 Trie로서 별도의 루트 해시를 가지고 있으므로 해당 Trie의 특정 내용을 검증하는 데 사용할 수 있습니다.
Trie의 하위 섹션은 이러한 요구 사항을 자동으로 충족시키는 루트 해시와 같은 표현을 가지고 있지 않기 때문에 대신 자식 Trie가 사용됩니다.

## 스토리지 조회

Substrate로 구축된 블록체인은 런타임 스토리지를 조회하기 위해 사용할 수 있는 원격 프로시저 호출(RPC) 서버를 노출합니다. Substrate RPC를 사용하여 스토리지 항목에 액세스할 때, 해당 항목과 관련된 [키](#Key-Value-데이터베이스)만 제공하면 됩니다.
Substrate의 [런타임 스토리지 API](./runtime-storage.md)는 여러 가지 스토리지 항목 유형을 노출합니다. 다른 유형의 스토리지 항목에 대한 스토리지 키를 계산하는 방법을 알아보려면 계속 읽어보세요.

### StorageValue keys

간단한 [스토리지 값](./runtime-storage.md#스토리지-값)에 대한 키를 계산하려면, 스토리지 값을 포함하는 팔렛의 이름의 [TwoX 128 해시](https://github.com/Cyan4973/xxHash)를 취한 후, 스토리지 값 자체의 이름의 TwoX 128 해시를 이어 붙입니다.
예를 들어, [Sudo](https://paritytech.github.io/substrate/master/pallet_sudo/index.html) 팔렛은 `Key`라는 스토리지 값 항목을 노출합니다:

```rust
twox_128("Sudo")                   = "0x5c0d1176a568c1f92944340dbfed9e9c"
twox_128("Key")                    = "0x530ebca703c85910e7164cb7d1c9e47b"
twox_128("Sudo") + twox_128("Key") = "0x5c0d1176a568c1f92944340dbfed9e9c530ebca703c85910e7164cb7d1c9e47b"
```

익숙한 `Alice` 계정이 sudo 사용자인 경우, Sudo 팔렛의 `Key` 스토리지 값에 대한 RPC 요청과 응답은 다음과 같이 나타낼 수 있습니다:

```rust
state_getStorage("0x5c0d1176a568c1f92944340dbfed9e9c530ebca703c85910e7164cb7d1c9e47b") = "0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d"
```

이 경우, 반환된 값(`"0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d"`)은 Alice의 [SCALE](/reference/scale-codec)로 인코딩된 계정 ID(`5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY`)입니다.

[암호학적이 아닌](./runtime-storage.md#암호학적-해시-알고리즘) TwoX 128 해시 알고리즘을 사용하여 스토리지 값 키를 생성하는 것을 알 수 있습니다.
이는 해시 함수의 성능 비용을 지불할 필요가 없기 때문에, 해시 함수의 입력(팔렛과 스토리지 항목의 이름)은 런타임 개발자에 의해 결정되며, 잠재적으로 악의적인 블록체인 사용자에 의해 결정되지 않기 때문입니다.

### StorageMap keys

스토리지 맵([Storage Map](./runtime-storage.md#스토리지-맵))의 키는 스토리지 맵을 포함하는 팔렛의 이름의 TwoX 128 해시와 스토리지 맵 자체의 이름의 TwoX 128 해시와 동일합니다.
맵에서 요소를 검색하려면, 원하는 맵 키의 해시를 스토리지 맵의 스토리지 키에 추가합니다.
두 개의 키(Storage Double Maps)를 가진 맵의 경우, 첫 번째 맵 키의 해시 다음에 두 번째 맵 키의 해시를 Storage Double Map의 스토리지 키에 추가합니다.

스토리지 값과 마찬가지로, Substrate는 팔렛과 스토리지 맵 이름에 대해 TwoX 128 해시 알고리즘을 사용하지만, 맵의 요소에 대한 해시된 키를 결정할 때 올바른 [해시 알고리즘](./runtime-storage.md#해시-알고리즘)`#[pallet::storage]` 매크로에서 선언된 알고리즘을 사용해야 합니다.

다음은 `Balances` 팔렛의 `FreeBalance` 맵에서 `Alice` 계정의 잔액을 조회하는 예제입니다.
이 예제에서 `FreeBalance` 맵은 [투명한 Blake2 128 Concat 해시 알고리즘](./runtime-storage.md#투명한-해싱-알고리즘)을 사용합니다:

```rust
twox_128("Balances")                                             = "0xc2261276cc9d1f8598ea4b6a74b15c2f"
twox_128("FreeBalance")                                          = "0x6482b9ade7bc6657aaca787ba1add3b4"
scale_encode("5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY") = "0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d"

blake2_128_concat("0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d") = "0xde1e86a9a8c739864cf3cc5ec2bea59fd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d"

state_getStorage("0xc2261276cc9d1f8598ea4b6a74b15c2f6482b9ade7bc6657aaca787ba1add3b4de1e86a9a8c739864cf3cc5ec2bea59fd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d") = "0x0000a0dec5adc9353600000000000000"
```

스토리지 조회에서 반환된 값(`"0x0000a0dec5adc9353600000000000000"`)은 Alice의 계정 잔액의 [SCALE](./scale-codec.md)-인코딩된 값(`"1000000000000000000000"`)입니다.
Alice의 계정 ID를 해시하기 전에 SCALE로 인코딩해야 한다는 점에 유의하십시오.
또한 `blake2_128_concat` 함수의 출력은 32개의 16진수 문자로 시작하고 함수의 입력이 이어지는 것에 주목하십시오.
이는 Blake2 128 Concat이 [투명한 해싱 알고리즘](./runtime-storage.md#투명한-해시-함수)임을 의미합니다.

위의 예제에서는 이 특성이 사소한 것처럼 보일 수 있지만, 맵의 키를 반복하는 것이 목표인 경우(단일 키와 관련된 값을 검색하는 것이 아닌 경우)에는 유용성이 더욱 명확해집니다.
맵의 키를 반복하는 기능은 맵을 사람들이 자연스럽게 사용할 수 있도록 하는 데 필요한 일반적인 요구 사항입니다(예: UI에서): 먼저, 사용자에게 맵의 요소 목록이 제시되고, 그 사용자는 해당하는 요소에 대한 자세한 내용을 조회하기 위해 맵에 대한 쿼리를 선택할 수 있습니다.

다음은 동일한 예제 스토리지 맵(`Balances` 팔렛의 `FreeBalances` 맵)을 사용하는 또 다른 예제입니다. 이 맵은 Blake2 128 Concat 해시 알고리즘을 사용하며, Substrate RPC를 사용하여 맵의 키 목록을 `state_getKeys` RPC 엔드포인트를 통해 쿼리하는 방법을 보여줍니다:

```rust
twox_128("Balances")                                      = "0xc2261276cc9d1f8598ea4b6a74b15c2f"
twox_128("FreeBalance")                                   = "0x6482b9ade7bc6657aaca787ba1add3b4"

state_getKeys("0xc2261276cc9d1f8598ea4b6a74b15c2f6482b9ade7bc6657aaca787ba1add3b4") = [
 "0xc2261276cc9d1f8598ea4b6a74b15c2f6482b9ade7bc6657aaca787ba1add3b4de1e86a9a8c739864cf3cc5ec2bea59fd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d",
 "0xc2261276cc9d1f8598ea4b6a74b15c2f6482b9ade7bc6657aaca787ba1add3b432a5935f6edc617ae178fef9eb1e211fbe5ddb1579b72e84524fc29e78609e3caf42e85aa118ebfe0b0ad404b5bdd25f",
 ...
]
```

Substrate RPC의 `state_getKeys` 엔드포인트에서 반환된 목록의 각 요소는 직접적으로 Substrate RPC의 `state_getStorage` 엔드포인트의 입력으로 사용할 수 있습니다.
실제로, 위의 예제 목록의 첫 번째 요소는 이전 예제에서 `Alice`의 잔액을 찾기 위해 사용된 `state_getStorage` 쿼리의 입력과 동일합니다.
이러한 키가 속한 맵은 투명한 해싱 알고리즘을 사용하여 키를 생성하므로, 목록의 두 번째 요소와 연결된 계정을 결정하는 것이 가능합니다.
목록의 각 요소는 동일한 맵 내의 키를 나타내는 64개의 문자로 시작하는 16진수 값입니다. 이는 각 목록 요소가 동일한 맵 내의 키를 나타내며, 이 맵은 두 개의 128비트 또는 32개의 16진수 문자로 구성된 TwoX 128 해시를 연결하여 식별됩니다.
목록의 두 번째 요소에서 이 부분을 제거한 후에는 `0x32a5935f6edc617ae178fef9eb1e211fbe5ddb1579b72e84524fc29e78609e3caf42e85aa118ebfe0b0ad404b5bdd25f`라는 16진수 값이 남습니다. 이 값은 [SCALE](./scale-codec.md)-인코딩된 계정 ID입니다.
이 값을 디코딩하면 `5GNJqTPyNqANBkUVMN1LPPrxXnFouWXoe2wNSmmEoLctxiZY`라는 결과가 나옵니다. 이는 익숙한 `Alice_Stash` 계정의 계정 ID입니다.

## 런타임 스토리지 API

Substrate의 [FRAME Support 크레이트](https://paritytech.github.io/substrate/master/frame_support/index.html)는 런타임의 스토리지 항목에 대한 고유하고 결정론적인 키를 생성하기 위한 유틸리티를 제공합니다.
이러한 스토리지 항목은 [상태 Trie](#Trie-추상화)에 배치되며, [키로 Trie를 쿼리](#스토리지-조회)하여 액세스할 수 있습니다.

## 다음으로 어디로 가야 할까요

- [런타임 스토리지](./runtime-storage.md)
- [타입 인코딩 (SCALE)](./scale-codec.md)