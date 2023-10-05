---
title: 서브키
description: 서브키 프로그램을 사용하여 키를 생성하고 관리하는 방법에 대한 명령 줄 참조 정보를 제공합니다.
keywords:
---

`subkey` 프로그램은 서브스트레이트 저장소에 포함된 키 생성 및 관리 유틸리티입니다.
`subkey` 프로그램을 사용하여 다음 작업을 수행할 수 있습니다:

- 암호학적으로 안전한 공개 및 개인 키 쌍을 생성하고 검사합니다.
- 비밀 구문 및 원시 시드에서 키를 복원합니다.
- 메시지에 대한 서명을 생성하고 검증합니다.
- 인코딩된 트랜잭션에 대한 서명을 생성하고 검증합니다.
- 계층적 결정론적 자식 키 쌍을 파생합니다.

## 서명 스키마

`subkey` 프로그램은 현재 다음 서명 스키마를 지원합니다:

- [sr25519](https://wiki.polkadot.network/docs/en/learn-cryptography): Ristretto 그룹에서의 Schorr 서명.
- [ed25519](https://en.wikipedia.org/wiki/EdDSA#Ed25519): Curve25519에서의 SHA-512 (SHA-2).
- [secp256k1](https://en.bitcoin.it/wiki/Secp256k1): secp256k1에서의 ECDSA 서명.

서브스트레이트 기반 네트워크에서는 `sr25519` 인코딩된 키가 블록체인과 상호 작용하기 위한 공개 키로 사용됩니다.

## 설치

`subkey`를 전체 서브스트레이트 저장소를 클론하지 않고도 `cargo`를 사용하여 다운로드, 설치 및 컴파일할 수 있습니다.
그러나 `subkey`를 독립 실행형 이진 파일로 설치하려면 환경에 서브스트레이트 빌드 종속성을 추가해야 합니다.
의존성을 사용할 수 있도록 하려면 서브스트레이트 저장소를 클론한 후 `subkey` 이진 파일을 빌드할 수 있습니다.

`subkey` 프로그램을 설치하고 컴파일하려면 다음 단계를 따르세요:

1. 필요한 경우 터미널 셸을 엽니다.

2. 필요한 경우 Rust 컴파일러와 툴체인이 설치되어 있는지 확인합니다.

3. 필요한 경우 다음 명령을 실행하여 서브스트레이트 저장소를 클론합니다:

   ```bash
   git clone https://github.com/paritytech/polkadot-sdk.git
   ```

4. 다음 명령을 실행하여 서브스트레이트 저장소의 루트 디렉토리로 이동합니다:

   ```bash
   cd substrate
   ```

5. 다음 명령을 실행하여 `nightly` 툴체인을 사용하여 `subkey` 프로그램을 빌드합니다:

   ```bash
   cargo +nightly build --package subkey --release
   ```

   프로그램을 컴파일하는 데는 여러 분이 걸릴 수 있습니다.

6. 다음 명령을 실행하여 `subkey` 프로그램을 사용할 준비가 되었는지 확인하고 사용 가능한 옵션에 대한 정보를 확인합니다:

   ```bash
   ./target/release/subkey --help
   ```

## 계층적 결정론적 키

`subkey` 프로그램은 계층적 결정론적 키를 지원합니다.
계층적 결정론적 (HD) 키를 사용하면 부모 시드를 사용하여 계층적 트리 구조에서 자식 키 쌍을 파생시킬 수 있습니다.
이 계층적 구조에서 부모로부터 파생된 각 자식은 고유한 키 쌍을 갖습니다.
파생된 키는 계층적 디렉터리 구조에서 중첩된 디렉터리를 가질 수 있는 파일 시스템과 유사하게 추가적인 자식 키 쌍을 파생시킬 수도 있습니다.
계층적 결정론적 키가 어떻게 파생되는지에 대한 배경 정보는 계층적 결정론적 월렛에 대한 [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) 사양을 참조하십시오.

서브키 명령을 사용하여 계층적 결정론적 키를 파생하는 방법에 대한 정보는 [파생된 키 사용](#파생된-키-사용)을 참조하십시오.

## 기본 명령 사용법

`subkey` 명령을 실행하는 기본 구문은 다음과 같습니다:

```text
subkey [하위 명령] [플래그]
```

지정한 하위 명령에 따라 추가 인수, 옵션 및 플래그가 적용되거나 필요할 수 있습니다.
특정 `subkey` 하위 명령에 대한 사용 정보를 보려면 해당 하위 명령과 `--help` 플래그를 지정하십시오.
예를 들어, `subkey inspect`의 사용 정보를 보려면 다음 명령을 실행할 수 있습니다:

```bash
subkey inspect --help
```

### 플래그

`subkey` 명령과 다음 선택적 플래그를 사용할 수 있습니다.

| 플래그          | 설명                          |
| --------------- | ----------------------------- |
| -h, --help      | 사용 정보를 표시합니다.        |
| -V, --version   | 버전 정보를 표시합니다.        |

### 하위 명령

`subkey` 명령과 다음 하위 명령을 사용할 수 있습니다.
서브키 하위 명령을 사용하는 방법을 설명하는 참조 정보 및 예제를 선택하십시오.

| 명령어                                          | 설명                                                                                                                            |
| ------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------- |
| [`generate`](#subkey-generate)                   | 무작위 계정 키를 생성합니다.                                                                                                    |
| [`generate-node-key`](#subkey-generate-node-key) | 무작위 노드 `libp2p` 비밀 키를 생성합니다. 비밀 키를 파일에 저장하거나 표준 출력 (`stdout`)으로 표시할 수 있습니다. |
| [`help`](#subkey-help)                           | `subkey` 또는 지정된 하위 명령에 대한 사용 정보를 표시합니다.                                                                   |
| [`inspect`](#subkey-inspect)                     | 지정한 비밀 URI에 대한 공개 키와 SS58 주소를 표시합니다.                                                                         |
| [`inspect-node-key`](#subkey-inspect-node-key)   | 지정한 파일 이름에 해당하는 비밀 노드 키와 일치하는 피어 ID를 표시합니다.                                                          |
| [`sign`](#subkey-sign)                           | 지정한 비밀 키로 메시지에 서명합니다.                                                                                            |
| [`vanity`](#subkey-vanity)                       | 원하는 주소를 제공하는 시드를 생성합니다.                                                                                        |
| [`verify`](#subkey-verify)                       | 메시지에 대한 서명이 지정한 공개 또는 비밀 키에 대해 유효한지 확인합니다.                                                          |

### 출력

지정한 하위 명령에 따라 `subkey` 프로그램의 출력에는 다음 정보 중 일부 또는 모두 포함됩니다:

| 이 필드           | 포함 내용                                                                                                                                                                                                                                                 |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 비밀 구문         | 사람이 읽기 쉬운 방식으로 비밀 키를 인코딩하는 영어 단어의 연속입니다. 이 단어의 연속은 또한 기억구문 구문 또는 시드 구문이라고도 하며, 올바른 단어 세트가 올바른 순서로 제공되면 비밀 키를 복구하는 데 사용할 수 있습니다.                                                    |
| 비밀 시드         | 키 쌍을 복원하는 데 필요한 최소한의 정보입니다. 비밀 시드는 때로는 개인 키 또는 원시 시드라고도 합니다. 다른 모든 정보는 이 값을 기반으로 계산됩니다.                                                                                                          |
| 공개 키 (16진수)  | 16진수 형식의 암호키 쌍의 공개 부분입니다.                                                                                                                                                                                                               |
| 공개 키 (SS58)    | SS58 인코딩된 암호키 쌍의 공개 부분입니다.                                                                                                                                                                                                                |
| 계정 ID           | 16진수 형식의 공개 키에 대한 별칭입니다.                                                                                                                                                                                                                  |
| SS58 주소         | 공개 키를 기반으로 한 SS58 인코딩된 공개 주소입니다.                                                                                                                                                                                                      |

### 예제

`subkey` 프로그램의 버전 정보를 표시하려면 다음 명령을 실행합니다:

```bash
subkey --version
```

`subkey verify` 명령의 사용 정보를 표시하려면 다음 명령을 실행합니다:

```bash
subkey verify --help
```

## subkey generate

`subkey generate` 명령을 사용하여 공개 및 개인 키 및 계정 주소를 생성할 수 있습니다.
서명 스키마를 다르게 사용하거나 단어 수가 더 많거나 적은 기억구문을 사용하여 키를 생성하려면 명령줄 옵션을 사용할 수 있습니다.

#### 기본 사용법

```text
subkey generate [flags] [options]
```

#### 플래그

`subkey generate` 명령과 다음 선택적 플래그를 사용할 수 있습니다.

| 플래그 | 설명 |
| ---- | ----------- |
| `-h`, `--help` | 사용 정보를 표시합니다.|
| `--password-interactive` | 터미널에서 대화식으로 키스토어에 액세스하는 비밀번호를 입력할 수 있도록 합니다. |
| `-V`, `--version`| 버전 정보를 표시합니다. |

#### 옵션

`subkey generate` 명령과 다음 명령줄 옵션을 사용할 수 있습니다.

| 옵션 | 설명 |
| ------ | ----------- |
| `--keystore-path <경로>` | 사용자 정의 키스토어 경로를 지정합니다. |
| `--keystore-uri <키스토어-URI>` | 키스토어 서비스에 연결할 사용자 정의 URI를 지정합니다. |
| `-n`, `--network <네트워크>` | 사용할 네트워크 주소 형식을 지정합니다. 예를 들어, `kusama` 또는 `polkadot`입니다. 지원되는 네트워크의 전체 목록은 온라인 사용 정보를 참조하십시오. |
| `--output-type <형식>` | 사용할 출력 형식을 지정합니다. 유효한 값은 Json 및 Text입니다. 기본 출력 형식은 Text입니다. |
| `--password <비밀번호>` | 키스토어에서 사용하는 비밀번호를 지정합니다. 이 옵션을 사용하면 시드에 추가적인 비밀을 추가할 수 있습니다. |
| `--password-filename <경로>` | 키스토어에서 사용하는 비밀번호가 포함된 파일의 이름을 지정합니다. |
| `--scheme <스키마>` | 생성하는 키의 암호 스키마를 지정합니다. 유효한 값은 `Ecdsa`, `Ed25519`, `Sr25519`입니다. 기본 스키마는 `Sr25519`입니다. |
| `-w`, `--words <단어>` | 생성하는 키의 비밀 구문에 포함되는 단어 수를 지정합니다. 유효한 값은 12, 15, 18, 21, 24입니다. 기본적으로 비밀 구문은 12개의 단어로 구성됩니다. |

#### 예제

sr25519 서명 스키마를 사용하는 새로운 키 쌍을 생성하려면 다음 명령을 실행합니다:

```bash
subkey generate
```

다음과 유사한 출력이 표시됩니다. 12개의 단어로 구성된 비밀 구문이 포함됩니다:

```text
Secret phrase:       bread tongue spell stadium clean grief coin rent spend total practice document
  Secret seed:       0xd5836897dc77e6c87e5cc268abaaa9c661bcf19aea9f0f50a1e149d21ce31eb7
  Public key (hex):  0xb6a8b4b6bf796991065035093d3265e314c3fe89e75ccb623985e57b0c2e0c30
  Account ID:        0xb6a8b4b6bf796991065035093d3265e314c3fe89e75ccb623985e57b0c2e0c30
  Public key (SS58): 5GCCgshTQCfGkXy6kAkFDW1TZXAdsbCNZJ9Uz2c7ViBnwcVg
  SS58 Address:      5GCCgshTQCfGkXy6kAkFDW1TZXAdsbCNZJ9Uz2c7ViBnwcVg
```

`subkey` 프로그램은 사용되는 네트워크에 따라 공개/개인 키 쌍과 연결된 주소를 다르게 인코딩합니다.
Kusama 및 Polkadot 네트워크에서 **동일한 개인 키**를 사용하려면 `--network` 옵션을 사용하여 Kusama 및 Polkadot 네트워크에 대한 별도의 주소 형식을 생성할 수 있습니다.
공개 키는 동일하지만 주소 형식은 네트워크별로 다릅니다.
특정 네트워크에 대한 키 쌍을 생성하려면 다음과 유사한 명령을 실행합니다:

```bash
subkey generate --network picasso
```

이 명령은 동일한 필드를 출력하지만 지정한 네트워크에 대한 주소 형식을 사용합니다.

보다 안전한 키 쌍을 생성하려면 `ed25519` 서명 스키마와 24개의 단어로 구성된 비밀 구문을 사용하여 `moonriver` 네트워크에 대한 키 쌍을 생성할 수 있습니다.
다음 명령을 실행합니다:

```bash
subkey generate --scheme ed25519 --words 24 --network moonriver
```

이 명령은 동일한 필드를 출력하지만 Ed25519 서명 스키마, 24개의 단어로 구성된 비밀 구문 및 `moonriver` 네트워크의 주소 형식을 사용합니다.

```text
Secret phrase:       cloth elevator sadness twice arctic adjust axis vendor grant angle face section key safe under fee fine garage pupil hotel museum valve popular motor
  Secret seed:       0x5fa5923c1d6753fa30f268ffd363efb730ca0db906f55bc17efe65cd24f92097
  Public key (hex):  0x3f1da4d35489e3d84739de1490f51b567ad2a62793cca1357e624fbfa534fc85
  Account ID:        0x3f1da4d35489e3d84739de1490f51b567ad2a62793cca1357e624fbfa534fc85
  Public key (SS58): VkFLVqcighJnssSbL4LDSGy5ShJQZvYhm7G8K8W1ZXt96Z5VG
  SS58 Address:      VkFLVqcighJnssSbL4LDSGy5ShJQZvYhm7G8K8W1ZXt96Z5VG
```

비밀번호로 보호되는 키 쌍을 생성하려면 다음과 유사한 명령을 실행합니다. `--password` 옵션과 비밀번호를 사용하여 명령줄 및 비밀번호 문자열에 비밀번호를 추가할 수 있습니다.

```bash
subkey generate --password "pencil laptop kitchen cutter"
```

비밀번호가 필요한 키를 생성한 후에는 명령줄에 `--password` 옵션과 비밀번호 문자열을 포함하여 비밀번호를 검색하거나 비밀 구문 끝에 세 개의 슬래시 (`///`)를 추가하여 검색할 수 있습니다.
비밀번호, 비밀 구문 및 비밀 시드를 안전하게 보관하고 안전한 위치에 백업하는 것이 중요합니다.

## subkey generate-node-key

`subkey generate-node-key` 명령을 사용하여 서브스트레이트 노드 간의 피어 투 피어 (`libp2p`) 통신을 위한 무작위 공개 및 개인 키를 생성할 수 있습니다.
공개 키는 체인 사양 파일이나 명령줄 인수에서 노드를 식별하는 데 사용되는 피어 식별자입니다.
대부분의 경우 이 명령은 비밀 키를 파일에 저장하기 위해 명령줄 옵션과 함께 실행됩니다.

#### 기본 사용법

```text
subkey generate-node-key [flags] [options]
```

#### 플래그

`subkey generate-node-key` 명령과 다음 선택적 플래그를 사용할 수 있습니다.

| 플래그              | 설명                          |
| ----------------- | ----------------------------- |
| `-h`, `--help`    | 사용 정보를 표시합니다.        |
| `-V`, `--version` | 버전 정보를 표시합니다.        |

#### 옵션

`subkey generate-node-key` 명령과 다음 명령줄 옵션을 사용할 수 있습니다.

| 옵션  | 설명 |
| ------- | ----------- |
| `--file <파일-이름>` | 로컬 노드에 대한 생성된 비밀 키를 저장할 파일 위치를 지정합니다. 이 옵션을 지정하지 않으면 생성된 키가 표준 출력 (`stdout`)으로 표시됩니다. |

#### 예제

피어 간 통신을 위한 무작위 키 쌍을 생성하고 비밀 키를 파일에 저장하려면 다음과 유사한 명령을 실행합니다:

```bash
subkey generate-node-key --file ../generated-node-key
```

이 명령은 터미널에서 노드 키에 대한 피어 식별자를 표시하고 개인 키가 `generated-node-key` 파일에 저장됩니다.
이 예제에서는 현재 작업 디렉토리가 아닌 상위 디렉토리에 키가 저장됩니다.

```text
12D3KooWHALHfL7dDBiGTt4JTEAvCbDWts8zHwvcPvJXDF9fxue7
```

## subkey help

`subkey help` 명령을 사용하여 `subkey` 또는 지정된 하위 명령에 대한 사용 메시지를 표시합니다.

#### 기본 사용법

```text
subkey help [subcommand]
```

#### 예제

`verify` 하위 명령에 대한 사용 정보를 표시하려면 다음 명령을 실행합니다:

```bash
subkey help verify
```

## subkey inspect

`subkey inspect` 명령을 사용하여 지정된 비밀 키 또는 기억 구문에 대한 공개 키와 공개 주소를 다시 계산할 수 있습니다.

#### 기본 사용법

```text
subkey inspect [flags] [options] uri
```

#### 플래그

`subkey inspect` 명령과 다음 선택적 플래그를 사용할 수 있습니다.

| 플래그 | 설명 |
| ---- | ----------- |
| `-h`, `--help` | 사용 정보를 표시합니다. |
| `--password-interactive` | 터미널에서 대화식으로 키스토어에 액세스하는 비밀번호를 입력할 수 있도록 합니다. |
| `--public` | 검사할 `uri`가 16진수로 인코딩된 공개 키임을 나타냅니다. |
| `-V`, `--version` | 버전 정보를 표시합니다. |

#### 옵션

`subkey inspect` 명령과 다음 명령줄 옵션을 사용할 수 있습니다.

| 옵션 | 설명 |
| ------ | ----------- |
| `--keystore-path <경로>` | 사용자 정의 키스토어 경로를 지정합니다. |
| `--keystore-uri <키스토어-URI>` | 키스토어 서비스에 연결할 사용자 정의 URI를 지정합니다. |
| `-n`, `--network <네트워크>` | 사용할 네트워크 주소 형식을 지정합니다. 예를 들어, `kusama` 또는 `polkadot`입니다. 지원되는 네트워크의 전체 목록은 온라인 사용 정보를 참조하십시오. |
| `--output-type <형식>` | 사용할 출력 형식을 지정합니다. 유효한 값은 Json 및 Text입니다. 기본 출력 형식은 Text입니다. |
| `--password <비밀번호>` | 키스토어에서 사용하는 비밀번호를 지정합니다. 이 옵션을 사용하면 시드에 추가적인 비밀을 추가할 수 있습니다. |
| `--password-filename <경로>` | 키스토어에서 사용하는 비밀번호가 포함된 파일의 이름을 지정합니다. |
| `--scheme <스키마>` | 검사하는 키의 암호 스키마를 지정합니다. 유효한 값은 `Ecdsa`, `Ed25519`, `Sr25519`입니다. 기본 스키마는 `Sr25519`입니다. |

#### 인수

`subkey inspect` 명령에는 다음 필수 인수를 지정해야 합니다.

| 인수 | 설명 |
| -------- | ----------- |
| `uri` | 검사할 키 URI를 지정합니다. 비밀 구문, 비밀 시드 (파생 경로 및 비밀번호 포함), SS58 주소, 공개 키 또는 16진수로 인코딩된 공개 키를 사용하여 키를 지정할 수 있습니다. `uri`를 파일 이름으로 지정하면 파일 내용이 URI로 사용됩니다. |

#### 예제

기억 구문에서 파생된 공개 키를 검사하려면 다음과 유사한 명령을 실행합니다:

```bash
subkey inspect "caution juice atom organ advance problem want pledge someone senior holiday very"
```

다음과 유사한 출력이 표시됩니다:

```text
Secret phrase `caution juice atom organ advance problem want pledge someone senior holiday very` is account:
  Secret seed:       0xc8fa03532fb22ee1f7f6908b9c02b4e72483f0dbd66e4cd456b8f34c6230b849
  Public key (hex):  0xd6a3105d6768e956e9e5d41050ac29843f98561410d3a47f9dd5b3b227ab8746
  Public key (SS58): 5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR
  Account ID:        0xd6a3105d6768e956e9e5d41050ac29843f98561410d3a47f9dd5b3b227ab8746
  SS58 Address:      5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR
```

비밀 시드에서 파생된 공개 키를 검사하려면 다음과 유사한 명령을 실행합니다:

```bash
subkey inspect 0xc8fa03532fb22ee1f7f6908b9c02b4e72483f0dbd66e4cd456b8f34c6230b849
```

비밀 구문이 포함된 텍스트 파일 (예: `my-secret-key`)에 비밀 구문이 저장된 경우 명령줄에서 파일 이름을 지정하여 파일의 내용을 전달하고 해당 비밀 구문 또는 비밀 시드와 연결된 공개 키를 표시할 수 있습니다.
예를 들어, 다음과 유사한 명령을 실행할 수 있습니다:

```bash
subkey inspect my-secret-key
```

16진수로 인코딩된 공개 키를 사용하여 공개 키를 검사하려면 다음과 유사한 명령을 실행합니다:

```bash
subkey inspect --public 0xd6a3105d6768e956e9e5d41050ac29843f98561410d3a47f9dd5b3b227ab8746
```

이 경우 명령은 다음과 유사한 공개 정보만 표시합니다:

```text
Network ID/version: substrate
  Public key (hex):   0xd6a3105d6768e956e9e5d41050ac29843f98561410d3a47f9dd5b3b227ab8746
  Account ID:         0xd6a3105d6768e956e9e5d41050ac29843f98561410d3a47f9dd5b3b227ab8746
  Public key (SS58):  5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR
  SS58 Address:       5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR
```

`subkey` 프로그램은 사용되는 네트워크에 따라 공개/개인 키 쌍과 연결된 주소를 다르게 인코딩합니다.
Kusama 및 Polkadot 네트워크에서 **동일한 개인 키**를 사용하려면 `--network` 옵션을 사용하여 특정 네트워크에 대한 주소를 검사할 수 있습니다.
공개 키는 동일하지만 주소 형식은 네트워크별로 다릅니다.
특정 네트워크에 대한 키 쌍을 검사하려면 다음과 유사한 명령을 실행합니다:

```bash
subkey inspect --network kusama "caution juice atom organ advance problem want pledge someone senior holiday very"
```

이 명령의 출력에서 비밀 구문, 비밀 시드 및 공개 키는 동일하지만 Kusama 네트워크에 대한 주소는 다음과 같습니다:

```text
  SS58 Address:      HRkCrbmke2XeabJ5fxJdgXWpBRPkXWfWHY8eTeCKwDdf4k6
```

동일한 개인 키에 대한 Polkadot 네트워크의 주소를 검사하려면 다음과 유사한 명령을 실행합니다:

```bash
subkey inspect --network polkadot "caution juice atom organ advance problem want pledge someone senior holiday very"
```

이 명령의 출력에서 비밀 구문, 비밀 시드 및 공개 키는 Kusama 네트워크와 동일하지만 Polkadot 네트워크의 주소는 다음과 같습니다:

```text
  SS58 Address:      15rRgsWxz4H5LTnNGcCFsszfXD8oeAFd8QRsR6MbQE2f6XFF
```

`--password` 옵션과 비밀번호를 지정하여 비밀번호로 보호된 키를 검사하려면 다음과 유사한 명령을 실행합니다:

```bash
subkey inspect "caution juice atom organ advance problem want pledge someone senior holiday very" --password "pencil laptop kitchen cutter"
```

명령줄에서 `--password` 옵션과 비밀번호를 지정하면 명령 출력에 사용된 비밀번호가 _표시되지 않습니다_.

```text
Secret phrase `caution juice atom organ advance problem want pledge someone senior holiday very` is account:
  Secret seed:       0xdfc5d5d5235a37fdc907ee1cb720299f96aeb02f9c7c2fcad7ee8c7bfbd2a4db
  Public key (hex):  0xdef8f78b123475265815b65a7c55e105e1ab185f4969954f68d92b7bb67a1045
  Public key (SS58): 5H74SqH1iQCWh5Gumyghh1WJMcmM6TdBHYSK7mKVJbv9NuSK
  Account ID:        0xdef8f78b123475265815b65a7c55e105e1ab185f4969954f68d92b7bb67a1045
  SS58 Address:      5H74SqH1iQCWh5Gumyghh1WJMcmM6TdBHYSK7mKVJbv9NuSK
```

비밀번호로 보호된 키를 검사하려면 비밀 구문 끝에 `///`와 비밀번호를 추가하여 다음과 유사한 명령을 실행할 수도 있습니다.

```bash
subkey inspect "caution juice atom organ advance problem want pledge someone senior holiday very///pencil laptop kitchen cutter"
```

이 경우 명령 출력에 사용된 비밀번호가 표시됩니다. 예를 들어:



시크릿 키 URI `caution juice atom organ advance problem want pledge someone senior holiday very///pencil laptop kitchen cutter`는 다음과 같은 계정입니다:
  시크릿 시드:       0xdfc5d5d5235a37fdc907ee1cb720299f96aeb02f9c7c2fcad7ee8c7bfbd2a4db
  공개 키 (16진수):  0xdef8f78b123475265815b65a7c55e105e1ab185f4969954f68d92b7bb67a1045
  공개 키 (SS58): 5H74SqH1iQCWh5Gumyghh1WJMcmM6TdBHYSK7mKVJbv9NuSK
  계정 ID:        0xdef8f78b123475265815b65a7c55e105e1ab185f4969954f68d92b7bb67a1045
  SS58 주소:      5H74SqH1iQCWh5Gumyghh1WJMcmM6TdBHYSK7mKVJbv9NuSK
```

## subkey inspect-node-key

`subkey inspect-node-key` 명령을 사용하여 지정된 파일 이름의 노드 키와 해당하는 노드의 피어 식별자를 표시합니다.
이 명령을 사용하기 전에 이전에 `subkey generate-node-key` 명령을 사용하고 키를 파일에 저장해야 합니다.

#### 기본 사용법

```text
subkey inspect-node-key [flags] [options] --file <file-name>
```

#### 플래그

`subkey inspect-node-key` 명령과 함께 다음 선택적 플래그를 사용할 수 있습니다.

| 플래그             | 설명                         |
| ----------------- | ----------------------------- |
| `-h`, `--help`    | 사용법 정보를 표시합니다.   |
| `-V`, `--version` | 버전 정보를 표시합니다. |

#### 옵션

`subkey inspect-node-key` 명령과 함께 다음 명령줄 옵션을 사용할 수 있습니다.

| 옵션 | 설명 |
| ------ | ----------- |
| `-n, --network <network>` | 사용할 네트워크 주소 형식을 지정합니다. 예를 들어, `kusama` 또는 `polkadot`입니다. 지원되는 네트워크의 전체 목록은 온라인 사용 정보를 참조하십시오. |

#### 인수

`subkey inspect-node-key` 명령과 함께 다음 필수 인수를 지정해야 합니다.

| 인수 | 설명 |
| -------- | ----------- |
| `--file <file-name>` | 노드와의 피어 간 통신을 위해 생성된 비밀 키를 포함하는 파일을 지정합니다. |

## subkey sign

`subkey sign` 명령을 사용하여 메시지에 서명합니다. 메시지는 표준 입력(`stdin`)으로 전달됩니다.
비밀 시드 또는 비밀 문구를 사용하여 메시지에 서명할 수 있습니다.

#### 기본 사용법

```text
subkey sign [flags] [options]
```

#### 플래그

`subkey sign` 명령과 함께 다음 선택적 플래그를 사용할 수 있습니다.

| 플래그 | 설명 |
| ---- | ----------- |
| `-h`, `--help` | 사용법 정보를 표시합니다. |
| `--hex` | 표준 입력으로 지정한 메시지가 16진수로 인코딩된 메시지임을 나타냅니다. |
| `--password-interactive` | 터미널에서 키스토어에 액세스하기 위한 비밀번호를 대화식으로 입력할 수 있도록 합니다. |
| `-V`, `--version` | 버전 정보를 표시합니다. |

#### 옵션

`subkey sign` 명령과 함께 다음 명령줄 옵션을 사용할 수 있습니다.

| 옵션 | 설명 |
| ------ | ----------- |
| `--keystore-path <path>` | 사용자 정의 키스토어 경로를 지정합니다. |
| `--keystore-uri <keystore-uri>` | 키스토어 서비스에 연결하기 위한 사용자 정의 URI를 지정합니다. |
| `--message <network>` | 서명할 메시지 문자열을 지정합니다. |
| `--password <password>` | 키스토어에서 사용하는 비밀번호를 지정합니다. 이 옵션을 사용하면 시드에 추가 비밀을 추가할 수 있습니다. |
| `--password-filename <path>` | 키스토어에서 사용하는 비밀번호가 포함된 파일의 이름을 지정합니다. |
| `--scheme <scheme>` | 키의 암호 서명 스키마를 지정합니다. 유효한 값은 `Ecdsa`, `Ed25519`, `Sr25519`입니다. 기본 스키마는 `Sr25519`입니다. |
| `--suri <secret-seed>` | 메시지에 서명하기 위해 사용할 비밀 키 URI를 지정합니다. 비밀 문구, 비밀 시드(유도 경로 및 비밀번호 포함)를 사용하여 키를 지정할 수 있습니다. `--suri` 옵션에 파일 이름을 지정하면 파일 내용이 URI로 사용됩니다. 이 옵션을 생략하면 URI를 입력하라는 메시지가 표시됩니다. |

#### 예제

다음 예제는 `echo` 명령을 사용하여 테스트 메시지를 `subkey sign` 명령의 표준 입력으로 파이프합니다.
터미널에서 텍스트 메시지에 서명하려면 다음과 유사한 명령을 실행할 수 있습니다:

```bash
echo "test message" | subkey sign --suri 0xc8fa03532fb22ee1f7f6908b9c02b4e72483f0dbd66e4cd456b8f34c6230b849
```

명령 출력은 메시지에 대한 서명을 표시합니다.
예를 들면:

```text
f052504de653a5617c46eeb1daa73e2dbbf625b6bf8f16d9d8de6767bc40d91dfbd38c13207f8a03594221c9f68c00a158eb3120311b80ab2da563b82a995b86
```

16진수로 인코딩된 메시지에 서명하려면 다음과 유사한 명령을 실행합니다:

```bash
subkey sign --hex --message 68656c6c6f2c20776f726c64 --suri 0xc8fa03532fb22ee1f7f6908b9c02b4e72483f0dbd66e4cd456b8f34c6230b849
```

명령 출력은 메시지에 대한 서명을 표시합니다.
예를 들면:

```text
9ae07defc0ddb752651836c25ac643fbdf9d45ba180ec6d09e4423ff6446487a52b609d69c06bd1c3ec09b3d06a43f019bacba12dc5a5697291c5e9faab13288
```

## subkey vanity

`subkey vanity` 명령을 사용하여 지정된 문자열 패턴을 포함하는 주소를 생성합니다.
이 명령은 사용자 정의 주소에 대한 비밀 문구를 생성하지 않습니다.

#### 기본 사용법

```text
subkey vanity [flags] [options] --pattern <pattern>
```

#### 플래그

`subkey vanity` 명령과 함께 다음 선택적 플래그를 사용할 수 있습니다.

| 플래그              | 설명                         |
| ----------------- | ----------------------------- |
| `-h`, `--help`    | 사용법 정보를 표시합니다.   |
| `-V`, `--version` | 버전 정보를 표시합니다. |

#### 옵션

`subkey vanity` 명령과 함께 다음 명령줄 옵션을 사용할 수 있습니다.

| 옵션 | 설명 |
| ------ | ----------- |
| `-n, --network <network>` | 사용할 네트워크 주소 형식을 지정합니다. 예를 들어, `kusama` 또는 `polkadot`입니다. 지원되는 네트워크의 전체 목록은 온라인 사용 정보를 참조하십시오. |
| `--output-type <format>`  | 사용할 출력 형식을 지정합니다. 유효한 값은 Json 및 Text입니다. 기본 출력 형식은 Text입니다. |
| `--scheme <scheme>` | 키의 암호 서명 스키마를 지정합니다. 유효한 값은 `Ecdsa`, `Ed25519`, `Sr25519`입니다. 기본 스키마는 `Sr25519`입니다. |

#### 인수

`subkey vanity` 명령과 함께 다음 필수 인수를 지정해야 합니다.

| 인수 | 설명 |
| -------- | ----------- |
| `--pattern <pattern>` | 생성된 주소에 포함할 문자열을 지정합니다. |

#### 예제

지정한 패턴에 따라 사용자 정의 문자열을 포함하는 주소를 생성하려면 다음과 유사한 명령을 실행할 수 있습니다:

```bash
subkey vanity --network kusama --pattern DUNE
```

명령은 다음과 유사한 출력을 표시합니다:

```text
Generating key containing pattern 'DUNE'
100000 keys searched; best is 187/237 complete
200000 keys searched; best is 189/237 complete
300000 keys searched; best is 221/237 complete
400000 keys searched; best is 221/237 complete
500000 keys searched; best is 221/237 complete
600000 keys searched; best is 221/237 complete
best: 237 == top: 237
Secret Key URI `0x82737756075d15409053afd19a6b29ae2abeed96a3487d71d2af9b3eff19cbfa` is account:
  Secret seed:       0x82737756075d15409053afd19a6b29ae2abeed96a3487d71d2af9b3eff19cbfa
  Public key (hex):  0xe025cc93383436f61f067ff918ec632d0933c2d81da3bc1fbc27c9d33579bc40
  Account ID:        0xe025cc93383436f61f067ff918ec632d0933c2d81da3bc1fbc27c9d33579bc40
  Public key (SS58): HeDUNE7vd4cYtwHadXBWTgYrsKGQXZ5xFLVbgPVo71X1ccF
  SS58 Address:      HeDUNE7vd4cYtwHadXBWTgYrsKGQXZ5xFLVbgPVo71X1ccF
```

키 쌍이 생성된 후 SS58 주소와 공개 키 모두 사용자 정의 문자열 `DUNE`을 포함합니다.

## subkey verify

`subkey verify` 명령을 사용하여 공개 또는 비밀 키를 사용하여 메시지의 서명을 확인합니다.

#### 기본 구문

```text
subkey verify [flags] [options] <signature> <uri>
```

#### 플래그

`subkey verify` 명령과 함께 다음 선택적 플래그를 사용할 수 있습니다.

| 플래그 | 설명 |
| ---- | ----------- |
| `-h`, `--help` | 사용법 정보를 표시합니다. |
| `--hex` | 표준 입력으로 지정한 메시지가 16진수로 인코딩된 메시지임을 나타냅니다. |
| `-V`, `--version` | 버전 정보를 표시합니다. |

#### 옵션

`subkey verify` 명령과 함께 다음 명령줄 옵션을 사용할 수 있습니다.

| 옵션 | 설명 |
| ------ | ----------- |
| `--message <message>` | 확인할 메시지를 지정합니다. |
| `--scheme <scheme>` | 키의 암호 서명 스키마를 지정합니다. 유효한 값은 `Ecdsa`, `Ed25519`, `Sr25519`입니다. 기본 스키마는 `Sr25519`입니다. |

#### 인수

`subkey verify` 명령과 함께 다음 필수 인수를 지정해야 합니다.

| 인수 | 설명 |
| -------- | ----------- |
| `<signature>` | 확인할 16진수로 인코딩된 서명을 지정합니다. |
| `<uri>` | 메시지를 확인하기 위해 사용할 공개 또는 비밀 키 URI를 지정합니다. `uri`에 파일 이름을 지정하면 파일 내용이 URI로 사용됩니다. 이 옵션을 생략하면 URI를 입력하라는 메시지가 표시됩니다. |

#### 예제

다음 예제는 `echo` 명령을 사용하여 테스트 메시지를 `subkey verify` 명령의 표준 입력으로 파이프합니다.

```bash
echo "test message" | subkey verify f052504de653a5617c46eeb1daa73e2dbbf625b6bf8f16d9d8de6767bc40d91dfbd38c13207f8a03594221c9f68c00a158eb3120311b80ab2da563b82a995b86 5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR
```

메시지 서명이 확인되면 명령 출력은 서명을 확인한다는 메시지를 표시합니다.
예를 들면:

```text
Signature verifies correctly.
```

16진수로 인코딩된 메시지의 서명을 확인하려면 다음과 유사한 명령을 실행합니다:

```bash
subkey verify --hex --message 68656c6c6f2c20776f726c64 4e9d84c9d67241f916272c3f39cd145d847cfeed322b3a4fcba67e1113f8b21440396cb7624113c14af2cd76850fc8445ec538005d7d39ce664e5fb0d926a48f 5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR
```

메시지 서명이 확인되면 명령 출력은 서명을 확인한다는 메시지를 표시합니다.
예를 들면:

```text
Signature verifies correctly.
```

## 파생 키 작업

Substrate에서 계층적 결정론적 파생 키는 파생 방법에 따라 하드 키 또는 소프트 키로 분류됩니다.
예를 들어, 하드 키는 부모 **비밀 키**와 유도 경로를 사용하여만 파생할 수 있습니다.
부모 공개 키는 하드 키를 파생할 수 없습니다.

소프트 키는 부모 비밀 키 또는 부모 **공개 키**와 유도 경로를 사용하여 파생할 수 있습니다.
소프트 키는 부모 공개 키를 사용하여 파생할 수 있으므로 부모 시드를 노출하지 않고도 부모 키를 식별하는 데 사용할 수 있습니다.
파생된 하드 키 또는 소프트 키와 관련된 주소를 사용하여 루트 키로 서명한 메시지와 동일한 보안으로 메시지에 서명할 수 있습니다.

### 하드 키 파생

하드 자식 키 쌍을 파생하기 위해 부모 키와 유도 경로, 인덱스 뒤에 두 개의 슬래시(`//`)를 추가합니다.
이전에 생성된 키에서 자식 키 쌍과 주소를 파생하기 때문에 `subkey inspect` 명령을 사용합니다.
예를 들어:

```bash
subkey inspect "caution juice atom organ advance problem want pledge someone senior holiday very//derived-hard-key//0"
```

명령은 다음과 유사한 출력을 표시합니다:

```text
Secret Key URI `caution juice atom organ advance problem want pledge someone senior holiday very//derived-hard-key//0` is account:
  Secret seed:       0x667fe31c1d1d8f00811aa0163001b5b3055b26f11e82ae17e28668d0e08ced51
  Public key (hex):  0xd61bbc562fc43d43d80a3372a25c52e4aa862bbfdbb4aa1a5ec86f042f787f24
  Account ID:        0xd61bbc562fc43d43d80a3372a25c52e4aa862bbfdbb4aa1a5ec86f042f787f24
  Public key (SS58): 5GuSLtVEbYgYT9Q78CX99RSSPuHjsAyAadwC1GmweDvTvFTZ
  SS58 Address:      5GuSLtVEbYgYT9Q78CX99RSSPuHjsAyAadwC1GmweDvTvFTZ
```

### 소프트 키 파생

부모 비밀 키에서 소프트 자식 키 쌍을 파생하기 위해 비밀 문구와 유도 경로, 인덱스 뒤에 하나의 슬래시(`/`)를 추가합니다.
이전에 생성된 키에서 새로운 키 쌍과 주소를 파생하기 때문에 `subkey inspect` 명령을 사용합니다.
예를 들어:

```bash
subkey inspect "caution juice atom organ advance problem want pledge someone senior holiday very/derived-soft-key/0"
```

명령은 다음과 유사한 출력을 표시합니다:

```text
Secret Key URI `caution juice atom organ advance problem want pledge someone senior holiday very/derived-soft-key/0` is account:
  Secret seed:       n/a
  Public key (hex):  0x8826cc3730441dc4b67ea118997780db878ce7848c1548a9d36624ca39cf7c2c
  Account ID:        0x8826cc3730441dc4b67ea118997780db878ce7848c1548a9d36624ca39cf7c2c
  Public key (SS58): 5F9DtrPk3SaFs6U6S8HxnuJcoQ2jF8Wdt3ygwbbBnbVcsdiC
  SS58 Address:      5F9DtrPk3SaFs6U6S8HxnuJcoQ2jF8Wdt3ygwbbBnbVcsdiC
```

부모 공개 키에서 소프트 자식 키 쌍을 파생하기 위해 부모 공개 주소를 사용할 수도 있습니다.
부모 공개 주소와 유도 경로, 인덱스를 구분하는 단일 슬래시(`/`)를 사용합니다.
예를 들어:

```bash
subkey inspect "5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR/derived-public/0"
```

명령은 다음과 유사한 출력을 표시합니다:

```text
Public Key URI `5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR/derived-public/0` is account:
  Network ID/version: substrate
  Public key (hex):   0xee3792a82ba43fc503bcbdabd7d090df71496a43928e25f87843f1d9d40e8a14
  Account ID:         0xee3792a82ba43fc503bcbdabd7d090df71496a43928e25f87843f1d9d40e8a14
  Public key (SS58):  5HT3mAg8NnpgeZFUsM9aUYVdS5iq6ZWVUoNcTDiuqVvjkgKR
  SS58 Address:       5HT3mAg8NnpgeZFUsM9aUYVdS5iq6ZWVUoNcTDiuqVvjkgKR
```

동일한 유도 경로와 인덱스를 사용하는 경우 부모 비밀 키 또는 부모 공개 주소를 사용하여 소프트 자식 키는 동일합니다.
유도 경로(`derived-soft-key`에서 `derived-public`으로) 또는 인덱스(`0`에서 `1`로)를 변경하면 다른 주소로 다른 자식 키를 파생시킵니다.
예를 들어:

```bash
subkey inspect "5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR/derived-soft-key/1"
```

명령은 다음과 유사한 출력을 표시합니다:

```text
Public Key URI `5Gv8YYFu8H1btvmrJy9FjjAWfb99wrhV3uhPFoNEr918utyR/derived-soft-key/1` is account:
  Network ID/version: substrate
  Public key (hex):   0x688d5a9761bc2705efd3ff0171a535e97de12aab59659177efae453873b18673
  Account ID:         0x688d5a9761bc2705efd3ff0171a535e97de12aab59659177efae453873b18673
  Public key (SS58):  5ERnpynLaQweDhrBQLe3vz8aWYodKYEeJ92xsbnpgG7GhHvo
  SS58 Address:       5ERnpynLaQweDhrBQLe3vz8aWYodKYEeJ92xsbnpgG7GhHvo
```

### 유도 경로와 비밀번호 결합

기본적으로 비밀 시드는 비밀번호로 보호되지 않습니다.
파생된 키에 추가 보호를 위해 비밀번호를 추가할 수 있습니다.
그러나 비밀 시드에서 파생된 키 쌍은 비밀 번호를 사용하여 파생된 키 쌍과 다릅니다.
동일한 비밀 시드는 다른 유도 경로나 비밀번호를 사용하면 다른 키를 파생시킵니다.
키 쌍을 보호하기 위해 비밀 시드 문구에 비밀번호를 추가할 수 있습니다.
비밀 시드 문구와 비밀번호 모두 필요합니다.

하드 키의 자식으로 소프트 키를 파생할 수 있습니다.
이렇게 하면 하드 키의 공개 주소와 선택적 비밀번호를 사용하여 새로운 공개 주소를 파생할 수 있습니다.
파생된 하드 키로 소프트 키를 파생하기 위해 하드 키의 공개 주소를 사용합니다.
파생 경로와 비밀번호를 추가합니다.

```bash
subkey inspect "5D8JkugWWMDmQ4h2yUuBLWwQXaBi2nBdiDmY4DR7hW76QmuW/0"
```

명령은 다음과 유사한 출력을 표시합니다:

```text
Public Key URI `5D8JkugWWMDmQ4h2yUuBLWwQXaBi2nBdiDmY4DR7hW76QmuW/0` is account:
  Network ID/version: substrate
  Public key (hex):   0xcaf1f5ec14b507b5c365c6528cc06de74a5615274694a0c895ed4109c0ff0d32
  Public key (SS58):  5GeoQa3nkeNmzZSfgBFuK3BkAggnTHcX3S1j94sffJYYphrP
  Account ID:         0xcaf1f5ec14b507b5c365c6528cc06de74a5615274694a0c895ed4109c0ff0d32
  SS58 Address:       5GeoQa3nkeNmzZSfgBFuK3BkAggnTHcX3S1j94sffJYYphrP
```

이 전략을 사용하여 하드 키와 소프트 키를 결합하면 부모 공개 주소와 소프트 파생 경로를 노출하지 않고도 비밀 문구나 비밀번호를 노출하지 않고도 파생된 주소를 모두 제어할 수 있습니다.

### 사전 정의된 계정과 키

Substrate에는 로컬 개발 환경에서 테스트에 사용할 수 있는 여러 사전 정의된 계정이 포함되어 있습니다.
이러한 사전 정의된 계정은 모두 동일한 시드를 사용하여 단일 비밀 문구를 생성한 키에서 파생됩니다.
모든 사전 정의된 계정의 키를 검사하려면 유도 경로를 사용합니다.
예를 들어:

```bash
subkey inspect //Alice
```

명령은 다음과 유사한 출력을 표시합니다:

```text
Secret Key URI `//Alice` is account:
  Secret seed:       0xe5be9a5092b81bca64be81d212e7f2f9eba183bb7a90954f7b76361f6edb5c0a
  Public key (hex):  0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d
  Account ID:        0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d
  Public key (SS58): 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
  SS58 Address:      5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
```

`//Alice`와 `//alice`는 서로 다른 유도 경로이며, 사전 정의된 계정의 비밀 문구와 유도 경로는 실제로 다음과 같습니다:

```text
bottom drive obey lake curtain smoke basket hold race lonely fit walk//Alice
```

다음 명령을 실행하여 키가 일치하는지 확인할 수 있습니다:

```bash
subkey inspect "bottom drive obey lake curtain smoke basket hold race lonely fit walk//Alice"
```

명령 출력은 다음과 같습니다:

```text
Secret Key URI `bottom drive obey lake curtain smoke basket hold race lonely fit walk//Alice` is account:
  Secret seed:       0xe5be9a5092b81bca64be81d212e7f2f9eba183bb7a90954f7b76361f6edb5c0a
  Public key (hex):  0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d
  Account ID:        0xd43593c715fdd31c61141abd04a99fd6822c8558854ccde39a5684e7a56da27d
  Public key (SS58): 5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
  SS58 Address:      5GrwvaEF5zXb26Fz9rcQpDWS57CtERHpNehXCPcNoHGKutQY
```

## 추가 리소스

- [Subkey README](https://github.com/paritytech/polkadot-sdk/tree/master/substrate/bin/utils/subkey).
- [PolkadotJS Apps UI](https://polkadot.js.org/apps/).
- [암호화 설명서](https://wiki.polkadot.network/docs/en/learn-cryptography).