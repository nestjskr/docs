### Custom providers

이전 장에서 **DI 종속성 주입**의 다양한 측면과 Nest에서 사용되는 방법에 대해 설명했습니다. 이에 대한 한가지 예는 인스턴스(종종 서비스 프로바이더)를 클래스에 주입하는데 사용되는 [생성자 기반](/providers#dependency-injection) 종속성 주입입니다. 의존성 주입이 기본적인 방식으로 Nest 코어에 내장되어 있다는 사실에 놀라지 않을 것입니다. 지금까지 우리는 하나의 주요 패턴만 살펴보았습니다. 애플리케이션이 더 복잡해짐에 따라 DI 시스템의 전체기능을 활용해야할 수도 있으므로 더 자세히 살펴 보겠습니다.

#### DI fundamentals

종속성 주입은 대신 IoC 컨테이너(이 경우 NestJS 런타임 시스템)에 종속성 인스턴스화를 위임하는 [IoC(inversion of control)](https://en.wikipedia.org/wiki/Inversion_of_control) 기술입니다. 자신의 코드에서 명령적으로 수행하는 것입니다. [프로바이더 장](/providers)에서 이 예제에서 어떤 일이 발생하는지 살펴보겠습니다.

먼저 프로바이더를 정의합니다. `@Injectable()` 데코레이터는 `CatsService` 클래스를 프로바이더로 표시합니다.

```typescript
@@filename(cats.service)
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  findAll(): Cat[] {
    return this.cats;
  }
}
@@switch
import { Injectable } from '@nestjs/common';

@Injectable()
export class CatsService {
  constructor() {
    this.cats = [];
  }

  findAll() {
    return this.cats;
  }
}
```

그런 다음 Nest가 프로바이더를 컨트롤러 클래스에 주입하도록 요청합니다.

```typescript
@@filename(cats.controller)
import { Controller, Get } from '@nestjs/common';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
@@switch
import { Controller, Get, Bind, Dependencies } from '@nestjs/common';
import { CatsService } from './cats.service';

@Controller('cats')
@Dependencies(CatsService)
export class CatsController {
  constructor(catsService) {
    this.catsService = catsService;
  }

  @Get()
  async findAll() {
    return this.catsService.findAll();
  }
}
```

마지막으로 Nest IoC 컨테이너에 프로바이더를 등록합니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class AppModule {}
```

이 작업을 수행하기 위해 정확히 어떤 일이 일어나고 있습니까? 프로세스에는 세가지 주요 단계가 있습니다.

1. `cats.service.ts`에서 `@Injectable()` 데코레이터는 `CatsService` 클래스를 Nest IoC 컨테이너에서 관리할 수 있는 클래스로 선언합니다.
2. `cats.controller.ts`에서 `CatsController`는 생성자(constructor) 주입으로 `CatsService` 토큰에 대한 종속성을 선언합니다.

```typescript
  constructor(private catsService: CatsService)
```

1. `app.module.ts`에서 `CatsService` 토큰을 `cats.service.ts` 파일의 `CatsService` 클래스와 연관시킵니다. 이 연결 (_registration_이라고도 함)이 어떻게 발생하는지 정확히 [아래를 참조](/fundamentals/custom-providers#standard-providers)합니다.

Nest IoC 컨테이너가 `CatsController`를 인스턴스화할 때 먼저 종속성\*을 찾습니다. `CatsService` 종속성을 찾으면 등록단계(위 #3)에 따라 `CatsService` 클래스를 반환하는 `CatsService` 토큰에 대한 조회를 수행합니다. `SINGLETON` 범위(기본 동작)를 가정하면 Nest는 `CatsService`의 인스턴스를 만들고 캐시한 다음 반환하거나 이미 캐시된 경우 기존 인스턴스를 반환합니다.

\*이 설명은 요점을 설명하기 위해 약간 단순화되었습니다. 우리가 살펴본 중요한 영역중 하나는 종속성에 대한 코드 분석 프로세스가 매우 정교하고 애플리케이션 부트스트랩중에 발생한다는 것입니다. 한가지 주요 기능은 종속성 분석(또는 "종속성 그래프 생성")이 **전이적**(transitive)이라는 것입니다. 위의 예에서 `CatsService` 자체에 종속성이 있으면 이것도 해결됩니다. 종속성 그래프는 종속성이 올바른 순서(기본적으로 "상향(bottom up)")로 해결되도록 합니다. 이 메커니즘은 개발자가 복잡한 종속성 그래프를 관리할 필요가 없도록 합니다.

<app-banner-courses></app-banner-courses>

#### Standard providers

`@Module()` 데코레이터를 자세히 살펴 보겠습니다. `app.module`에서 다음을 선언합니다.

```typescript
@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
```

`providers` 속성은 `providers`의 배열을 받습니다. 지금까지 클래스 이름 목록을 통해 이러한 프로바이더를 제공했습니다. 실제로 `providers: [CatsService]` 구문은 보다 완전한 구문을 위한 축약형입니다.

```typescript
providers: [
  {
    provide: CatsService,
    useClass: CatsService,
  },
];
```

이제이 명시적인 구성을 보았으므로 등록 프로세스를 이해할 수 있습니다. 여기서는 `CatsService` 토큰을 `CatsService` 클래스와 명확하게 연관시킵니다. 축약 표기법은 가장 일반적인 사용사례를 단순화하기 위한 편의일뿐입니다. 토큰은 동일한 이름으로 클래스의 인스턴스를 요청하는 데 사용됩니다.

#### Custom providers

귀하의 요구 사항이 _표준 프로바이더_ 에서 제공하는 것 이상으로 넘어가면 어떻게됩니까? 다음은 몇가지 예입니다.

- Nest가 클래스를 인스턴스화(또는 캐시된 인스턴스 반환)하는 대신 사용자 지정 인스턴스를 만들고 싶습니다.
- 두번째 종속성에서 기존 클래스를 재사용하려는 경우
- 테스트를 위해 모의(mock) 버전으로 클래스를 재정의하려는 경우

Nest를 사용하면 이러한 경우를 처리할 사용자 지정 프로바이더를 정의할 수 있습니다. 사용자 지정 프로바이더를 정의하는 여러 방법을 제공합니다. 그것들을 살펴 보겠습니다.

#### Value providers: `useValue`

`useValue` 구문은 상수값을 삽입하거나 Nest 컨테이너에 외부 라이브러리를 넣거나 실제 구현을 모의 객체로 대체하는 데 유용합니다. Nest가 테스트 목적으로 모의 `CatsService`를 사용하도록 강제하고 싶다고 가정해 보겠습니다.

```typescript
import { CatsService } from './cats.service';

const mockCatsService = {
  /* mock implementation
  ...
  */
};

@Module({
  imports: [CatsModule],
  providers: [
    {
      provide: CatsService,
      useValue: mockCatsService,
    },
  ],
})
export class AppModule {}
```

이 예에서 `CatsService` 토큰은 `mockCatsService` 모의 객체로 확인됩니다. `useValue`에는 값이 필요합니다. 이 경우 교체 할 `CatsService` 클래스와 동일한 인터페이스를 가진 리터럴 객체입니다. TypeScript의 [구조적 유형](https://www.typescriptlang.org/docs/handbook/type-compatibility.html)때문에 리터럴 객체 또는 `new`로 인스턴스화된 클래스 인스턴스를 포함하여 호환되는 인터페이스가 있는 모든 객체를 사용할 수 있습니다.

#### Non-class-based provider tokens

지금까지 우리는 클래스 이름을 프로바이더 토큰(`providers` 배열에 나열된 프로바이더의 `provide` 속성 값)으로 사용했습니다. 이는 토큰이 클래스 이름이기도 한 [생성자 기반 주입](/providers#dependency-injection)과 함께 사용되는 표준 패턴과 일치합니다. (이 개념이 완전히 명확하지 않은 경우 토큰에 대한 복습은 [DI 기초](/fundamentals/custom-providers#di-fundamentals)를 다시 참조하세요). 때때로 우리는 DI 토큰으로 문자열이나 기호를 사용할 수 있는 유연성을 원할 수 있습니다. 예를 들면:

```typescript
import { connection } from './connection';

@Module({
  providers: [
    {
      provide: 'CONNECTION',
      useValue: connection,
    },
  ],
})
export class AppModule {}
```

이 예에서는 문자열 값 토큰 (`'CONNECTION'`)을 외부 파일에서 가져온 기존 `connection` 객체와 연결합니다.

> warning **알림** 문자열을 토큰 값으로 사용하는 것 외에도 자바스크립트 [심볼 symbols](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol) 또는 TypeScript [열거형 enums](https://www.typescriptlang.org/docs/handbook/enums.html)를 사용할 수 있습니다.

이전에 표준 [생성자 기반 주입](/providers#dependency-injection) 패턴을 사용하여 공급자를 주입하는 방법을 살펴 보았습니다. 이 패턴은 종속성이 클래스 이름으로 선언되도록 **요구**합니다. `'CONNECTION'` 커스텀 프로바이더는 문자열 값 토큰을 사용합니다. 그러한 프로바이더를 주입하는 방법을 살펴 보겠습니다. 이를 위해 `@Inject()` 데코레이터를 사용합니다. 이 데코레이터는 단일인수인 토큰을 받습니다.

```typescript
@@filename()
@Injectable()
export class CatsRepository {
  constructor(@Inject('CONNECTION') connection: Connection) {}
}
@@switch
@Injectable()
@Dependencies('CONNECTION')
export class CatsRepository {
  constructor(connection) {}
}
```

> info **힌트** `@Inject()` 데코레이터는 `@nestjs/common` 패키지에서 가져옵니다.

위의 예에서는 설명 목적으로 `'CONNECTION'` 문자열을 직접 사용하지만 깔끔한 코드 구성을 위해 `constants.ts`와 같은 별도의 파일에 토큰을 정의하는 것이 가장 좋습니다. 자체 파일에 정의되고 필요한 곳에 가져오는 기호 또는 열거형처럼 처리하십시오.

#### Class providers: `useClass`

`useClass` 구문을 사용하면 토큰이 확인해야 하는 클래스를 동적으로 결정할 수 있습니다. 예를 들어 추상(또는 기본) `ConfigService` 클래스가 있다고 가정합니다. 현재 환경에 따라 Nest가 구성 서비스의 다른 구현을 제공하기를 원합니다. 다음 코드는 이러한 전략을 구현합니다.

```typescript
const configServiceProvider = {
  provide: ConfigService,
  useClass:
    process.env.NODE_ENV === 'development'
      ? DevelopmentConfigService
      : ProductionConfigService,
};

@Module({
  providers: [configServiceProvider],
})
export class AppModule {}
```

이 코드 샘플에서 몇가지 세부 사항을 살펴 보겠습니다. 먼저 리터럴 객체로 `configServiceProvider`를 정의한 다음 모듈 데코레이터의 `providers` 속성에 전달합니다. 이것은 약간의 코드 구성에 불과하지만 이 장에서 지금까지 사용한 예제와 기능적으로 동일합니다.

또한 토큰으로 `ConfigService` 클래스 이름을 사용했습니다. `ConfigService`에 의존하는 모든 클래스의 경우 Nest는 제공된 클래스(`DevelopmentConfigService` 또는 `ProductionConfigService`)의 인스턴스를 삽입하여 다른 곳에서 선언되었을 수 있는 기본 구현(예: `@Injectable()` 데코레이터로 선언된 `ConfigService`)을 재정의합니다.

#### Factory providers: `useFactory`

`useFactory` 구문을 사용하면 **동적으로** 프로바이더를 만들 수 있습니다. 실제 프로바이더는 팩토리 함수에서 반환된 값으로 제공됩니다. 팩토리 함수는 필요에 따라 간단하거나 복잡할 수 있습니다. 단순한 팩토리는 다른 프로바이더에 의존하지 않을 수 있습니다. 더 복잡한 팩토리는 결과를 계산하는 데 필요한 다른 프로바이더를 자체적으로 주입할 수 있습니다. 후자의 경우 팩토리 프로바이더 구문에는 한 쌍의 관련 메커니즘이 있습니다.

1. 팩토리 함수는 (선택 사항) 인수를 받을 수 있습니다.
2. (선택 사항) `inject` 속성은 Nest가 인스턴스화 프로세스중에 확인하고 팩토리 함수에 인수로 전달할 프로바이더 배열을 허용합니다. 두 목록은 상호 연관되어야 합니다. Nest는 `inject` 목록의 인스턴스를 동일한 순서로 팩토리 함수에 대한 인수로 전달합니다.

아래 예가 이를 보여줍니다.

```typescript
@@filename()
const connectionFactory = {
  provide: 'CONNECTION',
  useFactory: (optionsProvider: OptionsProvider) => {
    const options = optionsProvider.get();
    return new DatabaseConnection(options);
  },
  inject: [OptionsProvider],
};

@Module({
  providers: [connectionFactory],
})
export class AppModule {}
@@switch
const connectionFactory = {
  provide: 'CONNECTION',
  useFactory: (optionsProvider) => {
    const options = optionsProvider.get();
    return new DatabaseConnection(options);
  },
  inject: [OptionsProvider],
};

@Module({
  providers: [connectionFactory],
})
export class AppModule {}
```

#### Alias providers: `useExisting`

`useExisting` 구문을 사용하면 기존 프로바이더의 별칭을 만들 수 있습니다. 이것은 동일한 프로바이더에 액세스하는 두가지 방법을 만듭니다. 아래 예에서 (문자열 기반) 토큰 `'AliasedLoggerService'`는 (클래스 기반) 토큰 `LoggerService`의 별칭입니다. `'AliasedLoggerService'`와 `LoggerService`에 대한 두개의 서로 다른 종속성이 있다고 가정합니다. 두 종속성이 `SINGLETON` 범위로 지정되면 둘 다 동일한 인스턴스로 확인됩니다.

```typescript
@Injectable()
class LoggerService {
  /* implementation details */
}

const loggerAliasProvider = {
  provide: 'AliasedLoggerService',
  useExisting: LoggerService,
};

@Module({
  providers: [LoggerService, loggerAliasProvider],
})
export class AppModule {}
```

#### Non-service based providers

프로바이더가 서비스를 제공하는 경우가 많지만 해당 용도에 국한되지는 않습니다. 프로바이더는 **모든** 값을 제공할 수 있습니다. 예를 들어 프로바이더는 아래와 같이 현재 환경을 기반으로 구성 객체의 배열을 제공할 수 있습니다.

```typescript
const configFactory = {
  provide: 'CONFIG',
  useFactory: () => {
    return process.env.NODE_ENV === 'development' ? devConfig : prodConfig;
  },
};

@Module({
  providers: [configFactory],
})
export class AppModule {}
```

#### Export custom provider

다른 프로바이더와 마찬가지로 커스텀 프로바이더는 선언 모듈로 범위가 지정됩니다. 다른 모듈에 표시하려면 내보내야 합니다. 커스텀 프로바이더를 내보내려면 해당 토큰 또는 전체 프로바이더 객체를 사용할 수 있습니다.

다음 예제는 토큰을 사용한 내보내기를 보여줍니다.

```typescript
@@filename()
const connectionFactory = {
  provide: 'CONNECTION',
  useFactory: (optionsProvider: OptionsProvider) => {
    const options = optionsProvider.get();
    return new DatabaseConnection(options);
  },
  inject: [OptionsProvider],
};

@Module({
  providers: [connectionFactory],
  exports: ['CONNECTION'],
})
export class AppModule {}
@@switch
const connectionFactory = {
  provide: 'CONNECTION',
  useFactory: (optionsProvider) => {
    const options = optionsProvider.get();
    return new DatabaseConnection(options);
  },
  inject: [OptionsProvider],
};

@Module({
  providers: [connectionFactory],
  exports: ['CONNECTION'],
})
export class AppModule {}
```

또는 전체 프로바이더 객체를 사용하여 내 보냅니다.

```typescript
@@filename()
const connectionFactory = {
  provide: 'CONNECTION',
  useFactory: (optionsProvider: OptionsProvider) => {
    const options = optionsProvider.get();
    return new DatabaseConnection(options);
  },
  inject: [OptionsProvider],
};

@Module({
  providers: [connectionFactory],
  exports: [connectionFactory],
})
export class AppModule {}
@@switch
const connectionFactory = {
  provide: 'CONNECTION',
  useFactory: (optionsProvider) => {
    const options = optionsProvider.get();
    return new DatabaseConnection(options);
  },
  inject: [OptionsProvider],
};

@Module({
  providers: [connectionFactory],
  exports: [connectionFactory],
})
export class AppModule {}
```
