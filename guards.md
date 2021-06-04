### Guards

가드는 `@Injectable()` 데코레이터로 주석이 달린 클래스입니다. 가드는 `CanActivate` 인터페이스를 구현해야 합니다.

<figure><img src="/assets/Guards_1.png" /></figure>

가드는 **단일 책임**(single responsibility)이 있습니다. 런타임에 존재하는 특정조건(예: 권한(premissions), 역할(roles), ACL 등)에 따라 지정된 요청을 라우터 핸들러에 의해 처리할지 여부를 결정합니다. 이를 종종 **승인**(authorization)이라고 합니다. 승인(및 일반적으로 공동 작업하는 사촌 **인증**(authentication))은 일반적으로 기존 Express 애플리케이션의 [middleware](/middleware)에 의해 처리되었습니다. 미들웨어는 인증을 위한 좋은 선택입니다. 토큰 유효성 검사와 속성을 `request` 객체에 연결하는 것은 특정 라우트 컨텍스트(및 해당 메타데이터)와 강하게 연결되어 있지않기 때문입니다.

그러나 미들웨어는 본질적으로 멍청합니다. `next()` 함수를 호출한 후 어떤 핸들러가 실행될지 알 수 없습니다. 반면 **가드**는 `ExecutionContext` 인스턴스에 액세스할 수 있으므로 다음에 실행될 작업을 정확히 알고 있습니다. 예외필터, 파이프 및 인터셉터와 매우 유사하게 요청/응답주기의 정확한 지점에서 처리 로직을 삽입하고 선언적으로 수행할 수 있도록 설계되었습니다. 이는 코드를 건조하고 선언적으로 유지하는데 도움이됩니다.

> info **힌트** 가드는 각 미들웨어**이후**에 실행되지만 인터셉터나 파이프는 **앞에** 실행됩니다.

#### Authorization guard

언급했듯이 **승인**은 호출자(일반적으로 특정 인증된 사용자)에게 충분한 권한이 있는 경우에만 특정 라우트를 사용할 수 있어야하므로 가드의 훌륭한 사용사례입니다. 이제 우리가 빌드할 `AuthGuard`는 인증된 사용자를 가정합니다(따라서 토큰이 요청 헤더에 첨부됨). 토큰을 추출하고 유효성을 검사하고 추출된 정보를 사용하여 요청을 진행할 수 있는지 여부를 결정합니다.

```typescript
@@filename(auth.guard)
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    const request = context.switchToHttp().getRequest();
    return validateRequest(request);
  }
}
@@switch
import { Injectable } from '@nestjs/common';

@Injectable()
export class AuthGuard {
  async canActivate(context) {
    const request = context.switchToHttp().getRequest();
    return validateRequest(request);
  }
}
```

> info **힌트** 애플리케이션에서 인증 메커니즘을 구현하는 방법에 대한 실제 예제를 찾고 있다면 [이 장](/security/authentication)을 방문하십시오. 마찬가지로 더 복잡한 인증 예제를 보려면 [이 페이지](/security/authorization)를 확인하십시오.

`validateRequest()` 함수 내부의 로직은 필요에 따라 간단하거나 정교할 수 있습니다. 이 예의 요점은 가드가 요청/응답주기에 어떻게 부합하는지 보여주는 것입니다.

모든 가드는 `canActivate()` 함수를 구현해야 합니다. 이 함수는 현재 요청이 허용되는지 여부를 나타내는 값(부울)을 반환해야 합니다. 응답을 동기식 또는 비동기식(`Promise` 또는 `Observable`을 통해)으로 반환할 수 있습니다. Nest는 반환값을 사용하여 다음 작업을 제어합니다.

- `true`를 반환하면 요청이 처리됩니다.
- `false`를 반환하면 Nest는 요청을 거부합니다.

<app-banner-enterprise></app-banner-enterprise>

#### Execution context

