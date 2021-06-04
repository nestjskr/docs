### CLI Plugin

TypeScript의 메타데이터 리플렉션 시스템에는 몇가지 제한 사항이 있어 클래스가 어떤 속성으로 구성되어 있는지 확인하거나 지정된 속성이 선택사항인지 필수인지 여부를 인식할 수 없습니다. 그러나 이러한 제약 조건중 일부는 컴파일 타임에 해결할 수 있습니다. Nest는 필요한 상용구 코드의 양을 줄이기 위해 TypeScript 컴파일 프로세스를 향상시키는 플러그인을 제공합니다.

> info **힌트**이 플러그인은 **선택**입니다. 원하는 경우 모든 데코레이터를 수동으로 선언하거나 필요한 곳에 특정 데코레이터만 선언할 수 있습니다.

#### Overview

Swagger 플러그인은 자동으로 다음을 수행합니다.

- `@ApiHideProperty`를 사용하지 않는 한 모든 DTO 속성에 `@ApiProperty` 주석을 추가합니다.
- 물음표에 따라 `required` 속성을 설정합니다(예: `name?: string`은 `required: false`로 설정됨).
- 유형에 따라 `type` 또는 `enum` 속성 설정(배열도 지원)
- 할당된 기본값에 따라 `default` 속성 설정
- `class-validator` 데코레이터를 기반으로 여러 유효성 검사 규칙을 설정합니다(`classValidatorShim`이 `true`로 설정된 경우).
- 적절한 상태와 `type`(응답 모델)을 사용하여 모든 엔드포인트에 응답 데코레이터를 추가합니다.
- 주석을 기반으로 속성 및 엔드포인트에 대한 설명을 생성합니다 (`introspectComments`가 `true`로 설정된 경우).
- 주석을 기반으로 속성에 대한 예제 값 생성(`introspectComments`가 `true`로 설정된 경우)

파일 이름은 다음 접미사 중 하나를 **반드시 가져야합니다**: `['.dto.ts', '.entity.ts']`(예: `create-user.dto.ts`) 플러그인에 의해 분석됩니다.

다른 접미사를 사용하는 경우 `dtoFileNameSuffix` 옵션을 지정하여 플러그인의 동작을 조정할 수 있습니다(아래 참조).

Previously, if you wanted to provide an interactive experience with the Swagger UI,
you had to duplicate a lot of code to let the package know how your models/components should be declared in the specification. For example, you could define a simple `CreateUserDto` class as follows:
이전에는 Swagger UI로 대화형 환경을 제공하려는 경우, 사양에서 모델/컴포넌트가 어떻게 선언되어야 하는지 패키지에 알리기 위해 많은 코드를 복제해야했습니다. 예를 들어 다음과 같이 간단한 `CreateUserDto` 클래스를 정의할 수 있습니다.

```typescript
export class CreateUserDto {
  @ApiProperty()
  email: string;

  @ApiProperty()
  password: string;

  @ApiProperty({ enum: RoleEnum, default: [], isArray: true })
  roles: RoleEnum[] = [];

  @ApiProperty({ required: false, default: true })
  isEnabled?: boolean = true;
}
```

중간 규모의 프로젝트에서 중요한 문제는 아니지만, 많은 클래스가 있으면 장황하고 유지하기가 어렵습니다.

Swagger 플러그인을 활성화하면 위의 클래스 정의를 간단하게 선언할 수 있습니다.

```typescript
export class CreateUserDto {
  email: string;
  password: string;
  roles: RoleEnum[] = [];
  isEnabled?: boolean = true;
}
```

플러그인은 **추상 구문 트리**를 기반으로 적절한 데코레이터를 즉시 추가합니다. 따라서 코드 전체에 흩어져있는 `@ApiProperty` 데코레이터와 씨름할 필요가 없습니다.

> info **힌트** 플러그인은 누락된 swagger 속성을 자동으로 생성하지만 재정의해야 하는 경우 `@ApiProperty()`를 통해 명시적으로 설정하기만하면 됩니다.

#### Comments introspection

코멘트 내부검사 기능이 활성화된 상태에서 CLI 플러그인은 코멘트를 기반으로 속성에 대한 설명과 예제값을 생성합니다.

예를 들어 `roles` 속성의 예가 다음과 같습니다.

