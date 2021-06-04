### Logger

Nest는 애플리케이션 부트스트랩 및 포착된 예외 표시(예: 시스템 로깅)와 같은 여러 다른 상황에서 사용되는 내장 텍스트 기반 로거와 함께 제공됩니다. 이 기능은 `@nestjs/common` 패키지의 `Logger` 클래스를 통해 제공됩니다. 다음을 포함하여 로깅 시스템의 동작을 완전히 제어할 수 있습니다.

- 로깅을 완전히 비활성화
- 세부 로그 수준 지정 (예: 디스플레이 오류, 경고, 디버그 정보 등)
- 기본 로거의 타임스탬프 재정의 (예: 날짜 형식으로 ISO8601 표준 사용)
- 기본 로거를 완전히 무시
- 기본 로거를 확장하여 사용자 지정
- 종속성 주입을 사용하여 애플리케이션 작성 및 테스트를 단순화합니다.

또한 내장 로거를 사용하거나 사용자 지정 구현을 만들어 애플리케이션 수준 이벤트 및 메시지를 기록할 수 있습니다.

고급 로깅 기능을 위해 [Winston](https://github.com/winstonjs/winston)과 같은 Node.js 로깅 패키지를 사용하여 완전한 맞춤형 프로덕션 등급 로깅 시스템을 구현할 수 있습니다.

#### Basic customization

로깅을 사용 중지하려면 `NestFactory.create()` 메서드에 두번째 인수로 전달된(선택사항) Nest 애플리케이션 옵션 객체에서 `logger` 속성을 `false`로 설정합니다.

```typescript
const app = await NestFactory.create(AppModule, {
  logger: false,
});
await app.listen(3000);
```

특정 로깅 수준을 사용하려면 다음과 같이 표시할 로그 수준을 지정하는 문자열 배열로 `logger` 속성을 설정합니다.

```typescript
const app = await NestFactory.create(AppModule, {
  logger: ['error', 'warn'],
});
await app.listen(3000);
```

배열의 값은` 'log'`, `'error'`, `'warn'`, `'debug'` 및 `'verbose'`의 조합일 수 있습니다.

> info **힌트** 기본 로거 메시지에서 색상을 비활성화하려면 `NO_COLOR` 환경 변수를 설정하십시오.

#### Custom implementation

`logger` 속성의 값을 `LoggerService` 인터페이스를 충족하는 객체로 설정하여 Nest에서 시스템 로깅을 위해 사용할 사용자 정의 로거 구현을 제공할 수 있습니다. 예를 들어 Nest에 다음과 같이 내장된 전역 자바 스크립트 `console` 객체(`LoggerService` 인터페이스를 구현함)를 사용하도록 지시할 수 있습니다.

```typescript
const app = await NestFactory.create(AppModule, {
  logger: console,
});
await app.listen(3000);
```

사용자 정의 로거를 구현하는 것은 간단합니다. 아래와 같이 `LoggerService` 인터페이스의 각 메소드를 구현하기만 하면 됩니다.

```typescript
import { LoggerService } from '@nestjs/common';

export class MyLogger implements LoggerService {
  log(message: string) {
    /* your implementation */
  }
  error(message: string, trace: string) {
    /* your implementation */
  }
  warn(message: string) {
    /* your implementation */
  }
  debug(message: string) {
    /* your implementation */
  }
  verbose(message: string) {
    /* your implementation */
  }
}
```

그런 다음 Nest 애플리케이션 옵션 객체의 `logger` 속성을 통해 `MyLogger` 인스턴스를 제공할 수 있습니다.

```typescript
const app = await NestFactory.create(AppModule, {
  logger: new MyLogger(),
});
await app.listen(3000);
```

이 기술은 간단하지만 `MyLogger` 클래스에 대한 종속성 주입을 사용하지 않습니다. 이로 인해 특히 테스트시 몇가지 문제가 발생할 수 있으며 `MyLogger`의 재사용 가능성이 제한됩니다. 더 나은 솔루션은 아래의 <a href="techniques/logger#dependency-injection">종속성 주입</a> 섹션을 참조하세요.

#### Extend built-in logger

처음부터 로거를 작성하는 대신 내장 `Logger` 클래스를 확장하고 기본 구현의 선택된 동작을 재정의하여 요구 사항을 충족할 수 있습니다.

```typescript
import { Logger } from '@nestjs/common';

export class MyLogger extends Logger {
  error(message: string, trace: string) {
    // add your tailored logic here
    super.error(message, trace);
  }
}
```

아래 <a href="techniques/logger#using-the-logger-for-application-logging">애플리케이션 로깅을 위해 로거 사용</a> 섹션에 설명된대로 기능 모듈에서 이러한 확장 로거를 사용할 수 있습니다.

애플리케이션 옵션 객체의 `logger` 속성(위의 [사용자 정의 구현](/techniques/logger#custom-logger-implementation) 섹션에 표시됨)을 통해 인스턴스를 전달하거나 [종속성 주입](/techniques/logger#dependency-injection)에 표시된 기술을 사용하여 Nest가 시스템 로깅에 확장 로거를 사용하도록 지시할 수 있습니다. 아래 섹션. 이 경우 위의 샘플 코드에 표시된대로 `super`를 호출하여 Nest가 기본 제공 기능에 의존할 수 있도록 상위(내장) 클래스에 특정 로그 메서드 호출을 위임해야 합니다. 기대합니다.

<app-banner-courses></app-banner-courses>

#### Dependency injection

고급 로깅 기능을 사용하려면 종속성 주입을 활용해야 합니다. 예를 들어, 로거에 `ConfigService`를 삽입하여 사용자 정의하고 사용자 정의 로거를 다른 컨트롤러 및/또는 프로바이더에 삽입할 수 있습니다. 사용자 정의 로거에 대한 종속성 주입을 활성화하려면 `LoggerService`를 구현하는 클래스를 만들고 해당 클래스를 일부 모듈에서 프로바이더로 등록합니다. 예를 들어 다음을 수행할 수 있습니다.

1. 이전 섹션에서 볼 수 있듯이 내장 `Logger`를 확장하거나 완전히 재정의하는 `MyLogger` 클래스를 정의합니다. `LoggerService` 인터페이스를 구현해야 합니다.
2. 아래와 같이 `LoggerModule`을 생성하고 해당 모듈에서 `MyLogger`를 제공합니다.

```typescript
import { Module } from '@nestjs/common';
import { MyLogger } from './my-logger.service';

@Module({
  providers: [MyLogger],
  exports: [MyLogger],
})
export class LoggerModule {}
```

이 구성을 사용하면 이제 다른 모듈에서 사용할 사용자 정의 로거를 제공합니다. `MyLogger` 클래스가 모듈의 일부이기 때문에 종속성 주입을 사용할 수 있습니다 (예: `ConfigService` 주입). 시스템 로깅(예: 부트 스트랩 및 오류 처리)을 위해 Nest에서 사용하기 위해 이 사용자 지정 로거를 제공하는 데 필요한 기술이 하나 더 있습니다.

애플리케이션 인스턴스화(`NestFactory.create()`)는 모듈의 컨텍스트 외부에서 발생하기 때문에 초기화의 일반적인 종속성 주입 단계에 참여하지 않습니다. 따라서 적어도 하나의 애플리케이션 모듈이 `LoggerModule`을 가져와 Nest를 트리거하여 `MyLogger` 클래스의 단일 인스턴스를 인스턴스화하도록 해야 합니다. 그런 다음 Nest에 다음 구성으로 `MyLogger`의 동일한 싱글톤 인스턴스를 사용하도록 지시 할 수 있습니다.

```typescript
const app = await NestFactory.create(AppModule, {
  logger: false,
});
app.useLogger(app.get(MyLogger));
await app.listen(3000);
```

여기서는 `NestApplication` 인스턴스의 `get()` 메서드를 사용하여 `MyLogger` 객체의 싱글톤 인스턴스를 검색합니다. 이 기술은 기본적으로 Nest에서 사용할 로거 인스턴스를 "주입"하는 방법입니다. `app.get()` 호출은 `MyLogger`의 싱글톤 인스턴스를 검색하며, 위에서 설명한대로 해당 인스턴스가 다른 모듈에 처음 주입되는 것에 의존합니다.

이 솔루션의 유일한 단점은 첫번째 초기화 메시지가 어디에도 인쇄되지 않으므로 드물게 중요한 초기화 오류를 놓칠 수 있다는 것입니다.
또는 기본 로거를 사용하여 첫번째 초기화 메시지를 인쇄한 다음 사용자 지정 로거로 전환할 수 있습니다.

```typescript
const app = await NestFactory.create(AppModule);
app.useLogger(app.get(MyLogger));
await app.listen(3000);
```

이 `MyLogger` 프로바이더를 피쳐 클래스에 삽입하여 Nest 시스템 로깅과 애플리케이션 로깅 모두에서 일관된 로깅 동작을 보장할 수도 있습니다. 자세한 내용은 아래의 <a href="techniques/logger#using-the-logger-for-application-logging">애플리케이션 로깅에 로거 사용</a> 및 <a href="techniques/logger#injecting-a-custom-logger">사용자 정의 로거 삽입</a>을 참조하세요.

#### Using the logger for application logging

위의 여러 기술을 결합하여 Nest 시스템 로깅과 자체 애플리케이션 이벤트/메시지 로깅 모두에서 일관된 동작과 형식을 제공할 수 있습니다.

각 서비스의 `@nestjs/common`에서 `Logger` 클래스를 인스턴스화하는 것이 좋습니다. 다음과 같이 `Logger` 생성자에서 `context` 인수로 서비스 이름을 제공할 수 있습니다.

```typescript
import { Logger, Injectable } from '@nestjs/common';

@Injectable()
class MyService {
  private readonly logger = new Logger(MyService.name);

  doSomething() {
    this.logger.log('Doing something...');
  }
}
```

기본 로거 구현에서 `context`는 아래 예의 `NestFactory`와 같이 대괄호 안에 인쇄됩니다.

```bash
[Nest] 19096   - 12/08/2019, 7:12:59 AM   [NestFactory] Starting Nest application...
```

`app.useLogger()`를 통해 커스텀 로거를 제공하면 실제로 Nest에서 내부적으로 사용됩니다. 즉, 우리 코드는 구현에 구애받지 않고 `app.useLogger()`를 호출하여 사용자 정의 로거에 대한 기본 로거를 쉽게 대체 할 수 있습니다.

이렇게하면 이전 섹션의 단계를 따르고 `app.useLogger(app.get(MyLogger))`를 호출하면 `MyService에서`에서 `this.logger.log`를 다음과 같이 호출하면 `MyLogger` 인스턴스에서 메소드 `log`가 호출됩니다. .

대부분의 경우에 적합합니다. 그러나 사용자 지정 메서드 추가 및 호출과 같은 추가 사용자 지정이 필요한 경우 다음 섹션으로 이동하십시오.

#### Injecting a custom logger

시작하려면 다음과 같은 코드로 내장 로거를 확장하십시오. 각 기능 모듈에서 `MyLogger`의 고유한 인스턴스를 갖도록 `Logger` 클래스의 구성 메타 데이터로 `scope` 옵션을 제공하고 [transient](/fundamentals/injection-scopes) 범위를 지정합니다. 이 예에서는 개별 `Logger` 메서드(`log()`, `warn()` 등)를 확장하지 않습니다. 하지만 그렇게 선택할 수도 있습니다.

```typescript
import { Injectable, Scope, Logger } from '@nestjs/common';

@Injectable({ scope: Scope.TRANSIENT })
export class MyLogger extends Logger {
  customLog() {
    this.log('Please feed the cat!');
  }
}
```

다음으로 다음과 같은 구성으로 `LoggerModule`을 만듭니다.

```typescript
import { Module } from '@nestjs/common';
import { MyLogger } from './my-logger.service';

@Module({
  providers: [MyLogger],
  exports: [MyLogger],
})
export class LoggerModule {}
```

다음으로 `LoggerModule`을 기능 모듈로 가져옵니다. 기본 `Logger`를 확장했기 때문에 `setContext` 메소드를 사용하는 것이 편리합니다. 따라서 다음과 같이 컨텍스트 인식 사용자 정의 로거 사용을 시작할 수 있습니다.

```typescript
import { Injectable } from '@nestjs/common';
import { MyLogger } from './my-logger.service';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  constructor(private myLogger: MyLogger) {
    // Due to transient scope, CatsService has its own unique instance of MyLogger,
    // so setting context here will not affect other instances in other services
    this.myLogger.setContext('CatsService');
  }

  findAll(): Cat[] {
    // You can call all the default methods
    this.myLogger.warn('About to return cats!');
    // And your custom methods
    this.myLogger.customLog();
    return this.cats;
  }
}
```

마지막으로 Nest가 아래와 같이 `main.ts` 파일에서 커스텀 로거의 인스턴스를 사용하도록 지시합니다. 물론 이 예제에서는 실제로 로거 동작을 사용자 정의하지 않았으므로 (`log()`, `warn()`등과 같은 `Logger` 메서드를 확장하여)이 단계가 실제로 필요하지 않습니다. 하지만 이러한 메소드에 커스텀 로직을 추가하고 Nest가 동일한 구현을 사용하도록 하려면 **필요**합니다.

```typescript
const app = await NestFactory.create(AppModule, {
  logger: false,
});
app.useLogger(new MyLogger());
await app.listen(3000);
```

`logger: false`를 `NestFactory.create`에 제공하면 `useLogger`를 호출할 때까지 아무것도 기록되지 않으므로 몇가지 중요한 초기화 오류를 놓칠 수 있습니다. 초기 메시지중 일부가 기본 로거로 기록된다는 점이 마음에 들지 않으면 `logger: false` 옵션을 생략할 수 있습니다.

#### Use external logger

프로덕션 애플리케이션에는 고급 필터링, 형식 지정 및 중앙 집중식 로깅을 비롯한 특정 로깅 요구 사항이 있는 경우가 많습니다. Nest의 내장 로거는 Nest 시스템 동작을 모니터링하는데 사용되며 개발중 기능 모듈의 기본 형식화된 텍스트 로깅에도 유용할 수 있지만 프로덕션 애플리케이션은 종종 [Winston](https://github.com/winstonjs/winston)과 같은 전용 로깅 모듈을 활용합니다. 표준 Node.js 애플리케이션과 마찬가지로 Nest에서 이러한 모듈을 최대한 활용할 수 있습니다.
