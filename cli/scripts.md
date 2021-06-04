### Nest CLI and scripts

이 섹션에서는 DevOps 직원이 개발환경을 관리하는 데 도움이되도록 `nest` 명령이 컴파일러 및 스크립트와 상호 작용하는 방법에 대한 추가 배경 정보를 제공합니다.

Nest 애플리케이션은 실행되기 전에 자바 스크립트로 컴파일되어야 하는 **표준** TypeScript 애플리케이션입니다. 컴파일 단계를 수행하는 방법에는 여러가지가 있으며 개발자/팀은 자신에게 가장 적합한 방법을 자유롭게 선택할 수 있습니다. 이를 염두에 두고 Nest는 다음을 수행하는 즉시 사용할 수 있는 도구 세트를 제공합니다.

- 적절한 기본값으로 "작동"하는 표준 빌드/실행 프로세스를 명령줄에서 제공합니다.
- 빌드/실행 프로세스가 **개방** 상태인지 확인하여 개발자가 기본 기능 및 옵션을 사용하여 기본 도구에 직접 액세스할 수 있도록 합니다.
- 완전한 표준 TypeScript/Node.js 프레임워크를 유지하여 전체 컴파일/배포/실행 파이프 라인을 개발팀이 사용하기로 선택한 외부 도구로 관리할 수 있습니다.

이 목표는 `nest` 명령, 로컬에 설치된 TypeScript 컴파일러 및 `package.json` 스크립트의 조합을 통해 달성됩니다. 아래에서 이러한 기술이 함께 작동하는 방식을 설명합니다. 이는 빌드/실행 프로세스의 각 단계에서 일어나는 일과 필요한 경우 해당 동작을 사용자 정의하는 방법을 이해하는 데 도움이됩니다.

#### The nest binary

