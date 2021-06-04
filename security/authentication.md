### Authentication

인증은 대부분의 애플리케이션에서 **필수** 부분입니다. 인증을 처리하기 위한 다양한 접근 방식과 전략이 있습니다. 모든 프로젝트에 적용되는 접근 방식은 특정 애플리케이션 요구사항에 따라 다릅니다. 이 장에서는 다양한 요구사항에 적용할 수 있는 인증에 대한 몇가지 접근 방식을 제공합니다.

[Passport](https://github.com/jaredhanson/passport)는 커뮤니티에서 잘 알려진 가장 인기있는 node.js 인증 라이브러리로 많은 프로덕션 애플리케이션에서 성공적으로 사용되었습니다. `@nestjs/passport` 모듈을 사용하여 이 라이브러리를 **Nest** 애플리케이션과 통합하는 것은 간단합니다. 높은 수준에서 Passport는 다음을 위해 일련의 단계를 실행합니다.

- "자격 증명" (예: 사용자 이름/암호, JSON 웹 토큰 ([JWT](https://jwt.io/)) 또는 아이덴티티 제공업체의 아이덴티티 토큰)을 확인하여 사용자를 인증합니다.
- 인증 상태 관리 (JWT와 같은 포터블 토큰 발행 또는 [Express 세션](https://github.com/expressjs/session) 생성)
- 인증된 사용자에 대한 정보를 라우트 핸들러에서 나중에 사용할 수 있도록 `Request` 객체에 첨부합니다.

Passport는 다양한 인증 메커니즘을 구현하는 [전략](http://www.passportjs.org/)의 풍부한 생태계를 가지고 있습니다. 개념은 간단하지만 선택할 수 있는 Passport 전략 세트는 크고 다양합니다. Passport는 이러한 다양한 단계를 표준 패턴으로 추상화하고 `@nestjs/passport` 모듈은 이 패턴을 익숙한 Nest 구성으로 래핑하고 표준화합니다.

이 장에서는 강력하고 유연한 모듈을 사용하여 RESTful API 서버를 위한 완전한 종단간 인증 솔루션을 구현합니다. 여기에 설명된 개념을 사용하여 Passport 전략을 구현하여 인증 체계를 사용자 지정할 수 있습니다. 이 완전한 예제를 빌드하려면 이 장의 단계를 따르십시오. 완성된 샘플 앱이 있는 저장소는 [여기](https://github.com/nestjs/nest/tree/master/sample/19-auth-jwt)에서 찾을 수 있습니다.


#### Authentication requirements

우리의 요구 사항을 구체화합시다. 이 사용 사례의 경우 클라이언트는 사용자 이름과 암호로 인증을 시작합니다. 인증되면 서버는 인증을 증명하기 위한 후속 요청에서 [인증 헤더의 전달자 토큰](https://tools.ietf.org/html/rfc6750)으로 전송할 수 있는 JWT를 발행합니다. 또한 유효한 JWT를 포함하는 요청에만 액세스할 수 있는 보호 경로를 생성합니다.

첫번째 요구 사항인 사용자 인증부터 시작하겠습니다. 그런 다음 JWT를 발행하여 확장합니다. 마지막으로 요청에서 유효한 JWT를 확인하는 보호 경로를 생성합니다.

먼저 필요한 패키지를 설치해야 합니다. Passport는 사용자 이름/비밀번호 인증 메커니즘을 구현하는 [passport-local](https://github.com/jaredhanson/passport-local)이라는 전략을 제공합니다.

```bash
$ npm install --save @nestjs/passport passport passport-local
$ npm install --save-dev @types/passport-local
```

> warning **알림** 선택한 **모든** Passport 전략의 경우 항상 `@nestjs/passport` 및 `passport` 패키지가 필요합니다. 그런 다음 구축중인 특정 인증 전략을 구현하는 전략별 패키지 (예: `passport-jwt` 또는 `passport-local`)를 설치해야 합니다. 또한 TypeScript 코드를 작성하는 동안 도움을 제공하는 `@types/passport-local`을 사용하여 위에 표시된대로 모든 Passport 전략에 대한 타입 정의를 설치할 수도 있습니다.

#### Implementing Passport strategies

이제 인증 기능을 구현할 준비가 되었습니다. **모든** Passport 전략에 사용되는 프로세스의 개요부터 시작하겠습니다. Passport를 그 자체로 미니 프레임워크로 생각하면 도움이 됩니다. 프레임워크의 장점은 구현중인 전략에 따라 사용자 지정하는 몇가지 기본 단계로 인증 프로세스를 추상화한다는 것입니다. 사용자 지정 매개 변수 (일반 JSON 개체로)와 Passport가 적절한 시간에 호출하는 콜백 함수의 형태로 사용자 지정 코드를 제공하여 구성하기 때문에 프레임워크와 같습니다. `@nestjs/passport` 모듈은 이 프레임워크를 Nest 스타일 패키지로 래핑하여 Nest 애플리케이션에 쉽게 통합할 수 있습니다. 아래에서는 `@nestjs/passport`를 사용하지만 먼저 **vanilla Passport**의 작동 방식을 고려해 보겠습니다.

vanilla Passport에서는 다음 두 가지를 제공하여 전략을 구성합니다:

1. 해당 전략에 특정한 옵션 세트. 예를 들어 JWT 전략에서 토큰에 서명하기 위한 비밀을 제공할 수 있습니다.
2. "확인 콜백": Passport에 사용자 저장소 (사용자 계정을 관리하는 곳)와 상호 작용하는 방법을 알려줍니다. 여기에서 사용자의 존재 여부 (및/또는 새 사용자 생성) 및 해당 자격 증명이 유효한지 확인합니다. Passport 라이브러리는 이 콜백이 유효성 검사에 성공하면 전체 사용자를 반환하고 실패하면 null을 반환할 것으로 예상합니다 (실패는 사용자를 찾을 수 없거나 passport-local의 경우 암호가 일치하지 않는 것으로 정의 됨). .

`@nestjs/passport`를 사용하면 `PassportStrategy` 클래스를 확장하여 Passport 전략을 구성합니다. 서브 클래스에서 `super ()`메소드를 호출하고 선택적으로 옵션 객체를 전달하여 전략 옵션 (위 항목 1)을 전달합니다. 하위 클래스에서 `validate()`메서드를 구현하여 확인 콜백 (위 항목 2)을 제공합니다.

`AuthModule`과 그 안에 `AuthService`를 생성하는 것으로 시작합니다.

```bash
$ nest g module auth
$ nest g service auth
```

`AuthService`를 구현할 때 `UsersService`에 사용자 작업을 캡슐화하는 것이 유용하다는 것을 알게될 것이므로 이제 해당 모듈과 서비스를 생성해 보겠습니다.

```bash
$ nest g module users
$ nest g service users
```

아래와 같이 생성된 파일의 기본 내용을 바꿉니다. 샘플 앱의 경우 `UsersService`는 하드 코딩된 메모리 내 사용자 목록과 사용자 이름별로 검색하는 find 메서드를 유지합니다. 실제 앱에서는 선택한 라이브러리 (예: TypeORM, Sequelize, Mongoose 등)를 사용하여 사용자 모델과 지속성 레이어를 빌드하는 곳입니다.

```typescript
@@filename(users/users.service)
import { Injectable } from '@nestjs/common';

// This should be a real class/interface representing a user entity
export type User = any;

@Injectable()
export class UsersService {
  private readonly users = [
    {
      userId: 1,
      username: 'john',
      password: 'changeme',
    },
    {
      userId: 2,
      username: 'maria',
      password: 'guess',
    },
  ];

  async findOne(username: string): Promise<User | undefined> {
    return this.users.find(user => user.username === username);
  }
}
@@switch
import { Injectable } from '@nestjs/common';

@Injectable()
export class UsersService {
  constructor() {
    this.users = [
      {
        userId: 1,
        username: 'john',
        password: 'changeme',
      },
      {
        userId: 2,
        username: 'maria',
        password: 'guess',
      },
    ];
  }

  async findOne(username) {
    return this.users.find(user => user.username === username);
  }
}
```

`UsersModule`에서 필요한 유일한 변경 사항은 `@Module` 데코레이터의 내보내기 배열에 `UsersService`를 추가하여 이 모듈 외부에서 볼 수 있도록 하는 것입니다 (곧 `AuthService`에서 사용할 예정입니다).

```typescript
@@filename(users/users.module)
import { Module } from '@nestjs/common';
import { UsersService } from './users.service';

@Module({
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
@@switch
import { Module } from '@nestjs/common';
import { UsersService } from './users.service';

@Module({
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

우리의 `AuthService`는 사용자를 검색하고 암호를 확인하는 작업을 수행합니다. 이를 위해 `validateUser()`메소드를 생성합니다. 아래 코드에서 편리한 ES6 스프레드 연산자를 사용하여 사용자 개체에서 암호 속성을 제거하고 반환합니다. 잠시후 Passport 로컬 전략에서 `validateUser()` 메서드를 호출할 것입니다.

```typescript
@@filename(auth/auth.service)
import { Injectable } from '@nestjs/common';
import { UsersService } from '../users/users.service';

@Injectable()
export class AuthService {
  constructor(private usersService: UsersService) {}

  async validateUser(username: string, pass: string): Promise<any> {
    const user = await this.usersService.findOne(username);
    if (user && user.password === pass) {
      const { password, ...result } = user;
      return result;
    }
    return null;
  }
}
@@switch
import { Injectable, Dependencies } from '@nestjs/common';
import { UsersService } from '../users/users.service';

@Injectable()
@Dependencies(UsersService)
export class AuthService {
  constructor(usersService) {
    this.usersService = usersService;
  }

  async validateUser(username, pass) {
    const user = await this.usersService.findOne(username);
    if (user && user.password === pass) {
      const { password, ...result } = user;
      return result;
    }
    return null;
  }
}
```

> warning **경고** 물론 실제 애플리케이션에서는 암호를 일반 텍스트로 저장하지 않습니다. 대신 솔티드(salted) 단방향 해시 알고리즘과 함께 [bcrypt](https://github.com/kelektiv/node.bcrypt.js#readme)와 같은 라이브러리를 사용합니다. 이 접근 방식을 사용하면 해시된 암호만 저장한 다음 저장된 암호를 **수신** 암호의 해시된 버전과 비교하므로 사용자 암호를 일반 텍스트로 저장하거나 노출하지 않습니다. 샘플 앱을 단순하게 유지하기 위해 절대적인 명령을 위반하고 일반 텍스트를 사용합니다. **실제 앱에서는 이 작업을 수행하지 마세요!**

이제 `AuthModule`을 업데이트하여 `UsersModule`을 가져옵니다.

```typescript
@@filename(auth/auth.module)
import { Module } from '@nestjs/common';
import { AuthService } from './auth.service';
import { UsersModule } from '../users/users.module';

@Module({
  imports: [UsersModule],
  providers: [AuthService],
})
export class AuthModule {}
@@switch
import { Module } from '@nestjs/common';
import { AuthService } from './auth.service';
import { UsersModule } from '../users/users.module';

@Module({
  imports: [UsersModule],
  providers: [AuthService],
})
export class AuthModule {}
```

#### Implementing Passport local

이제 Passport **로컬 인증 전략**을 구현할 수 있습니다. `auth` 폴더에 `local.strategy.ts`라는 파일을 만들고 다음 코드를 추가합니다.

```typescript
@@filename(auth/local.strategy)
import { Strategy } from 'passport-local';
import { PassportStrategy } from '@nestjs/passport';
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { AuthService } from './auth.service';

@Injectable()
export class LocalStrategy extends PassportStrategy(Strategy) {
  constructor(private authService: AuthService) {
    super();
  }

  async validate(username: string, password: string): Promise<any> {
    const user = await this.authService.validateUser(username, password);
    if (!user) {
      throw new UnauthorizedException();
    }
    return user;
  }
}
@@switch
import { Strategy } from 'passport-local';
import { PassportStrategy } from '@nestjs/passport';
import { Injectable, UnauthorizedException, Dependencies } from '@nestjs/common';
import { AuthService } from './auth.service';

@Injectable()
@Dependencies(AuthService)
export class LocalStrategy extends PassportStrategy(Strategy) {
  constructor(authService) {
    super();
    this.authService = authService;
  }

  async validate(username, password) {
    const user = await this.authService.validateUser(username, password);
    if (!user) {
      throw new UnauthorizedException();
    }
    return user;
  }
}
```

우리는 모든 Passport 전략에 대해 앞서 설명한 레시피를 따랐습니다. passport-local의 사용 사례에는 구성 옵션이 없으므로 생성자는 옵션 객체없이 단순히 `super()`를 호출합니다.

> info **힌트** `super()` 호출에서 옵션 객체를 전달하여 패스포트 전략의 동작을 사용자 지정할 수 있습니다. 이 예에서 passport-local 전략은 기본적으로 요청 본문에 `username` 및 `password`라는 속성을 예상합니다. 옵션 객체를 전달하여 다른 속성 이름을 지정합니다 예: `super({{ '{' }} usernameField: 'email'{{ '}' }})`. 자세한 내용은 [패스포트 문서](http://www.passportjs.org/docs/configure/)를 참조하십시오.

`validate()` 메소드도 구현했습니다. 각 전략에 대해 Passport는 적절한 전략별 매개변수 세트를 사용하여 확인함수 (`@nestjs/passport`의 `validate()` 메소드로 구현됨)를 호출합니다. 로컬 전략의 경우 Passport는 서명이 `validate(username: string, password: string): any`인 `validate()` 메소드를 예상합니다.

대부분의 유효성 검사 작업은 `AuthService` (`UsersService`의 도움으로)에서 수행되므로 이 방법은 매우 간단합니다. **모든** Passport 전략에 대한 `validate()` 메서드는 자격증명이 표시되는 방식에 대한 세부사항 만 다른 유사한 패턴을 따릅니다. 사용자가 발견되고 자격증명이 유효하면 사용자가 반환되어 Passport가 작업을 완료하고 (예: `Request` 객체에 `user` 속성 생성) 요청 처리 파이프 라인을 계속할 수 있습니다. 찾을 수 없는 경우 예외(exception)를 던지고(throw) <a href="exception-filters">예외 레이어</a>가 처리하도록 합니다.

일반적으로 각 전략의 `validate()` 메서드에서 유일한 중요한 차이점은 사용자가 존재하고 유효한지 **어떻게** 결정하는 것입니다. 예를 들어 JWT 전략에서 요구 사항에 따라 디코딩된 토큰에 포함된 `userId`가 사용자 데이터베이스의 레코드와 일치하는지 아니면 취소된 토큰 목록과 일치하는지 평가할 수 있습니다. 따라서 이러한 하위 분류 및 전략별 유효성 검사 구현 패턴은 일관되고 우아하며 확장 가능합니다.

방금 정의한 Passport 기능을 사용하려면 `AuthModule`을 구성해야 합니다. `auth.module.ts`를 다음과 같이 업데이트하십시오.

```typescript
@@filename(auth/auth.module)
import { Module } from '@nestjs/common';
import { AuthService } from './auth.service';
import { UsersModule } from '../users/users.module';
import { PassportModule } from '@nestjs/passport';
import { LocalStrategy } from './local.strategy';

@Module({
  imports: [UsersModule, PassportModule],
  providers: [AuthService, LocalStrategy],
})
export class AuthModule {}
@@switch
import { Module } from '@nestjs/common';
import { AuthService } from './auth.service';
import { UsersModule } from '../users/users.module';
import { PassportModule } from '@nestjs/passport';
import { LocalStrategy } from './local.strategy';

@Module({
  imports: [UsersModule, PassportModule],
  providers: [AuthService, LocalStrategy],
})
export class AuthModule {}
```

#### Built-in Passport Guards

[가드](/guards) 장에서는 가드의 기본 기능에 대해 설명합니다. 요청이 경로 핸들러에 의해 처리되는지 여부를 결정합니다. 이는 사실이며 곧 표준 기능을 사용할 것입니다. 그러나 `@nestjs/passport` 모듈을 사용하는 맥락에서 처음에는 혼란스러울 수 있는 약간의 새로운 주름(wrinkle)도 소개할 것이므로 지금 논의해 보겠습니다. 인증 관점에서 앱이 두가지 상태로 존재할 수 있습니다.

1. 사용자/클라이언트가 로그인되지 **않았습니다** (인증되지 않음)
2. 사용자/클라이언트가 로그인되어 **있습니다** (인증 됨).

첫번째 경우 (사용자가 로그인하지 않은 경우) 두 가지 다른 기능을 수행해야합니다.

- 인증되지 않은 사용자가 액세스할 수 있는 경로를 제한합니다 (즉, 제한된 경로에 대한 액세스 거부). 우리는 보호된 경로에 경비를 배치하여 이 기능을 처리하기 위해 익숙한 능력으로 가드를 사용할 것입니다. 예상할 수 있듯이 이 가드에서 유효한 JWT가 있는지 확인할 것이므로 JWT를 성공적으로 발급하면 나중에 이 가드에 대해 작업할 것입니다.

- 이전에 인증되지 않은 사용자가 로그인을 시도할 때 **인증 단계** 자체를 시작합니다. 이것은 유효한 사용자에게 JWT를 **발급**하는 단계입니다. 잠시 생각해 보면 인증을 시작하려면 사용자 이름/암호 자격증명을 `POST`해야 한다는 것을 알고 있으므로 이를 처리하기 위해 `POST/auth/login` 경로를 설정합니다. 이것은 질문을 제기합니다: 우리가 그 경로에서 패스포트-로컬 전략을 어떻게 정확히 호출합니까?

대답은 간단합니다. 약간 다른 유형의 가드를 사용하는 것입니다. `@nestjs/passport` 모듈은 우리를 위해 이 작업을 수행하는 내장 가드를 제공합니다. 이 가드는 패스포트 전략을 호출하고 위에서 설명한 단계 (자격증명 검색, 확인기능 실행, `user` 속성 생성 등)를 시작합니다.

위에 열거된 두번째 경우(로그인한 사용자)는 로그인한 사용자가 보호된 경로에 액세스할 수 있도록 이미 논의한 표준유형의 가드에 의존합니다.

<app-banner-courses></app-banner-courses>

#### Login route

전략이 마련되면 이제 베어본 `/auth/login` 경로를 구현하고 내장 가드를 적용하여 패스포트-로컬 흐름을 시작할 수 있습니다.

`app.controller.ts` 파일을 열고 해당 내용을 다음으로 바꿉니다.

```typescript
@@filename(app.controller)
import { Controller, Request, Post, UseGuards } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Controller()
export class AppController {
  @UseGuards(AuthGuard('local'))
  @Post('auth/login')
  async login(@Request() req) {
    return req.user;
  }
}
@@switch
import { Controller, Bind, Request, Post, UseGuards } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Controller()
export class AppController {
  @UseGuards(AuthGuard('local'))
  @Post('auth/login')
  @Bind(Request())
  async login(req) {
    return req.user;
  }
}
```

`@UseGuards(AuthGuard('local'))`에서는 패스포트-로컬 전략을 확장할 때 `@nestjs/passport`가 **자동으로 프로비저닝** 된 `AuthGuard`를 사용하고 있습니다. 그것을 분해합시다. 패스포트 로컬 전략의 기본 이름은 `'local'`입니다. `@UseGuards()` 데코레이터에서 해당 이름을 참조하여 `passport-local` 패키지에서 제공하는 코드와 연결합니다. 이는 앱에 여러 Passport 전략이 있는 경우 호출할 전략을 명확하게하는 데 사용됩니다 (각 전략별로 `AuthGuard`를 프로비저닝할 수 있음). 지금까지는 이러한 전략이 하나 뿐이지 만 잠시 추가할 예정이므로 명확성을 위해 필요합니다.

경로를 테스트하기 위해 `/auth/login` 경로가 지금은 사용자를 반환하도록 할 것입니다. 이를 통해 또 다른 Passport 기능을 보여줄 수 있습니다. Passport는 `validate()` 메서드에서 반환한 값을 기반으로 `user` 객체를 자동으로 생성하고 `req.user`로 `Request` 객체에 할당합니다. 나중에 이를 대신 JWT를 생성하고 반환하는 코드로 대체할 것입니다.

API 경로이므로 일반적으로 사용 가능한 [cURL](https://curl.haxx.se/) 라이브러리를 사용하여 테스트합니다. `UsersService`에 하드코딩된 모든 `user` 객체로 테스트할 수 있습니다.

```bash
$ # POST to /auth/login
$ curl -X POST http://localhost:3000/auth/login -d '{"username": "john", "password": "changeme"}' -H "Content-Type: application/json"
$ # result -> {"userId":1,"username":"john"}
```

이것이 작동하는 동안 전략 이름을 `AuthGuard()`에 직접 전달하면 코드베이스에 매직 문자열이 도입됩니다. 대신 아래와 같이 자신만의 클래스를 만드는 것이 좋습니다.

```typescript
@@filename(auth/local-auth.guard)
import { Injectable } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class LocalAuthGuard extends AuthGuard('local') {}
```

이제 `/auth/login` 경로 핸들러를 업데이트하고 대신 `LocalAuthGuard`를 사용할 수 있습니다.

```typescript
@UseGuards(LocalAuthGuard)
@Post('auth/login')
async login(@Request() req) {
  return req.user;
}
```

#### JWT functionality

인증 시스템의 JWT 부분으로 이동할 준비가 되었습니다. 요구사항을 검토하고 수정해 보겠습니다.

- 사용자가 사용자 이름/암호로 인증할 수 있도록 허용하고 보호된 API 엔드 포인트에 대한 후속 호출에서 사용할 JWT를 반환합니다. 우리는 이 요구 사항을 잘 충족하고 있습니다. 이를 완료하려면 JWT를 발행하는 코드를 작성해야 합니다.
- 베어러(Bearer) 토큰으로 유효한 JWT의 존재를 기반으로 보호되는 API 경로 생성

JWT 요구 사항을 지원하려면 몇가지 패키지를 더 설치해야 합니다.

```bash
$ npm install --save @nestjs/jwt passport-jwt
$ npm install --save-dev @types/passport-jwt
```

`@nestjs/jwt` 패키지 ([여기에서 자세히보기](https://github.com/nestjs/jwt))는 JWT 조작에 도움이 되는 유틸리티 패키지입니다. `passport-jwt` 패키지는 JWT 전략을 구현하는 Passport 패키지이며 `@types/passport-jwt`는 TypeScript 타입 정의를 제공합니다.

`POST/auth/login` 요청이 어떻게 처리되는지 자세히 살펴 보겠습니다. 우리는 패스포트-로컬 전략에서 제공하는 내장된 `AuthGuard`를 사용하여 경로를 장식했습니다. 이는 다음을 의미합니다.

1. 경로 핸들러는 **사용자가 확인 된 경우에만 호출됩니다**
2. `req` 매개변수는 `user` 속성을 포함합니다 (패스포트-로컬 인증 흐름중에 Passport에 의해 채워짐).

이를 염두에 두고 이제 마침내 실제 JWT를 생성하고 이 경로로 반환할 수 있습니다. 서비스를 깔끔하게 모듈화하기 위해 `authService`에서 JWT 생성을 처리합니다. `auth` 폴더에서 `auth.service.ts` 파일을 열고 `login()`메소드를 추가하고 다음과 같이 `JwtService`를 가져옵니다.

```typescript
@@filename(auth/auth.service)
import { Injectable } from '@nestjs/common';
import { UsersService } from '../users/users.service';
import { JwtService } from '@nestjs/jwt';

@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService
  ) {}

  async validateUser(username: string, pass: string): Promise<any> {
    const user = await this.usersService.findOne(username);
    if (user && user.password === pass) {
      const { password, ...result } = user;
      return result;
    }
    return null;
  }

  async login(user: any) {
    const payload = { username: user.username, sub: user.userId };
    return {
      access_token: this.jwtService.sign(payload),
    };
  }
}

@@switch
import { Injectable, Dependencies } from '@nestjs/common';
import { UsersService } from '../users/users.service';
import { JwtService } from '@nestjs/jwt';

@Dependencies(UsersService, JwtService)
@Injectable()
export class AuthService {
  constructor(usersService, jwtService) {
    this.usersService = usersService;
    this.jwtService = jwtService;
  }

  async validateUser(username, pass) {
    const user = await this.usersService.findOne(username);
    if (user && user.password === pass) {
      const { password, ...result } = user;
      return result;
    }
    return null;
  }

  async login(user) {
    const payload = { username: user.username, sub: user.userId };
    return {
      access_token: this.jwtService.sign(payload),
    };
  }
}
```

We're using the `@nestjs/jwt` library, which supplies a `sign()` function to generate our JWT from a subset of the `user` object properties, which we then return as a simple object with a single `access_token` property. Note: we choose a property name of `sub` to hold our `userId` value to be consistent with JWT standards. Don't forget to inject the JwtService provider into the `AuthService`.

We now need to update the `AuthModule` to import the new dependencies and configure the `JwtModule`.

First, create `constants.ts` in the `auth` folder, and add the following code:

`@nestjs/jwt` 라이브러리를 사용합니다. `sign()` 함수를 제공하여 `user` 객체 속성의 하위 집합에서 JWT를 생성한 다음 단일 `access_token` 속성이 있는 간단한 객체로 반환합니다. 참고: JWT 표준과 일치하도록 `userId` 값을 유지하기 위해 `sub`의 속성 이름을 선택합니다. JwtService 공급자를 `AuthService`에 삽입하는 것을 잊지 마십시오.

이제 `AuthModule`을 업데이트하여 새 종속성을 가져오고 `JwtModule`을 구성해야 합니다.

먼저 `auth` 폴더에 `constants.ts`를 만들고 다음 코드를 추가합니다.

```typescript
@@filename(auth/constants)
export const jwtConstants = {
  secret: 'secretKey',
};
@@switch
export const jwtConstants = {
  secret: 'secretKey',
};
```

이를 사용하여 JWT 서명 및 확인 단계간에 키를 공유합니다.

> warning **경고** **이 키를 공개하지 마세요**. 코드가 수행하는 작업을 명확히하기 위해 여기에서 수행했지만 프로덕션 시스템에서는 비밀 금고, 환경 변수 또는 구성 서비스와 같은 적절한 조치를 사용하여 **이 키를 보호해야 합니다**.

이제 `auth` 폴더에서 `auth.module.ts`를 열고 다음과 같이 업데이트합니다.

```typescript
@@filename(auth/auth.module)
import { Module } from '@nestjs/common';
import { AuthService } from './auth.service';
import { LocalStrategy } from './local.strategy';
import { UsersModule } from '../users/users.module';
import { PassportModule } from '@nestjs/passport';
import { JwtModule } from '@nestjs/jwt';
import { jwtConstants } from './constants';

@Module({
  imports: [
    UsersModule,
    PassportModule,
    JwtModule.register({
      secret: jwtConstants.secret,
      signOptions: { expiresIn: '60s' },
    }),
  ],
  providers: [AuthService, LocalStrategy],
  exports: [AuthService, JwtModule],
})
export class AuthModule {}
@@switch
import { Module } from '@nestjs/common';
import { AuthService } from './auth.service';
import { LocalStrategy } from './local.strategy';
import { UsersModule } from '../users/users.module';
import { PassportModule } from '@nestjs/passport';
import { JwtModule } from '@nestjs/jwt';
import { jwtConstants } from './constants';

@Module({
  imports: [
    UsersModule,
    PassportModule,
    JwtModule.register({
      secret: jwtConstants.secret,
      signOptions: { expiresIn: '60s' },
    }),
  ],
  providers: [AuthService, LocalStrategy],
  exports: [AuthService],
})
export class AuthModule {}
```

`register()`를 사용하여 `JwtModule`을 구성하고 구성 객체를 전달합니다. Nest `JwtModule`에 대한 자세한 내용은 [여기](https://github.com/nestjs/jwt/blob/master/README.md)를, 사용 가능한 구성옵션에 대한 자세한 내용은 [여기](https://github.com/auth0/node-jsonwebtoken#usage)를 참조하세요.

이제 `/auth/login` 경로를 업데이트하여 JWT를 반환할 수 있습니다.

```typescript
@@filename(app.controller)
import { Controller, Request, Post, UseGuards } from '@nestjs/common';
import { LocalAuthGuard } from './auth/local-auth.guard';
import { AuthService } from './auth/auth.service';

@Controller()
export class AppController {
  constructor(private authService: AuthService) {}

  @UseGuards(LocalAuthGuard)
  @Post('auth/login')
  async login(@Request() req) {
    return this.authService.login(req.user);
  }
}
@@switch
import { Controller, Bind, Request, Post, UseGuards } from '@nestjs/common';
import { LocalAuthGuard } from './auth/local-auth.guard';
import { AuthService } from './auth/auth.service';

@Controller()
export class AppController {
  constructor(private authService: AuthService) {}

  @UseGuards(LocalAuthGuard)
  @Post('auth/login')
  @Bind(Request())
  async login(req) {
    return this.authService.login(req.user);
  }
}
```

계속해서 cURL을 사용하여 경로를 테스트해 보겠습니다. `UsersService`에 하드코딩된 모든 `user` 객체로 테스트할 수 있습니다.

```bash
$ # POST to /auth/login
$ curl -X POST http://localhost:3000/auth/login -d '{"username": "john", "password": "changeme"}' -H "Content-Type: application/json"
$ # result -> {"access_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."}
$ # Note: above JWT truncated
```

#### Implementing Passport JWT

이제 최종 요구사항을 해결할 수 있습니다. 요청에 유효한 JWT를 요구하여 엔드 포인트를 보호합니다. 패스포트는 여기에서도 우리를 도울 수 있습니다. JSON 웹 토큰으로 RESTful 엔드 포인트를 보호하기위한 [passport-jwt](https://github.com/mikenicholson/passport-jwt) 전략을 제공합니다. 먼저 `auth` 폴더에 `jwt.strategy.ts`라는 파일을 만들고 다음 코드를 추가합니다.

```typescript
@@filename(auth/jwt.strategy)
import { ExtractJwt, Strategy } from 'passport-jwt';
import { PassportStrategy } from '@nestjs/passport';
import { Injectable } from '@nestjs/common';
import { jwtConstants } from './constants';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: jwtConstants.secret,
    });
  }

  async validate(payload: any) {
    return { userId: payload.sub, username: payload.username };
  }
}
@@switch
import { ExtractJwt, Strategy } from 'passport-jwt';
import { PassportStrategy } from '@nestjs/passport';
import { Injectable } from '@nestjs/common';
import { jwtConstants } from './constants';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: jwtConstants.secret,
    });
  }

  async validate(payload) {
    return { userId: payload.sub, username: payload.username };
  }
}
```

우리의 `JwtStrategy`를 사용하여 모든 Passport 전략에 대해 앞서 설명한 것과 동일한 레시피를 따랐습니다. 이 전략에는 약간의 초기화가 필요하므로 `super()` 호출에서 옵션 객체를 전달하여 수행합니다. 사용 가능한 옵션에 대한 자세한 내용은 [여기](https://github.com/mikenicholson/passport-jwt#configure-strategy)를 참조하세요. 우리의 경우 이러한 옵션은 다음과 같습니다.

- `jwtFromRequest`: `Request`에서 JWT를 추출하는 방법을 제공합니다. API 요청의 Authorization 헤더에 전달자 토큰을 제공하는 표준 접근 방식을 사용합니다. 다른 옵션은 [여기](https://github.com/mikenicholson/passport-jwt#extracting-the-jwt-from-the-request)에 설명되어 있습니다.
- `ignoreExpiration`: 명시적으로 말하자면, JWT가 Passport 모듈에 만료되지 않았는지 확인하는 책임을 위임하는 기본 `false` 설정을 선택합니다. 즉, 경로에 만료된 JWT가 제공되면 요청이 거부되고 '401 Unauthorized'응답이 전송됩니다. Passport는 이를 자동으로 처리합니다.
- `secretOrKey`: 토큰 서명을 위해 대칭 암호를 제공하는 편리한 옵션을 사용하고 있습니다. PEM으로 인코딩된 공개키와 같은 다른 옵션은 프로덕션앱에 더 적합할 수 있습니다 (자세한 내용은 [여기](https://github.com/mikenicholson/passport-jwt#extracting-the-jwt-from-the-request)). 어쨌든 앞서 언급했듯이 **이 비밀을 공개하지 마세요**.


`validate()` 메소드는 약간의 논의가 필요합니다. jwt-strategy의 경우 Passport는 먼저 JWT의 서명을 확인하고 JSON을 디코딩합니다. 그런 다음 디코딩된 JSON을 단일 매개 변수로 전달하는 `validate()`메소드를 호출합니다. JWT 서명이 작동하는 방식에 따라 이전에 서명하여 유효한 사용자에게 발급한 **유효한 토큰**을 받음을 보장합니다.

이 모든 결과로 `validate()` 콜백에 대한 응답은 사소합니다. `userId` 및 `username` 속성을 포함하는 객체를 반환하기 만하면 됩니다. Passport는 `validate()` 메서드의 반환 값을 기반으로 `user` 객체를 빌드하고 `Request` 객체에 속성으로 첨부합니다.

이 접근 방식은 다른 비즈니스 로직을 프로세스에 주입할 여지 (이전의 '후크')를 남긴다는 점도 지적할 가치가 있습니다. 예를 들어 `validate()` 메서드에서 데이터베이스 조회를 수행하여 사용자에 대한 더 많은 정보를 추출할 수 있으며, 결과적으로 `Request`에서 더 풍부한 `user` 객체를 사용할 수 있습니다. 또한 해지된 토큰 목록에서 `userId`를 조회하여 토큰 해지를 수행할 수 있도록 추가 토큰 유효성 검사를 수행하기로 결정할 수 있습니다. 샘플 코드에서 구현한 모델은 빠른 "상태 비 저장(stateless) JWT" 모델입니다. 각 API 호출은 유효한 JWT의 존재와 요청자에 대한 약간의 정보 (`userId` 및 `username`)은 요청 파이프 라인에서 사용할 수 있습니다.

`AuthModule`에 새 `JwtStrategy`를 프로바이더로 추가합니다.

```typescript
@@filename(auth/auth.module)
import { Module } from '@nestjs/common';
import { AuthService } from './auth.service';
import { LocalStrategy } from './local.strategy';
import { JwtStrategy } from './jwt.strategy';
import { UsersModule } from '../users/users.module';
import { PassportModule } from '@nestjs/passport';
import { JwtModule } from '@nestjs/jwt';
import { jwtConstants } from './constants';

