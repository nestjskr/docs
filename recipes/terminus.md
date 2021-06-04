### Healthchecks (Terminus)

NestJS **Terminus** 통합은 **준비/활성 상태** 상태확인을 지원합니다. 상태확인은 복잡한 백엔드 설정과 관련하여 매우 중요합니다. 간단히 말해서 웹 개발 영역의 상태 확인은 일반적으로 `https://my-website.com/health/readiness`와 같은 특수 주소로 구성됩니다.
서비스 또는 인프라의 구성 요소(예: Kubernetes)는 이 주소를 지속적으로 확인합니다. 이 주소에 대한 `GET` 요청에서 반환된 HTTP 상태 코드에 따라 서비스는 "비정상" 응답을 수신할 때 조치를 취합니다.
"정상" 또는 "비정상"의 정의는 제공하는 서비스 유형에 따라 다르기 때문에 NestJS **Terminus** 통합은 일련의 **상태 표시기**를 지원합니다.

예를 들어, 웹 서버가 MongoDB를 사용하여 데이터를 저장하는 경우 MongoDB가 계속 실행되고 있는지 여부는 중요한 정보가됩니다.
이 경우 `MongooseHealthIndicator`를 사용할 수 있습니다. 올바르게 구성된 경우(나중에 자세히 설명) MongoDB가 실행중인지 여부에 따라 상태확인 주소가 정상 또는 비정상 HTTP 상태 코드를 반환합니다.

#### Getting started

`@nestjs/terminus`를 시작하려면 필요한 종속성을 설치해야합니다.

```bash
$ npm install --save @nestjs/terminus
```

#### Setting up a Healthcheck

상태확인은 **상태 표시기**의 요약을 나타냅니다. 상태 표시기는 정상 또는 비정상 상태인지 여부에 관계없이 서비스 검사를 실행합니다. 할당된 모든 상태 표시기가 실행중이면 상태확인이 긍정적입니다. 많은 애플리케이션에 유사한 상태 표시기가 필요하기 때문에 [@nestjs/terminus](https://github.com/nestjs/terminus)는 다음과 같은 사전 정의된 표시기를 제공합니다.

- `HttpHealthIndicator`
- `TypeOrmHealthIndicator`
- `MongooseHealthIndicator`
- `SequelizeHealthIndicator`
- `MicroserviceHealthIndicator`
- `GRPCHealthIndicator`
- `MemoryHealthIndicator`
- `DiskHealthIndicator`


첫번째 상태확인을 시작하려면 `TerminusModule`을 `AppModule`로 가져와야합니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { TerminusModule } from '@nestjs/terminus';

@Module({
  imports: [TerminusModule]
})
export class AppModule { }
```

상태확인은 [controller](/controllers)를 사용하여 실행할 수 있으며 [NestJS CLI](cli/overview)를 사용하여 쉽게 설정할 수 있습니다.

```bash
$ nest generate controller health
```

> info **정보** 애플리케이션에서 종료 후크를 활성화하는 것이 좋습니다. Terminus 통합은 활성화된 경우 이 수명주기 이벤트를 사용합니다. 종료 후크에 대한 자세한 내용은 [여기](fundamentals/lifecycle-events#application-shutdown)를 참조하세요.

#### HTTP Healthcheck

`@nestjs/terminus`를 설치하고 `TerminusModule`을 가져와서 새 컨트롤러를 생성했으면 상태확인을 생성할 준비가 된 것입니다.

```typescript
@@filename(health.controller)
@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private http: HttpHealthIndicator,
  ) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      () => this.http.pingCheck('nestjs-docs', 'https://docs.nestjs.com'),
    ]);
  }
}
@@switch
@Controller('health')
@Dependencies(HealthCheckService, HttpHealthIndicator)
export class HealthController {
  constructor(
    private health,
    private http,
  ) { }

