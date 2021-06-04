### Unions

유니온 타입은 인터페이스와 매우 유사하지만 타입간에 공통 필드를 지정하지 않습니다 ([여기](https://graphql.org/learn/schema/#union-types)에서 자세히 읽어보십시오). 유니온은 단일 필드에서 분리된 데이터 타입을 반환하는 데 유용합니다.

#### Code first

GraphQL 유니온 타입을 정의하려면 이 유니온이 구성될 클래스를 정의해야 합니다. Apollo 문서의 [예제](https://www.apollographql.com/docs/apollo-server/schema/unions-interfaces/#union-type)에 따라 두개의 클래스를 생성합니다. 첫번째, `Book`:

```typescript
import { Field, ObjectType } from '@nestjs/graphql';

@ObjectType()
export class Book {
  @Field()
  title: string;
}
```

그리고 `Author`:

```typescript
import { Field, ObjectType } from '@nestjs/graphql';

@ObjectType()
export class Author {
  @Field()
  name: string;
}
```

이 위치에 `@nestjs/graphql` 패키지에서 내보낸 `createUnionType` 함수를 사용하여 `ResultUnion` 유니온을 등록합니다.

```typescript
export const ResultUnion = createUnionType({
  name: 'ResultUnion',
  types: () => [Author, Book],
});
```

이제 쿼리에서 `ResultUnion`을 참조할 수 있습니다.

```typescript
@Query(returns => [ResultUnion])
search(): Array<typeof ResultUnion> {
  return [new Author(), new Book()];
}
```

그러면 SDL에서 GraphQL 스키마의 다음 부분이 생성됩니다.

```graphql
type Author {
  name: String!
}

type Book {
  title: String!
}

union ResultUnion = Author | Book

type Query {
  search: [ResultUnion!]!
}
```

라이브러리에서 생성된 기본 `resolveType()`함수는 리졸버 메서드에서 반환된 값을 기반으로 타입을 추출합니다. 즉, 리터럴 JavaScript 객체 대신 클래스 인스턴스를 반환해야 합니다.

사용자 정의된 `resolveType()` 함수를 제공하려면 다음과 같이 `createUnionType()` 함수에 전달된 옵션 객체에 `resolveType` 속성을 전달합니다.

```typescript
export const ResultUnion = createUnionType({
  name: 'ResultUnion',
  types: () => [Author, Book],
  resolveType(value) {
    if (value.name) {
      return Author;
    }
    if (value.title) {
      return Book;
    }
    return null;
  },
});
```

#### Schema first

스키마 우선 접근 방식에서 통합을 정의하려면 SDL을 사용하여 GraphQL 통합을 생성하면 됩니다.

```graphql
type Author {
  name: String!
}

type Book {
  title: String!
}

union ResultUnion = Author | Book
```

그런 다음 타이핑 생성 기능 ([빠른 시작](/graphql/quick-start) 장에 표시된대로)을 사용하여 해당하는 TypeScript 정의를 생성할 수 있습니다.

```typescript
export class Author {
  name: string;
}

export class Book {
  title: string;
}

export type ResultUnion = Author | Book;
```

유니온은 유니온이 해석해야 하는 타입을 결정하기 위해 리졸버 맵에 추가 `__resolveType` 필드가 필요합니다. 또한 `ResultUnionResolver` 클래스는 모든 모듈에서 프로바이더로 등록해야 합니다. `ResultUnionResolver` 클래스를 만들고 `__resolveType` 메서드를 정의해 보겠습니다.

```typescript
@Resolver('ResultUnion')
export class ResultUnionResolver {
  @ResolveField()
  __resolveType(value) {
    if (value.name) {
      return 'Author';
    }
    if (value.title) {
      return 'Book';
    }
    return null;
  }
}
```

> info **힌트** 모든 데코레이터는 `@nestjs/graphql` 패키지에서 내보내집니다.

### Enums

Enum형은 허용되는 특정 값 집합으로 제한되는 특별한 종류의 스칼라입니다 ([여기](https://graphql.org/learn/schema/#enumeration-types)에서 자세히 읽어보십시오). 이를 통해 다음을 수행할 수 있습니다.

- 이 타입의 인수가 허용된 값 중 하나인지 확인
- 타입 시스템을 통해 필드가 항상 유한한 값 집합중 하나가 될 것임을 전달합니다.

#### Code first

코드 우선 접근 방식을 사용할 때 TypeScript enum형을 생성하여 GraphQL enum 형을 정의합니다.

```typescript
export enum AllowedColor {
  RED,
  GREEN,
  BLUE,
}
```

이 위치에 `@nestjs/graphql` 패키지에서 내보낸 `registerEnumType` 함수를 사용하여 `AllowedColor` enum형을 등록합니다.

```typescript
registerEnumType(AllowedColor, {
  name: 'AllowedColor',
});
```

이제 타입에서 `AllowedColor`를 참조할 수 있습니다.

```typescript
@Field(type => AllowedColor)
favoriteColor: AllowedColor;
```

그러면 SDL에서 GraphQL 스키마의 다음 부분이 생성됩니다.

```graphql
enum AllowedColor {
  RED
  GREEN
  BLUE
}
```

enum형에 대한 설명을 제공하려면 `description` 속성을 `registerEnumType()` 함수에 전달합니다.

```typescript
registerEnumType(AllowedColor, {
  name: 'AllowedColor',
  description: 'The supported colors.',
});
```

enum 형 값에 대한 설명을 제공하거나 값을 지원 중단됨으로 표시하려면 다음과 같이 `valuesMap` 속성을 전달합니다.

```typescript
registerEnumType(AllowedColor, {
  name: 'AllowedColor',
  description: 'The supported colors.',
  valuesMap: {
    RED: {
      description: 'The default color.',
    },
    BLUE: {
      deprecationReason: 'Too blue.',
    },
  },
});
```

그러면 SDL에 다음 GraphQL 스키마가 생성됩니다.

```graphql
"""
The supported colors.
"""
enum AllowedColor {
  """
  The default color.
  """
  RED
  GREEN
  BLUE @deprecated(reason: "Too blue.")
}
```

#### Schema first

스키마 우선 접근 방식에서 열거자(enumerator)를 정의하려면 SDL로 GraphQL enum 형을 생성하면 됩니다.

```graphql
enum AllowedColor {
  RED
  GREEN
  BLUE
}
```

그런 다음 타이핑 생성 기능 ([빠른 시작](/graphql/quick-start) 장에 표시된대로)을 사용하여 해당 TypeScript 정의를 생성할 수 있습니다.

```typescript
export enum AllowedColor {
  RED
  GREEN
  BLUE
}
```

때로는 백엔드가 공개 API와 내부적으로 enum 형에 다른 값을 강제합니다. 이 예에서 API에는 `RED`가 포함되어 있지만 리졸버에서는 대신 `#f00`을 사용할 수 있습니다 ([여기](https://www.apollographql.com/docs/apollo-server/schema/scalars-enums/#internal-values)). 이를 수행하려면 `AllowedColor` enum 형에 대한 리졸버 객체를 선언합니다.

```typescript
export const allowedColorResolver: Record<keyof typeof AllowedColor, any> = {
  RED: '#f00',
};
```

> info **힌트** 모든 데코레이터는 `@nestjs/graphql` 패키지에서 내보내집니다.

그런 다음이 리졸버 객체를 다음과 같이 `GraphQLModule.forRoot()` 메서드의 `resolvers` 속성과 함께 사용합니다.

```typescript
GraphQLModule.forRoot({
  resolvers: {
    AllowedColor: allowedColorResolver,
  },
});
```