@Module({
  imports: [
    UsersModule,
    PassportModule,
    JwtModule.register({
      secret: jwtConstants.secret,
      signOptions: { expiresIn: '60s' },
    }),
  ],
  providers: [AuthService, LocalStrategy, JwtStrategy],
  exports: [AuthService],
})
export class AuthModule {}
@@switch
import { Module } from '@nestjs/common';
import { AuthService } from './auth.service';
import { LocalStrategy } from './local.strategy';
import { JwtStrategy } from './jwt.strategy';
import { UsersModule } from '../users/users.module';
import { PassportModule } from '@nestjs/passport';
import { JwtModule } from '@nestjs/jwt';
import { jwtConstants } from './constants';

@Module({
  imports: [
    UsersModule,
    PassportModule,
    JwtModule.register({
      secret: jwtConstants.secret,
      signOptions: { expiresIn: '60s' },
    }),
  ],
  providers: [AuthService, LocalStrategy, JwtStrategy],
  exports: [AuthService],
})
export class AuthModule {}
```

JWT에 서명할 때 사용한 것과 동일한 비밀(secret)을 가져옴으로써 Passport에서 수행하는 **verify** 단계와 `AuthService`에서 수행한 **sign** 단계가 공통 비밀(common secret)을 사용하는지 확인합니다.

마지막으로 내장된 `AuthGuard`를 확장하는 `JwtAuthGuard` 클래스를 정의합니다.

```typescript
@@filename(auth/jwt-auth.guard)
import { Injectable } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {}
```

#### Implement protected route and JWT strategy guards

이제 보호된 경로와 관련 가드를 구현할 수 있습니다.

`app.controller.ts` 파일을 열고 아래와 같이 업데이트합니다.

```typescript
@@filename(app.controller)
import { Controller, Get, Request, Post, UseGuards } from '@nestjs/common';
import { JwtAuthGuard } from './auth/jwt-auth.guard';
import { LocalAuthGuard } from './auth/local-auth.guard';
import { AuthService } from './auth/auth.service';