  @Get()
  @HealthCheck()
  healthCheck() {
    return this.health.check([
      async () => this.http.pingCheck('nestjs-docs', 'https://docs.nestjs.com'),
    ])
  }
}
```

상태 확인은 이제 `https://docs.nestjs.com` 주소로 *GET*-요청을 보냅니다. 해당 주소에서 정상적인 응답을 받으면 `http://localhost:3000/health`의 라우트가 200 상태코드와 함께 다음 객체를 반환합니다.

```json
{
  "status": "ok",
  "info": {
    "nestjs-docs": {
      "status": "up"
    }
  },
  "error": {},
  "details": {
    "nestjs-docs": {
      "status": "up"
    }
  }
}
```

이 응답 객체의 인터페이스는 `HealthCheckResult` 인터페이스가 있는 `@nestjs/terminus` 패키지에서 액세스할 수 있습니다.

|           |                                                                                                                                                                                             |                                      |
|-----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------|
| `status`  | 상태 표시기가 실패하면 상태는 `'error'`가 됩니다. NestJS 앱이 종료되지만 여전히 HTTP 요청을 수락하는 경우 상태 확인은 `'shutting_down'` 상태가 됩니다. | `'error' \| 'ok' \| 'shutting_down'` |
| `info`    | 상태가 `'up'`이거나 다른 말로 "healthy"인 각 상태 표시기의 정보를 포함하는 객체입니다.                                                                              | `object`                             |
| `error`   | 상태가 `'down'`이거나 다른 말로 "unhealthy"인 각 상태 표시기의 정보를 포함하는 객체입니다.                                                                          | `object`                             |
| `details` | 각 상태 표시기의 모든 정보를 포함하는 객체                                                                                                                                  | `object`                             |

#### TypeOrm health indicator

Terminus는 상태 확인에 데이터베이스 확인을 추가하는 기능을 제공합니다. 이 Health 표시기를 시작하려면 [데이터베이스 장](/techniques/sql)을 확인하고 애플리케이션내에서 데이터베이스 연결이 설정되었는지 확인해야합니다.

> info **힌트** 뒤에서 `TypeOrmHealthIndicator`는 단순히 데이터베이스가 아직 살아 있는지 확인하는 데 자주 사용되는 `SELECT 1`-SQL 명령을 실행합니다. Oracle 데이터베이스를 사용하는 경우 `SELECT 1 FROM DUAL`을 사용합니다.


```typescript
@@filename(health.controller)
@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
  ) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      () => this.db.pingCheck('database'),
    ]);
  }
}
@@switch
@Controller('health')
@Dependencies(HealthCheckService, TypeOrmHealthIndicator)
export class HealthController {
  constructor(
    private health,
    private db,
  ) { }

  @Get()
  @HealthCheck()
  healthCheck() {
    return this.health.check([
      async () => this.db.pingCheck('database'),
    ])
  }
}
```

데이터베이스에 연결할 수 있는 경우 이제 `GET` 요청으로 `http://localhost:3000`을 요청할 때 다음 JSON 결과가 표시됩니다.

