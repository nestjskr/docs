### Helmet

[Helmet](https://github.com/helmetjs/helmet)은 HTTP 헤더를 적절하게 설정하여 잘 알려진 웹 취약성으로부터 앱을 보호할 수 있습니다. 일반적으로 Helmet은 보안 관련 HTTP 헤더를 설정하는 14 개의 작은 미들웨어 함수 모음 일뿐입니다 (자세한 것은 [여기서](https://github.com/helmetjs/helmet#how-it-works) 참조).

> info **힌트** `helmet`을 전역으로 적용하거나 등록하려면 `app.use()`를 호출하거나 `app.use()`를 호출할 수 있는 설정 함수를 먼저 호출해야 합니다. 이는 미들웨어/라우트가 정의되는 순서가 중요한 기본 플랫폼 (즉, Express 또는 Fastify)이 작동하는 방식 때문입니다. 경로를 정의한 후 `helmet` 또는 `cors`와 같은 미들웨어를 사용하는 경우 해당 미들웨어는 해당 경로에 적용되지 않고 경로 뒤에 정의된 미들웨어에만 적용됩니다.

#### Use with Express (default)

필요한 패키지를 설치하여 시작하십시오.

```bash
$ npm i --save helmet
```

설치가 완료되면 글로벌 미들웨어로 적용하십시오.

```typescript
import * as helmet from 'helmet';
// somewhere in your initialization file
app.use(helmet());
```

> info **힌트** `Helmet`을 가져 오는 동안 `This expression is not callable` 오류가 발생하면 프로젝트의 `tsconfig.json` 파일에서 `allowSyntheticDefaultImports` 및 `esModuleInterop` 옵션이 `true`로 설정되었을 가능성이 높습니다.  이 경우 import 문을 `import helmet from 'helmet'`로 변경하십시오.

#### Use with Fastify

`FastifyAdapter`를 사용하는 경우 [fastify-helmet](https://github.com/fastify/fastify-helmet) 패키지를 설치합니다.

```bash
$ npm i --save fastify-helmet
```

[fastify-helmet](https://github.com/fastify/fastify-helmet)은 미들웨어로 사용해서는 안되며 [Fastify 플러그인](https://www.fastify.io/docs/latest/Plugins/)으로 사용됩니다. 즉,`app.register ()`를 사용하여

```typescript
import { fastifyHelmet } from 'fastify-helmet';
// somewhere in your initialization file
await app.register(fastifyHelmet);
```

> warning **경고** `apollo-server-fastify` 및 `fastify-helmet`을 사용할 때 GraphQL 플레이 그라운드의 [CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)에 문제가 있을 수 있습니다. 이 충돌을 해결하기 위해 아래와 같이 CSP를 구성합니다.
>
> ```typescript
> await app.register(fastifyHelmet, {
>   contentSecurityPolicy: {
>     directives: {
>       defaultSrc: [`'self'`],
>       styleSrc: [`'self'`, `'unsafe-inline'`, 'cdn.jsdelivr.net', 'fonts.googleapis.com'],
>       fontSrc: [`'self'`, 'fonts.gstatic.com'],
>       imgSrc: [`'self'`, 'data:', 'cdn.jsdelivr.net'],
>       scriptSrc: [`'self'`, `https: 'unsafe-inline'`, `cdn.jsdelivr.net`],
>     },
>   },
> });
>
> // If you are not going to use CSP at all, you can use this:
> await app.register(fastifyHelmet, {
>   contentSecurityPolicy: false,
> });
> ```