@Controller()
export class AppController {
  constructor(private authService: AuthService) {}

  @UseGuards(LocalAuthGuard)
  @Post('auth/login')
  async login(@Request() req) {
    return this.authService.login(req.user);
  }

  @UseGuards(JwtAuthGuard)
  @Get('profile')
  getProfile(@Request() req) {
    return req.user;
  }
}
@@switch
import { Controller, Dependencies, Bind, Get, Request, Post, UseGuards } from '@nestjs/common';
import { JwtAuthGuard } from './auth/jwt-auth.guard';
import { LocalAuthGuard } from './auth/local-auth.guard';
import { AuthService } from './auth/auth.service';

@Dependencies(AuthService)
@Controller()
export class AppController {
  constructor(authService) {
    this.authService = authService;
  }

  @UseGuards(LocalAuthGuard)
  @Post('auth/login')
  @Bind(Request())
  async login(req) {
    return this.authService.login(req.user);
  }

  @UseGuards(JwtAuthGuard)
  @Get('profile')
  @Bind(Request())
  getProfile(req) {
    return req.user;
  }
}
```

다시 한 번, passport-jwt 모듈을 구성할 때 `@nestjs/passport` 모듈이 자동으로 프로비저닝한 `AuthGuard`를 적용합니다. 이 가드는 기본 이름 `jwt`로 참조됩니다. `GET /profile` 경로가 적중되면 가드는 자동으로 passport-jwt 사용자 정의 구성 로직을 호출하여 JWT를 검증하고 `user` 속성을 `Request` 객체에 할당합니다.

앱이 실행 중인지 확인하고 'cURL'을 사용하여 경로를 테스트합니다.

```bash
$ # GET /profile
$ curl http://localhost:3000/profile
$ # result -> {"statusCode":401,"error":"Unauthorized"}

