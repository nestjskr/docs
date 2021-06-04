### Model-View-Controller

Nest는 기본적으로 [Express](https://github.com/expressjs/express) 라이브러리를 사용합니다. 따라서 Express에서 MVC(Model-View-Controller) 패턴을 사용하는 모든 기술은 Nest에도 적용됩니다.

먼저 [CLI](https://github.com/nestjs/nest-cli) 도구를 사용하여 간단한 Nest 애플리케이션을 스캐폴딩해 보겠습니다.

```bash
$ npm i -g @nestjs/cli
$ nest new project
```

MVC 앱을 만들려면 HTML 뷰를 렌더링할 [템플릿 엔진](https://expressjs.com/en/guide/using-template-engines.html)도 필요합니다.

```bash
$ npm install --save hbs
```

우리는 `hbs` ([Handlebars](https://github.com/pillarjs/hbs#readme)) 엔진을 사용했지만 요구 사항에 맞는 모든 것을 사용할 수 있습니다. 설치 프로세스가 완료되면 다음 코드를 사용하여 익스프레스 인스턴스를 구성해야합니다.

```typescript
@@filename(main)
import { NestFactory } from '@nestjs/core';
import { NestExpressApplication } from '@nestjs/platform-express';
import { join } from 'path';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(
    AppModule,
  );

  app.useStaticAssets(join(__dirname, '..', 'public'));
  app.setBaseViewsDir(join(__dirname, '..', 'views'));
  app.setViewEngine('hbs');

  await app.listen(3000);
}
bootstrap();
@@switch
import { NestFactory } from '@nestjs/core';
import { join } from 'path';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(
    AppModule,
  );

  app.useStaticAssets(join(__dirname, '..', 'public'));
  app.setBaseViewsDir(join(__dirname, '..', 'views'));
  app.setViewEngine('hbs');

  await app.listen(3000);
}
bootstrap();
```

[Express](https://github.com/expressjs/express)에 `public` 디렉토리는 정적 자산을 저장하는 데 사용되고 `views`에는 템플릿이 포함되며 `hbs` 템플릿 엔진을 사용하여 HTML 출력을 렌더링합니다.

#### Template rendering

이제 `views` 디렉토리와 `index.hbs` 템플릿을 그 안에 만들어 보겠습니다. 템플릿에서 컨트롤러에서 전달된 `message`를 인쇄합니다.

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>App</title>
  </head>
  <body>
    {{ "{{ message }\}" }}
  </body>
</html>
```

다음으로 `app.controller` 파일을 열고 `root()` 메서드를 다음 코드로 바꿉니다.

```typescript
@@filename(app.controller)
import { Get, Controller, Render } from '@nestjs/common';

@Controller()
export class AppController {
  @Get()
  @Render('index')
  root() {
    return { message: 'Hello world!' };
  }
}
```

이 코드에서는 `@Render()` 데코레이터에서 사용할 템플릿을 지정하고 라우트 핸들러 메서드의 반환값을 렌더링을 위해 템플릿에 전달합니다. 반환값은 템플릿에서 만든 `message` 자리 표시자와 일치하는 `message` 속성을 가진 객체입니다.

애플리케이션이 실행되는 동안 브라우저를 열고 `http://localhost:3000`으로 이동합니다. `Hello world!` 메시지가 표시됩니다.

#### Dynamic template rendering

애플리케이션 로직이 렌더링할 템플릿을 동적으로 결정해야하는 경우 `@Res()` 데코레이터를 사용하고 `@Render()` 데코레이터가 아닌 라우트 핸들러에 뷰 이름(view name)을 제공해야합니다.

> info **힌트** Nest가 `@Res()` 데코레이터를 감지하면 라이브러리별 `response` 객체를 삽입합니다. 이 객체를 사용하여 템플릿을 동적으로 렌더링할 수 있습니다. `response` 객체 API에 대한 자세한 내용은 [여기](https://expressjs.com/en/api.html)를 참조하세요.

```typescript
@@filename(app.controller)
import { Get, Controller, Res, Render } from '@nestjs/common';
import { Response } from 'express';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private appService: AppService) {}

  @Get()
  root(@Res() res: Response) {
    return res.render(
      this.appService.getViewName(),
      { message: 'Hello world!' },
    );
  }
}
```

#### Example

작동하는 예는 [여기](https://github.com/nestjs/nest/tree/master/sample/15-mvc)에서 확인할 수 있습니다.

#### Fastify

이 [장](/techniques/performance)에서 언급했듯이 Nest와 함께 호환되는 모든 HTTP 공급자를 사용할 수 있습니다. 이러한 라이브러리중 하나가 [Fastify](https://github.com/fastify/fastify)입니다. Fastify로 MVC 애플리케이션을 생성하려면 다음 패키지를 설치해야합니다.

```bash
$ npm i --save fastify-static point-of-view handlebars
```

다음 단계는 플랫폼에 따라 약간의 차이를 제외하고 Express에서 사용되는 거의 동일한 프로세스를 다룹니다. 설치 프로세스가 완료되면 `main.ts` 파일을 열고 내용을 업데이트합니다.

```typescript
@@filename(main)
import { NestFactory } from '@nestjs/core';
import { NestFastifyApplication, FastifyAdapter } from '@nestjs/platform-fastify';
import { AppModule } from './app.module';
import { join } from 'path';

async function bootstrap() {
  const app = await NestFactory.create<NestFastifyApplication>(
    AppModule,
    new FastifyAdapter(),
  );
  app.useStaticAssets({
    root: join(__dirname, '..', 'public'),
    prefix: '/public/',
  });
  app.setViewEngine({
    engine: {
      handlebars: require('handlebars'),
    },
    templates: join(__dirname, '..', 'views'),
  });
  await app.listen(3000);
}
bootstrap();
@@switch
import { NestFactory } from '@nestjs/core';
import { FastifyAdapter } from '@nestjs/platform-fastify';
import { AppModule } from './app.module';
import { join } from 'path';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, new FastifyAdapter());
  app.useStaticAssets({
    root: join(__dirname, '..', 'public'),
    prefix: '/public/',
  });
  app.setViewEngine({
    engine: {
      handlebars: require('handlebars'),
    },
    templates: join(__dirname, '..', 'views'),
  });
  await app.listen(3000);
}
bootstrap();
```

Fastify API는 약간 다르지만 이러한 메서드 호출의 최종 결과는 동일하게 유지됩니다. Fastify에서 주목해야할 한가지 차이점은 `@Render()` 데코레이터에 전달된 템플릿 이름에 파일 확장자가 포함되어야 한다는 것입니다.

```typescript
@@filename(app.controller)
import { Get, Controller, Render } from '@nestjs/common';

@Controller()
export class AppController {
  @Get()
  @Render('index.hbs')
  root() {
    return { message: 'Hello world!' };
  }
}
```

애플리케이션이 실행되는 동안 브라우저를 열고 `http://localhost:3000`으로 이동합니다. `Hello world!`메시지가 표시됩니다.

#### Example

작동하는 예는 [여기](https://github.com/nestjs/nest/tree/master/sample/17-mvc-fastify)에서 확인할 수 있습니다.
