---
title: polkadot-launch
description: /home/dan/git/docs/content/md/en/docs/reference/account-data-structures.md
keywords:
---

`polkadot-launch` 프로그램을 사용하면 로컬 릴레이 체인과 파라체인 테스트 네트워크를 설정할 수 있습니다.
`polkadot-launch`를 사용하기 전에 `bin` 폴더에 릴레이 체인과 콜레이터의 이진 파일이 있어야 합니다.
릴레이 체인과 콜레이터의 이진 파일은 `polkadot-launch` 저장소와 같은 루트 디렉토리에 있는 `polkadot` 및 `cumulus` 프로젝트의 `rococo-v1` 브랜치를 복제하여 생성할 수 있습니다.
필요한 이진 파일을 복제, 컴파일 및 복사하는 자세한 지침은 `polkadot-launch` 저장소의 [README](https://github.com/paritytech/polkadot-launch#binary-files)를 참조하십시오.

`polkadot-launch` 프로그램은 또한 설정 파일을 제공해야 합니다. 이 설정 파일은 설정하려는 테스트 네트워크의 속성을 정의합니다.
설정 파일은 JSON 또는 JavaScript 파일일 수 있습니다.
`polkadot-launch` 저장소에서 [JSON](https://github.com/paritytech/polkadot-launch/blob/master/config.json) 및 [JavaScript](https://github.com/paritytech/polkadot-launch/blob/master/config.js) 설정 파일의 예제를 찾을 수 있습니다.

## 시작하기 전에

로컬 테스트 네트워크를 설정하려면 `polkadot-launch`를 사용하기 전에 필요한 파일로 환경을 준비해야 합니다:

- `polkadot` 릴레이 체인을 다운로드하고 컴파일한 다음 `polkadot` 이진 파일을 `polkadot-launch/bin` 디렉토리에 복사합니다.
- `polkadot-collator`를 다운로드하고 컴파일한 다음 `polkadot-collator` 이진 파일을 `polkadot-launch/bin` 디렉토리에 복사합니다.
- 테스트 네트워크의 속성을 포함하는 설정 파일을 준비합니다.

이러한 단계를 수행하는 방법에 대한 자세한 내용은 `polkadot-launch` 저장소의 문서를 참조하십시오.

## 설치

`polkadot-launch` 프로그램을 설치하려면 다음 단계를 따르십시오:

1. 컴퓨터에서 터미널 쉘을 엽니다.

2. 다음 명령을 실행하여 `polkadot-launch` 저장소를 복제합니다:

   ```
   git clone https://github.com/paritytech/polkadot-launch.git
   ```

3. 다음 명령을 실행하여 `polkadot-launch` 저장소의 루트 디렉토리로 이동합니다:

   ```
   cd polkadot-launch
   ```

4. 다음 명령 중 하나를 실행하여 `yarn` 또는 `npm`이 설치되어 있는지 확인합니다:

   ```
   yarn --version
   ```

   또는

   ```
   npm --version
   ```

5. 다음 명령 중 하나를 실행하여 polkadot-launch를 설치합니다:

   ```
   yarn global add polkadot-launch
   ```

   또는

   ```
   npm i polkadot-launch -g
   ```

6. 설정 파일의 속성이 릴레이 체인 및 콜레이터 이진 파일의 위치를 반영하는지 확인합니다.

## 기본 명령 사용법

`polkadot-launch` 명령을 실행하는 기본 구문은 다음과 같습니다:

`polkadot-launch <configuration-file> [flag]`

### 플래그

`polkadot-launch` 명령과 함께 다음 선택적 플래그를 사용할 수 있습니다.

| 플래그         | 설명                         |
| ------------- | ----------------------------- |
| -h, --help    | 사용법 정보 표시             |
| -V, --version | 버전 정보 표시               |

`polkadot-launch` 사용에 대한 추가 정보는 `polkadot-launch` 저장소의 [README](https://github.com/paritytech/polkadot-launch)를 참조하십시오.