`nest` 명령은 OS 레벨 바이너리입니다(즉, OS 명령줄에서 실행 됨). 이 명령은 실제로 아래에 설명된 세가지 영역을 포함합니다. 프로젝트가 스캐폴딩될 때 자동으로 제공되는 `package.json` 스크립트를 통해 빌드(`nest build`) 및 실행(`nest start`) 하위 명령어를 실행하는 것이 좋습니다 (`nest new`를 실행하는 대신 저장소를 복제하여 시작하려면 [typescript starter](https://github.com/nestjs/typescript-starter) 참조).

#### Build

See the [nest build](/cli/usages#nest-build) documentation for more details.
`nest build`는 표준 `tsc` 컴파일러 ([표준 프로젝트](/cli/overview#project-structure)) 또는 웹팩 컴파일러 ([monorepos](/cli/overview#project-structure)). `tsconfig-paths`를 즉시 처리하는 것을 제외하고는 다른 컴파일 기능이나 단계를 추가하지 않습니다. 그 이유는 대부분의 개발자, 특히 Nest로 시작할 때 가끔 까다로울 수 있는 컴파일러 옵션(예: `tsconfig.json` 파일)을 조정할 필요가 없기 때문입니다.

자세한 내용은 [nest build](/cli/usages#nest-build) 문서를 참조하세요.

#### Execution

`nest start`는 단순히 프로젝트가 빌드되었는지 확인한 다음(`nest build`와 동일) 컴파일된 애플리케이션을 실행하기 위한 이식 가능하고 쉬운 방법으로 `node` 명령을 호출합니다. 빌드와 마찬가지로 `nest start` 명령과 해당 옵션을 사용하거나 완전히 대체하여 필요에 따라 이 프로세스를 자유롭게 사용자 지정할 수 있습니다. 전체 프로세스는 표준 TypeScript 애플리케이션 빌드 및 실행 파이프 라인이며 프로세스를 자유롭게 관리할 수 있습니다.

자세한 내용은 [nest start](/cli/usages#nest-start) 문서를 참조하세요.

#### Generation

`nest generate` 명령어는 이름에서 알 수 있듯이 새 Nest 프로젝트 또는 그 안에 구성 요소를 생성합니다.

#### Package scripts

OS 명령 수준에서 `nest` 명령을 실행하려면 `nest` 바이너리를 전역으로 설치해야 합니다. 이것은 npm의 표준 기능이며 Nest의 직접 제어 범위를 벗어납니다. 그 결과 글로벌하게 설치된 `nest` 바이너리가 `package.json`에서 프로젝트 종속성으로 관리되지 **않습니다**. 예를 들어, 두명의 다른 개발자가 `nest` 바이너리의 두가지 버전을 실행할 수 있습니다. 이를 위한 표준 솔루션은 패키지 스크립트를 사용하여 빌드 및 실행 단계에 사용된 도구를 개발 종속성으로 처리할 수 있도록 하는 것입니다.

`nest new`를 실행하거나 [typescript starter](https://github.com/nestjs/typescript-starter)를 복제하면 Nest는 새 프로젝트의 `package.json` 스크립트를 `build` 및 `start`와 같은 명령으로 채웁니다. 또한 기본 컴파일러 도구(예: `typescript`)를 **dev 종속성**으로 설치합니다.

다음과 같은 명령으로 빌드를 실행하고 스크립트를 실행합니다.

```bash
$ npm run build
```

and

```bash
$ npm run start
```

이 명령어는 npm의 스크립트 실행 기능을 사용하여 **로컬에 설치된** `nest` 바이너리를 사용하여 `nest build` 또는 `nest start`를 실행합니다. 이러한 기본 제공 패키지 스크립트를 사용하면 Nest CLI 명령\*에 대한 완전한 종속성 관리가 가능합니다. 즉, 이 **권장** 사용법을 따르면 조직의 모든 구성원이 동일한 버전의 명령을 실행할 수 있습니다.

\*이것은 `build` 및 `start` 명령에 적용됩니다. `nest new` 및 `nest generate` 명령은 빌드/실행 파이프 라인의 일부가 아니므로 다른 컨텍스트에서 작동하며 기본 제공 `package.json` 스크립트와 함께 제공되지 않습니다.

대부분의 개발자/팀의 경우 Nest 프로젝트를 빌드하고 실행하기 위해 패키지 스크립트를 사용하는 것이 좋습니다. 옵션(`--path`, `--webpack`, `--webpackPath`)을 통해 이러한 스크립트의 동작을 완전히 사용자 정의하거나 `tsc` 또는 필요에 따라 웹팩 컴파일러 옵션 파일(예: `tsconfig.json`)을 사용자 정의할 수 있습니다. TypeScript를 컴파일하기 위해 완전히 사용자 지정 빌드 프로세스를 자유롭게 실행할 수도 있습니다 (또는 `ts-node`를 사용하여 TypeScript를 직접 실행할 수도 있습니다).

#### Backward compatibility

Nest 애플리케이션은 순수한 TypeScript 애플리케이션이기 때문에 이전 버전의 Nest 빌드/실행 스크립트는 계속 작동합니다. 업그레이드할 필요는 없습니다. 준비가 되면 새로운 `nest build` 및 `nest start` 명령을 활용하거나 이전 또는 사용자 정의된 스크립트를 계속 실행하도록 선택할 수 있습니다.

#### Migration

변경할 필요는 없지만 `tsc-watch` 또는 `ts-node`와 같은 도구를 사용하는 대신 새 CLI 명령을 사용하여 마이그레이션할 수 있습니다. 이 경우 전역 및 로컬에서 최신 버전의 `@nestjs/cli`를 설치하기만 하면 됩니다.

```bash
$ npm install -g @nestjs/cli
$ cd  /some/project/root/folder
$ npm install -D @nestjs/cli
```

그런 다음 `package.json`에 정의된 `scripts`를 다음과 같이 바꿀 수 있습니다.

```typescript
"build": "nest build",
"start": "nest start",
"start:dev": "nest start --watch",
"start:debug": "nest start --debug --watch",
```
