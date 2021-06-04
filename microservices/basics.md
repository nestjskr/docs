### Overview

기존(모놀리식이라고도 함) 애플리케이션 아키텍처 외에도 Nest는 기본적으로 마이크로서비스 아키텍처 스타일의 개발을 지원합니다. 종속성 주입, 데코레이터, 예외 필터, 파이프, 가드 및 인터셉터와 같이 이 문서의 다른 부분에서 논의된 대부분의 개념은 마이크로 서비스에 동일하게 적용됩니다. 가능한 경우 Nest는 HTTP 기반 플랫폼, WebSocket 및 마이크로서비스에서 동일한 구성요소를 실행할 수 있도록 구현 세부정보를 추상화합니다. 이 섹션에서는 마이크로서비스와 관련된 Nest의 측면을 다룹니다.

Nest에서 마이크로서비스는 기본적으로 HTTP와 다른 **전송** 계층을 사용하는 애플리케이션입니다.

<figure><img src="/assets/Microservices_1.png" /></figure>

Nest는 여러 마이크로서비스 인스턴스간에 메시지를 전송하는 **전송자 transporter**라고 하는 여러가지 기본제공 전송 계층 구현을 지원합니다. 대부분의 전송자는 기본적으로 **요청-응답** 및 **이벤트 기반** 메시지 스타일을 모두 지원합니다. Nest는 요청-응답 및 이벤트 기반 메시징 모두에 대해 표준 인터페이스 뒤에 있는 각 전송자의 구현 세부정보를 추상화합니다. 따라서 애플리케이션 코드에 영향을 주지않고 특정 전송 레이어의 특정 안정성 또는 성능 기능을 활용하기 위해 한 전송 레이어에서 다른 전송 레이어로 쉽게 전환할 수 있습니다.

#### Installation

마이크로 서비스 빌드를 시작하려면 먼저 필요한 패키지를 설치하세요.

```bash
$ npm i --save @nestjs/microservices
```

#### Getting started

마이크로 서비스를 인스턴스화하려면 `NestFactory` 클래스의 `createMicroservice()` 메소드를 사용하십시오.

```typescript
@@filename(main)
import { NestFactory } from '@nestjs/core';
import { Transport, MicroserviceOptions } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.TCP,
    },
  );
  app.listen(() => console.log('Microservice is listening'));
}
bootstrap();
@@switch
import { NestFactory } from '@nestjs/core';
import { Transport, MicroserviceOptions } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.createMicroservice(AppModule, {
    transport: Transport.TCP,
  });
  app.listen(() => console.log('Microservice is listening'));
}
bootstrap();
```

> info **힌트** 마이크로서비스는 기본적으로 **TCP** 전송 레이어를 사용합니다.

`createMicroservice()` 메소드의 두번째 인수는 `options` 객체입니다. 이 객체는 두개의 멤버로 구성될 수 있습니다.

<table>
  <tr>
    <td><code>transport</code></td>
    <td>전송자를 지정합니다(예: <code>Transport.NATS</code>).</td>
  </tr>
  <tr>
    <td><code>options</code></td>
    <td>전송자 동작을 결정하는 전송자별 옵션 개체</td>
  </tr>
</table>
<p>
  <code>options</code> 객체는 선택한 전송자에 따라 다릅니다. <strong>TCP</strong> 전송자는 아래 설명된 속성을 노출합니다. 다른 전송자(예: Redis, MQTT 등)의 경우 사용가능한 옵션에 대한 설명은 관련장을 참조하십시오.
</p>
<table>
  <tr>
    <td><code>host</code></td>
    <td>연결 호스트 이름</td>
  </tr>
  <tr>
    <td><code>port</code></td>
    <td>연결 포트</td>
  </tr>
  <tr>
    <td><code>retryAttempts</code></td>
    <td>메시지 재시도 횟수(기본값: <code>0</code>)</td>
  </tr>
  <tr>
    <td><code>retryDelay</code></td>
    <td>메시지 재시도 간격(밀리 초)(기본값: <code>0</code>)</td>
  </tr>
</table>

#### Patterns

