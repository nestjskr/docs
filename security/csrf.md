### CSRF Protection

사이트 간 요청 위조 (CSRF 또는 XSRF라고도 함)는 웹 애플리케이션이 신뢰하는 사용자로부터 **승인되지 않은** 명령이 전송되는 웹 사이트의 악의적인 악용 유형입니다. 이러한 종류의 공격을 완화하려면 [csurf](https://github.com/expressjs/csurf) 패키지를 사용할 수 있습니다.

#### Use with Express (default)

필요한 패키지를 설치하여 시작하십시오.

```bash
$ npm i --save csurf
```

> warning **경고** [csurf 미들웨어 페이지](https://github.com/expressjs/csurf#csurf)에 설명된대로 csurf 모듈을 사용하려면 먼저 세션 미들웨어 또는 쿠키 파서를 초기화해야 합니다. 자세한 지침은 해당 설명서를 참조하십시오.

설치가 완료되면 csurf 미들웨어를 글로벌 미들웨어로 적용하십시오.

```typescript
import * as csurf from 'csurf';
// somewhere in your initialization file
app.use(csurf());
```

#### Use with Fastify

필요한 패키지를 설치하여 시작하십시오.

```bash
$ npm i --save fastify-csrf
```

설치가 완료되면 다음과 같이 `fastify-csrf` 플러그인을 등록합니다.

```typescript
import fastifyCsrf from 'fastify-csrf';
// somewhere in your initialization file
app.register(fastifyCsrf);
```
