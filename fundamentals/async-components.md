### Asynchronous providers

때때로 애플리케이션 시작은 하나 이상의 **비동기 작업**이 완료될 때까지 지연되어야 합니다. 예를 들어, 데이터베이스와의 연결이 설정될 때까지 요청 수락을 시작하지 않을 수 있습니다. _비동기 프로바이더_ 를 사용하여 이를 달성할 수 있습니다.

이에 대한 구문은 `useFactory` 구문과 함께 `async/await`를 사용하는 것입니다. 팩토리는 `Promise`를 반환하고 팩토리 함수는 비동기 작업을 `await`할 수 있습니다. Nest는 그러한 프로바이더에 의존하는 (주입하는) 클래스를 인스턴스화하기 전에 promise의 해결을 기다립니다.

```typescript
{
  provide: 'ASYNC_CONNECTION',
  useFactory: async () => {
    const connection = await createConnection(options);
    return connection;
  },
}
```

> info **힌트** 사용자 지정 프로바이더 구문에 대한 자세한 내용은 [여기](/fundamentals/custom-providers)를 참조하세요.

#### Injection

비동기 프로바이더는 다른 프로바이더와 마찬가지로 토큰에 의해 다른 구성 요소에 삽입됩니다. 위의 예에서는 `@Inject('ASYNC_CONNECTION')`구문을 사용합니다.

#### Example

[TypeORM 레시피](/recipes/sql-typeorm)에는 비동기 프로바이더의 보다 실질적인 예가 있습니다.