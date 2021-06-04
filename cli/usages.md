### CLI command reference

#### nest new

새(표준 모드) Nest 프로젝트를 만듭니다.

```bash
$ nest new <name> [options]
$ nest n <name> [options]
```

##### Description

새 Nest 프로젝트를 만들고 초기화합니다. 패키지 관리자를 묻는 메시지가 표시됩니다.

- 주어진 `<name>`으로 폴더를 만듭니다.
- 구성 파일로 폴더를 채웁니다.
- 소스 코드(`/src`) 및 end-to-end 테스트 (`/test`)를 위한 하위 폴더 생성
- 앱 구성요소 및 테스트를 위한 기본 파일로 하위 폴더를 채웁니다.

##### Arguments

| Argument | Description                 |
| -------- | --------------------------- |
| `<name>` | 새 프로젝트의 이름 |

##### Options

| Option                                | Description                                                                                                     |
| ------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| `--dry-run`                           | 변경될 사항을 보고하지만 파일 시스템은 변경하지 않습니다. <br/>별칭: `-d`                        |
| `--skip-git`                          | git 저장소 초기화를 건너 뜁니다. <br/>별칭: `-g`                                                            |
| `--skip-install`                      | 패키지 설치를 건너 뜁니다. <br/>별칭: `-s`                                                                     |
| `--package-manager [package-manager]` | 패키지 관리자를 지정하십시오. `npm` 또는 `yarn`을 사용하세요. 패키지 관리자는 전역으로 설치해야 합니다. <br/>별칭: `-p`      |
| `--language [language]`               | 프로그래밍 언어(`TS` 또는 `JS`)를 지정합니다. <br/>별칭: `-l`                                                   |
| `--collection [collectionName]`       | 스키메틱 컬렉션을 지정합니다. 스키메틱이 포함된 설치된 npm 패키지의 패키지 이름을 사용합니다. <br/>별칭: `-c` |

#### nest generate

스키메틱을 기반으로 파일 생성 및/또는 수정

```bash
$ nest generate <schematic> <name> [options]
$ nest g <schematic> <name> [options]
```

##### Arguments

| Argument      | Description                                                                                              |
| ------------- | -------------------------------------------------------------------------------------------------------- |
| `<schematic>` | 생성할 `schematic` 또는 `collection:schematic`입니다. 사용 가능한 스키메틱은 아래 표를 참조하십시오. |
| `<name>`      | 생성된 구성 요소의 이름입니다.                                                                     |

##### Schematics

| Name          | Alias | Description                                                                                         |
| ------------- | ----- | --------------------------------------------------------------------------------------------------- |
| `app`         |       | 모노레포내에서 새 애플리케이션을 생성합니다 (표준 구조인 경우 모노레포로 변환). |
| `library`     | `lib` | 모노레포내에 새 라이브러리를 생성합니다 (표준 구조인 경우 모노레포로 변환).     |
| `class`       | `cl`  | 새 클래스를 생성하십시오.                                                                               |
| `controller`  | `co`  | 컨트롤러 선언을 생성합니다.                                                                  |
| `decorator`   | `d`   | 커스텀 데코레이터를 생성합니다.                                                                        |
| `filter`      | `f`   | 필터 선언을 생성합니다.                                                                      |
| `gateway`     | `ga`  | 게이트웨이 선언을 생성합니다.                                                                     |
| `guard`       | `gu`  | 가드 선언을 생성합니다.                                                                       |
| `interface`   |       | 인터페이스를 생성합니다.                                                                              |
| `interceptor` | `in`  | 인터셉터 선언을 생성합니다.                                                                |
| `middleware`  | `mi`  | 미들웨어 선언을 생성합니다.                                                                  |
| `module`      | `mo`  | 모듈 선언을 생성합니다.                                                                      |
| `pipe`        | `pi`  | 파이프 선언을 생성합니다.                                                                        |
| `provider`    | `pr`  | 프로바이더 선언을 생성합니다.                                                                    |
| `resolver`    | `r`   | 리졸버 선언을 생성합니다.
| `resource`    | `res` | 새 CRUD 리소스를 생성합니다. 자세한 내용은 [CRUD(리소스) 생성기](/recipes/crud-generator)를 참조하세요.                                                                    |
| `service`     | `s`   | 서비스 선언을 생성합니다.                                                                    |

##### Options

| Option                          | Description                                                                                                     |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| `--dry-run`                     | 변경될 사항을 보고하지만 파일 시스템은 변경하지 않습니다. <br/>별칭: `-d`                        |
| `--project [project]`           | 엘리먼트를 추가해야 하는 프로젝트입니다. <br/>별칭: `-p`                                                       |
| `--flat`                        | 엘리먼트에 대한 폴더를 생성하지 마십시오.                                                                       |
| `--collection [collectionName]` | 스키메틱 컬렉션을 지정합니다. 스키메틱이 포함된 설치된 npm 패키지의 패키지 이름을 사용합니다. <br/>별칭: `-c` |
| `--spec`                        | 스펙 파일 생성 적용 (기본값)                                                                         |
| `--no-spec`                     | 스펙 파일 생성 비활성화                                                                                   |

