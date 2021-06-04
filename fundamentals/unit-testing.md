### Testing

자동화된 테스트는 심각한 소프트웨어 개발 노력의 필수적인 부분으로 간주됩니다. 자동화를 통해 개발중에 개별 테스트 또는 테스트 스위트를 쉽고 빠르게 반복할 수 있습니다. 이는 릴리스가 품질 및 성능 목표를 충족하는지 확인하는 데 도움이 됩니다. 자동화는 적용 범위를 늘리고 개발자에게 더 빠른 피드백 루프를 제공합니다. 자동화는 개별 개발자의 생산성을 높이고 소스 코드 제어 체크인, 기능 통합 및 버전 릴리스와 같은 중요한 개발 수명주기 단계에서 테스트가 실행되도록 합니다.

이러한 테스트는 단위 테스트, 종단 간 (e2e) 테스트, 통합 테스트 등 다양한 유형에 걸쳐있는 경우가 많습니다. 이점은 의심 할 여지가 없지만 설정하는 것은 지루할 수 있습니다. Nest는 효과적인 테스트를 포함한 개발 모범 사례를 홍보하기 위해 노력하고 있으므로 개발자와 팀이 테스트를 빌드하고 자동화하는 데 도움이되는 다음과 같은 기능을 포함합니다. 둥지:

