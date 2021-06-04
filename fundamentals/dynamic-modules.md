### Dynamic modules

[모듈 장](/modules)은 Nest 모듈의 기본 사항을 다루고 [동적 모듈](/modules#dynamic-modules)에 대한 간략한 소개를 포함합니다. 이 장에서는 동적 모듈의 주제를 확장합니다. 완료되면 그것이 무엇이며 언제 어떻게 사용하는지 잘 이해해야 합니다.

#### Introduction

문서의 **개요** 섹션에 있는 대부분의 애플리케이션 코드 예제는 일반 또는 정적 모듈을 사용합니다. 모듈은 전체 애플리케이션의 모듈식 부분으로 함께 맞는 [프로바이더](/providers) 및 [컨트롤러](/controllers)와 같은 구성 요소 그룹을 정의합니다. 이러한 구성 요소에 대한 실행 컨텍스트 또는 범위를 제공합니다. 예를 들어, 모듈에 정의된 프로바이더는 내보낼 필요없이 모듈의 다른 멤버에게 표시됩니다. 프로바이더가 모듈 외부에서 표시되어야 하는 경우 먼저 호스트 모듈에서 내보낸 다음 소비 모듈(consuming moduel)로 가져옵니다.

익숙한 예를 살펴 보겠습니다.

먼저 `UsersService`를 제공하고 내보내는 `UsersModule`을 정의합니다. `UsersModule`은 `UsersService`의 **호스트** 모듈입니다.

```typescript
import { Module } from '@nestjs/common';
import { UsersService } from './users.service';

@Module({
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

다음으로 `UsersModule`을 가져 오는 `AuthModule`을 정의하여 `UsersModule`의 내보낸 프로바이더를 `AuthModule` 내에서 사용할 수 있도록 합니다.

```typescript
import { Module } from '@nestjs/common';
import { AuthService } from './auth.service';
import { UsersModule } from '../users/users.module';

@Module({
  imports: [UsersModule],
  providers: [AuthService],
  exports: [AuthService],
})
export class AuthModule {}
```

이러한 구성을 통해 예를 들어 `AuthModule`에서 호스팅되는 `AuthService`에 `UsersService`를 삽입할 수 있습니다.

```typescript
import { Injectable } from '@nestjs/common';
import { UsersService } from '../users/users.service';

@Injectable()
export class AuthService {
  constructor(private usersService: UsersService) {}
  /*
    Implementation that makes use of this.usersService
  */
}
```

이를 **정적** 모듈 바인딩이라고 합니다. Nest가 모듈을 함께 연결하는 데 필요한 모든 정보는 호스트 및 소비 모듈에서 이미 선언되었습니다. 이 과정에서 무슨 일이 일어나고 있는지 살펴 보겠습니다. Nest는 다음을 통해 `AuthModule` 내에서 `UsersService`를 사용할 수 있습니다.

1. `UsersModule` 자체가 소비하는 다른 모듈을 전이적으로 가져오고 모든 종속성을 전이적으로 해결하는 것을 포함하여 `UsersModule`을 인스턴스화합니다 ([사용자 정의 공급자](/fundamentals/custom-providers) 참조).
2. `AuthModule`을 인스턴스화하고 `UsersModule`의 내보낸 공급자를 `AuthModule`의 구성 요소에 사용할 수 있도록 합니다 (`AuthModule`에서 선언된 것처럼).
3. `AuthService`에 `UsersService`의 인스턴스를 삽입합니다.

#### Dynamic module use case

정적 모듈 바인딩을 사용하면 소비 모듈이 호스트 모듈의 프로바이더가 구성되는 방식에 **영향을 주는** 기회가 없습니다. 이것이 왜 중요합니까? 다른 사용 사례에서 다르게 동작해야 하는 범용 모듈이 있는 경우를 고려하십시오. 이는 일반 기능이 소비자가 사용하기 전에 일부 구성을 요구하는 많은 시스템의 "플러그인" 개념과 유사합니다.

Nest의 좋은 예는 **구성 모듈**입니다. 많은 애플리케이션은 구성 모듈을 사용하여 구성 세부 정보를 구체화하는 것이 유용하다고 생각합니다. 이를 통해 개발자를 위한 개발 데이터베이스, 스테이징/테스트 환경을 위한 스테이징 데이터베이스 등 다양한 배포에서 애플리케이션 설정을 동적으로 쉽게 변경할 수 있습니다. 구성 매개 변수 관리를 구성 모듈, 애플리케이션 소스 코드에 위임함으로써 구성 매개 변수와 무관합니다.

문제는 구성 모듈 자체가 일반("플러그인"과 유사)이기 때문에 소비 모듈에 의해 사용자 정의되어야 한다는 것입니다. 여기서 _동적 모듈_ 이 작동합니다. 동적 모듈 기능을 사용하여 구성 모듈을 **동적**으로 만들 수 있으므로 소비 모듈이 API를 사용하여 가져올 때 구성 모듈을 사용자 정의하는 방법을 제어할 수 있습니다.

즉, 동적 모듈은 지금까지 살펴본 정적 바인딩을 사용하는 것과 반대로 한 모듈을 다른 모듈로 가져오고 해당 모듈을 가져올 때 해당 모듈의 속성과 동작을 사용자 정의하는 API를 제공합니다.

<app-banner-shop></app-banner-shop>

#### Config module example

이 섹션에서는 [configuration chapter](/techniques/configuration#service)에 있는 예제 코드의 기본 버전을 사용합니다. 이 장이 끝날 때까지 완성된 버전은 [예제 여기](https://github.com/nestjs/nest/tree/master/sample/25-dynamic-modules)로 제공됩니다.

우리의 요구 사항은 `ConfigModule`이 `options` 객체를 받아 사용자 정의하도록 만드는 것입니다. 지원하려는 기능은 다음과 같습니다. 기본 샘플은 프로젝트 루트 폴더에 있는 `.env` 파일의 위치를 하드코딩합니다. 원하는 폴더에서 `.env` 파일을 관리할 수 있도록 구성 가능하게 만들고 싶다고 가정해 보겠습니다. 예를 들어 다양한 `.env` 파일을 `config`라는 프로젝트 루트 아래의 폴더(즉, `src`의 형제 폴더)에 저장한다고 가정 해보십시오. 다른 프로젝트에서 `ConfigModule`을 사용할 때 다른 폴더를 선택할 수 있기를 원합니다.

동적 모듈은 가져 오는 모듈에 매개 변수를 전달하여 동작을 변경할 수 있도록 합니다. 이것이 어떻게 작동하는지 봅시다. 소비 모듈의 관점에서 이것이 어떻게 보이는지 최종 목표에서 시작한 다음 거꾸로 작업하면 도움이 됩니다. 먼저 `ConfigModule`을 _정적으로_ 가져 오는 예제 (즉, 가져온 모듈의 동작에 영향을 주지 않는 접근 방식)를 빠르게 검토해 보겠습니다. `@Module()` 데코레이터의 `imports` 배열에 주의를 기울이십시오.

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { ConfigModule } from './config/config.module';

@Module({
  imports: [ConfigModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

구성 객체를 전달하는 _동적 모률_ 가져오기가 어떻게 생겼는지 생각해 봅시다. 다음 두 예의 `imports` 배열의 차이점을 비교하십시오.

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { ConfigModule } from './config/config.module';

@Module({
  imports: [ConfigModule.register({ folder: './config' })],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

위의 동적 예제에서 어떤 일이 발생하는지 살펴 보겠습니다. 움직이는 부분은 무엇입니까?

1. `ConfigModule`은 일반 클래스이므로 `register()`라는 **정적 메서드**가 있어야한다고 추론할 수 있습니다. 클래스의 **인스턴스**가 아닌 `ConfigModule` 클래스에서 호출하기 때문에 정적이라는 것을 알고 있습니다. 참고: 곧 만들게 될 이 메서드는 임의의 이름을 가질 수 있지만 관례에 따라 `forRoot()` 또는 `register()`로 호출해야 합니다.
2. `register()` 메소드는 우리가 정의하므로 원하는 입력 인수를 받을 수 있습니다. 이 경우 적절한 속성을 가진 간단한 `options` 객체를 받아들일 것입니다. 이것이 일반적인 경우입니다.
3. `register()` 메소드는 우리가 지금까지 살펴본 모듈 목록을 포함하는 친숙한 `imports` 목록에 반환 값이 나타나기 때문에 `module`과 같은 것을 반환해야 한다고 추론할 수 있습니다.

사실, `register()` 메소드가 반환하는 것은 `DynamicModule`입니다. 동적 모듈은 런타임에 생성되는 모듈에 지나지 않으며 정적 모듈과 동일한 속성과 `moduel`이라는 하나의 추가 속성을 포함합니다. 데코레이터에 전달된 모듈 옵션에 세심한 주의를 기울여 샘플 정적 모듈 선언을 빠르게 검토해 보겠습니다.

```typescript
@Module({
  imports: [DogsModule],
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService]
})
```

동적 모듈은 정확히 동일한 인터페이스와 `module`이라는 추가 속성을 가진 객체를 반환해야 합니다. `module` 속성은 모듈의 이름 역할을 하며 아래 예제와 같이 모듈의 클래스 이름과 동일해야합니다.

> info **힌트** 동적 모듈의 경우 모듈 옵션 개체의 모든 속성은 `module`을 **제외한** 선택사항입니다.

정적 `register()` 메서드는 어떻습니까? 이제 작업이 `DynamicModule` 인터페이스가 있는 객체를 반환하는 것임을 알 수 있습니다. 우리가 그것을 호출할 때, 우리는 모듈 클래스 이름을 나열하여 정적 사례에서 수행하는 방식과 유사하게 `imports` 목록에 모듈을 효과적으로 제공합니다. 즉, 동적 모듈 API는 단순히 모듈을 반환하지만 `@Module` 데코레이터의 속성을 수정하는 대신 프로그래밍 방식으로 지정합니다.

그림을 완성하는 데 도움이 되는 몇가지 세부 정보가 여전히 있습니다.

1. 이제 `@Module()` 데코레이터의 `imports` 속성이 모듈 클래스 이름(예: `imports: [UsersModule]`)뿐만 아니라 동적을 **반환**하는 함수 모듈(예: `imports: [ConfigModule.register(...)]`).
2. 동적 모듈은 자체적으로 다른 모듈을 가져올 수 있습니다. 이 예제에서는 그렇게하지 않겠지만, 동적 모듈이 다른 모듈의 프로바이더에 의존하는 경우 선택적 `imports` 속성을 사용하여 가져옵니다. 다시 말하지만 이것은 `@Module()` 데코레이터를 사용하여 정적 모듈에 대한 메타 데이터를 선언하는 방식과 정확히 유사합니다.

이러한 이해를 바탕으로 이제 동적 `ConfigModule` 선언이 어떻게 생겼는지 살펴볼 수 있습니다. 그것에 균열을 가져 가자.

```typescript
import { DynamicModule, Module } from '@nestjs/common';
import { ConfigService } from './config.service';

