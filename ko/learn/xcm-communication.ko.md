---
title: 크로스 컨센서스 메시징
description: 크로스 컨센서스 통신과 크로스 컨센서스 메시징(XCM) 형식에 대한 개요를 제공합니다.
keywords:
  - XCM
  - XCMP
  - Polkadot
  - 파라체인
  - 릴레이 체인
---

크로스 컨센서스 통신은 서로 다른 컨센서스 시스템, 트랜잭션 형식 및 전송 프로토콜에 의해 생성된 경계를 횡단하는 데 사용되는 메시지 형식인 XCM을 기반으로 합니다.

XCM 형식은 메시지의 내용을 표현합니다.
각 메시지는 **보낸 사람**이 요청한 **명령어** 집합으로 구성되며, 메시지 **수신자**가 수락하거나 거부할 수 있습니다.
메시지 형식은 메시지를 보내고 받는 데 사용되는 **메시지 프로토콜**과 완전히 독립적입니다.

## 메시지 프로토콜

Polkadot 생태계에서는 체인 간 메시지를 전송하는 데 사용되는 세 가지 주요 통신 채널인 메시지 프로토콜이 있습니다:

- Upward message passing (UMP)는 파라체인이 릴레이 체인으로 메시지를 전달할 수 있게 합니다.
- Downward message passing (DMP)는 릴레이 체인이 파라체인으로 메시지를 전달할 수 있게 합니다.
- Cross-consensus message passing (XCMP)는 같은 릴레이 체인에 연결된 다른 파라체인과 메시지를 교환할 수 있게 합니다.

Upward와 Downward 메시지 전달 프로토콜은 수직적인 메시지 전달 채널을 제공합니다.
크로스 컨센서스 메시지 전달은 수평적인(파라체인 간) 전송 프로토콜로 생각할 수 있습니다.
전체 크로스 컨센서스 메시지 전달(XCMP) 프로토콜은 아직 개발 중이므로, 수평적인 릴레이 라우팅 메시지 전달(HRMP)은 파라체인을 위한 메시지를 릴레이 체인을 통해 라우팅하는 임시 솔루션을 제공합니다.
수평적인 릴레이 라우팅 메시지 전달(HRMP)은 XCMP가 제품으로 출시될 때 폐기될 예정인 임시 솔루션입니다.

이러한 메시지 전달 프로토콜은 Polkadot 생태계에서 체인 간 통신의 주요 수단이지만, XCM 자체는 이러한 전송 프로토콜에 제한되지 않습니다.
대신, XCM을 사용하여 메시지의 출발지와 목적지에 관계없이 많은 일반적인 유형의 트랜잭션을 표현할 수 있습니다.
예를 들어, 스마트 컨트랙트나 팔레트, 브릿지를 통해 라우팅되거나 Polkadot 생태계의 일부가 아닌 전송 프로토콜을 사용하여 메시지를 구성할 수 있습니다.

![XCM은 메시지 내용을 메시지 전달과 분리합니다](/media/images/docs/xcm-channel-overview.png)

XCM은 수신 시스템에서 수행해야 할 작업을 통신하는 것을 목적으로 특별히 설계되었기 때문에, 일반적인 유형의 트랜잭션에 대한 유연하고 중립적인 트랜잭션 형식을 제공할 수 있습니다.

## XCM 형식의 메시지

XCM 형식을 사용하는 메시지에 대해 이해해야 할 네 가지 중요한 원칙이 있습니다:

- 메시지는 **비동기적**입니다. 메시지를 보낸 후에는 메시지가 전달되거나 실행되었음을 나타내는 응답을 기다릴 필요가 없습니다.
- 메시지는 **절대적**입니다. 메시지는 **순서대로** 전달되고 해석되며 **효율적으로** 실행됩니다.
- 메시지는 **비대칭적**이며 결과를 보낸 사람에게 반환하지 않습니다. 결과를 보낸 사람에게는 추가 메시지를 사용하여 결과를 통신할 수 있습니다.
- 메시지는 **중립적**이며 메시지가 전달되는 컨센서스 시스템 간에 가정을 하지 않습니다.

이러한 기본 원칙을 기반으로 XCM을 사용하여 메시지를 구성할 수 있습니다.
Rust에서 메시지는 다음과 같이 정의됩니다:

```rust
pub struct Xcm<Call>(pub Vec<Instruction<Call>>);
```

