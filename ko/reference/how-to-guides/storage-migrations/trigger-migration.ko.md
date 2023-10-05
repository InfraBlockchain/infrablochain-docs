---
title: 스토리지 마이그레이션 트리거
desciption: Polkadot-JS Apps를 사용하여 스토리지 마이그레이션을 트리거합니다.
keywords:
  - 스토리지 마이그레이션
  - 런타임
  - 고급

이 간단한 가이드는 [Polkadot-JS Apps](https://polkadot.js.org/apps/)를 사용하여 런타임 마이그레이션을 트리거하는 단계를 제시합니다. 마이그레이션 코드가 이미 작성되었고 새로운 런타임이 이미 컴파일되었다고 가정합니다.


## 사용자 정의 타입 추가

`types.json`에 새로운 스토리지 타입을 넣어 UI를 사용하여 마이그레이션을 트리거해야 합니다. JSON에서의 새로운 타입은 다음과 같습니다:

```rust
{
  "Nickname": {
    "first": "BoundedVec<u8>",
    "last": "Option<BoundedVec<u8>>"
  }
}
```
Polkadot-js apps UI에서 설정 > 개발자로 이동하여 types.json에서 사용자 정의 타입을 추가합니다. 파일을 직접 업로드하거나 타입을 UI에 직접 붙여넣을 수 있습니다. 추가하려면 저장하세요.

## spec_version 증가

`spec_version`을 증가시켜 새로운 런타임 버전을 지정합니다.
runtime/src/lib.rs에서 `runtime_version` 매크로를 찾으세요.
```rust
 #[sp_version::runtime_version]
 pub const VERSION: RuntimeVersion = RuntimeVersion {
    //--snip
    spec_version: 100,
    //--snip
 }
```
`spec_version`을 증가시켜 새로운 런타임 버전을 지정하세요.
```rust
    spec_version: 101,
```

## 런타임 업로드

`cargo build --release`로 런타임을 다시 컴파일합니다. 이렇게 하면 블록체인 네트워크에 제출하기에 적합한 더 작은 빌드 아티팩트가 생성됩니다.
WebAssembly 빌드 아티팩트는 target/release/wbuild/node-template-runtime 디렉토리에 있습니다. 예를 들어, 다음과 같은 WebAssembly 아티팩트를 볼 수 있어야 합니다:
```
    node_template_runtime.compact.compressed.wasm
    node_template_runtime.compact.wasm
    node_template_runtime.wasm
```
이제 수정된 런타임 로직을 설명하는 WebAssembly 아티팩트가 있습니다. 업그레이드를 완료하려면 노드를 업그레이드된 런타임을 사용하도록 업데이트하는 트랜잭션을 제출해야 합니다:

`개발자 > 익스트린식`으로 이동하여 관리자 계정을 사용하여 `sudo` 팔렛과 `sudoUncheckedWeight(call, weight)` 함수를 선택합니다.
`system`을 선택하고 호출로 `setCode(code)`를 선택합니다.
파일 업로드를 클릭한 다음, 업데이트된 런타임을 위해 생성한 compact 및 compressed WebAssembly 파일(`node_template_runtime.compact.compressed.wasm`)을 선택하거나 드래그 앤 드롭합니다. (예: `./target/release/wbuild/node-template-runtime/node_template_runtime.compact.wasm`).

트랜잭션을 제출하고 트랜잭션이 블록에 포함된 후에는 노드 템플릿 버전 번호가 런타임 버전이 이제 101임을 나타내는 것을 확인할 수 있습니다.

이전 스토리지를 사용하여 닉네임을 생성한 경우, 이제 체인 상태를 쿼리하여 이 스토리지가 새로운 구조로 업데이트되었음을 확인할 수 있으며, 이제 이름과 성을 가진 닉네임을 생성할 수 있습니다.