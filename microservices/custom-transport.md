### Custom transporters

Nest는 개발자가 새로운 맞춤형 전송 전략을 구축할 수 있는 API뿐만 아니라 즉시 사용할 수 있는 다양한 **트랜스포터**를 제공합니다.
트랜스포터를 사용하면 플러그형 통신 계층과 매우 간단한 애플리케이션 수준 메시지 프로토콜을 사용하여 네트워크를 통해 구성요소를 연결할 수 있습니다 (전체 [기사](https://dev.to/nestjs/integrate-nestjs-with-external-services-using-microservice-transporters-part-1-p3) 읽기).

> info **힌트** Nest로 마이크로서비스를 구축한다고해서 반드시 `@nestjs/microservices` 패키지를 사용해야하는 것은 아닙니다. 예를 들어, 외부 서비스(다른 언어로 작성된 다른 마이크로서비스)와 통신하려는 경우 `@nestjs/microservice` 라이브러리에서 제공하는 모든 기능이 필요하지 않을 수 있습니다.
> 사실, 가입자를 선언적으로 정의할 수있는 데코레이터(`@EventPattern` 또는 `@MessagePattern`)가 필요하지 않은 경우 [Standalone Application](/application-context)을 실행하고 수동으로 채널에 대한 연결/구독을 유지해야합니다. 대부분의 사용 사례에 충분하며 더 많은 유연성을 제공합니다.

커스텀 전송자를 사용하면 모든 메시징 시스템/프로토콜(Google Cloud Pub/Sub, Amazon Kinesis등 포함)을 통합하거나 기존 항목을 확장하여 추가 기능(예: MQTT용 [QoS](https://github.com/mqttjs/MQTT.js/blob/master/README.md#qos))을 추가 할 수 있습니다.

> info **힌트** Nest 마이크로서비스의 작동 방식과 기존 전송기의 기능을 확장하는 방법을 더 잘 이해하려면 [NestJS Microservices in Action](https://dev.to/johnbiundo/series/4724) 및 [Advanced NestJS Microservices](https://dev.to/nestjs/part-1-introduction-and-setup-1a2l) 기사 시리즈를 읽어 보는 것이 좋습니다.

#### Creating a strategy

먼저 커스텀 트랜스포터를 나타내는 클래스를 정의하겠습니다.

```typescript
import { CustomTransportStrategy, Server } from '@nestjs/microservices';

class GoogleCloudPubSubServer
  extends Server
  implements CustomTransportStrategy {
  /**
   * This method is triggered when you run "app.listen()".
   */
  listen(callback: () => void) {
    callback();
  }

  /**
   * This method is triggered on application shutdown.
   */
  close() {}
}
```

> warning **경고** 이 장에서는 모든 기능을 갖춘 Google Cloud Pub/Sub 서버를 구현하지 않을 것입니다.이 경우 전송업체(transporter)별 기술 세부 정보를 살펴 봐야합니다.

위의 예에서는 `GoogleCloudPubSubServer` 클래스를 선언하고 `CustomTransportStrategy` 인터페이스에 의해 시행되는 `listen()` 및 `close()` 메소드를 제공했습니다.
또한 우리 클래스는 `@nestjs/microservices` 패키지에서 가져온 `Server` 클래스를 확장하여 몇가지 유용한 메서드를 제공합니다. 예를 들어 Nest 런타임에서 메시지 핸들러를 등록하는데 사용하는 메서드가 있습니다. 또는 기존 전송 전략의 기능을 확장하려는 경우 해당 서버 클래스(예: `ServerRedis`)를 확장할 수 있습니다.
일반적으로 메시지/이벤트 구독(필요한 경우 응답)을 담당하므로 클래스에 `"Server"` 접미사를 추가했습니다.

이를 통해 이제 다음과 같이 기본제공(built-in) 전송기를 사용하는 대신 사용자 지정 전략을 사용할 수 있습니다.

```typescript
const app = await NestFactory.createMicroservice<MicroserviceOptions>(
  AppModule,
  {
    strategy: new GoogleCloudPubSubServer(),
  },
);
```

기본적으로 `transport` 및 `options` 속성이 있는 일반 트랜스포터 옵션 객체를 전달하는 대신 값이 맞춤 트랜스포터 클래스의 인스턴스인 단일 속성인 `strategy`를 전달합니다.

`GoogleCloudPubSubServer` 클래스로 돌아가 실제 애플리케이션에서 메시지 브로커/외부 서비스에 대한 연결을 설정하고 `listen()` 메서드에서 구독자 등록/특정 채널 청취(그런 다음 구독 제거 및 종료 `close()` 해체 메서드의 연결), 하지만 이를 위해서는 Nest 마이크로서비스가 서로 통신하는 방식을 잘 이해해야 하므로 이 [문서 시리즈](https://dev.to/nestjs/part-1-introduction-and-setup-1a2l)를 읽는 것이 좋습니다. 대신 이 장에서는 `Server` 클래스가 제공하는 기능과 이를 활용하여 맞춤형 전략을 구축하는 방법에 초점을 맞출 것입니다.

예를 들어, 애플리케이션 어딘가에 다음 메시지 핸들러가 정의되어 있다고 가정해 보겠습니다.

```typescript
@MessagePattern('echo')
echo(@Payload() data: object) {
  return data;
}
```

이 메시지 핸들러는 Nest 런타임에 의해 자동으로 등록됩니다. `Server` 클래스를 이용하면 어떤 메시지 패턴이 등록되었는지 확인할 수 있으며, 할당된 실제 메소드에 접근하여 실행할 수 있습니다.
이것을 테스트하기 위해, `callback` 함수가 호출되기 전에 `listen()` 메소드 안에 간단한 `console.log`를 추가해 봅시다 :

```typescript
listen(callback: () => void) {
  console.log(this.messageHandlers);
  callback();
}
```

애플리케이션이 다시 시작되면 터미널에 다음 로그가 표시됩니다.

```typescript
Map { 'echo' => [AsyncFunction] { isEventHandler: false } }
```

> info **힌트** `@EventPattern` 데코레이터를 사용하면 동일한 출력이 표시되지만 `isEventHandler` 속성이 `true`로 설정되어 있습니다.

보시다시피 `messageHandlers` 속성은 패턴이 키로 사용되는 모든 메시지(및 이벤트) 핸들러의 `Map` 모음입니다.
이제 키(예: `"echo"`)를 사용하여 메시지 핸들러에 대한 참조를 받을 수 있습니다.

```typescript
async listen(callback: () => void) {
  const echoHandler = this.messageHandlers.get('echo');
  console.log(await echoHandler('Hello world!'));
  callback();
}
```

임의의 문자열을 인수로 전달하는 `echoHandler`를 실행하면(여기서는 `"Hello world!"`) 콘솔에서 확인할 수 있습니다.

```json
Hello world!
```

이는 우리의 메소드 핸들러가 제대로 실행되었음을 의미합니다.

#### Client proxy

첫번째 섹션에서 언급했듯이 반드시 `@nestjs/microservices` 패키지를 사용하여 마이크로서비스를 생성할 필요는 없지만 그렇게 하기로 결정하고 사용자 지정 전략을 통합해야하는 경우 다음을 제공해야합니다. "클라이언트" 클래스도 마찬가지입니다.

> info **힌트** 다시 말하지만, 모든 `@nestjs/microservices` 기능 (예: 스트리밍)과 호환되는 모든 기능을 갖춘 클라이언트 클래스를 구현하려면 프레임워크에서 사용하는 통신 기술을 잘 이해해야합니다. 자세한 내용은 이 [기사](https://dev.to/nestjs/part-4-basic-client-component-16f9)를 확인하십시오.

외부 서비스/발신 및 메시지(또는 이벤트)와 통신하려면 라이브러리별 SDK 패키지를 사용하거나 다음과 같이 `ClientProxy`를 확장하는 사용자 지정 클라이언트 클래스를 구현할 수 있습니다.

```typescript
import { ClientProxy, ReadPacket, WritePacket } from '@nestjs/microservices';

class GoogleCloudPubSubClient extends ClientProxy {
  async connect(): Promise<any> {}
  async close() {}
  async dispatchEvent(packet: ReadPacket<any>): Promise<any> {}
  publish(
    packet: ReadPacket<any>,
    callback: (packet: WritePacket<any>) => void,
  ): Function {}
}
```

> warning **경고** 이 장에서는 모든 기능을 갖춘 Google Cloud Pub/Sub 클라이언트를 구현하지 않을 것입니다.이 경우 전송 업체별 기술 세부 정보를 살펴봐야합니다.

보시다시피 `ClientProxy` 클래스는 연결을 설정 및 종료하고 메시지(`publish`)와 이벤트(`dispatchEvent`)를 게시하기 위한 여러 메서드를 제공해야합니다.
요청-응답 통신 스타일 지원이 필요하지 않은 경우 `publish()` 메서드를 비워둘 수 있습니다. 마찬가지로 이벤트 기반 통신을 지원할 필요가 없으면 `dispatchEvent()` 메서드를 건너뜁니다.

이러한 메소드가 실행되는 내용과 시기를 관찰하기 위해 다음과 같이 여러 `console.log` 호출을 추가해 보겠습니다.

```typescript
class GoogleCloudPubSubClient extends ClientProxy {
  async connect(): Promise<any> {
    console.log('connect');
  }

  async close() {
    console.log('close');
  }

  async dispatchEvent(packet: ReadPacket<any>): Promise<any> {
    return console.log('event to dispatch: ', packet);
  }

  publish(
    packet: ReadPacket<any>,
    callback: (packet: WritePacket<any>) => void,
  ): Function {
    console.log('message:', packet);

    // In a real-world application, the "callback" function should be executed
    // with payload sent back from the responder. Here, we'll simply simulate (5 seconds delay)
    // that response came through by passing the same "data" as we've originally passed in.
    setTimeout(() => callback({ response: packet.data }), 5000);

    return () => console.log('teardown');
  }
}
```

이를 통해 `GoogleCloudPubSubClient` 클래스의 인스턴스를 만들고 반환된 관찰 가능한 스트림을 구독하여 `send()` 메서드(이전 장에서 보셨을 것임)를 실행해 보겠습니다.


```typescript
const googlePubSubClient = new GoogleCloudPubSubClient();
googlePubSubClient
  .send('pattern', 'Hello world!')
  .subscribe((response) => console.log(response));
```

이제 터미널에 다음 출력이 표시됩니다.

```typescript
connect
message: { pattern: 'pattern', data: 'Hello world!' }
Hello world! // <-- after 5 seconds
```

"teardown" 메서드(`publish()` 메서드가 반환하는)가 제대로 실행되는지 테스트하기 위해 스트림에 타임아웃 연산자를 적용하고 `setTimeout`이 `callback` 함수를 호출하는 것보다 일찍 발생하는지 확인하기 위해 2초로 설정합니다. .

```typescript
const googlePubSubClient = new GoogleCloudPubSubClient();
googlePubSubClient
  .send('pattern', 'Hello world!')
  .pipe(timeout(2000))
  .subscribe(
    (response) => console.log(response),
    (error) => console.error(error.message),
  );
```

> info **힌트** `timeout` 연산자는 `rxjs/operators` 패키지에서 가져옵니다.

`timeout` 연산자가 적용된 상태에서 터미널 출력은 다음과 같아야합니다.

```typescript
connect
message: { pattern: 'pattern', data: 'Hello world!' }
teardown // <-- teardown
Timeout has occurred
```

이벤트를 전달하려면(메시지를 보내는 대신) `emit()` 메서드를 사용합니다.

```typescript
googlePubSubClient.emit('event', 'Hello world!');
```

그리고 이것이 콘솔에 표시되어야하는 것입니다.

```typescript
connect
event to dispatch:  { pattern: 'event', data: 'Hello world!' }
```
