### Migration guide

이 문서에서는 Nest 버전 6에서 버전 7로 이전하기 위한 일련의 가이드 라인을 제공합니다.

#### Custom route decorators

[커스텀 데코레이터](/custom-decorators) API는 모든 유형의 애플리케이션에 대해 통합되었습니다. 이제 GraphQL 애플리케이션을 생성하든 REST API를 생성하든 `createParamDecorator()` 함수에 전달된 팩토리는 `ExecutionContext`(자세히 알아보기 [여기](/fundamentals/execution-context))  객체를 두 번째 인수로 사용합니다.

```typescript
@@filename()
// Before
import { createParamDecorator } from '@nestjs/common';

export const User = createParamDecorator((data, req) => {
  return req.user;
});

// After
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const User = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  },
);
@@switch
// Before
import { createParamDecorator } from '@nestjs/common';

export const User = createParamDecorator((data, req) => {
  return req.user;
});

// After
import { createParamDecorator } from '@nestjs/common';

export const User = createParamDecorator((data, ctx) => {
  const request = ctx.switchToHttp().getRequest();
  return request.user;
});
```

#### Microservices

코드 중복을 방지하기 위해 `@nestjs/common` 패키지에서 `MicroserviceOptions` 인터페이스가 제거되었습니다. 따라서 이제 마이크로 서비스를 생성할 때(`createMicroservice()` 또는 `connectMicroservice()` 메서드를 통해) 타입 일반 매개변수를 전달하여 코드 자동 완성을 가져와야합니다.

```typescript
@@filename()
// Before
const app = await NestFactory.createMicroservice(AppModule);

// After
const app = await NestFactory.createMicroservice<MicroserviceOptions>(AppModule);
@@switch
// Before
const app = await NestFactory.createMicroservice(AppModule);

// After
const app = await NestFactory.createMicroservice(AppModule);
```

> info **힌트** `MicroserviceOptions` 인터페이스는 `@nestjs/microservices` 패키지에서 내 보냅니다.

#### GraphQL

NestJS의 버전 6 주요 릴리스에서는 `type-graphql` 패키지와 `@nestjs/graphql` 모듈 간의 호환성 계층으로 코드 우선 접근 방식을 도입했습니다. 결국 우리 팀은 유연성 부족으로 모든 기능을 처음부터 다시 구현하기로 결정했습니다. 수많은 주요 변경 사항을 방지하기 위해 공개 API는 이전 버전과 호환되며 `type-graphql`과 유사할 수 있습니다.

기존 애플리케이션을 마이그레이션하려면 모든 `type-graphql` 가져 오기의 이름을 `@nestjs/graphql`로 변경하면 됩니다. 더 많은 고급기능을 사용한 경우 다음을 수행해야 할 수도 있습니다.

- `ClassType`(`type-graphql`에서 가져옴) 대신 `Type` (`@nestjs/common`에서 가져옴) 사용
- 리졸버 클래스 아래의 객체 타입(`@ObjectType()` 데코레이터로 주석이 달린 클래스)에서 `@Args()`가 필요한 메소드를 이동합니다 (그리고 `@Field()` 대신 `@ResolveField()` 데코레이터 사용).

#### Terminus

