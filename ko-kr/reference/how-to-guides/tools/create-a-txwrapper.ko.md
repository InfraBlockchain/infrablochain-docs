---
title: 체인을 위한 txwrapper 생성
description: 체인 사용자들을 위한 오프라인 서명 옵션을 확장하세요.
keywords:
  - 런타임
  - 도구
---

`txwrapper` 패키지를 생성하면 체인 사용자들을 위한 오프라인 서명 옵션을 확장할 수 있습니다.
이는 거래 서명, 구성 및/또는 디코딩을 공기차단 장치로 수행해야 하는 보안에 민감한 사용자들에게 중요합니다.
이에는 (하지만 이에 국한되지 않음) 보관인, 거래소 및 콜드 스토리지 사용자들이 포함됩니다.

자체 체인을 위한 `txwrapper`를 빌드하기 전에 [`txwrapper-examples`](https://github.com/paritytech/txwrapper-core/blob/main/packages/txwrapper-examples/README.md)를 확인해보세요.

Polkadot 예제를 이해하고 `txwrapper-core`의 메서드들을 살펴보세요. (디코딩, `construct.{signingPayload, signedTx, txHash}` 참조)
패키지는 이러한 메서드들을 다시 내보내므로 생성할 공개 API를 이해하는 것이 중요합니다.

## 목표

체인의 `txwrapper` 패키지의 공개 API를 구성합니다.

## 사용 사례

기존 `txwrapper` 사용자들이 새로운 txwrapper를 쉽게 통합할 수 있도록 합니다.

## 단계

### 1. `txwrapper-template`을 사용하여 저장소 생성

[`txwrapper-template`](https://github.com/paritytech/txwrapper-core/tree/main/packages/txwrapper-template) 디렉토리를 작업 저장소로 복사하세요.

이 템플릿은 `NPM`에 게시할 준비가 된 TypeScript 패키지의 기본 구조를 제공합니다. 내보내기는 `balances`, `proxy`, `utility` 팔렛을 적어도 사용하는 FRAME 기반 체인에 관련된 몇 가지 메서드를 보여줍니다.

[`txwrapper-core\`](https://github.com/paritytech/txwrapper-core)가 최상위 수준에서 다시 내보내져 사용자가 해당 도구에 액세스할 수 있도록 합니다.

### 2. package.json 업데이트

다음 필드를 체인 정보에 맞게 수정하세요:

- name
- author
- description
- repository
- bugs
- homepage
- private (false로 표시)

또한 다음 필드를 추가하여 게시 권한을 부여하세요:

```js
  "publishConfig": {
    "access": "public"
  },
```

### 3. 내보낼 관련 메서드 선택

`txwrapper`가 노출할 팔렛 메서드를 선택해야 합니다.
오프라인으로 저장된 키로 서명될 가능성이 높은 메서드를 선택하는 것이 좋습니다.

Substrate 또는 ORML 팔렛에서 메서드가 이미 정의되어 있는지 확인하려면 [txwrapper-substrate](https://github.com/paritytech/txwrapper-core/blob/main/packages/txwrapper-substrate/README.md) 및 [txwrapper-orml](https://github.com/paritytech/txwrapper-core/blob/main/packages/txwrapper-orml/README.md)을 확인하세요.

### 4. getRegistry 메서드 생성

사용자가 최신 체인 유형에 대한 Polkadot-js `TypeRegistry`를 얻을 수 있도록 `getRegistry` 메서드를 내보내야 합니다.

아래의 `foo` 예제는 Polkadot-js 유형과 호환되는 `FRAME` 기반 체인에 적용할 수 있도록 약간의 수정을 거칩니다:

```js
// src/index.ts

import { typesBundleForPolkadot } from '@foo-network/type-definitions';
import { OverrideBundleType } from '@polkadot/types/types';
import {
  getRegistryBase,
  GetRegistryOptsCore,
  getSpecTypes,
  TypeRegistry,
} from '@substrate/txwrapper-core';

// 사용자에게 편의를 위해 하드코딩된 체인 속성을 제공할 수 있습니다.
// 이는 일반적으로 `system_properties` 호출에 의해 반환되지만, 변경되는 경우가 거의 없으므로 하드코딩하는 것이 안전합니다.
/**
 * txwrapper-foo가 지원하는 네트워크의 `ChainProperties`입니다.
 */
const KNOWN_CHAIN_PROPERTIES = {
  foo: {
    ss58Format: 3,
    tokenDecimals: 18,
    tokenSymbol: 'FOO',
  },
  bar: {
    ss58Format: 42,
    tokenDecimals: 18,
    tokenSymbol: 'FOO',
  },
};

// `GetRegistryOptsCore`의 `specName` 속성을 오버라이드하여 더 좁은 유형의 특정성을 얻을 수 있도록 합니다.
/**
 * `getRegistry` 함수의 옵션입니다.
 */
export interface GetRegistryOpts extends GetRegistryOptsCore {
  specName: keyof typeof KNOWN_CHAIN_PROPERTIES;
}

/**
 * txwrapper-foo가 지원하는 네트워크의 타입 레지스트리를 가져옵니다.
 *
 * @param GetRegistryOptions 현재 런타임의 specName, chainName, specVersion 및 metadataRpc
 */
export function getRegistry({
  specName,
  chainName,
  specVersion,
  metadataRpc,
  properties,
}: GetRegistryOpts): TypeRegistry {
  const registry = new TypeRegistry();
  registry.setKnownTypes({
    // 타입이 `OverrideBundleType` 형식으로 패키지화되지 않은 경우,
    // `RegisteredTypes`에서 지원하는 형식 중 하나로 타입을 지정할 수 있습니다:
    // https://github.com/polkadot-js/api/blob/4ff9b51af2c49294c676cc80abc6476565c70b11/packages/types/src/types/registry.ts#L59
    typesBundle: (typesBundleForPolkadot as unknown) as OverrideBundleType,
  });

  return getRegistryBase({
    chainProperties: properties || KNOWN_CHAIN_PROPERTIES[specName],
    specTypes: getSpecTypes(registry, chainName, specName, specVersion),
    metadataRpc,
  });
}
```

그리고 관련된 내보내기를 추가하세요:

```js
// src/methods/currencies/index.ts

// 메서드를 내보내어 `currencies` 네임스페이스 아래에서 사용할 수 있도록 합니다.
export * from "./transfer";
// src/methods

// `methods` 내부에서 `currencies` 네임스페이스를 포함하여 모든 것을 내보내어,
// `methods.currencies.transfer`를 통해 메서드에 액세스할 수 있도록 합니다.
export * as methods from "./methods";
```

### 5. 작동하는 예제 생성

좋은 예제는 사용자의 어려움을 줄이고 유지 보수자의 작업 부담을 줄일 수 있습니다.
체인에 대한 오프라인 거래 생성의 전체 흐름을 명확하게 이해할 수 있는 최종 예제를 만드세요.

1. `template-example.ts`를 체인에 적합한 이름으로 변경하고 파일 내의 모든 TODO 섹션을 업데이트하세요.
2. TODO로 표시된 섹션에서 `examples/README.md`를 업데이트하세요.
3. 개발 노드를 사용하여 예제를 실행할 수 있는지 확인하세요.

### 6. 패키지 게시

버전 관리가 의미가 있는지 확인하고 [로컬에서 패키지가 작동하는지](https://docs.npmjs.com/cli/v6/commands/npm-pack) 확인한 후, [이 가이드](https://docs.npmjs.com/cli/v6/commands/npm-publish)를 참조하여 패키지를 `NPM`에 게시하는 방법을 알아보세요.

## 예제

- [템플릿 예제](https://github.com/paritytech/txwrapper-core/blob/main/packages/txwrapper-template/examples/template-example.ts)

## 리소스

- [`tx-wrapper-polkadot`](https://github.com/paritytech/txwrapper-core/blob/main/packages/) 사용 방법
- `jest`를 사용한 직렬화/역직렬화 [유닛 테스트](https://github.com/paritytech/txwrapper-core/blob/main/packages/txwrapper-orml/src/methods/currencies/transfer.spec.ts)