### Module reference

Nest는 `ModuleRef` 클래스를 제공하여 내부 프로바이더 목록을 탐색하고 주입 토큰을 조회 키로 사용하는 프로바이더에 대한 참조를 얻습니다. `ModuleRef` 클래스는 정적 및 범위가 지정된 공급자를 모두 동적으로 인스턴스화하는 방법도 제공합니다. `ModuleRef`는 일반적인 방법으로 클래스에 삽입될 수 있습니다.

```typescript
@@filename(cats.service)
@Injectable()
export class CatsService {
  constructor(private moduleRef: ModuleRef) {}
}
@@switch
@Injectable()
@Dependencies(ModuleRef)
export class CatsService {
  constructor(moduleRef) {
    this.moduleRef = moduleRef;
  }
}
```

> info **힌트** `ModuleRef` 클래스는 `@nestjs/core` 패키지에서 가져옵니다.

#### Retrieving instances

`ModuleRef` 인스턴스(이하 **모듈 참조**라고 함)에는 `get()` 메서드가 있습니다. 이 메서드는 주입 토큰/클래스 이름을 사용하여 **현재** 모듈에 존재(인스턴스화)된 프로바이더, 컨트롤러 또는 주입 가능(예: 가드, 인터셉터 등)을 검색합니다.

```typescript
@@filename(cats.service)
@Injectable()
export class CatsService implements OnModuleInit {
  private service: Service;
  constructor(private moduleRef: ModuleRef) {}

  onModuleInit() {
    this.service = this.moduleRef.get(Service);
  }
}
@@switch
@Injectable()
@Dependencies(ModuleRef)
export class CatsService {
  constructor(moduleRef) {
    this.moduleRef = moduleRef;
  }

  onModuleInit() {
    this.service = this.moduleRef.get(Service);
  }
}
```