이 정의에서 알 수 있듯이, 메시지는 단순히 실행할 명령어의 정렬된 집합을 호출하는 것입니다.
`Instruction` 타입은 enum 데이터 타입이며, 각 변형이 메시지를 구성할 때 일반적으로 사용되는 순서로 정의됩니다.
예를 들어, `WithdrawAsset`는 주문된 명령어 목록에서 `BuyExecution` 또는 `DepositAsset`과 같은 다른 명령어보다 먼저 실행되는 경우가 일반적이므로 첫 번째 변형입니다.

대부분의 XCM 명령어는 자산을 새 위치로 이전하거나 다른 계정에 자산을 예치하는 등 일반적인 작업을 수행할 수 있도록 합니다.
이러한 유형의 작업을 수행하는 명령어를 사용하여 메시지를 일관되게 구성할 수 있으며, 통신하는 컨센서스 시스템이 어떻게 구성되었는지에 관계없이 예상대로 작동합니다.
그러나 명령어를 실행하는 방식을 사용자 정의하거나 `Transact` 명령어를 사용할 수도 있습니다.

`Transact` 명령어를 사용하면 메시지 수신자가 노출한 임의의 호출 가능한 함수를 실행할 수 있습니다.
`Transact` 명령어를 사용하면 수신 시스템의 어떤 함수에도 일반적인 호출을 할 수 있지만, 해당 시스템이 어떻게 구성되었는지에 대한 정보를 알고 있어야 합니다.
예를 들어, 다른 파라체인의 특정 팔레트를 호출하려면 수신 런타임이 올바른 팔레트에 도달하기 위해 어떻게 메시지를 구성해야 하는지 알아야 합니다.
이 정보는 각 체인마다 다를 수 있으며, 모든 런타임은 다르게 구성될 수 있습니다.

## 가상 머신에서의 실행

크로스 컨센서스 가상 머신(XCVM)은 XCM 명령어를 실행하는 XCM 실행 프로그램을 가지고 있는 고수준 가상 머신입니다.
프로그램은 명령어를 순서대로 실행하고 끝까지 실행하거나 오류가 발생하면 실행을 중지합니다.

XCM 명령어가 실행되는 동안 XCVM은 여러 특수 레지스터를 사용하여 내부 상태를 유지합니다.
XCVM은 명령어가 실행되는 기반이 되는 컨센서스 시스템의 상태에도 액세스할 수 있습니다.
수행되는 작업에 따라 XCM 명령어는 레지스터, 컨센서스 시스템의 상태 또는 둘 다를 변경할 수 있습니다.

예를 들어, `TransferAsset` 명령어는 전송할 자산과 자산을 전송할 위치를 지정합니다.
이 명령어가 실행되면 **origin 레지스터**가 자동으로 메시지가 어디에서 왔는지를 나타내고, 이 정보를 통해 전송할 자산을 어디에서 가져와야 하는지를 식별합니다.
XCM 명령어를 실행할 때 조작되는 다른 레지스터는 **holding 레지스터**입니다.
holding 레지스터는 자산을 일시적으로 저장하여 추가적인 명령어가 수행될 때까지 어떻게 처리해야 하는지를 기다리는 데 사용됩니다.

XCVM에는 특정 작업을 처리하기 위한 여러 다른 레지스터가 있습니다.
예를 들어, 수수료의 과대 추정을 저장하는 surplus weight 레지스터와 환불된 과잉 weight의 일부를 저장하는 refunded weight 레지스터가 있습니다.
일반적으로 레지스터에 저장된 값을 직접 수정할 수는 없습니다.
대신, 값은 XCM 실행 프로그램이 시작될 때 설정되고, 특정 명령어에 의해 조작되거나 특정 상황에 따라 또는 특정 규칙에 따라 변경됩니다.
각 레지스터에 저장된 내용에 대한 자세한 정보는 [XCM 참조](/reference/xcm-reference/)를 참조하십시오.

### 구성

Substrate 및 FRAME 기반 체인의 다른 구성 요소와 마찬가지로 XCM 실행 프로그램은 모듈식이며 구성 가능합니다.
`Config` 트레이트를 사용하여 XCM 실행 프로그램의 여러 측면을 구성할 수 있으며, XCM 명령어를 다양한 방식으로 처리하는 구현을 사용자 정의할 수 있습니다.
예를 들어, `Config` 트레이트는 다음과 같은 유형 정의를 제공합니다:

