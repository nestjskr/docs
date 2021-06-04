### Workspaces

Nest에는 코드 구성을 위한 두가지 모드가 있습니다.

- **표준 모드**: 자체 종속성 및 설정이 있고 모듈 공유를 위해 최적화하거나 복잡한 빌드를 최적화할 필요가 없는 개별 프로젝트 중심 애플리케이션을 빌드하는 데 유용합니다. 이것이 기본 모드입니다.
- **모노레포 모드**: 이 모드는 코드 아티팩트(artifact)를 경량 **모노레포**의 일부로 취급하며 개발자 팀 및/또는 다중 프로젝트 환경에 더 적합할 수 있습니다. 빌드 프로세스의 일부를 자동화하여 모듈식 구성요소를 쉽게 만들고 구성하고, 코드 재사용을 촉진하고, 통합 테스트를 더 쉽게 만들고, `tslint` 규칙 및 기타 구성 정책과 같은 프로젝트 전체 아티팩트를 쉽게 공유할 수 있도록 합니다. github 하위 모듈과 같은 대안보다 사용하기 쉽습니다. Monorepo 모드는 `nest-cli.json` 파일에 표시되는 **workspace** 개념을 사용하여 monorepo 구성요소간의 관계를 조정합니다.

사실상 Nest의 모든 기능은 코드 구성 모드와 무관하다는 점에 유의해야 합니다. 이 선택의 **유일한** 영향은 프로젝트 구성방식과 빌드 아티팩트 생성방식입니다. CLI에서 핵심 모듈, 추가 모듈에 이르는 다른 모든 기능은 두 모드에서 동일하게 작동합니다.

또한 언제든지 ** 표준 모드 **에서 ** 모노 레포 모드 **로 쉽게 전환 할 수 있으므로 하나 또는 다른 접근 방식의 이점이 더 명확해질 때까지이 결정을 지연 할 수 있습니다.

#### Standard mode

`nest new`를 실행하면 내장 스키메틱을 사용하여 새 **프로젝트**가 생성됩니다. Nest는 다음을 수행합니다.