`canActivate()` 함수는 `ExecutionContext` 인스턴스라는 단일인수를받습니다. `ExecutionContext`는 `ArgumentsHost`에서 상속됩니다. 이전에 예외필터 장에서 `ArgumentsHost`를 보았습니다. 위의 샘플에서는 이전에 사용했던 `ArgumentsHost`에 정의된 동일한 헬퍼 메서드를 사용하여 `Request` 객체에 대한 참조를 얻습니다. 이 주제에 대한 자세한 내용은 [예외필터](https:///exception-filters#arguments-host) 장의 **인수 호스트**(Arguments host) 섹션을 다시 참조할 수 있습니다.

`ArgumentsHost`를 확장함으로써 `ExecutionContext`는 현재 실행 프로세스에 대한 추가 세부정보를 제공하는 몇가지 새로운 헬퍼 메서드도 추가합니다. 이러한 세부정보는 광범위한 컨트롤러, 메서드 및 실행 컨텍스트에서 작동할 수 있는 보다 일반적인 가드를 구축하는 데 도움이될 수 있습니다. `ExecutionContext`는 [여기](/fundamentals/execution-context)에 대해 자세히 알아보세요.

#### Role-based authentication

특정 역할을 가진 사용자에게만 액세스를 허용하는 보다 기능적인 가드를 구축해 보겠습니다. 기본 가드 템플릿으로 시작하여 다음 섹션에서 빌드할 것입니다. 지금은 모든 요청을 진행할 수 있습니다.

```typescript
@@filename(roles.guard)
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    return true;
  }
}
@@switch
import { Injectable } from '@nestjs/common';

@Injectable()
export class RolesGuard {
  canActivate(context) {
    return true;
  }
}
```

#### Binding guards

파이프 및 예외필터와 마찬가지로 가드는 **컨트롤러 범위**(controller-scoped), 메서드범위 또는 전역범위일 수 있습니다. 아래에서는 `@UseGuards()` 데코레이터를 사용하여 컨트롤러 범위 가드를 설정했습니다. 이 데코레이터는 단일인수 또는 쉼표로 구분된 인수 목록을 사용할 수 있습니다. 이렇게하면 하나의 선언으로 적절한 가드 세트를 쉽게 적용할 수 있습니다.

```typescript
@@filename()
@Controller('cats')
@UseGuards(RolesGuard)
export class CatsController {}
```

> info **힌트** `@UseGuards()` 데코레이터는 `@nestjs/common` 패키지에서 가져옵니다.

위에서 우리는 인스턴스 대신 `RolesGuard` 타입을 전달하여 인스턴스화를 프레임워크에 맡기고 종속성 주입을 활성화했습니다. 파이프 및 예외필터와 마찬가지로 내부(in-place) 인스턴스를 전달할 수도 있습니다.

```typescript
@@filename()
@Controller('cats')
@UseGuards(new RolesGuard())
export class CatsController {}
```

위의 구성은 이 컨트롤러가 선언한 모든 핸들러에 가드를 연결합니다. 가드가 단일메서드에만 적용되도록 하려면 **메서드 수준**에서 `@UseGuards()` 데코레이터를 적용합니다.

전역가드를 설정하려면 Nest 애플리케이션 인스턴스의 `useGlobalGuards()` 메서드를 사용하세요.

```typescript
@@filename()
const app = await NestFactory.create(AppModule);
app.useGlobalGuards(new RolesGuard());
```

> warning **알림** 하이브리드 앱의 경우 `useGlobalGuards()` 메서드는 기본적으로 게이트웨이 및 마이크로서비스에 대한 보호를 설정하지 않습니다(이 동작을 변경하는 방법에 대한 정보는 [하이브리드 애플리케이션](/faq/hybrid-application)을 참조하십시오). "표준" (비 하이브리드) 마이크로서비스 앱의 경우 `useGlobalGuards()`는 가드를 전역적으로 마운트합니다.

글로벌 가드는 모든 컨트롤러 및 모든 라우트 핸들러에 대해 전체 애플리케이션에서 사용됩니다. 의존성 주입의 경우, 모듈 외부에서 등록된 전역가드(위의 예에서와 같이 `useGlobalGuards()` 사용)는 모듈의 컨텍스트 외부에서 수행되기 때문에 종속성을 주입할 수 없습니다. 이 문제를 해결하기 위해 다음 구성을 사용하여 모든 모듈에서 직접 가드를 설정할 수 있습니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { APP_GUARD } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: RolesGuard,
    },
  ],
})
export class AppModule {}
```

> info **힌트** 이 접근방식을 사용하여 가드에 대한 종속성 주입을 수행할 때 이 구성이 사용되는 모듈에 관계없이 가드는 실제로 전역적입니다. 어디에서 해야 합니까? 가드(위 예에서는 `RolesGuard`)가 정의된 모듈을 선택합니다. 또한 `useClass`가 커스텀 프로바이더 등록을 처리하는 유일한 방법은 아닙니다. [여기](/fundamentals/custom-providers)에서 자세히 알아보세요.

#### Setting roles per handler

우리의 `RolesGuard`는 작동하지만 아직 똑똑하지는 않습니다. 우리는 아직 가장 중요한 가드 기능인 [실행 컨텍스트(execution context)](/fundamentals/execution-context)를 활용하고 있지 않습니다. 역할이나 각 핸들러에 대해 허용되는 역할에 대해서는 아직 알지 못합니다. 예를 들어 `CatsController`는 라우트마다 다른 권한체계를 가질 수 있습니다. 일부는 관리자만 사용할 수 있고 다른 일부는 모든 사용자가 사용할 수 있습니다. 유연하고 재사용 가능한 방식으로 역할을 라우트에 어떻게 일치시킬 수 있습니까?

여기에서 **맞춤 메타데이터**가 작동합니다 ([여기](/fundamentals/execution-context#reflection-and-metadata)에서 자세히 알아보기). Nest는 `@SetMetadata()` 데코레이터를 통해 라우트 핸들러에 커스텀 **메타데이터**를 첨부하는 기능을 제공합니다. 이 메타데이터는 스마트 가드가 결정을 내리는데 필요한 누락된 `role` 데이터를 제공합니다. `@SetMetadata()` 사용을 살펴 보겠습니다.

```typescript
@@filename(cats.controller)
@Post()
@SetMetadata('roles', ['admin'])
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
@@switch
@Post()
@SetMetadata('roles', ['admin'])
@Bind(Body())
async create(createCatDto) {
  this.catsService.create(createCatDto);
}
```

> info **힌트** `@SetMetadata() `데코레이터는 `@nestjs/common` 패키지에서 가져옵니다.

위의 구성을 통해 `create()` 메소드에 `roles` 메타데이터(`roles`는 키이고 `['admin']`은 특정 값)를 첨부했습니다. 이것이 작동하지만 라우트에서 직접 `@SetMetadata()`를 사용하는 것은 좋지 않습니다. 대신 아래와 같이 자신만의 데코레이터를 만드세요.

```typescript
@@filename(roles.decorator)
import { SetMetadata } from '@nestjs/common';