@Module({})
export class ConfigModule {
  static register(): DynamicModule {
    return {
      module: ConfigModule,
      providers: [ConfigService],
      exports: [ConfigService],
    };
  }
}
```

이제 조각이 어떻게 연결되는지 명확해야합니다. `ConfigModule.register(...)`를 호출하면 지금까지 `@Module()` 데코레이터를 통해 메타 데이터로 제공했던 속성과 본질적으로 동일한 속성을 가진 `DynamicModule` 객체가 반환됩니다.

> info **힌트** `@nestjs/common`에서 `DynamicModule`을 가져옵니다.

그러나 동적 모듈은 아직 별로 흥미롭지 않습니다. 하지만 원하는대로 **구성**하는 기능을 도입하지 않았기 때문입니다. 다음으로 설명하겠습니다.

### Module configuration

`ConfigModule`의 동작을 사용자 정의하는 확실한 해결책은 위에서 추측한 것처럼 정적 `register()` 메서드에서 `options` 객체를 전달하는 것입니다. 소비 모듈의 `imports` 속성을 다시 한번 살펴보겠습니다.

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { ConfigModule } from './config/config.module';

@Module({
  imports: [ConfigModule.register({ folder: './config' })],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

그것은 우리의 동적 모듈에 `options` 객체를 전달하는 것을 멋지게 처리합니다. 그러면 `ConfigModule`에서 `options` 객체를 어떻게 사용합니까? 잠시 생각해 봅시다. 우리는 `ConfigModule`이 기본적으로 다른 프로바이더가 사용하기 위해 주입 가능한 서비스 인 `ConfigService`를 제공하고 내보내는 호스트라는 것을 알고 있습니다. 동작을 사용자 정의하기 위해 `options` 객체를 읽어야하는 것은 실제로 `ConfigService`입니다. `register()` 메서드에서 `ConfigService`로 `options`를 가져오는 방법을 알고 있다고 가정해 보겠습니다. 이러한 가정하에 `options` 객체의 속성을 기반으로 동작을 사용자 지정하기 위해 서비스를 몇가지 변경할 수 있습니다. (**참고**: 당분간은 실제로 전달 방법을 결정하지 _않았으므로_ `옵션`을 하드코딩할 것입니다. 잠시 후에 이 문제를 해결할 것입니다).

```typescript
import { Injectable } from '@nestjs/common';
import * as dotenv from 'dotenv';
import * as fs from 'fs';
import { EnvConfig } from './interfaces';

