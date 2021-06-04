### Hybrid application

하이브리드 애플리케이션은 HTTP 요청을 수신하고 연결된 마이크로서비스를 사용하는 애플리케이션입니다. `INestApplication` 인스턴스는 `connectMicroservice()` 메소드를 통해 `INestMicroservice` 인스턴스와 연결할 수 있습니다.

```typescript
const app = await NestFactory.create(AppModule);
const microservice = app.connectMicroservice({
  transport: Transport.TCP,
});

await app.startAllMicroservicesAsync();
await app.listen(3001);
```

여러 마이크로서비스 인스턴스를 연결하려면 각 마이크로서비스에 대해 `connectMicroservice()`를 호출합니다.

```typescript
const app = await NestFactory.create(AppModule);
// microservice #1
const microserviceTcp = app.connectMicroservice<MicroserviceOptions>({
  transport: Transport.TCP,
  options: {
    port: 3001,
  },
});
// microservice #2
const microserviceRedis = app.connectMicroservice<MicroserviceOptions>({
  transport: Transport.REDIS,
  options: {
    url: 'redis://localhost:6379',
  },
});

await app.startAllMicroservicesAsync();
await app.listen(3001);
```

여러 마이크로서비스가 있는 하이브리드 애플리케이션에서 `@MessagePattern()`을 하나의 전송 전략(예: MQTT)에만 바인딩하려면 모든 기본 제공 전송 전략이 정의된 열거형 `Transport` 타입의 두번째 인수를 전달할 수 있습니다.

```typescript
@@filename()
@MessagePattern('time.us.*', Transport.NATS)
getDate(@Payload() data: number[], @Ctx() context: NatsContext) {
  console.log(`Subject: ${context.getSubject()}`); // e.g. "time.us.east"
  return new Date().toLocaleTimeString(...);
}
@MessagePattern({ cmd: 'time.us' }, Transport.TCP)
getTCPDate(@Payload() data: number[]) {
  return new Date().toLocaleTimeString(...);
}
@@switch
@Bind(Payload(), Ctx())
@MessagePattern('time.us.*', Transport.NATS)
getDate(data, context) {
  console.log(`Subject: ${context.getSubject()}`); // e.g. "time.us.east"
  return new Date().toLocaleTimeString(...);
}
@Bind(Payload(), Ctx())
@MessagePattern({ cmd: 'time.us' }, Transport.TCP)
getTCPDate(data, context) {
  return new Date().toLocaleTimeString(...);
}
```

> info **힌트** `@Payload()`, `@Ctx()`, `Transport` 및 `NatsContext`는 `@nestjs/microservices`에서 가져옵니다.

#### Sharing configuration

기본적으로 하이브리드 애플리케이션은 기본(HTTP 기반) 애플리케이션에 대해 구성된 글로벌 파이프, 인터셉터, 가드 및 필터를 상속하지 않습니다.
기본 애플리케이션에서 이러한 구성 속성을 상속하려면 다음과 같이 `connectMicroservice()` 호출의 두번째 인수(선택적 옵션 객체)에 `inheritAppConfig` 속성을 설정합니다.

```typescript
const microservice = app.connectMicroservice({
  transport: Transport.TCP
}, { inheritAppConfig: true });
```
