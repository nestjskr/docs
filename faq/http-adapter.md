### HTTP adapter

경우에 따라 Nest 애플리케이션 컨텍스트내에서 또는 외부에서 기본 HTTP 서버에 액세스하려고 할 수 있습니다.

모든 네이티브(플랫폼 별) HTTP 서버/라이브러리(예: Express 및 Fastify) 인스턴스는 **어댑터**로 래핑됩니다. 어댑터는 애플리케이션 컨텍스트에서 검색할 수 있을 뿐만 아니라 다른 프로바이더에 삽입할 수 있는 전역적으로 사용 가능한 프로바이더로 등록됩니다.

#### Outside application context strategy

애플리케이션 컨텍스트 외부에서 `HttpAdapter`에 대한 참조를 가져오려면 `getHttpAdapter ()` 메소드를 호출하십시오.

```typescript
@@filename()
const app = await NestFactory.create(AppModule);
const httpAdapter = app.getHttpAdapter();
```

#### In-context strategy

애플리케이션 컨텍스트 내에서 `HttpAdapterHost`에 대한 참조를 가져오려면 다른 기존 프로바이더와 동일한 기술을 사용하여 삽입합니다 (예: 생성자 삽입 사용).

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

> info **힌트** `HttpAdapterHost`는 `@nestjs/core` 패키지에서 가져옵니다.

`HttpAdapterHost`는 실제 `HttpAdapter`가 **아닙니다**. 실제 `HttpAdapter` 인스턴스를 얻으려면 `httpAdapter` 속성에 액세스하면 됩니다.

```typescript
const adapterHost = app.get(HttpAdapterHost);
const httpAdapter = adapterHost.httpAdapter;
```

`httpAdapter`는 기본 프레임워크에서 사용하는 HTTP 어댑터의 실제 인스턴스입니다. `ExpressAdapter` 또는 `FastifyAdapter`의 인스턴스입니다(두 클래스 모두 `AbstractHttpAdapter`를 확장함).

어댑터 객체는 HTTP 서버와 상호작용하는 몇가지 유용한 메서드를 제공합니다. 그러나 라이브러리 인스턴스(예: Express 인스턴스)에 직접 액세스하려면 `getInstance()` 메서드를 호출하십시오.

```typescript
const instance = httpAdapter.getInstance();
```
