---
title: 합의
description: Substrate가 제공하는 합의 모델을 설명합니다.
keywords:
---

모든 블록체인은 블록체인 상태에 동의하기 위한 어떤 형태의 합의 메커니즘을 필요로 합니다. Substrate는 블록체인을 구축하기 위한 모듈식 프레임워크를 제공하기 때문에, 노드들이 합의에 도달하기 위해 몇 가지 다른 모델을 지원합니다.
일반적으로, 다른 합의 모델은 서로 다른 트레이드오프를 가지고 있으므로, 체인에서 사용할 합의 유형을 선택하는 것은 중요한 고려사항입니다.
Substrate가 기본적으로 지원하는 합의 모델은 최소한의 구성을 필요로 하지만, 필요한 경우 사용자 정의 합의 모델을 구축할 수도 있습니다.

## 두 단계로 나뉜 합의

일부 블록체인과 달리, Substrate는 합의에 도달하기 위한 요구사항을 두 개의 별도 단계로 분리합니다:

- **블록 생성**은 노드들이 새로운 블록을 생성하는 과정입니다.

- **블록 확정**은 포크를 처리하고 **정규** 체인을 선택하는 과정입니다.

## 블록 생성

합의에 도달하기 전에, 블록체인 네트워크의 일부 노드들은 새로운 블록을 생성할 수 있어야 합니다.
블록체인이 어떤 합의 모델을 사용하는지에 따라, 어떤 노드가 블록을 생성할 권한을 가지는지 결정됩니다.
예를 들어, 중앙 집중화된 네트워크에서는 단일 노드가 모든 블록을 생성할 수 있습니다.
신뢰할 수 있는 노드가 없는 완전히 탈중앙화된 네트워크에서는 알고리즘이 각 블록 높이마다 블록 생성자를 선택해야 합니다.

Substrate 기반 블록체인에서는 다음 중 하나의 블록 생성 알고리즘을 선택하거나 직접 구현할 수 있습니다:

