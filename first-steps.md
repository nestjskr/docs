### First steps

이 문서에서는 Nest의 **핵심 기본사항**에 대해 알아봅니다. Nest 애플리케이션의 필수 구성요소에 익숙해지기 위해 입문수준에서 많은 부분을 다루는 기능이 포함된 기본 CRUD 애플리케이션을 빌드할 것입니다.

#### Language

우리는 [TypeScript](https://www.typescriptlang.org/)를 좋아하지만 무엇보다도 [Node.js](https://nodejs.org/en/)를 좋아합니다. 이것이 Nest가 TypeScript 및 **순수 자바스크립트**와 모두 호환되는 이유입니다. Nest는 최신 언어기능을 활용하므로 바닐라 자바스크립트와 함께 사용하려면 [Babel](https://babeljs.io/) 컴파일러가 필요합니다.

우리가 제공하는 예제에서는 대부분 TypeScript를 사용하지만 언제든지 **코드 스니펫을 바닐라 자바스크립트 구문으로 전환**할 수 있습니다(각 스니펫의 오른쪽 상단에 있는 언어 버튼을 클릭하여 전환하기만 하면 됩니다).

#### Prerequisites

운영체제에 [Node.js](https://nodejs.org/)(> 10.13.0, v13을 제외한)가 설치되어 있는지 확인하세요.

#### Setup

[Nest CLI](/cli/overview)를 사용하면 새 프로젝트를 설정하는 것이 매우 간단합니다. [npm](https://www.npmjs.com/)이 설치되어 있으면 OS 터미널에서 다음 명령을 사용하여 새 Nest 프로젝트를 만들 수 있습니다.

```bash
$ npm i -g @nestjs/cli
$ nest new project-name
```

`project` 디렉토리가 생성되고 노드 모듈과 몇가지 다른 상용구 파일이 설치되며 `src/` 디렉토리가 생성되고 여러 코어 파일로 채워집니다.

<div class="file-tree">
  <div class="item">src</div>
  <div class="children">
    <div class="item">app.controller.spec.ts</div>
    <div class="item">app.controller.ts</div>
    <div class="item">app.module.ts</div>
    <div class="item">app.service.ts</div>
    <div class="item">main.ts</div>
  </div>
</div>

다음은 이러한 핵심파일에 대한 간략한 개요입니다.

|                          |                                                                                                                     |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------- |
| `app.controller.ts`      | 하나의 라우트가 있는 기본 컨트롤러.                                                                             |
| `app.controller.spec.ts` | 컨트롤러를 위한 유닛 테스트                                                                                  |
| `app.module.ts`          | 애플리케이션의 루트 모듈                                                                                 |
| `app.service.ts`         | 단일 메소드를 사용하는 기본 서비스                                                                               |
| `main.ts`                | 핵심기능 `NestFactory`를 사용하여 Nest 애플리케이션 인스턴스를 생성하는 애플리케이션의 엔트리 파일입니다. |

`main.ts`에는 애플리케이션을 **부트스트랩**하는 비동기 함수가 포함되어 있습니다.

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

Nest 애플리케이션 인스턴스를 생성하기 위해 핵심 `NestFactory` 클래스를 사용합니다. `NestFactory`는 애플리케이션 인스턴스를 생성할 수 있는 몇가지 정적 메서드를 제공합니다. `create()` 메소드는 `INestApplication` 인터페이스를 충족하는 애플리케이션 객체를 반환합니다. 이 객체는 다음 장에서 설명하는 일련의 메서드를 제공합니다. 위의 `main.ts` 예제에서 우리는 HTTP 리스너를 시작하기만하면 애플리케이션이 인바운드 HTTP 요청을 기다릴 수 있습니다.

Nest CLI로 스캐폴딩된 프로젝트는 개발자가 각 모듈을 자체 전용 디렉터리에 보관하는 규칙을 따르도록 권장하는 초기 프로젝트 구조를 생성합니다.

<app-banner-courses></app-banner-courses>

#### Platform

Nest는 플랫폼에 구애받지 않는 프레임워크를 목표로 합니다. 플랫폼 독립성을 통해 개발자가 여러 유형의 애플리케이션에서 활용할 수 있는 재사용 가능한 논리적 부분을 만들 수 있습니다. 기술적으로 Nest는 어댑터가 생성되면 모든 Node HTTP 프레임워크에서 작동할 수 있습니다. 기본적으로 지원되는 HTTP 플랫폼은 [express](https://expressjs.com/) 및 [fastify](https://www.fastify.io)의 두가지입니다. 귀하의 요구에 가장 적합한 것을 선택할 수 있습니다.

|                    |                                                                                                                                                                                                                                                                                                                                    |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `platform-express` | [Express](https://expressjs.com/)는 노드용으로 잘 알려진 미니멀리스트 웹 프레임워크입니다. 커뮤니티에서 구현한 많은 리소스를 갖춘 수많은 테스트를 거친 프로덕션에 사용할 수 있도록 준비된 라이브러리입니다. 기본적으로 `@nestjs/platform-express` 패키지가 사용됩니다. 많은 사용자가 Express를 잘 사용하고 있으며 이를 활성화하기 위해 조치를 취할 필요가 없습니다. |
| `platform-fastify` | [Fastify](https://www.fastify.io/)는 최대의 효율성과 속도를 제공하는데 중점을 둔 고성능 및 낮은 오버헤드 프레임워크입니다. [여기](/techniques/performance)에서 사용 방법을 읽어보십시오.                                                                                                                                  |

어떤 플랫폼을 사용하든 자체 애플리케이션 인터페이스를 노출합니다. 이들은 각각 `NestExpressApplication` 및 `NestFastifyApplication`으로 표시됩니다.

아래 예제와 같이 `NestFactory.create()` 메소드에 유형을 전달하면 `app` 객체는 해당 특정 플랫폼에서만 사용할 수 있는 메소드를 갖게 됩니다. 그러나 실제로 기본 플랫폼 API에 액세스하려는 경우를 **제외**하고는 유형을 지정할 **필요**는 없습니다.

```typescript
const app = await NestFactory.create<NestExpressApplication>(AppModule);
```

#### Running the application

설치 프로세스가 완료되면 OS 명령 프롬프트에서 다음 명령을 실행하여 인바운드 HTTP 요청을 수신하는 애플리케이션을 시작할 수 있습니다.

```bash
$ npm run start
```

이 명령어는 `src/main.ts` 파일에 정의된 포트에서 수신대기하는 HTTP 서버로 앱을 시작합니다. 애플리케이션이 실행되면 브라우저를 열고 `http://localhost:3000/`로 이동합니다. `Hello World!`메시지가 표시됩니다.
