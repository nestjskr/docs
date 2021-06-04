### CLI Plugin

> warning **경고** 이 장은 코드 우선 접근 방식에만 적용됩니다.

TypeScript의 메타 데이터 리플렉션 시스템에는 몇가지 제한 사항이 있어 클래스가 어떤 속성으로 구성되어 있는지 확인하거나 지정된 속성이 선택 사항인지 필수인지 여부를 인식할 수 없습니다. 그러나 이러한 제약조건중 일부는 컴파일 타임에 해결할 수 있습니다. Nest는 필요한 상용구 코드의 양을 줄이기 위해 TypeScript 컴파일 프로세스를 향상시키는 플러그인을 제공합니다.

> info **힌트** 이 플러그인은 **선택**입니다. 원하는 경우 모든 데코레이터를 수동으로 선언하거나 필요한 곳에 특정 데코레이터만 선언할 수 있습니다.

#### Overview

GraphQL 플러그인은 자동으로 다음을 수행합니다.

- `@HideField`를 사용하지 않는 한 모든 입력 객체, 객체 타입 및 args 클래스 속성에 `@Field` 주석을 추가합니다.
- 물음표에 따라 `nullable` 속성을 설정합니다 (예: `name?: string`은 `nullable: true`로 설정 됨).
- 타입에 따라 `type` 속성을 설정합니다 (배열도 지원).
- 주석을 기반으로 속성에 대한 설명을 생성합니다 (`introspectComments`가 `true`로 설정된 경우).

플러그인에서 분석하려면 파일 이름에 다음 접미사중 하나가 **반드시** 있어야 합니다: `['.input.ts', '.args.ts', '.entity.ts', ' .model.ts ']` (예: `author.entity.ts`). 다른 접미사를 사용하는 경우 `typeFileNameSuffix` 옵션을 지정하여 플러그인의 동작을 조정할 수 있습니다 (아래 참조).

지금까지 배운 내용으로 GraphQL에서 타입을 선언하는 방법을 패키지에 알리기 위해 많은 코드를 복제해야 합니다. 예를 들어 다음과 같이 간단한 `Author` 클래스를 정의할 수 있습니다.

```typescript
@@filename(authors/models/author.model)
@ObjectType()
export class Author {
  @Field(type => ID)
  id: number;

  @Field({ nullable: true })
  firstName?: string;

  @Field({ nullable: true })
  lastName?: string;

  @Field(type => [Post])
  posts: Post[];
}
```

중간 규모의 프로젝트에서 중요한 문제는 아니지만, 많은 클래스 세트가 있으면 장황하고 유지하기가 어렵습니다.

GraphQL 플러그인을 활성화하면 위의 클래스 정의를 간단하게 선언할 수 있습니다.

```typescript
@@filename(authors/models/author.model)
@ObjectType()
export class Author {
  @Field(type => ID)
  id: number;
  firstName?: string;
  lastName?: string;
  posts: Post[];
}
```

플러그인은 **추상 구문 트리**를 기반으로 즉석에서 적절한 데코레이터를 추가합니다. 따라서 코드 전체에 흩어져있는 `@Field` 데코레이터와 씨름할 필요가 없습니다.

> info **힌트** 플러그인은 누락된 GraphQL 속성을 자동으로 생성하지만 재 정의해야 하는 경우 `@Field()`를 통해 명시적으로 설정하면 됩니다.

#### Comments introspection

코멘트 검사 기능이 활성화된 상태에서 CLI 플러그인은 코멘트를 기반으로 필드에 대한 설명을 생성합니다.

예를 들어 `roles` 속성의 예가 다음과 같습니다.

```typescript
/**
 * A list of user's roles
 */
@Field(() => [String], {
  description: `A list of user's roles`
})
roles: string[];
```

설명 값을 중복해야 합니다. `introspectComments`를 활성화하면 CLI 플러그인이 이러한 코멘트을 추출하고 속성에 대한 설명을 자동으로 제공할 수 있습니다. 이제 위 필드는 다음과 같이 간단하게 선언할 수 있습니다.

```typescript
/**
 * A list of user's roles
 */
roles: string[];
```

#### Using the CLI plugin

플러그인을 활성화하려면 `nest-cli.json` ([Nest CLI](/cli/overview)을 사용하는 경우)을 열고 다음 `plugins` 구성을 추가합니다.

```javascript
{
  "collection": "@nestjs/schematics",
  "sourceRoot": "src",
  "compilerOptions": {
    "plugins": ["@nestjs/graphql"]
  }
}
```

`options` 속성을 사용하여 플러그인의 동작을 사용자 지정할 수 있습니다.

```javascript
"plugins": [
  {
    "name": "@nestjs/graphql",
    "options": {
      "typeFileNameSuffix": [".input.ts", ".args.ts"],
      "introspectComments": true
    }
  }
]
```

`options` 속성은 다음 인터페이스를 충족해야합니다.

```typescript
export interface PluginOptions {
  typeFileNameSuffix?: string[];
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
    <td><code>typeFileNameSuffix</code></td>
    <td><code>['.input.ts', '.args.ts', '.entity.ts', '.model.ts']</code></td>
    <td>GraphQL types files suffix</td>
  </tr>
  <tr>
    <td><code>introspectComments</code></td>
      <td><code>false</code></td>
      <td>If set to true, plugin will generate descriptions for properties based on comments</td>
  </tr>
</table>

CLI를 사용하지 않고 대신 사용자 지정 `webpack` 구성이 있는 경우 이 플러그인을 `ts-loader`와 함께 사용할 수 있습니다.

```javascript
getCustomTransformers: (program: any) => ({
  before: [require('@nestjs/graphql/plugin').before({}, program)]
}),
```

#### Integration with `ts-jest` (e2e tests)

이 플러그인이 활성화된 상태에서 e2e 테스트를 실행할 때 스키마 컴파일에 문제가 발생할 수 있습니다. 예를 들어 가장 일반적인 오류중 하나는 다음과 같습니다.

```json
Object type <name> must define one or more fields.
```

이는 `jest` 구성이 `@nestjs/graphql/plugin` 플러그인을 어디로도 가져오지 않기 때문에 발생합니다.

이 문제를 해결하려면 e2e 테스트 디렉터리에 다음 파일을 만듭니다.

```javascript
const transformer = require('@nestjs/graphql/plugin');

module.exports.name = 'nestjs-graphql-transformer';
// you should change the version number anytime you change the configuration below - otherwise, jest will not detect changes
module.exports.version = 1;

module.exports.factory = (cs) => {
  return transformer.before(
    {
      // @nestjs/graphql/plugin options (can be empty)
    },
    cs.tsCompiler.program,
  );
};
```

이 위치에서 `jest` 구성 파일 내에서 AST 변환기를 가져옵니다. 기본적으로 (시작 애플리케이션에서) e2e 테스트 구성 파일은 `test` 폴더 아래에 있으며 이름은 `jest-e2e.json`입니다.

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
