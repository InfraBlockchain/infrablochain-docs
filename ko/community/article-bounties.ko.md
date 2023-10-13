---
title: 보상을 받기 위한 기사 제출
slug: /v3/contribute/bounties
version: "3.0"
section: docs
category: style guide
keywords:
  - 기여
  - 스타일 가이드
  - 보상
---

> 🚧 경고
>
> 이 섹션은 아직 작업 중입니다. 공식 Substrate 개발자 허브 보상 프로그램은 아직 없지만 곧 도입할 예정입니다.

커뮤니티의 지원과 개발자 생태계에 기여를 장려하기 위해 보상 프로그램을 운영하고 있습니다.
보상 프로그램은 새로운 "어떻게 하는지" 유형의 주제를 다루는 기사를 제출하는 콘텐츠 개발자들에게 XXX 형태의 상금을 제공합니다.

## 참여 방법

프로그램에 참여하려면 다음 단계를 따르세요:

1. Substrate Developer Hub 문서 저장소의 [Issues](https://github.com/substrate-developer-hub/substrate-docs/issues) 페이지에서 **how-to guide**와 **new content** 라벨을 선택하여 표시되는 이슈 목록을 필터링합니다.

1. 가이드가 필요한 관심 있는 이슈를 선택하세요.

예를 들어:

- [다른 팔렛에서 함수 호출하는 방법](https://github.com/substrate-developer-hub/substrate-docs/issues/75)

- [벤치마킹을 사용하여 가중치 계산하는 방법](https://github.com/substrate-developer-hub/substrate-docs/issues/88)

기여하려는 주제에 대한 이슈가 없는 경우, 먼저 해당 주제를 설명하고 적어도 하나의 사용 사례를 포함하는 이슈를 생성하세요. 커뮤니티에 중요한 이슈를 알아내어 우선순위를 정하는 데 도움이 됩니다.

1. [how-to-guide 템플릿](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/contribute-templates/how-to-template.md)을 사용하여 주제에 대한 정보를 구성하세요.

기사에 다음 **필수** 섹션을 포함해야 합니다.

- 개요
- 단계
- 예제

**해당되는 경우**, 기사에는 다음 섹션도 포함되어야 합니다.

- 사용 사례
- 시작하기 전에
- 관련 자료

제출하기 전에, 컨텐츠가 다음 요구 사항을 충족하는지 확인하세요:

- how-to guide 템플릿 구조를 따릅니다.
- 가이드 제목이나 개요 섹션에서 명확한 목표를 명시합니다.
- 명시된 목표를 달성하는 데 초점을 맞춥니다.
- 작동하는 코드나 예제 저장소에 대한 링크를 포함합니다.
- 관련성이 있는 경우 지원하는 참고 자료를 제공합니다.

1. [complexity]와 [category]와 같은 적절한 태그를 추가하세요.

1. [how-to-guides](https://github.com/substrate-developer-hub/substrate-docs/blob/main/content/md/en/docs/reference/how-to-guides/index.md) 저장소에서 브랜치를 생성하세요.

1. 기여하려는 기사에 대해 Pull Request (PR)를 생성하세요.

1. 기사의 PR에 **Bounty submission** 라벨을 추가하세요.

1. 선택한 이슈 또는 생성한 이슈에 기사의 PR 링크를 추가하세요.

제출한 기사는 Pull Request 검토의 일환으로 평가되며 다음 기준에 따라 판단됩니다:

- 유용성. 기사가 다른 기존 자료가 없으며 적어도 하나의 명확한 사용 사례를 제시합니다.

- 구조. [how-to guide 템플릿](https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/contribute-templates/how-to-template.md) 구조와 [contributor guidelines]에서 설명한 규칙을 따릅니다.

- 정확성과 완결성. 각 단계가 명확하게 표현되고 올바르며 완전합니다.

- 재현성. 단계가 일관되게 예상 결과를 얻을 수 있도록 합니다.