### CQRS

간단한 [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete)(만들기, 읽기, 업데이트 및 삭제) 애플리케이션의 흐름은 다음 단계를 사용하여 설명 할 수 있습니다.

1. **컨트롤러** 계층은 HTTP 요청을 처리하고 작업을 서비스 계층에 위임합니다.
2. **서비스 계층**은 대부분의 비즈니스 로직이 있는 곳입니다.
3. 서비스는 **저장소 / DAO**를 사용하여 엔티티를 변경/유지합니다.
4. 엔티티는 setter 및 getter와 함께 값에 대한 컨테이너 역할을 합니다.

대부분의 경우 중소 규모 애플리케이션의 경우이 패턴으로 충분합니다. 그러나 요구사항이 더 복잡해지면 **CQRS** 모델이 더 적절하고 확장 가능할 수 있습니다. 이 모델을 용이하게하기 위해 Nest는 가벼운 [CQRS 모듈](https://github.com/nestjs/cqrs)을 제공합니다. 이 장에서는 사용 방법에 대해 설명합니다.

#### Installation

먼저 필요한 패키지를 설치하십시오.

```bash
$ npm install --save @nestjs/cqrs
```

#### Commands

이 모델에서는 각 작업을 **명령**이라고 합니다. 명령이 전달되면 애플리케이션이 이에 반응합니다. 명령은 서비스 계층에서 발송하거나 컨트롤러/게이트웨이에서 직접 발송할 수 있습니다. 명령은 **명령 핸들러 Command Handlers**에서 사용됩니다.

```typescript
@@filename(heroes-game.service)
@Injectable()
export class HeroesGameService {
  constructor(private commandBus: CommandBus) {}

  async killDragon(heroId: string, killDragonDto: KillDragonDto) {
    return this.commandBus.execute(
      new KillDragonCommand(heroId, killDragonDto.dragonId)
    );
  }
}
@@switch
@Injectable()
@Dependencies(CommandBus)
export class HeroesGameService {
  constructor(commandBus) {
    this.commandBus = commandBus;
  }

  async killDragon(heroId, killDragonDto) {
    return this.commandBus.execute(
      new KillDragonCommand(heroId, killDragonDto.dragonId)
    );
  }
}
```

다음은 `KillDragonCommand`를 전달하는 샘플 서비스입니다. 명령이 어떻게 보이는지 보겠습니다.

```typescript
@@filename(kill-dragon.command)
export class KillDragonCommand {
  constructor(
    public readonly heroId: string,
    public readonly dragonId: string,
  ) {}
}
@@switch
export class KillDragonCommand {
  constructor(heroId, dragonId) {
    this.heroId = heroId;
    this.dragonId = dragonId;
  }
}
```

`CommandBus`는 명령의 **스트림**입니다. 명령을 동등한 핸들러에 위임합니다. 각 명령에는 해당 **명령 핸들러**가 있어야합니다.

```typescript
@@filename(kill-dragon.handler)
@CommandHandler(KillDragonCommand)
export class KillDragonHandler implements ICommandHandler<KillDragonCommand> {
  constructor(private repository: HeroRepository) {}

  async execute(command: KillDragonCommand) {
    const { heroId, dragonId } = command;
    const hero = this.repository.findOneById(+heroId);

    hero.killEnemy(dragonId);
    await this.repository.persist(hero);
  }
}
@@switch
@CommandHandler(KillDragonCommand)
@Dependencies(HeroRepository)
export class KillDragonHandler {
  constructor(repository) {
    this.repository = repository;
  }

  async execute(command) {
    const { heroId, dragonId } = command;
    const hero = this.repository.findOneById(+heroId);

    hero.killEnemy(dragonId);
    await this.repository.persist(hero);
  }
}
```

이 접근 방식을 사용하면 모든 애플리케이션 상태 변경이 **명령 Command** 발생에 의해 주도됩니다. 로직은 핸들러에 캡슐화됩니다. 이 접근 방식을 사용하면 로깅 또는 데이터베이스에 명령 유지(예: 진단 목적)와 같은 동작을 간단히 추가할 수 있습니다.

#### Events

명령 핸들러는 논리를 깔끔하게 캡슐화합니다. 유용하지만 애플리케이션 구조는 **반응성 reactive**이 아니라 여전히 충분히 유연하지 않습니다. 이를 해결하기 위해 **이벤트**도 소개합니다.

```typescript
@@filename(hero-killed-dragon.event)
export class HeroKilledDragonEvent {
  constructor(
    public readonly heroId: string,
    public readonly dragonId: string,
  ) {}
}
@@switch
export class HeroKilledDragonEvent {
  constructor(heroId, dragonId) {
    this.heroId = heroId;
    this.dragonId = dragonId;
  }
}
```

이벤트는 비동기적입니다. **모델**에 의해 또는 `EventBus`를 사용하여 직접 전달됩니다. 이벤트를 전달하려면 모델이 `AggregateRoot` 클래스를 확장해야합니다.

```typescript
@@filename(hero.model)
export class Hero extends AggregateRoot {
  constructor(private id: string) {
    super();
  }

  killEnemy(enemyId: string) {
    // logic
    this.apply(new HeroKilledDragonEvent(this.id, enemyId));
  }
}
@@switch
export class Hero extends AggregateRoot {
  constructor(id) {
    super();
    this.id = id;
  }

  killEnemy(enemyId) {
    // logic
    this.apply(new HeroKilledDragonEvent(this.id, enemyId));
  }
}
```

`apply()` 메소드는 모델과 `EventPublisher` 클래스간에 관계가 없기 때문에 아직 이벤트를 전달하지 않습니다. 모델과 게시자를 어떻게 연결합니까? 명령 핸들러 내에서 게시자 `mergeObjectContext ()` 메서드를 사용합니다.

```typescript
@@filename(kill-dragon.handler)
@CommandHandler(KillDragonCommand)
export class KillDragonHandler implements ICommandHandler<KillDragonCommand> {
  constructor(
    private repository: HeroRepository,
    private publisher: EventPublisher,
  ) {}

  async execute(command: KillDragonCommand) {
    const { heroId, dragonId } = command;
    const hero = this.publisher.mergeObjectContext(
      await this.repository.findOneById(+heroId),
    );
    hero.killEnemy(dragonId);
    hero.commit();
  }
}
@@switch
@CommandHandler(KillDragonCommand)
@Dependencies(HeroRepository, EventPublisher)
export class KillDragonHandler {
  constructor(repository, publisher) {
    this.repository = repository;
    this.publisher = publisher;
  }

  async execute(command) {
    const { heroId, dragonId } = command;
    const hero = this.publisher.mergeObjectContext(
      await this.repository.findOneById(+heroId),
    );
    hero.killEnemy(dragonId);
    hero.commit();
  }
}
```

이제 모든 것이 예상대로 작동합니다. 이벤트는 즉시 전달되지 않으므로 `commit()` 이벤트가 필요합니다. 분명히, 객체가 앞에 존재할 필요는 없습니다. 타입 컨텍스트도 쉽게 병합할 수 있습니다.

```typescript
const HeroModel = this.publisher.mergeClassContext(Hero);
new HeroModel('id');
```

이제 모델에 이벤트를 게시하는 기능이 있습니다. 또한 `EventBus`를 사용하여 수동으로 이벤트를 내보낼 수 있습니다.

```typescript
this.eventBus.publish(new HeroKilledDragonEvent());
```

> info **힌트** `EventBus`는 주입 가능한 클래스입니다.

각 이벤트에는 여러 **이벤트 핸들러**가 있을 수 있습니다.

```typescript
@@filename(hero-killed-dragon.handler)
@EventsHandler(HeroKilledDragonEvent)
export class HeroKilledDragonHandler implements IEventHandler<HeroKilledDragonEvent> {
  constructor(private repository: HeroRepository) {}

  handle(event: HeroKilledDragonEvent) {
    // logic
  }
}
```

이제 **쓰기 로직**을 이벤트 핸들러로 이동할 수 있습니다.

#### Sagas

이러한 유형의 **이벤트 기반 아키텍처**는 애플리케이션 **반응성 및 확장성**을 향상시킵니다. 이제 이벤트가 있을 때 다양한 방식으로 간단히 반응할 수 있습니다. **Sagas**는 건축적 관점에서 볼 때 마지막 빌딩 블록입니다.

Sagas는 매우 강력한 기능입니다. 단일 사가는 1..\* 이벤트를 수신 할 수 있습니다. [RxJS](https://github.com/ReactiveX/rxjs) 라이브러리를 사용하여 이벤트 스트림에서 다른 `RxJS` 연산자를 결합, 병합, 필터링 또는 적용할 수 있습니다. 각 saga는 명령을 포함하는 Observable을 반환합니다. 이 명령어는 **비동기적으로** 전달됩니다.

```typescript
@@filename(heroes-game.saga)
@Injectable()
export class HeroesGameSagas {
  @Saga()
  dragonKilled = (events$: Observable<any>): Observable<ICommand> => {
    return events$.pipe(
      ofType(HeroKilledDragonEvent),
      map((event) => new DropAncientItemCommand(event.heroId, fakeItemID)),
    );
  }
}
@@switch
@Injectable()
export class HeroesGameSagas {
  @Saga()
  dragonKilled = (events$) => {
    return events$.pipe(
      ofType(HeroKilledDragonEvent),
      map((event) => new DropAncientItemCommand(event.heroId, fakeItemID)),
    );
  }
}
```

> info **힌트** `ofType` 연산자는 `@nestjs/cqrs` 패키지에서 내보내집니다.

우리는 규칙을 선언했습니다. 영웅이 드래곤을 죽이면 고대 아이템을 드롭해야 합니다. 이것이 제자리에 있으면 `DropAncientItemCommand`가 적절한 핸들러에 의해 전달되고 처리됩니다.

#### Queries

`CqrsModule`은 쿼리 처리에도 사용할 수 있습니다. `QueryBus`는 `CommandsBus`와 동일한 패턴을 따릅니다. 쿼리 핸들러는 `IQueryHandler` 인터페이스를 구현하고 `@QueryHandler()` 데코레이터로 표시되어야합니다.

#### Setup

마지막으로 전체 CQRS 메커니즘을 설정하는 방법을 살펴 보겠습니다.

```typescript
@@filename(heroes-game.module)
export const CommandHandlers = [KillDragonHandler, DropAncientItemHandler];
export const EventHandlers =  [HeroKilledDragonHandler, HeroFoundItemHandler];

@Module({
  imports: [CqrsModule],
  controllers: [HeroesGameController],
  providers: [
    HeroesGameService,
    HeroesGameSagas,
    ...CommandHandlers,
    ...EventHandlers,
    HeroRepository,
  ]
})
export class HeroesGameModule {}
```

#### Summary

`CommandBus`, `QueryBus` 및 `EventBus`는 **Observables**입니다. 즉, 전체 스트림을 쉽게 구독하고 **이벤트 소싱**으로 애플리케이션을 풍부하게 만들 수 있습니다.

#### Example

작동하는 예는 [여기](https://github.com/kamilmysliwiec/nest-cqrs-example)에서 확인할 수 있습니다.
