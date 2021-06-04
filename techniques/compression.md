### Compression

압축은 응답 본문의 크기를 크게 줄여 웹 앱의 속도를 높일 수 있습니다.

프로덕션 환경에서 **트래픽이 많은** 웹 사이트의 경우 애플리케이션 서버(일반적으로 역방향 프록시 (예: Nginx))에서 압축을 오프로드하는 것이 좋습니다. 이 경우 압축 미들웨어를 사용해서는 안됩니다.

#### Use with Express (default)

[압축](https://github.com/expressjs/compression) 미들웨어 패키지를 사용하여 gzip 압축을 활성화합니다.

먼저 필요한 패키지를 설치하십시오.

```bash
$ npm i --save compression
```

설치가 완료되면 압축 미들웨어를 글로벌 미들웨어로 적용하십시오.

```typescript
import * as compression from 'compression';
// somewhere in your initialization file
app.use(compression());
```

#### Use with Fastify

`FastifyAdapter`를 사용하는 경우 [fastify-compress](https://github.com/fastify/fastify-compress)를 사용하는 것이 좋습니다.

```bash
$ npm i --save fastify-compress
```

설치가 완료되면 fastify-compress 미들웨어를 글로벌 미들웨어로 적용하십시오.

```typescript
import compression from 'fastify-compress';
// somewhere in your initialization file
app.register(compression);
```

기본적으로 fastify-compress는 브라우저가 인코딩 지원을 표시할 때 Brotli 압축(노드 >= 11.7.0)을 사용합니다. Brotli는 압축 비율 측면에서 상당히 효율적이지만 속도도 상당히 느립니다. 이로 인해 fastify-compress에 deflate 및 gzip 만 사용하여 응답을 압축하도록 지시할 수 있습니다. 더 큰 응답으로 끝나지만 훨씬 더 빨리 전달됩니다.

인코딩을 지정하려면 `app.register`에 두번째 인수를 제공하세요.

```typescript
app.register(compression, { encodings: ['gzip', 'deflate'] });
```

위의 내용은 `fastify-compress`에 gzip 및 deflate 인코딩만 사용하도록 지시하며 클라이언트가 둘 다 지원하는 경우 gzip을 선호합니다.
