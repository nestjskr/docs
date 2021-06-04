### HTTP module

[Axios](https://github.com/axios/axios)는 널리 사용되는 풍부한 기능의 HTTP 클라이언트 패키지입니다. Nest는 Axios를 래핑하고 내장된 `HttpModule`을 통해 노출합니다. `HttpModule`은 Axios 기반 메소드를 노출하여 HTTP 요청을 수행하는 `HttpService` 클래스를 내보냅니다. 라이브러리는 또한 결과 HTTP 응답을 `Observables`로 변환합니다.

`HttpService`를 사용하려면 먼저 `HttpModule`을 가져옵니다.

```typescript
@Module({
  imports: [HttpModule],
  providers: [CatsService],
})
export class CatsModule {}
```

다음으로 일반 생성자 주입을 사용하여 `HttpService`를 주입합니다.

> info **힌트** `HttpModule` 및 `HttpService`는 `@nestjs/common` 패키지에서 가져옵니다.

```typescript
@@filename()
@Injectable()
export class CatsService {
  constructor(private httpService: HttpService) {}

  findAll(): Observable<AxiosResponse<Cat[]>> {
    return this.httpService.get('http://localhost:3000/cats');
  }
}
@@switch
@Injectable()
@Dependencies(HttpService)
export class CatsService {
  constructor(httpService) {
    this.httpService = httpService;
  }

  findAll() {
    return this.httpService.get('http://localhost:3000/cats');
  }
}
```

모든 `HttpService` 메소드는 `Observable` 객체에 래핑된 `AxiosResponse`를 반환합니다.

#### Configuration

다양한 옵션으로 [Axios](https://github.com/axios/axios)를 구성하여 `HttpService`의 동작을 맞춤 설정할 수 있습니다. [여기](https://github.com/axios/axios#request-config)에서 자세히 알아보세요. 기본 Axios 인스턴스를 구성하려면 가져올 때 선택적 옵션 객체를 `HttpModule`의 `register()` 메서드에 전달합니다. 이 옵션 객체는 기본 Axios 생성자에 직접 전달됩니다.

```typescript
@Module({
  imports: [
    HttpModule.register({
      timeout: 5000,
      maxRedirects: 5,
    }),
  ],
  providers: [CatsService],
})
export class CatsModule {}
```

#### Async configuration


One technique is to use a factory function:
모듈 옵션을 정적으로 전달하는 대신 비동기적으로 전달해야 하는 경우 `registerAsync()` 메소드를 사용하십시오. 대부분의 동적 모듈과 마찬가지로 Nest는 비동기 구성을 처리하는 몇가지 기술을 제공합니다.

한가지 기술은 팩토리 기능을 사용하는 것입니다.

```typescript
HttpModule.registerAsync({
  useFactory: () => ({
    timeout: 5000,
    maxRedirects: 5,
  }),
});
```

다른 팩토리 공급자와 마찬가지로 팩토리 함수는 [async](/fundamentals/custom-providers#factory-providers-usefactory)일 수 있으며 `inject`를 통해 종속성을 주입할 수 있습니다.

```typescript
HttpModule.registerAsync({
  imports: [ConfigModule],
  useFactory: async (configService: ConfigService) => ({
    timeout: configService.getString('HTTP_TIMEOUT'),
    maxRedirects: configService.getString('HTTP_MAX_REDIRECTS'),
  }),
  inject: [ConfigService],
});
```

또는 아래와 같이 팩토리 대신 클래스를 사용하여 `HttpModule`을 구성할 수 있습니다.

```typescript
HttpModule.registerAsync({
  useClass: HttpConfigService,
});
```

위의 구성은 `HttpModule` 내부에서 `HttpConfigService`를 인스턴스화하여 옵션 객체를 생성하는데 사용합니다. 이 예에서 `HttpConfigService`는 아래와 같이 `HttpModuleOptionsFactory` 인터페이스를 구현해야합니다. `HttpModule`은 제공된 클래스의 인스턴스화된 객체에 대해 `createHttpOptions()` 메서드를 호출합니다.

```typescript
@Injectable()
class HttpConfigService implements HttpModuleOptionsFactory {
  createHttpOptions(): HttpModuleOptions {
    return {
      timeout: 5000,
      maxRedirects: 5,
    };
  }
}
```

`HttpModule` 내에 비공개 사본을 생성하는 대신 기존 옵션 공급자를 재사용하려면 `useExisting` 구문을 사용하세요.

```typescript
HttpModule.registerAsync({
  imports: [ConfigModule],
  useExisting: ConfigService,
});
```