```typescript
/**
 * A list of user's roles
 * @example ['admin']
 */
@ApiProperty({
  description: `A list of user's roles`,
  example: ['admin'],
})
roles: RoleEnum[] = [];
```

설명 및 예제값을 모두 복제해야합니다. `introspectComments`를 활성화하면 CLI 플러그인이 이러한 코멘트를 추출하고 속성에 대한 설명(및 정의된 경우 예제)을 자동으로 제공할 수 있습니다. 이제 위의 속성을 다음과 같이 간단히 선언할 수 있습니다.

```typescript
/**
 * A list of user's roles
 * @example ['admin']
 */
roles: RoleEnum[] = [];
```

#### Using the CLI plugin

플러그인을 활성화하려면 `nest-cli.json` ([Nest CLI](/cli/overview)을 사용하는 경우)을 열고 다음 `plugins` 구성을 추가합니다.

```javascript
{
  "collection": "@nestjs/schematics",
  "sourceRoot": "src",
  "compilerOptions": {
    "plugins": ["@nestjs/swagger"]
  }
}
```

`options` 속성을 사용하여 플러그인의 동작을 사용자 지정할 수 있습니다.

```javascript
"plugins": [
  {
    "name": "@nestjs/swagger",
    "options": {
      "classValidatorShim": false,
      "introspectComments": true
    }
  }
]
```

`options` 속성은 다음 인터페이스를 충족해야합니다.

```typescript
export interface PluginOptions {
  dtoFileNameSuffix?: string[];
  controllerFileNameSuffix?: string[];
  classValidatorShim?: boolean;
  introspectComments?: boolean;
}
```

<table>
  <tr>
    <th>Option</th>
    <th>Default</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>dtoFileNameSuffix</code></td>
    <td><code>['.dto.ts', '.entity.ts']</code></td>
    <td>DTO (Data Transfer Object) 파일 접미사</td>
  </tr>
  <tr>
    <td><code>controllerFileNameSuffix</code></td>
    <td><code>.controller.ts</code></td>
    <td>컨트롤러 파일 접미사</td>
  </tr>
  <tr>
    <td><code>classValidatorShim</code></td>
    <td><code>true</code></td>
    <td>true로 설정하면 모듈은 <code>class-validator</code> 유효성 검사 데코레이터를 재사용합니다 (예: <code>@Max(10)</code>는 스키마 정의에 <code>max: 10</code>을 추가합니다)</td>
  </tr>
  <tr>
  <td><code>introspectComments</code></td>
    <td><code>false</code></td>
    <td>true로 설정하면 플러그인은 코멘트를 기반으로 속성에 대한 설명 및 예제 값을 생성합니다.</td>
  </tr>
</table>

CLI를 사용하지 않고 대신 사용자 지정 `webpack` 구성이 있는 경우 이 플러그인을 `ts-loader`와 함께 사용할 수 있습니다.

```javascript
getCustomTransformers: (program: any) => ({
  before: [require('@nestjs/swagger/plugin').before({}, program)]
}),
```

#### Integration with `ts-jest` (e2e tests)

e2e 테스트를 실행하기 위해 `ts-jest`는 소스코드 파일을 메모리에서 즉시 컴파일합니다. 즉, Nest CLI 컴파일러를 사용하지 않으며 플러그인을 적용하거나 AST 변환을 수행하지 않습니다.

플러그인을 활성화하려면 e2e 테스트 디렉토리에 다음 파일을 만듭니다.

```javascript
const transformer = require('@nestjs/swagger/plugin');

module.exports.name = 'nestjs-swagger-transformer';
// you should change the version number anytime you change the configuration below - otherwise, jest will not detect changes
module.exports.version = 1;

module.exports.factory = (cs) => {
  return transformer.before(
    {
      // @nestjs/swagger/plugin options (can be empty)
    },
    cs.tsCompiler.program,
  );
};
```

이 위치에서 `jest` 구성 파일 내에서 AST 변환기를 가져옵니다. 기본적으로(시작 애플리케이션에서) e2e 테스트 구성 파일은 `test` 폴더 아래에 있으며 이름은 `jest-e2e.json`입니다.

```json
{
  ... // other configuration
  "globals": {
    "ts-jest": {
      "astTransformers": {
        "before": ["<path to the file created above>"],
      }
    }
  }
}
```
