### Introduction

Nest (NestJS)는 효율적이고 확장 가능한 [Node.js](https://nodejs.org/) 서버측 애플리케이션을 구축하기 위한 프레임워크입니다. 프로그레시브 자바스크립트를 사용하고 [TypeScript](http://www.typescriptlang.org/)로 빌드되고 완벽하게 지원하며(하지만 여전히 개발자가 순수 자바스크립트로 코딩할 수 있음), OOP (객체 지향 프로그래밍 Object Oriented Programming), FP (함수형 프로그래밍 Functional Programming) 및 FRP (함수형 반응형 프로그래밍 Functional Reactive Programming) 요소를 결합합니다.

내부적으로 Nest는 [Express](https://expressjs.com/)(디폴트)와 같은 강력한 HTTP 서버 프레임워크를 사용하며 선택적으로 [Fastify](https://github.com/fastify/fastify)를 사용하도록 구성할 수도 있습니다.

Nest는 이러한 공통 Node.js 프레임워크(Express/Fastify)위에 추상화 수준을 제공하지만 API를 개발자에게 직접 노출합니다. 이를 통해 개발자는 기본 플랫폼에서 사용할 수 있는 수많은 타사 모듈을 자유롭게 사용할 수 있습니다.

#### Philosophy

최근 몇년동안 Node.js 덕분에 JavaScript는 프론트 및 백엔드 애플리케이션 모두를 위한 웹의 ["링구아 프랑카"](https://ko.wikipedia.org/wiki/%EB%A7%81%EA%B5%AC%EC%95%84_%ED%94%84%EB%9E%91%EC%B9%B4)가 되었습니다. 이로 인해 [Angular](https://angular.io/), [React](https://github.com/facebook/react) 및 [Vue](https://github.com)와 같은 멋진 프로젝트가 생겨서 개발자 생산성이 향상되고 빠르고 테스트 가능하며 확장 가능한 프런트엔드 애플리케이션을 만들 수 있습니다. 그러나 Node(및 서버측 자바스크립트)를 위한 훌륭한 라이브러리, 헬퍼 및 도구가 많이 존재하지만 이들중 어느것도 **아키텍처**의 주요 문제를 효과적으로 해결하지 못합니다.

Nest는 개발자와 팀이 고도로 테스트 가능하고 확장 가능하며 느슨하게 결합되고 유지관리가 쉬운 애플리케이션을 만들 수 있는 즉시 사용가능한 애플리케이션 아키텍처를 제공합니다. 아키텍처는 Angular에서 크게 영감을 받았습니다.

#### Installation

시작하려면 [Nest CLI](/cli/overview)를 사용하여 프로젝트를 스캐폴딩하거나 시작 프로젝트를 복제할 수 있습니다(둘 다 동일한 결과를 생성함).

Nest CLI로 프로젝트를 스캐폴드하려면 다음 명령어를 실행하세요. 이렇게하면 새 프로젝트 디렉토리가 생성되고 초기 핵심 Nest 파일 및 지원 모듈로 디렉토리가 채워져 프로젝트의 기존 기본구조가 생성됩니다. **Nest CLI**를 사용하여 새 프로젝트를 만드는 것은 처음 사용하는 사용자에게 권장됩니다. 이 접근방식은 [첫 번째 단계](/first-steps)에서 계속 진행합니다.

```bash
$ npm i -g @nestjs/cli
$ nest new project-name
```

#### Alternatives

또는 **Git**으로 TypeScript 시작 프로젝트를 설치하려면:

```bash
$ git clone https://github.com/nestjs/typescript-starter.git project
$ cd project
$ npm install
$ npm run start
```

브라우저를 열고 [`http://localhost:3000/`](http://localhost:3000/)로 이동합니다.

시작 프로젝트의 자바스크립트 버전을 설치하려면 위의 명령어 시퀀스에서 `javascript-starter.git`을 사용하세요.

**npm**(또는 **yarn**)으로 코어 및 지원파일을 설치하여 처음부터 새 프로젝트를 수동으로 만들 수도 있습니다. 물론 이 경우에는 프로젝트 상용구 파일을 직접 생성해야합니다.
```bash
$ npm i --save @nestjs/core @nestjs/common rxjs reflect-metadata
```