@Injectable()
export class ConfigService {
  private readonly envConfig: EnvConfig;

  constructor() {
    const options = { folder: './config' };

    const filePath = `${process.env.NODE_ENV || 'development'}.env`;
    const envFile = path.resolve(__dirname, '../../', options.folder, filePath);
    this.envConfig = dotenv.parse(fs.readFileSync(envFile));
  }

  get(key: string): string {
    return this.envConfig[key];
  }
}
```

이제 `ConfigService`는 `options`에서 지정한 폴더에서 `.env` 파일을 찾는 방법을 알고 있습니다.

나머지 작업은 `register()` 단계의 `options` 객체를 `ConfigService`에 주입하는 것입니다. 그리고 물론, 우리는 그것을 하기 위해 _종속성 주입_ 을 사용할 것입니다. 이것이 핵심 사항이므로 반드시 이해해야 합니다. 우리의 `ConfigModule`은 `ConfigService`를 제공합니다. 차례로 `ConfigService`는 런타임에만 제공되는 `options` 객체에 의존합니다. 따라서 런타임에 먼저 `options` 객체를 Nest IoC 컨테이너에 바인딩한 다음 Nest가 이를 `ConfigService`에 주입하도록 해야합니다. **사용자 지정 프로바이더**장에서 프로바이더는 서비스뿐만 아니라 [모든 값을 포함](/fundamentals/custom-providers#non-service-based-providers)할 수 있음을 기억하십시오. 간단한 `options` 객체를 처리하기 위해 의존성 주입을 사용해도 괜찮습니다.

먼저 옵션 개체를 IoC 컨테이너에 바인딩하는 방법을 살펴 보겠습니다. 정적 `register()` 메서드에서 이 작업을 수행합니다. 우리는 모듈을 동적으로 구성하고 있으며 모듈의 속성중 하나는 프로바이더 목록입니다. 그래서 우리가 해야 할 일은 옵션 객체를 프로바이더로 정의하는 것입니다. 그러면 다음 단계에서 활용할 `ConfigService`에 주입할 수 있습니다. 아래 코드에서 `providers` 배열에 주의하십시오.

```typescript
import { DynamicModule, Module } from '@nestjs/common';
import { ConfigService } from './config.service';

