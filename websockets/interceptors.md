### Interceptors

[일반 인터셉터](/interceptors)와 웹소켓 인터셉터 사이에는 차이가 없습니다. 다음 예제에서는 수동으로 인스턴스화된 메서드 범위 인터셉터를 사용합니다. HTTP 기반 애플리케이션과 마찬가지로 게이트웨이 범위의 인터셉터를 사용할 수도 있습니다(즉, 게이트웨이 클래스에 `@UseInterceptors()` 데코레이터를 접두사로 추가).

```typescript
@@filename()
@UseInterceptors(new TransformInterceptor())
@SubscribeMessage('events')
handleEvent(client: Client, data: unknown): WsResponse<unknown> {
  const event = 'events';
  return { event, data };
}
@@switch
@UseInterceptors(new TransformInterceptor())
@SubscribeMessage('events')
handleEvent(client, data) {
  const event = 'events';
  return { event, data };
}
```