```json
{
  "status": "ok",
  "info": {
    "database": {
      "status": "up"
    }
  },
  "error": {},
  "details": {
    "database": {
      "status": "up"
    }
  }
}
```
앱에서 [다중 데이터베이스](techniques/database#multiple-databases)를 사용하는 경우 각 연결을 `HealthController`에 삽입해야합니다. 그런 다음 연결 참조를 `TypeOrmHealthIndicator`에 간단히 전달할 수 있습니다.

```typescript
@@filename(health.controller)
@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
    @InjectConnection('albumsConnection')
    private albumsConnection: Connection,
    @InjectConnection()
    private defaultConnection: Connection,
  ) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      () => this.db.pingCheck('albums-database', { connection: this.albumsConnection }),
      () => this.db.pingCheck('database', { connection: this.defaultConnection }),
    ]);
  }
}
```

#### Custom health indicator

경우에 따라 `@nestjs/terminus`에서 제공하는 사전 정의된 상태 표시기가 모든 상태확인 요구사항을 다루지 않습니다. 이 경우 필요에 따라 사용자 지정 상태 표시기를 설정할 수 있습니다.

사용자 지정 표시기(custom indicator)를 나타내는 서비스를 만들어 시작하겠습니다. 표시기의 구조에 대한 기본적인 이해를 위해 `DogHealthIndicator` 예제를 생성합니다. 모든 `Dog` 객체에 `'goodboy'` 타입이 있는 경우이 서비스는 `'up` 상태여야합니다. 해당 조건이 충족되지 않으면 오류가 발생합니다.

```typescript
@@filename(dog.health)
import { Injectable } from '@nestjs/common';
import { HealthIndicator, HealthIndicatorResult, HealthCheckError } from '@nestjs/terminus';

export interface Dog {
  name: string;
  type: string;
}

@Injectable()
export class DogHealthIndicator extends HealthIndicator {
  private dogs: Dog[] = [
    { name: 'Fido', type: 'goodboy' },
    { name: 'Rex', type: 'badboy' },
  ];

  async isHealthy(key: string): Promise<HealthIndicatorResult> {
    const badboys = this.dogs.filter(dog => dog.type === 'badboy');
    const isHealthy = badboys.length === 0;
    const result = this.getStatus(key, isHealthy, { badboys: badboys.length });

    if (isHealthy) {
      return result;
    }
    throw new HealthCheckError('Dogcheck failed', result);
  }
}
@@switch
import { Injectable } from '@nestjs/common';
import { HealthCheckError } from '@godaddy/terminus';

@Injectable()
export class DogHealthIndicator extends HealthIndicator {
  dogs = [
    { name: 'Fido', type: 'goodboy' },
    { name: 'Rex', type: 'badboy' },
  ];

  async isHealthy(key) {
    const badboys = this.dogs.filter(dog => dog.type === 'badboy');
    const isHealthy = badboys.length === 0;
    const result = this.getStatus(key, isHealthy, { badboys: badboys.length });

    if (isHealthy) {
      return result;
    }
    throw new HealthCheckError('Dogcheck failed', result);
  }
}
```

다음으로 해야할 일은 상태 표시기를 프로바이더로 등록하는 것입니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { TerminusModule } from '@nestjs/terminus';
import { DogHealthIndicator } from './dog.health';

@Module({
  controllers: [HealthController],
  imports: [TerminusModule],
  providers: [DogHealthIndicator]
})
export class AppModule { }
```

> info **힌트** 실제 애플리케이션에서 `DogHealthIndicator`는 별도의 모듈(예: `DogModule`)로 제공되어야하며, 그러면 `AppModule`에서 가져옵니다.

마지막으로 필요한 단계는 필요한 상태확인 엔드포인트에 현재 사용 가능한 상태 표시기를 추가하는 것입니다. 이를 위해 `HealthController`로 돌아가 `check` 함수에 추가합니다.

```typescript
@@filename(health.controller)
import { HealthCheckService } from '@nestjs/terminus';
import { Injectable } from '@nestjs/common';
import { DogHealthIndicator } from './dog.health';

@Injectable()
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private dogHealthIndicator: DogHealthIndicator
  ) {}

  @Get()
  @HealthCheck()
  healthCheck() {
    return this.health.check([
      async () => this.dogHealthIndicator.isHealthy('dog'),
    ])
  }
}
@@switch
import { HealthCheckService } from '@nestjs/terminus';
import { Injectable } from '@nestjs/common';
import { DogHealthIndicator } from './dog.health';

@Injectable()
@Dependencies(HealthCheckService, DogHealthIndicator)
export class HealthController {
  constructor(
    private health,
    private dogHealthIndicator
  ) {}

  @Get()
  @HealthCheck()
  healthCheck() {
    return this.health.check([
      async () => this.dogHealthIndicator.isHealthy('dog'),
    ])
  }
}
```

#### Examples

일부 작동 예제는 [여기](https://github.com/nestjs/terminus/tree/master/sample)에서 확인할 수 있습니다.
