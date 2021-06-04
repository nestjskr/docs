### Gateways

종속성 주입, 데코레이터, 예외 필터, 파이프, 가드 및 인터셉터와 같이 이 문서의 다른 부분에서 논의된 대부분의 개념은 게이트웨이에 동일하게 적용됩니다. 가능한 경우 Nest는 HTTP 기반 플랫폼, WebSocket 및 마이크로서비스에서 동일한 구성요소를 실행할 수 있도록 구현 세부 정보를 추상화합니다. 이 섹션에서는 WebSocket과 관련된 Nest의 측면을 다룹니다.

Nest에서 게이트웨이는 단순히 `@WebSocketGateway()` 데코레이터로 주석이 달린 클래스입니다. 기술적으로 게이트웨이는 플랫폼에 구애받지 않으므로 어댑터가 생성되면 모든 WebSocket 라이브러리와 호환됩니다. 기본적으로 지원되는 두가지 WS 플랫폼이 있습니다: [socket.io](https://github.com/socketio/socket.io) 및 [ws](https://github.com/websockets/ws ). 귀하의 요구에 가장 적합한 것을 선택할 수 있습니다. 또한 이 [가이드](/websockets/adapter)를 따라 자체 어댑터를 구축할 수 있습니다.

<figure><img src="/assets/Gateways_1.png" /></figure>

> info **힌트** 게이트웨이는 [프로바이더](/providers)로 취급할 수 있습니다; 이는 클래스 생성자를 통해 종속성을 주입할 수 있음을 의미합니다. 또한 다른 클래스(프로바이더 및 컨트롤러)에서도 게이트웨이를 삽입할 수 있습니다.

#### Installation

WebSockets 기반 애플리케이션 빌드를 시작하려면 먼저 필요한 패키지를 설치하십시오.

```bash
@@filename()
$ npm i --save @nestjs/websockets @nestjs/platform-socket.io
$ npm i --save-dev @types/socket.io
@@switch
$ npm i --save @nestjs/websockets @nestjs/platform-socket.io
```

> 경고 **경고** `@nestjs/platform-socket.io`는 현재 socket.io v2.3에 의존하며 socket.io v3.0 클라이언트와 서버는 이전 버전과 호환되지 않습니다. 그러나 socket.io v3.0을 사용하도록 사용자 정의 어댑터를 구현할 수 있습니다. 자세한 내용은 [이번 호](https://github.com/nestjs/nest/issues/5676)를 참조하세요.

#### Overview

일반적으로 앱이 웹 애플리케이션이 아니거나 포트를 수동으로 변경한 경우를 제외하고 각 게이트웨이는 **HTTP 서버**와 동일한 포트에서 수신 대기합니다. 이 기본 동작은 `@WebSocketGateway(80)` 데코레이터에 인수를 전달하여 수정할 수 있습니다. 여기서 `80`은 선택된 포트번호입니다. 다음 구성을 사용하여 게이트웨이에서 사용하는 [네임 스페이스](https://socket.io/docs/rooms-and-namespaces/)를 설정할 수도 있습니다.

```typescript
@WebSocketGateway(80, { namespace: 'events' })
```

> warning **경고** 게이트웨이는 기존 모듈의 프로바이더 배열에서 참조될 때까지 인스턴스화되지 않습니다.

지원되는 모든 [option](https://socket.io/docs/server-api/)을 소켓 생성자에 전달할 수 있으며 두번째 인수는 `@WebSocketGateway()` 데코레이터에 다음과 같이 전달할 수 있습니다.

```typescript
@WebSocketGateway(81, { transports: ['websocket'] })
```

게이트웨이는 현재 수신중이지만 아직 수신 메시지를 구독하지 않았습니다. `events` 메시지를 구독하고 똑같은 데이터로 사용자에게 응답하는 핸들러를 만들어 보겠습니다.

```typescript
@@filename(events.gateway)
@SubscribeMessage('events')
handleEvent(@MessageBody() data: string): string {
  return data;
}
@@switch
@Bind(MessageBody())
@SubscribeMessage('events')
handleEvent(data) {
  return data;
}
```

> info **힌트** `@SubscribeMessage()` 및 `@MessageBody()` 데코레이터는 `@nestjs/websockets` 패키지에서 가져옵니다.

데코레이터를 사용하지 않으려면 다음 코드는 기능적으로 동일합니다.

```typescript
@@filename(events.gateway)
@SubscribeMessage('events')
handleEvent(client: Socket, data: string): string {
  return data;
}
@@switch
@SubscribeMessage('events')
handleEvent(client, data) {
  return data;
}
```

위의 예에서 `handleEvent()` 함수는 두개의 인수를 사용합니다. 첫번째는 플랫폼별 [소켓 인스턴스](https://socket.io/docs/server-api/#socket)이고 두번째는 클라이언트에서 수신한 데이터입니다. 하지만이 방법은 각 단위 테스트에서 `socket` 인스턴스를 모의해야하므로 권장되지 않습니다.

`events` 메시지가 수신되면 핸들러는 네트워크를 통해 전송된 것과 동일한 데이터를 사용하여 승인을 보냅니다. 또한 예를 들어 `client.emit()` 메서드를 사용하여 라이브러리별 접근방식을 사용하여 메시지를 내보낼 수 있습니다. 연결된 소켓 인스턴스에 액세스하려면 `@ConnectedSocket()` 데코레이터를 사용하십시오.

```typescript
@@filename(events.gateway)
@SubscribeMessage('events')
handleEvent(
  @MessageBody() data: string,
  @ConnectedSocket() client: Socket,
): string {
  return data;
}
@@switch
@Bind(MessageBody(), ConnectedSocket())
@SubscribeMessage('events')
handleEvent(data, client) {
  return data;
}
```

> info **힌트** `@ConnectedSocket()` 데코레이터는 `@nestjs/websockets` 패키지에서 가져옵니다.

그러나 이 경우 인터셉터를 활용할 수 없습니다. 사용자에게 응답하지 않으려면 `return`문을 건너뛸 수 있습니다(또는 명시적으로 "허위 falsy" 값을 반환합니다 (예: `undefined`)).

이제 클라이언트가 다음과 같이 메시지를 내보냅니다.

```typescript
socket.emit('events', { name: 'Nest' });
```

`handleEvent()` 메소드가 실행됩니다. 위의 핸들러내에서 발신된 메시지를 수신하려면 클라이언트가 해당 승인 리스너를 연결해야합니다.

```typescript
socket.emit('events', { name: 'Nest' }, data => console.log(data));
```

#### Multiple responses

승인은 한번만 발송됩니다. 또한 네이티브 WebSockets 구현에서는 지원되지 않습니다. 이 제한을 해결하기 위해 두가지 속성으로 구성된 객체를 반환할 수 있습니다. 생성된 이벤트의 이름인 `event`와 클라이언트로 전달되어야하는 `data`입니다.

```typescript
@@filename(events.gateway)
@SubscribeMessage('events')
handleEvent(@MessageBody() data: unknown): WsResponse<unknown> {
  const event = 'events';
  return { event, data };
}
@@switch
@Bind(MessageBody())
@SubscribeMessage('events')
handleEvent(data) {
  const event = 'events';
  return { event, data };
}
```

> info **힌트** `WsResponse` 인터페이스는 `@nestjs/websockets` 패키지에서 가져옵니다.

> warning **경고** `data` 필드가 `ClassSerializerInterceptor`에 의존하는 경우 `WsResponse`를 구현하는 클래스 인스턴스를 반환해야합니다. 이는 일반 자바스크립트 객체 응답을 무시하기 때문입니다.

수신 응답을 수신하려면 클라이언트가 다른 이벤트 리스너를 적용해야합니다.

```typescript
socket.on('events', data => console.log(data));
```

#### Asynchronous responses

메시지 핸들러는 동기식 또는 **비동기식**으로 응답할 수 있습니다. 따라서 `async` 메소드가 지원됩니다. 메시지 핸들러는 `Observable`을 반환할 수도 있습니다. 이 경우 스트림이 완료될 때까지 결과 값이 내보내집니다.

```typescript
@@filename(events.gateway)
@SubscribeMessage('events')
onEvent(@MessageBody() data: unknown): Observable<WsResponse<number>> {
  const event = 'events';
  const response = [1, 2, 3];

  return from(response).pipe(
    map(data => ({ event, data })),
  );
}
@@switch
@Bind(MessageBody())
@SubscribeMessage('events')
onEvent(data) {
  const event = 'events';
  const response = [1, 2, 3];

  return from(response).pipe(
    map(data => ({ event, data })),
  );
}
```

위의 예에서 메시지 핸들러는 **3번**(배열의 각 항목에 대해) 응답합니다.

#### Lifecycle hooks

세가지 유용한 수명주기 후크를 사용할 수 있습니다. 이들 모두에는 해당 인터페이스가 있으며 다음 표에 설명되어 있습니다.

<table>
  <tr>
    <td>
      <code>OnGatewayInit</code>
    </td>
    <td>
      <code>afterInit()</code>메소드를 강제로 구현합니다. 라이브러리별 서버 인스턴스를 인수로 사용합니다(및
       필요한 경우 나머지를 퍼뜨립니다).
    </td>
  </tr>
  <tr>
    <td>
      <code>OnGatewayConnection</code>
    </td>
    <td>
      <code>handleConnection()</code> 메서드를 강제로 구현합니다. 라이브러리 특정 클라이언트 소켓 인스턴스를 인수로 사용합니다.
    </td>
  </tr>
  <tr>
    <td>
      <code>OnGatewayDisconnect</code>
    </td>
    <td>
      an argument.
      <code>handleDisconnect()</code> 메서드를 강제로 구현합니다. 라이브러리 특정 클라이언트 소켓 인스턴스를 인수로 사용합니다.
    </td>
  </tr>
</table>

> info **힌트** 각 라이프사이클 인터페이스는 `@nestjs/websockets` 패키지에서 노출됩니다.

#### Server

경우에 따라 기본 **플랫폼 별** 서버 인스턴스에 직접 액세스할 수 있습니다. 이 객체에 대한 참조는 `afterInit()` 메소드(`OnGatewayInit` 인터페이스)에 인수로 전달됩니다. 또 다른 옵션은 `@WebSocketServer()` 데코레이터를 사용하는 것입니다.

```typescript
@WebSocketServer()
server: Server;
```

> warning **알림** `@WebSocketServer()` 데코레이터는 `@nestjs/websockets` 패키지에서 가져옵니다.

Nest는 사용할 준비가 되면 이 속성에 서버 인스턴스를 자동으로 할당합니다.

<app-banner-enterprise></app-banner-enterprise>

#### Example

작동하는 예는 [여기](https://github.com/nestjs/nest/tree/master/sample/02-gateways)에서 확인할 수 있습니다.
