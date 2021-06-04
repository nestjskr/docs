### Lifecycle Events

Nest 애플리케이션과 모든 애플리케이션 요소에는 Nest에서 관리하는 수명주기가 있습니다. Nest는 주요 라이프 사이클 이벤트에 대한 가시성을 제공하는 **라이프 사이클 후크**를 제공하고, 발생시 조치(`모듈`, `주입 가능` 또는 `컨트롤러`에 등록된 코드 실행)할 수 있는 기능을 제공합니다.

#### Lifecycle sequence

다음 다이어그램은 애플리케이션이 부트스트랩된 시간부터 노드 프로세스가 종료될 때까지 주요 애플리케이션 라이프 사이클 이벤트의 순서를 보여줍니다. 전체 수명주기를 **초기화**, **실행중** 및 **종료**의 세단계로 나눌 수 있습니다. 이 수명주기를 사용하여 모듈 및 서비스의 적절한 초기화를 계획하고, 활성 연결을 관리하고, 종료 신호를 수신할 때 애플리케이션을 정상적으로 종료할 수 있습니다.

<figure><img src="/assets/lifecycle-events.png" /></figure>

#### Lifecycle events

라이프 사이클 이벤트는 애플리케이션 부트스트랩 및 종료중에 발생합니다. Nest는 다음 각 수명주기 이벤트에서 `모듈`, `인젝터블` 및 `컨트롤러`에 등록된 수명주기 후크 메서드를 호출합니다([아래](/fundamentals/lifecycle-events#application-shutdown)에 설명된대로 **종료 후크**를 먼저 사용 설정해야 합니다.). 위의 다이어그램에 표시된 것처럼 Nest는 적절한 기본 메서드를 호출하여 연결 수신을 시작하고 연결 수신을 중지합니다.

다음 표에서 `onModuleDestroy`, `beforeApplicationShutdown` 및 `onApplicationShutdown`은 명시적으로 `app.close()`를 호출하거나 프로세스가 특수 시스템 신호(예: SIGTERM)를 수신하고 애플리케이션 부트 스트랩에서 `enableShutdownHooks`를 올바르게 호출했습니다(아래 **애플리케이션 종료** 부분 참조).

| 라이프 사이클 후크 방법           | 후크 메서드 호출을 트리거하는 수명주기 이벤트                                                                                                                                                                   |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `onModuleInit()`                | 호스트 모듈의 종속성이 해결되면 호출됩니다.                                                                                                                                                    |
| `onApplicationBootstrap()`      | 모든 모듈이 초기화된 후 연결을 수신하기 전에 호출됩니다.                                                                                                                              |
| `onModuleDestroy()`\*           | 종료 신호(예: `SIGTERM`)가 수신된 후 호출됩니다.                                                                                                                                            |
| `beforeApplicationShutdown()`\* | 모든 `onModuleDestroy()` 핸들러가 완료된 후 호출됩니다(Promise가 해결 또는 거부됨). <br /> 완료되면(Promise가 해결되거나 거부됨) 모든 기존 연결이 닫힙니다(`app.close()` 호출됨). |
| `onApplicationShutdown()`\*     | 연결이 닫힌 후 호출됩니다(`app.close()`가 해결됩니다).                                                                                                                                                           |


이러한 이벤트의  `app.close()`를 명시적으로 호출하지 않는 경우 `SIGTERM`과 같은 시스템 신호와 함께 작동하도록 옵트 인해야 합니다. 아래의 [애플리케이션 종료](fundamentals/lifecycle-events#application-shutdown)를 참조하세요.

> warning **경고** 위에 나열된 수명주기 후크는 **요청 범위** 클래스에 대해 트리거되지 않습니다. 요청 범위 클래스는 애플리케이션 수명주기와 관련이 없으며 수명을 예측할 수 없습니다. 각 요청에 대해 독점적으로 생성되고 응답이 전송된 후 자동으로 가비지 수집됩니다.

#### Usage

각 라이프 사이클 후크는 인터페이스로 표시됩니다. 인터페이스는 TypeScript 컴파일 후에 존재하지 않기 때문에 기술적으로 선택 사항입니다. 그럼에도 불구하고 강력한 타이핑 및 편집기 도구의 이점을 얻으려면 이를 사용하는 것이 좋습니다. 라이프 사이클 후크를 등록하려면 적절한 인터페이스를 구현하십시오. 예를 들어 특정 클래스(예: Controller, Provider 또는 Module)에서 모듈 초기화 중에 호출할 메서드를 등록하려면 아래와 같이 `onModuleInit()` 메서드를 제공하여 `OnModuleInit` 인터페이스를 구현합니다.

```typescript
@@filename()
import { Injectable, OnModuleInit } from '@nestjs/common';

@Injectable()
export class UsersService implements OnModuleInit {
  onModuleInit() {
    console.log(`The module has been initialized.`);
  }
}
@@switch
import { Injectable } from '@nestjs/common';

@Injectable()
export class UsersService {
  onModuleInit() {
    console.log(`The module has been initialized.`);
  }
}
```

#### Asynchronous initialization

`OnModuleInit`및 `OnApplicationBootstrap` 후크를 사용하면 애플리케이션 초기화 프로세스를 연기할 수 있습니다( `Promise`를 반환하거나 메서드를 `async`로 표시하고 메서드 본문에서 비동기 메서드 완료를 `await`로 표시).

```typescript
@@filename()
async onModuleInit(): Promise<void> {
  await this.fetch();
}
@@switch
async onModuleInit() {
  await this.fetch();
}
```

#### Application shutdown

`onModuleDestroy()`, `beforeApplicationShutdown()` 및 `onApplicationShutdown()` 후크는 종료 단계에서 호출됩니다(`app.close()`에 대한 명시적 호출에 대한 응답으로 또는 SIGTERM과 같은 시스템 신호 수신시 선택). 이 기능은 종종 [Kubernetes](https://kubernetes.io/)와 함께 사용되어 컨테이너의 수명주기를 관리하고 dynos 또는 유사한 서비스의 경우 [Heroku](https://www.heroku.com/)에서 사용합니다.

종료 후크 리스너는 시스템 리소스를 사용하므로 기본적으로 비활성화됩니다. 종료 후크를 사용하려면 `enableShutdownHooks()`를 호출하여 **리스너를 활성화해야**합니다.

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Starts listening for shutdown hooks
  app.enableShutdownHooks();

  await app.listen(3000);
}
bootstrap();
```

> warning **경고** 고유한 플랫폼 제한으로 인해 NestJS는 Windows에서 애플리케이션 종료 후크를 제한적으로 지원합니다. `SIGINT`는 물론 `SIGBREAK` 및 어느 정도 `SIGHUP` - [자세히 알아보기](https://nodejs.org/api/process.html#process_signal_events)가 작동할 것으로 예상할 수 있습니다. 그러나 `SIGTERM`은 작업 관리자에서 프로세스를 죽이는 것은 무조건 "즉, 응용 프로그램이 이를 감지하거나 방지할 방법이 없기 때문에" Windows에서 작동하지 않습니다. 다음은 libuv의 일부 [관련 문서](https://docs.libuv.org/en/v1.x/signal.html)에서 `SIGINT`, `SIGBREAK` 등이 Windows에서 처리되는 방법에 대해 자세히 알아볼 수 있습니다. 또한 [Process Signal Events](https://nodejs.org/api/process.html#process_signal_events)의 Node.js 문서를 참조하세요.

> info **정보** `enableShutdownHooks`는 리스너를 시작하여 메모리를 소비합니다. 단일 노드 프로세스에서 여러 Nest 앱을 실행하는 경우(예: Jest로 병렬 테스트를 실행할 때) Node는 과도한 리스너 프로세스에 대해 불평할 수 있습니다. 따라서 `enableShutdownHooks`는 기본적으로 활성화되어 있지 않습니다. 단일 노드 프로세스에서 여러 인스턴스를 실행할 때 이 조건에 유의하십시오.

애플리케이션이 종료 신호를 수신하면 등록된 모든 `onModuleDestroy()`, `beforeApplicationShutdown()`, `onApplicationShutdown()` 메서드(위에서 설명한 순서대로)를 호출하고 해당 신호를 첫번째 매개변수로 사용합니다. 등록된 함수가 비동기 호출을 기다리는 경우(Promise를 반환) Nest는 promise가 해결되거나 거부될 때까지 시퀀스를 계속하지 않습니다.

```typescript
@@filename()
@Injectable()
class UsersService implements OnApplicationShutdown {
  onApplicationShutdown(signal: string) {
    console.log(signal); // e.g. "SIGINT"
  }
}
@@switch
@Injectable()
class UsersService implements OnApplicationShutdown {
  onApplicationShutdown(signal) {
    console.log(signal); // e.g. "SIGINT"
  }
}
```