#### nest build

애플리케이션 또는 작업 영역을 출력 폴더로 컴파일합니다.

```bash
$ nest build <name> [options]
```

##### Arguments

| Argument | Description                       |
| -------- | --------------------------------- |
| `<name>` | 빌드할 프로젝트의 이름입니다. |

##### Options

| Option            | Description                                            |
| ----------------- | ------------------------------------------------------ |
| `--path [path]`   | `tsconfig` 파일의 경로입니다. <br/>별칭: `-p`              |
| `--config [path]` | `nest-cli` 구성 파일의 경로입니다. <br/>별칭: `-c` |
| `--watch`         | 감시 모드에서 실행(실시간 새로 고침) <br/>별칭: `-w`       |
| `--webpack`       | 컴파일에는 웹팩을 사용하십시오.                          |
| `--webpackPath`   | 웹팩 구성 경로.                         |
| `--tsc`           | 컴파일에 `tsc`를 강제로 사용합니다.                       |

#### nest start

애플리케이션 (또는 작업 영역의 기본 프로젝트)을 컴파일하고 실행합니다.

```bash
$ nest start <name> [options]
```

##### Arguments

| Argument | Description                     |
| -------- | ------------------------------- |
| `<name>` | 실행할 프로젝트의 이름입니다. |

##### Options

| Option                  | Description                                                                                                          |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `--path [path]`         | `tsconfig` 파일의 경로입니다. <br/>별칭 `-p`                                                                             |
| `--config [path]`       | `nest-cli` 구성 파일의 경로입니다. <br/>별칭 `-c`                                                               |
| `--watch`               | 감시 모드에서 실행(실시간 새로 고침) <br/>별칭 `-w`                                                                      |
| `--preserveWatchOutput` | 화면을 지우는 대신 오래된 콘솔 출력을 감시 모드로 유지합니다. (`tsc` 감시 모드만 해당)                   |
| `--watchAssets`         | 감시 모드 (실시간 새로 고침)에서 실행하여 비 TS 파일(자산)을 관찰합니다. 자세한 내용은 [Assets](cli/monorepo#assets)를 참조하세요. |
| `--debug [hostport]`    | 디버그 모드에서 실행(--inspect 플래그 사용) <br/>별칭 `-d`                                                              |
| `--webpack`             | 컴파일에는 웹팩을 사용하십시오.                                                                                         |
| `--webpackPath`         | 웹팩 구성 경로.                                                                                       |
| `--tsc`                 | 컴파일에 `tsc`를 강제로 사용합니다.                                                                                     |
| `--exec [binary]`       | 실행할 바이너리 (기본값: `node`). <br/>별칭 `-e`                                                                     |

#### nest add

**nest 라이브러리**로 패키지된 라이브러리를 가져와서 설치 스키메틱을 실행합니다.

```bash
$ nest add <name> [options]
```

##### Arguments

| Argument | Description                        |
| -------- | ---------------------------------- |
| `<name>` | 가져올 라이브러리의 이름입니다. |

#### nest update

`package.json` `"dependencies"` 목록의 `@nestjs` 종속성을 `@latest` 버전으로 업데이트합니다.

##### Options

| Option    | Description                                                              |
| --------- | ------------------------------------------------------------------------ |
| `--force` | 업데이트 대신 **업그레이드** 수행<br/>별칭 `-f`                        |
| `--tag`   | 태그가 지정된 버전으로 업데이트 (`@latest`, `@<tag>`등 사용)<br/>별칭 `-t` |

#### nest info

설치된 중첩 패키지 및 기타 유용한 시스템 정보에 대한 정보를 표시합니다. 예를 들면:

```bash
 _   _             _      ___  _____  _____  _     _____
| \ | |           | |    |_  |/  ___|/  __ \| |   |_   _|
|  \| |  ___  ___ | |_     | |\ `--. | /  \/| |     | |
| . ` | / _ \/ __|| __|    | | `--. \| |    | |     | |
| |\  ||  __/\__ \| |_ /\__/ //\__/ /| \__/\| |_____| |_
\_| \_/ \___||___/ \__|\____/ \____/  \____/\_____/\___/

[System Information]
OS Version : macOS High Sierra
NodeJS Version : v8.9.0
YARN Version : 1.5.1
[Nest Information]
microservices version : 6.0.0
websockets version : 6.0.0
testing version : 6.0.0
common version : 6.0.0
core version : 6.0.0
```
