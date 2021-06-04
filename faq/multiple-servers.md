### HTTPS

HTTPS 프로토콜을 사용하는 애플리케이션을 만들려면 `NestFactory` 클래스의 `create()` 메서드에 전달된 옵션 객체에서 `httpsOptions` 속성을 설정합니다.

```typescript
const httpsOptions = {
  key: fs.readFileSync('./secrets/private-key.pem'),
  cert: fs.readFileSync('./secrets/public-certificate.pem'),
};
const app = await NestFactory.create(AppModule, {
  httpsOptions,
});
await app.listen(3000);
```

`FastifyAdapter`를 사용하는 경우 다음과 같이 애플리케이션을 만듭니다.

```typescript
const app = await NestFactory.create<NestFastifyApplication>(
  AppModule,
  new FastifyAdapter({ https: httpsOptions }),
);
```

#### Multiple simultaneous servers

다음 레시피는 여러 포트(예: 비 HTTPS 포트 및 HTTPS 포트)에서 동시에 수신하는 Nest 애플리케이션을 인스턴스화하는 방법을 보여줍니다.

```typescript
const httpsOptions = {
  key: fs.readFileSync('./secrets/private-key.pem'),
  cert: fs.readFileSync('./secrets/public-certificate.pem'),
};

const server = express();
const app = await NestFactory.create(
  AppModule,
  new ExpressAdapter(server),
);
await app.init();

http.createServer(server).listen(3000);
https.createServer(httpsOptions, server).listen(443);
```

> info **힌트** `ExpressAdapter`는 `@nestjs/platform-express` 패키지에서 가져옵니다. `http` 및 `https` 패키지는 기본 Node.js 패키지입니다.

> **경고** 이 레시피는 [GraphQL 구독](/graphql/subscriptions)에서 작동하지 않습니다.
