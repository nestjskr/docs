### Interfaces

많은 타입 시스템과 마찬가지로 GraphQL은 인터페이스를 지원합니다. **인터페이스**는 인터페이스를 구현하기 위해 타입이 포함해야하는 특정 필드 집합을 포함하는 추상 타입입니다 ([여기](https://graphql.org/learn/schema/#interfaces)에서 자세히 알아보기).

#### Code first

코드 우선 접근 방식을 사용할 때 `@nestjs/graphql`에서 내보낸 `@InterfaceType()` 데코레이터로 주석이 달린 추상 클래스를 생성하여 GraphQL 인터페이스를 정의합니다.

```typescript
import { Field, ID, InterfaceType } from '@nestjs/graphql';

@InterfaceType()
export abstract class Character {
  @Field(type => ID)
  id: string;

  @Field()
  name: string;
}
```

> warning **경고** TypeScript 인터페이스는 GraphQL 인터페이스를 정의하는 데 사용할 수 없습니다.

그러면 SDL에서 GraphQL 스키마의 다음 부분이 생성됩니다:

```graphql
interface Character {
  id: ID!
  name: String!
}
```

이제 `Character` 인터페이스를 구현하려면 `implements` 키를 사용하십시오.

```typescript
@ObjectType({
  implements: () => [Character],
})
export class Human implements Character {
  id: string;
  name: string;
}
```

> info **힌트** `@ObjectType()` 데코레이터는 `@nestjs/graphql` 패키지에서 내보내집니다.

라이브러리에서 생성된 기본 `resolveType()` 함수는 리졸버 메서드에서 반환된 값을 기반으로 타입을 추출합니다. 즉, 클래스 인스턴스를 반환해야 합니다 (리터럴 JavaScript 객체는 반환할 수 없음).

사용자 정의된 `resolveType()` 함수를 제공하려면 다음과 같이 `@InterfaceType()`데코레이터에 전달된 옵션 객체에 `resolveType` 속성을 전달합니다.

```typescript
@InterfaceType({
  resolveType(book) {
    if (book.colors) {
      return ColoringBook;
    }
    return TextBook;
  },
})
export abstract class Book {
  @Field(type => ID)
  id: string;

  @Field()
  title: string;
}
```

#### Schema first

스키마 우선 접근 방식에서 인터페이스를 정의하려면 SDL을 사용하여 GraphQL 인터페이스를 생성하면됩니다.

```graphql
interface Character {
  id: ID!
  name: String!
}
```

그런 다음 타이핑 생성 기능 ([빠른 시작](/graphql/quick-start) 장에 표시된대로)을 사용하여 해당하는 TypeScript 정의를 생성할 수 있습니다.

```typescript
export interface Character {
  id: string;
  name: string;
}
```

인터페이스가 해석해야 하는 타입을 결정하려면 리졸버 맵에 추가 `__resolveType` 필드가 필요합니다. `CharactersResolver` 클래스를 만들고 `__resolveType` 메서드를 정의해 보겠습니다.

```typescript
@Resolver('Character')
export class CharactersResolver {
  @ResolveField()
  __resolveType(value) {
    if ('age' in value) {
      return Person;
    }
    return null;
  }
}
```

> info **힌트** 모든 데코레이터는 `@nestjs/graphql` 패키지에서 내보내집니다.