$ # POST /auth/login
$ curl -X POST http://localhost:3000/auth/login -d '{"username": "john", "password": "changeme"}' -H "Content-Type: application/json"
$ # result -> {"access_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2Vybm... }

$ # GET /profile using access_token returned from previous step as bearer code
$ curl http://localhost:3000/profile -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2Vybm..."
$ # result -> {"userId":1,"username":"john"}
```

`AuthModule`에서 JWT를 `60초`의 만료 시간으로 구성했습니다. 만료 시간이 너무 짧을 수 있으며 토큰 만료 및 새로 고침에 대한 세부 사항을 다루는 것은 이 기사의 범위를 벗어납니다. 그러나 우리는 JWT와 passport-jwt 전략의 중요한 품질을 보여주기 위해 그것을 선택했습니다. 인증후 60초 후에 `GET /profile` 요청을 시도하면 `401 Unauthorized` 응답을 받게됩니다. 이는 Passport가 JWT의 만료 시간을 자동으로 확인하여 응용 프로그램에서 수행하는 수고를 덜어주기 때문입니다.

이제 JWT 인증 구현을 완료했습니다. JavaScript 클라이언트 (예: Angular/React/Vue) 및 기타 JavaScript 앱은 이제 API 서버를 인증하고 안전하게 통신할 수 있습니다.

#### Example

이 장 [여기](https://github.com/nestjs/nest/tree/master/sample/19-auth-jwt)에서 전체 코드 버전을 찾을 수 있습니다.

#### Extending guards

대부분의 경우 제공된 `AuthGuard` 클래스를 사용하는 것으로 충분합니다. 그러나 단순히 기본 오류처리 또는 인증 논리를 확장하려는 경우 사용사례가 있을 수 있습니다. 이를 위해 내장 클래스를 확장하고 하위 클래스 내에서 메서드를 재정의할 수 있습니다.

```typescript
import {
  ExecutionContext,
  Injectable,
  UnauthorizedException,
} from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  canActivate(context: ExecutionContext) {
    // Add your custom authentication logic here
    // for example, call super.logIn(request) to establish a session.
    return super.canActivate(context);
  }

  handleRequest(err, user, info) {
    // You can throw an exception based on either "info" or "err" arguments
    if (err || !user) {
      throw err || new UnauthorizedException();
    }
    return user;
  }
}
```

기본 오류 처리 및 인증 논리를 확장하는 것 외에도 인증이 일련의 전략을 거치도록 할 수 있습니다. 성공, 리디렉션 또는 오류에 대한 첫번째 전략은 체인을 중지합니다. 인증 실패는 일련의 각 전략을 통해 진행되며 모든 전략이 실패하면 궁극적으로 실패합니다.

```typescript
export class JwtAuthGuard extends AuthGuard(['strategy_jwt_1', 'strategy_jwt_2', '...']) { ... }
```

#### Enable authentication globally

대부분의 엔드포인트를 기본적으로 보호해야하는 경우 각 컨트롤러 상단에서 `@UseGuards()` 데코레이터를 사용하는 대신 [전역 가드](/guards#binding-guards)로 인증 가드를 등록할 수 있습니다. 공개할 라우트를 간단히 표시할 수 있습니다.

먼저 다음 구성을 사용하여 `JwtAuthGuard`를 전역 가드로 등록합니다(모든 모듈에서).

```typescript
providers: [
  {
    provide: APP_GUARD,
    useClass: JwtAuthGuard,
  },
],
```

이를 통해 Nest는 자동으로 `JwtAuthGuard`를 모든 엔드 포인트에 바인딩합니다.

이제 우리는 경로를 공개로 선언하는 메커니즘을 제공해야 합니다. 이를 위해 `SetMetadata` 데코레이터 팩토리 함수를 사용하여 커스텀 데코레이터를 만들 수 있습니다.

```typescript
import { SetMetadata } from '@nestjs/common';

