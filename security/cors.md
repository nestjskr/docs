### CORS

CORS(Cross-Origin Resource Sharing)는 다른 도메인에서 리소스를 요청할 수 있도록 하는 메커니즘입니다. 내부적으로 Nest는 [Express cors](https://github.com/expressjs/cors) 패키지를 사용합니다. 이 패키지는 요구 사항에 따라 사용자 지정할 수 있는 다양한 옵션을 제공합니다.

#### Getting started

CORS를 활성화하려면 Nest 애플리케이션 객체에서 `enableCors()` 메서드를 호출하세요.

```typescript
const app = await NestFactory.create(AppModule);
app.enableCors();
await app.listen(3000);
```

`enableCors()` 메소드는 선택적 구성 개체 인수를 사용합니다. 이 개체의 사용 가능한 속성은 공식 [CORS](https://github.com/expressjs/cors#configuration-options) 문서에 설명되어 있습니다. 또 다른 방법은 요청에 따라 구성 객체를 비동기식으로 정의할 수있는 [콜백 함수](https://github.com/expressjs/cors#configuring-cors-asynchronously)를 전달하는 것입니다.

또는 `create()` 메소드의 옵션 객체를 통해 CORS를 활성화합니다. 기본 설정으로 CORS를 활성화하려면 `cors` 속성을 `true`로 설정합니다.
또는 [CORS 구성 개체](https://github.com/expressjs/cors#configuration-options) 또는 [콜백 함수](https://github.com/expressjs/cors#configuring-cors-asynchronously)를 전달합니다. `cors`속성 값으로 사용하여 동작을 맞춤 설정합니다.

```typescript
const app = await NestFactory.create(AppModule, { cors: true });
await app.listen(3000);
```

위의 방법은 REST 엔드포인트에만 적용됩니다.

GraphQL에서 CORS를 활성화하려면 GraphQL 모듈을 가져올 때 `cors` 속성을 `true`로 설정하거나 [CORS 구성 객체](https://github.com/expressjs/cors#configuration-options) 또는 [콜백 함수](https://github.com/expressjs/cors#configuring-cors-asynchronously)를 `cors` 속성 값으로 전달합니다.

> warning **경고** `CorsOptionsDelegate` 솔루션은 아직 `apollo-server-fastify` 패키지에서 작동하지 않습니다.

```typescript
GraphQLModule.forRoot({
  cors: {
    origin: 'http://localhost:3000',
    credentials: true,
  },
}),
```