- 권한 기반 라운드 로빈 스케줄링 ([Aura](../basic/glossary.md#권한-라운드-aura)).
- 블록체인 확장의 블라인드 할당 ([BABE](../basic/glossary.md#블록체인-확장의-블라인드-할당-babe)) 슬롯 기반 스케줄링.
- 작업 증명 계산 기반 스케줄링.

Aura와 BABE 합의 모델은 블록을 생성할 수 있는 **검증자 노드**의 알려진 집합을 가져야 합니다.
이러한 합의 모델에서는 시간을 이산적인 슬롯으로 나눕니다.
각 슬롯에서는 일부 검증자만 블록을 생성할 수 있습니다.
Aura 합의 모델에서는 블록을 생성할 수 있는 검증자들이 순환 방식으로 변경됩니다.
BABE 합의 모델에서는 검증자들은 라운드 로빈 선택 방식이 아닌 검증 가능한 무작위 함수 (VRF)에 기반하여 선택됩니다.

작업 증명 합의 모델에서는 어떤 노드든지 계산적으로 문제를 해결한 경우 언제든지 블록을 생성할 수 있습니다.
문제를 해결하는 데에는 CPU 시간이 소요되므로, 노드는 자신의 컴퓨팅 리소스에 비례하여 블록을 생성할 수 있습니다.
Substrate는 작업 증명 블록 생성 엔진을 제공합니다.

## 확정과 포크

블록은 헤더와 [트랜잭션](../basic/transaction-types.md)을 포함하는 기본 단위입니다.
각 블록 헤더에는 부모 블록에 대한 참조가 포함되어 있으므로, 체인을 제네시스 블록까지 역추적할 수 있습니다.
포크는 두 개의 블록이 동일한 부모를 참조할 때 발생합니다.
블록 확정은 이러한 포크를 해결하여 정규 체인만 존재하도록 하는 메커니즘입니다.

포크 선택 규칙은 확장할 최상의 체인을 선택하는 알고리즘입니다.
Substrate는 이 포크 선택 규칙을 [`SelectChain`](https://paritytech.github.io/substrate/master/sp_consensus/trait.SelectChain.html) 트레이트를 통해 노출시킵니다.
이 트레이트를 사용하여 사용자 정의 포크 선택 규칙을 작성하거나 Polkadot 및 유사한 체인에서 사용되는 최종성 메커니즘인 [GRANDPA](https://github.com/w3f/consensus/blob/master/pdf/grandpa.pdf)를 사용할 수 있습니다.

GRANDPA 프로토콜에서는 Longest chain rule이 단순히 가장 좋은 체인을 가장 긴 체인으로 선택한다고 말합니다.
Substrate는 이 체인 선택 규칙을 [`LongestChain`](https://paritytech.github.io/substrate/master/sc_consensus/struct.LongestChain.html) 구조체로 제공합니다.
GRANDPA는 투표 메커니즘에서 Longest chain rule을 사용합니다.

![Longest chain rule](/media/images/docs/infrablockchain/learn/substrate/learn/consensus-longest.png)

Longest chain rule의 단점은 공격자가 네트워크를 앞지르는 블록체인을 생성하여 사실상 메인 체인을 탈취할 수 있다는 것입니다.
Greedy Heaviest Observed SubTree (GHOST) 규칙은 제네시스 블록에서 시작하여, 각 포크를 해결할 때 가장 많은 블록이 쌓인 가장 무거운 브랜치를 선택합니다.

![GHOST rule](/media/images/docs/infrablockchain/learn/substrate/learn/consensus-ghost.png)

이 다이어그램에서 가장 무거운 체인은 가장 많은 블록이 쌓인 포크입니다.
체인 선택에 GHOST rule을 사용하는 경우, 이 포크가 가장 긴 체인 포크보다 더 적은 블록을 가지더라도 메인 체인으로 선택됩니다.

## 결정론적 확정

사용자들은 트랜잭션이 확정되고 영수증이 전달되거나 서류가 서명되는 등의 이벤트로 신호를 받기를 원합니다.
그러나 지금까지 설명한 블록 생성 및 포크 선택 규칙에서는 트랜잭션이 완전히 확정되지 않습니다.
더 긴 또는 더 무거운 체인이 트랜잭션을 되돌릴 수 있는 가능성이 항상 존재합니다.
하지만 특정 블록 위에 더 많은 블록이 쌓일수록, 해당 블록이 되돌려지지 않을 가능성은 줄어듭니다.
이러한 방식으로, 블록 생성과 적절한 포크 선택 규칙은 **확률적 확정성**을 제공합니다.

블록체인이 결정론적 확정성을 필요로 하는 경우, 블록체인 로직에 확정성 메커니즘을 추가할 수 있습니다.
예를 들어, 고정된 권한 집합의 구성원들이 **확정성** 투표를 수행할 수 있습니다.
특정 블록에 대해 충분한 투표가 이루어지면, 해당 블록은 확정됩니다.
대부분의 블록체인에서 이 임계값은 2/3입니다.
확정된 블록은 하드 포크와 같은 외부 조정 없이는 되돌릴 수 없습니다.

일부 합의 모델에서는 블록 생성과 블록 확정이 결합되어, 새로운 블록 `N+1`은 블록 `N`이 확정될 때까지 생성될 수 없습니다.
Substrate에서는 블록 생성과 블록 확정을 서로 분리합니다.
블록 생성과 블록 확정을 분리함으로써, Substrate는 블록 생성 알고리즘과 확률적 확정성을 함께 사용하거나 결정론적 확정성을 달성하기 위해 확정성 메커니즘을 결합할 수 있도록 합니다.

블록체인이 확정성 메커니즘을 사용하는 경우, 포크 선택 규칙을 수정하여 확정성 투표 결과를 고려해야 합니다.
예를 들어, 가장 긴 체인 기간을 취하는 대신, 노드는 가장 최근에 확정된 블록을 포함하는 가장 긴 체인을 선택합니다.

## 기본 합의 모델

사용자 정의 합의 메커니즘을 구현할 수 있지만, [Substrate 노드 템플릿](https://github.com/substrate-developer-hub/substrate-node-template)은 기본적으로 블록 생성에 Aura를, 확정에 GRANDPA를 포함하고 있습니다.
Substrate는 또한 BABE와 작업 증명 합의 모델의 구현을 제공합니다.

### Aura

[Aura](https://paritytech.github.io/substrate/master/sc_consensus_aura/index.html)는 슬롯 기반 블록 생성 메커니즘을 제공합니다.
Aura에서는 알려진 권한 집합의 차례대로 블록을 생성합니다.

### BABE

[BABE](https://paritytech.github.io/substrate/master/sc_consensus_babe/index.html)는 슬롯 기반 블록 생성을 제공하며 알려진 검증자 집합을 사용하며 일반적으로 스테이크 기반 블록체인에서 사용됩니다.
Aura와 달리, 슬롯 할당은 검증 가능한 무작위 함수 (VRF)의 평가에 기반합니다.
각 검증자는 _에포크_를 위한 가중치가 할당되며, 이 에포크는 슬롯으로 분할되고 검증자는 각 슬롯에서 VRF를 평가합니다.
검증자의 VRF 출력이 가중치보다 작은 경우, 해당 검증자는 블록을 생성할 수 있습니다.

동일한 슬롯에서 여러 검증자가 블록을 생성할 수 있는 경우, BABE에서는 Aura보다 포크가 더 일반적이며, 좋은 네트워크 조건에서도 포크가 발생할 수 있습니다.

Substrate의 BABE 구현은 특정 슬롯에서 권한이 선택되지 않은 경우를 대비한 예비 슬롯 할당 메커니즘을 갖고 있습니다.
이 보조 슬롯 할당은 BABE가 일정한 블록 시간을 달성할 수 있도록 합니다.

### 작업 증명

[작업 증명](https://paritytech.github.io/substrate/master/sc_consensus_pow/index.html) 블록 생성은 슬롯 기반이 아니며 알려진 권한 집합을 필요로 하지 않습니다.
작업 증명에서는 누구나 언제든지 블록을 생성할 수 있으며, 계산적으로 어려운 문제 (일반적으로 해시 선행 이미지 검색)를 해결할 수 있는 경우에만 블록을 생성할 수 있습니다.
이 문제의 난이도는 통계적인 목표 블록 시간을 제공하기 위해 조정될 수 있습니다.

### GRANDPA

[GRANDPA](https://paritytech.github.io/substrate/master/sc_consensus_grandpa/index.html)는 블록 확정을 제공합니다.
GRANDPA는 BABE와 마찬가지로 알려진 가중치 기반 권한 집합을 가지고 있습니다.
하지만 GRANDPA는 블록을 생성하지 않고, 블록 생성 노드가 생성한 블록에 대한 공동체 정보를 수신합니다.
GRANDPA 검증자들은 _체인_이 아닌 _블록_에 투표합니다.
GRANDPA 검증자들은 가장 우수하다고 생각하는 블록에 투표하며, 그들의 투표는 이전 모든 블록에 대해 추이적으로 적용됩니다.
GRANDPA 권한의 2/3 이상이 특정 블록에 투표한 후에야 해당 블록이 확정됩니다.

GRANDPA를 포함한 모든 결정론적 확정 알고리즘은 적어도 `2f + 1`개의 장애 노드가 아닌 노드가 필요합니다. 여기서 `f`는 장애 또는 악의적인 노드의 수입니다.
이 임계값이 어디에서 나왔는지 및 왜 이상적인지에 대해서는 [Reaching Agreement in the Presence of Faults](https://lamport.azurewebsites.net/pubs/reaching.pdf)라는 고전적인 논문이나 [Wikipedia: Byzantine Fault](https://en.wikipedia.org/wiki/Byzantine_fault)에서 자세히 알아볼 수 있습니다.

모든 합의 프로토콜이 단일, 정규 체인을 정의하지는 않습니다.
일부 프로토콜은 동일한 부모를 참조하는 두 블록이 충돌하는 상태 변경이 없는 경우 [유향 비순환 그래프](https://en.wikipedia.org/wiki/Directed_acyclic_graph) (DAG)를 유효화합니다.
이와 같은 예로 [AlephBFT](https://github.com/aleph-zero-foundation/aleph-node)를 참조하십시오.

## 다음으로 어디로 가야 할까요

- [BABE 연구](https://research.web3.foundation/Polkadot/protocols/block-production/Babe)
- [GRANDPA 연구](https://research.web3.foundation/Polkadot/protocols/finality)