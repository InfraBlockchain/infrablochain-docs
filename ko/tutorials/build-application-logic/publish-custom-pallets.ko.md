---
title: 사용자 정의 팔레트 게시
description: 사용자 정의 팔레트와 상자를 게시하여 널리 사용 가능하게 하는 방법을 제안합니다.
keywords:
---

블록체인 빌더 또는 파라체인 개발자로서, 특정 프로젝트 목표에 맞게 런타임을 사용자 정의하기 위해 조립할 수 있는 다양한 특수 팔레트 라이브러리에 액세스할 수 있습니다.
현재 사용 가능한 미리 정의된 팔레트의 수를 파악하기 위해 Substrate [FRAME 저장소](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/frame)를 탐색해보세요.
사용 가능한 팔레트의 선택과 해당 사용 사례는 지속적으로 발전하며, 커뮤니티 구성원의 다양한 기여도 포함됩니다.

특정 응용 프로그램 로직을 실행하기 위해 사용자 정의 팔레트를 작성하기 시작하면, 해당 사용자 정의 로직이 다른 팀이나 전반적인 생태계에 도움이 될 수 있다는 것을 알 수 있을 것입니다.
커뮤니티와 공유하고자 하는 사용자 정의 팔레트가 있다면, GitHub나 crates.io와 같은 공개적으로 접근 가능한 포럼에 게시할 수 있습니다.

이 튜토리얼은 사용자 정의 팔레트를 게시하고 다른 개발자들이 사용할 수 있는 오픈 소스 프로젝트로 제공하는 단계를 요약합니다.

## GitHub에 게시하기

GitHub에 팔레트를 게시하려면:

1. [GitHub 저장소를 생성](https://help.github.com/en/articles/create-a-repo)합니다.

   저장소 가시성으로 **Public**을 선택합니다.

2. 팔레트의 모든 소스 코드를 원격 저장소에 [푸시](https://help.github.com/en/articles/pushing-to-a-remote)합니다.

3. 팔레트의 기능과 사용법을 설명하는 [README](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-readmes) 파일을 추가합니다.

4. 필요한 경우 LICENSE, CODE-OF-CONDUCT, CONTRIBUTING 또는 기타 파일을 저장소에 추가합니다.

팔레트를 게시한 후, 다른 개발자들은 다음과 같이 `Cargo.toml` 파일에 팔레트를 포함하여 자신의 런타임에 가져올 수 있습니다:

5. 터미널 쉘을 열고 노드 템플릿의 루트 디렉토리로 이동합니다.

6. 텍스트 편집기에서 `runtime/Cargo.toml` 설정 파일을 엽니다.

7. [dependencies] 섹션을 찾습니다.

8. 다음과 유사한 줄을 추가합니다:

   ```toml
   your-pallet-name = { version = "1.0.0", default-features = false, git = "https://github.com/<your-organization-name>/<your-pallet-repo-name>", branch = "<default-or-specific-branch-name" }
   ```

   팔레트를 가져오기 위해 개발자가 특정 태그나 커밋을 사용하도록 하려면, 이 정보를 README에 포함해야 합니다.

## crates.io에 게시하기

Rust 커뮤니티는 [crates.io](https://crates.io/) 웹사이트를 유지하여 어떤 Rust 모듈이든 허가 없이 게시할 수 있도록 합니다.
[crates.io에 게시하는 방법](https://doc.rust-lang.org/cargo/reference/publishing.html)에 대한 가이드를 따라 절차를 익힐 수 있습니다.

팔레트를 게시한 후, 다른 개발자들은 다음과 같이 `Cargo.toml` 파일에 팔레트를 포함하여 자신의 런타임에 가져올 수 있습니다:

1. 터미널 쉘을 열고 노드 템플릿의 루트 디렉토리로 이동합니다.

2. 텍스트 편집기에서 `runtime/Cargo.toml` 설정 파일을 엽니다.

3. [dependencies] 섹션을 찾습니다.

4. 다음과 유사한 줄을 추가합니다:

   ```toml
   your-pallet-name = { version = "<compatible-version>", default-features = false }
   ```

   팔레트를 crates.io에 게시한 경우, 개발자들은 대상 목적지를 지정할 필요가 없습니다.
   기본적으로 cargo는 crates.io 레지스트리에서 지정된 패키지를 검색합니다.

## 다음 단계

- [런타임 개발](/learn/runtime-development/)
- [사용자 정의 팔레트](/build/custom-pallets/)
- [사용자 정의 팔레트에서 매크로 사용하기](/tutorials/build-application-logic/use-macros-in-a-custom-pallet/)
- [How-to: 팔레트 가져오기](/reference/how-to-guides/basics/import-a-pallet/).