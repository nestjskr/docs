### Scalars

GraphQL 객체 타입에는 이름과 필드가 있지만 어느 시점에서 이러한 필드는 구체적인 데이터로 확인되어야 합니다. 여기에서 스칼라 타입이 등장합니다. 이들은 쿼리의 잎을 나타냅니다 (자세한 내용은 [여기](https://graphql.org/learn/schema/#scalar-types) 참조). GraphQL에는 `Int`, `Float`, `String`, `Boolean` 및 `ID`와 같은 기본 타입이 포함됩니다. 이러한 내장 타입 외에도 커스텀 원자 데이터 타입 (예: `Date`)을 지원해야 할 수도 있습니다.

#### Code first

코드 우선 접근 방식은 5 개의 스칼라와 함께 제공되며 그중 3 개는 기존 GraphQL 타입에 대한 간단한 별칭입니다.

- `ID` (`GraphQLID`의 별칭) - 객체를 다시 가져오는 데 자주 사용되거나 캐시의 키로 사용되는 고유 식별자를 나타냅니다.
- `Int` (`GraphQLInt`의 별칭) - 부호있는 32 비트 정수
- `Float` (`GraphQLFloat`의 별칭) - 부호있는 배정 밀도 부동 소수점 값
- `GraphQLISODateTime` - UTC의 날짜-시간 문자열 (기본적으로 `Date` 타입을 나타내는 데 사용됨)
- `GraphQLTimestamp` - 시간과 날짜를 UNIX 시대의 시작부터 밀리 초 단위로 나타내는 숫자 문자열

`GraphQLISODateTime` (예: `2019-12-03T09:54:33Z`)은 기본적으로 `Date` 타입을 나타내는 데 사용됩니다. 대신 `GraphQLTimestamp`를 사용하려면 `buildSchemaOptions` 객체의 `dateScalarMode`를 다음과 같이 `'timestamp'`로 설정합니다.

```typescript
GraphQLModule.forRoot({
  buildSchemaOptions: {
    dateScalarMode: 'timestamp',
  }
}),
```

마찬가지로 `GraphQLFloat`는 기본적으로 `number`타입을 나타내는 데 사용됩니다. 대신 `GraphQLInt`를 사용하려면 `buildSchemaOptions` 객체의 `numberScalarMode`를 다음과 같이 `'integer'`로 설정합니다.

```typescript
GraphQLModule.forRoot({
  buildSchemaOptions: {
    numberScalarMode: 'integer',
  }
}),
```

또한 사용자 지정 스칼라를 만들 수 있습니다. 예를 들어 `Date`스칼라를 만들려면 새 클래스를 만들기 만하면됩니다.

```typescript
import { Scalar, CustomScalar } from '@nestjs/graphql';
import { Kind, ValueNode } from 'graphql';

@Scalar('Date', (type) => Date)
export class DateScalar implements CustomScalar<number, Date> {
  description = 'Date custom scalar type';

  parseValue(value: number): Date {
    return new Date(value); // value from the client
  }

  serialize(value: Date): number {
    return value.getTime(); // value sent to the client
  }

  parseLiteral(ast: ValueNode): Date {
    if (ast.kind === Kind.INT) {
      return new Date(ast.value);
    }
    return null;
  }
}
```

이 위치에 `DateScalar`를 프로바이더로 등록하십시오.

```typescript
@Module({
  providers: [DateScalar],
})
export class CommonModule {}
```

이제 클래스에서 `Date` 타입을 사용할 수 있습니다.

```typescript
@Field()
creationDate: Date;
```

#### Schema first

사용자 지정 스칼라([여기](https://www.apollographql.com/docs/graphql-tools/scalars.html)에서 스칼라에 대해 자세히 알아보세요.)를 정의하려면 타입 정의와 전용 리졸버를 만듭니다. 여기(공식 문서에서와 같이)서는 데모용으로 `graphql-type-json` 패키지를 사용합니다. 이 npm 패키지는 `JSON` GraphQL 스칼라 타입을 정의합니다.

패키지를 설치하여 시작하십시오.

```bash
$ npm i --save graphql-type-json
```

패키지가 설치되면 사용자 정의 리졸버를 `forRoot()` 메소드에 전달합니다.

```typescript
import GraphQLJSON from 'graphql-type-json';

@Module({
  imports: [
    GraphQLModule.forRoot({
      typePaths: ['./**/*.graphql'],
      resolvers: { JSON: GraphQLJSON },
    }),
  ],
})
export class AppModule {}
```

이제 타입 정의에서 `JSON` 스칼라를 사용할 수 있습니다.

```graphql
scalar JSON

type Foo {
  field: JSON
}
```

스칼라 타입을 정의하는 또 다른 방법은 간단한 클래스를 만드는 것입니다. `Date` 타입으로 스키마를 향상시키고 싶다고 가정합니다.

```typescript
import { Scalar, CustomScalar } from '@nestjs/graphql';
import { Kind, ValueNode } from 'graphql';

@Scalar('Date')
export class DateScalar implements CustomScalar<number, Date> {
  description = 'Date custom scalar type';

  parseValue(value: number): Date {
    return new Date(value); // value from the client
  }

  serialize(value: Date): number {
    return value.getTime(); // value sent to the client
  }

  parseLiteral(ast: ValueNode): Date {
    if (ast.kind === Kind.INT) {
      return new Date(ast.value);
    }
    return null;
  }
}
```

이 위치에 `DateScalar`를 프로바이더로 등록하십시오.

```typescript
@Module({
  providers: [DateScalar],
})
export class CommonModule {}
```

이제 타입 정의에서 `Date` 스칼라를 사용할 수 있습니다.

```graphql
scalar Date
```

기본적으로 모든 스칼라에 대해 생성된 TypeScript 정의는 `any`이며 특히 타입세이프(typesafe)하지 않습니다.
그러나 타입을 생성하는 방법을 지정할 때 Nest가 커스텀 스칼라에 대한 타입을 생성하는 방법을 구성할 수 있습니다.

```typescript
import { GraphQLDefinitionsFactory } from '@nestjs/graphql';
import { join } from 'path';

const definitionsFactory = new GraphQLDefinitionsFactory()

definitionsFactory.generate({
  typePaths: ['./src/**/*.graphql'],
  path: join(process.cwd(), 'src/graphql.ts'),
  outputAs: 'class',
  defaultScalarType: 'unknown',
  customScalarTypeMapping: {
    DateTime: 'Date',
    BigNumber: '_BigNumber',
  },
  additionalHeader: "import _BigNumber from 'bignumber.js'",
})
```

> info **힌트** 또는 대신 타입 참조를 사용할 수 있습니다 (예: `DateTime: Date`). 이 경우 `GraphQLDefinitionsFactory`는 지정된 타입 (`Date.name`)의 이름 속성을 추출하여 Typescript 정의를 생성합니다. 참고: 내장되지 않은 타입 (사용자 정의 타입)에 대한 import 문을 추가해야 합니다.

이제 다음 GraphQL 커스텀 스칼라 타입이 주어집니다.

```graphql
scalar DateTime
scalar BigNumber
scalar Payload
```

이제 `src/graphql.ts`에서 생성된 다음 TypeScript 정의를 볼 수 있습니다.

```typescript
import _BigNumber from 'bignumber.js'

export type DateTime = Date
export type BigNumber = _BigNumber
export type Payload = unknown
```

여기서는 `customScalarTypeMapping` 속성을 사용하여 사용자 지정 스칼라에 대해 선언하려는 타입의 맵을 제공했습니다. 또한 이러한 타입 정의에 필요한 가져오기를 추가할 수 있도록 `additionalHeader` 속성도 제공했습니다. 마지막으로 `'unknown'`의 `defaultScalarType`을 추가하여 `customScalarTypeMapping`에 지정되지 않은 모든 커스텀 스칼라가 `any` 대신 `unknown`으로 별칭이 지정됩니다. ([TypeScript 권장](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-0.html#new-unknown-top-type) 사용 추가된 타입 안전을 위해 3.0부터).

> info **힌트** `bignumber.js`에서 `_BigNumber`를 가져 왔습니다. 이것은 [순환 타입 참조](https://github.com/Microsoft/TypeScript/issues/12525#issuecomment-263166239)를 피하기 위한 것입니다.
