### Guards

웹소켓 가드와 [일반 HTTP 애플리케이션 가드](/guards) 사이에는 근본적인 차이가 없습니다. 유일한 차이점은 `HttpException`을 던지는 대신 `WsException`을 사용해야 한다는 것입니다.

> info **힌트** `WsException` 클래스는 `@nestjs/websockets` 패키지에서 노출됩니다.

#### Binding guards

다음 예제에서는 메서드 범위 가드를 사용합니다. HTTP 기반 애플리케이션과 마찬가지로 게이트웨이 범위 가드를 사용할 수도 있습니다(즉, 게이트웨이 클래스 앞에 `@UseGuards()` 데코레이터를 붙임).

```typescript
@@filename()
@UseGuards(AuthGuard)
@SubscribeMessage('events')
handleEvent(client: Client, data: unknown): WsResponse<unknown> {
  const event = 'events';
  return { event, data };
}
@@switch
@UseGuards(AuthGuard)
@SubscribeMessage('events')
handleEvent(client, data) {
  const event = 'events';
  return { event, data };
}
```
