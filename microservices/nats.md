### NATS

[NATS](https://nats.io)는 클라우드 네이티브 애플리케이션, IoT 메시징, 마이크로서비스 아키텍처를 위한 간단하고 안전한 고성능 오픈소스 메시징 시스템입니다. NATS 서버는 Go 프로그래밍 언어로 작성되었지만 서버와 상호 작용하는 클라이언트 라이브러리는 수십개의 주요 프로그래밍 언어로 제공됩니다. NATS는 **최대 한번** 및 **최소 한번** 전송을 모두 지원합니다. 대형 서버 및 클라우드 인스턴스에서 에지 게이트웨이 및 사물 인터넷 장치를 통해 어디서나 실행할 수 있습니다.

#### Installation

NATS 기반 마이크로서비스 빌드를 시작하려면 먼저 필요한 패키지를 설치하세요.

```bash
$ npm i --save nats@^1.4.12
```

#### Overview

NATS 전송자를 사용하려면 다음 옵션 객체를 `createMicroservice()` 메서드에 전달합니다.

```typescript
@@filename(main)
const app = await NestFactory.createMicroservice<MicroserviceOptions>(AppModule, {
  transport: Transport.NATS,
  options: {
    url: 'nats://localhost:4222',
  },
});
@@switch
const app = await NestFactory.createMicroservice(AppModule, {
  transport: Transport.NATS,
  options: {
    url: 'nats://localhost:4222',
  },
});
```

> info **힌트** `Transport` 열거형은 `@nestjs/microservices` 패키지에서 가져옵니다.

#### Options

`options` 객체는 선택한 트랜스포터에 따라 다릅니다. **NATS** 전송기는 [여기](https://github.com/nats-io/node-nats#connect-options)에 설명된 속성을 노출합니다.
또한 서버가 구독해야하는 대기열의 이름을 지정할 수 있는 `queue` 속성이 있습니다(이 설정을 무시하려면` undefined`로 두십시오). [아래](/microservices/nats#queue-groups)에서 NATS 대기열 그룹에 대해 자세히 알아보세요.

#### Client

다른 마이크로 서비스전송기와 마찬가지로 NATS `ClientProxy` 인스턴스를 생성하는 [몇가지 옵션](/microservices/basics#client)이 있습니다.

인스턴스를 만드는 한가지 방법은 `ClientsModule`을 사용하는 것입니다. `ClientsModule`을 사용하여 클라이언트 인스턴스를 만들려면 가져 와서 `register()` 메서드를 사용하여 위의 `createMicroservice()` 메서드에 표시된 것과 동일한 속성과 `name` 속성을 가진 옵션 객체를 전달합니다. 주입 토큰으로 사용됩니다. [여기](/microservices/basics#client)에서 `ClientsModule`에 대해 자세히 알아보세요.

```typescript
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'MATH_SERVICE',
        transport: Transport.NATS,
        options: {
          url: 'nats://localhost:4222',
        }
      },
    ]),
  ]
  ...
})
```

클라이언트를 만드는 다른 옵션(`ClientProxyFactory` 또는 `@Client()`)도 사용할 수 있습니다. [여기](/microservices/basics#client)에서 이에 대해 읽을 수 있습니다.

#### Request-response

**요청-응답 request-response** 메시지 스타일 ([자세히 알아보기](/microservices/basics#request-response))의 경우 NATS 전송자는 NATS 기본 제공 [Request-Reply](https://docs.nats.io/nats-concepts/reqreply) 메커니즘을 사용합니다. 응답 주제가 있는 지정된 주제에 요청이 게시되고 응답자는 해당 주제를 듣고 응답 주제에 응답을 보냅니다. 응답 제목은 일반적으로 당사자의 위치에 관계없이 요청자에게 동적으로 다시 전달되는 `_INBOX`라는 제목입니다.

#### Event-based

**이벤트 기반** 메시지 스타일 ([자세히 알아보기](/microservices/basics#event-based))의 경우 NATS 전송자는 NATS 기본 제공 [Publish-Subscribe](https://docs.nats.io/nats-concepts/pubsub) 메커니즘을 사용합니다. 게시자는 주제에 대한 메시지를 보내고 해당 주제를 수신하는 활성 구독자는 메시지를받습니다. 구독자는 정규 표현식처럼 약간 작동하는 와일드카드 주제에 관심을 등록할 수도 있습니다. 이 일대다 패턴을 팬아웃이라고도 합니다.

#### Queue groups

NATS는 [분산 대기열](https://docs.nats.io/nats-concepts/queue)이라는 기본 제공 부하 분산 기능을 제공합니다. 대기열 구독을 생성하려면 다음과 같이 `queue` 속성을 사용합니다.

```typescript
@@filename(main)
const app = await NestFactory.createMicroservice(AppModule, {
  transport: Transport.NATS,
  options: {
    url: 'nats://localhost:4222',
    queue: 'cats_queue',
  },
});
```

#### Context

더 복잡한 시나리오에서는 들어오는 요청에 대한 추가 정보에 액세스할 수 있습니다. NATS 전송자를 사용하는 경우 `NatsContext` 객체에 액세스할 수 있습니다.

```typescript
@@filename()
@MessagePattern('notifications')
getNotifications(@Payload() data: number[], @Ctx() context: NatsContext) {
  console.log(`Subject: ${context.getSubject()}`);
}
@@switch
@Bind(Payload(), Ctx())
@MessagePattern('notifications')
getNotifications(data, context) {
  console.log(`Subject: ${context.getSubject()}`);
}
```

> info **힌트** `@Payload()`, `@Ctx()`및 `NatsContext`는 `@nestjs/microservices` 패키지에서 가져옵니다.

#### Wildcards

구독은 명시적 주제에 대한 것이거나 와일드카드를 포함할 수 있습니다.

```typescript
@@filename()
@MessagePattern('time.us.*')
getDate(@Payload() data: number[], @Ctx() context: NatsContext) {
  console.log(`Subject: ${context.getSubject()}`); // e.g. "time.us.east"
  return new Date().toLocaleTimeString(...);
}
@@switch
@Bind(Payload(), Ctx())
@MessagePattern('time.us.*')
getDate(data, context) {
  console.log(`Subject: ${context.getSubject()}`); // e.g. "time.us.east"
  return new Date().toLocaleTimeString(...);
}
```
