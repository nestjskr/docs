### CORS

크로스 오리진 리소스 쉐어링(Cross-origin resource sharing) (CORS) 은 다른 도메인이 가지고 있는 리소스를 요청하는 메커니즘을 뜻합니다.네스트(Nest.js)는 Node.js의 프레임워크인 Express의  [cors](https://github.com/expressjs/cors) 패키지를 사용합니다. 이 패키지는 당신의 요구사항을 바탕으로 다양한 옵션을 설정할 수 있도록 제공하고 있습니다.

#### 시작하기

CORS를 사용하려면, Nest 어플리케이션(Application) 객체에서 `enableCors()` 메서드(Method)를 호출합니다.

```typescript
const app = await NestFactory.create(AppModule);
app.enableCors();
await app.listen(3000);
```

`enableCors()`메서드는 선택적 구성 객체 인수를 사용합니다. 이 객체가 사용 가능한 속성(properties)은 공식 [CORS](https://github.com/expressjs/cors#configuration-options) 문서에 설명되어 있습니다.

다른 방법으로는, `create()` 메서드의 옵션 객체를 통해 CORS를 활성화 합니다. 기본 설정으로 CORS를 활성화 하려면`cors`속성을 <code>true</code>로 설정합니다.또는 <a>CORS 구성 객체</a>(Configuration object)를 <code>cors</code> 속성 값 으로 전달하여 동작을 사용자 지정합니다.

```typescript
const app = await NestFactory.create(AppModule, { cors: true });
await app.listen(3000);
```

위의 방법은 REST endpoints에만 적용됩니다.GraphQL에서 CORS를 사용하려면, GraphQL module을 가져올때(import) <code>cors</code> 속성을 `true`로 설정하거나 [CORS 구성 객체](https://github.com/expressjs/cors#configuration-options)를 `cors` 속성값을  전달해주면됩니다.

```typescript
GraphQLModule.forRoot({
  cors: {
    origin: 'http://localhost:3000',
    credentials: true,
  },
}),
```