마이크로서비스는 **패턴**으로 메시지와 이벤트를 모두 인식합니다. 패턴은 리터럴 객체 또는 문자열과 같은 일반값입니다. 패턴은 자동으로 직렬화되어 메시지의 데이터 부분과 함께 네트워크를 통해 전송됩니다. 이러한 방식으로 메시지 보낸 사람과 소비자는 핸들러가 어떤 요청을 사용하는지 조정할 수 있습니다.

#### Request-response

요청-응답 메시지 스타일은 다양한 외부 서비스간에 메시지를 **교환**해야할 때 유용합니다. 이 패러다임을 사용하면 서비스가 실제로 메시지를 수신했는지 확인할 수 있습니다(메시지 ACK 프로토콜을 수동으로 구현할 필요없이). 그러나 요청-응답 패러다임이 항상 최선의 선택은 아닙니다. 예를 들어 [Kafka](https://docs.confluent.io/3.0.0/streams/) 또는 [NATS streaming](https://github.com/nats-io/node-nats-streaming)과 같이 로그 기반 지속성을 사용하는 스트리밍 전송기는 다양한 문제를 해결하는 데 최적화되어 있으며 이벤트 메시징 패러다임에 더 잘 부합합니다(자세한 내용은 아래의 [이벤트 기반 메시징](/microservices/basics#event-based) 참조).

요청-응답 메시지 유형을 활성화하기 위해 Nest는 두개의 논리 채널을 만듭니다. 하나는 데이터 전송을 담당하고 다른 하나는 수신 응답을 기다립니다. [NATS](https://nats.io/)와 같은 일부 기본 전송의 경우 이 이중 채널 지원은 기본적으로 제공됩니다. 다른 경우 Nest는 별도의 채널을 수동으로 생성하여 보상합니다. 이에 대한 오버헤드가 있을 수 있으므로 요청-응답 메시지 스타일이 필요하지 않은 경우 이벤트 기반 방법 사용을 고려해야합니다.

요청-응답 패러다임을 기반으로 메시지 핸들러를 생성하려면 `@nestjs/microservices` 패키지에서 가져온 `@MessagePattern()` 데코레이터를 사용합니다. 이 데코레이터는 애플리케이션의 진입점이므로 [controller](/controllers) 클래스내에서만 사용해야합니다. 프로바이더 내부에서 사용하는 것은 단순히 Nest 런타임에서 무시되므로 아무런 효과가 없습니다.

```typescript
@@filename(math.controller)
import { Controller } from '@nestjs/common';
import { MessagePattern } from '@nestjs/microservices';

@Controller()
export class MathController {
  @MessagePattern({ cmd: 'sum' })
  accumulate(data: number[]): number {
    return (data || []).reduce((a, b) => a + b);
  }
}
@@switch
import { Controller } from '@nestjs/common';
import { MessagePattern } from '@nestjs/microservices';

@Controller()
export class MathController {
  @MessagePattern({ cmd: 'sum' })
  accumulate(data) {
    return (data || []).reduce((a, b) => a + b);
  }
}
```

위 코드에서 `accumulate()` **메시지 핸들러**는 `{{ '{' }} cmd: 'sum' {{ '}' }}` 메시지 패턴을 충족하는 메시지를 수신합니다. 메시지 핸들러는 클라이언트에서 전달된 `data`인 단일인수를 사용합니다. 이 경우 데이터는 누적될 숫자의 배열입니다.

#### Asynchronous responses

메시지 핸들러는 동기식 또는 **비동기식**으로 응답할 수 있습니다. 따라서 `async` 메소드가 지원됩니다.

```typescript
@@filename()
@MessagePattern({ cmd: 'sum' })
async accumulate(data: number[]): Promise<number> {
  return (data || []).reduce((a, b) => a + b);
}
@@switch
@MessagePattern({ cmd: 'sum' })
async accumulate(data) {
  return (data || []).reduce((a, b) => a + b);
}
```

메시지 핸들러는 `Observable`을 반환할 수도 있습니다. 이 경우 스트림이 완료될 때까지 결과값이 내보내집니다.

```typescript
@@filename()
@MessagePattern({ cmd: 'sum' })
accumulate(data: number[]): Observable<number> {
  return from([1, 2, 3]);
}
@@switch
@MessagePattern({ cmd: 'sum' })
accumulate(data: number[]): Observable<number> {
  return from([1, 2, 3]);
}
```

위의 예에서 메시지 핸들러는 **3번**(배열의 각 항목에 대해) 응답합니다.

#### Event-based

요청-응답 방법은 서비스간에 메시지를 교환하는데 이상적이지만 메시지 스타일이 이벤트 기반인 경우(응답을 기다리지 않고 **이벤트**만 게시하려는 경우) 적합하지 않습니다. 이 경우 두 채널을 유지하기 위해 요청-응답에 필요한 오버헤드를 원하지 않습니다.

시스템의 이 부분에서 특정 조건이 발생했음을 다른 서비스에 간단히 알리고싶다고 가정합니다. 이것은 이벤트 기반 메시지 스타일의 이상적인 사용 사례입니다.

이벤트 핸들러를 생성하기 위해 `@nestjs/microservices` 패키지에서 가져온 `@EventPattern()`v데코레이터를 사용합니다.

```typescript
@@filename()
@EventPattern('user_created')
async handleUserCreated(data: Record<string, unknown>) {
  // business logic
}
@@switch
@EventPattern('user_created')
async handleUserCreated(data) {
  // business logic
}
```

`handleUserCreated()` **이벤트 핸들러**는 `'user_created'` 이벤트를 수신합니다. 이벤트 핸들러는 클라이언트에서 전달된 `data`(이 경우 네트워크를 통해 전송된 이벤트 페이로드)인 단일인수를 받습니다.

<app-banner-enterprise></app-banner-enterprise>

#### Decorators

더 복잡한 시나리오에서는 들어오는 요청에 대한 추가정보에 액세스할 수 있습니다. 예를 들어 와일드카드 구독이 있는 NATS의 경우 생산자(producer)가 메시지를 보낸 원래 제목을 가져올 수 있습니다. 마찬가지로 Kafka에서 메시지 헤더에 액세스할 수 있습니다. 이를 수행하기 위해 다음과 같이 내장 데코레이터를 사용할 수 있습니다.

```typescript
@@filename()
@MessagePattern('time.us.*')
getDate(@Payload() data: number[], @Ctx() context: NatsContext) {
  console.log(`Subject: ${context.getSubject()}`); // e.g. "time.us.east"
  return new Date().toLocaleTimeString(...);
}
@@switch
@Bind(Payload(), Ctx())
@MessagePattern('time.us.*')
getDate(data, context) {
  console.log(`Subject: ${context.getSubject()}`); // e.g. "time.us.east"
  return new Date().toLocaleTimeString(...);
}
```

> info **힌트** `@Payload()`, `@Ctx()` 및 `NatsContext`는 `@nestjs/microservices`에서 가져옵니다.

#### Client

클라이언트 Nest 애플리케이션은 `ClientProxy` 클래스를 사용하여 Nest 마이크로서비스에 메시지를 교환하거나 이벤트를 게시할 수 있습니다. 이 클래스는 원격 마이크로서비스와 통신할 수 있는 `send()` (요청-응답 메시징용) 및 `emit()`(이벤트 기반 메시징용)과 같은 여러 메서드를 정의합니다. 다음 방법중 하나로 이 클래스의 인스턴스를 얻습니다.

한가지 기술은 정적 `register()` 메서드를 노출하는 `ClientsModule`을 가져오는 것입니다. 이 메서드는 마이크로서비스 전송자를 나타내는 객체의 배열인 인수를 사용합니다. 이러한 각 객체에는 `name` 속성, 선택적 `transport` 속성(기본값은 `Transport.TCP`) 및 선택적 전송자별 `options`속성이 있습니다.

`name` 속성은 필요한 곳에 `ClientProxy`의 인스턴스를 삽입하는데 사용할 수 있는 **주입 토큰** 역할을합니다. 인젝션 토큰인 `name` 속성의 값은 [여기](/fundamentals/custom-providers#non-class-based-provider-tokens)에 설명된대로 임의의 문자열 또는 JavaScript 기호 일 수 있습니다.

`options` 속성은 앞서 `createMicroservice()` 메서드에서 본 것과 동일한 속성을 가진 객체입니다.

```typescript
@Module({
  imports: [
    ClientsModule.register([
      { name: 'MATH_SERVICE', transport: Transport.TCP },
    ]),
  ]
  ...
})
```

모듈을 가져온 후에는 `@Inject()` 데코레이터를 사용하여 위에 표시된 `'MATH_SERVICE'` 트랜스포터 옵션을 통해 지정된대로 구성된 `ClientProxy`의 인스턴스를 삽입할 수 있습니다.

```typescript
constructor(
  @Inject('MATH_SERVICE') private client: ClientProxy,
) {}
```

> info **힌트** `ClientsModule` 및 `ClientProxy` 클래스는 `@nestjs/microservices` 패키지에서 가져옵니다.

때때로 우리는 클라이언트 애플리케이션에서 하드코딩하는 대신 다른 서비스(예: `ConfigService`)에서 전송자 구성을 가져와야할 수 있습니다. 이를 위해 `ClientProxyFactory` 클래스를 사용하여 [custom provider](/fundamentals/custom-providers)를 등록할 수 있습니다. 이 클래스에는 트랜스포터 옵션 객체를 받아들이고 사용자 정의된 `ClientProxy` 인스턴스를 반환하는 정적 `create()` 메서드가 있습니다.

```typescript
@Module({
  providers: [
    {
      provide: 'MATH_SERVICE',
      useFactory: (configService: ConfigService) => {
        const mathSvcOptions = configService.getMathSvcOptions();
        return ClientProxyFactory.create(mathSvcOptions);
      },
      inject: [ConfigService],
    }
  ]
  ...
})
```

> info **힌트** `ClientProxyFactory`는 `@nestjs/microservices` 패키지에서 가져옵니다.

또 다른 옵션은 `@Client()` 속성 데코레이터를 사용하는 것입니다.

```typescript
@Client({ transport: Transport.TCP })
client: ClientProxy;
```

> info **힌트** `@Client()` 데코레이터는 `@nestjs/microservices` 패키지에서 가져옵니다.

`@Client()` 데코레이터를 사용하는 것은 테스트하기가 더 어렵고 클라이언트 인스턴스를 공유하기가 더 어렵기 때문에 선호되는 기술이 아닙니다.

`ClientProxy`는 **lazy**입니다. 즉시 연결을 시작하지 않습니다. 대신 첫번째 마이크로서비스 호출전에 설정한 다음 각 후속 호출에서 재사용됩니다. 그러나 연결이 설정될 때까지 애플리케이션 부트스트랩 프로세스를 지연시키려면 `OnApplicationBootstrap` 수명주기 후크내에서 `ClientProxy` 객체의 `connect()` 메서드를 사용하여 수동으로 연결을 시작할 수 있습니다.

```typescript
@@filename()
async onApplicationBootstrap() {
  await this.client.connect();
}
```

연결을 만들 수 없는 경우 `connect()` 메서드는 해당 오류 객체와 함께 거부됩니다.

#### Sending messages

`ClientProxy`는 `send()` 메소드를 노출합니다. 이 메서드는 마이크로서비스를 호출하기 위한 것이며 응답과 함께 `Observable`을 반환합니다. 따라서 방출된 값을 쉽게 구독할 수 있습니다.

```typescript
@@filename()
accumulate(): Observable<number> {
  const pattern = { cmd: 'sum' };
  const payload = [1, 2, 3];
  return this.client.send<number>(pattern, payload);
}
@@switch
accumulate() {
  const pattern = { cmd: 'sum' };
  const payload = [1, 2, 3];
  return this.client.send(pattern, payload);
}
```

`send()` 메소드는 `pattern`과 `payload`라는 두개의 인수를 받습니다. `pattern`은 `@MessagePattern()` 데코레이터에 정의된 것과 일치해야합니다. `payload`는 원격 마이크로서비스로 전송하려는 메시지입니다. 이 메소드는 **cold `Observable`**을 반환합니다. 즉, 메시지를 보내기전에 명시적으로 구독해야합니다.

#### Publishing events

이벤트를 보내려면 `ClientProxy` 객체의 `emit()` 메서드를 사용하세요. 이 메서드는 메시지 브로커에 이벤트를 게시합니다.

```typescript
@@filename()
async publish() {
  this.client.emit<number>('user_created', new UserCreatedEvent());
}
@@switch
async publish() {
  this.client.emit('user_created', new UserCreatedEvent());
}
```

`emit()` 메서드는 `pattern`과 `payload`라는 두개의 인수를 받습니다. `pattern`은 `@EventPattern()` 데코레이터에 정의된 것과 일치해야합니다. `payload`는 원격 마이크로서비스로 전송하려는 이벤트 페이로드입니다. 이 메소드는 **hot `Observable`**을 반환합니다 (`send()`에 의해 반환된 콜드 `Observable`과 달리). 이는 Observable을 명시적으로 구독하는지 여부에 관계없이 프록시가 즉시 이벤트 전달을 시도함을 의미합니다.

<app-banner-shop></app-banner-shop>

#### Scopes

다른 프로그래밍 언어 배경을 가진 사람들의 경우 Nest에서 거의 모든 것이 들어오는 요청에서 공유된다는 사실을 배우는 것은 예상치 못한 일입니다. 데이터베이스에 대한 연결 풀, 전역 상태의 싱글톤 서비스 등이 있습니다. Node.js는 모든 요청이 별도의 스레드에 의해 처리되는 요청/응답 다중 스레드 상태 비 저장 모델을 따르지 않습니다. 따라서 싱글톤 인스턴스를 사용하는 것은 애플리케이션에 완전히 **안전**합니다.

그러나 요청 기반 핸들러의 수명이 바람직한 동작일 수 있는 경우가 있습니다(예: GraphQL 애플리케이션의 요청별 캐싱, 요청 추적 또는 다중 테넌시). [여기](/fundamentals/injection-scopes)에서 범위를 제어하는 방법을 알아보세요.

요청 범위 핸들러 및 제공자는 `CONTEXT` 토큰과 함께 `@Inject()` 데코레이터를 사용하여 `RequestContext`를 삽입할 수 있습니다.

```typescript
import { Injectable, Scope, Inject } from '@nestjs/common';
import { CONTEXT, RequestContext } from '@nestjs/microservices';

@Injectable({ scope: Scope.REQUEST })
export class CatsService {
  constructor(@Inject(CONTEXT) private ctx: RequestContext) {}
}
```

이렇게 하면 두가지 속성이 있는 `RequestContext` 객체에 대한 액세스가 제공됩니다.

```typescript
export interface RequestContext<T = any> {
  pattern: string | Record<string, any>;
  data: T;
}
```

`data` 속성은 메시지 생성자가 보낸 메시지 페이로드입니다. `pattern` 속성은 수신 메시지를 처리하기위한 적절한 핸들러를 식별하는데 사용되는 패턴입니다.

#### Handling timeouts

분산 시스템에서는 때때로 마이크로서비스가 다운되거나 사용 불가능할 수 있습니다. 무한히 오래 기다리지 않으려면 제한시간을 사용할 수 있습니다. 타임아웃은 다른 서비스와 통신할 때 매우 유용한 패턴입니다. 마이크로서비스 호출에 타임아웃을 적용하려면 `RxJS` 타임아웃 연산자를 사용할 수 있습니다. 마이크로서비스가 특정 시간 내에 요청에 응답하지 않으면 예외가 발생하여 적절히 포착하고 처리할 수 있습니다.

이 문제를 해결하려면 [rxjs](https://github.com/ReactiveX/rxjs) 패키지를 사용해야합니다. 파이프에서 `timeout` 연산자를 사용하십시오.

```typescript
@@filename()
this.client
      .send<TResult, TInput>(pattern, data)
      .pipe(timeout(5000))
      .toPromise();
@@switch
this.client
      .send(pattern, data)
      .pipe(timeout(5000))
      .toPromise();
```

> info **힌트** `timeout` 연산자는 `rxjs/operators` 패키지에서 가져옵니다.

5초 후 마이크로서비스가 응답하지 않으면 오류가 발생합니다.
