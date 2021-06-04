### Interceptors

[일반 인터셉터](/interceptors)와 마이크로서비스 인터셉터 사이에는 차이가 없습니다. 다음 예제에서는 수동으로 인스턴스화된 메서드 범위 인터셉터를 사용합니다. HTTP 기반 애플리케이션과 마찬가지로 컨트롤러 범위의 인터셉터를 사용할 수도 있습니다(즉, 컨트롤러 클래스에 `@UseInterceptors()` 데코레이터를 접두사로 지정).

```typescript
@@filename()
@UseInterceptors(new TransformInterceptor())
@MessagePattern({ cmd: 'sum' })
accumulate(data: number[]): number {
  return (data || []).reduce((a, b) => a + b);
}
@@switch
@UseInterceptors(new TransformInterceptor())
@MessagePattern({ cmd: 'sum' })
accumulate(data) {
  return (data || []).reduce((a, b) => a + b);
}
```