@Module({})
export class ConfigModule {
  static register(options): DynamicModule {
    return {
      module: ConfigModule,
      providers: [
        {
          provide: 'CONFIG_OPTIONS',
          useValue: options,
        },
        ConfigService,
      ],
      exports: [ConfigService],
    };
  }
}
```

이제 `'CONFIG_OPTIONS'` 프로바이더를 `ConfigService`에 삽입하여 프로세스를 완료할 수 있습니다. 클래스가 아닌 토큰을 사용하여 프로바이더를 정의할 때 [여기에 설명 된대로](/fundamentals/custom-providers#non-class-based-provider-tokens) `@Inject()` 데코레이터를 사용해야 합니다.

```typescript
import * as dotenv from 'dotenv';
import * as fs from 'fs';
import { Injectable, Inject } from '@nestjs/common';
import { EnvConfig } from './interfaces';

@Injectable()
export class ConfigService {
  private readonly envConfig: EnvConfig;

  constructor(@Inject('CONFIG_OPTIONS') private options) {
    const filePath = `${process.env.NODE_ENV || 'development'}.env`;
    const envFile = path.resolve(__dirname, '../../', options.folder, filePath);
    this.envConfig = dotenv.parse(fs.readFileSync(envFile));
  }

  get(key: string): string {
    return this.envConfig[key];
  }
}
```

마지막 참고 사항: 단순화를 위해 위에서 문자열 기반 주입 토큰(`'CONFIG_OPTIONS'`)을 사용했지만 모범 사례는 이를 별도의 파일에 상수 (또는 `Symbol`)로 정의하고 해당 파일을 가져 오는 것입니다. 예를 들면:

```typescript
export const CONFIG_OPTIONS = 'CONFIG_OPTIONS';
```

### Example

이 장에 있는 코드의 전체 예제는 [여기](https://github.com/nestjs/nest/tree/master/sample/25-dynamic-modules)에서 찾을 수 있습니다.
