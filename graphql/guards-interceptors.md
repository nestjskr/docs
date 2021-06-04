### Other features

GraphQL 세계에서는 **인증** 또는 작업의 **부작용**과 같은 문제를 처리하는 것에 대해 많은 논쟁이 있습니다. 비즈니스 로직 내부에서 처리해야합니까? 권한 부여 논리로 쿼리 및 뮤테이션을 향상시키기 위해 고차 함수를 사용해야합니까? 아니면 [스키마 지시문](https://www.apollographql.com/docs/apollo-server/schema/directives/)을 사용해야 합니까? 이러한 질문에 대한 간단한(one-size-fits-all) 대답은 없습니다.

Nest는 [guards](/guards) 및 [interceptors](/interceptors)와 같은 크로스 플랫폼 기능으로 이러한 문제를 해결하는 데 도움이 됩니다. 이 철학은 중복성을 줄이고 잘 구조화되고 읽기 가능하며 일관된 애플리케이션을 만드는 데 도움이 되는 도구를 제공하는 것입니다.

#### Overview

표준 [guards](/guards), [interceptors](/interceptors), [filters](/exception-filters) 및 [pipes](/pipes)를 모든 RESTful 응용 프로그램과 동일한 방식으로 GraphQL에서 사용할 수 있습니다. 또한 [커스텀 데코레이터](/custom-decorators) 기능을 활용하여 자신만의 데코레이터를 쉽게 만들 수 있습니다. 샘플 GraphQL 쿼리 핸들러를 살펴 보겠습니다.

```typescript
@Query('author')
@UseGuards(AuthGuard)
async getAuthor(@Args('id', ParseIntPipe) id: number) {
  return this.authorsService.findOneById(id);
}
```

보시다시피 GraphQL은 HTTP REST 핸들러와 동일한 방식으로 가드 및 파이프 모두에서 작동합니다. 이 때문에 인증 로직을 가드로 이동할 수 있습니다. REST 및 GraphQL API 인터페이스에서 동일한 가드 클래스를 재사용할 수도 있습니다. 마찬가지로 인터셉터는 두 타입의 애플리케이션에서 동일한 방식으로 작동합니다.

```typescript
@Mutation()
@UseInterceptors(EventsInterceptor)
async upvotePost(@Args('postId') postId: number) {
  return this.postsService.upvoteById({ id: postId });
}
```

#### Execution context

GraphQL은 수신 요청에서 다른 타입의 데이터를 수신하기 때문에 가드와 인터셉터가 수신하는 [실행 컨텍스트](/fundamentals/execution-context)는 GraphQL과 REST에서 다소 다릅니다. GraphQL 리졸버에는 `root`, `args`, `context` 및 `info`와 같은 고유한 전달인자 세트가 있습니다. 따라서 가드와 인터셉터는 일반 `ExecutionContext`를 `GqlExecutionContext`로 변환해야 합니다. 이것은 간단합니다.

```typescript
import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const ctx = GqlExecutionContext.create(context);
    return true;
  }
}
```

`GqlExecutionContext.create()`에서 반환된 GraphQL 컨텍스트 객체는 각 GraphQL 리졸버 전달인수 (예: `getArgs()`, `getContext()`등)에 대해 **get** 메서드를 노출합니다. 일단 변환되면 현재 요청에 대한 GraphQL 전달인수를 쉽게 선택할 수 있습니다.

#### Exception filters

Nest 표준 [예외 필터](/exception-filters)는 GraphQL 애플리케이션과도 호환됩니다. `ExecutionContext`와 마찬가지로 GraphQL 앱은 `ArgumentsHost` 객체를 `GqlArgumentsHost` 객체로 변환해야합니다.

```typescript
@Catch(HttpException)
export class HttpExceptionFilter implements GqlExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const gqlHost = GqlArgumentsHost.create(host);
    return exception;
  }
}
```

> info **힌트** `GqlExceptionFilter`와 `GqlArgumentsHost`는 모두 `@nestjs/graphql` 패키지에서 가져옵니다.

REST의 경우와 달리 응답을 생성하는 데 네이티브 `response` 객체를 사용하지 않습니다.

#### Custom decorators

언급했듯이 [커스텀 데코레이터](/custom-decorators) 기능은 GraphQL 리졸버에서 예상대로 작동합니다.

```typescript
export const User = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) =>
    GqlExecutionContext.create(ctx).getContext().user,
);
```

다음과 같이 `@User()` 커스텀 데코레이터를 사용하세요.

```typescript
@Mutation()
async upvotePost(
  @User() user: UserEntity,
  @Args('postId') postId: number,
) {}
```

> info **힌트** 위의 예에서는 `user` 객체가 GraphQL 애플리케이션의 컨텍스트에 할당되었다고 가정했습니다.

#### Execute enhancers at the field resolver level

GraphQL 컨텍스트에서 Nest는 필드 수준에서 **인핸서** (인터셉터, 가드 및 필터의 일반적인 이름)를 실행하지 않습니다 [이 문제 참조](https://github.com/nestjs/graphql/issues/320#issuecomment-511193229): 최상위 `@Query()`/`@ Mutation()` 메서드에 대해서만 실행됩니다. `GqlModuleOptions`에서 `fieldResolverEnhancers` 옵션을 설정하여 `@ResolveField()` 코멘트가 달린 메소드에 대한 인터셉터, 가드 또는 필터를 실행하도록 Nest에 지시할 수 있습니다. `'interceptors'`, `'guards'` 및/또는 `'filters'` 목록을 적절하게 전달합니다.

```typescript
GraphQLModule.forRoot({
  fieldResolverEnhancers: ['interceptors']
}),
```

> warning **경고** 필드 리졸버용 인핸서를 활성화하면 많은 레코드를 반환하고 필드 리졸버가 수천번 실행될 때 성능 문제가 발생할 수 있습니다. 따라서 `fieldResolverEnhancers`를 활성화할 때 필드 리졸버에 꼭 필요하지 않은 인핸서 실행을 건너뛰는 것이 좋습니다. 다음 도우미 기능을 사용하여 이 작업을 수행할 수 있습니다.

```typescript
export function isResolvingGraphQLField(context: ExecutionContext): boolean {
  if (context.getType<GqlContextType>() === 'graphql') {
    const gqlContext = GqlExecutionContext.create(context);
    const info = gqlContext.getInfo();
    const parentType = info.parentType.name;
    return parentType !== 'Query' && parentType !== 'Mutation';
  }
  return false;
}
```
