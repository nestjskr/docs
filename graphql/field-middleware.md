### Field middleware

> warning **경고** 이 장은 코드 우선 접근 방식에만 적용됩니다.

필드 미들웨어를 사용하면 필드가 해결되기 **전 또는 후에** 임의 코드를 실행할 수 있습니다. 필드 미들웨어를 사용하여 필드의 결과를 변환하고, 필드의 인수를 확인하거나, 필드 수준 역할을 확인할 수도 있습니다 (예: 미들웨어 기능이 실행되는 대상 필드에 액세스하는 데 필요함).

여러 미들웨어 기능을 필드에 연결할 수 있습니다. 이 경우 이전 미들웨어가 다음 미들웨어를 호출하기로 결정한 체인을 따라 순차적으로 호출됩니다. `middleware` 배열에서 미들웨어 함수의 순서가 중요합니다. 첫번째 리졸버는 "최 외곽" 계층이므로 처음과 마지막에 실행됩니다 (`graphql-middleware` 패키지와 유사). 두번째 리졸버는 "두번째 외부" 레이어이므로 두번째로 그리고 두번째에서 마지막으로 실행됩니다.

#### Getting started

클라이언트로 다시 전송되기 전에 필드 값을 기록하는 간단한 미들웨어를 만들어 시작하겠습니다.

```typescript
import { FieldMiddleware, MiddlewareContext, NextFn } from '@nestjs/graphql';

const loggerMiddleware: FieldMiddleware = async (
  ctx: MiddlewareContext,
  next: NextFn,
) => {
  const value = await next();
  console.log(value);
  return value;
};
```

> info **힌트** `MiddlewareContext`는 GraphQL 리졸버 함수 (`{{ '{' }} source, args, context, info {{ '}' }}`)에서 일반적으로 수신하는 동일한 인수로 구성된 객체입니다, `NextFn`은 스택 (이 필드에 바인딩 됨) 또는 실제 필드 리졸버에서 다음 미들웨어를 실행할 수 있는 함수입니다.

> warning **경고** 필드 미들웨어 함수는 매우 가볍고 잠재적으로 시간 소모적인 작업 (데이터베이스에서 데이터 검색)을 수행하지 않아야하므로 종속성을 주입하거나 Nest의 DI 컨테이너에 액세스할 수 없습니다. 데이터 소스에서 외부 서비스/쿼리 데이터를 호출해야 하는 경우 루트 쿼리/뮤테이션 처리기에 바인딩된 가드/인터셉터에서 이를 수행하고 필드 미들웨어 내에서 액세스할 수 있는 `context` 객체에 할당해야 합니다 (특히 `MiddlewareContext` 객체에서).

필드 미들웨어는 `FieldMiddleware` 인터페이스와 일치해야 합니다. 위의 예에서 먼저 `next()`함수 (실제 필드 리졸버를 실행하고 필드 값을 반환)를 실행한 다음 이 값을 터미널에 기록합니다. 또한 미들웨어 함수에서 반환된 값은 이전 값을 완전히 무시하고 변경을 수행하고 싶지 않기 때문에 단순히 원래 값을 반환합니다.

이를 통해 다음과 같이 `@Field()`데코레이터에 미들웨어를 직접 등록할 수 있습니다.

```typescript
@ObjectType()
export class Recipe {
  @Field({ middleware: [loggerMiddleware] })
  title: string;
}
```

이제 `Recipe` 객체 유형의 `title` 필드를 요청할 때마다 원래 필드의 값이 콘솔에 기록됩니다.

> info **힌트** [extensions](/graphql/extensions) 기능을 사용하여 필드 수준 권한 시스템을 구현하는 방법을 알아 보려면 이 [섹션](/graphql/extensions#using-custom-metadata)을 확인하세요.

또한 위에서 언급했듯이 미들웨어 기능 내에서 필드 값을 제어할 수 있습니다. 데모 목적으로 레시피 제목 (있는 경우)을 대문자로 사용하겠습니다.

```typescript
const value = await next();
return value?.toUpperCase();
```

이 경우 요청시 모든 제목이 자동으로 대문자로 표시됩니다.

마찬가지로, 다음과 같이 필드 미들웨어를 사용자 정의 필드 리졸버 (`@ResolveField()` 데코레이터로 주석이 달린 메서드)에 바인딩할 수 있습니다.

```typescript
@ResolveField(() => String, { middleware: [loggerMiddleware] })
title() {
  return 'Placeholder';
}
```

> warning **경고** 인핸서가 필드 리졸버 수준에서 활성화된 경우 ([더 읽기](/graphql/other-features#execute-enhancers-at-the-field-resolver-level)), 필드 미들웨어 기능은 인터셉터, 가드 등이 실행되기 전에 **메서드에 바인딩** (그러나 쿼리 또는 변이 핸들러에 등록 된 루트 레벨 인핸서 이후).

#### Global field middleware

미들웨어를 특정 필드에 직접 바인딩하는 것 외에도 하나 이상의 미들웨어 기능을 전역적으로 등록할 수 있습니다. 이 경우 개체 유형의 모든 필드에 자동으로 연결됩니다.

```typescript
GraphQLModule.forRoot({
  autoSchemaFile: 'schema.gql',
  buildSchemaOptions: {
    fieldMiddleware: [loggerMiddleware],
  },
}),
```

> info **힌트** 전역적으로 등록된 필드 미들웨어 기능은 로컬로 등록된 것 (특정 필드에 직접 바인딩된 것) **전에** 실행됩니다.
