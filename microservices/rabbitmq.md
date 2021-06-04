### RabbitMQ

[RabbitMQ](https://www.rabbitmq.com/)는 여러 메시징 프로토콜을 지원하는 오픈소스 경량 메시지 브로커입니다. 대규모 고 가용성 요구사항을 충족하기 위해 분산 및 연합 구성으로 배포할 수 있습니다. 또한 전세계적으로 소규모 스타트업 및 대기업에서 사용되는 가장 널리 배포된 메시지 브로커입니다.

#### Installation

RabbitMQ 기반 마이크로서비스 빌드를 시작하려면 먼저 필요한 패키지를 설치합니다.

```bash
$ npm i --save amqplib amqp-connection-manager
```

#### Overview

RabbitMQ 트랜스포터를 사용하려면 다음 옵션 객체를 `createMicroservice()` 메서드에 전달합니다.

```typescript
@@filename(main)
const app = await NestFactory.createMicroservice<MicroserviceOptions>(AppModule, {
  transport: Transport.RMQ,
  options: {
    urls: ['amqp://localhost:5672'],
    queue: 'cats_queue',
    queueOptions: {
      durable: false
    },
  },
});
@@switch
const app = await NestFactory.createMicroservice(AppModule, {
  transport: Transport.RMQ,
  options: {
    urls: ['amqp://localhost:5672'],
    queue: 'cats_queue',
    queueOptions: {
      durable: false
    },
  },
});
```

> info **힌트** `Transport` 열거형은 `@nestjs/microservices` 패키지에서 가져옵니다.

#### Options

`options` 속성은 선택한 전송자에 따라 다릅니다. **RabbitMQ** 전송자는 아래 설명된 속성을 노출합니다.

<table>
  <tr>
    <td><code>urls</code></td>
    <td>Connection urls</td>
  </tr>
  <tr>
    <td><code>queue</code></td>
    <td>서버가 수신할 대기열 이름</td>
  </tr>
  <tr>
    <td><code>prefetchCount</code></td>
    <td>채널의 미리 가져오기 수를 설정합니다.</td>
  </tr>
  <tr>
    <td><code>isGlobalPrefetchCount</code></td>
    <td>채널당 미리 가져오기 사용</td>
  </tr>
  <tr>
    <td><code>noAck</code></td>
    <td><code>false</code>인 경우 수동 확인 모드가 활성화됩니다.</td>
  </tr>
  <tr>
    <td><code>queueOptions</code></td>
    <td>추가 대기열 옵션 (<a href="https://www.squaremobius.net/amqp.node/channel_api.html#channel_assertQueue" rel="nofollow" target="_blank">여기</a> 참조)</td>
  </tr>
  <tr>
    <td><code>socketOptions</code></td>
    <td>추가 소켓 옵션 (<a href="https://www.squaremobius.net/amqp.node/channel_api.html#socket-options" rel="nofollow" target="_blank"> 여기 </a> 참조)</td>
  </tr>
</table>

#### Client

다른 마이크로서비스 전송자와 마찬가지로 RabbitMQ `ClientProxy` 인스턴스를 생성하는 [몇가지 옵션](/microservices/basics#client)이 있습니다.

인스턴스를 만드는 한가지 방법은 `ClientsModule`을 사용하는 것입니다. `ClientsModule`을 사용하여 클라이언트 인스턴스를 만들려면 가져 와서 `register()` 메서드를 사용하여 위의 `createMicroservice()` 메서드에 표시된 것과 동일한 속성과 `name` 속성을 가진 옵션 객체를 전달합니다. 주입 토큰으로 사용됩니다. [여기](/microservices/basics#client)에서 `ClientsModule`에 대해 자세히 알아보세요.

```typescript
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'MATH_SERVICE',
        transport: Transport.RMQ,
        options: {
          urls: ['amqp://localhost:5672'],
          queue: 'cats_queue',
          queueOptions: {
            durable: false
          },
        },
      },
    ]),
  ]
  ...
})
```

클라이언트를 만드는 다른 옵션(`ClientProxyFactory` 또는 `@Client()`)도 사용할 수 있습니다. [여기](/microservices/basics#client)에서 이에 대해 읽을 수 있습니다.

#### Context

더 복잡한 시나리오에서는 들어오는 요청에 대한 추가 정보에 액세스할 수 있습니다. RabbitMQ 트랜스포터를 사용할 때 `RmqContext` 객체에 액세스할 수 있습니다.

```typescript
@@filename()
@MessagePattern('notifications')
getNotifications(@Payload() data: number[], @Ctx() context: RmqContext) {
  console.log(`Pattern: ${context.getPattern()}`);
}
@@switch
@Bind(Payload(), Ctx())
@MessagePattern('notifications')
getNotifications(data, context) {
  console.log(`Pattern: ${context.getPattern()}`);
}
```

> info **힌트** `@Payload()`, `@Ctx()` 및 `RmqContext`는 `@nestjs/microservices` 패키지에서 가져옵니다.

원본 RabbitMQ 메시지(`properties`, `fields` 및 `content` 포함)에 액세스하려면 다음과 같이 `RmqContext` 객체의 `getMessage()` 메서드를 사용합니다.

```typescript
@@filename()
@MessagePattern('notifications')
getNotifications(@Payload() data: number[], @Ctx() context: RmqContext) {
  console.log(context.getMessage());
}
@@switch
@Bind(Payload(), Ctx())
@MessagePattern('notifications')
getNotifications(data, context) {
  console.log(context.getMessage());
}
```

RabbitMQ [channel](https://www.rabbitmq.com/channels.html)에 대한 참조를 검색하려면 다음과 같이 `RmqContext` 객체의 `getChannelRef` 메소드를 사용하세요.

```typescript
@@filename()
@MessagePattern('notifications')
getNotifications(@Payload() data: number[], @Ctx() context: RmqContext) {
  console.log(context.getChannelRef());
}
@@switch
@Bind(Payload(), Ctx())
@MessagePattern('notifications')
getNotifications(data, context) {
  console.log(context.getChannelRef());
}
```

#### Message acknowledgement

메시지가 손실되지 않도록 RabbitMQ는 [메시지 확인](https://www.rabbitmq.com/confirms.html)을 지원합니다. 특정 메시지가 수신, 처리되었으며 RabbitMQ가 자유롭게 삭제할 수 있음을 RabbitMQ에 알리기 위해 소비자가 확인 응답을 보냅니다. 소비자가 ack를 보내지 않고 죽으면(채널이 닫히거나, 연결이 닫히거나, TCP 연결이 끊어지면) RabbitMQ는 메시지가 완전히 처리되지 않았음을 인식하고 다시 대기열에 넣습니다.

수동 확인 모드를 활성화하려면 `noAck` 속성을 `false`로 설정합니다.

```typescript
options: {
  urls: ['amqp://localhost:5672'],
  queue: 'cats_queue',
  noAck: false,
  queueOptions: {
    durable: false
  },
},
```

수동 소비자 승인이 켜져 있으면 작업이 완료되었음을 알리기 위해 작업자로부터 적절한 승인을 보내야합니다.

```typescript
@@filename()
@MessagePattern('notifications')
getNotifications(@Payload() data: number[], @Ctx() context: RmqContext) {
  const channel = context.getChannelRef();
  const originalMsg = context.getMessage();

  channel.ack(originalMsg);
}
@@switch
@Bind(Payload(), Ctx())
@MessagePattern('notifications')
getNotifications(data, context) {
  const channel = context.getChannelRef();
  const originalMsg = context.getMessage();

  channel.ack(originalMsg);
}
```
