### MQTT

[MQTT](https://mqtt.org/)(Message Queuing Telemetry Transport)는 높은 지연 시간에 최적화된 오픈소스 경량 메시징 프로토콜입니다. 이 프로토콜은 **게시/구독** 모델을 사용하여 기기를 연결하는 확장 가능하고 비용 효율적인 방법을 제공합니다. MQTT에 구축된 통신 시스템은 게시 서버, 브로커 및 하나 이상의 클라이언트로 구성됩니다. 제한된 장치 및 낮은 대역폭, 높은 지연 시간 또는 불안정한 네트워크를 위해 설계되었습니다.

#### Installation

MQTT 기반 마이크로서비스 빌드를 시작하려면 먼저 필요한 패키지를 설치하세요.

```bash
$ npm i --save mqtt
```

#### Overview

MQTT 전송자를 사용하려면 다음 옵션 객체를 `createMicroservice()` 메서드에 전달합니다.

```typescript
@@filename(main)
const app = await NestFactory.createMicroservice<MicroserviceOptions>(AppModule, {
  transport: Transport.MQTT,
  options: {
    url: 'mqtt://localhost:1883',
  },
});
@@switch
const app = await NestFactory.createMicroservice(AppModule, {
  transport: Transport.MQTT,
  options: {
    url: 'mqtt://localhost:1883',
  },
});
```

> info **힌트** `Transport` 열거형은 `@nestjs/microservices` 패키지에서 가져옵니다.

#### Options

`options` 객체는 선택한 트랜스포터에 따라 다릅니다. **MQTT** 전송기는 [여기](https://github.com/mqttjs/MQTT.js/#mqttclientstreambuilder-options)에 설명된 속성을 노출합니다.

#### Client

다른 마이크로서비스 전송자와 마찬가지로 MQTT `ClientProxy` 인스턴스를 생성하기위한 [여러 옵션](/microservices/basics#client)이 있습니다.

인스턴스를 만드는 한가지 방법은 `ClientsModule`을 사용하는 것입니다. `ClientsModule`을 사용하여 클라이언트 인스턴스를 만들려면 가져와서 `register()` 메서드를 사용하여 위의 `createMicroservice()` 메서드에 표시된 것과 동일한 속성과 `name` 속성을 가진 옵션 객체를 전달합니다. 주입 토큰으로 사용됩니다. [여기](/microservices/basics#client)에서 `ClientsModule`에 대해 자세히 알아보세요.

```typescript
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'MATH_SERVICE',
        transport: Transport.MQTT,
        options: {
          url: 'mqtt://localhost:1883',
        }
      },
    ]),
  ]
  ...
})
```

클라이언트를 만드는 다른 옵션(`ClientProxyFactory` 또는 `@Client()`)도 사용할 수 있습니다. [여기](/microservices/basics#client)에서 이에 대해 읽을 수 있습니다.

#### Context

더 복잡한 시나리오에서는 들어오는 요청에 대한 추가 정보에 액세스할 수 있습니다. MQTT 전송자를 사용할 때 `MqttContext` 객체에 액세스할 수 있습니다.

```typescript
@@filename()
@MessagePattern('notifications')
getNotifications(@Payload() data: number[], @Ctx() context: MqttContext) {
  console.log(`Topic: ${context.getTopic()}`);
}
@@switch
@Bind(Payload(), Ctx())
@MessagePattern('notifications')
getNotifications(data, context) {
  console.log(`Topic: ${context.getTopic()}`);
}
```

> info **힌트** `@Payload()`, `@Ctx()` 및 `MqttContext`는 `@nestjs/microservices` 패키지에서 가져옵니다.

원본 mqtt [패킷](https://github.com/mqttjs/mqtt-packet)에 액세스하려면 다음과 같이 `MqttContext` 객체의 `getPacket()` 메서드를 사용하세요.

```typescript
@@filename()
@MessagePattern('notifications')
getNotifications(@Payload() data: number[], @Ctx() context: MqttContext) {
  console.log(context.getPacket());
}
@@switch
@Bind(Payload(), Ctx())
@MessagePattern('notifications')
getNotifications(data, context) {
  console.log(context.getPacket());
}
```

#### Wildcards

구독은 명시적 주제에 대한 것이거나 와일드카드를 포함할 수 있습니다. `+` 및 `#`의 두가지 와일드카드를 사용할 수 있습니다. `+`는 단일 레벨 와일드카드이고 `#`은 여러 주제 레벨을 포함하는 다중 레벨 와일드카드입니다.

```typescript
@@filename()
@MessagePattern('sensors/+/temperature/+')
getTemperature(@Ctx() context: MqttContext) {
  console.log(`Topic: ${context.getTopic()}`);
}
@@switch
@Bind(Ctx())
@MessagePattern('sensors/+/temperature/+')
getTemperature(context) {
  console.log(`Topic: ${context.getTopic()}`);
}
```
