### Cookies

**HTTP 쿠키**는 사용자의 브라우저에 저장되는 작은 데이터입니다. 쿠키는 웹사이트가 상태 저장 정보를 기억할 수 있는 신뢰할 수 있는 메커니즘으로 설계되었습니다. 사용자가 웹사이트를 다시 방문하면 요청과 함께 쿠키가 자동으로 전송됩니다.

#### Use with Express (default)

먼저 필요한 패키지 (및 타입스크립트 사용자를 위한 타입)를 설치합니다.

```shell
$ npm i cookie-parser
$ npm i -D @types/cookie-parser
```

설치가 완료되면 `cookie-parser` 미들웨어를 글로벌 미들웨어로 적용합니다 (예: `main.ts` 파일).

```typescript
import * as cookieParser from 'cookie-parser';
// somewhere in your initialization file
app.use(cookieParser());
```

`cookieParser` 미들웨어에 몇가지 옵션을 전달할 수 있습니다.

- `secret`은 쿠키 서명에 사용되는 문자열 또는 배열입니다. 이는 선택 사항이며 지정되지 않은 경우 서명된 쿠키를 구문 분석하지 않습니다. 문자열이 제공되면 이것은 비밀(secret)로 사용됩니다. 배열이 제공되면 순서대로 각 암호를 사용하여 쿠키 서명을 해제하려고 시도합니다.
- `options`는 두번째 옵션으로 `cookie.parse`에 전달되는 객체입니다. 자세한 내용은 [cookie](https://www.npmjs.org/package/cookie)를 참조하세요.

미들웨어는 요청에서 `Cookie` 헤더를 구문 분석하고 쿠키 데이터를 `req.cookies`속성으로 노출하고, 비밀이 제공된 경우 `req.signedCookies` 속성으로 노출합니다. 이러한 속성은 쿠키 이름과 쿠키값의 이름 값 쌍입니다.

비밀이 제공되면 이 모듈은 서명된 쿠키 값의 서명을 해제하고 유효성을 검사하고 해당 이름 값 쌍을 `req.cookies`에서 `req.signedCookies`로 이동합니다. 서명된 쿠키는 `s:`로 시작하는 값이 있는 쿠키입니다. 서명 유효성 검사에 실패한 서명된 쿠키는 변조된 값 대신 `false`값을 갖습니다.

이를 통해 이제 다음과 같이 라우트 핸들러 내에서 쿠키를 읽을 수 있습니다.

```typescript
@Get()
findAll(@Req() request: Request) {
  console.log(request.cookies); // or "request.cookies['cookieKey']"
  // or console.log(request.signedCookies);
}
```

> info **Hint** The `@Req()` decorator is imported from the `@nestjs/common`, while `Request` from the `express` package.

발신 응답에 쿠키를 첨부하려면 `Response#cookie()` 메소드를 사용하십시오.

```typescript
@Get()
findAll(@Res({ passthrough: true }) response: Response) {
  response.cookie('key', 'value')
}
```

> warning **경고** 응답 처리 로직을 프레임워크에 맡기려면 위에 표시된대로 `passthrough` 옵션을 `true`로 설정해야합니다. 자세한 내용은 [여기](/controllers#appendix-library-specific-approach)를 참조하세요.

> info **힌트** `@Res()` 데코레이터는 `@nestjs/common`에서 가져오고 `Response`는 `express` 패키지에서 가져옵니다.

#### Use with Fastify

먼저 필요한 패키지를 설치하십시오.

```shell
$ npm i fastify-cookie
```

설치가 완료되면 `fastify-cookie` 플러그인을 등록합니다.

```typescript
import fastifyCookie from 'fastify-cookie';

// somewhere in your initialization file
const app = await NestFactory.create<NestFastifyApplication>(
  AppModule,
  new FastifyAdapter(),
);
app.register(fastifyCookie, {
  secret: 'my-secret', // for cookies signature
});
```

이를 통해 이제 다음과 같이 라우트 핸들러 내에서 쿠키를 읽을 수 있습니다.

```typescript
@Get()
findAll(@Req() request: FastifyRequest) {
  console.log(request.cookies); // or "request.cookies['cookieKey']"
}
```

> info **힌트** `@Req()` 데코레이터는 `@nestjs/common`에서 가져오고 `FastifyRequest`는 `fastify` 패키지에서 가져옵니다.

발신 응답에 쿠키를 첨부하려면 `FastifyReply#setCookie()` 메소드를 사용하세요.

```typescript
@Get()
findAll(@Res({ passthrough: true }) response: FastifyReply) {
  response.setCookie('key', 'value')
}
```

`FastifyReply#setCookie()` 메소드에 대한 자세한 내용은 이 [페이지](https://github.com/fastify/fastify-cookie#sending)를 확인하세요.

> warning **경고** 응답 처리 로직을 프레임워크에 맡기려면 위에 표시된대로 `passthrough` 옵션을 `true`로 설정해야 합니다. 자세한 내용은 [여기](/controllers#appendix-library-specific-approach)를 참조하세요.

> info **힌트** `@Res()`데코레이터는 `@nestjs/common`에서 가져오고 `FastifyReply`는 `fastify` 패키지에서 가져옵니다.

#### Creating a custom decorator (cross-platform)

들어오는 쿠키에 접근하는 편리하고 선언적인 방법을 제공하기 위해 [custom decorator](/ custom-decorators)를 만들 수 있습니다.

```typescript
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const Cookies = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return data ? request.cookies?.[data] : request.cookies;
  },
);
```

`@Cookies()` 데코레이터는 `req.cookies` 객체에서 모든 쿠키 또는 명명된 쿠키를 추출하고 데코레이팅된 매개변수를 해당 값으로 채 웁니다.

이제 다음과 같이 라우트 핸들러 서명에서 데코레이터를 사용할 수 있습니다.

```typescript
@Get()
findAll(@Cookies('name') name: string) {}
```
