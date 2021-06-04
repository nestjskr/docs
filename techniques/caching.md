### Caching

캐싱은 앱의 성능을 개선하는 데 도움이 되는 훌륭하고 간단한 **기술**입니다. 고성능 데이터 액세스를 제공하는 임시 데이터 저장소 역할을합니다.

#### Installation

먼저 필수 패키지를 설치하십시오.

```bash
$ npm install cache-manager
$ npm install -D @types/cache-manager
```

#### In-memory cache

Nest는 다양한 캐시 스토리지 제공업체를 위한 통합 API를 제공합니다. 내장된 것은 메모리내 데이터 저장소입니다. 그러나 Redis와 같은 보다 포괄적인 솔루션으로 쉽게 전환할 수 있습니다.

캐싱을 활성화하려면 `CacheModule`을 가져 와서 `register()`메서드를 호출합니다.

```typescript
import { CacheModule, Module } from '@nestjs/common';
import { AppController } from './app.controller';

@Module({
  imports: [CacheModule.register()],
  controllers: [AppController],
})
export class AppModule {}
```

#### Interacting with the Cache store

캐시 관리자 인스턴스와 상호 작용하려면 다음과 같이 `CACHE_MANAGER` 토큰을 사용하여 클래스에 삽입합니다.

```typescript
constructor(@Inject(CACHE_MANAGER) private cacheManager: Cache) {}
```

> info **힌트** `Cache` 클래스는 `cache-manager`에서 가져오는 반면 `CACHE_MANAGER` 토큰은 `@nestjs/common` 패키지에서 가져옵니다.

`Cache` 인스턴스의 `get` 메소드(`cache-manager` 패키지)는 캐시에서 항목을 검색하는 데 사용됩니다. 항목이 캐시에 없으면 예외가 발생합니다.

```typescript
const value = await this.cacheManager.get('key');
```

캐시에 항목을 추가하려면 `set` 메소드를 사용하세요.

```typescript
await this.cacheManager.set('key', 'value');
```

캐시의 기본 만료 시간은 5 초입니다.

다음과 같이 이 특정 키에 대한 TTL(만료 시간)을 수동으로 지정할 수 있습니다.

```typescript
await this.cacheManager.set('key', 'value', { ttl: 1000 });
```

캐시 만료를 비활성화하려면 `ttl` 구성 속성을 `null`로 설정합니다.

```typescript
await this.cacheManager.set('key', 'value', { ttl: null });
```

캐시에서 항목을 제거하려면 `del` 메소드를 사용하십시오.

```typescript
await this.cacheManager.del('key');
```

전체 캐시를 지우려면 `reset` 메소드를 사용하십시오.

```typescript
await this.cacheManager.reset();
```

#### Auto-caching responses

> warning **경고** [GraphQL](/graphql/quick-start) 애플리케이션에서 인터셉터는 각 필드 리졸버에 대해 개별적으로 실행됩니다. 따라서 `CacheModule`(인터셉터를 사용하여 응답을 캐시함)은 제대로 작동하지 않습니다.

To enable auto-caching responses, just tie the `CacheInterceptor` where you want to cache data.
자동 캐싱 응답을 사용하려면 데이터를 캐시하려는 위치에 `CacheInterceptor`를 연결하면 됩니다.

```typescript
@Controller()
@UseInterceptors(CacheInterceptor)
export class AppController {
  @Get()
  findAll(): string[] {
    return [];
  }
}
```

