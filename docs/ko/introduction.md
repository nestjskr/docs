### 소개

Nest (NestJS)는 효율적이고 확장 가능한 [Node.js](https://nodejs.org/) 서버 사이드 애플리케이션을 구축하기위한 프레임워크입니다. 최신 자바스크립트 또는 [TypeScript](http://www.typescriptlang.org/)를 완전히 지원하고 사용합니다. (순수 JavasScript만으로도 호환이 가능합니다.) 그리고 OOP(객체 지향 프로그래밍), FP(함수형 프로그래밍), 및 FRP(함수형 리액티브 프로그래밍)를 결합하고 있습니다.

내부적으로, Nest는 HTTP 서버 프레임워크로 강력한 [Express](https://expressjs.com/)를 사용하고 있으며, 선택적으로 [Fastify](https://github.com/fastify/fastify)를 사용하여 구성 할 수도 있습니다.

Nest는 이러한 공통 Node.js 프레임워크 (Express / Fastify) 위에 추상화 레벨을 제공하며 API를 개발자에게 직접 제공합니다. 이를 통해 개발자는 기본 플랫폼에서 사용할 수있는 다양한 서드파티 모듈을 자유롭게 사용할 수 있습니다.

#### 철학

최근 몇 년 동안 Node.js 덕분에 JavaScript는 프론트엔드 그리고 백엔드 영역에서 하나의 공용어가 되었습니다. 이로 인해 [Angluar](https://angular.io/), [React](https://github.com/facebook/react), [Vue](https://github.com/vuejs/vue)와 같은 멋진 프레임워크들이 탄생하게 되었고, 개발자의 생산성을 향상시키고 빠르게 테스트가 가능하며 확장 가능한 프론트엔드 애플리케이션을 만들 수 있게 되었습니다. 그리하여 우수한 라이브러리 및 헬퍼, 툴은 많이 등장하였지만, 아쉽게도 **아키텍처**와 연관된 문제를 효과적으로 해결하기 위한 라이브러리는 드문실정입니다.

Nest는 개발자와 팀이 고도로 테스트 가능하고 확장 가능하며 느슨한 결합(loosely coupled)을 유지하며, 쉽게 유지 관리 할 수있는 애플리케이션을 만들 수있는, 즉시 사용 가능한 애플리케이션 아키텍처를 제공합니다. 아키텍처는 Angular에서 크게 영감을 받았습니다.

#### 설치

새로운 프로젝트를 시작하려면, [Nest CLI](/cli/overview)를 사용하여 프로젝트를 스캐폴딩하거나, 스타터 프로젝트를 복제하여 사용할 수 있습니다 (두가지의 방법은 동일한 결과물을 생성합니다).

Nest CLI로 프로젝트를 스캐폴딩하려면 아래의 명령어를 실행하세요. 이렇게하면 새 프로젝트 디렉토리가 생성되고 기본적인 소스 파일 및 모듈이 설치됩니다. Nest를 처음 사용한다면 **Nest CLI**로 새 프로젝트를 만드는 것을 추천합니다. 이 방법에 대한 자세한 설명은 [시작하기](first-steps)에서 계속됩니다.

```bash
$ npm i -g @nestjs/cli
$ nest new project-name
```

#### 대안

Nest CLI를 통한 설치가 꺼려진다면, **Git**을 사용하여 새로운 프로젝트를 시작할 수 있습니다. TypeScript 스사터 프로젝트를 설치하려면 아래의 명령어를 실행하세요.

```bash
$ git clone https://github.com/nestjs/typescript-starter.git 프로젝트명
$ cd 프로젝트명
$ npm install
$ npm run start
```

실행이 정상적으로 되면, 주로 사용하는 웹브라우저를 열고 [`http://localhost:3000/`](http://localhost:3000/)로 이동합니다.

만약, TypeScript가 아닌 JavaScript 스타터 프로젝트를 설치하려면 위의 명령어에서 레포지토리명을 `javascript-starter.git`으로 변경하여 사용하세요.

선택적으로, **npm** (또는 **yarn** )을 사용하여 코어 및 구성 패키지들을 직접 설치하여 새 프로젝트를 수동으로 만들 수도 있습니다. 이 경우에는 프로젝트 기본 구성 파일들은 직접 생성해야합니다.

```bash
$ npm i --save @nestjs/core @nestjs/common rxjs reflect-metadata
```
