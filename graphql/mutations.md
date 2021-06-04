### Mutations

GraphQL에 대한 대부분의 논의는 데이터 가져오기에 초점을 맞추지만 모든 완전한 데이터 플랫폼에는 서버측 데이터도 수정할 수 있는 방법이 필요합니다. REST에서 모든 요청은 결국 서버에 부작용을 일으킬 수 있지만 모범 사례는 GET 요청의 데이터를 수정하지 않는 것이 좋습니다. GraphQL도 유사합니다. 기술적으로 모든 쿼리를 구현하여 데이터를 쓸 수 있습니다. 그러나 REST와 마찬가지로 쓰기를 유발하는 모든 작업은 뮤테이션을 통해 명시적으로 전송되어야 한다는 규칙을 준수하는 것이 좋습니다 (자세한 내용은 [여기](https://graphql.org/learn/queries/#mutations)를 읽어보세요).

공식 [Apollo](https://www.apollographql.com/docs/graphql-tools/generate-schema.html) 문서에서는 `upvotePost()` 뮤테이션 예제를 사용합니다. 이 뮤테이션은 게시물의 `votes` 속성 값을 증가시키는 방법을 구현합니다. Nest에서 동등한 뮤테이션을 만들기 위해 `@Mutation()` 데코레이터를 사용할 것입니다.

#### Code first

이전 섹션에서 사용한 `AuthorResolver`에 다른 메소드를 추가해 보겠습니다([리졸버](/graphql/resolvers)를 보세요).

```typescript
@Mutation(returns => Post)
async upvotePost(@Args({ name: 'postId', type: () => Int }) postId: number) {
  return this.postsService.upvoteById({ id: postId });
}
```

> info **힌트** 모든 데코레이터 (예: `@Resolver`, `@ResolveField`, `@Args` 등)는 `@nestjs/graphql` 패키지에서 내보내집니다.

그러면 SDL에서 GraphQL 스키마의 다음 부분이 생성됩니다:

```graphql
type Mutation {
  upvotePost(postId: Int!): Post
}
```

`upvotePost()` 메소드는 `postId` (`Int`)를 인수로 취하고 업데이트된 `Post` 항목을 반환합니다. [리졸버](/graphql/resolvers) 섹션에 설명된 이유 때문에 예상 유형을 명시적으로 설정해야 합니다.

뮤테이션이 객체를 인수로 취해야 하는 경우 **입력 유형**을 만들 수 있습니다. 입력 유형은 인수로 전달할 수 있는 특수한 유형의 객체 유형입니다 ([여기](https://graphql.org/learn/schema/#input-types)에서 자세히 알아보세요). 입력 유형을 선언하려면 `@InputType()` 데코레이터를 사용하십시오.

```typescript
import { InputType, Field } from '@nestjs/graphql';

@InputType()
export class UpvotePostInput {
  @Field()
  postId: number;
}
```

> info **힌트** `@InputType()` 데코레이터는 옵션 객체를 인수로 사용하므로 예를 들어 입력 유형의 설명을 지정할 수 있습니다. TypeScript의 메타 데이터 반영 시스템 제한으로 인해 `@Field` 데코레이터를 사용하여 유형을 수동으로 표시하거나 [CLI 플러그인](/graphql/cli-plugin)을 사용해야 합니다.

그런 다음 리졸버 클래스에서 이 유형을 사용할 수 있습니다.

```typescript
@Mutation(returns => Post)
async upvotePost(
  @Args('upvotePostData') upvotePostData: UpvotePostInput,
) {}
```

#### Schema first

이전 섹션에서 사용한 `AuthorResolver`를 확장해 보겠습니다 ([리졸버](/graphql/resolvers) 참조).

```typescript
@Mutation()
async upvotePost(@Args('postId') postId: number) {
  return this.postsService.upvoteById({ id: postId });
}
```

위에서 비즈니스 로직이 `PostsService` (post 쿼리 및 `votes` 속성 증가)로 이동되었다고 가정했습니다. `PostsService` 클래스 내부의 로직은 필요에 따라 간단하거나 정교할 수 있습니다. 이 예제의 요점은 리졸버가 다른 프로바이더와 상호 작용할 수 있는 방법을 보여주는 것입니다.

마지막 단계는 기존 유형 정의에 변형을 추가하는 것입니다.

```graphql
type Author {
  id: Int!
  firstName: String
  lastName: String
  posts: [Post]
}

type Post {
  id: Int!
  title: String
  votes: Int
}

type Query {
  author(id: Int!): Author
}

type Mutation {
  upvotePost(postId: Int!): Post
}
```

이제 `upvotePost(postId: Int!): Post` 뮤테이션을 애플리케이션의 GraphQL API의 일부로 호출할 수 있습니다.