> warning **경고** `GET` 엔드포인트만 캐시됩니다. 또한 네이티브 응답 객체 (`@Res()`)를 삽입하는 HTTP 서버 경로는 Cache Interceptor를 사용할 수 없습니다. 자세한 내용은 [응답 매핑](/interceptors#response-mapping)을 참조하세요.

필요한 상용구의 양을 줄이려면 `CacheInterceptor`를 모든 엔드포인트에 전역적으로 바인딩할 수 있습니다.

```typescript
import { CacheModule, Module, CacheInterceptor } from '@nestjs/common';
import { AppController } from './app.controller';
import { APP_INTERCEPTOR } from '@nestjs/core';

@Module({
  imports: [CacheModule.register()],
  controllers: [AppController],
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: CacheInterceptor,
    },
  ],
})
export class AppModule {}
```

#### Customize caching

캐시된 모든 데이터에는 자체 만료 시간(TTL)이 있습니다. 기본값을 맞춤 설정하려면 옵션 객체를 `register()` 메서드에 전달합니다.

```typescript
CacheModule.register({
  ttl: 5, // seconds
  max: 10, // maximum number of items in cache
});
```

#### Global cache overrides

전역 캐시가 활성화된 동안 캐시 항목은 라우트 경로에 따라 자동 생성되는 `CacheKey` 아래에 저장됩니다. 특정 캐시 설정 (`@CacheKey()` 및 `@CacheTTL()`)을 메서드별로 재정의하여 개별 컨트롤러 메서드에 대한 사용자 지정 캐싱 전략을 허용할 수 있습니다. [다른 캐시 저장소](/techniques/caching#different-stores)를 사용하는 동안 가장 관련이 있을 수 있습니다.

```typescript
@Controller()
export class AppController {
  @CacheKey('custom_key')
  @CacheTTL(20)
  findAll(): string[] {
    return [];
  }
}
```

> info **힌트** `@CacheKey()` 및 `@CacheTTL()` 데코레이터는 `@nestjs/common` 패키지에서 가져옵니다.

`@CacheKey()` 데코레이터는 해당하는 `@CacheTTL()` 데코레이터와 함께 또는 없이 사용할 수 있으며 그 반대의 경우도 마찬가지입니다. `@CacheKey()` 또는 `@CacheTTL()`만 재정의하도록 선택할 수 있습니다. 데코레이터로 재정의되지 않은 설정은 전역적으로 등록된 기본값을 사용합니다 ([캐싱 사용자 지정](/techniques/caching#customize-caching) 참조).

#### WebSockets and Microservices

WebSocket 구독자 및 Microservice의 패턴(사용중인 전송 방법에 관계없이)에 `CacheInterceptor`를 적용 할 수도 있습니다.

```typescript
@@filename()
@CacheKey('events')
@UseInterceptors(CacheInterceptor)
@SubscribeMessage('events')
handleEvent(client: Client, data: string[]): Observable<string[]> {
  return [];
}
@@switch
@CacheKey('events')
@UseInterceptors(CacheInterceptor)
@SubscribeMessage('events')
handleEvent(client, data) {
  return [];
}
```

그러나 나중에 캐시된 데이터를 저장하고 검색하는 데 사용되는 키를 지정하려면 추가 `@CacheKey()` 데코레이터가 필요합니다. 또한 **모든 것을 캐시해서는 안됩니다**. 단순히 데이터를 쿼리하는 것이 아니라 일부 비즈니스 작업을 수행하는 작업은 캐시되지 않아야합니다.

또한 `@CacheTTL()` 데코레이터를 사용하여 캐시 만료 시간(TTL)을 지정할 수 있으며, 이는 전역 기본 TTL 값을 재정의합니다.

```typescript
@@filename()
@CacheTTL(10)
@UseInterceptors(CacheInterceptor)
@SubscribeMessage('events')
handleEvent(client: Client, data: string[]): Observable<string[]> {
  return [];
}
@@switch
@CacheTTL(10)
@UseInterceptors(CacheInterceptor)
@SubscribeMessage('events')
handleEvent(client, data) {
  return [];
}
```

> info **힌트** `@CacheTTL()` 데코레이터는 해당하는 `@CacheKey()` 데코레이터와 함께 또는 없이 사용할 수 있습니다.

#### Adjust tracking

기본적으로 Nest는 요청 URL(HTTP 앱에서) 또는 캐시 키(`@CacheKey()` 데코레이터를 통해 설정된 웹소켓 및 마이크로서비스 앱에서)를 사용하여 캐시 레코드를 엔드포인트와 연결합니다. 그럼에도 불구하고, 예를 들어 HTTP 헤더(예: `profile` 엔드포인트를 올바르게 식별하기 위한 `Authorization`)를 사용하는 등 다양한 요인을 기반으로 추적을 설정하고자 할 수 있습니다.

이를 위해 `CacheInterceptor`의 하위 클래스를 만들고 `trackBy()` 메서드를 재정의합니다.

```typescript
@Injectable()
class HttpCacheInterceptor extends CacheInterceptor {
  trackBy(context: ExecutionContext): string | undefined {
    return 'key';
  }
}
```

#### Different stores

이 서비스는 내부적으로 [cache-manager](https://github.com/BryanDonovan/node-cache-manager)를 활용합니다. `cache-manager` 패키지는 [Redis](https://github.com/dabroek/node-cache-manager-redis-store) 저장소와 같은 다양한 유용한 저장소(store)를 지원합니다. 지원되는 저장소의 전체 목록은 [여기](https://github.com/BryanDonovan/node-cache-manager#store-engines)에서 확인할 수 있습니다. Redis 저장소(store)를 설정하려면 해당 옵션과 함께 패키지를 `register()` 메서드에 전달하면됩니다.

```typescript
import * as redisStore from 'cache-manager-redis-store';
import { CacheModule, Module } from '@nestjs/common';
import { AppController } from './app.controller';

@Module({
  imports: [
    CacheModule.register({
      store: redisStore,
      host: 'localhost',
      port: 6379,
    }),
  ],
  controllers: [AppController],
})
export class AppModule {}
```

#### Async configuration

컴파일 타임에 정적으로 전달하는 대신 모듈 옵션을 비동기적으로 전달할 수 있습니다. 이 경우 비동기 구성을 처리하는 여러 방법을 제공하는 `registerAsync()` 메서드를 사용합니다.

한가지 접근 방식은 팩토리 함수를 사용하는 것입니다.

```typescript
CacheModule.registerAsync({
  useFactory: () => ({
    ttl: 5,
  }),
});
```

우리 팩토리는 다른 모든 비동기 모듈 팩토리처럼 동작합니다 (`async`일 수 있으며 `inject`을 통해 종속성을 주입할 수 있습니다).

```typescript
CacheModule.registerAsync({
  imports: [ConfigModule],
  useFactory: async (configService: ConfigService) => ({
    ttl: configService.get('CACHE_TTL'),
  }),
  inject: [ConfigService],
});
```

또는 `useClass` 메소드를 사용할 수 있습니다.

```typescript
CacheModule.registerAsync({
  useClass: CacheConfigService,
});
```

위의 구성은 `CacheModule` 내에서 `CacheConfigService`를 인스턴스화하고 이를 사용하여 옵션 객체를 가져옵니다. `CacheConfigService`는 구성 옵션을 제공하기 위해 `CacheOptionsFactory` 인터페이스를 구현해야합니다.

```typescript
@Injectable()
class CacheConfigService implements CacheOptionsFactory {
  createCacheOptions(): CacheModuleOptions {
    return {
      ttl: 5,
    };
  }
}
```

다른 모듈에서 가져온 기존 구성 프로바이더를 사용하려면 `useExisting` 구문을 사용하십시오.

```typescript
CacheModule.registerAsync({
  imports: [ConfigModule],
  useExisting: ConfigService,
});
```

이것은 한가지 중요한 차이점을 제외하고는 `useClass`와 동일하게 작동합니다. `CacheModule`은 자체 인스턴스를 생성하는 대신 이미 생성된 `ConfigService`를 재사용하기 위해 가져온 모듈을 조회합니다.

#### Example

작동하는 예는 [여기](https://github.com/nestjs/nest/tree/master/sample/20-cache)에서 확인할 수 있습니다.
