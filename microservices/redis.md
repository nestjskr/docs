### Redis

[Redis](https://redis.io/) 전송자는 게시/구독 메시징 패러다임을 구현하고 Redis의 [Pub/Sub](https://redis.io/topics/pubsub) 기능을 활용합니다. 게시된 메시지는 어떤 구독자(있는 경우)가 결국 메시지를 받을지 알지 못한 채 채널로 분류됩니다. 각 마이크로서비스는 원하는 수의 채널을 구독할 수 있습니다. 또한 한번에 두개 이상의 채널을 구독할 수 있습니다. 채널을 통해 교환되는 메시지는 **실행 후 삭제**입니다. 즉, 메시지가 게시되고 관심있는 구독자가 없으면 메시지가 제거되고 복구할 수 없습니다. 따라서 메시지 나 이벤트가 하나 이상의 서비스에서 처리된다는 보장이 없습니다. 여러 구독자가 단일 메시지를 구독(및 수신)할 수 있습니다.

<figure><img src="/assets/Redis_1.png" /></figure>

#### Installation

Redis 기반 마이크로서비스 빌드를 시작하려면 먼저 필요한 패키지를 설치하세요.

```bash
$ npm i --save redis
```

#### Overview

Redis 전송자를 사용하려면 다음 옵션 객체를 `createMicroservice()` 메서드에 전달합니다.

```typescript
@@filename(main)
const app = await NestFactory.createMicroservice<MicroserviceOptions>(AppModule, {
  transport: Transport.REDIS,
  options: {
    url: 'redis://localhost:6379',
  },
});
@@switch
const app = await NestFactory.createMicroservice(AppModule, {
  transport: Transport.REDIS,
  options: {
    url: 'redis://localhost:6379',
  },
});
```

> info **힌트** `Transport` 열거형은 `@nestjs/microservices` 패키지에서 가져옵니다.

#### Options

`options` 속성은 선택한 전송자에 따라 다릅니다. **Redis** 전송자는 아래 설명된 속성을 노출합니다.

<table>
  <tr>
    <td><code>url</code></td>
    <td>Connection url</td>
  </tr>
  <tr>
    <td><code>retryAttempts</code></td>
    <td>메시지 재시도 횟수(기본값: <code>0</code>)</td>
  </tr>
  <tr>
    <td><code>retryDelay</code></td>
    <td>메시지 재시도 간격(밀리 초)(기본값: <code>0</code>)</td>
  </tr>
</table>

공식 [redis](https://www.npmjs.com/package/redis#options-object-properties) 클라이언트에서 지원하는 모든 속성도 이 트랜스포터에서 지원됩니다.

#### Client

다른 마이크로서비스 전송자와 마찬가지로 Redis `ClientProxy` 인스턴스를 만들기 위한 [여러 옵션](/microservices/basics#client)이 있습니다.

인스턴스를 만드는 한가지 방법은 `ClientsModule`을 사용하는 것입니다. `ClientsModule`을 사용하여 클라이언트 인스턴스를 만들려면 가져와서 `register()` 메서드를 사용하여 위의 `createMicroservice()` 메서드에 표시된 것과 동일한 속성과 `name` 속성을 가진 옵션 객체를 전달합니다. 주입 토큰으로 사용됩니다. [여기](/microservices/basics#client)에서 `ClientsModule`에 대해 자세히 알아보세요.

```typescript
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'MATH_SERVICE',
        transport: Transport.REDIS,
        options: {
          url: 'redis://localhost:6379',
        }
      },
    ]),
  ]
  ...
})
```

클라이언트를 만드는 다른 옵션(`ClientProxyFactory` 또는 `@Client()`)도 사용할 수 있습니다. [여기](/microservices/basics#client)에서 이에 대해 읽을 수 있습니다.

#### Context

더 복잡한 시나리오에서는 들어오는 요청에 대한 추가 정보에 액세스할 수 있습니다. Redis 전송자를 사용하는 경우 `RedisContext` 객체에 액세스할 수 있습니다.

```typescript
@@filename()
@MessagePattern('notifications')
getNotifications(@Payload() data: number[], @Ctx() context: RedisContext) {
  console.log(`Channel: ${context.getChannel()}`);
}
@@switch
@Bind(Payload(), Ctx())
@MessagePattern('notifications')
getNotifications(data, context) {
  console.log(`Channel: ${context.getChannel()}`);
}
```

> info **힌트** `@Payload()`, `@Ctx()` 및 `RedisContext`는 `@nestjs/microservices` 패키지에서 가져옵니다.