export const Roles = (...roles: string[]) => SetMetadata('roles', roles);
@@switch
import { SetMetadata } from '@nestjs/common';

export const Roles = (...roles) => SetMetadata('roles', roles);
```

이 접근방식은 훨씬 깔끔하고 읽기 쉬우며 강력하게 입력됩니다. 이제 사용자 정의 `@Roles()` 데코레이터가 있으므로 이를 사용하여 `create()` 메소드를 데코레이션할 수 있습니다.

```typescript
@@filename(cats.controller)
@Post()
@Roles('admin')
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
@@switch
@Post()
@Roles('admin')
@Bind(Body())
async create(createCatDto) {
  this.catsService.create(createCatDto);
}
```

#### Putting it all together

이제 돌아가서 이것을 `RolesGuard`와 함께 묶어보겠습니다. 현재는 모든 경우에 `true`를 반환하므로 모든 요청이 진행될 수 있습니다. **현재 사용자에게 할당된 역할**을 처리중인 현재 라우트에 필요한 실제 역할과 비교하여 반환값을 조건부로 만들고 싶습니다. 라우트의 역할(커스텀 메타데이터)에 액세스하기 위해 프레임워크에서 즉시 제공되고 `@nestjs/core` 패키지에서 노출되는 `Reflector` 헬퍼 클래스를 사용합니다.

```typescript
@@filename(roles.guard)
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    if (!roles) {
      return true;
    }
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    return matchRoles(roles, user.roles);
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
    const roles = this.reflector.get('roles', context.getHandler());
    if (!roles) {
      return true;
    }
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    return matchRoles(roles, user.roles);
  }
}
```

> info **힌트** node.js 세계에서는 승인된 사용자를 `request` 객체에 연결하는 것이 일반적입니다. 따라서 위의 샘플코드에서 `request.user`에 사용자 인스턴스와 허용된 역할이 포함되어 있다고 가정합니다. 앱에서 사용자 정의 **인증 가드**(authentication guard)(또는 미들웨어)에서 해당 연결을 만들 수 있습니다. 이 항목에 대한 자세한 내용은 [이 장](/security/authentication)을 확인하십시오.

> warning **경고** `matchRoles()` 함수 내부의 로직은 필요에 따라 간단하거나 정교할 수 있습니다. 이 예의 요점은 가드가 요청/응답 주기에 어떻게 부합하는지 보여주는 것입니다.

상황에 맞는 방식으로 `Reflector`를 사용하는 방법에 대한 자세한 내용은 **실행 컨텍스트**장의 [리플렉션 및 메타 데이터](/fundamentals/execution-context#reflection-and-metadata) 섹션을 참조하세요.

권한이 부족한 사용자가 엔드포인트를 요청하면 Nest는 자동으로 다음 응답을 반환합니다.

```typescript
{
  "statusCode": 403,
  "message": "Forbidden resource",
  "error": "Forbidden"
}
```

배후에서 가드가 `false`를 반환하면 프레임워크에서 `ForbiddenException`이 발생합니다. 다른 오류 응답을 반환하려면 고유한 예외를 발생시켜야합니다. 예를 들면:

```typescript
throw new UnauthorizedException();
```

가드가 던진 모든 예외는 [예외계층](/exception-filters)(전역 예외필터 및 현재 컨텍스트에 적용되는 모든 예외필터)에 의해 처리됩니다.

> info **힌트** 인증 구현방법에 대한 실제 예제를 찾고 있다면 [이 장](/security/authorization)을 확인하십시오.