export const IS_PUBLIC_KEY = 'isPublic';
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);
```

위 파일에서 두 개의 상수를 내보냈습니다. 하나는 `IS_PUBLIC_KEY`라는 메타 데이터 키이고 다른 하나는 `Public`이라고 할 새로운 데코레이터 자체입니다 (프로젝트에 맞는 이름으로 `SkipAuth` 또는 `AllowAnon` 이름을 지정할 수도 있습니다).

이제 사용자 정의 `@Public()`데코레이터가 있으므로 이를 사용하여 다음과 같이 모든 메소드를 데코레이션할 수 있습니다.

```typescript
@Public()
@Get()
findAll() {
  return [];
}
```

마지막으로 `"isPublic"` 메타 데이터가 발견되면 `true`를 반환하는 `JwtAuthGuard`가 필요합니다. 이를 위해 우리는 `Reflector` 클래스를 사용할 것입니다 ([여기](/guards#putting-it-all-together)에서 자세히 읽어보십시오).

```typescript
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  constructor(private reflector: Reflector) {
    super();
  }

  canActivate(context: ExecutionContext) {
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);
    if (isPublic) {
      return true;
    }
    return super.canActivate(context);
  }
}
```

#### Request-scoped strategies

패스포트 API는 라이브러리의 글로벌 인스턴스에 대한 등록 전략을 기반으로 합니다. 따라서 전략은 요청 종속 옵션을 갖거나 요청마다 동적으로 인스턴스화되도록 설계되지 않았습니다 ([요청 범위](/fundamentals/injection-scopes) 프로바이더에 대해 자세히 읽어보십시오). 요청 범위가 되도록 전략을 구성하면 Nest는 특정 경로에 연결되어 있지 않으므로 절대 인스턴스화하지 않습니다. 요청당 실행할 "요청 범위" 전략을 결정하는 물리적 방법은 없습니다.

그러나 전략내에서 요청 범위 프로바이더를 동적으로 확인하는 방법이 있습니다. 이를 위해 [모듈 참조](/fundamentals/module-ref) 기능을 활용합니다.

먼저 `local.strategy.ts` 파일을 열고 일반적인 방법으로 `ModuleRef`를 삽입합니다.

```typescript
constructor(private moduleRef: ModuleRef) {
  super({
    passReqToCallback: true,
  });
}
```

> info **힌트** `ModuleRef` 클래스는 `@nestjs/core` 패키지에서 가져옵니다.

위에 표시된대로 `passReqToCallback` 구성 속성을 `true`로 설정해야 합니다.

다음 단계에서 요청 인스턴스는 새 컨텍스트 식별자를 생성하는 대신 현재 컨텍스트 식별자를 얻는 데 사용됩니다 (요청 컨텍스트 [여기](/fundamentals/module-ref#getting-current-sub-tree) 참조). .

이제`LocalStrategy` 클래스의`validate ()` 메소드 내에서 `ContextIdFactory` 클래스의 `getByRequest ()` 메소드를 사용하여 요청 객체를 기반으로 컨텍스트 ID를 생성하고이를 `resolve( )` 호출:

```typescript
async validate(
  request: Request,
  username: string,
  password: string,
) {
  const contextId = ContextIdFactory.getByRequest(request);
  // "AuthService" is a request-scoped provider
  const authService = await this.moduleRef.resolve(AuthService, contextId);
  ...
}
```

위의 예에서 `resolve()` 메소드는 `AuthService` 공급자의 요청 범위 인스턴스를 비동기적으로 반환합니다 (`AuthService`가 요청 범위 공급자로 표시되어 있다고 가정).

#### Customize Passport

모든 표준 Passport 사용자 지정 옵션은 `register()` 메서드를 사용하여 동일한 방식으로 전달할 수 있습니다. 사용 가능한 옵션은 구현되는 전략에 따라 다릅니다. 예를 들면:

```typescript
PassportModule.register({ session: true });
```

또한 전략을 구성하기 위해 생성자(constructor)에서 옵션 개체를 전달할 수 있습니다.
로컬 전략의 경우 다음을 전달할 수 있습니다.

```typescript
constructor(private authService: AuthService) {
  super({
    usernameField: 'email',
    passwordField: 'password',
  });
}
```

속성 이름은 공식 [패스포트 웹사이트](http://www.passportjs.org/docs/oauth/)를 참조하십시오.

#### Named strategies

전략을 구현할 때 두번째 인수를 `PassportStrategy` 함수에 전달하여 전략의 이름을 제공할 수 있습니다. 이렇게하지 않으면 각 전략에 기본 이름이 지정됩니다 (예: jwt-strategy의 경우 `jwt`).

```typescript
export class JwtStrategy extends PassportStrategy(Strategy, 'myjwt')
```

그런 다음 `@UseGuards(AuthGuard('myjwt'))`와 같은 데코레이터를 통해 이를 참조합니다.

#### GraphQL

[GraphQL](/graphql/quick-start)과 함께 `AuthGuard`를 사용하려면 기본 제공 AuthGuard 클래스를 확장하고 `getRequest()` 메서드를 재정의합니다.

```typescript
@Injectable()
export class GqlAuthGuard extends AuthGuard('jwt') {
  getRequest(context: ExecutionContext) {
    const ctx = GqlExecutionContext.create(context);
    return ctx.getContext().req;
  }
}
```

graphql 리졸버에서 현재 인증된 사용자를 얻으려면 `@CurrentUser()` 데코레이터를 정의할 수 있습니다.

```typescript
import { createParamDecorator, ExecutionContext } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';

export const CurrentUser = createParamDecorator(
  (data: unknown, context: ExecutionContext) => {
    const ctx = GqlExecutionContext.create(context);
    return ctx.getContext().req.user;
  },
);
```

리졸버에서 위 데코레이터를 사용하려면 쿼리 또는 변형의 매개 변수로 포함해야합니다.

```typescript
@Query(returns => User)
@UseGuards(GqlAuthGuard)
whoAmI(@CurrentUser() user: User) {
  return this.usersService.findById(user.id);
}
```
