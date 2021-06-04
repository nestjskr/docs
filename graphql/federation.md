### Federation

[Apollo Federation](https://www.apollographql.com/docs/apollo-server/federation/introduction/)은 모놀리식 GraphQL 서버를 독립적인 마이크로 서비스로 분할하는 수단을 제공합니다. 이는 게이트웨이와 하나 이상의 페더레이션된 마이크로 서비스의 두가지 구성 요소로 구성됩니다. 각 마이크로 서비스는 스키마의 일부를 보유하고 게이트웨이는 스키마를 클라이언트가 사용할 수 있는 단일 스키마로 병합합니다.

[Apollo 문서](https://blog.apollographql.com/apollo-federation-f260cf525d21)를 인용하기 위해 Federation은 다음과 같은 핵심 원칙으로 설계되었습니다.

- 그래프 작성은 **선언적이어야 합니다.** 연합을 사용하면 명령형 스키마 스티칭 코드를 작성하는 대신 스키마 내에서 선언적으로 그래프를 작성합니다.
- 코드는 유형이 아닌 **관심 사항**으로 구분해야 합니다. 종종 단일팀이 사용자 또는 제품과 같은 중요한 유형의 모든 측면을 제어하지 않으므로 이러한 타입의 정의는 중앙 집중식이 아닌 팀과 코드베이스에 분산되어야 합니다.
- 그래프는 클라이언트가 소비할 수 있도록 간단해야 합니다. 페더레이션 서비스는 함께 클라이언트에서 소비되는 방식을 정확하게 반영하는 완전한 제품 중심 그래프를 형성할 수 있습니다.
- 언어의 사양을 준수하는 기능만 사용하는 **GraphQL**입니다. JavaScript뿐만 아니라 모든 언어로 연합을 구현할 수 있습니다.

> warning **경고** Apollo Federation은 현재 구독을 지원하지 않습니다.

다음 예제에서는 게이트웨이와 두 개의 페더레이션된 엔드 포인트 (Users 서비스 및 Posts 서비스)가 있는 데모 애플리케이션을 설정합니다.

#### Federated example: Users

먼저 페더레이션을 위한 선택적 종속성을 설치합니다.

```bash
$ npm install --save @apollo/federation
```

#### Schema first

사용자 서비스에는 간단한 스키마가 있습니다. `@key` 지시문을 참고하십시오. 이 지시문은 `id`가 있는 경우 User의 특정 인스턴스를 가져올 수 있음을 Apollo 쿼리 플래너에 알립니다. 또한 `Query` 타입을 확장했습니다.

```graphql
type User @key(fields: "id") {
  id: ID!
  name: String!
}

extend type Query {
  getUser(id: ID!): User
}
```

리졸버에는 `resolveReference()`라는 하나의 추가 메서드가 있습니다. 관련 리소스에 User 인스턴스가 필요할 때마다 Apollo Gateway에 의해 호출됩니다. 나중에 Posts 서비스에서 이에 대한 예를 볼 수 있습니다. `@ResolveReference()` 데코레이터를 참고하세요.

```typescript
import { Args, Query, Resolver, ResolveReference } from '@nestjs/graphql';
import { UsersService } from './users.service';

@Resolver('User')
export class UsersResolvers {
  constructor(private usersService: UsersService) {}

  @Query()
  getUser(@Args('id') id: string) {
    return this.usersService.findById(id);
  }

  @ResolveReference()
  resolveReference(reference: { __typename: string; id: string }) {
    return this.usersService.findById(reference.id);
  }
}
```

마지막으로 `GraphQLFederationModule`과 함께 모듈의 모든 것을 연결합니다. 이 모듈은 일반 `GraphQLModule`과 동일한 옵션을 허용합니다.

```typescript
import { Module } from '@nestjs/common';
import { GraphQLFederationModule } from '@nestjs/graphql';
import { UsersResolvers } from './users.resolvers';

@Module({
  imports: [
    GraphQLFederationModule.forRoot({
      typePaths: ['**/*.graphql'],
    }),
  ],
  providers: [UsersResolvers],
})
export class AppModule {}
```

#### Code first

코드 우선 연합은 일반 코드 우선 GraphQL과 매우 유사합니다. `User` 엔티티에 데코레이터를 추가하기만 하면 됩니다.

```ts
import { Directive, Field, ID, ObjectType } from '@nestjs/graphql';

@ObjectType()
@Directive('@key(fields: "id")')
export class User {
  @Field((type) => ID)
  id: number;

  @Field()
  name: string;
}
```

리졸버에는 `resolveReference()`라는 하나의 추가 메서드가 있습니다. 관련 리소스에 `User` 인스턴스가 필요할 때마다 Apollo Gateway에 의해 호출됩니다. 나중에 Posts 서비스에서 이에 대한 예를 볼 수 있습니다. `@ResolveReference()` 데코레이터를 참고하세요.

```ts
import { Args, Query, Resolver, ResolveReference } from '@nestjs/graphql';
import { User } from './user.entity';
import { UsersService } from './users.service';

@Resolver((of) => User)
export class UsersResolvers {
  constructor(private usersService: UsersService) {}

  @Query((returns) => User)
  getUser(@Args('id') id: number): User {
    return this.usersService.findById(id);
  }

  @ResolveReference()
  resolveReference(reference: { __typename: string; id: number }): User {
    return this.usersService.findById(reference.id);
  }
}
```

마지막으로 `GraphQLFederationModule`과 함께 모듈의 모든 것을 연결합니다. 이 모듈은 일반 `GraphQLModule`과 동일한 옵션을 허용합니다.

```typescript
import { Module } from '@nestjs/common';
import { GraphQLFederationModule } from '@nestjs/graphql';
import { UsersResolvers } from './users.resolvers';
import { UsersService } from './users.service'; // Not included in this example

@Module({
  imports: [
    GraphQLFederationModule.forRoot({
      autoSchemaFile: true,
    }),
  ],
  providers: [UsersResolvers, UsersService],
})
export class AppModule {}
```

#### Federated example: Posts

Post 서비스는 `getPosts` 쿼리를 통해 집계된 게시물을 제공하지만 `user.posts`로 `User` 유형을 확장합니다.

#### Schema first

Posts 서비스는 `extend` 키워드로 표시하여 스키마에서 사용자 타입을 참조합니다. 또한 사용자 타입에 하나의 속성을 추가합니다. User 인스턴스를 일치시키는 데 사용되는 `@key` 지시문과 `id` 필드가 다른 곳에서 관리됨을 나타내는 `@external` 지시문에 유의하십시오.

```graphql
type Post @key(fields: "id") {
  id: ID!
  title: String!
  body: String!
  user: User
}

extend type User @key(fields: "id") {
  id: ID! @external
  posts: [Post]
}

extend type Query {
  getPosts: [Post]
}
```

우리 리졸버는 여기에 하나의 관심있는 방법인 `getUser()`가 있습니다. `__typename`을 포함하는 참조와 애플리케이션이 참조를 확인하는 데 필요한 추가 속성을 반환합니다. 이 경우에는 `id`만 반환합니다. `__typename`은 GraphQL 게이트웨이에서 사용자 타입을 담당하는 마이크로 서비스를 정확히 찾아 인스턴스를 요청하는 데 사용됩니다. 위에서 설명한 사용자 서비스는 `resolveReference()` 메소드에서 호출됩니다.

```typescript
import { Query, Resolver, Parent, ResolveField } from '@nestjs/graphql';
import { PostsService } from './posts.service';
import { Post } from './posts.interfaces';

@Resolver('Post')
export class PostsResolvers {
  constructor(private postsService: PostsService) {}

  @Query('getPosts')
  getPosts() {
    return this.postsService.findAll();
  }

  @ResolveField('user')
  getUser(@Parent() post: Post) {
    return { __typename: 'User', id: post.userId };
  }
}
```

Posts 서비스에는 거의 동일한 모듈이 있지만 완전성을 위해 아래에 포함되어 있습니다.

```typescript
import { Module } from '@nestjs/common';
import { GraphQLFederationModule } from '@nestjs/graphql';
import { PostsResolvers } from './posts.resolvers';

@Module({
  imports: [
    GraphQLFederationModule.forRoot({
      typePaths: ['**/*.graphql'],
    }),
  ],
  providers: [PostsResolvers],
})
export class AppModule {}
```

#### Code first

`User` 엔터티를 나타내는 클래스를 만들어야 합니다. 다른 서비스에 있지만 우리는 그것을 사용하고 확장할 것입니다. `@extends` 및 `@ external` 지시문을 참고하십시오.

```ts
import { Directive, ObjectType, Field, ID } from '@nestjs/graphql';
import { Post } from './post.entity';

@ObjectType()
@Directive('@extends')
@Directive('@key(fields: "id")')
export class User {
  @Field((type) => ID)
  @Directive('@external')
  id: number;

  @Field((type) => [Post])
  posts?: Post[];
}
```

다음과 같이 `User` 엔티티에 확장에 대한 리졸버를 만듭니다.

```ts
import { Parent, ResolveField, Resolver } from '@nestjs/graphql';
import { PostsService } from './posts.service';
import { Post } from './post.entity';
import { User } from './user.entity';

@Resolver((of) => User)
export class UsersResolvers {
  constructor(private readonly postsService: PostsService) {}

  @ResolveField((of) => [Post])
  public posts(@Parent() user: User): Post[] {
    return this.postsService.forAuthor(user.id);
  }
}
```

`Post` 엔티티도 만들어야 합니다.

```ts
import { Directive, Field, ID, Int, ObjectType } from '@nestjs/graphql';
import { User } from './user.entity';

@ObjectType()
@Directive('@key(fields: "id")')
export class Post {
  @Field((type) => ID)
  id: number;

  @Field()
  title: string;

  @Field((type) => Int)
  authorId: number;

  @Field((type) => User)
  user?: User;
}
```

그리고 리졸버는:

```ts
import { Query, Args, ResolveField, Resolver, Parent } from '@nestjs/graphql';
import { PostsService } from './posts.service';
import { Post } from './post.entity';
import { User } from './user.entity';

@Resolver((of) => Post)
export class PostsResolvers {
  constructor(private readonly postsService: PostsService) {}

  @Query((returns) => Post)
  findPost(@Args('id') id: number): Post {
    return this.postsService.findOne(id);
  }

  @Query((returns) => [Post])
  getPosts(): Post[] {
    return this.postsService.all();
  }

  @ResolveField((of) => User)
  user(@Parent() post: Post): any {
    return { __typename: 'User', id: post.authorId };
  }
}
```

마지막으로 모듈을 하나로 묶습니다. `User`가 외부 타입임을 지정하는 스키마 빌드 옵션에 유의하십시오.

```ts
import { Module } from '@nestjs/common';
import { GraphQLFederationModule } from '@nestjs/graphql';
import { User } from './user.entity';
import { PostsResolvers } from './posts.resolvers';
import { UsersResolvers } from './users.resolvers';
import { PostsService } from './posts.service'; // Not included in example

@Module({
  imports: [
    GraphQLFederationModule.forRoot({
      autoSchemaFile: true,
      buildSchemaOptions: {
        orphanedTypes: [User],
      },
    }),
  ],
  providers: [PostsResolvers, UsersResolvers, PostsService],
})
export class AppModule {}
```

#### Federated example: Gateway

먼저 게이트웨이에 대한 선택적 종속성을 설치하십시오.

```bash
$ npm install --save @apollo/gateway
```

우리의 게이트웨이는 엔드 포인트 목록만 필요하며 거기에서 스키마를 자동 검색합니다. 따라서 먼저 코드와 스키마에 대해 동일하며 게이트웨이에 대한 코드는 매우 짧습니다.

```typescript
import { Module } from '@nestjs/common';
import { GraphQLGatewayModule } from '@nestjs/graphql';

@Module({
  imports: [
    GraphQLGatewayModule.forRoot({
      server: {
        // ... Apollo server options
        cors: true,
      },
      gateway: {
        serviceList: [
          { name: 'users', url: 'http://user-service/graphql' },
          { name: 'posts', url: 'http://post-service/graphql' },
        ],
      },
    }),
  ],
})
export class AppModule {}
```

> info **힌트** Apollo는 프로덕션 환경에서 서비스 검색에 의존하지 않고 대신 [Graph Manager](https://www.apollographql.com/docs/graph-manager/federation/)를 사용할 것을 권장합니다. .

#### Sharing context

빌드 서비스를 사용하여 게이트웨이와 연합 서비스간의 요청을 사용자 정의할 수 있습니다. 이를 통해 요청에 대한 컨텍스트를 공유할 수 있습니다. 기본 `RemoteGraphQLDataSource`를 쉽게 확장하고 후크중 하나를 구현할 수 있습니다. 가능성에 대한 자세한 내용은 `RemoteGraphQLDataSource`의 [Apollo 문서](https://www.apollographql.com/docs/apollo-server/api/apollo-gateway/#remotegraphqldatasource)를 참조하세요.

```typescript
import { Module } from '@nestjs/common';
import { GATEWAY_BUILD_SERVICE, GraphQLGatewayModule } from '@nestjs/graphql';
import { RemoteGraphQLDataSource } from '@apollo/gateway';
import { decode } from 'jsonwebtoken';

class AuthenticatedDataSource extends RemoteGraphQLDataSource {
  async willSendRequest({ request, context }) {
    const { userId } = await decode(context.jwt);
    request.http.headers.set('x-user-id', userId);
  }
}

@Module({
  providers: [
    {
      provide: AuthenticatedDataSource,
      useValue: AuthenticatedDataSource,
    },
    {
      provide: GATEWAY_BUILD_SERVICE,
      useFactory: (AuthenticatedDataSource) => {
        return ({ name, url }) => new AuthenticatedDataSource({ url });
      },
      inject: [AuthenticatedDataSource],
    },
  ],
  exports: [GATEWAY_BUILD_SERVICE],
})
class BuildServiceModule {}

@Module({
  imports: [
    GraphQLGatewayModule.forRootAsync({
      useFactory: async () => ({
        gateway: {
          serviceList: [
            /* services */
          ],
        },
        server: {
          context: ({ req }) => ({
            jwt: req.headers.authorization,
          }),
        },
      }),
      imports: [BuildServiceModule],
      inject: [GATEWAY_BUILD_SERVICE],
    }),
  ],
})
export class AppModule {}
```

#### Async configuration

연합 및 게이트웨이 모듈은 모두 [빠른 시작](/graphql/quick-start#async-configuration)에 설명된 동일한 `forRootAsync`를 사용하는 비동기 초기화를 지원합니다.
