### Introduction

[OpenAPI](https://swagger.io/specification/) 사양은 RESTful API를 설명하는 데 사용되는 언어에 구애받지 않는 정의 형식입니다. Nest는 데코레이터를 활용하여 이러한 사양을 생성할 수 있는 전용 [모듈](https://github.com/nestjs/swagger)을 제공합니다.

#### Installation

사용을 시작하려면 먼저 필요한 종속성을 설치합니다.

```bash
$ npm install --save @nestjs/swagger swagger-ui-express
```

fastify를 사용하는 경우 `swagger-ui-express` 대신 `fastify-swagger`를 설치합니다.

```bash
$ npm install --save @nestjs/swagger fastify-swagger
```

#### Bootstrap

설치 프로세스가 완료되면 `main.ts` 파일을 열고 `SwaggerModule` 클래스를 사용하여 Swagger를 초기화합니다.

```typescript
@@filename(main)
import { NestFactory } from '@nestjs/core';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const config = new DocumentBuilder()
    .setTitle('Cats example')
    .setDescription('The cats API description')
    .setVersion('1.0')
    .addTag('cats')
    .build();
  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api', app, document);

  await app.listen(3000);
}
bootstrap();
```

> info **힌트** `document`(`SwaggerModule#createDocument()` 메소드에서 반환됨)는 [OpenAPI Document](https://swagger.io/specification/#openapi-document)를 준수하는 직렬화 가능한 객체입니다. HTTP를 통해 호스팅하는 대신 JSON/YAML 파일로 저장하고 다른 방식으로 사용할 수도 있습니다.

`DocumentBuilder`는 OpenAPI 사양을 준수하는 기본 문서를 구성하는데 도움이됩니다. 제목, 설명, 버전 등과 같은 속성을 설정할 수 있는 여러 메서드를 제공합니다. 전체 문서(모든 HTTP 경로가 정의됨)를 만들기 위해 `SwaggerModule` 클래스의 `createDocument()` 메서드를 사용합니다. 이 메소드는 애플리케이션 인스턴스와 Swagger 옵션 오브젝트라는 두개의 인수를 사용합니다. 또는 `SwaggerDocumentOptions` 타입이어야 하는 세번째 인수를 제공할 수 있습니다. 자세한 내용은 [문서 옵션 섹션](/openapi/introduction#document-options)에서 확인하세요.

문서를 생성하면 `setup()` 메서드를 호출할 수 있습니다. 다음을 허용합니다.

1. Swagger UI를 마운트하는 경로
2. 애플리케이션 인스턴스
3. 위에서 인스턴스화된 문서 객체
4. 선택적 구성 매개변수(자세히 알아보기 [여기](/openapi/introduction#document-options))

이제 다음 명령을 실행하여 HTTP 서버를 시작할 수 있습니다.

```bash
$ npm run start
```

애플리케이션이 실행되는 동안 브라우저를 열고 `http://localhost:3000/api`로 이동합니다. Swagger UI가 표시되어야 합니다.

<figure><img src="/assets/swagger1.png" /></figure>

`SwaggerModule`은 모든 엔드포인트를 자동으로 반영합니다. Swagger UI는 플랫폼에 따라 `swagger-ui-express` 또는 `fastify-swagger`를 사용하여 생성됩니다.

> info **힌트** Swagger JSON 파일을 생성하고 다운로드하려면 `http://localhost:3000/api-json` (`swagger-ui-express`)로 이동합니다. 또는 브라우저의 `http://localhost:3000/api/json` (`fastify-swagger`) (Swagger 문서가 `http://localhost:3000/api`에서 사용 가능하다고 가정).

> warning **경고** `fastify-swagger` 및 `helmet` 사용시 [CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)에 문제가 있을 수 있습니다, 이 충돌을 해결하려면 아래와 같이 CSP를 구성하십시오.
>
> ```typescript
> app.register(helmet, {
>   contentSecurityPolicy: {
>     directives: {
>       defaultSrc: [`'self'`],
>       styleSrc: [`'self'`, `'unsafe-inline'`],
>       imgSrc: [`'self'`, 'data:', 'validator.swagger.io'],
>       scriptSrc: [`'self'`, `https: 'unsafe-inline'`],
>     },
>   },
> });
>
> // If you are not going to use CSP at all, you can use this:
> app.register(helmet, {
>   contentSecurityPolicy: false,
> });
> ```

#### Document options

문서를 만들 때 라이브러리의 동작을 미세 조정하기 위한 몇가지 추가 옵션을 제공할 수 있습니다. 이러한 옵션은 `SwaggerDocumentOptions` 타입이어야 하며 다음과 같을 수 있습니다.

```TypeScript
export interface SwaggerDocumentOptions {
  /**
   * List of modules to include in the specification
   */
  include?: Function[];

  /**
   * Additional, extra models that should be inspected and included in the specification
   */
  extraModels?: Function[];

  /**
   * If `true`, swagger will ignore the global prefix set through `setGlobalPrefix()` method
   */
  ignoreGlobalPrefix?: boolean;

  /**
   * If `true`, swagger will also load routes from the modules imported by `include` modules
   */
  deepScanRoutes?: boolean;

  /**
   * Custom operationIdFactory that will be used to generate the `operationId`
   * based on the `controllerKey` and `methodKey`
   * @default () => controllerKey_methodKey
   */
  operationIdFactory?: (controllerKey: string, methodKey: string) => string;
}
```

예를 들어 라이브러리에서 `UserController_createUser` 대신 `createUser`와 같은 작업 이름을 생성하도록 하려면 다음을 설정할 수 있습니다.

```TypeScript
const options: SwaggerDocumentOptions =  {
  operationIdFactory: (
    controllerKey: string,
    methodKey: string
  ) => methodKey
};
const document = SwaggerModule.createDocument(app, config, options);
```

#### Setup options

`SwaggerCustomOptions` 인터페이스를 충족하는 옵션 객체를 `SwaggerModule#setup` 메소드의 네번째 인수로 전달하여 Swagger UI를 구성할 수 있습니다.

```TypeScript
export interface SwaggerCustomOptions {
  explorer?: boolean;
  swaggerOptions?: Record<string, any>;
  customCss?: string;
  customCssUrl?: string;
  customJs?: string;
  customfavIcon?: string;
  swaggerUrl?: string;
  customSiteTitle?: string;
  validatorUrl?: string;
  url?: string;
  urls?: Record<'url' | 'name', string>[];
}
```

예를 들어 페이지를 새로 고친 후에도 인증 토큰이 유지되는지 확인하거나 페이지 제목(브라우저에 표시됨)을 변경하려는 경우 다음 설정을 사용할 수 있습니다.

```TypeScript
const customOptions: SwaggerCustomOptions = {
  swaggerOptions: {
    persistAuthorization: true,
  },
  customSiteTitle: 'My API Docs',
};
SwaggerModule.setup('docs', app, document, customOptions);
```

#### Example

작동하는 예는 [여기](https://github.com/nestjs/nest/tree/master/sample/11-swagger)에서 확인할 수 있습니다.
