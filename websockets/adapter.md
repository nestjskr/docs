### Adapters

WebSockets 모듈은 플랫폼에 구애받지 않으므로 `WebSocketAdapter` 인터페이스를 사용하여 자체 라이브러리(또는 기본 구현)를 가져올 수 있습니다. 이 인터페이스는 다음 표에 설명된 몇가지 메서드를 강제로 구현합니다.

<table>
  <tr>
    <td><code>create</code></td>
    <td>전달된 인수를 기반으로 소켓 인스턴스를 만듭니다.</td>
  </tr>
  <tr>
    <td><code>bindClientConnect</code></td>
    <td>클라이언트 연결 이벤트를 바인드합니다.</td>
  </tr>
  <tr>
    <td><code>bindClientDisconnect</code></td>
    <td>클라이언트 연결 해제 이벤트를 바인딩합니다(선택사항*).</td>
  </tr>
  <tr>
    <td><code>bindMessageHandlers</code></td>
    <td>수신 메시지를 해당 메시지 핸들러에 바인드합니다.</td>
  </tr>
  <tr>
    <td><code>close</code></td>
    <td>서버 인스턴스를 종료합니다.</td>
  </tr>
</table>

#### Extend socket.io

[socket.io](https://github.com/socketio/socket.io) 패키지는 `IoAdapter` 클래스로 래핑됩니다. 어댑터의 기본 기능을 향상 시키려면 어떻게해야 합니까? 예를 들어 기술 요구 사항에는 웹 서비스의 여러로드 밸런싱된 인스턴스에서 이벤트를 브로드캐스트하는 기능이 필요합니다. 이를 위해 `IoAdapter`를 확장하고 새 socket.io 서버를 인스턴스화하는 단일 메소드를 재정의할 수 있습니다. 하지만 먼저 필요한 패키지를 설치하겠습니다.

```bash
$ npm i --save socket.io-redis
```

패키지가 설치되면 `RedisIoAdapter` 클래스를 만들 수 있습니다.

```typescript
import { IoAdapter } from '@nestjs/platform-socket.io';
import * as redisIoAdapter from 'socket.io-redis';

export class RedisIoAdapter extends IoAdapter {
  createIOServer(port: number, options?: any): any {
    const server = super.createIOServer(port, options);
    const redisAdapter = redisIoAdapter({ host: 'localhost', port: 6379 });

    server.adapter(redisAdapter);
    return server;
  }
}
```

그런 다음 새로 생성된 Redis 어댑터로 전환하기만 하면됩니다.

```typescript
const app = await NestFactory.create(AppModule);
app.useWebSocketAdapter(new RedisIoAdapter(app));
```

#### Ws library

사용 가능한 또 다른 어댑터는 프레임워크 간의 프록시 역할을 하는 `WsAdapter`이며, 매우 빠르고 철저하게 테스트된 [ws](https://github.com/websockets/ws) 라이브러리를 통합합니다. 이 어댑터는 기본 브라우저 WebSocket과 완벽하게 호환되며 socket.io 패키지보다 훨씬 빠릅니다. 불행히도 기본적으로 사용할 수 있는 기능이 훨씬 적습니다. 어떤 경우에는 꼭 필요하지는 않을 수도 있습니다.

`ws`를 사용하려면 먼저 필요한 패키지를 설치해야합니다.

```bash
$ npm i --save @nestjs/platform-ws
```

패키지가 설치되면 어댑터를 전환할 수 있습니다.

```typescript
const app = await NestFactory.create(AppModule);
app.useWebSocketAdapter(new WsAdapter(app));
```

> info **힌트** `WsAdapter`는 `@nestjs/platform-ws`에서 가져옵니다.

#### Advanced (custom adapter)

데모 목적으로 [ws](https://github.com/websockets/ws) 라이브러리를 수동으로 통합할 것입니다. 앞서 언급했듯이 이 라이브러리의 어댑터는 이미 생성되었으며 `@nestjs/platform-ws` 패키지에서 `WsAdapter` 클래스로 노출됩니다. 단순화된 구현이 잠재적으로 어떻게 보일 수 있는지는 다음과 같습니다.

```typescript
@@filename(ws-adapter)
import * as WebSocket from 'ws';
import { WebSocketAdapter, INestApplicationContext } from '@nestjs/common';
import { MessageMappingProperties } from '@nestjs/websockets';
import { Observable, fromEvent, EMPTY } from 'rxjs';
import { mergeMap, filter } from 'rxjs/operators';

export class WsAdapter implements WebSocketAdapter {
  constructor(private app: INestApplicationContext) {}

  create(port: number, options: any = {}): any {
    return new ws.Server({ port, ...options });
  }

  bindClientConnect(server, callback: Function) {
    server.on('connection', callback);
  }

  bindMessageHandlers(
    client: WebSocket,
    handlers: MessageMappingProperties[],
    process: (data: any) => Observable<any>,
  ) {
    fromEvent(client, 'message')
      .pipe(
        mergeMap(data => this.bindMessageHandler(data, handlers, process)),
        filter(result => result),
      )
      .subscribe(response => client.send(JSON.stringify(response)));
  }

  bindMessageHandler(
    buffer,
    handlers: MessageMappingProperties[],
    process: (data: any) => Observable<any>,
  ): Observable<any> {
    const message = JSON.parse(buffer.data);
    const messageHandler = handlers.find(
      handler => handler.message === message.event,
    );
    if (!messageHandler) {
      return EMPTY;
    }
    return process(messageHandler.callback(message.data));
  }

  close(server) {
    server.close();
  }
}
```

> info **힌트** [ws](https://github.com/websockets/ws) 라이브러리를 활용하려면 자체 라이브러리를 만드는 대신 기본 제공 `WsAdapter`를 사용하세요.

그런 다음 `useWebSocketAdapter()` 메서드를 사용하여 사용자 지정 어댑터를 설정할 수 있습니다.

```typescript
@@filename(main)
const app = await NestFactory.create(AppModule);
app.useWebSocketAdapter(new WsAdapter(app));
```

#### Example

`WsAdapter`를 사용하는 실제 예제는 [여기](https://github.com/nestjs/nest/tree/master/sample/16-gateways-ws)에서 확인할 수 있습니다.
