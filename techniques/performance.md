### Performance (Fastify)

기본적으로 Nest는 [Express](https://expressjs.com/) 프레임워크를 사용합니다. 앞서 언급했듯이 Nest는 [Fastify](https://github.com/fastify/fastify)와 같은 다른 라이브러리와의 호환성도 제공합니다. Nest는 미들웨어 및 핸들러를 적절한 라이브러리별 구현에 프록시하는 것이 주요 기능인 프레임워크 어댑터를 구현하여 이러한 프레임워크 독립성을 달성합니다.

> info **힌트** 프레임워크 어댑터를 구현하려면 대상 라이브러리가 Express에 있는 것과 유사한 요청/응답 파이프라인 처리를 제공해야합니다.

[Fastify](https://github.com/fastify/fastify)는 Express와 유사한 방식으로 디자인 문제를 해결하기 때문에 Nest에 적합한 대체 프레임워크를 제공합니다. 그러나 fastify는 Express보다 훨씬 **빠르므로** 벤치마크 결과가 거의 두배나 더 우수합니다. 공정한 질문은 Nest가 Express를 기본 HTTP 공급자로 사용하는 이유입니다. 그 이유는 Express가 널리 사용되고 잘 알려져 있으며 Nest 사용자가 즉시 사용할 수 있는 수많은 호환 가능한 미들웨어 세트를 가지고 있기 때문입니다.

그러나 Nest는 프레임워크 독립성을 제공하므로 이들간에 쉽게 마이그레이션할 수 있습니다. Fastify는 매우 빠른 성능에 높은 가치를 두는 경우 더 나은 선택이 될 수 있습니다. Fastify를 사용하려면 이 장에 표시된대로 내장된 `FastifyAdapter`를 선택하기만하면 됩니다.

#### Installation

먼저 필요한 패키지를 설치해야합니다.

```bash
$ npm i --save @nestjs/platform-fastify
```
> warning **경고** `@nestjs/platform-fastify` 버전 `>=7.5.0` 및 `apollo-server-fastify`를 사용할 때 `fastify` 버전 `^3.0.0`과 호환되지 않아 GraphQL 플레이 그라운드가 작동하지 않을 수 있습니다. 불안정한 `apollo-server-fastify` 버전 `^3.0.0-alpha.3`을 사용하거나 임시로 대신 express를 선택할 수 있습니다.

#### Adapter

Fastify 플랫폼이 설치되면 `FastifyAdapter`를 사용할 수 있습니다.

```typescript
@@filename(main)
import { NestFactory } from '@nestjs/core';
import {
  FastifyAdapter,
  NestFastifyApplication,
} from '@nestjs/platform-fastify';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create<NestFastifyApplication>(
    AppModule,
    new FastifyAdapter()
  );
  await app.listen(3000);
}
bootstrap();
```

기본적으로 Fastify는 `localhost 127.0.0.1` 인터페이스 ([자세히 알아보기](https://www.fastify.io/docs/latest/Getting-Started/#your-first-server))에서만 수신합니다. 다른 호스트에서 연결을 허용하려면 `listen()` 호출에 `'0.0.0.0'`을 지정해야합니다.

```typescript
async function bootstrap() {
  const app = await NestFactory.create<NestFastifyApplication>(
    AppModule,
    new FastifyAdapter()
  );
  await app.listen(3000, '0.0.0.0');
}
```

#### Platform specific packages

`FastifyAdapter`를 사용할 때 Nest는 Fastify를 **HTTP 공급자**로 사용합니다. 이는 Express에 의존하는 각 레시피가 더 이상 작동하지 않을 수 있음을 의미합니다. 대신 Fastify 동등한 패키지를 사용해야합니다.

#### Redirect response

Fastify는 Express와 약간 다른 방식으로 리디렉션 응답을 처리합니다. Fastify로 적절한 리디렉션을 수행하려면 다음과 같이 상태 코드와 URL을 모두 반환합니다.

```typescript
@Get()
index(@Res() res) {
  res.status(302).redirect('/login');
}
```

#### Fastify options

You can pass options into the Fastify constructor through the `FastifyAdapter` constructor. For example:
`FastifyAdapter` 생성자를 통해 Fastify 생성자에 옵션을 전달할 수 있습니다. 예를 들면:

```typescript
new FastifyAdapter({ logger: true });
```

#### Example

작동하는 예는 [여기](https://github.com/nestjs/nest/tree/master/sample/10-fastify)에서 확인할 수 있습니다.
