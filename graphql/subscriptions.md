### Subscriptions

쿼리를 사용하여 데이터를 가져오고 변형을 사용하여 데이터를 수정하는 것 외에도 GraphQL 사양은 `구독 subscription`이라는 세번째 작업 유형을 지원합니다. GraphQL 구독은 서버에서 실시간 메시지를 수신하도록 선택한 클라이언트로 서버에서 데이터를 푸시하는 방법입니다. 구독은 클라이언트에 전달할 필드 집합을 지정한다는 점에서 쿼리와 유사하지만 단일 응답을 즉시 반환하는 대신 채널이 열리고 서버에서 특정 이벤트가 발생할 때마다 결과가 클라이언트에 전송됩니다. .

구독의 일반적인 사용 사례는 새 객체 생성, 업데이트된 필드 등과 같은 특정 이벤트에 대해 클라이언트 측에 알리는 것입니다 (자세한 내용은 [여기](https://www.apollographql.com/docs/react/data/subscriptions/) 참조).

#### Enable subscriptions

구독을 활성화하려면 `installSubscriptionHandlers` 속성을 `true`로 설정합니다.

```typescript
GraphQLModule.forRoot({
  installSubscriptionHandlers: true,
}),
```

#### Code first

코드 우선 접근 방식을 사용하여 구독을 생성하려면 간단한 **publish/subscribe API**를 제공하는 `graphql-subscriptions` 패키지의 `@Subscription()` 데코레이터와 `PubSub` 클래스를 사용합니다.

다음 구독 핸들러는 `PubSub#asyncIterator`를 호출하여 이벤트에 대한 **구독**을 처리합니다. 이 메소드는 이벤트 주제 이름에 해당하는 `triggerName`이라는 단일 인수를 사용합니다.

```typescript
const pubSub = new PubSub();

@Resolver(of => Author)
export class AuthorResolver {
  // ...
  @Subscription(returns => Comment)
  commentAdded() {
    return pubSub.asyncIterator('commentAdded');
  }
}
```

> info **힌트** 모든 데코레이터는 `@nestjs/graphql` 패키지에서 내보내는 반면 `PubSub`  클래스는 `graphql-subscriptions` 패키지에서 내보내집니다.

> warning **참고** `PubSub`는 간단한 `publish` 및 `subscribe API`를 노출하는 클래스입니다. [여기](https://www.apollographql.com/docs/graphql-subscriptions/setup.html)에서 자세히 알아보세요. Apollo 문서는 기본 구현이 프로덕션에 적합하지 않다고 경고합니다 ([여기](https://github.com/apollographql/graphql-subscriptions#getting-started-with-your-first-subscription)). 프로덕션 앱은 외부 저장소에서 지원하는 `PubSub`구현을 사용해야 합니다 ([여기](https://github.com/apollographql/graphql-subscriptions#pubsub-implementations)에서 자세히 알아보기).

그러면 SDL에서 GraphQL 스키마의 다음 부분이 생성됩니다:

```graphql
type Subscription {
  commentAdded(): Comment!
}
```

정의에 따라 구독은 키가 구독의 이름인 단일 최상위 속성이 있는 객체를 반환합니다. 이 이름은 구독 핸들러 메서드의 이름 (즉, 위의 `commentAdded`)에서 상속되거나 `@Subscription()` 데코레이터에 두번째 인수로 `name` 키가 있는 옵션을 전달하여 명시적으로 제공됩니다. 아래와 같이.

```typescript
  @Subscription(returns => Comment, {
    name: 'commentAdded',
  })
  addCommentHandler() {
    return pubSub.asyncIterator('commentAdded');
  }
```

이 구조는 이전 코드 샘플과 동일한 SDL을 생성하지만 구독에서 메서드 이름을 분리할 수 있습니다.

#### Publishing

이제 이벤트를 게시하기 위해 `PubSub#publish` 메소드를 사용합니다. 이것은 객체 그래프의 일부가 변경되었을 때 클라이언트측 업데이트를 트리거하기 위해 뮤테이션내에서 자주 사용됩니다. 예를 들면:

```typescript
@@filename(posts/posts.resolver)
@Mutation(returns => Post)
async addComment(
  @Args('postId', { type: () => Int }) postId: number,
  @Args('comment', { type: () => Comment }) comment: CommentInput,
) {
  const newComment = this.commentsService.addComment({ id: postId, comment });
  pubSub.publish('commentAdded', { commentAdded: newComment });
  return newComment;
}
```

`PubSub#publish` 메소드는 첫번째 매개 변수로 `triggerName` (다시 말하지만 이벤트 주제 이름으로 생각)을, 두번째 매개 변수로 이벤트 페이로드를 사용합니다. 언급했듯이 구독은 정의에 따라 값을 반환하고 해당 값은 모양(shape)을 갖습니다. `commentAdded` 구독에 대해 생성된 SDL을 다시 살펴보십시오.

```graphql
type Subscription {
  commentAdded(): Comment!
}
```

이것은 구독이 `Comment` 객체인 값을 가진 `commentAdded`라는 최상위 속성 이름을 가진 객체를 반환해야 함을 알려줍니다. 주목해야 할 중요한 점은 `PubSub#publish` 메서드에서 내보낸 이벤트 페이로드의 모양(shape)이 구독에서 반환될 것으로 예상되는 값의 모양과 일치해야 한다는 것입니다. 따라서 위의 예에서 `pubSub.publish('commentAdded', {{ '{' }} commentAdded: newComment {{ '}' }})`문은 적절한 모양의 페이로드와 함께 `commentAdded` 이벤트를 게시합니다. 이러한 모양(shape)가 일치하지 않으면 GraphQL 유효성 검사 단계에서 구독이 실패합니다.

#### Filtering subscriptions

특정 이벤트를 필터링하려면 `filter` 속성을 필터 함수로 설정하세요. 이 함수는 배열 `filter`에 전달된 함수와 유사하게 작동합니다. 이벤트 페이로드 (이벤트 게시자가 보낸대로)를 포함하는 `payload`와 구독 요청 중에 전달된 모든 인수를 받는 `variables`의 두 가지 인수가 필요합니다. 이 이벤트가 클라이언트 리스너에 게시되어야 하는지 여부를 결정하는 부울(boolean)을 반환합니다.

```typescript
@Subscription(returns => Comment, {
  filter: (payload, variables) =>
    payload.commentAdded.title === variables.title,
})
commentAdded(@Args('title') title: string) {
  return pubSub.asyncIterator('commentAdded');
}
```

#### Mutating subscription payloads

게시된 이벤트 페이로드를 변경하려면 `resolve` 속성을 함수로 설정합니다. 이 함수는 이벤트 게시자가 보낸 이벤트 페이로드를 수신하고 적절한 값을 반환합니다.

```typescript
@Subscription(returns => Comment, {
  resolve: value => value,
})
commentAdded() {
  return pubSub.asyncIterator('commentAdded');
}
```

> warning **참고** `resolve` 옵션을 사용하는 경우 래핑되지 않은 페이로드를 반환해야 합니다 (예: 이 예제에서는 `{{ '{' }} commentAdded: newComment {{ '}' }}` 객체).

삽입된 공급자에 액세스해야 하는 경우 (예: 외부 서비스를 사용하여 데이터 유효성 검사) 다음 구성을 사용합니다.

```typescript
@Subscription(returns => Comment, {
  resolve(this: AuthorResolver, value) {
    // "this" refers to an instance of "AuthorResolver"
    return value;
  }
})
commentAdded() {
  return pubSub.asyncIterator('commentAdded');
}
```

동일한 구성이 필터에서도 작동합니다.

```typescript
@Subscription(returns => Comment, {
  filter(this: AuthorResolver, payload, variables) {
    // "this" refers to an instance of "AuthorResolver"
    return payload.commentAdded.title === variables.title;
  }
})
commentAdded() {
  return pubSub.asyncIterator('commentAdded');
}
```

#### Schema first

Nest에서 동등한 구독을 생성하기 위해 `@Subscription()` 데코레이터를 사용할 것입니다.

```typescript
const pubSub = new PubSub();

@Resolver('Author')
export class AuthorResolver {
  // ...
  @Subscription()
  commentAdded() {
    return pubSub.asyncIterator('commentAdded');
  }
}
```

컨텍스트 및 인수를 기반으로 특정 이벤트를 필터링하려면 `filter` 속성을 설정하세요.

```typescript
@Subscription('commentAdded', {
  filter: (payload, variables) =>
    payload.commentAdded.title === variables.title,
})
commentAdded() {
  return pubSub.asyncIterator('commentAdded');
}
```

게시된 페이로드를 변경(mutate)하려면 `resolve` 함수를 사용할 수 있습니다.

```typescript
@Subscription('commentAdded', {
  resolve: value => value,
})
commentAdded() {
  return pubSub.asyncIterator('commentAdded');
}
```

삽입된 프로바이더에 액세스해야 하는 경우 (예: 외부 서비스를 사용하여 데이터 유효성 검사) 다음 구성을 사용합니다.

```typescript
@Subscription('commentAdded', {
  resolve(this: AuthorResolver, value) {
    // "this" refers to an instance of "AuthorResolver"
    return value;
  }
})
commentAdded() {
  return pubSub.asyncIterator('commentAdded');
}
```

동일한 구성이 필터에서도 작동합니다.

```typescript
@Subscription('commentAdded', {
  filter(this: AuthorResolver, payload, variables) {
    // "this" refers to an instance of "AuthorResolver"
    return payload.commentAdded.title === variables.title;
  }
})
commentAdded() {
  return pubSub.asyncIterator('commentAdded');
}
```

마지막 단계는 유형 정의 파일을 업데이트하는 것입니다.

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

type Comment {
  id: String
  content: String
}

type Subscription {
  commentAdded(title: String!): Comment
}
```

이를 통해 단일 `commentAdded(title: String!): Comment` 구독을 만들었습니다. 전체 샘플 구현은 [여기](https://github.com/nestjs/nest/blob/master/sample/12-graphql-schema-first)에서 확인할 수 있습니다.

#### PubSub

위에서 로컬 `PubSub` 인스턴스를 인스턴스화했습니다. 선호되는 접근 방식은 `PubSub`를 [provider](/fundamentals/custom-providers)로 정의하고 생성자(constructor)를 통해 삽입하는 것입니다 (`@Inject()` 데코레이터 사용). 이를 통해 전체 애플리케이션에서 인스턴스를 재사용할 수 있습니다. 예를 들어 다음과 같이 프로바이더를 정의한 다음 필요한 곳에 `'PUB_SUB'`를 삽입합니다.

```typescript
{
  provide: 'PUB_SUB',
  useValue: new PubSub(),
}
```

#### Customize subscriptions server

구독 서버를 사용자 지정하려면 (예: 리스너 포트 변경) `subscriptions` 옵션 속성을 사용합니다 ([자세한 것을 읽어보세요](https://www.apollographql.com/docs/apollo-server/v2/api/apollo-server.html#constructor-options-lt-ApolloServer-gt)).

```typescript
GraphQLModule.forRoot({
  installSubscriptionHandlers: true,
  subscriptions: {
    keepAlive: 5000,
  }
}),
```