`@nestjs/terminus`의 버전 7 주요 릴리스에서 새로운 단순화된 API가 도입되었습니다.
상태 확인을 실행합니다. 이전에 필요했던 피어 종속성 `@godaddy/terminus`가 제거되어 상태확인을 Swagger에 자동으로 통합할 수 있습니다! `@godaddy/terminus` 삭제에 대한 자세한 내용은 [여기](https://github.com/nestjs/terminus/issues/340)를 참조하세요.

대부분의 사용자에게 가장 큰 변화는 `TerminusModule.forRootAsync` 함수의 제거입니다. 다음 메이저 버전에서는 이 기능이 완전히 제거됩니다.
새 API로 마이그레이션하려면 상태 확인을 처리할 새 컨트롤러를 만들어야 합니다.

```typescript
@@filename()
// Before
@Injectable()
export class TerminusOptionsService implements TerminusOptionsFactory {
  constructor(
    private http: HttpHealthIndicator,
  ) {}

  createTerminusOptions(): TerminusModuleOptions {
    const healthEndpoint: TerminusEndpoint = {
      url: '/health',
      healthIndicators: [
        async () => this.http.pingCheck('google', 'https://google.com'),
      ],
    };
    return {
      endpoints: [healthEndpoint],
    };
  }
}

@Module({
  imports: [
    TerminusModule.forRootAsync({
      useClass: TerminusOptionsService
    })
  ]
})
export class AppModule { }

// After
@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private http: HttpHealthIndicator,
  ) { }

  @Get()
  @HealthCheck()
  healthCheck() {
    return this.health.check([
      async () => this.http.pingCheck('google', 'https://google.com'),
    ]);
  }
}

@Module({
  controllers: [
    HealthController
  ],
  imports: [
    TerminusModule
  ]
})
export class AppModule { }

@@switch

// Before
@Injectable()
@Dependencies(HttpHealthIndicator)
export class TerminusOptionsService {
  constructor(
    private http,
  ) {}

  createTerminusOptions() {
    const healthEndpoint = {
      url: '/health',
      healthIndicators: [
        async () => this.http.pingCheck('google', 'https://google.com'),
      ],
    };
    return {
      endpoints: [healthEndpoint],
    };
  }
}

@Module({
  imports: [
    TerminusModule.forRootAsync({
      useClass: TerminusOptionsService
    })
  ]
})
export class AppModule { }

// After
@Controller('/health')
@Dependencies(HealthCheckService, HttpHealthIndicator)
export class HealthController {
  constructor(
    private health,
    private http,
  ) { }

  @Get('/')
  @HealthCheck()
  healthCheck() {
    return this.health.check([
      async () => this.http.pingCheck('google', 'https://google.com'),
    ])
  }
}

@Module({
  controllers: [
    HealthController
  ],
  imports: [
    TerminusModule
  ]
})
export class AppModule { }
```

> warning **경고** Nest 애플리케이션에서 [Global Prefix](faq#global-prefix)를 설정하고 `useGlobalPrefix` Terminus 옵션을 사용하지 않은 경우 상태 확인의 URL이 변경됩니다. 해당 URL에 대한 참조를 업데이트하거나 [nestjs/nest#963](https://github.com/nestjs/nest/issues/963)이 수정될 때까지 기존 Terminus API를 사용하세요.

레거시 API를 사용해야 하는 경우 당분간 지원 중단 메시지를 비활성화할 수도 있습니다.

```typescript
TerminusModule.forRootAsync({
  useFactory: () => ({
    disableDeprecationWarnings: true,
    endpoints: [
      // ...
    ]
  })
}
```

`main.ts` 파일에서 종료 후크를 활성화해야 합니다. Terminus 통합은 SIGTERM과 같은 POSIX 신호를 수신합니다 (자세한 내용은 [응용 프로그램 종료 장](fundamentals/lifecycle-events#application-shutdown) 참조). 활성화되면 상태확인 라우트는 서버가 종료될 때 서비스를 사용할 수 없음(503) HTTP 오류 응답으로 자동 응답합니다.

`@godaddy/terminus`를 제거하면 `import` 문을 업데이트해야 합니다.
대신 `@nestjs/terminus`를 사용합니다. 가장 주목할만한 것은 `HealthCheckError`의 가져오기입니다.

```typescript
@@filename(custom.health)
// Before
import { HealthCheckError } from '@godaddy/terminus';
// After
import { HealthCheckError } from '@nestjs/terminus';
```

완전히 마이그레이션 한 후에는 `@godaddy/terminus`를 제거해야 합니다.

```bash
npm uninstall --save @godaddy/terminus
```

##### `DNSHealthIndicator` deprecation

`DNSHealthIndicator`는 공식 [`HttpService`](/techniques/http-module#http-module)와의 일관성을 개선하고 기능에 더 적합한 이름을 선택하기 위해 `HttpHealthIndicator`로 이름이 변경되었습니다.

`DNSHealthIndicator` 참조를 `HttpHealthIndicator`로 바꾸어 마이그레이션하면 됩니다. 기능은 변경되지 않았습니다.

```typescript
// Before
@Controller('health')
export class HealthController {
  constructor(
    private dns: DNSHealthIndicator,
  ) { }
  ...
}
// After
@Controller('health')
export class HealthController {
  constructor(
    private http: HttpHealthIndicator,
  ) { }
  ...
}
```

#### HTTP exceptions body

이전에는 `HttpException` 클래스에 대해 생성된 응답 본문과 이 클래스에서 파생된 기타 예외(예: `BadRequestException` 또는 `NotFoundException`)가 일치하지 않았습니다. 최신 주요 릴리스에서 이러한 예외 응답은 동일한 구조를 따릅니다.

```typescript
/*
 * Sample outputs for "throw new ForbiddenException('Forbidden resource')"
 */

// Before
{
  "statusCode": 403,
  "message": "Forbidden resource"
}

// After
{
  "statusCode": 403,
  "message": "Forbidden resource",
  "error": "Forbidden"
}
```

#### Validation errors schema

이전 릴리스에서 `ValidationPipe`는 `class-validator` 패키지에서 반환한 `ValidationError` 객체의 배열을 던졌습니다. 이제 `ValidationPipe`는 오류 메시지를 나타내는 일반 문자열 목록에 오류를 매핑합니다.

```typescript
// Before
{
  "statusCode": 400,
  "error": "Bad Request",
  "message": [
    {
      "target": {},
      "property": "email",
      "children": [],
      "constraints": {
        "isEmail": "email must be an email"
      }
    }
  ]
}

// After
{
  "statusCode": 400,
  "message": ["email must be an email"],
  "error": "Bad Request"
}
```

이전 접근 방식을 선호하는 경우 `exceptionFactory` 함수를 설정하여 복원할 수 있습니다.

```typescript
new ValidationPipe({
  exceptionFactory: (errors) => new BadRequestException(errors),
});
```

#### Implicit type conversion (`ValidationPipe`)

자동 변환 옵션이 활성화되면(`transform: true`) `ValidationPipe`는 이제 기본 타입의 변환을 수행합니다. 다음 예에서 `findOne()` 메소드는 추출된 `id` 경로 매개변수를 나타내는 하나의 인수를 사용합니다.

```typescript
@Get(':id')
findOne(@Param('id') id: number) {
  console.log(typeof id === 'number'); // true
  return 'This action returns a user';
}
```

기본적으로 모든 경로 매개변수와 쿼리 매개변수는 네트워크를 통해 `string`로 제공됩니다. 위의 예에서 `id` 타입을 `number`(메서드 서명에서)로 지정했습니다. 따라서 `ValidationPipe`는 문자열 식별자를 숫자로 자동 변환하려고 시도합니다.

#### Microservice channels (bidirectional communication)

요청-응답 메시지 유형을 활성화하기 위해 Nest는 두개의 논리 채널을 만듭니다. 하나는 데이터 전송을 담당하고 다른 하나는 수신 응답을 기다립니다. NATS와 같은 일부 기본 전송의 경우 이 이중 채널 지원이 즉시 제공됩니다. 다른 경우 Nest는 별도의 채널을 수동으로 생성하여 보상합니다.

단일 메시지 핸들러 `@MessagePattern('getUsers')`가 있다고 가정해 보겠습니다. 과거에 Nest는 이 패턴에서 `getUsers_ack`(요청용)과 `getUsers_res`(응답용)의 두 채널을 구축했습니다. 버전 7에서는 이 이름 지정 체계가 변경됩니다. 이제 Nest는 대신 `getUsers`(요청용)와 `getUsers.reply`(응답용)를 빌드합니다. 또한 특히 MQTT 전송 전략의 경우 응답 채널은 `getUsers/reply`입니다 (토픽 와일드 카드와의 충돌을 방지하기 위해).

#### Deprecations

All deprecations (from Nest version 5 to version 6) have been finally removed (e.g., the deprecated `@ReflectMetadata` decorator).
모든 지원 중단(Nest 버전 5에서 버전 6까지)이 마침내 제거되었습니다 (예: 지원 중단 된 `@ReflectMetadata` 데코레이터).

#### Node.js

이 릴리스는 Node v8에 대한 지원을 중단합니다. 최신 LTS 버전을 사용하는 것이 좋습니다.
