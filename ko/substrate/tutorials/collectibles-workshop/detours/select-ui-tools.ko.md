---
title: "프론트엔드 도구 선택"
---

다음 라이브러리들은 [JSON-RPC API](https://github.com/paritytech/jsonrpsee)를 사용하여 애플리케이션이 Substrate 노드와 상호작용할 수 있도록 합니다:

| 이름 | 설명 | 언어 |
| :---- | :----------- | :-------- |
| [Chain API](https://github.com/paritytech/capi) | Substrate 기반 체인과 상호작용하기 위한 TypeScript 툴킷을 제공합니다. 이 툴킷에는 FRAME 유틸리티, 기능적 효과 시스템, 그리고 성능이나 안전성을 저해하지 않으면서 다단계, 다중 체인 상호작용을 용이하게 하는 유창한 API가 포함되어 있습니다. 이 툴킷은 React와 같은 인기있는 프론트엔드 프레임워크와 함께 사용할 수 있습니다. |
| [Polkadot JS API](https://polkadot.js.org/docs/api) | Substrate 기반 체인과 상호작용할 때 노드의 변경에 동적으로 적응할 수 있는 애플리케이션을 구축하기 위한 Javascript 라이브러리를 제공합니다. 이 라이브러리는 React와 같은 인기있는 프론트엔드 프레임워크와 함께 사용할 수 있습니다. | Javascript |
| [Polkadot JS extension](https://polkadot.js.org/docs/extension/) | Polkadot JS API로 구축된 브라우저 확장 프로그램과 프로바이더와 상호작용하기 위한 API를 제공합니다. | Javascript |
| [Substrate Connect](/learn/light-clients-in-substrate-connect/) | 브라우저 내부의 라이트 클라이언트 노드를 사용하여 Substrate 기반 체인에 직접 연결하는 애플리케이션을 구축하기 위한 라이브러리와 브라우저 확장 프로그램을 제공합니다. Substrate Connect를 사용하면 여러 체인에 연결하는 애플리케이션을 구축할 수 있으며, 사용자가 여러 체인과 상호작용하기 위해 애플리케이션을 사용하는 경우에도 일관된 경험을 제공할 수 있습니다. | Javascript |