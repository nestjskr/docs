### Session

**HTTP 세션**은 여러 요청에서 사용자에 대한 정보를 저장하는 방법을 제공하며, 이는 특히 [MVC](/techniques/mvc) 애플리케이션에 유용합니다.

#### Use with Express (default)

먼저 필요한 패키지(및 TypeScript 사용자를 위한 타입)를 설치합니다.

```shell
$ npm i express-session
$ npm i -D @types/express-session
```

설치가 완료되면 `express-session` 미들웨어를 글로벌 미들웨어로 적용합니다 (예: `main.ts` 파일).

```typescript
import * as session from 'express-session';
// somewhere in your initialization file
app.use(
  session({
    secret: 'my-secret',
    resave: false,
    saveUninitialized: false,
  }),
);
```

> warning **알림** 기본 서버측 세션 저장소는 의도적으로 프로덕션 환경용으로 설계되지 않았습니다. 대부분의 조건에서 메모리가 누출되고 단일 프로세스를 지나 확장되지 않으며 디버깅 및 개발을 위한 것입니다. [공식 저장소](https://github.com/expressjs/session)에서 자세히 알아보세요.

`secret`은 세션 ID 쿠키에 서명하는 데 사용됩니다. 이는 단일 비밀에 대한 문자열이거나 여러 비밀의 배열일 수 있습니다. 비밀 배열이 제공되면 첫번째 요소만 세션 ID 쿠키에 서명하는데 사용되며 모든 요소는 요청에서 서명을 확인할 때 고려됩니다. 비밀 자체는 사람이 쉽게 파싱해서는 안되며 임의의 문자 집합이 가장 좋습니다.

`resave` 옵션을 활성화하면 요청중에 세션이 수정되지 않은 경우에도 세션이 세션 저장소에 다시 저장됩니다. 기본값은 `true`이지만 기본값이 향후 변경될 예정이므로 기본값 사용은 더 이상 사용되지 않습니다.

마찬가지로 `saveUninitialized` 옵션을 활성화하면 "초기화되지 않은" 세션이 저장소에 저장됩니다. 세션은 새 세션이지만 수정되지 않은 경우 초기화되지 않습니다. `false`를 선택하면 로그인 세션을 구현하거나 서버 스토리지 사용량을 줄이거나 쿠키를 설정하기 전에 권한이 필요한 법률을 준수하는데 유용합니다. `false`를 선택하면 클라이언트가 세션([source](https://github.com/expressjs/session#saveuninitialized))없이 여러개의 병렬 요청을 수행하는 경합 조건에도 도움이 됩니다.

여러 다른 옵션을 `session` 미들웨어에 전달할 수 있습니다. 자세한 내용은 [API 문서](https://github.com/expressjs/session#options)를 참조하세요.

> info **힌트** `secure: true`가 권장되는 옵션입니다. 그러나 HTTPS가 활성화된 웹사이트가 필요합니다. 즉, 보안 쿠키에는 HTTPS가 필요합니다. 보안이 설정되어 있고 HTTP를 통해 사이트에 액세스하면 쿠키가 설정되지 않습니다. node.js가 프록시 뒤에 있고 `secure: true`를 사용하는 경우 express에서 `"trust proxy"`를 설정해야합니다.

이를 통해 이제 다음과 같이 라우트 핸들러내에서 세션값을 설정하고 읽을 수 있습니다.

```typescript
@Get()
findAll(@Req() request: Request) {
  request.session.visits = request.session.visits ? request.session.visits + 1 : 1;
}
```

> info **힌트** `@Req()` 데코레이터는 `@nestjs/common`에서 가져오고 `Request`는 `express` 패키지에서 가져옵니다.

또는 다음과 같이 `@Session()` 데코레이터를 사용하여 요청에서 세션 객체를 추출할 수 있습니다.

```typescript
@Get()
findAll(@Session() session: Record<string, any>) {
  session.visits = session.visits ? session.visits + 1 : 1;
}
```

> info **힌트** `@Session()` 데코레이터는 `@nestjs/common` 패키지에서 가져옵니다.

#### Use with Fastify

먼저 필요한 패키지를 설치하십시오.

```shell
$ npm i fastify-secure-session
```

설치가 완료되면 `fastify-secure-session` 플러그인을 등록하십시오.

```typescript
import secureSession from 'fastify-secure-session';

// somewhere in your initialization file
const app = await NestFactory.create<NestFastifyApplication>(
  AppModule,
  new FastifyAdapter(),
);
app.register(secureSession, {
  secret: 'averylogphrasebiggerthanthirtytwochars',
  salt: 'mq9hDxBVDbspDR6n',
});
```

> info **힌트** 키를 미리 생성하거나 ([지침 참조](https://github.com/fastify/fastify-secure-session)) [키 순환](https://github.com/fastify/fastify-secure-session#using-keys-with-key-rotation)을 사용할 수도 있습니다.

[공식 저장소](https://github.com/fastify/fastify-secure-session)에서 사용 가능한 옵션에 대해 자세히 알아보세요.

이를 통해 이제 다음과 같이 라우트 핸들러내에서 세션값을 설정하고 읽을 수 있습니다.

```typescript
@Get()
findAll(@Req() request: FastifyRequest) {
  const visits = request.session.get('visits');
  request.session.set('visits', visits ? visits + 1 : 1);
}
```

또는 다음과 같이 `@Session()` 데코레이터를 사용하여 요청에서 세션 객체를 추출할 수 있습니다.

```typescript
@Get()
findAll(@Session() session: secureSession.Session) {
  const visits = session.get('visits');
  session.set('visits', visits ? visits + 1 : 1);
}
```

> info **힌트** `@Session()` 데코레이터는 `@nestjs/common`에서 가져 오며 `secureSession.Session`은 `fastify-secure-session` 패키지에서 가져옵니다 (import 문: `import * as secureSession from 'fastify-secure-session'`).