> warning **경고** `get()` 메서드를 사용하여 범위가 지정된 프로바이더(일시적 또는 요청 범위)를 검색할 수 없습니다. 대신 [아래](/fundamentals/module-ref#resolving-scoped-providers)에 설명된 기술을 사용하세요. [여기](/fundamentals/injection-scopes)에서 범위를 제어하는 방법을 알아보세요.

전역 컨텍스트에서 프로바이더를 검색하려면(예: 프로바이더가 다른 모듈에 삽입된 경우) `{{ '{' }} strict: false {{ '}' }}` 옵션을 `get()`에 두번째 인수로 전달합니다.

```typescript
this.moduleRef.get(Service, { strict: false });
```

#### Resolving scoped providers

범위가 지정된 프로바이더(임시 transient 또는 요청 범위 request-scoped)를 동적으로 확인하려면 `resolve()` 메서드를 사용하여 프로바이더의 주입 토큰을 인수로 전달합니다.

```typescript
@@filename(cats.service)
@Injectable()
export class CatsService implements OnModuleInit {
  private transientService: TransientService;
  constructor(private moduleRef: ModuleRef) {}

  async onModuleInit() {
    this.transientService = await this.moduleRef.resolve(TransientService);
  }
}
@@switch
@Injectable()
@Dependencies(ModuleRef)
export class CatsService {
  constructor(moduleRef) {
    this.moduleRef = moduleRef;
  }

  async onModuleInit() {
    this.transientService = await this.moduleRef.resolve(TransientService);
  }
}
```

`resolve()` 메서드는 자체 **DI 컨테이너 하위 트리**에서 공급자의 고유한 인스턴스를 반환합니다. 각 하위 트리에는 고유한 **컨텍스트 식별자**가 있습니다. 따라서 이 메서드를 두번 이상 호출하고 인스턴스 참조를 비교하면 같지 않음을 알 수 있습니다.

```typescript
@@filename(cats.service)
@Injectable()
export class CatsService implements OnModuleInit {
  constructor(private moduleRef: ModuleRef) {}

  async onModuleInit() {
    const transientServices = await Promise.all([
      this.moduleRef.resolve(TransientService),
      this.moduleRef.resolve(TransientService),
    ]);
    console.log(transientServices[0] === transientServices[1]); // false
  }
}
@@switch
@Injectable()
@Dependencies(ModuleRef)
export class CatsService {
  constructor(moduleRef) {
    this.moduleRef = moduleRef;
  }

  async onModuleInit() {
    const transientServices = await Promise.all([
      this.moduleRef.resolve(TransientService),
      this.moduleRef.resolve(TransientService),
    ]);
    console.log(transientServices[0] === transientServices[1]); // false
  }
}
```

여러 `resolve()` 호출에서 단일 인스턴스를 생성하고 생성된 동일한 DI 컨테이너 하위 트리를 공유하도록 하려면 컨텍스트 식별자를 `resolve()` 메서드에 전달할 수 있습니다. 컨텍스트 식별자를 생성하려면 `ContextIdFactory` 클래스를 사용하십시오. 이 클래스는 적절한 고유 식별자를 반환하는 `create()` 메서드를 제공합니다.

```typescript
@@filename(cats.service)
@Injectable()
export class CatsService implements OnModuleInit {
  constructor(private moduleRef: ModuleRef) {}

  async onModuleInit() {
    const contextId = ContextIdFactory.create();
    const transientServices = await Promise.all([
      this.moduleRef.resolve(TransientService, contextId),
      this.moduleRef.resolve(TransientService, contextId),
    ]);
    console.log(transientServices[0] === transientServices[1]); // true
  }
}
@@switch
@Injectable()
@Dependencies(ModuleRef)
export class CatsService {
  constructor(moduleRef) {
    this.moduleRef = moduleRef;
  }

  async onModuleInit() {
    const contextId = ContextIdFactory.create();
    const transientServices = await Promise.all([
      this.moduleRef.resolve(TransientService, contextId),
      this.moduleRef.resolve(TransientService, contextId),
    ]);
    console.log(transientServices[0] === transientServices[1]); // true
  }
}
```

> info **힌트** `ContextIdFactory` 클래스는 `@nestjs/core` 패키지에서 가져옵니다.

#### Registering `REQUEST` provider

수동으로 생성된 컨텍스트 식별자(`ContextIdFactory.create()` 사용)는 `REQUEST` 프로바이더가 Nest 종속성 주입 시스템에 의해 인스턴스화 및 관리되지 않으므로 `정의되지 않은 undefined` DI 하위 트리를 나타냅니다.

수동으로 생성된 DI 하위 트리에 대한 사용자 지정 `REQUEST` 객체를 등록하려면 다음과 같이 `ModuleRef#registerRequestByContextId()` 메서드를 사용합니다.

```typescript
const contextId = ContextIdFactory.create();
this.moduleRef.registerRequestByContextId(/* YOUR_REQUEST_OBJECT */, contextId);
```

#### Getting current sub-tree

경우에 따라 **요청 컨텍스트**내에서 요청 범위 프로바이더의 인스턴스를 확인하려고 할 수 있습니다. '`CatsService`가 요청 범위이고 요청 범위 프로바이더로도 표시된 `CatsRepository` 인스턴스를 확인하려고 한다고 가정해 보겠습니다. 동일한 DI 컨테이너 하위 트리를 공유하려면 새 컨텍스트 식별자(identifier)를 생성하는 대신 현재 컨텍스트 식별자를 가져와야합니다 (예: 위에 표시된대로 `ContextIdFactory.create()` 함수 사용). 현재 컨텍스트 식별자를 얻으려면 먼저 `@Inject()` 데코레이터를 사용하여 요청 객체를 삽입합니다.

```typescript
@@filename(cats.service)
@Injectable()
export class CatsService {
  constructor(
    @Inject(REQUEST) private request: Record<string, unknown>,
  ) {}
}
@@switch
@Injectable()
@Dependencies(REQUEST)
export class CatsService {
  constructor(request) {
    this.request = request;
  }
}
```

> info **힌트** [여기](/fundamentals/injection-scopes#request-provider)에서 요청 프로바이더에 대해 자세히 알아보세요.

이제 `ContextIdFactory` 클래스의 `getByRequest()` 메소드를 사용하여 요청 객체를 기반으로 컨텍스트 ID를 만들고 이를 `resolve()` 호출에 전달합니다.

```typescript
const contextId = ContextIdFactory.getByRequest(this.request);
const catsRepository = await this.moduleRef.resolve(CatsRepository, contextId);
```

#### Instantiating custom classes dynamically

**이전에 등록되지 않은** 클래스를 **프로바이더**로 동적으로 인스턴스화하려면 모듈 참조의 `create()` 메서드를 사용합니다.

```typescript
@@filename(cats.service)
@Injectable()
export class CatsService implements OnModuleInit {
  private catsFactory: CatsFactory;
  constructor(private moduleRef: ModuleRef) {}

  async onModuleInit() {
    this.catsFactory = await this.moduleRef.create(CatsFactory);
  }
}
@@switch
@Injectable()
@Dependencies(ModuleRef)
export class CatsService {
  constructor(moduleRef) {
    this.moduleRef = moduleRef;
  }

  async onModuleInit() {
    this.catsFactory = await this.moduleRef.create(CatsFactory);
  }
}
```

이 기술을 사용하면 프레임워크 컨테이너 외부에서 다른 클래스를 조건부로 인스턴스화 할 수 있습니다.

<app-banner-shop></app-banner-shop>
