### Kafka

[Kafka](https://kafka.apache.org/)는 다음과 같은 세가지 주요 기능을 갖춘 오픈소스 분산 스트리밍 플랫폼입니다.

- 메시지 대기열 또는 엔터프라이즈 메시징 시스템과 유사한 레코드 스트림을 게시하고 구독합니다.
- 내결함성(fault-tolerant) 지속 방식으로 레코드 스트림을 저장합니다.
- 레코드 스트림이 발생하면 처리합니다.

Kafka 프로젝트는 실시간 데이터 피드를 처리하기 위해 처리량이 높고 지연 시간이 짧은 통합 플랫폼을 제공하는 것을 목표로합니다. 실시간 스트리밍 데이터 분석을 위해 Apache Storm 및 Spark와 매우 잘 통합됩니다.

**Kafka 전송자는 실험적입니다.**

#### Installation

Kafka 기반 마이크로서비스 빌드를 시작하려면 먼저 필요한 패키지를 설치하세요.

```bash
$ npm i --save kafkajs
```

#### Overview

다른 Nest 마이크로서비스 전송 계층 구현과 마찬가지로, 아래와 같이 `createMicroservice()` 메서드에 전달된 옵션 객체의 `transport` 속성과 함께 선택적 `options` 속성을 사용하여 Kafka 전송자 메커니즘을 선택합니다.

```typescript
@@filename(main)
const app = await NestFactory.createMicroservice<MicroserviceOptions>(AppModule, {
  transport: Transport.KAFKA,
  options: {
    client: {
      brokers: ['localhost:9092'],
    }
  }
});
@@switch
const app = await NestFactory.createMicroservice(AppModule, {
  transport: Transport.KAFKA,
  options: {
    client: {
      brokers: ['localhost:9092'],
    }
  }
});
```

> info **힌트** `Transport` 열거형은 `@nestjs/microservices` 패키지에서 가져옵니다.

#### Options

`options` 속성은 선택한 전송자에 따라 다릅니다. **Kafka** 전송자는 아래 설명된 속성을 노출합니다.

<table>
  <tr>
    <td><code>client</code></td>
    <td>클라이언트 구성 옵션(
      <a
        href="https://kafka.js.org/docs/configuration"
        rel="nofollow"
        target="blank"
        >여기</a
      >에서 자세히 알아보기)</td>
  </tr>
  <tr>
    <td><code>consumer</code></td>
    <td>소비자 구성 옵션(
      <a
        href="https://kafka.js.org/docs/consuming#a-name-options-a-options"
        rel="nofollow"
        target="blank"
        >여기</a
      >에서 자세히 알아보기)</td>
  </tr>
  <tr>
    <td><code>run</code></td>
    <td>실행 구성 옵션(
      <a
        href="https://kafka.js.org/docs/consuming"
        rel="nofollow"
        target="blank"
        >여기</a
      >에서 자세히 알아보기)</td>
  </tr>
  <tr>
    <td><code>subscribe</code></td>
    <td>구독 구성 옵션(
      <a
        href="https://kafka.js.org/docs/consuming#frombeginning"
        rel="nofollow"
        target="blank"
        >여기</a
      >에서 자세히 알아보기)</td>
  </tr>
  <tr>
    <td><code>producer</code></td>
    <td>생산자 구성 옵션(
      <a
        href="https://kafka.js.org/docs/producing#options"
        rel="nofollow"
        target="blank"
        >여기</a
      >에서 자세히 알아보기)</td>
  </tr>
  <tr>
    <td><code>send</code></td>
    <td>보내기 구성 옵션(read more
      <a
        href="https://kafka.js.org/docs/producing#options"
        rel="nofollow"
        target="blank"
        >여기</a
      >에서 자세히 알아보기)</td>
  </tr>
</table>

#### Client

Kafka는 다른 마이크로서비스 전송자와 비교했을 때 약간의 차이가 있습니다. `ClientProxy` 클래스 대신 `ClientKafka` 클래스를 사용합니다.

다른 마이크로서비스 전송자와 마찬가지로 `ClientKafka` 인스턴스를 만들기위한 [여러 옵션](/microservices/basics#client)이 있습니다.

인스턴스를 만드는 한가지 방법은 `ClientsModule`을 사용하는 것입니다. `ClientsModule`을 사용하여 클라이언트 인스턴스를 만들려면 가져 와서 `register()` 메서드를 사용하여 위의 `createMicroservice()` 메서드에 표시된 것과 동일한 속성과 `name` 속성을 가진 옵션 객체를 전달합니다. 주입 토큰으로 사용됩니다. [여기](/microservices/basics#client)에서 `ClientsModule`에 대해 자세히 알아보세요.

```typescript
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'HERO_SERVICE',
        transport: Transport.KAFKA,
        options: {
          client: {
            clientId: 'hero',
            brokers: ['localhost:9092'],
          },
          consumer: {
            groupId: 'hero-consumer'
          }
        }
      },
    ]),
  ]
  ...
})
```


클라이언트를 만드는 다른 옵션(`ClientProxyFactory` 또는 `@Client()`)도 사용할 수 있습니다. [여기](/microservices/basics#client)에서 이에 대해 읽을 수 있습니다.

다음과 같이 `@Client()` 데코레이터를 사용하십시오.

```typescript
@Client({
  transport: Transport.KAFKA,
  options: {
    client: {
      clientId: 'hero',
      brokers: ['localhost:9092'],
    },
    consumer: {
      groupId: 'hero-consumer'
    }
  }
})
client: ClientKafka;
```

#### Message response subscription

`ClientKafka` 클래스는 `subscribeToResponseOf()` 메소드를 제공합니다. `subscribeToResponseOf()` 메소드는 요청의 주제 이름을 인수로 취하고 파생된 응답 주제 이름을 응답 주제 모음에 추가합니다. 이 메소드는 메시지 패턴을 구현할 때 필요합니다.

```typescript
@@filename(heroes.controller)
onModuleInit() {
  this.client.subscribeToResponseOf('hero.kill.dragon');
}
```

`ClientKafka` 인스턴스가 비동기적으로 생성되는 경우 `connect()` 메서드를 호출하기 전에 `subscribeToResponseOf()` 메서드를 호출해야합니다.

```typescript
@@filename(heroes.controller)
async onModuleInit() {
  this.client.subscribeToResponseOf('hero.kill.dragon');
  await this.client.connect();
}
```

#### Message pattern

Kafka 마이크로서비스 메시지 패턴은 요청 및 응답 채널에 대해 두가지 주제를 활용합니다. `ClientKafka#send()`메서드는 [상관 ID correlation id](https://www.enterpriseintegrationpatterns.com/patterns/messaging/CorrelationIdentifier.html), 응답 주제, 응답 파티션을 요청 메시지와 연결하여 [반환 주소 return address](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ReturnAddress.html)로 메시지를 보냅니다. 이를 위해서는 `ClientKafka` 인스턴스가 응답 주제를 구독하고 메시지를 보내기 전에 하나 이상의 파티션에 할당되어야합니다.

따라서 실행중인 모든 Nest 애플리케이션에 대해 하나 이상의 응답 주제 파티션이 있어야합니다. 예를 들어 4개의 Nest 애플리케이션을 실행중이지만 응답 항목에 3개의 파티션만 있는 경우 Nest 애플리케이션중 하나는 메시지를 보내려고할 때 오류가 발생합니다.

새로운 `ClientKafka` 인스턴스가 시작되면 소비자 그룹에 가입하고 해당 주제를 구독합니다. 이 프로세스는 소비자 그룹의 소비자에게 할당된 토픽 파티션의 재조정을 트리거합니다.

일반적으로 토픽 파티션은 라운드 로빈 파티셔너를 사용하여 할당됩니다.이 파티셔너는 애플리케이션 실행시 무작위로 설정된 소비자 이름별로 정렬된 소비자 컬렉션에 토픽 파티션을 할당합니다. 그러나 새 소비자가 소비자 그룹에 가입하면 새 소비자는 소비자 컬렉션내의 어느 위치 에나 배치될 수 있습니다. 이렇게하면 기존 소비자가 새 소비자 다음에 배치될 때 기존 소비자에게 다른 파티션을 할당할 수 있는 조건이 생성됩니다. 결과적으로 서로 다른 파티션이 할당된 소비자는 재조정 이전에 전송된 요청의 응답 메시지를 잃게됩니다.

`ClientKafka` 소비자가 응답 메시지를 잃지 않도록하기 위해 Nest 전용 내장 사용자 지정 파티셔너가 사용됩니다. 이 사용자 지정 파티셔너는 애플리케이션 시작시 설정된 고해상도 타임 스탬프(`process.hrtime()`)별로 정렬된 소비자 모음에 파티션을 할당합니다.

#### Incoming

Nest는 `Buffer` 타입의 값이 있는 `key`, `value`, `headers` 속성이 있는 객체로 수신 Kafka 메시지를 수신합니다. 그런 다음 Nest는 버퍼를 문자열로 변환하여 이러한 값을 구문분석합니다. 문자열이 "객체 유사"인 경우 Nest는 문자열을 `JSON`으로 파싱합니다. 그런 다음 `value`가 관련 핸들러에 전달됩니다.

#### Outgoing

Nest는 이벤트를 게시하거나 메시지를 보낼 때 직렬화 프로세스 후에 나가는 Kafka 메시지를 보냅니다. 이는 `ClientKafka` `emit()` 및 `send()` 메서드에 전달된 인수 또는 `@MessagePattern` 메서드에서 반환된 값에서 발생합니다. 이 직렬화는 `JSON.stringify()` 또는 `toString()` 프로토타입 메소드를 사용하여 문자열이나 버퍼가 아닌 객체를 "문자열화"합니다.

```typescript
@@filename(heroes.controller)
@Controller()
export class HeroesController {
  @MessagePattern('hero.kill.dragon')
  killDragon(@Payload() message: KillDragonMessage): any {
    const dragonId = message.dragonId;
    const items = [
      { id: 1, name: 'Mythical Sword' },
      { id: 2, name: 'Key to Dungeon' },
    ];
    return items;
  }
}
```

> info **힌트** `@Payload()`는 `@nestjs/microservices`에서 가져옵니다.

`key` 및 `value` 속성이 있는 객체를 전달하여 발신 메시지에 키를 지정할 수도 있습니다. 키잉 메시지는[공동 파티션 요구 사항](https://docs.confluent.io/current/ksql/docs/developer-guide/partition-data.html#co-partitioning-requirements)을 충족하는 데 중요합니다.

```typescript
@@filename(heroes.controller)
@Controller()
export class HeroesController {
  @MessagePattern('hero.kill.dragon')
  killDragon(@Payload() message: KillDragonMessage): any {
    const realm = 'Nest';
    const heroId = message.heroId;
    const dragonId = message.dragonId;

    const items = [
      { id: 1, name: 'Mythical Sword' },
      { id: 2, name: 'Key to Dungeon' },
    ];

    return {
      headers: {
        realm
      },
      key: heroId,
      value: items
    }
  }
}
```

또한 이 형식으로 전달된 메시지는 `headers` 해시 속성에 설정된 사용자 지정 헤더를 포함할 수도 있습니다. 헤더 해시 속성 값은 `string` 타입 또는 `Buffer` 타입이어야합니다.

```typescript
@@filename(heroes.controller)
@Controller()
export class HeroesController {
  @MessagePattern('hero.kill.dragon')
  killDragon(@Payload() message: KillDragonMessage): any {
    const realm = 'Nest';
    const heroId = message.heroId;
    const dragonId = message.dragonId;

    const items = [
      { id: 1, name: 'Mythical Sword' },
      { id: 2, name: 'Key to Dungeon' },
    ];

    return {
      headers: {
        kafka_nestRealm: realm
      },
      key: heroId,
      value: items
    }
  }
}
```

#### Context

더 복잡한 시나리오에서는 들어오는 요청에 대한 추가 정보에 액세스할 수 있습니다. Kafka 전송자를 사용하는 경우 `KafkaContext` 객체에 액세스할 수 있습니다.

```typescript
@@filename()
@MessagePattern('hero.kill.dragon')
killDragon(@Payload() message: KillDragonMessage, @Ctx() context: KafkaContext) {
  console.log(`Topic: ${context.getTopic()}`);
}
@@switch
@Bind(Payload(), Ctx())
@MessagePattern('hero.kill.dragon')
killDragon(message, context) {
  console.log(`Topic: ${context.getTopic()}`);
}
```

> info **힌트** `@Payload()`, `@Ctx()` 및 `KafkaContext`는 `@nestjs/microservices` 패키지에서 가져옵니다.

원래 Kafka `IncomingMessage` 객체에 액세스하려면 다음과 같이 `KafkaContext` 객체의 `getMessage()` 메서드를 사용합니다.

```typescript
@@filename()
@MessagePattern('hero.kill.dragon')
killDragon(@Payload() message: KillDragonMessage, @Ctx() context: KafkaContext) {
  const originalMessage = context.getMessage();
  const { headers, partition, timestamp } = originalMessage;
}
@@switch
@Bind(Payload(), Ctx())
@MessagePattern('hero.kill.dragon')
killDragon(message, context) {
  const originalMessage = context.getMessage();
  const { headers, partition, timestamp } = originalMessage;
}
```

`IncomingMessage`가 다음 인터페이스를 충족하는 곳:

```typescript
interface IncomingMessage {
  topic: string;
  partition: number;
  timestamp: string;
  size: number;
  attributes: number;
  offset: string;
  key: any;
  value: any;
  headers: Record<string, any>;
}
```

#### Naming conventions

Kafka 마이크로서비스 구성요소는 각각의 역할에 대한 설명을 `client.clientId` 및 `consumer.groupId` 옵션에 추가하여 Nest 마이크로서비스 클라이언트와 서버 구성요소 간의 충돌을 방지합니다. 기본적으로 `ClientKafka` 구성요소는 `-client`를 추가하고 `ServerKafka` 구성요소는 이 두옵션에 `-server`를 추가합니다. 아래 제공된 값이 이러한 방식으로 어떻게 변환되는지 확인하십시오 (주석에 표시됨).

```typescript
@@filename(main)
const app = await NestFactory.createMicroservice(AppModule, {
  transport: Transport.KAFKA,
  options: {
    client: {
      clientId: 'hero', // hero-server
      brokers: ['localhost:9092'],
    },
    consumer: {
      groupId: 'hero-consumer' // hero-consumer-server
    },
  }
});
```

그리고 클라이언트의 경우:

```typescript
@@filename(heroes.controller)
@Client({
  transport: Transport.KAFKA,
  options: {
    client: {
      clientId: 'hero', // hero-client
      brokers: ['localhost:9092'],
    },
    consumer: {
      groupId: 'hero-consumer' // hero-consumer-client
    }
  }
})
client: ClientKafka;
```

> info **힌트** Kafka 클라이언트 및 소비자 명명 규칙은 사용자 지정 공급자에서 `ClientKafka` 및 `KafkaServer`를 확장하고 생성자를 재정의하여 사용자 지정할 수 있습니다.

Kafka 마이크로서비스 메시지 패턴은 요청 및 응답 채널에 대해 두가지 주제를 사용하므로 응답 패턴은 요청 주제에서 파생되어야합니다. 기본적으로 응답 주제의 이름은 `.reply`가 추가된 요청 주제 이름의 합성입니다.

```typescript
@@filename(heroes.controller)
onModuleInit() {
  this.client.subscribeToResponseOf('hero.get'); // hero.get.reply
}
```

> info **힌트** Kafka 응답 주제 명명 규칙은 사용자 지정 공급자에서 `ClientKafka`를 확장하고 `getResponsePatternName` 메서드를 재정의하여 사용자 지정할 수 있습니다.
