### Pipes

[일반 파이프](/pipes)와 웹소켓 파이프 사이에는 근본적인 차이가 없습니다. 유일한 차이점은 `HttpException`을 던지는 대신 `WsException`을 사용해야 한다는 것입니다. 또한 모든 파이프는 `data` 매개변수에만 적용됩니다(`client` 인스턴스의 유효성을 검사하거나 변환하는 것은 쓸모가 없기 때문입니다).

> info **힌트** `WsException` 클래스는 `@nestjs/websockets` 패키지에서 노출됩니다.

#### Binding pipes

다음 예제에서는 수동으로 인스턴스화된 메서드 범위 파이프를 사용합니다. HTTP 기반 애플리케이션과 마찬가지로 게이트웨이 범위 파이프를 사용할 수도 있습니다(즉, 게이트웨이 클래스 앞에 `@UsePipes()` 데코레이터를 붙임).

```typescript
@@filename()
@UsePipes(new ValidationPipe())
@SubscribeMessage('events')
handleEvent(client: Client, data: unknown): WsResponse<unknown> {
  const event = 'events';
  return { event, data };
}
@@switch
@UsePipes(new ValidationPipe())
@SubscribeMessage('events')
handleEvent(client, data) {
  const event = 'events';
  return { event, data };
}
```
