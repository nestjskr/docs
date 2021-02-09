### HTTP adapter

가끔 Nest 애플리케이션 컨텍스트 내에서 또는 외부에서 기본 HTTP 서버에 액세스하려고 할 수 있습니다.

모든 네티이브(플랫폼별) HTTP 서버/라이브러리 (예 : Express 및 Fastify) 인스턴스는 **어댑터**에 랩핑됩니다.<br>어댑터는 어플리케이션 컨텍스트에서 검색할 수 있을 뿐만 아니라 다른 공급자(Provider)에 주입할 수 있는 글로벌 프로바이더로 등록됩니다.

#### 어플리케이션 외부에서의 접근 전략

응용 프로그램의 외부 컨텍스트에서 `HttpAdapter`에 대한 참조를 가져오려면, `getHttpAdapter()` 메서드를 호출해야합니다.

```typescript
@@filename()
const app = await NestFactory.create(ApplicationModule);
const httpAdapter = app.getHttpAdapter();
```

#### 어플리케이션 내부에서의 접근 전략

어플리케이션의 내에서 `HttpAdapterHost`에 대한 참조를 가져오려면, 다른 기존 공급자(Provider)와 동일한 매개변수(예 : 생성자 주입)를 사용하여 `HttpAdapterHost`를 주입하세요.

```typescript
@@filename()
export class CatsService {
  constructor(private adapterHost: HttpAdapterHost) {}
}
@@switch
@Dependencies(HttpAdapterHost)
export class CatsService {
  constructor(adapterHost) {
    this.adapterHost = adapterHost;
  }
}
```

> `HttpAdapterHost` 에 대한 **힌트**는  `@nestjs/core` 패키지를 참조하세요.

`HttpAdapterHost` 는 실제 `HttpAdapter`가 **아닙니다**. 실제 `HttpAdapter` 인스턴스를 가져오려면 `httpAdapter` 속성에 액세스하기만 하면 됩니다.

```typescript
const adapterHost = app.get(HttpAdapterHost);
const httpAdapter = adapterHost.httpAdapter;
```

`httpAdapter` 는 기본 프레임워크에서 사용하는 HTTP 어댑터의 실제 인스턴스입니다. <br>`ExpressAdapter` 또는 `FastifyAdapter` 의 인스턴스입니다. (두 클래스 모두 `AbstractHttpAdapter`를 확장합니다).

어댑터 객체는 HTTP 서버와 상호 작용하는 몇가지 유용한 방법을 제공합니다. 하지만 라이브러리 인스턴스 (예 : Express 인스턴스)에 직접 액세스하려면 `getInstance()` 메서드를 호출해야합니다.

```typescript
const instance = httpAdapter.getInstance();
```
