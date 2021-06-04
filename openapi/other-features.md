### Other features

#### Global prefix

`setGlobalPrefix()`를 통해 설정된 경로에 대한 전역 접두사를 무시하려면 `ignoreGlobalPrefix`를 사용합니다.

```typescript
const document = SwaggerModule.createDocument(app, options, {
  ignoreGlobalPrefix: true,
});
```

#### Multiple specifications

`SwaggerModule`은 여러 사양을 지원하는 방법을 제공합니다. 즉, 서로 다른 엔드 포인트에서 서로 다른 UI를 사용하여 서로 다른 문서를 제공할 수 있습니다.

여러 사양을 지원하려면 애플리케이션을 모듈 방식으로 작성해야합니다. `createDocument()` 메소드는 `include`라는 속성을 가진 객체인 세번째 인수 `extraOptions`를 사용합니다. `include` 속성은 모듈의 배열인 값을 받습니다.

아래와 같이 여러 사양 지원을 설정할 수 있습니다.

```typescript
import { NestFactory } from '@nestjs/core';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
import { AppModule } from './app.module';
import { CatsModule } from './cats/cats.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  /**
   * createDocument(application, configurationOptions, extraOptions);
   *
   * createDocument method takes an optional 3rd argument "extraOptions"
   * which is an object with "include" property where you can pass an Array
   * of Modules that you want to include in that Swagger Specification
   * E.g: CatsModule and DogsModule will have two separate Swagger Specifications which
   * will be exposed on two different SwaggerUI with two different endpoints.
   */

  const options = new DocumentBuilder()
    .setTitle('Cats example')
    .setDescription('The cats API description')
    .setVersion('1.0')
    .addTag('cats')
    .build();

  const catDocument = SwaggerModule.createDocument(app, options, {
    include: [CatsModule],
  });
  SwaggerModule.setup('api/cats', app, catDocument);

  const secondOptions = new DocumentBuilder()
    .setTitle('Dogs example')
    .setDescription('The dogs API description')
    .setVersion('1.0')
    .addTag('dogs')
    .build();

  const dogDocument = SwaggerModule.createDocument(app, secondOptions, {
    include: [DogsModule],
  });
  SwaggerModule.setup('api/dogs', app, dogDocument);

  await app.listen(3000);
}
bootstrap();
```

이제 다음 명령을 사용하여 서버를 시작할 수 있습니다.

```bash
$ npm run start
```

고양이용 Swagger UI를 보려면 `http://localhost:3000/api/cats`로 이동하십시오.

<figure><img src="/assets/swagger-cats.png" /></figure>

차례로 `http://localhost:3000/api/dogs`는 강아지용 Swagger UI를 노출합니다.

<figure><img src="/assets/swagger-dogs.png" /></figure>
