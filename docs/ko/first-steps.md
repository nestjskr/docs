### 첫 번째 단계

이 문서에서는 Nest의 **핵심 기본**(core fundamentals)에 대해 배우게 됩니다. Nest 애플리케이션의 필수 구성 요소들(essential building blocks)에 익숙해지기 위해, 많은 입문 수준의 기능을 포함한 CRUD 애플리케이션을 만들 것입니다.

#### 언어(Language)

우리는 [TypeScript](https://www.typescriptlang.org/)와 사랑에 빠졌습니다만 무엇보다도 [Node.js](https://nodejs.org/en/)를 사랑합니다. 이것이 Nest가 TypeScript**와 순수 JavaScript**와 호환되는 이유입니다. Nest는 최신 언어 기능을 활용하므로 바닐라 JavaScript와 함께 사용하려면 [Babel](https://babeljs.io/) 컴파일러가 필요합니다.

우리가 제공하는 예제에서는 대부분 TypeScript를 사용하지만 언제든지 **코드 스니펫**을 바닐라 JavaScript 문법으로 전환 할 수 있습니다 (각 스니펫의 오른쪽 상단에있는 언어 버튼을 클릭하여 전환하면 됩니다).

#### 전제조건

운영체제에 [Node.js](https://nodejs.org/) (&gt;=10.13.0)가 설치되어 있어야 합니다.

#### 설치

[npm이](https://www.npmjs.com/)이 설치되어 있다면, [Nest CLI를](/cli/overview)를 사용하여 새 프로젝트를 시작하는 것이 매우 간단합니다. OS 터미널에서 아래 명령을 사용하여 새 Nest 프로젝트를 만들 수 있습니다.

```bash
$ npm i -g @nestjs/cli
$ nest new project-name
```

`project` 디렉토리가 생성되고, 노드 모듈들과 몇 가지 다른 보일러플레이트 파일이 설치되며, `src/` 디렉토리가 생성되고 여러 핵심 파일들이 생성됩니다.

<div class="file-tree">
  <div class="item">src</div>
  <div class="children">
    <div class="item">app.controller.ts</div>
    <div class="item">app.controller.spec.ts</div>
    <div class="item">app.module.ts</div>
    <div class="item">app.service.ts</div>
    <div class="item">main.ts</div>
  </div>
</div>

다음은 핵심 파일에 대한 간략한 설명입니다.

 |
--- | ---
`app.controller.ts` | A basic controller with a single route.
`app.controller.spec.ts` | The unit tests for the controller.
`app.module.ts` | The root module of the application.
`app.service.ts` | A basic service with a single method.
`main.ts` | The entry file of the application which uses the core function `NestFactory` to create a Nest application instance.

`main.ts`는 애플리케이션을 부트 스트랩하는 비동기(async) 함수를 포함하고 있습니다.

```typescript
@@filename(main)

import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
@@switch
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

Nest 애플리케이션 인스턴스를 생성하기 위해 우리는 핵심 `NestFactory` 클래스를 사용합니다. `NestFactory` 는 애플리케이션 인스턴스 생성을 허용하는 몇 가지 정적 메서드(static methods)를 제공합니다. `create()` 메서드는 `INestApplication` 인터페이스(interface)를 충족하는 응용 프로그램 객체를 반환합니다. 이 객체는 다음 장에서 설명하는 일련의 메서드들(methods)을 제공합니다. 위의 `main.ts` 예제에서는 인바운드(inbound) HTTP 요청을 기다리는 HTTP 리스너를 만들었습니다.

Nest CLI로 만들어진(scaffolded) 프로젝트는 각 모듈을 전용 디렉터리에 보관하는 규칙을 제시하고 있기 때문에 개발자들이 그 규칙을 따르기를 권장합니다.

<app-banner-courses></app-banner-courses>

#### 플랫폼(Platform)

Nest는 플랫폼 지식 없이도 구동할 수 있는(platform-agnostic) 프레임워크를 지향합니다. 플랫폼으로부터의 독립성(platform independence)은 개발자들이 여러 타입의 애플리케이션에서 재사용할 수 있는 논리파츠(reusable logical parts)를 만드는 것이 가능하도록 합니다. 엄밀히 말하자면, Nest는 어답터(adapter)가 생성되면 어떤 Node HTTP 프레임워크와도 동작합니다. 바로 사용할 수 있는 HTTP 플랫폼은 [express](https://expressjs.com/)와 [fastify](https://www.fastify.io) 입니다. 필요에 따라 맞는 플랫폼을 선택하면 됩니다.

 |
--- | ---
`platform-express` | [Express](https://expressjs.com/) is a well-known minimalist web framework for node. It's a battle tested, production-ready library with lots of resources implemented by the community. The `@nestjs/platform-express` package is used by default. Many users are well served with Express, and need take no action to enable it.
`platform-fastify` | [Fastify](https://www.fastify.io/) is a high performance and low overhead framework highly focused on providing maximum efficiency and speed. Read how to use it [here](/techniques/performance).

어떤 플랫폼을 사용하든, Nest는 자체 애플리케이션 인터페이스를 생성합니다. `NestExpressApplication` 또는 `NestFastifyApplication` 입니다.

아래 예제에서와 같이 `NestFactory.create()` 메서드에 유형(type)을 전달하면, `app` 객체는 특정 플랫폼에서만 사용 가능한 메서드(methods)를 갖게됩니다.<br>그러나 실제로 기본 플랫폼 API에 액세스하려는 **경우가 아니면** 유형(type)을 지정할 **필요** 가 없습니다.

```typescript
const app = await NestFactory.create<NestExpressApplication>(AppModule);
```

#### 애플리케이션 실행

설치가 완료되면, 운영체제(OS)의 명령 프롬프트에서 다음 명령을 실행하여 인바운드 HTTP 요청을 수신하는 애플리케이션을 실행할 수 있습니다.

```bash
$ npm run start
```

이 명령은 `src/main.ts` 파일에 명시된 포트(port)에 HTTP 서버를 실행합니다. 애플리케이션잉 실행되면, 브라우저를 열어 `http://localhost:3000/`에 접속하세요. `Hello World!` 메세지를 볼 수 있을거에요!