1. `nest new`에 제공한 `name` 인수에 해당하는 새 폴더를 만듭니다.
2. 최소 기본 수준 Nest 애플리케이션에 해당하는 기본 파일로 해당 폴더를 채웁니다. 이러한 파일은 [typescript-starter](https://github.com/nestjs/typescript-starter) 저장소에서 확인할 수 있습니다.
3. 애플리케이션 컴파일, 테스트 및 제공을 위한 다양한 도구를 구성하고 활성화하는 `nest-cli.json`, `package.json` 및 `tsconfig.json`과 같은 추가 파일을 제공합니다.

여기에서 시작 파일을 수정하고, 새 구성요소를 추가하고, 종속성(예: `npm install`)을 추가하고, 이 문서의 나머지 부분에서 설명하는대로 애플리케이션을 개발할 수 있습니다.

#### Monorepo mode

모로레포 모드를 활성화하려면 _standard mode_ 구조로 시작하고 **프로젝트**를 추가합니다. 프로젝트는 전체 **애플리케이션** (`nest generate app` 명령으로 작업 공간에 추가) 또는 **라이브러리**(`nest generate library` 명령으로 작업 공간에 추가)일 수 있습니다. 아래에서 이러한 특정 유형의 프로젝트 구성요소에 대해 자세히 설명합니다. 지금 주목해야 할 요점은 기존 표준 모드 구조에 **프로젝트를 추가하는 작업**이라는 점에서 모노레포 모드로 **변환**하는 것입니다. 예를 살펴 보겠습니다.

실행하면:

```bash
nest new my-project
```

다음과 같은 폴더 구조로 _standard mode_ 구조를 구성했습니다.

<div class="file-tree">
  <div class="item">node_modules</div>
  <div class="item">src</div>
  <div class="children">
    <div class="item">app.controller.ts</div>
    <div class="item">app.module.ts</div>
    <div class="item">app.service.ts</div>
    <div class="item">main.ts</div>
  </div>
  <div class="item">nest-cli.json</div>
  <div class="item">package.json</div>
  <div class="item">tsconfig.json</div>
  <div class="item">tslint.json</div>
</div>

다음과 같이 이를 모로레포 모드 구조로 변환할 수 있습니다.

```bash
cd my-project
nest generate app my-app
```

이 시점에서 `nest`는 기존 구조를 **모노레포 모드** 구조로 변환합니다. 이로 인해 몇가지 중요한 변경사항이 발생합니다. 이제 폴더 구조는 다음과 같습니다.

<div class="file-tree">
  <div class="item">apps</div>
    <div class="children">
      <div class="item">my-app</div>
      <div class="children">
        <div class="item">src</div>
        <div class="children">
          <div class="item">app.controller.ts</div>
          <div class="item">app.module.ts</div>
          <div class="item">app.service.ts</div>
          <div class="item">main.ts</div>
        </div>
        <div class="item">tsconfig.app.json</div>
      </div>
      <div class="item">my-project</div>
      <div class="children">
        <div class="item">src</div>
        <div class="children">
          <div class="item">app.controller.ts</div>
          <div class="item">app.module.ts</div>
          <div class="item">app.service.ts</div>
          <div class="item">main.ts</div>
        </div>
        <div class="item">tsconfig.app.json</div>
      </div>
    </div>
  <div class="item">nest-cli.json</div>
  <div class="item">package.json</div>
  <div class="item">tsconfig.json</div>
  <div class="item">tslint.json</div>
</div>

`generate app` 스키메틱은 코드를 재구성했습니다. 각 **application** 프로젝트를 `apps` 폴더 아래로 이동하고 각 프로젝트의 루트 폴더에 프로젝트별 `tsconfig.app.json` 파일을 추가합니다. 우리의 원래 `my-project` 앱은 모노레포의 **기본 프로젝트**가 되었으며 이제 `apps` 폴더 아래에 방금 추가된 `my-app`과 동료가 되었습니다. 아래에서 기본 프로젝트를 다룰 것입니다.

> error **경고** 표준 모드 구조를 모로레포로 변환하는 것은 표준 Nest 프로젝트 구조를 따르는 프로젝트에서만 작동합니다. 특히 변환 중에 스키메틱은 루트의 `apps` 폴더 아래에 있는 프로젝트 폴더의 `src` 및 `test` 폴더를 재배치하려고 시도합니다. 프로젝트에서 이 구조를 사용하지 않으면 변환이 실패하거나 신뢰할 수 없는 결과가 생성됩니다.

#### Workspace projects

모노레포는 작업 공간의 개념을 사용하여 구성원 엔티티를 관리합니다. 작업 공간은 **프로젝트**로 구성됩니다. 프로젝트는 다음중 하나일 수 있습니다.

- **애플리케이션**: 애플리케이션을 부트 스트랩하기 위한 `main.ts` 파일을 포함하는 전체 Nest 애플리케이션. 컴파일 및 빌드 고려 사항 외에도 작업 영역 내의 애플리케이션 유형 프로젝트는 _standard mode_ 구조내의 애플리케이션과 기능적으로 동일합니다.
- **라이브러리**: 라이브러리는 다른 프로젝트 내에서 사용할 수 있는 범용 기능 집합(모듈, 프로바이더, 컨트롤러 등)을 패키징하는 방법입니다. 라이브러리는 자체적으로 실행할 수 없으며 `main.ts` 파일이 없습니다. 라이브러리에 대한 자세한 내용은 [여기](/cli/libraries)를 참조하세요.

모든 작업 공간에는 **기본 프로젝트**(애플리케이션 유형 프로젝트여야 함)가 있습니다. 이것은 기본 프로젝트의 루트를 가리키는 `nest-cli.json` 파일의 최상위 `"root"` 속성으로 정의됩니다([CLI 속성](/cli/monorepo#cli-properties) 참조). 자세한 내용은 아래 참조). 일반적으로 이것은 당신이 시작한 **표준 모드** 애플리케이션이며 나중에 `nest generate app`을 사용하여 모노레포로 변환됩니다. 이 단계를 수행하면 이 속성이 자동으로 채워집니다.

기본 프로젝트는 프로젝트 이름이 제공되지 않은 경우 `nest build`및 `nest start`와 같은 `nest` 명령어에서 사용됩니다.

예를 들어, 위의 모로레포 구조에서

```bash
$ nest start
```

`my-project` 앱이 시작됩니다. `my-app`을 시작하려면 다음을 사용합니다.

```bash
$ nest start my-app
```

#### Applications

애플리케이션 유형 프로젝트 또는 비공식적으로 "애플리케이션"이라고 부르는 것은 실행 및 배포할 수 있는 완전한 Nest 애플리케이션입니다. `nest generate app`을 사용하여 애플리케이션 유형 프로젝트를 생성합니다.

이 명령어는 [typescript starter](https://github.com/nestjs/typescript-starter)의 표준 `src` 및 `test` 폴더를 포함한 프로젝트 스켈레톤을 자동으로 생성합니다. 표준 모드와 달리 모로레포의 애플리케이션 프로젝트에는 패키지 종속성(`package.json`)이나 `.prettierrc` 및 `tslint.json`과 같은 기타 프로젝트 구성 아티팩트가 없습니다. 대신 모노레포 전체 종속성 및 구성 파일이 사용됩니다.

그러나 스키메틱은 프로젝트의 루트 폴더에 프로젝트별ㅍ`tsconfig.app.json` 파일을 생성합니다. 이 구성 파일은 컴파일 출력 폴더를 올바르게 설정하는 것을 포함하여 적절한 빌드 옵션을 자동으로 설정합니다. 이 파일은 최상위(monorepo) `tsconfig.json` 파일을 확장하므로 전역 설정을 단일 저장소 전체에서 관리할 수 있지만 필요한 경우 프로젝트 수준에서 재정의할 수 있습니다.

#### Libraries

언급했듯이 라이브러리 유형 프로젝트 또는 단순히 "라이브러리"는 실행하기 위해 애플리케이션으로 구성되어야하는 Nest 구성요소의 패키지입니다. `nest generate library`를 사용하여 라이브러리 유형 프로젝트를 생성합니다. 라이브러리에 속한 것을 결정하는 것은 건축 설계 결정입니다. [라이브러리](/cli/libraries) 장에서 라이브러리에 대해 자세히 설명합니다.

#### CLI properties

Nest는 `nest-cli.json` 파일에 표준 및 단일 저장소 구조화된 프로젝트를 구성, 빌드 및 배포하는 데 필요한 메타 데이터를 보관합니다. Nest는 프로젝트를 추가할 때 이 파일을 자동으로 추가하고 업데이트하므로 일반적으로 파일에 대해 생각하거나 내용을 편집할 필요가 없습니다. 그러나 수동으로 변경할 수 있는 몇가지 설정이 있으므로 파일을 전체적으로 이해하는 것이 좋습니다.

위의 단계를 실행하여 모노레포를 만든 후 `nest-cli.json` 파일은 다음과 같습니다.

```javascript
{
  "collection": "@nestjs/schematics",
  "sourceRoot": "apps/my-project/src",
  "monorepo": true,
  "root": "apps/my-project",
  "compilerOptions": {
    "webpack": true,
    "tsConfigPath": "apps/my-project/tsconfig.app.json"
  },
  "projects": {
    "my-project": {
      "type": "application",
      "root": "apps/my-project",
      "entryFile": "main",
      "sourceRoot": "apps/my-project/src",
      "compilerOptions": {
        "tsConfigPath": "apps/my-project/tsconfig.app.json"
      }
    },
    "my-app": {
      "type": "application",
      "root": "apps/my-app",
      "entryFile": "main",
      "sourceRoot": "apps/my-app/src",
      "compilerOptions": {
        "tsConfigPath": "apps/my-app/tsconfig.app.json"
      }
    }
  }
}
```

파일은 섹션으로 나뉩니다.

- 표준 및 단일 저장소 전체 설정을 제어하는 최상위 속성이 있는 전역 섹션
- 각 프로젝트에 대한 메타데이터가 포함된 최상위 속성(`"projects"`) 이 섹션은 모노레포 모드 구조에만 존재합니다.

최상위 속성은 다음과 같습니다.

- `"collection"`: 구성요소를 생성하는 데 사용되는 스키메틱 모음의 지점. 일반적으로 이 값을 변경해서는 안됩니다.
- `"sourceRoot"`: 표준 모드 구조의 단일 프로젝트에 대한 소스 코드의 루트 또는 모노레포 모드 구조의 _기본 프로젝트_ 를 가리 킵니다.
- `"compilerOptions"`: 컴파일러 옵션을 지정하는 키와 옵션 설정을 지정하는 값이 있는 맵. 아래 세부정보를 참조하십시오
- `"generateOptions"`: 전역 생성 옵션을 지정하는 키와 옵션 설정을 지정하는 값이있는 맵. 아래 세부 정보를 참조하십시오
- `"monorepo"`: (모노레포 전용) 모노레포 모드 구조의 경우 이 값은 항상 `true`입니다.
- `"root"`: (모노레포만 해당) _기본 프로젝트_ 의 프로젝트 루트를 가리킵니다.

#### Global compiler options

이러한 속성은 사용할 컴파일러뿐만 아니라 `nest build` 또는 `nest start`의 일부로, 컴파일러에 상관없이 `tsc` 또는 webpack에 관계없이 **모든** 컴파일 단계에 영향을 미치는 다양한 옵션을 지정합니다.

| 프러퍼티 이름      | 프러퍼티 값 유형 | 설명                                                                                                                                                                                                                          |
| ------------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `webpack`           | boolean             | `true` 인 경우 [webpack compiler](https://webpack.js.org/)를 사용합니다. `false`이거나 없는 경우 `tsc`를 사용합니다. 모노레포 모드에서 기본값은 `true`(웹팩 사용)이고, 표준 모드에서 기본값은 `false`('tsc'사용)입니다. 자세한 내용은 아래를 참조하십시오. |
| `tsConfigPath`      | string              | (**모노레포 전용**) `project` 옵션없이 `nest build` 또는 `nest start`가 호출될 때 사용되는 `tsconfig.json` 설정이 포함된 파일을 가리킵니다(예: 기본 프로젝트가 구축 또는 시작).        |
| `webpackConfigPath` | string              | 웹팩 옵션 파일을 가리킵니다. 지정하지 않으면 Nest는 `webpack.config.js` 파일을 찾습니다. 자세한 내용은 아래를 참조하십시오.                                                                                                      |
| `deleteOutDir`      | boolean             | `true`이면 컴파일러가 호출될 때마다 먼저 컴파일 출력 디렉토리를 제거합니다 (기본값은 `./dist` 인 `tsconfig.json`에 구성된대로).                                                                 |
| `assets`            | array               | 컴파일 단계가 시작될 때마다 TypeScript가 아닌 자산(asset)을 자동으로 배포할 수 있습니다 (자산 배포는 `--watch` 모드의 증분 컴파일에서 발생하지 **않습니다**.). 자세한 내용은 아래를 참조하십시오.                                |
| `watchAssets`       | boolean             | `true`인 경우 감시 모드에서 실행하여 **모든** 비 TypeScript 자산을 감시합니다. (감시할 자산을 보다 세밀하게 제어하려면 아래의 [Assets](cli/monorepo#assets) 섹션을 참조하십시오).                                                        |

#### Global generate options

이러한 속성은 `nest generate` 명령에서 사용할 기본 생성 옵션을 지정합니다.

| 프러퍼티 이름 | 프러퍼티 값 유형 |  설명                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ------------- | ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `spec`        | boolean _or_ object | 값이 부울인 경우 `true` 값은 기본적으로 `spec` 생성을 활성화하고 `false` 값은 비활성화합니다. CLI 명령줄에 전달된 플래그는 프로젝트 별 `generateOptions` 설정(아래 참조)과 마찬가지로 이 설정을 재정의합니다. 값이 객체인 경우 각 키는 회로도 이름을 나타내고 부울 값은 해당 특정 스키메틱에 대해 기본 사양 생성을 활성화/비활성화할지 여부를 결정합니다.|

다음 예에서는 부울 값을 사용하여 모든 프로젝트에 대해 기본적으로 사양 파일 생성을 사용하지 않도록 지정합니다.

```javascript
{
  "generateOptions": {
    "spec": false
  },
  ...
}
```

다음 예제에서 `spec` 파일 생성은 `service` 스키메틱에 대해서만 비활성화됩니다 (예: `nest generate service...`).

```javascript
{
  "generateOptions": {
    "spec": {
      "service": false
    }
  },
  ...
}
```

> warning **경고** `spec`을 객체로 지정할 때 생성 스키메틱의 키는 현재 자동 별칭 처리를 지원하지 않습니다. 이는 예를 들어 `service: false`와 같은 키를 지정하고 `s` 별칭을 통해 서비스를 생성하려고 시도해도 사양이 계속 생성된다는 것을 의미합니다. 일반 스키메틱 이름과 별칭이 모두 의도한대로 작동하는지 확인하려면 아래와 같이 일반 명령 이름과 별칭을 모두 지정합니다.
>
> ```javascript
> {
>   "generateOptions": {
>     "spec": {
>       "service": false,
>       "s": false
>     }
>   },
>   ...
> }
> ```

#### Project-specific generate options

전역 생성 옵션을 제공하는 것 외에도 프로젝트별 생성 옵션을 지정할 수도 있습니다. 프로젝트별 생성 옵션은 전역 생성 옵션과 정확히 동일한 형식을 따르지만 각 프로젝트에서 직접 지정됩니다.

프로젝트별 생성 옵션은 전역 생성 옵션보다 우선합니다.

```javascript
{
  "projects": {
    "cats-project": {
      "generateOptions": {
        "spec": {
          "service": false
        }
      },
      ...
    }
  },
  ...
}
```

> warning **경고** 생성 옵션의 우선 순위는 다음과 같습니다. CLI 명령줄에 지정된 옵션은 프로젝트별 옵션보다 우선합니다. 프로젝트별 옵션은 전역 옵션보다 우선합니다.

#### Specified compiler

기본 컴파일러가 다른 이유는 더 큰 프로젝트(예: monorepo에서 더 일반적)의 경우 웹팩이 빌드 시간과 모든 프로젝트 구성요소를 함께 묶는 단일 파일을 생성하는 데 상당한 이점을 가질 수 있기 때문입니다. 개별 파일을 생성하려면 `"webpack"`을 `false`로 설정하십시오. 그러면 빌드 프로세스가 `tsc`를 사용하게됩니다.

#### Webpack options

웹팩 옵션 파일에는 표준 [webpack 구성 옵션](https://webpack.js.org/configuration/)이 포함될 수 있습니다. 예를 들어, webpack에 `node_modules`(기본적으로 제외됨)를 번들로 지정하려면 `webpack.config.js`에 다음을 추가하십시오.

```javascript
module.exports = {
  externals: [],
};
```

웹팩 구성 파일은 JavaScript 파일이므로 기본 옵션을 사용하고 수정된 객체를 반환하는 함수를 노출할 수도 있습니다.

```javascript
module.exports = function(options) {
  return {
    ...options,
    externals: [],
  };
};
```

#### Assets

TypeScript 컴파일은 컴파일러 출력(`.js` 및 `.d.ts` 파일)을 지정된 출력 디렉토리에 자동으로 배포합니다. `.graphql` 파일, `images`, `.html` 파일 및 기타 자산과 같은 비 TypeScript 파일을 배포하는 것도 편리할 수 있습니다. 이를 통해 `nest build`(및 모든 초기 컴파일 단계)를 가벼운 **개발 빌드** 단계로 취급할 수 있습니다. 여기서 TypeScript가 아닌 파일을 편집하고 반복적으로 컴파일 및 테스트할 수 있습니다.

`assets` 키의 값은 배포할 파일을 지정하는 요소의 배열이어야 합니다. 요소는 `glob`과 유사한 파일 사양이 있는 간단한 문자열일 수 있습니다. 예를 들면 다음과 같습니다.

```typescript
"assets": ["**/*.graphql"],
"watchAssets": true,
```

보다 세밀한 제어를 위해 요소는 다음 키가 있는 객체일 수 있습니다.

- `"include"`: 배포할 자산에 대한 `glob` 유사 파일 사양
- `"exclude"`:`include` 목록에서 **제외**될 자산에 대한 `glob` 유사 파일 사양
- `"outDir"`: 자산이 배포되어야 하는 경로(루트 폴더 기준)를 지정하는 문자열입니다. 기본값은 컴파일러 출력용으로 구성된 동일한 출력 디렉토리입니다.
- `"watchAssets"`: 부울; `true`인 경우 지정된 자산을 감시하는 감시 모드로 실행합니다.

예를 들면:

```typescript
"assets": [
  { "include": "**/*.graphql", "exclude": "**/omitted.graphql", "watchAssets": true },
]
```

> warning **경고** 최상위 `compilerOptions` 속성에서 `watchAssets`를 설정하면 `assets`속성 내의 모든 `watchAssets` 설정이 재정의됩니다.

#### Project properties

이 요소는 단일 저장소 모드 구조에만 존재합니다. 이러한 속성은 Nest에서 모노레포내에서 프로젝트 및 해당 구성 옵션을 찾는 데 사용하므로 일반적으로 편집해서는 안됩니다.
