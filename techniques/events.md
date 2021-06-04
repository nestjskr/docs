### Events

[이벤트 이미터](https://www.npmjs.com/package/@nestjs/event-emitter) 패키지(`@nestjs/event-emitter`)는 간단한 관찰자(observer) 구현을 제공하여 다양한 이벤트를 구독하고 수신할 수 있습니다. 애플리케이션에서 발생합니다. 단일 이벤트에 서로 의존하지 않는 여러 리스너가 있을 수 있으므로 이벤트는 애플리케이션의 다양한 측면을 분리하는 좋은 방법입니다.

`EventEmitterModule` internally uses the [eventemitter2](https://github.com/EventEmitter2/EventEmitter2) package.

#### Getting started

먼저 필요한 패키지를 설치하십시오.

```shell
$ npm i --save @nestjs/event-emitter
```

설치가 완료되면 `EventEmitterModule`을 루트 `AppModule`로 가져오고 아래와 같이 `forRoot()` 정적 메서드를 실행합니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { EventEmitterModule } from '@nestjs/event-emitter';

@Module({
  imports: [
    EventEmitterModule.forRoot()
  ],
})
export class AppModule {}
```

`.forRoot()` 호출은 이벤트 이미터를 초기화하고 앱 내에 존재하는 선언적 이벤트 리스너를 등록합니다. 등록은 `onApplicationBootstrap` 라이프사이클 후크가 발생할 때 발생하여 모든 모듈이 예약된 작업을 로드하고 선언했는지 확인합니다.

기본 `EventEmitter` 인스턴스를 구성하려면 다음과 같이 구성 객체를 `.forRoot()` 메서드에 전달합니다.

```typescript
EventEmitterModule.forRoot({
  // set this to `true` to use wildcards
  wildcard: false,
  // the delimiter used to segment namespaces
  delimiter: '.',
  // set this to `true` if you want to emit the newListener event
  newListener: false,
  // set this to `true` if you want to emit the removeListener event
  removeListener: false,
  // the maximum amount of listeners that can be assigned to an event
  maxListeners: 10,
  // show event name in memory leak message when more than maximum amount of listeners is assigned
  verboseMemoryLeak: false,
  // disable throwing uncaughtException if an error event is emitted and it has no listeners
  ignoreErrors: false,
});
```

#### Dispatching Events

이벤트를 전달(즉, 실행)하려면 먼저 표준 생성자 주입을 사용하여 `EventEmitter2`를 주입합니다.

```typescript
constructor(private eventEmitter: EventEmitter2) {}
```

> info **힌트** `@nestjs/event-emitter` 패키지에서 `EventEmitter2`를 가져옵니다.

그런 다음 다음과 같이 클래스에서 사용하십시오.

```typescript
this.eventEmitter.emit(
  'order.created',
  new OrderCreatedEvent({
    orderId: 1,
    payload: {},
  }),
);
```

#### Listening to Events

이벤트 리스너를 선언하려면 실행할 코드가 포함된 메서드 정의 앞에 `@OnEvent()` 데코레이터를 사용하여 다음과 같이 메서드를 데코레이션합니다.

```typescript
@OnEvent('order.created')
handleOrderCreatedEvent(payload: OrderCreatedEvent) {
  // handle and process "OrderCreatedEvent" event
}
```

> warning **경고** 이벤트 구독자는 요청 범위를 지정할 수 없습니다.

첫번째 인수는 간단한 이벤트 이미터의 경우 `string` 또는 `symbol`과 `string | symbol | Array<string | symbol>`. 두번째 인수(선택 사항)는 리스너 옵션 객체입니다 ([자세히 알아보기](https://github.com/EventEmitter2/EventEmitter2#emitteronevent-listener-options-objectboolean)).

```typescript
@OnEvent('order.created', { async: true })
handleOrderCreatedEvent(payload: OrderCreatedEvent) {
  // handle and process "OrderCreatedEvent" event
}
```

네임스페이스/와일드카드를 사용하려면 `wildcard` 옵션을 `EventEmitterModule#forRoot()` 메서드에 전달합니다. 네임스페이스/와일드카드가 활성화된 경우 이벤트는 구분기호로 구분된 문자열(`foo.bar`)이거나 배열(`['foo', 'bar']`)일 수 있습니다. 구분기호는 구성 속성(`delimiter`)으로도 구성할 수 있습니다. 네임스페이스 기능을 활성화하면 와일드카드를 사용하여 이벤트를 구독할 수 있습니다.

```typescript
@OnEvent('order.*')
handleOrderEvents(payload: OrderCreatedEvent | OrderRemovedEvent | OrderUpdatedEvent) {
  // handle and process an event
}
```

이러한 와일드카드는 하나의 블록에만 적용됩니다. 예를 들어 `order.*` 인수는 `order.created` 및 `order.shipped` 이벤트와 일치하지만 `order.delayed.out_of_stock`과는 일치하지 않습니다. 이러한 이벤트를 듣기 위해 `EventEmitter2` [문서](https://github.com/EventEmitter2/EventEmitter2#multi-level-wildcards)에 설명된 `multilevel wildcard` 패턴(예: `**`)을 사용합니다.

예를 들어 이 패턴을 사용하면 모든 이벤트를 포착하는 이벤트 리스너를 만들 수 있습니다.

```typescript
@OnEvent('**')
handleEverything(payload: any) {
  // handle and process an event
}
```

> info **힌트** `EventEmitter2` 클래스는 `waitFor` 및 `onAny`와 같은 이벤트와 상호 작용하는데 유용한 여러 메소드를 제공합니다. 자세한 내용은 [여기](https://github.com/EventEmitter2/EventEmitter2)에서 확인할 수 있습니다.

#### Example

작동하는 예는 [여기](https://github.com/nestjs/nest/tree/master/sample/30-event-emitter)에서 확인할 수 있습니다.
