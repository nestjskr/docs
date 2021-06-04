### Resolvers

리졸버는 [GraphQL](https://graphql.org/) 작업(쿼리(Query), 뮤테이션(mutation) 또는 구독(sbuscription))을 데이터로 변환하기 위한 지침을 제공합니다. 그들은 우리가 스키마에서 지정한 것과 같은 형태의 데이터를 동기적으로 또는 그 형태의 결과로 해결되는 약속으로 반환합니다. 일반적으로 **리졸버 맵**을 수동으로 만듭니다. 반면 `@nestjs/graphql` 패키지는 클래스에 주석을 달기 위해 사용하는 데코레이터가 제공하는 메타 데이터를 사용하여 자동으로 리졸버 맵을 생성합니다. 패키지 기능을 사용하여 GraphQL API를 만드는 과정을 보여주기 위해 간단한 작성자 API를 만듭니다.

#### Code first

코드 우선 접근 방식에서는 GraphQL SDL을 직접 작성하여 GraphQL 스키마를 만드는 일반적인 프로세스를 따르지 않습니다. 대신 TypeScript 데코레이터를 사용하여 TypeScript 클래스 정의에서 SDL을 생성합니다. `@nestjs/graphql` 패키지는 데코레이터를 통해 정의된 메타 데이터를 읽고 자동으로 스키마를 생성합니다.

#### Object types

GraphQL 스키마의 대부분의 정의는 **객체 유형**입니다. 정의하는 각 개체 유형은 응용 프로그램 클라이언트가 상호 작용해야 하는 도메인 객체를 나타내야합니다. 예를 들어 샘플 API는 작성자 및 게시물 목록을 가져올 수 있어야하므로 이 기능을 지원하려면 `Author` 유형과 `Post` 유형을 정의해야 합니다.

스키마 우선 접근 방식을 사용하는 경우 다음과 같이 SDL을 사용하여 이러한 스키마를 정의합니다.

```graphql
type Author {
  id: Int!
  firstName: String
  lastName: String
  posts: [Post]
}
```

이 경우 코드 우선 접근 방식을 사용하여 TypeScript 클래스를 사용하고 TypeScript 데코레이터를 사용하여 해당 클래스의 필드에 주석을 달아 스키마를 정의합니다. 코드 우선 접근 방식에서 위의 SDL에 해당하는 것은 다음과 같습니다.

```typescript
@@filename(authors/models/author.model)
import { Field, Int, ObjectType } from '@nestjs/graphql';
import { Post } from './post';

@ObjectType()
export class Author {
  @Field(type => Int)
  id: number;

  @Field({ nullable: true })
  firstName?: string;

  @Field({ nullable: true })
  lastName?: string;

  @Field(type => [Post])
  posts: Post[];
}
```

> info **힌트** TypeScript의 메타 데이터 리플렉션 시스템에는 몇가지 제한 사항이 있습니다. 예를 들어 클래스가 어떤 속성으로 구성되어 있는지 확인하거나 주어진 속성이 선택 사항인지 필수인지 여부를 인식할 수 없습니다. 이러한 제한으로 인해 스키마 정의 클래스에서 `@Field()` 데코레이터를 명시적으로 사용하여 각 필드의 GraphQL 유형 및 옵션에 대한 메타 데이터를 제공하거나 [CLI 플러그인] (/graphql/cli-plugin)을 사용하여 우리를 위해 생성합니다.

다른 클래스와 마찬가지로 `Author` 객체 유형은 필드 모음으로 구성되며 각 필드는 유형을 선언합니다. 필드의 유형은 [GraphQL 유형](https://graphql.org/learn/schema/)에 해당합니다. 필드의 GraphQL 유형은 다른 객체 유형이거나 스칼라 유형일 수 있습니다. GraphQL 스칼라 유형은 단일 값으로 해석되는 프리미티브(`ID`,`String`,`Boolean` 또는`Int`와 같은)입니다.

> info **힌트** GraphQL의 내장 스칼라 유형 외에도 사용자 정의 스칼라 유형을 정의할 수 있습니다 ([더 읽어보기](/graphql/scalars)).

위의 `Author` 객체 유형 정의는 Nest가 위에서 보여준 SDL을 **생성**합니다.

```graphql
type Author {
  id: Int!
  firstName: String
  lastName: String
  posts: [Post]
}
```

`@Field()` 데코레이터는 선택적 유형 함수 (예: `type => Int`)와 선택적으로 옵션 객체를 허용합니다.

TypeScript 유형 시스템과 GraphQL 유형 시스템간에 모호성이 있을 가능성이 있는 경우 유형 함수가 필요합니다. 특히 `문자열 string`및 `부울 boolean`유형에는 필수 사항이 **아닙니다**. `숫자 number`에 **필요**합니다 (GraphQL `Int` 또는 `Float`에 매핑되어야 함). 유형 함수는 단순히 원하는 GraphQL 유형을 반환해야 합니다 (이 장의 다양한 예에서 볼 수 있음).

옵션 개체는 다음 키/값 쌍을 가질 수 있습니다.

- `nullable`: 필드가 nullable인지 여부를 지정합니다 (SDL에서 각 필드는 기본적으로 nullable이 아님); `boolean`
- `description` : 필드 설명을 설정합니다; `string`
- `deprecationReason` : 필드를 사용되지 않음으로 표시 `string`

예:

```typescript
@Field({ description: `Book title`, deprecationReason: 'Not useful in v2 schema' })
title: string;
```

> info **힌트** 전체 개체 유형에 대한 설명을 추가하거나 지원 중단할 수도 있습니다: `@ObjectType({{ '{' }} description: 'Author model' {{ '}' }})`.

필드가 배열인 경우 아래와 같이 `Field()` 데코레이터의 유형 함수에서 배열 유형을 수동으로 표시해야 합니다.

```typescript
@Field(type => [Post])
posts: Post[];
```

> info **힌트** 배열 대괄호 표기법 (`[ ]`)을 사용하여 배열의 깊이를 나타낼 수 있습니다. 예를 들어 `[[Int]]`를 사용하면 정수 행렬을 나타냅니다.

배열의 항목 (배열 자체가 아님)이 nullable임을 선언하려면 아래와 같이 `nullable` 속성을 `'items'`로 설정합니다.

```typescript
@Field(type => [Post], { nullable: 'items' })
posts: Post[];
```

> info **힌트** 배열과 해당 항목이 모두 nullable이면 `nullable`을 `'itemsAndList'`로 설정하세요.

이제 `Author` 객체 유형이 생성되었으므로 `Post` 객체 유형을 정의해 보겠습니다.

```typescript
@@filename(posts/models/post.model)
import { Field, Int, ObjectType } from '@nestjs/graphql';

@ObjectType()
export class Post {
  @Field(type => Int)
  id: number;

  @Field()
  title: string;

  @Field(type => Int, { nullable: true })
  votes?: number;
}
```

`Post` 객체 유형은 SDL에서 GraphQL 스키마의 다음 부분을 생성합니다.

```graphql
type Post {
  id: Int!
  title: String!
  votes: Int
}
```

#### Code first resolver

이 시점에서 데이터 그래프에 존재할 수 있는 객체 (유형 정의)를 정의했지만 클라이언트는 아직 이러한 객체와 상호 작용할 수 있는 방법이 없습니다. 이를 해결하려면 리졸버 클래스를 만들어야 합니다. 코드 우선 방법에서 리졸버 클래스는 리졸버 함수를 정의**하고** **쿼리 유형**을 생성합니다. 아래 예를 살펴보면 명확해집니다.

```typescript
@@filename(authors/authors.resolver)
@Resolver(of => Author)
export class AuthorsResolver {
  constructor(
    private authorsService: AuthorsService,
    private postsService: PostsService,
  ) {}

  @Query(returns => Author)
  async author(@Args('id', { type: () => Int }) id: number) {
    return this.authorsService.findOneById(id);
  }

  @ResolveField()
  async posts(@Parent() author: Author) {
    const { id } = author;
    return this.postsService.findAll({ authorId: id });
  }
}
```


> info **힌트** 모든 데코레이터 (예: `@ Resolver`, `@ ResolveField`, `@ Args` 등)는 `@nestjs/graphql` 패키지에서 내보내집니다.

여러 리졸버 클래스를 정의할 수 있습니다. Nest는 런타임에 이를 결합합니다. 코드 구성에 대한 자세한 내용은 아래의 [module](/graphql/resolvers#module) 섹션을 참조하십시오.

> warning **참고** `AuthorsService` 및 `PostsService` 클래스 내부의 논리는 필요에 따라 간단하거나 정교할 수 있습니다. 이 예제의 요점은 리졸버를 구성하는 방법과 리졸버가 다른 프로바이더와 상호 작용할 수 있는 방법을 보여주는 것입니다.

위의 예에서 우리는 하나의 쿼리 리졸버 함수와 하나의 필드 리졸버 함수를 정의하는 `AuthorsResolver`를 만들었습니다. 리졸버를 생성하기 위해 리졸버 함수를 메소드로 사용하는 클래스를 생성하고 `@Resolver()` 데코레이터로 클래스에 주석을 추가합니다.

이 예에서는 요청에서 전송된 `id`를 기반으로 author 객체를 가져 오는 쿼리 핸들러를 정의했습니다. 메소드가 쿼리 핸들러임을 지정하려면 `@Query()`데코레이터를 사용하십시오.

`@Resolver()` 데코레이터에 전달된 인수는 선택 사항이지만 그래프가 사소하지 않게될 때 작동합니다. 객체 그래프를 통해 아래로 이동할 때 필드 확인자 함수에서 사용하는 부모 객체를 제공하는 데 사용됩니다.


이 예에서 클래스에는 **필드 리졸버** 함수 (`Author` 객체 유형의 `posts` 속성용)가 포함되어 있으므로 **반드시** `@Resolver()` 데코레이터에 값을 제공해야 합니다. 이 클래스 내에 정의된 모든 필드 리졸버에 대해 어떤 클래스가 상위 유형 (즉, 해당 `ObjectType` 클래스 이름)인지 표시합니다. 예제에서 명확하게 알 수 있듯이 필드 확인자 함수를 작성할 때 부모 객체 (확인중인 필드가 구성원인 객체)에 액세스해야 합니다. 이 예에서는 author의 `id`를 인수로 취하는 서비스를 호출하는 필드 리졸버로 작성자의 posts 배열을 채웁니다. 따라서 `@Resolver()` 데코레이터에서 상위 객체를 식별해야 합니다. `@Parent()` 메소드 매개 변수 데코레이터를 사용하여 필드 리졸버에서 해당 상위 객체에 대한 참조를 추출합니다.

여러 `@Query()` 리졸버 함수 (이 클래스 내 및 다른 리졸버 클래스 모두)를 정의할 수 있으며, 생성된 SDL에서 적절한 항목과 함께 단일 **쿼리 유형** 정의로 집계됩니다. 리졸버 맵에서. 이를 통해 사용하는 모델 및 서비스에 가까운 쿼리를 정의하고 모듈에서 잘 구성할 수 있습니다.

> info **힌트** Nest CLI는 **모든 상용구 코드**를 자동으로 생성하는 생성기(generator) (스키메틱)를 제공하여 이 모든 작업을 피하고 개발자 환경을 훨씬 더 간단하게 만들 수 있습니다. 이 기능에 대한 자세한 내용은 [여기](/recipes/crud-generator)를 참조하세요.

#### Query type names

위의 예에서 `@Query ()` 데코레이터는 메서드 이름을 기반으로 GraphQL 스키마 쿼리 유형 이름을 생성합니다. 예를 들어, 위의 예에서 다음 구성을 고려하십시오.

```typescript
@Query(returns => Author)
async author(@Args('id', { type: () => Int }) id: number) {
  return this.authorsService.findOneById(id);
}
```

그러면 스키마의 작성자 쿼리에 대해 다음 항목이 생성됩니다 (쿼리 유형은 메서드 이름과 동일한 이름을 사용함).

```graphql
type Query {
  author(id: Int!): Author
}
```

> info **힌트** [여기](https://graphql.org/learn/queries/)에서 GraphQL 쿼리에 대해 자세히 알아보세요.

일반적으로 우리는 이러한 이름을 분리하는 것을 선호합니다. 예를 들어 쿼리 핸들러 메서드에 `getAuthor()`와 같은 이름을 사용하는 것을 선호하지만 쿼리 유형 이름에는 여전히 `author`를 사용합니다. 필드 리졸버에도 동일하게 적용됩니다. 아래와 같이 `@Query()` 및 `@ResolveField()` 데코레이터의 인수로 매핑 이름을 전달하여 쉽게 수행할 수 있습니다.

```typescript
@@filename(authors/authors.resolver)
@Resolver(of => Author)
export class AuthorsResolver {
  constructor(
    private authorsService: AuthorsService,
    private postsService: PostsService,
  ) {}

  @Query(returns => Author, { name: 'author' })
  async getAuthor(@Args('id', { type: () => Int }) id: number) {
    return this.authorsService.findOneById(id);
  }

  @ResolveField('posts', returns => [Post])
  async getPosts(@Parent() author: Author) {
    const { id } = author;
    return this.postsService.findAll({ authorId: id });
  }
}
```

위의 `getAuthor` 핸들러 메서드는 SDL에서 GraphQL 스키마의 다음 부분을 생성합니다.

```graphql
type Query {
  author(id: Int!): Author
}
```

#### Query decorator options

`@Query()` 데코레이터의 옵션 객체 (위에서 `{{ '{' }}name: 'author'{{ '}' }}`를 전달함)는 여러 키/값 쌍을 허용합니다.

- `name`: 쿼리 이름; `string`
- `description`: GraphQL 스키마 문서를 생성하는 데 사용되는 설명 (예: GraphQL 플레이 그라운드에서) `string`
- `deprecationReason`: 쿼리를 지원 중단된 것으로 표시하도록 쿼리 메타 데이터를 설정합니다 (예: GraphQL 플레이 그라운드에서). `string`
- `nullable`: 쿼리가 null 데이터 응답을 반환할 수 있는지 여부. `boolean` 또는 `'items'` 또는 `'itemsAndList'` (`'items'` 및 `'itemsAndList'`에 대한 자세한 내용은 위 참조)

#### Args decorator options

메소드 핸들러에서 사용할 요청에서 인수를 추출하려면 `@Args()` 데코레이터를 사용하십시오. 이것은 [REST 경로 매개 변수 인수 추출](/controllers#route-parameters)과 매우 유사한 방식으로 작동합니다.

일반적으로 `@Args()` 데코레이터는 간단하며 위의 `getAuthor()` 메소드에서 볼 수 있듯이 객체 인수가 필요하지 않습니다. 예를 들어 식별자 유형이 문자열인 경우 다음 구성으로 충분하며 메서드 인수로 사용하기 위해 인바운드 GraphQL 요청에서 명명된 필드를 간단히 뽑습니다.

```typescript
@Args('id') id: string
```

`getAuthor()`의 경우 `number` 유형이 사용되어 챌린지를 표시합니다. `number` TypeScript 유형은 예상되는 GraphQL 표현에 대한 충분한 정보를 제공하지 않습니다 (예: `Int` 대 `Float`). 따라서 유형 참조를 **명시적으로** 전달해야 합니다. 다음과 같이 인수 옵션을 포함하는 두번째 인수를 `Args()` 데코레이터에 전달하면 됩니다.

```typescript
@Query(returns => Author, { name: 'author' })
async getAuthor(@Args('id', { type: () => Int }) id: number) {
  return this.authorsService.findOneById(id);
}
```

옵션 개체를 사용하면 다음과 같은 선택적 키 값 쌍을 지정할 수 있습니다:

- `type`: GraphQL 유형을 반환하는 함수
- `defaultValue`: 기본값; `any`
- `description`: 설명 메타 데이터; `string`
- `deprecationReason`: 필드를 더 이상 사용하지 않고 이유를 설명하는 메타 데이터 제공; `string`
- `nullable`: 필드가 널 입력 가능 여부

쿼리 핸들러 메서드는 여러 인수를 사용할 수 있습니다. `firstName` 과 `lastName`을 기반으로 author를 가져오고 싶다고 가정해 보겠습니다. 이 경우 `@Args`를 두번 호출할 수 있습니다.

```typescript
getAuthor(
  @Args('firstName', { nullable: true }) firstName?: string,
  @Args('lastName', { defaultValue: '' }) lastName?: string,
) {}
```

#### Dedicated arguments class

인라인 `@Args()` 호출을 사용하면 위 예제와 같은 코드가 부풀어집니다. 대신 전용 `GetAuthorArgs` 인수 클래스를 만들고 다음과 같이 핸들러 메서드에서 액세스할 수 있습니다.

```typescript
@Args() args: GetAuthorArgs
```

아래와 같이 `@ArgsType()`을 사용하여 `GetAuthorArgs` 클래스를 만듭니다:


```typescript
@@filename(authors/dto/get-author.args)
import { MinLength } from 'class-validator';
import { Field, ArgsType } from '@nestjs/graphql';

@ArgsType()
class GetAuthorArgs {
  @Field({ nullable: true })
  firstName?: string;

  @Field({ defaultValue: '' })
  @MinLength(3)
  lastName: string;
}
```

> info **힌트** 다시 말하지만, TypeScript의 메타 데이터 반영 시스템 제한으로 인해 `@Field` 데코레이터를 사용하여 유형과 옵션을 수동으로 표시하거나 [CLI 플러그인](/graphql/cli-plugin)을 사용해야 합니다.

그러면 SDL에서 GraphQL 스키마의 다음 부분이 생성됩니다:

```graphql
type Query {
  author(firstName: String, lastName: String = ''): Author
}
```

> info **힌트** `GetAuthorArgs`와 같은 인수 클래스는 `ValidationPipe`와 매우 잘 작동합니다 ([더 읽어보기](/techniques/validation)).

#### Class inheritance

Base `@ArgsType()` class:
표준 TypeScript 클래스 상속을 사용하여 확장 가능한 일반 유틸리티 유형 기능 (필드 및 필드 속성, 유효성 검사 등)이 있는 기본 클래스를 만들 수 있습니다. 예를 들어 항상 표준 `offset`및 `limit`필드를 포함하는 페이지 매김 관련 인수 집합이 있을 수 있지만 유형별 다른 색인 필드도 포함될 수 있습니다. 아래와 같이 클래스 계층을 설정할 수 있습니다.

기본 `@ArgsType()` 클래스:

```typescript
@ArgsType()
class PaginationArgs {
  @Field((type) => Int)
  offset: number = 0;

  @Field((type) => Int)
  limit: number = 10;
}
```

기본 `@ArgsType()` 클래스의 유형별 서브 클래스:

```typescript
@ArgsType()
class GetAuthorArgs extends PaginationArgs {
  @Field({ nullable: true })
  firstName?: string;

  @Field({ defaultValue: '' })
  @MinLength(3)
  lastName: string;
}
```

`@ObjectType()`객체에 대해서도 동일한 접근 방식을 취할 수 있습니다. 기본 클래스에 대한 일반 속성을 정의합니다.

```typescript
@ObjectType()
class Character {
  @Field((type) => Int)
  id: number;

  @Field()
  name: string;
}
```

서브 클래스에 유형별 속성을 추가합니다.

```typescript
@ObjectType()
class Warrior extends Character {
  @Field()
  level: number;
}
```

리졸버와 함께 상속을 사용할 수도 있습니다. 상속과 TypeScript 제네릭을 결합하여 형식 안전성을 보장할 수 있습니다. 예를 들어 일반적인 `findAll` 쿼리로 기본 클래스를 만들려면 다음과 같은 구성을 사용합니다.

```typescript
function BaseResolver<T extends Type<unknown>>(classRef: T): any {
  @Resolver({ isAbstract: true })
  abstract class BaseResolverHost {
    @Query((type) => [classRef], { name: `findAll${classRef.name}` })
    async findAll(): Promise<T[]> {
      return [];
    }
  }
  return BaseResolverHost;
}
```

다음 사항에 유의하십시오.

- 명시적인 반환 유형 (위의 `any`)이 필요합니다. 그렇지 않으면 TypeScript가 개인 클래스 정의 사용에 대해 불평합니다. 권장합니다: `any`를 사용하는 대신 인터페이스를 정의하십시오.
- `@nestjs/common` 패키지에서 `Type`을 가져옵니다.
- `isAbstract: true` 속성은 이 클래스에 대해 SDL (스키마 정의 언어 문)을 생성하지 않아야 함을 나타냅니다. SDL 생성을 억제하기 위해 다른 유형에 대해서도 이 속성을 설정할 수 있습니다.

`BaseResolver`의 구체적인 서브 클래스를 생성하는 방법은 다음과 같습니다.

```typescript
@Resolver((of) => Recipe)
export class RecipesResolver extends BaseResolver(Recipe) {
  constructor(private recipesService: RecipesService) {
    super();
  }
}
```

이 구조는 다음 SDL을 생성합니다.

```graphql
type Query {
  findAllRecipe: [Recipe!]!
}
```

#### Generics

우리는 위에서 제네릭의 한가지 사용을 보았습니다. 이 강력한 TypeScript 기능을 사용하여 유용한 추상화를 만들 수 있습니다. 예를 들어, 다음은 [이 문서](https://graphql.org/learn/pagination/#pagination-and-edges)를 기반으로 한 커서 기반 페이지 매김 구현 샘플입니다.

```typescript
import { Field, ObjectType, Int } from '@nestjs/graphql';
import { Type } from '@nestjs/common';

export function Paginated<T>(classRef: Type<T>): any {
  @ObjectType(`${classRef.name}Edge`)
  abstract class EdgeType {
    @Field((type) => String)
    cursor: string;

    @Field((type) => classRef)
    node: T;
  }

  @ObjectType({ isAbstract: true })
  abstract class PaginatedType {
    @Field((type) => [EdgeType], { nullable: true })
    edges: EdgeType[];

    @Field((type) => [classRef], { nullable: true })
    nodes: T[];

    @Field((type) => Int)
    totalCount: number;

    @Field()
    hasNextPage: boolean;
  }
  return PaginatedType;
}
```

With the above base class defined, we can now easily create specialized types that inherit this behavior. For example:
위의 기본 클래스가 정의되었으므로 이제 이 동작을 상속하는 특수 유형을 쉽게 만들 수 있습니다. 예를 들면:

```typescript
@ObjectType()
class PaginatedAuthor extends Paginated(Author) {}
```

#### Schema first

[이전](/graphql/quick-start) 장에서 언급했듯이 스키마 우선 접근 방식에서는 SDL([더 읽어보기](https://graphql.org/learn/schema/#type-language))에서 스키마 유형을 수동으로 정의하는 것으로 시작합니다. 다음 SDL 유형 정의를 고려하십시오.

> info **힌트** 이 장의 편의를 위해 모든 SDL을 한 위치에 모았습니다 (예: 아래와 같이 `.graphql` 파일 하나). 실제로는 모듈 식으로 코드를 구성하는 것이 적절할 수 있습니다. 예를 들어, 관련 서비스, 리졸버 코드 및 Nest 모듈 정의 클래스와 함께 각 도메인 엔터티를 나타내는 유형 정의가 있는 개별 SDL 파일을 해당 엔터티의 전용 디렉터리에 만드는 것이 유용할 수 있습니다. Nest는 런타임에 모든 개별 스키마 유형 정의를 집계합니다.

```graphql
type Author {
  id: Int!
  firstName: String
  lastName: String
  posts: [Post]
}

type Post {
  id: Int!
  title: String!
  votes: Int
}

type Query {
  author(id: Int!): Author
}
```

#### Schema first resolver

위의 스키마는 단일 쿼리를 노출합니다. - `author(id: Int!): Author`.

> info **힌트** [여기]((https://graphql.org/learn/queries/))에서 GraphQL 쿼리에 대해 자세히 알아보세요.

이제 author 쿼리를 해결하는 `AuthorsResolver` 클래스를 만들어 보겠습니다.

```typescript
@@filename(authors/authors.resolver)
@Resolver('Author')
export class AuthorsResolver {
  constructor(
    private authorsService: AuthorsService,
    private postsService: PostsService,
  ) {}

  @Query()
  async author(@Args('id') id: number) {
    return this.authorsService.findOneById(id);
  }

  @ResolveField()
  async posts(@Parent() author) {
    const { id } = author;
    return this.postsService.findAll({ authorId: id });
  }
}
```

> info **힌트** 모든 데코레이터 (예: `@Resolver`, `@ResolveField`, `@Args` 등)는 `@nestjs/graphql` 패키지에서 내보내집니다.

> warning **참고** `AuthorsService` 및 `PostsService` 클래스 내부의 논리는 필요에 따라 간단하거나 정교할 수 있습니다. 이 예제의 요점은 리졸버를 구성하는 방법과 리졸버가 다른 프로바이더와 상호 작용할 수 있는 방법을 보여주는 것입니다.

`@Resolver()` 데코레이터가 필요합니다. 클래스 이름과 함께 선택적 문자열 인수를 받습니다. 이 클래스 이름은 데코레이팅된 메소드가 상위 유형 (현재 예제의 `Author`유형)과 연관되어 있음을 Nest에 알리기 위해 클래스에 `@ResolveField()` 데코레이터가 포함될 때마다 필요합니다. 또는 클래스 상단에 `@Resolver()`를 설정하는 대신 각 메서드에 대해 다음을 수행할 수 있습니다.

```typescript
@Resolver('Author')
@ResolveField()
async posts(@Parent() author) {
  const { id } = author;
  return this.postsService.findAll({ authorId: id });
}
```

이 경우 (메소드 수준의 `@Resolver()` 데코레이터), 클래스 내에 여러 개의 `@ResolveField()` 데코레이터가 있는 경우 모두에 `@Resolver()`를 추가해야 합니다. 이는 추가 오버헤드를 생성하므로 모범 사례로 간주되지 않습니다.

> info **힌트** `@Resolver()`에 전달된 모든 클래스 이름 인수는 쿼리 (`@Query()` 데코레이터) 또는 뮤테이션 (`@Mutation()` 데코레이터)에 **영향을 주지 않습니다**.

> warning **경고** **코드 우선** 접근 방식에서는 메서드 수준에서 `@Resolver` 데코레이터를 사용할 수 없습니다.

위의 예에서 `@Query()` 및 `@ResolveField()` 데코레이터는 메서드 이름을 기반으로 GraphQL 스키마 유형과 연결됩니다. 예를 들어, 위의 예에서 다음 구성을 고려하십시오.

```typescript
@Query()
async author(@Args('id') id: number) {
  return this.authorsService.findOneById(id);
}
```

그러면 스키마의 작성자 쿼리에 대해 다음 항목이 생성됩니다 (쿼리 유형은 메서드 이름과 동일한 이름을 사용함).

```graphql
type Query {
  author(id: Int!): Author
}
```

일반적으로 우리는 리졸버 메소드에 `getAuthor()` 또는 `getPosts()`와 같은 이름을 사용하여 이들을 분리하는 것을 선호합니다. 아래와 같이 매핑 이름을 데코레이터에 인수로 전달하여 쉽게 수행할 수 있습니다.

```typescript
@@filename(authors/authors.resolver)
@Resolver('Author')
export class AuthorsResolver {
  constructor(
    private authorsService: AuthorsService,
    private postsService: PostsService,
  ) {}

  @Query('author')
  async getAuthor(@Args('id') id: number) {
    return this.authorsService.findOneById(id);
  }

  @ResolveField('posts')
  async getPosts(@Parent() author) {
    const { id } = author;
    return this.postsService.findAll({ authorId: id });
  }
}
```

> info **힌트** Nest CLI는 **모든 상용구 코드**를 자동으로 생성하는 생성기 (스키메틱)를 제공하여 이 모든 작업을 피하고 개발자 환경을 훨씬 더 간단하게 만들 수 있습니다. [여기](/recipes/crud-generator)에서 이 기능에 대해 자세히 알아보세요.

#### Generating types

스키마 우선 접근 방식을 사용하고 유형 생성 기능([이전](/graphql/quick-start) 장에 표시된대로 `outputAs: 'class'` 사용)을 활성화했다고 가정하면 애플리케이션을 실행하면 다음 파일(`GraphQLModule.forRoot()` 메서드에서 지정한 위치)이 생성됩니다. 예를 들어 `src/graphql.ts`에서:

```typescript
@@filename(graphql)
export class Author {
  id: number;
  firstName?: string;
  lastName?: string;
  posts?: Post[];
}

export class Post {
  id: number;
  title: string;
  votes?: number;
}

export abstract class IQuery {
  abstract author(id: number): Author | Promise<Author>;
}
```

클래스를 생성하면 (인터페이스 생성의 기본 기술 대신) 선언적 유효성 검사 **데코레이터**를 스키마 우선 접근 방식과 함께 사용할 수 있습니다. 이는 매우 유용한 기술입니다 ([자세히](/techniques/validation) 참조). 예를 들어, 아래와 같이 생성 된 `CreatePostInput` 클래스에 `class-validator` 데코레이터를 추가하여 `title` 필드에 최소 및 최대 문자열 길이를 적용할 수 있습니다.

```typescript
import { MinLength, MaxLength } from 'class-validator';

export class CreatePostInput {
  @MinLength(3)
  @MaxLength(50)
  title: string;
}
```

> warning **알림** 입력 (및 매개 변수)의 자동 유효성 검사를 활성화하려면 `ValidationPipe`를 사용하십시오. 유효성 검사에 대한 자세한 내용은 [여기](/techniques/validation) 및 파이프에 대한 자세한 내용은 [여기](/pipes)를 읽어보세요.

그러나 자동 생성된 파일에 직접 데코레이터를 추가하면 파일이 생성될 때마다 **덮어 쓰기**됩니다. 대신 별도의 파일을 만들고 생성된 클래스를 확장하기만 하면 됩니다.

```typescript
import { MinLength, MaxLength } from 'class-validator';
import { Post } from '../../graphql.ts';

export class CreatePostInput extends Post {
  @MinLength(3)
  @MaxLength(50)
  title: string;
}
```

#### GraphQL argument decorators

전용 데코레이터를 사용하여 표준 GraphQL 리졸버 인수에 액세스할 수 있습니다. 아래는 Nest 데코레이터와 이들이 나타내는 일반 Apollo 매개 변수를 비교한 것입니다.

<table>
  <tbody>
    <tr>
      <td><code>@Root()</code> and <code>@Parent()</code></td>
      <td><code>root</code>/<code>parent</code></td>
    </tr>
    <tr>
      <td><code>@Context(param?: string)</code></td>
      <td><code>context</code> / <code>context[param]</code></td>
    </tr>
    <tr>
      <td><code>@Info(param?: string)</code></td>
      <td><code>info</code> / <code>info[param]</code></td>
    </tr>
    <tr>
      <td><code>@Args(param?: string)</code></td>
      <td><code>args</code> / <code>args[param]</code></td>
    </tr>
  </tbody>
</table>

이러한 인수의 의미는 다음과 같습니다:

- `root`: 상위 필드의 리졸버에서 반환된 결과를 포함하는 객체 또는 최상위 `Query` 필드의 경우 서버 구성에서 전달된 `rootValue`입니다.
- `context`: 특정 쿼리에서 모든 리졸버가 공유하는 객체. 일반적으로 요청별 상태를 포함하는 데 사용됩니다.
- `info`: 쿼리의 실행 상태에 대한 정보를 포함하는 객체입니다.
- `args`: 쿼리의 필드에 전달된 인수가 있는 객체입니다.

<app-banner-shop></app-banner-shop>

#### Module

위의 단계를 완료하면 `GraphQLModule`에서 리졸버 맵을 생성하는 데 필요한 모든 정보를 선언적으로 지정했습니다. `GraphQLModule`은 리플렉션을 사용하여 데코레이터를 통해 제공되는 메타 데이터를 검사하고 클래스를 올바른 리졸버 맵으로 자동 변환합니다.

주의해야할 유일한 다른 일은 해결 프로그램 클래스 (`AuthorsResolver`)를 **제공** (즉, 일부 모듈에서 `provider`로 나열)하는 것입니다. 모듈 (`AuthorsModule`)을 어딘가에 가져 와서 Nest에서 활용할 수 있습니다.

예를 들어, 이 컨텍스트에서 필요한 다른 서비스를 제공할 수도 있는 `AuthorsModule`에서 이를 수행할 수 있습니다. `AuthorsModule`을 어딘가 (예: 루트 모듈 또는 루트 모듈에서 가져온 다른 모듈)로 가져와야 합니다.

```typescript
@@filename(authors/authors.module)
@Module({
  imports: [PostsModule],
  providers: [AuthorsService, AuthorsResolver],
})
export class AuthorsModule {}
```

> info **힌트** 소위 **도메인 모델** (REST API에서 진입점을 구성하는 방식과 유사함) 별로 코드를 구성하는 것이 유용합니다. 이 접근 방식에서는 모델 (`ObjectType` 클래스), 리졸버 및 서비스를 도메인 모델을 나타내는 Nest 모듈내에 함께 유지합니다. 이러한 모든 구성 요소를 모듈당 단일 폴더에 보관하십시오. 이 작업을 수행하고 [Nest CLI](/cli/overview)를 사용하여 각 요소를 생성하면 Nest는 이러한 모든 요소(적절한 폴더에 파일 찾기, `provider` 및 `imports` 배열에 항목 생성 등)를 자동으로 연결합니다.