```rust
/// The trait to parameterize the `XcmExecutor`.
pub trait Config {
    /// The outer call dispatch type.
    type Call: Parameter + Dispatchable<PostInfo = PostDispatchInfo> + GetDispatchInfo;
    /// How to send an onward XCM message.
    type XcmSender: SendXcm;
    /// How to withdraw and deposit an asset.
    type AssetTransactor: TransactAsset;
    /// How to get a call origin from a `OriginKind` value.
    type OriginConverter: ConvertOrigin<<Self::Call as Dispatchable>::Origin>;
    /// Combinations of (Location, Asset) pairs trusted as reserves.
    type IsReserve: FilterAssetLocation;
    /// Combinations of (Location, Asset) pairs trusted as teleporters.
    type IsTeleporter: FilterAssetLocation;
    /// Means of inverting a location.
    type LocationInverter: InvertLocation;
    /// Whether to execute the given XCM at all.
    type Barrier: ShouldExecute;
    /// Handler for estimating weight for XCM execution.
    type Weigher: WeightBounds<Self::Call>;
    /// Handler for purchasing weight credit for XCM execution.
    type Trader: WeightTrader;
    /// Handler for the response to a query.
    type ResponseHandler: OnResponse;
    /// Handler for assets in the Holding register after execution.
    type AssetTrap: DropAssets;
    /// Handler for when there is an instruction to claim assets.
    type AssetClaims: ClaimAssets;
    /// Handler version subscription requests.
    type SubscriptionService: VersionChangeNotifier;
}
```

구성 설정과 XCM 명령어 집합(메시지 또는 더 정확히는 수신 시스템에서 실행될 프로그램)은 XCM 실행 프로그램에 입력으로 작용합니다.
XCM 빌더 모듈에서 제공되는 추가 유형 및 함수를 사용하여 XCM 실행 프로그램은 명령어에 포함된 작업을 제공된 순서대로 하나씩 해석하고 실행합니다.
다음 다이어그램은 작업 흐름의 단순화된 개요를 제공합니다.

![XCM 구성 및 실행](/media/images/docs/xcvm-overview.png)

## 위치

XCM은 서로 다른 컨센서스 시스템 간에 통신하기 위한 언어이기 때문에, 일반적이고 유연하며 모호하지 않은 방식으로 위치를 표현할 수 있는 추상적인 방법이 필요합니다.
예를 들어, XCM은 다음과 같은 유형의 활동에 대한 위치를 식별할 수 있어야 합니다:

- 명령어를 실행해야 하는 위치.
- 자산을 인출해야 하는 위치.
- 자산을 수신할 계정이 있는 위치.

이러한 활동 중 어느 것이든 위치는 릴레이 체인, 파라체인, 외부 체인, 특정 체인의 계정, 특정 스마트 컨트랙트 또는 개별 팔레트의 컨텍스트일 수 있습니다.
예를 들어, XCM은 다음과 같은 유형의 위치를 식별할 수 있어야 합니다:

- Polkadot 또는 Kusama 릴레이 체인과 같은 레이어-0 체인.
- Bitcoin 또는 Ethereum 메인넷 또는 파라체인과 같은 레이어-1 체인.
- Ethereum의 ERC-20과 같은 레이어-2 스마트 컨트랙트.
- 파라체인이나 Ethereum의 주소.
- 릴레이 체인이나 파라체인의 계정.
- Frame 기반 Substrate 체인의 특정 팔레트.
- Frame 기반 Substrate 체인의 단일 팔레트 인스턴스.

XCM은 컨센서스 시스템의 컨텍스트 내에서 위치를 설명하기 위해 `MultiLocation` 타입을 사용합니다.
`MultiLocation` 타입은 현재 위치와 관련된 상대적인 경로를 따라 상위 컨센서스 위치에서 `interior` 매개변수를 해석하기 전까지 이동하는 데 사용되는 두 개의 매개변수로 구성됩니다:

- `parents: u8`는 `interior` 매개변수를 해석하기 전에 현재 컨센서스 위치에서 위로 이동할 수 있는 수준의 수를 설명합니다.
- `interior: InteriorMultiLocation`은 `parents` 매개변수를 사용하여 상대적인 경로를 거쳐 상위 컨센서스 시스템 이후의 내부 위치를 설명합니다.

`InteriorMultiLocation`은 **접속점**(junctions) 개념을 사용하여 로컬 컨센서스 시스템 내부의 컨센서스 시스템을 식별합니다.
접속점은 이전 컨센서스 시스템 내부보다 더 내부적인 위치를 지정합니다.
접속점이 없는 `InteriorMultiLocation`은 로컬 컨센서스 시스템을 나타냅니다(Here).
접속점을 사용하여 XCM 명령어를 실행할 때 외부 컨센서스에 상대적인 위치인 파라체인, 계정 또는 팔레트 인스턴스와 같은 내부 컨텍스트를 지정할 수 있습니다.