- 구성 요소에 대한 기본 단위 테스트 및 애플리케이션에 대한 e2e 테스트를 자동으로 스캐폴드합니다.
- 기본 도구 제공(예: 격리된 모듈/애플리케이션 로더를 빌드하는 테스트 실행기)
- [Jest](https://github.com/facebook/jest) 및 [Supertest](https://github.com/visionmedia/supertest)와의 통합을 즉시 제공하는 동시에 테스트 도구에 대해 독립적입니다.
- 구성 요소를 쉽게 모의하기 위해 테스트 환경에서 Nest 종속성 주입 시스템을 사용할 수 있습니다.

언급했듯이 Nest는 특정 도구를 강제하지 않으므로 원하는 **테스트 프레임워크**를 사용할 수 있습니다. 필요한 요소(예: 테스트 실행기)를 교체하기 만하면 Nest의 기성 테스트 시설의 이점을 계속 누릴 수 있습니다.

#### Installation

시작하려면 먼저 필요한 패키지를 설치하십시오.

```bash
$ npm i --save-dev @nestjs/testing
```

#### Unit testing

다음 예제에서는 `CatsController`와 `CatsService`의 두 클래스를 테스트합니다. 앞서 언급했듯이 기본 테스트 프레임워크로 [Jest](https://github.com/facebook/jest)가 제공됩니다. 테스트 실행자 역할을 하며 모의(mocking), 감시(spying) 등에 도움이 되는 assert 함수 및 테스트 이중 유틸리티를 제공합니다. 다음 기본 테스트에서는 이러한 클래스를 수동으로 인스턴스화하고 컨트롤러와 서비스가 API 계약을 이행하는지 확인합니다.

```typescript
@@filename(cats.controller.spec)
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

describe('CatsController', () => {
  let catsController: CatsController;
  let catsService: CatsService;

  beforeEach(() => {
    catsService = new CatsService();
    catsController = new CatsController(catsService);
  });

  describe('findAll', () => {
    it('should return an array of cats', async () => {
      const result = ['test'];
      jest.spyOn(catsService, 'findAll').mockImplementation(() => result);

      expect(await catsController.findAll()).toBe(result);
    });
  });
});
@@switch
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

describe('CatsController', () => {
  let catsController;
  let catsService;

  beforeEach(() => {
    catsService = new CatsService();
    catsController = new CatsController(catsService);
  });

  describe('findAll', () => {
    it('should return an array of cats', async () => {
      const result = ['test'];
      jest.spyOn(catsService, 'findAll').mockImplementation(() => result);

      expect(await catsController.findAll()).toBe(result);
    });
  });
});
```

> info **힌트** 테스트 파일을 테스트하는 클래스 근처에 두십시오. 테스트 파일에는 `.spec` 또는 `.test` 접미사가 있어야 합니다.

위의 샘플은 사소한 것이기 때문에 실제로 Nest와 관련된 것은 테스트하지 않습니다. 실제로 우리는 의존성 주입을 사용하지도 않습니다(`CatsService`의 인스턴스를 `catsController`에 전달한다는 점에 유의하세요). 테스트중인 클래스를 수동으로 인스턴스화하는 이러한 형식의 테스트는 프레임워크와 독립적이므로 종종 **격리된 테스트**라고 합니다. Nest 기능을 보다 광범위하게 사용하는 애플리케이션을 테스트하는 데 도움이 되는 몇가지 고급기능을 소개하겠습니다.

#### Testing utilities

`@nestjs/testing` 패키지는 보다 강력한 테스트 프로세스를 가능하게 하는 유틸리티 세트를 제공합니다. 내장된 `Test` 클래스를 사용하여 이전 예제를 다시 작성해 보겠습니다.

```typescript
@@filename(cats.controller.spec)
import { Test } from '@nestjs/testing';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

describe('CatsController', () => {
  let catsController: CatsController;
  let catsService: CatsService;

  beforeEach(async () => {
    const moduleRef = await Test.createTestingModule({
        controllers: [CatsController],
        providers: [CatsService],
      }).compile();

    catsService = moduleRef.get<CatsService>(CatsService);
    catsController = moduleRef.get<CatsController>(CatsController);
  });

  describe('findAll', () => {
    it('should return an array of cats', async () => {
      const result = ['test'];
      jest.spyOn(catsService, 'findAll').mockImplementation(() => result);

      expect(await catsController.findAll()).toBe(result);
    });
  });
});
@@switch
import { Test } from '@nestjs/testing';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

describe('CatsController', () => {
  let catsController;
  let catsService;

  beforeEach(async () => {
    const moduleRef = await Test.createTestingModule({
        controllers: [CatsController],
        providers: [CatsService],
      }).compile();

    catsService = moduleRef.get(CatsService);
    catsController = moduleRef.get(CatsController);
  });

  describe('findAll', () => {
    it('should return an array of cats', async () => {
      const result = ['test'];
      jest.spyOn(catsService, 'findAll').mockImplementation(() => result);

      expect(await catsController.findAll()).toBe(result);
    });
  });
});
```

`Test` 클래스는 기본적으로 전체 Nest 런타임을 모의하는 애플리케이션 실행 컨텍스트를 제공하는 데 유용하지만 모킹 및 재정의를 포함하여 클래스 인스턴스를 쉽게 관리할 수 있는 후크를 제공합니다. `Test` 클래스에는 모듈 메타데이터 객체를 인수로 사용하는 `createTestingModule()` 메서드가 있습니다 (`@Module()` 데코레이터에 전달하는 동일한 객체). 이 메서드는 몇가지 메서드를 제공하는 `TestingModule` 인스턴스를 반환합니다. 단위 테스트에서 중요한 것은 `compile()` 메서드입니다. 이 메소드는 의존성이 있는 모듈을 부트스트랩하고(애플리케이션이 `NestFactory.create()`를 사용하여 기존 `main.ts` 파일에서 부트스트랩되는 방식과 유사함) 테스트할 준비가 된 모듈을 반환합니다.

> info **힌트** `compile()` 메서드는 **비동기**이므로 기다려야 합니다. 모듈이 컴파일되면 `get()`메서드를 사용하여 선언하는 **정적** 인스턴스(컨트롤러 및 프로바이더)를 검색할 수 있습니다.

`TestingModule`은 [모듈 참조](/fundamentals/module-ref) 클래스에서 상속되므로 범위가 지정된 프로바이더(임시 또는 요청 범위)를 동적으로 확인하는 기능입니다. `resolve()` 메서드를 사용하여 이 작업을 수행합니다(`get()` 메서드는 정적 인스턴스만 검색할 수 있음).

```typescript
const moduleRef = await Test.createTestingModule({
  controllers: [CatsController],
  providers: [CatsService],
}).compile();

catsService = await moduleRef.resolve(CatsService);
```

> warning **경고** `resolve()` 메서드는 자체 **DI 컨테이너 하위 트리**에서 프로바이더의 고유한 인스턴스를 반환합니다. 각 하위 트리에는 고유한 컨텍스트 식별자가 있습니다. 따라서 이 메서드를 두번 이상 호출하고 인스턴스 참조를 비교하면 같지 않음을 알 수 있습니다.

> info **힌트** 모듈 참조 기능에 대한 자세한 내용은 [여기](/fundamentals/module-ref)를 참조하세요.

프로바이더의 프로덕션 버전을 사용하는 대신 테스트 목적으로 [사용자 지정 공급자](/fundamentals/custom-providers)로 재정의할 수 있습니다. 예를 들어 라이브 데이터베이스에 연결하는 대신 데이터베이스 서비스를 모의할 수 있습니다. 다음 섹션에서 재정의를 다루 겠지만 단위 테스트에도 사용할 수 있습니다.

<app-banner-courses></app-banner-courses>

#### End-to-end testing

개별 모듈 및 클래스에 초점을 맞춘 단위 테스트와 달리 엔드-투-엔드(e2e) 테스트는 최종 사용자가 프로덕션에서 가질 수 있는 상호 작용의 종류에 더 가까운 보다 총체적인 수준에서 클래스 및 모듈의 상호 작용을 다룹니다. 체계. 애플리케이션이 성장함에 따라 각 API 엔드 포인트의 엔드-투-엔드 동작을 수동으로 테스트하기가 어려워집니다. 자동화된 종단간 테스트는 시스템의 전반적인 동작이 정확하고 프로젝트 요구사항을 충족하는지 확인하는 데 도움이됩니다. e2e 테스트를 수행하기 위해 방금 **단위 테스트**에서 다룬 것과 유사한 구성을 사용합니다. 또한 Nest를 사용하면 [Supertest](https://github.com/visionmedia/supertest) 라이브러리를 사용하여 HTTP 요청을 쉽게 시뮬레이션할 수 있습니다.

```typescript
@@filename(cats.e2e-spec)
import * as request from 'supertest';
import { Test } from '@nestjs/testing';
import { CatsModule } from '../../src/cats/cats.module';
import { CatsService } from '../../src/cats/cats.service';
import { INestApplication } from '@nestjs/common';

describe('Cats', () => {
  let app: INestApplication;
  let catsService = { findAll: () => ['test'] };

  beforeAll(async () => {
    const moduleRef = await Test.createTestingModule({
      imports: [CatsModule],
    })
      .overrideProvider(CatsService)
      .useValue(catsService)
      .compile();

    app = moduleRef.createNestApplication();
    await app.init();
  });

  it(`/GET cats`, () => {
    return request(app.getHttpServer())
      .get('/cats')
      .expect(200)
      .expect({
        data: catsService.findAll(),
      });
  });

  afterAll(async () => {
    await app.close();
  });
});
@@switch
import * as request from 'supertest';
import { Test } from '@nestjs/testing';
import { CatsModule } from '../../src/cats/cats.module';
import { CatsService } from '../../src/cats/cats.service';
import { INestApplication } from '@nestjs/common';

describe('Cats', () => {
  let app: INestApplication;
  let catsService = { findAll: () => ['test'] };

  beforeAll(async () => {
    const moduleRef = await Test.createTestingModule({
      imports: [CatsModule],
    })
      .overrideProvider(CatsService)
      .useValue(catsService)
      .compile();

    app = moduleRef.createNestApplication();
    await app.init();
  });

  it(`/GET cats`, () => {
    return request(app.getHttpServer())
      .get('/cats')
      .expect(200)
      .expect({
        data: catsService.findAll(),
      });
  });

  afterAll(async () => {
    await app.close();
  });
});
```

> info **힌트** HTTP 어댑터로 [Fastify](/techniques/performance)를 사용하는 경우 약간 다른 구성이 필요하며 기본 제공 테스트 기능이 있습니다.
>
> ```ts
> let app: NestFastifyApplication;
>
> beforeAll(async () => {
>   app = moduleRef.createNestApplication<NestFastifyApplication>(
>     new FastifyAdapter(),
>   );
>
>   await app.init();
>   await app.getHttpAdapter().getInstance().ready();
> })
>
> it(`/GET cats`, () => {
>   return app
>     .inject({
>       method: 'GET',
>       url: '/cats'
>     }).then(result => {
>       expect(result.statusCode).toEqual(200)
>       expect(result.payload).toEqual(/* expectedPayload */)
>     });
> })
> ```

이 예에서는 앞에서 설명한 몇가지 개념을 기반으로 합니다. 이전에 사용한 `compile()` 메서드 외에도 이제 `createNestApplication()` 메서드를 사용하여 전체 Nest 런타임 환경을 인스턴스화합니다. 실행중인 앱에 대한 참조를 `app` 변수에 저장하여 HTTP 요청을 시뮬레이션하는 데 사용할 수 있습니다.

Supertest의 `request()` 함수를 사용하여 HTTP 테스트를 시뮬레이션합니다. 이러한 HTTP 요청이 실행중인 Nest 앱으로 라우팅되기를 원하므로 `request()` 함수에 Nest의 기반이되는 HTTP 리스너에 대한 참조를 전달합니다(이는 Express 플랫폼에서 제공할 수 있음). 따라서 구성 `request(app.getHttpServer())`. `request()` 호출은 이제 Nest 앱에 연결된 래핑된 HTTP 서버를 전달합니다. 이 서버는 실제 HTTP 요청을 시뮬레이션하는 메소드를 노출합니다. 예를 들어 `request(...).get('/cats')`를 사용하면 네트워크를 통해 들어오는 `get '/cats'`와 같은 **실제** HTTP 요청과 동일한 Nest 앱에 대한 요청이 시작됩니다.

이 예에서는 테스트할 수 있는 하드코딩된 값을 간단히 반환하는 `CatsService`의 대체(test-double) 구현도 제공합니다. 이러한 대체 구현을 제공하려면 `overrideProvider()`를 사용하십시오. 마찬가지로 Nest는 각각 `overrideGuard()`, `overrideInterceptor()`, `overrideFilter()` 및 `overridePipe()` 메서드를 사용하여 가드, 인터셉터, 필터 및 파이프를 재정의하는 메서드를 제공합니다.

각 재정의 메서드는 [커스텀 프로바이더](/fundamentals/custom-providers)에 대해 설명된 메서드를 미러링하는 3가지 메서드가 있는 객체를 반환합니다.

- `useClass`: 객체(프로바이더, 가드 등)를 재정의할 인스턴스를 제공하기 위해 인스턴스화될 클래스를 제공합니다.
- `useValue`: 객체를 재정의할 인스턴스를 제공합니다.
- `useFactory`: 객체를 재정의할 인스턴스를 반환하는 함수를 제공합니다.

각 재정의 메서드 유형은 차례로 `TestingModule` 인스턴스를 반환하므로 [fluent style](https://en.wikipedia.org/wiki/Fluent_interface)의 다른 메서드와 연결할 수 있습니다. 이러한 체인의 끝에 `compile()`을 사용하여 Nest가 모듈을 인스턴스화하고 초기화하도록 해야합니다.

또한 때로는 사용자 정의 로거를 제공하고 싶을 수도 있습니다. 테스트가 실행될 때(예: CI 서버에서). `setLogger()` 메소드를 사용하고 `LoggerService` 인터페이스를 충족하는 객체를 전달하여 `TestModuleBuilder`에 테스트중에 기록하는 방법을 지시합니다 (기본적으로 "오류 error" 로그만 콘솔에 기록됩니다).

컴파일된 모듈에는 다음 표에 설명된대로 몇가지 유용한 메서드가 있습니다.

<table>
  <tr>
    <td>
      <code>createNestApplication()</code>
    </td>
    <td>
      주어진 모듈을 기반으로 Nest 애플리케이션 (<code>INestApplication</code> 인스턴스)을 만들고 반환합니다.
      <code>init()</code> 메서드를 사용하여 애플리케이션을 수동으로 초기화해야 합니다.
    </td>
  </tr>
  <tr>
    <td>
      <code>createNestMicroservice()</code>
    </td>
    <td>
      지정된 모듈을 기반으로 Nest 마이크로 서비스(<code>INestMicroservice</code> 인스턴스)를 만들고 반환합니다.
    </td>
  </tr>
  <tr>
    <td>
      <code>get()</code>
    </td>
    <td>
      애플리케이션 컨텍스트에서 사용 가능한 컨트롤러 또는 프로바이더(가드, 필터등 포함)의 정적 인스턴스를 검색합니다. <a href="/fundamentals/module-ref">모듈 참조</a> 클래스에서 상속됩니다.
    </td>
  </tr>
  <tr>
     <td>
      <code>resolve()</code>
    </td>
    <td>
      애플리케이션 컨텍스트에서 사용할 수 있는 컨트롤러 또는 프로바이더(가드, 필터등 포함)의 동적으로 생성된 범위 인스턴스(요청 또는 일시적)를 검색합니다. <a href="/fundamentals/module-ref">모듈 참조</a> 클래스에서 상속됩니다.
    </td>
  </tr>
  <tr>
    <td>
      <code>select()</code>
    </td>
    <td>
      모듈의 종속성 그래프를 탐색합니다. 선택한 모듈에서 특정 인스턴스를 검색하는 데 사용할 수 있습니다 (<code>get()</code> 메서드에서 엄격 모드 (<code>strict: true</code>)와 함께 사용됨).
    </td>
  </tr>
</table>

> info **힌트** e2e 테스트 파일을 `test` 디렉토리에 보관하세요. 테스트 파일에는 `.e2e-spec` 접미사가 있어야 합니다.

#### Overriding globally registered enhancers

전역적으로 등록된 가드(또는 파이프, 인터셉터 또는 필터)가 있는 경우 해당 인핸서를 재정의하기 위해 몇가지 단계를 더 수행해야 합니다. 원래 등록을 요약하면 다음과 같습니다.

```typescript
providers: [
  {
    provide: APP_GUARD,
    useClass: JwtAuthGuard,
  },
],
```

`APP_*` 토큰을 통해 가드를 "다중" 프로바이더로 등록하는 것입니다. 여기서 `JwtAuthGuard`를 교체하려면 등록시 이 슬롯에 있는 기존 프로바이더를 사용해야 합니다.

```typescript
providers: [
  {
    provide: APP_GUARD,
    useExisting: JwtAuthGuard,
  },
  JwtAuthGuard,
],
```

> info **힌트** `useClass`를 `useExisting`으로 변경하여 Nest가 토큰 뒤에서 인스턴스화하는 대신 등록된 프로바이더를 참조하세요.

이제 `JwtAuthGuard`는 `TestingModule`을 만들 때 재정의할 수 있는 일반 프로바이더로 Nest에 표시됩니다.

```typescript
const moduleRef = await Test.createTestingModule({
  imports: [AppModule],
})
  .overrideProvider(JwtAuthGuard)
  .useClass(MockAuthGuard)
  .compile();
```

이제 모든 테스트에서 모든 요청에 `MockAuthGuard`를 사용합니다.

#### Testing request-scoped instances

[요청 범위](/fundamentals/injection-scopes) 프로바이더는 들어오는 각 **요청**에 대해 고유하게 생성됩니다. 요청이 처리를 완료한 후 인스턴스가 가비지 수집됩니다. 테스트된 요청을 위해 특별히 생성된 종속성 주입 하위 트리에 액세스할 수 없기 때문에 문제가 됩니다.

우리는(위 섹션을 기반으로) `resolve()` 메서드를 사용하여 동적으로 인스턴스화된 클래스를 검색할 수 있다는 것을 알고 있습니다.
또한 [여기](/fundamentals/module-ref#resolving-scoped-providers)에 설명된 것처럼 DI 컨테이너 하위 트리의 수명주기를 제어하기 위해 고유한 컨텍스트 식별자를 전달할 수 있습니다. 이를 테스트 컨텍스트에서 어떻게 활용합니까?

전략은 미리 컨텍스트 식별자를 생성하고 Nest가 이 특정 ID를 사용하여 들어오는 모든 요청에 대한 하위 트리를 만들도록 하는 것입니다. 이러한 방식으로 테스트된 요청을 위해 생성된 인스턴스를 검색할 수 있습니다.

이를 수행하려면 `ContextIdFactory`에서 `jest.spyOn()`을 사용하십시오.

```typescript
const contextId = ContextIdFactory.create();
jest
  .spyOn(ContextIdFactory, 'getByRequest')
  .mockImplementation(() => contextId);
```

이제 `contextId`를 사용하여 후속 요청에 대해 생성된 단일 DI 컨테이너 하위 트리에 액세스할 수 있습니다.

```typescript
catsService = await moduleRef.resolve(CatsService, contextId);
```
