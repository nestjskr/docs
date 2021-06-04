### Directives

지시문(directive)은 필드 또는 조각 포함에 첨부될 수 있으며 서버가 원하는 방식으로 쿼리 실행에 영향을 미칠 수 있습니다 ([여기](https://graphql.org/learn/queries/#directives)에서 자세히 읽어보십시오). GraphQL 사양은 몇 가지 기본 지시문(directive)을 제공합니다.

- `@include(if: Boolean)` - 인수가 true 인 경우에만 결과에 이 필드를 포함합니다.
- `@skip(if: Boolean)` - 인수가 참이면 이 필드를 건너 뜁니다.
- `@deprecated(reason: String)` - 메시지와 함께 사용되지 않음으로 필드 표시

지시문은 앞에 `@`문자가 붙은 식별자이며, 선택적으로 이름이 지정된 인수 목록이 뒤 따릅니다. 이는 GraphQL 쿼리 및 스키마 언어의 거의 모든 요소 뒤에 나타날 수 있습니다.

#### Custom directives

커스텀 스키마 지시문을 생성하려면 `apollo-server` 패키지에서 내보낸 `SchemaDirectiveVisitor` 클래스를 확장하는 클래스를 선언합니다.

```typescript
import { SchemaDirectiveVisitor } from 'apollo-server';
import { defaultFieldResolver, GraphQLField } from 'graphql';

export class UpperCaseDirective extends SchemaDirectiveVisitor {
  visitFieldDefinition(field: GraphQLField<any, any>) {
    const { resolve = defaultFieldResolver } = field;
    field.resolve = async function(...args) {
      const result = await resolve.apply(this, args);
      if (typeof result === 'string') {
        return result.toUpperCase();
      }
      return result;
    };
  }
}
```

> info **힌트** 지시문은 `@Injectable()` 데코레이터로 데코레이팅할 수 없습니다. 따라서 종속성을 주입할 수 없습니다.

이제 `GraphQLModule.forRoot()` 메서드에 `UpperCaseDirective`를 등록합니다.

```typescript
GraphQLModule.forRoot({
  // ...
  schemaDirectives: {
    upper: UpperCaseDirective,
  },
});
```

일단 등록되면 `@upper` 지시문을 스키마에서 사용할 수 있습니다. 그러나 지시문을 적용하는 방법은 사용하는 접근 방식 (코드 우선 또는 스키마 우선)에 따라 다릅니다.

#### Code first

코드 우선 접근 방식에서는 `@Directive()` 데코레이터를 사용하여 지시문을 적용합니다.

```typescript
@Directive('@upper')
@Field()
title: string;
```

> info **힌트** `@Directive()` 데코레이터는 `@nestjs/graphql` 패키지에서 내보내집니다.

지시문은 필드, 필드 리졸버, 입력 및 객체 타입은 물론 쿼리, 뮤테이션 및 구독에 적용될 수 있습니다. 다음은 쿼리 핸들러 수준에 적용된 지시문의 예입니다.

```typescript
@Directive('@deprecated(reason: "This query will be removed in the next version")')
@Query(returns => Author, { name: 'author' })
async getAuthor(@Args({ name: 'id', type: () => Int }) id: number) {
  return this.authorsService.findOneById(id);
}
```

`@Directive()` 데코레이터를 통해 적용된 지시문은 생성된 스키마 정의 파일에 반영되지 않습니다.

#### Schema first

스키마 우선 접근 방식에서는 SDL에서 직접 지시문을 적용합니다.

```graphql
directive @upper on FIELD_DEFINITION

type Post {
  id: Int!
  title: String! @upper
  votes: Int
}
```