예를 들어, 다음 매개변수는 릴레이 체인의 컨텍스트에서 고유 식별자 1000을 가진 파라체인을 참조합니다:

```rust
{ "parents": 1,
"interior": { "X1": [{ "Parachain": 1000 }]}
}
```

이 예에서 `parents` 매개변수는 부모 체인으로 한 수준 이동하고, `interior`는 `Parachain` 유형과 인덱스 1000을 가진 내부 위치를 지정합니다.

텍스트에서 MultiLocation은 파일 시스템 경로를 설명하는 데 사용되는 규칙을 따릅니다.
예를 들어, `../PalletInstance(3)/GeneralIndex(42)`로 표현된 MultiLocation은 하나의 부모 (..)와 두 개의 접속점 (`PalletInstance{index: 3}`) 및 (`GeneralIndex{index: 42}`)를 가진 MultiLocation을 설명합니다.

위치 및 접속점을 지정하는 방법에 대한 자세한 내용은 [Universal consensus location identifiers](https://github.com/paritytech/xcm-format#7-universal-consensus-location-identifiers)를 참조하십시오.

## 자산

대부분의 블록체인은 네트워크의 보안에 중요한 역할을 하는 경제적 인센티브를 제공하기 위해 어떤 유형의 디지털 자산에 의존합니다.
일부 블록체인은 단일 네이티브 자산을 지원합니다.
다른 블록체인은 스마트 컨트랙트나 비네이티브 토큰으로 정의된 자산과 같이 여러 자산을 체인 상에서 관리할 수 있습니다.
또한 일부 블록체인은 유일무이한 수집 가능한 비통화 디지털 자산을 지원합니다.
XCM이 이러한 다양한 유형의 자산을 지원하기 위해 자산을 일반적이고 유연하며 모호하지 않은 방식으로 표현할 수 있어야 합니다.

체인 상의 자산을 설명하기 위해 XCM은 `MultiAsset` 타입을 사용합니다.
`MultiAsset` 타입은 자산 식별자와 자산이 편성 가능한지 여부를 지정합니다.
일반적으로 자산 식별자는 구체적인 위치를 사용하여 지정됩니다.
자산이 편성 가능한 경우, 정의에는 금액이 포함됩니다.

추상적인 식별자를 사용하여 자산을 식별하는 것도 가능하지만, 구체적인 식별자는 전역 자산 이름 등록부 없이 자산을 명확하게 식별하는 데 사용됩니다.

구체적인 식별자는 현재 컨텍스트를 기준으로 컨센서스 시스템 내에서 단일 자산을 특정합니다.
그러나 구체적인 자산 식별자는 컨센서스 시스템 간에 그대로 복사할 수 없습니다.
대신, 각 컨센서스 시스템에 상대적인 경로를 사용하여 자산을 이동합니다.
상대 경로는 수신 시스템의 관점에서 읽을 수 있도록 구성되어야 합니다.

Polkadot 릴레이 체인의 DOT과 같은 네이티브 자산의 경우, 자산 식별자는 일반적으로 자산을 발행하는 체인 또는 해당 파라체인의 상위 수준 (`..`)입니다.
자산이 팔레트 내에서 관리되는 경우, 자산 식별자는 팔레트 인스턴스 식별자와 해당 팔레트 내의 인덱스를 사용하여 위치를 지정합니다.
예를 들어, Karura 파라체인은 Statemine 파라체인의 위치 `../Parachain(1000)/PalletInstance(50)/GeneralIndex(42)`에 있는 자산을 참조할 수 있습니다.

위치 및 접속점을 지정하는 방법에 대한 자세한 내용은 [Universal asset identifiers](https://github.com/paritytech/xcm-format#6-universal-asset-identifiers)를 참조하십시오.

## 명령어

대부분의 XCM 명령어는 통신하는 컨센서스 시스템이 어떻게 구성되었는지에 관계없이 예상대로 작동하는 일관된 메시지를 구성할 수 있도록 합니다.
그러나 `Transact` 명령어를 사용하여 메시지 수신자가 노출한 임의의 호출 가능한 함수를 실행할 수도 있습니다.
`Transact` 명령어를 사용하면 수신 시스템의 어떤 함수에도 일반적인 호출을 할 수 있지만, 해당 시스템이 어떻게 구성되었는지에 대한 정보를 알고 있어야 합니다.
예를 들어, 다른 파라체인의 특정 팔레트를 호출하려면 수신 런타임이 올바른 팔레트에 도달하기 위해 어떻게 메시지를 구성해야 하는지 알아야 합니다.
이 정보는 각 체인마다 다를 수 있으며, 모든 런타임은 다르게 구성될 수 있습니다.