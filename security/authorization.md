### Authorization

**권한 부여(authorization)**은 사용자가 수행할 수 있는 작업을 결정하는 프로세스를 나타냅니다. 예를 들어, 관리 사용자는 게시물을 작성, 편집 및 삭제할 수 있습니다. 관리자가 아닌 사용자는 게시물을 읽을 수만 있습니다.

권한 부여는 직각이며 인증(authentication)과 독립적입니다. 그러나 권한 부여에는 인증 메커니즘이 필요합니다.

권한 부여를 처리하기 위한 다양한 접근 방식과 전략이 있습니다. 모든 프로젝트에 적용되는 접근 방식은 특정 애플리케이션 요구사항에 따라 다릅니다. 이 장에서는 다양한 요구사항에 적용할 수 있는 권한 부여에 대한 몇가지 접근 방식을 제공합니다.

#### Basic RBAC implementation

역할 기반 액세스 제어 (**RBAC(Role-based access control**)는 역할 및 권한에 대해 정의된 정책 중립적 액세스 제어 메커니즘입니다. 이 섹션에서는 Nest [가드](/guards)를 사용하여 매우 기본적인 RBAC 메커니즘을 구현하는 방법을 보여줍니다.

먼저 시스템에서 역할을 나타내는 `Role` 열거(enum)형을 만들어 보겠습니다.

```typescript
@@filename(role.enum)
export enum Role {
  User = 'user',
  Admin = 'admin',
}
```

> info **힌트** 보다 정교한 시스템에서는 데이터베이스내에 역할을 저장하거나 외부 인증 프로바이더로부터 역할을 가져올 수 있습니다.

이것으로 `@Roles()` 데코레이터를 만들 수 있습니다. 이 데코레이터를 사용하면 특정 리소스에 액세스하는 데 필요한 역할을 지정할 수 있습니다.

```typescript
@@filename(roles.decorator)
import { SetMetadata } from '@nestjs/common';
import { Role } from '../enums/role.enum';

export const ROLES_KEY = 'roles';
export const Roles = (...roles: Role[]) => SetMetadata(ROLES_KEY, roles);
@@switch
import { SetMetadata } from '@nestjs/common';

export const ROLES_KEY = 'roles';
export const Roles = (...roles) => SetMetadata(ROLES_KEY, roles);
```

이제 사용자 정의 `@Roles()` 데코레이터가 있으므로 이를 사용하여 모든 라우트 핸들러를 데코레이션할 수 있습니다.

```typescript
@@filename(cats.controller)
@Post()
@Roles(Role.Admin)
create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
@@switch
@Post()
@Roles(Role.Admin)
@Bind(Body())
create(createCatDto) {
  this.catsService.create(createCatDto);
}
```

마지막으로 현재 사용자에게 할당된 역할을 처리중인 현재 경로에 필요한 실제 역할과 비교하는 `RolesGuard` 클래스를 만듭니다. 경로의 역할 (사용자 지정 메타 데이터)에 액세스하기 위해 프레임 워크에서 즉시 제공되고 `@nestjs/core` 패키지에서 노출되는 `Reflector` 도우미 클래스를 사용합니다.

```typescript
@@filename(roles.guard)
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<Role[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);
    if (!requiredRoles) {
      return true;
    }
    const { user } = context.switchToHttp().getRequest();
    return requiredRoles.some((role) => user.roles?.includes(role));
  }
}
@@switch
import { Injectable, Dependencies } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

@Injectable()
@Dependencies(Reflector)
export class RolesGuard {
  constructor(reflector) {
    this.reflector = reflector;
  }

  canActivate(context) {
    const requiredRoles = this.reflector.getAllAndOverride(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);
    if (!requiredRoles) {
      return true;
    }
    const { user } = context.switchToHttp().getRequest();
    return requiredRoles.some((role) => user.roles.includes(role));
  }
}
```

> info **힌트** `Reflector`를 상황에 맞는 방식으로 활용하는 방법에 대한 자세한 내용은 실행 컨텍스트 장의 [Reflection and metadata](/fundamentals/execution-context#reflection-and-metadata) 섹션을 참조하세요.

> warning **알림** 이 예는 경로 핸들러 수준에서 역할의 존재 여부만 확인하므로 "**basic**"으로 이름이 지정됩니다. 실제 애플리케이션에서는 여러 작업을 포함하는 엔드 포인트/핸들러가 있을 수 있으며 각 작업에는 특정 권한 집합이 필요합니다. 이 경우 비즈니스 로직내 어딘가에서 역할을 확인하는 메커니즘을 제공해야 하므로 권한을 특정 작업과 연결하는 중앙 집중식 장소가 없기 때문에 유지 관리가 다소 어려워집니다.

이 예에서는 `request.user`에 사용자 인스턴스와 허용된 역할 (`roles`속성 아래)이 포함되어 있다고 가정했습니다. 앱에서 사용자 정의 **인증 가드**에서 해당 연결을 만들 수 있습니다. 자세한 내용은 [인증](/security/authentication) 장을 참조하십시오.

이 예제가 작동하는지 확인하려면 `User` 클래스가 다음과 같아야 합니다.

```typescript
class User {
  // ...other properties
  roles: Role[];
}
```

마지막으로 `RolesGuard`를 컨트롤러 수준에서 등록하거나 전역적으로 등록해야 합니다.

```typescript
providers: [
  {
    provide: APP_GUARD,
    useClass: RolesGuard,
  },
],
```

When a user with insufficient privileges requests an endpoint, Nest automatically returns the following response:
권한이 부족한 사용자가 엔드 포인트를 요청하면 Nest는 자동으로 다음 응답을 반환합니다.

```typescript
{
  "statusCode": 403,
  "message": "Forbidden resource",
  "error": "Forbidden"
}
```

> info **Hint** If you want to return a different error response, you should throw your own specific exception instead of returning a boolean value.
> info **힌트** 다른 오류 응답을 반환하려면 부울 값을 반환하는 대신 고유한 예외를 발생시켜야 합니다.

#### Claims-based authorization

ID가 생성되면 신뢰할 수 있는 당사자가 발행한 하나 이상의 클레임이 할당될 수 있습니다. 클레임은 주체가 아닌 주체가 할 수 있는 일을 나타내는 이름-값 쌍입니다.

Nest에서 클레임 기반 승인을 구현하려면 위의 [RBAC](/security/authorization#basic-rbac-implementation) 섹션에서 설명한 것과 동일한 단계를 수행 할 수 있습니다. 한 가지 중요한 차이점이 있습니다. 특정 역할을 확인하는 대신 **권한**을 비교해야합니다. 모든 사용자에게는 일련의 권한이 할당됩니다. 마찬가지로 각 리소스/엔드 포인트는 액세스하는 데 필요한 권한 (예: 전용 `@RequirePermissions()` 데코레이터를 통해)을 정의합니다.

```typescript
@@filename(cats.controller)
@Post()
@RequirePermissions(Permission.CREATE_CAT)
create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
@@switch
@Post()
@RequirePermissions(Permission.CREATE_CAT)
@Bind(Body())
create(createCatDto) {
  this.catsService.create(createCatDto);
}
```

> info **힌트** 위의 예에서 `Permission` (RBAC 섹션에 표시된 `Role`과 유사)은 시스템에서 사용 가능한 모든 권한을 포함하는 TypeScript 열거(enum) 형입니다.

#### Integrating CASL

[CASL](https://casl.js.org/)은 주어진 클라이언트가 액세스할 수 있는 리소스를 제한하는 동형 승인 라이브러리입니다. 점진적으로 채택할 수 있도록 설계되었으며 단순 클레임 기반과 완전한 기능을 갖춘 주제(subject) 및 속성(attribute) 기반 권한간에 쉽게 확장할 수 있습니다.

시작하려면 먼저 `@casl/ability` 패키지를 설치하십시오.

```bash
$ npm i @casl/ability
```

> info **힌트** 이 예에서는 CASL을 선택했지만 기본 설정과 프로젝트 요구 사항에 따라 `accesscontrol` 또는 `acl`과 같은 다른 라이브러리를 사용할 수 있습니다.

설치가 완료되면 CASL의 메커니즘을 설명하기 위해 두 개의 엔티티 클래스 인 `User`와 `Article`을 정의합니다.

```typescript
class User {
  id: number;
  isAdmin: boolean;
}
```

`User` 클래스는 고유한 사용자 식별자인 `id`와 사용자에게 관리자 권한이 있는지 여부를 나타내는 `isAdmin`의 두 가지 속성으로 구성됩니다.

```typescript
class Article {
  id: number;
  isPublished: boolean;
  authorId: string;
}
```

`Article` 클래스에는 각각 `id`, `isPublished` 및 `authorId`의 세가지 속성이 있습니다. `id`는 고유한 기사(Article) 식별자이고 `isPublished`는 기사(article)가 이미 게시되었는지 여부를 나타내고 `authorId`는 기사를 작성한 사용자의 ID입니다.

이제이 예제에 대한 요구 사항을 검토하고 구체화 해 보겠습니다.

- 관리자는 모든 엔티티를 관리 (생성/읽기/업데이트/삭제)할 수 있습니다.
- 사용자는 모든 것에 대한 읽기 전용 액세스 권한이 있습니다.
- 기사 업데이트 가능 (`article.authorId === userId`)
- 이미 게시된 기사는 삭제할 수 없습니다 (`article.isPublished === true`).

이를 염두에 두고 사용자가 엔티티로 수행할 수 있는 모든 가능한 작업을 나타내는 `Action` 열거(enum)형을 생성하여 시작할 수 있습니다.

```typescript
export enum Action {
  Manage = 'manage',
  Create = 'create',
  Read = 'read',
  Update = 'update',
  Delete = 'delete',
}
```

> warning **알림** `manage`는 CASL에서 "모든" 작업을 나타내는 특수 키워드입니다.

CASL 라이브러리를 캡슐화하기 위해 이제 `CaslModule`과 `CaslAbilityFactory`를 생성해 보겠습니다.

```bash
$ nest g module casl
$ nest g class casl/casl-ability.factory
```

이것으로 우리는 `CaslAbilityFactory`에 `createForUser()` 메소드를 정의할 수 있습니다. 이 메소드는 주어진 사용자에 대한 `Ability` 객체를 생성합니다 :

```typescript
type Subjects = InferSubjects<typeof Article | typeof User> | 'all';

export type AppAbility = Ability<[Action, Subjects]>;

@Injectable()
export class CaslAbilityFactory {
  createForUser(user: User) {
    const { can, cannot, build } = new AbilityBuilder<
      Ability<[Action, Subjects]>
    >(Ability as AbilityClass<AppAbility>);

    if (user.isAdmin) {
      can(Action.Manage, 'all'); // read-write access to everything
    } else {
      can(Action.Read, 'all'); // read-only access to everything
    }

    can(Action.Update, Article, { authorId: user.id });
    cannot(Action.Delete, Article, { isPublished: true });

    return build({
      // Read https://casl.js.org/v5/en/guide/subject-type-detection#use-classes-as-subject-types for details
      detectSubjectType: item => item.constructor as ExtractSubjectType<Subjects>
    });
  }
}
```

> warning **알림** `all`은 CASL에서 "모든 주제"를 나타내는 특수 키워드입니다.

> info **힌트** `Ability`, `AbilityBuilder`, `AbilityClass` 및 `ExtractSubjectType` 클래스는 `@casl/ability` 패키지에서 내보내집니다.

> info **힌트** `detectSubjectType` 옵션을 사용하면 CASL이 객체에서 주제 유형을 가져 오는 방법을 이해할 수 있습니다. 자세한 내용은 [CASL 문서](https://casl.js.org/v5/en/guide/subject-type-detection#use-classes-as-subject-types)를 참조하세요.

위의 예에서는 `AbilityBuilder` 클래스를 사용하여 `Ability` 인스턴스를 생성했습니다. 짐작했듯이 `can`과 `cannot`은 동일한 인수를 받아들이지만 다른 의미를 가지고 있습니다. `can`은 지정된 주제에 대한 작업을 수행하고 `cannot`은 금지합니다. 둘 다 최대 4 개의 인수를 허용할 수 있습니다. 이러한 기능에 대해 자세히 알아 보려면 공식 [CASL 문서](https://casl.js.org/v5/en/guide/intro)를 방문하세요.

마지막으로, `CaslModule` 모듈 정의에서 `providers` 및 `exports` 배열에 `CaslAbilityFactory`를 추가해야 합니다.

```typescript
import { Module } from '@nestjs/common';
import { CaslAbilityFactory } from './casl-ability.factory';

@Module({
  providers: [CaslAbilityFactory],
  exports: [CaslAbilityFactory],
})
export class CaslModule {}
```

이 기능을 사용하면 호스트 컨텍스트에서 `CaslModule`을 가져오는 한 표준 생성자 주입을 사용하여 모든 클래스에 `CaslAbilityFactory`를 주입할 수 있습니다.

```typescript
constructor(private caslAbilityFactory: CaslAbilityFactory) {}
```

그런 다음 다음과 같이 클래스에서 사용하십시오.

```typescript
const ability = this.caslAbilityFactory.createForUser(user);
if (ability.can(Action.Read, 'all')) {
  // "user" has read access to everything
}
```

> info **힌트** 공식 [CASL 문서](https://casl.js.org/v5/en/guide/intro)에서 `Ability` 클래스에 대해 자세히 알아보세요.

예를 들어 관리자가 아닌 사용자가 있다고 가정해 보겠습니다. 이 경우 사용자는 기사를 읽을 수 있어야 하지만 새로운 기사를 작성하거나 기존 기사를 제거하는 것은 금지되어야합니다.

```typescript
const user = new User();
user.isAdmin = false;

const ability = this.caslAbilityFactory.createForUser(user);
ability.can(Action.Read, Article); // true
ability.can(Action.Delete, Article); // false
ability.can(Action.Create, Article); // false
```

> info **힌트** `Ability`와 `AbilityBuilder` 클래스는 모두 `can` 및 `cannot` 메소드를 제공하지만 용도가 다르고 인수도 약간 다릅니다.

또한 요구 사항에서 지정한대로 사용자는 기사를 업데이트할 수 있어야 합니다.

```typescript
const user = new User();
user.id = 1;

const article = new Article();
article.authorId = user.id;

const ability = this.caslAbilityFactory.createForUser(user);
ability.can(Action.Update, article); // true

article.authorId = 2;
ability.can(Action.Update, article); // false
```

보시다시피 `Ability` 인스턴스를 사용하면 읽기 쉬운 방식으로 권한을 확인할 수 있습니다. 마찬가지로 `AbilityBuilder`를 사용하면 비슷한 방식으로 권한을 정의하고 다양한 조건을 지정할 수 있습니다. 더 많은 예제를 찾으려면 공식 문서를 방문하십시오.

#### Advanced: Implementing a `PoliciesGuard`

이 섹션에서는 사용자가 메서드 수준에서 구성할 수 있는 특정 **권한 부여 정책**을 충족하는지 확인하는 좀 더 정교한 가드를 구축하는 방법을 보여줄 것입니다 (이를 확장하여 클래스 수준도). 이 예제에서는 설명 목적으로 CASL 패키지를 사용하지만 이 라이브러리를 사용할 필요는 없습니다. 또한 이전 섹션에서 만든 `CaslAbilityFactory` 프로바이더를 사용합니다.

먼저 요구 사항을 구체화합시다. 목표는 경로 핸들러별로 정책 검사를 지정할 수 있는 메커니즘을 제공하는 것입니다. 우리는 객체와 함수를 모두 지원할 것입니다 (더 간단한 검사와 더 기능적인 스타일의 코드를 선호하는 사람들을 위해).

정책 핸들러를 위한 인터페이스를 정의하는 것으로 시작하겠습니다.

```typescript
import { AppAbility } from '../casl/casl-ability.factory';

interface IPolicyHandler {
  handle(ability: AppAbility): boolean;
}

type PolicyHandlerCallback = (ability: AppAbility) => boolean;

export type PolicyHandler = IPolicyHandler | PolicyHandlerCallback;
```

위에서 언급했듯이, 우리는 정책 핸들러를 정의하는 두가지 가능한 방법, 객체(`IPolicyHandler` 인터페이스를 구현하는 클래스의 인스턴스)와 함수(`PolicyHandlerCallback` 타입을 충족)를 제공했습니다.

이것으로 `@CheckPolicies()` 데코레이터를 만들 수 있습니다. 이 데코레이터를 사용하면 특정 리소스에 액세스하기 위해 충족해야하는 정책을 지정할 수 있습니다.

```typescript
export const CHECK_POLICIES_KEY = 'check_policy';
export const CheckPolicies = (...handlers: PolicyHandler[]) =>
  SetMetadata(CHECK_POLICIES_KEY, handlers);
```

이제 경로 핸들러에 바인딩된 모든 정책 핸들러를 추출하고 실행할 `PoliciesGuard`를 만들어 보겠습니다.

```typescript
@Injectable()
export class PoliciesGuard implements CanActivate {
  constructor(
    private reflector: Reflector,
    private caslAbilityFactory: CaslAbilityFactory,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const policyHandlers =
      this.reflector.get<PolicyHandler[]>(
        CHECK_POLICIES_KEY,
        context.getHandler(),
      ) || [];

    const { user } = context.switchToHttp().getRequest();
    const ability = this.caslAbilityFactory.createForUser(user);

    return policyHandlers.every((handler) =>
      this.execPolicyHandler(handler, ability),
    );
  }

  private execPolicyHandler(handler: PolicyHandler, ability: AppAbility) {
    if (typeof handler === 'function') {
      return handler(ability);
    }
    return handler.handle(ability);
  }
}
```

> info **Hint** In this example, we assumed that `request.user` contains the user instance. In your app, you will probably make that association in your custom **authentication guard** - see [authentication](/security/authentication) chapter for more details.
> info **힌트** 이 예에서는 `request.user`에 사용자 인스턴스가 포함되어 있다고 가정했습니다. 앱에서 사용자 정의 **인증 가드**에서 해당 연결을 만들 수 있습니다. 자세한 내용은 [인증](/security/authentication) 장을 참조하십시오.

이 예제를 분해해 보겠습니다. `policyHandlers`는 `@CheckPolicies()` 데코레이터를 통해 메소드에 할당된 핸들러의 배열입니다. 다음으로 `Ability` 객체를 구성하는 `CaslAbilityFactory#create` 메소드를 사용하여 사용자가 특정 작업을 수행할 수 있는 충분한 권한이 있는지 확인할 수 있습니다. 이 객체를 `IPolicyHandler`를 구현하는 클래스의 함수 또는 인스턴스인 정책 핸들러에 전달하여 부울을 반환하는 `handle()` 메서드를 노출합니다. 마지막으로 `Array#every` 메소드를 사용하여 모든 핸들러가 `true` 값을 반환하는지 확인합니다.

마지막으로 이 가드를 테스트하려면 다음과 같이 경로 핸들러에 바인딩하고 인라인 정책 핸들러 (기능적 접근 방식)를 등록합니다.

```typescript
@Get()
@UseGuards(PoliciesGuard)
@CheckPolicies((ability: AppAbility) => ability.can(Action.Read, Article))
findAll() {
  return this.articlesService.findAll();
}
```

또는 `IPolicyHandler` 인터페이스를 구현하는 클래스를 정의할 수 있습니다.

```typescript
export class ReadArticlePolicyHandler implements IPolicyHandler {
  handle(ability: AppAbility) {
    return ability.can(Action.Read, Article);
  }
}
```

다음과 같이 사용하십시오.

```typescript
@Get()
@UseGuards(PoliciesGuard)
@CheckPolicies(new ReadArticlePolicyHandler())
findAll() {
  return this.articlesService.findAll();
}
```

> warning **알림** `new` 키워드를 사용하여 정책 핸들러를 제자리에서 인스턴스화해야 하므로 `CreateArticlePolicyHandler` 클래스는 Dependency Injection을 사용할 수 없습니다. 이 문제는 `ModuleRef#get` 메소드로 해결할 수 있습니다 ([여기](/fundamentals/module-ref)에 대해 자세히 알아보기). 기본적으로 `@CheckPolicies()` 데코레이터를 통해 함수와 인스턴스를 등록하는 대신 `Type<IPolicyHandler>` 전달을 허용해야 합니다. 그런 다음 가드내에서 타입 참조 `moduleRef.get(YOUR_HANDLER_TYPE)`을 사용하여 인스턴스를 검색하거나 `ModuleRef#create` 메소드를 사용하여 동적으로 인스턴스화 할 수도 있습니다.
