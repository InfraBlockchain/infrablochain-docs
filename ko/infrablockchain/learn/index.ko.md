---
title: 인프라 블록체인 배우기
description: 이 튜토리얼은 엔터프라이즈 블록체인을 구축하기 위해 필요한 개념을 배웁니다.
keywords:
  - Proof-of-Transaction
  - 시스템 토큰
---

## _**인프라 블록체인(InfraBlockchain)**_ 

### 개요 

_**인프라 블록체인(InfraBlockchain)**_ 은 기업과 공공기관을 위한 퍼블릭/허가형 블록체인으로써 블록체인 프로토콜에 의해 자체적으로 네이티브 암호화폐(예를들면, 비트코인, 이더, 닷, 등)를 발행하지 않고 다음과 같은 기업 및 공공기관 중심의 _새로운 방식_ 을 도입하였습니다. 

### [시스템 토큰](./system-token.md)

_**인프라 블록체인(InfraBlockchain)**_ 을 사용하는 모든 기업 및 공공 기관은 법정 화폐가 담보된 스테이블 토큰인 **_시스템 토큰(System Token)_** 을 발행할 수 있습니다. 선출된 블록 생성자(밸리데이터)는 시스템 토큰으로 사용될 토큰을 선정하여 트랜잭션 수수료로 사용할 수 있게 합니다.

### [Proof-of-Transaction](./proof-of-transaction.md)

_**인프라 블록체인(InfraBlockchain)**_ 의 블록 생성자(밸리데이터)는 블록 생성자로 선출되기 위해 실제 블록체인 트랜잭션을 많이 일으켜 직접적으로 생태계 발전에 기여를 한 서비스 제공자들에게 인센티브를 부여하는 TaaV(Transaction-as-a-Vote) 에 의해 구현된 PoT(Proof-of-Transaction) 이라는 독자적인 합의 메커니즘으로 선출됩니다. 

## _**인프라 블록스페이스(InfraBlockspace)**_

_**인프라 블록 스페이스(InfraBlockspace)**_ 는 블록이 실행될 수 있는 추상적인 공간입니다. _**인프라 블록체인(InfraBlockchain)**_ 은 _**인프라 릴레이체인(InfraRelaychain)**_ 이 생성한 _**인프라 블록스페이스**_ 안에서 블록을 실행(execute), 검증(validate) 및 확정(finalize)됩니다. 또한, 여러개의 인프라 블록체인에서 생성되는 블록들은 _**인프라 블록스페이스(InfraBlockspace)**_ 안에서 _**인프라 릴레이체인(InfraRelaychain)**_ 블록 생성자(밸리데이터)들은 _shared_security_ 를 통해 안전을 보장받고 병렬적으로 실행되며 네이티브 브릿지인 XCM(Cross Consensus Message) 을 가능하게 합니다.