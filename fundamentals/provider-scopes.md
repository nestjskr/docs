### Injection scopes

다른 프로그래밍 언어 배경을 가진 사람들의 경우 Nest에서 거의 모든 것이 들어오는 요청에서 공유된다는 사실을 배우는 것은 예상치 못한 일입니다. 데이터베이스에 대한 연결 풀, 전역 상태의 싱글톤 서비스 등이 있습니다. Node.js는 모든 요청이 별도의 스레드에서 처리되는 요청/응답 다중 스레드 상태 비저장 모델을 따르지 않습니다. 따라서 싱글톤 인스턴스를 사용하는 것은 애플리케이션에 완전히 **안전**합니다.

그러나 GraphQL 애플리케이션의 요청별 캐싱, 요청 추적 및 멀티 테넌시와 같이 요청 기반 수명이 원하는 동작일 수 있는 경우가 있습니다. 주입 범위는 원하는 공급자 수명 동작을 얻기위한 메커니즘을 제공합니다.

#### Provider scope

A provider can have any of the following scopes:
프로바이더는 다음 범위중 하나를 가질 수 있습니다.

<table>
  <tr>
    <td><code>DEFAULT</code></td>
    <td>프로바이더의 단일 인스턴스는 전체 응용 프로그램에서 공유됩니다. 인스턴스 수명은 애플리케이션 수명주기와 직접 연결됩니다. 애플리케이션이 부트스트랩되면 모든 싱글톤 프로바이더가 인스턴스화되었습니다. 기본적으로 싱글톤 범위가 사용됩니다.</td>
  </tr>
  <tr>
    <td><code>REQUEST</code></td>
    <td>프로바이더의 새 인스턴스는 들어오는 각 <strong>요청</strong>에 대해 독점적으로 생성됩니다. 요청이 처리를 완료한 후 인스턴스가 가비지 수집(garbage-collected)됩니다.</td>
  </tr>
  <tr>
    <td><code>TRANSIENT</code></td>
    <td>일시적인 프로바이더는 소비자간에 공유되지 않습니다. 임시 프로바이더를 삽입하는 각 소비자는 새로운 전용 인스턴스를 받게됩니다.</td>
  </tr>
</table>

> info **힌트** 싱글톤 범위 사용은 대부분의 사용 사례에서 **권장**됩니다. 소비자와 요청간에 프로바이더를 공유한다는 것은 인스턴스를 캐시할 수 있고 초기화가 애플리케이션 시작중에 한번만 발생함을 의미합니다.

#### Usage

`scope` 속성을 `@Injectable()` 데코레이터 옵션 객체에 전달하여 주입 범위를 지정합니다.

```typescript
import { Injectable, Scope } from '@nestjs/common';

@Injectable({ scope: Scope.REQUEST })
export class CatsService {}
```

마찬가지로 [사용자 지정 프로바이더](/fundamentals/custom-providers)의 경우 프로바이더 등록을 위해 긴 형식으로 `scope` 속성을 설정합니다.

```typescript
{
  provide: 'CACHE_MANAGER',
  useClass: CacheManager,
  scope: Scope.TRANSIENT,
}
```

> info **힌트** `@nestjs/common`에서 `Scope` 열거형(enum) 가져오기

> warning **알림** 게이트웨이는 싱글톤으로 작동해야하므로 요청 범위 프로바이더를 사용해서는 안됩니다. 각 게이트웨이는 실제 소켓을 캡슐화하며 여러번 인스턴스화 할 수 없습니다.

싱글톤 범위는 기본적으로 사용되며 선언할 필요가 없습니다. 프로바이더를 싱글톤 범위로 선언하려면 `scope` 속성에 `Scope.DEFAULT` 값을 사용하세요.

#### Controller scope

컨트롤러는 해당 컨트롤러에서 선언된 모든 요청 메서드 핸들러에 적용되는 범위를 가질 수도 있습니다. 프로바이더 범위와 마찬가지로 컨트롤러의 범위는 수명을 선언합니다. 요청 범위 컨트롤러의 경우 각 인바운드 요청에 대해 새 인스턴스가 생성되고 요청이 처리를 완료하면 가비지 수집됩니다.

`ControllerOptions` 객체의 `scope` 속성으로 컨트롤러 범위를 선언합니다.

```typescript
@Controller({
  path: 'cats',
  scope: Scope.REQUEST,
})
export class CatsController {}
```

#### Scope hierarchy

스코프가 주입 체인을 위로 올립니다. 요청 범위 프로바이더에 의존하는 컨트롤러는 자체적으로 요청 범위가 됩니다.

다음 종속성 그래프를 상상해보십시오: `CatsController <- CatsService <- CatsRepository`. `CatsService`가 요청 범위이고 나머지는 기본 싱글톤인 경우 `CatsController`는 삽입된 서비스에 따라 달라지므로 요청 범위가 됩니다. 종속되지 않는 `CatsRepository`는 싱글톤 범위로 유지됩니다.

<app-banner-courses></app-banner-courses>

#### Request provider

HTTP 서버 기반 애플리케이션 (예: `@nestjs/platform-express` 또는 `@nestjs/platform-fastify` 사용)에서 요청 범위 프로바이더를 사용할 때 원래 요청 객체에 대한 참조에 액세스할 수 있습니다. `REQUEST`개체를 삽입하여 이를 수행할 수 있습니다.

```typescript
import { Injectable, Scope, Inject } from '@nestjs/common';
import { REQUEST } from '@nestjs/core';
import { Request } from 'express';

@Injectable({ scope: Scope.REQUEST })
export class CatsService {
  constructor(@Inject(REQUEST) private request: Request) {}
}
```

기본 플랫폼/프로토콜 차이로 인해 마이크로 서비스 또는 GraphQL 애플리케이션의 경우 인바운드 요청에 약간 다르게 액세스합니다. [GraphQL](/graphql/quick-start) 애플리케이션에서 `REQUEST`대신 `CONTEXT`를 삽입합니다.

```typescript
import { Injectable, Scope, Inject } from '@nestjs/common';
import { CONTEXT } from '@nestjs/graphql';

@Injectable({ scope: Scope.REQUEST })
export class CatsService {
  constructor(@Inject(CONTEXT) private context) {}
}
```

그런 다음 `request`을 속성으로 포함하도록 `context`값(`GraphQLModule`에서)을 구성합니다.

#### Performance

요청 범위 프로바이더를 사용하면 애플리케이션 성능에 영향을 미칩니다. Nest는 가능한 한 많은 메타데이터를 캐시하려고 하지만 각 요청에 대해 클래스의 인스턴스를 만들어야합니다. 따라서 평균 응답 시간과 전체 벤치마킹 결과가 느려집니다. 프로바이더가 요청 범위여야 하는 경우가 아니면 기본 싱글톤 범위를 사용하는 것이 좋습니다.
