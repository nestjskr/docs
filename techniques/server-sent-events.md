### Server-Sent Events

SSE(Server-Sent Events)는 클라이언트가 HTTP 연결을 통해 서버에서 자동 업데이트를 받을 수 있도록 하는 서버 푸시 기술입니다. 각 알림은 한쌍의 줄 바꿈으로 끝나는 텍스트 블록으로 전송됩니다 (자세히 알아보기 [여기](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)).

#### Usage

라우트(**컨트롤러 클래스**내에 등록된 라우트)에서 Server-Sent 이벤트를 활성화하려면 `@Sse()` 데코레이터로 메서드 핸들러에 주석을 추가합니다.

```typescript
@Sse('sse')
sse(): Observable<MessageEvent> {
  return interval(1000).pipe(map((_) => ({ data: { hello: 'world' } })));
}
```

> info **힌트** `@Sse()` 데코레이터는 `@nestjs/common`에서 가져오고 `Observable`, `interval`, `and map`은 `rxjs` 패키지에서 가져옵니다.

> warning **경고** 서버 전송 이벤트 경로는 `Observable` 스트림을 반환해야합니다.

위의 예에서는 실시간 업데이트를 전파 할 수있는 `sse`라는 경로를 정의했습니다. 이러한 이벤트는 [EventSource API](https://developer.mozilla.org/en-US/docs/Web/API/EventSource)를 사용하여 수신할 수 있습니다.

`sse` 메서드는 여러 `MessageEvent`를 내보내는 `Observable`을 반환합니다(이 예에서는 매초마다 새 `MessageEvent`를 내보냄). `MessageEvent` 객체는 사양과 일치하도록 다음 인터페이스를 준수해야합니다.

```typescript
export interface MessageEvent {
  data: string | object;
  id?: string;
  type?: string;
  retry?: number;
}
```

이를 통해 이제 클라이언트측 애플리케이션에서 `EventSource` 클래스의 인스턴스를 생성하여 생성자 인수로 `/sse` 라우트(위의 `@Sse()` 데코레이터에 전달한 엔드포인트와 일치)를 전달할 수 있습니다.

`EventSource` 인스턴스는 `text/event-stream` 형식으로 이벤트를 보내는 HTTP 서버에 대한 지속적인 연결을 엽니다. 연결은 `EventSource.close()`를 호출하여 닫힐 때까지 열린 상태로 유지됩니다.

연결이 열리면 서버에서 들어오는 메시지가 이벤트 형식으로 코드에 전달됩니다. 수신 메시지에 이벤트 필드가 있는 경우 트리거된 이벤트는 이벤트 필드값과 동일합니다. 이벤트 필드가 없으면 일반 `message` 이벤트가 발생합니다 ([source](https://developer.mozilla.org/en-US/docs/Web/API/EventSource)).

```javascript
const eventSource = new EventSource('/sse');
eventSource.onmessage = ({ data }) => {
  console.log('New message', JSON.parse(data));
};
```

#### Example

작동하는 예는 [여기](https://github.com/nestjs/nest/tree/master/sample/28-sse)에서 확인할 수 있습니다.
