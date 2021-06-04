## Harnessing the power of TypeScript & GraphQL

[GraphQL](https://graphql.org/)은 API를 위한 강력한 쿼리 언어이며 기존 데이터로 이러한 쿼리를 수행하기 위한 런타임입니다. REST API에서 일반적으로 발견되는 많은 문제를 해결하는 우아한 접근 방식입니다. 배경 지식을 위해 GraphQL과 REST간에 이 [비교](https://dev-blog.apollodata.com/graphql-vs-rest-5d425123e34b)를 읽어 보시기 바랍니다. [TypeScript](https://www.typescriptlang.org/)와 결합된 GraphQL은 GraphQL 쿼리를 통해 더 나은 유형 안전성을 개발하여 종단간 입력을 제공합니다.

이 장에서는 GraphQL에 대한 기본적인 이해를 가정하고 내장 `@nestjs/graphql` 모듈을 사용하는 방법에 중점을 둡니다.  `GraphQLModule`은 [Apollo](https://www.apollographql.com/) 서버를 둘러싼 래퍼입니다. 이 입증된 GraphQL 패키지를 사용하여 Nest와 함께 GraphQL을 사용하는 방법을 제공합니다.

#### Installation

필요한 패키지를 설치하여 시작하십시오.

```bash
$ npm i @nestjs/graphql graphql-tools graphql apollo-server-express
```
> info **힌트** Fastify를 사용하는 경우 `apollo-server-express`를 설치하는 대신 `apollo-server-fastify`를 설치해야 합니다.

#### Overview

Nest는 GraphQL 애플리케이션을 빌드하는 두가지 방법, 즉 **코드 우선(code first)** 및 **스키마 우선(schema first)** 방법을 제공합니다. 자신에게 가장 잘 맞는 것을 선택해야 합니다. 이 GraphQL 섹션의 대부분의 장은 두가지 주요 부분으로 나뉩니다. 하나는 **코드 우선** 채택시 따라야 하는 부분이고 다른 하나는 **스키마 먼저** 채택시 사용해야 합니다.

**코드 우선** 접근 방식에서는 데코레이터와 TypeScript 클래스를 사용하여 해당 GraphQL 스키마를 생성합니다. 이 방법은 TypeScript로만 작업하고 언어 구문 간의 컨텍스트 전환을 피하려는 경우 유용합니다.

**스키마 우선** 접근 방식에서 진실의 소스는 GraphQL SDL (Schema Definition Language) 파일입니다. SDL은 서로 다른 플랫폼간에 스키마 파일을 공유하는 언어에 구애받지 않는 방법입니다. Nest는 GraphQL 스키마를 기반으로 TypeScript 정의 (클래스 또는 인터페이스 사용)를 자동으로 생성하여 중복된 상용구 코드를 작성할 필요성을 줄입니다.

#### Getting started with GraphQL & TypeScript

패키지가 설치되면 `GraphQLModule`을 가져와 `forRoot()` 정적 메서드로 구성할 수 있습니다.

```typescript
@@filename()
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';

@Module({
  imports: [
    GraphQLModule.forRoot({}),
  ],
})
export class AppModule {}
```

`forRoot()` 메소드는 옵션 객체를 인수로받습니다. 이러한 옵션은 기본 Apollo 인스턴스로 전달됩니다 (사용 가능한 설정에 대한 자세한 내용은 [여기](https://www.apollographql.com/docs/apollo-server/v2/api/apollo-server.html#constructor-options-lt-ApolloServer-gt) 참조). 예를 들어 `playground`를 비활성화하고 `debug` 모드를 끄려면 다음 옵션을 전달합니다.

```typescript
@@filename()
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';

@Module({
  imports: [
    GraphQLModule.forRoot({
      debug: false,
      playground: false,
    }),
  ],
})
export class AppModule {}
```

언급했듯이 이러한 옵션은 `ApolloServer` 생성자(constructor)에 전달됩니다.

<app-banner-enterprise></app-banner-enterprise>

#### GraphQL playground

플레이 그라운드는 기본적으로 GraphQL 서버 자체와 동일한 URL에서 사용할 수 있는 그래픽, 대화형 브라우저 내 GraphQL IDE입니다. 플레이 그라운드에 액세스하려면 기본 GraphQL 서버를 구성하고 실행해야 합니다. 지금 확인하려면 [여기에서 작동하는 예제](https://github.com/nestjs/nest/tree/master/sample/23-graphql-code-first)를 설치하고 빌드할 수 있습니다. 또는 이러한 코드 샘플을 따라가는 경우 [Resolvers chapter](/graphql/resolvers-map)의 단계를 완료하면 플레이 그라운드에 액세스할 수 있습니다.

그런 다음 백그라운드에서 애플리케이션을 실행하고 웹 브라우저를 열고 `http://localhost:3000/graphql`로 이동할 수 있습니다 (호스트 및 포트는 구성에 따라 다를 수 있음). 그러면 아래와 같이 GraphQL 플레이 그라운드가 표시됩니다.

<figure>
  <img src="/assets/playground.png" alt="" />
</figure>

#### Multiple endpoints

`@nestjs/graphql` 모듈의 또 다른 유용한 기능은 한번에 여러 엔드 포인트를 제공하는 기능입니다. 이를 통해 어떤 모듈을 어떤 엔드 포인트에 포함할지 결정할 수 있습니다. 기본적으로 `GraphQL`은 전체 앱에서 리졸버를 검색합니다. 이 스캔을 모듈의 하위 집합으로 만 제한하려면 `include` 속성을 사용합니다.

```typescript
GraphQLModule.forRoot({
  include: [CatsModule],
}),
```

#### Code first

**코드 우선** 접근 방식에서는 데코레이터와 TypeScript 클래스를 사용하여 해당 GraphQL 스키마를 생성합니다.

코드 우선 접근 방식을 사용하려면 먼저 옵션 객체에 `autoSchemaFile` 속성을 추가합니다.

```typescript
GraphQLModule.forRoot({
  autoSchemaFile: join(process.cwd(), 'src/schema.gql'),
}),
```

`autoSchemaFile` 속성 값은 자동으로 생성된 스키마가 생성될 경로입니다. 또는 스키마를 메모리에서 즉석에서 생성할 수 있습니다. 이를 활성화하려면 `autoSchemaFile` 속성을 `true`로 설정합니다.

```typescript
GraphQLModule.forRoot({
  autoSchemaFile: true,
}),
```

기본적으로 생성된 스키마의 유형은 포함된 모듈에 정의된 순서대로 됩니다. 스키마를 사전순으로 정렬하려면 `sortSchema` 속성을 `true`로 설정합니다.

```typescript
GraphQLModule.forRoot({
  autoSchemaFile: join(process.cwd(), 'src/schema.gql'),
  sortSchema: true,
}),
```

#### Example

완전히 작동하는 코드 첫 번째 샘플은 [여기](https://github.com/nestjs/nest/tree/master/sample/23-graphql-code-first)에 있습니다.

#### Schema first

스키마 우선 접근 방식을 사용하려면 먼저 옵션 개체에 `typePaths` 속성을 추가합니다. `typePaths` 속성은 `GraphQLModule`이 작성할 GraphQL SDL 스키마 정의 파일을 찾아야하는 위치를 나타냅니다. 이러한 파일은 메모리에서 결합됩니다. 이를 통해 스키마를 여러 파일로 분할하고 해당 해석기 근처에서 찾을 수 있습니다.

```typescript
GraphQLModule.forRoot({
  typePaths: ['./**/*.graphql'],
}),
```

일반적으로 GraphQL SDL 유형에 해당하는 TypeScript 정의 (클래스 및 인터페이스)도 필요합니다. 해당 TypeScript 정의를 직접 만드는 것은 중복되고 지루합니다. 그것은 우리에게 단일 소스의 진실을 남기지 않습니다. SDL 내에서 변경된 각각의 경우에는 TypeScript 정의도 조정해야 합니다. 이를 해결하기 위해 `@nestjs/graphql` 패키지는 추상 구문 트리 ([AST](https://en.wikipedia.org/wiki/Abstract_syntax_tree))에서 TypeScript 정의를 **자동으로 생성**할 수 있습니다. 이 기능을 활성화하려면 `GraphQLModule`을 구성할 때 `definitions` 옵션 속성을 추가합니다.

```typescript
GraphQLModule.forRoot({
  typePaths: ['./**/*.graphql'],
  definitions: {
    path: join(process.cwd(), 'src/graphql.ts'),
  },
}),
```

`definitions` 객체의 경로 속성은 생성된 TypeScript 출력을 저장할 위치를 나타냅니다. 기본적으로 생성된 모든 TypeScript 유형은 인터페이스로 생성됩니다. 대신 클래스를 생성하려면 값이 `'class'`인 `outputAs`속성을 지정하세요.

```typescript
GraphQLModule.forRoot({
  typePaths: ['./**/*.graphql'],
  definitions: {
    path: join(process.cwd(), 'src/graphql.ts'),
    outputAs: 'class',
  },
}),
```

위의 접근 방식은 애플리케이션이 시작될 때마다 TypeScript 정의를 동적으로 생성합니다. 또는 요청시 생성하는 간단한 스크립트를 작성하는 것이 좋습니다. 예를 들어 다음 스크립트를 `generate-typings.ts`로 만든다고 가정합니다.

```typescript
import { GraphQLDefinitionsFactory } from '@nestjs/graphql';
import { join } from 'path';

const definitionsFactory = new GraphQLDefinitionsFactory();
definitionsFactory.generate({
  typePaths: ['./src/**/*.graphql'],
  path: join(process.cwd(), 'src/graphql.ts'),
  outputAs: 'class',
});
```

이제 요청시 이 스크립트를 실행할 수 있습니다.

```bash
$ ts-node generate-typings
```

> info **힌트** 스크립트를 미리 컴파일 (예: `tsc` 사용)하고 `node`를 사용하여 실행할 수 있습니다.

스크립트에 대해 감시 모드를 활성화하려면 (`.graphql` 파일이 변경될 때마다 자동으로 타이핑을 생성하기 위해) `watch` 옵션을 `generate()`메서드에 전달합니다.

```typescript
definitionsFactory.generate({
  typePaths: ['./src/**/*.graphql'],
  path: join(process.cwd(), 'src/graphql.ts'),
  outputAs: 'class',
  watch: true,
});
```

모든 객체 유형에 대해 추가 `__typename` 필드를 자동으로 생성하려면 `emitTypenameField` 옵션을 활성화합니다.

```typescript
definitionsFactory.generate({
  // ...,
  emitTypenameField: true,
});
```

인수가 없는 일반 필드로 리졸브 (쿼리(query), 변형(mutation), 구독(subscription))를 생성하려면 `skipResolverArgs` 옵션을 활성화합니다.

```typescript
definitionsFactory.generate({
  // ...,
  skipResolverArgs: true,
});
```

#### Example

완전히 작동하는 스키마 첫 번째 샘플은 [여기](https://github.com/nestjs/nest/tree/master/sample/12-graphql-schema-first)에 있습니다.

#### Accessing generated schema

경우에 따라 (예: 종단간 테스트) 생성된 스키마 개체에 대한 참조를 가져올 수 있습니다. 엔드 투 엔드 테스트에서는 HTTP 리스너를 사용하지 않고 `graphql` 객체를 사용하여 쿼리를 실행할 수 있습니다.

`GraphQLSchemaHost`클래스를 사용하여 생성된 스키마 (코드 우선 또는 스키마 우선 접근 방식)에 액세스할 수 있습니다.

```typescript
const { schema } = app.get(GraphQLSchemaHost);
```

> info **힌트** 애플리케이션이 초기화된 후 `GraphQLSchemaHost#schema` getter를 호출해야 합니다 (`app.listen()` 또는 `app.init()` 메소드에 의해 `onModuleInit` 후크가 트리거된 후).

#### Async configuration

모듈 옵션을 정적으로 전달하는 대신 비동기적으로 전달해야 하는 경우 `forRootAsync()`메서드를 사용하세요. 대부분의 동적 모듈과 마찬가지로 Nest는 비동기 구성을 처리하는 몇가지 기술을 제공합니다.

한가지 기술은 팩토리 기능을 사용하는 것입니다.

```typescript
GraphQLModule.forRootAsync({
  useFactory: () => ({
    typePaths: ['./**/*.graphql'],
  }),
}),
```

다른 팩토리 공급자와 마찬가지로 팩토리 함수는 [비동기](/fundamentals/custom-providers#factory-providers-usefactory)일 수 있으며 `inject`를 통해 종속성을 주입할 수 있습니다.

```typescript
GraphQLModule.forRootAsync({
  imports: [ConfigModule],
  useFactory: async (configService: ConfigService) => ({
    typePaths: configService.getString('GRAPHQL_TYPE_PATHS'),
  }),
  inject: [ConfigService],
}),
```

또는 아래와 같이 팩토리 대신 클래스를 사용(useClass)하여 `GraphQLModule`을 구성할 수 있습니다.

```typescript
GraphQLModule.forRootAsync({
  useClass: GqlConfigService,
}),
```

위의 구성은 `GraphQLModule` 내부에서 `GqlConfigService`를 인스턴스화하여 옵션 객체를 생성하는 데 사용합니다. 이 예에서 `GqlConfigService`는 아래와 같이 `GqlOptionsFactory` 인터페이스를 구현해야합니다. `GraphQLModule`은 제공된 클래스의 인스턴스화된 객체에 대해 `createGqlOptions()`메서드를 호출합니다.

```typescript
@Injectable()
class GqlConfigService implements GqlOptionsFactory {
  createGqlOptions(): GqlModuleOptions {
    return {
      typePaths: ['./**/*.graphql'],
    };
  }
}
```

`GraphQLModule` 내부에 비공개 사본을 만드는 대신 기존 옵션 제공 업체를 재사용하려면 `useExisting`구문을 사용하세요.

```typescript
GraphQLModule.forRootAsync({
  imports: [ConfigModule],
  useExisting: ConfigService,
}),
```
