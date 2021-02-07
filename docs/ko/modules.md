### 모듈

모듈은 `@Module()` 데코레이터로 어노테이션한 클래스를 말합니다. `@Module()` 데코레이터는 **Nest**가 어플리케이션 구조를 구성하는 데 사용하는 메타 데이터를 제공합니다.

<figure><img src="/assets/Modules_1.png"></figure>

각 어플리케이션은 최소한 하나 이상의 모듈, 즉 **루트 모듈(Root Module)**을 가지고 있습니다. 루트 모듈(Root Module)은 Nest가 **어플리케이션의 구조 그래프**(Nest의 모듈과 공급자(Provider)의 관계 및 의존성 해결을 위한 내부 데이터 구조)를 구성하는데 사용하는 진입점(시작점)입니다. 이론적으로 소규모의 어플리케이션은 Root 모듈만 가질 수 있지만, 실제 서비스는 크기에 Root 모듈만 가지기는 어렵습니다. 우리는 컴포넌트(Component)를 구성하기 위한 효과적인 방법으로 모듈을 **매우 권장합니다**. 이를 통해, 대부분 어플리케이션은 여러 모듈을 사용하며, 밀접하게 관련된 **기능**을 캡슐화하는 구조를 만들 수 있습니다.

`@Module()` 데코레이터는 속성을 통해 모듈에 대해 설명하는 하나의 객체를 사용합니다:

<table>
  <tr>
    <td><code>providers</code></td>
    <td>Nest의 인젝터(Injector)에 의해 인스턴스화 되며, 해당 모듈에서 공유되는 공급자(Provider)를 명시한다.</td>
  </tr>
  <tr>
    <td><code>controllers</code></td>
    <td>인스턴스화 해야 하는 해당 모듈에 정의된 컨트롤러의 집합</td>
  </tr>
  <tr>
    <td><code>imports</code></td>
    <td>해당 모듈에서 필요한 공급자(Provider)를 내보내는(Export) 모듈을 가져온(Import) 리스트. <br>(즉, 특정 모듈의 <code>export</code> 부분을 쓰기 위해서 <code>import</code>를 해줘야 한다.)</td>
  </tr>
  <tr>
    <td><code>exports</code></td>
    <td>해당 모듈을 가져간(<code>import</code>) 모듈이 사용할 수 있는 Provider의 하위 집합<br>(즉, 특정 모듈을 <code>import</code> 해서 Provider를 쓰기 위해서는 <code>exports</code> 부분에 명시해줘야 한다.)</td>
  </tr>
</table>

모듈은 기본적으로 공급자(Provider)를 **캡슐화(encapsulate)**합니다. 이는 현재 모듈에서 직접적으로 포함하거나 가져온(import) 모듈에서 내보낸(export) 공급자(Provider) 이외의 공급자(Provider)는 의존성 주입(Inject)을 할 수 없다는 것을 의미합니다. 따라서, 모듈에서 내보낸(export) 공급자(Provider)를 모듈의 공용 인터페이스나 API로 간주할 수 있습니다.

#### 기능 모듈(Feature modules)

`CatsController` 과 `CatsService`은 같은 어플리케이션의 도메인에 속합니다. 이는 서로 밀접하게 연관되어 있으므로 기능 모듈(Feature module)로 이동하는 것이 바람직합니다. 기능 모듈(Feature module)은 특정 기능과 관련된 코드를 정리하고 명확한 경계를 만듭니다. 이를 통해 복잡성 관리를 도우며 어플리케이션과 팀이 커지더라도 [SOLID](https://en.wikipedia.org/wiki/SOLID) 원칙에 따라 개발할 수 있도록 해줍니다.

이를 보여주기 위해 `CatsModule`을 구현해보겠습니다.

```typescript
@@filename(cats/cats.module)
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```

> 정보 **힌트** CLI를 통해 모듈을 생성하고 싶으면,`$ nest g module cats` 명령어를 실행하세요.

위에서, 우리는 `cats.module.ts`파일에서 `CatsModule`을 정의했으며, 이 모듈과 관련된 모든 것을  `cats` 디렉토리로 옮겼습니다. 끝으로 우리는 이 모듈을 루트 모듈에서 가져가(import) 사용할 수 있습니다. (`AppModule`은 `app.module.ts` 파일 안에서 정의되어 있습니다).

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule {}
```

이제 우리 디렉토리 구조는 다음과 같습니다:

<div class="file-tree">
  <div class="item">src</div>
  <div class="children">
    <div class="item">cats</div>
    <div class="children">
      <div class="item">dto</div>
      <div class="children">
        <div class="item">create-cat.dto.ts</div>
      </div>
      <div class="item">interfaces</div>
      <div class="children">
        <div class="item">cat.interface.ts</div>
      </div>
      <div class="item">cats.service.ts</div>
      <div class="item">cats.controller.ts</div>
      <div class="item">cats.module.ts</div>
    </div>
    <div class="item">app.module.ts</div>
    <div class="item">main.ts</div>
  </div>
</div>

#### 공유 모듈(Shared modules)

Nest에서는 모듈은 기본적으로 **싱글톤(singletons)**을 따릅니다. 따라서 별다른 노력 없이 여러 모듈에 존재하는 공급자(Provider)에서 같은 인스턴스(instance)를 공유합니다.

<figure><img src="/assets/Shared_Module_1.png"></figure>

모든 모듈은 자동적으로 **공유 모듈(shared module)** 입니다. 한번 생성되면 그 모듈은 어느 모듈에서도 다시 사용될 수 있습니다. `CatsService`의 인스턴스를 몇몇 다른 모듈과 공유해야 한다고 가정해봅시다. 이를 위해서, 우리는 첫번째로 `CatsService` 공급자(Provider)를 아래 그림과 같이 모듈의 `export` 배열에 추가하여 **내보내야(export)** 합니다.

```typescript
@@filename(cats.module)
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService]
})
export class CatsModule {}
```

이제 `CatsModule`을 가져오는(import) 어떤 모듈도 `CatsService`에 접근할 수 있으며 이를 가져오는(import) 다른 모듈과 같은 인스턴스를 공유할 것 입니다.

<app-banner-enterprise></app-banner-enterprise>

#### 모듈 다시 내보내기(Module re-exporting)

위처럼, 모듈은 자신의 내부 공급자(Provider)를 내보낼 수 있습니다. 게다가, 그들은 자신이 가져온 모듈을 다시 내보낼 수 있습니다. 아래 예제를 보면, `CoreModule`에서 <code>CommonModule</code>을 가져옴과 동시에 내보낸 것을 확인할 수 있습니다.  이를 통해 `CoreModule`를 가져온(import) 모듈에서도 <code>CommonModule</code>을 사용할 수 있게 됩니다.

```typescript
@Module({
  imports: [CommonModule],
  exports: [CommonModule],
})
export class CoreModule {}
```

#### 의존성 주입(Dependency injection)

모듈 클래스는 공급자(Provder)도 **주입할 수** 있습니다(예를 들어 configuration의 목적으로 사용할 수 있습니다):

```typescript
@@filename(cats.module)
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {
  constructor(private catsService: CatsService) {}
}
@@switch
import { Module, Dependencies } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
@Dependencies(CatsService)
export class CatsModule {
  constructor(catsService) {
    this.catsService = catsService;
  }
}
```

하지만, [순환 종속성(circular dependency)](/fundamentals/circular-dependency)으로 인해 모듈 클래스 자체를 공급자로 주입할 수 없습니다.

#### 전역 모듈(Global modules)

만약, 같은 모듈을 어디에서나 가져와야 한다면 다소 귀찮을 수 있습니다. Nest와 달리 [Angular](https://angular.io)의 `공급자(providers)` 전역 스코프에서 등록됩니다. 즉, 한번 정의되면 어디에서든 사용할 수 있습니다. 그러나 Nest에서는 모듈 스코프안에서 공급자(Provider)를 정의합니다. 우선적으로 캡슐화 된 모듈을 가져오지 않고는 다른 곳에서는 모듈의 공급자(Provider)를 사용할 수 없습니다.

어디서나 사용할 수 있는 공급자(Provider)를 만들기 위해서는(예를 들어 helpers, database connections 등등) 모듈을 `@Global()` 데코레이터를 통해 **전역적으로(global)** 만들어야 합니다.

```typescript
import { Module, Global } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Global()
@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService],
})
export class CatsModule {}
```

`@Global()` 데코레이터는 모듈을 전역 스코프로 만듭니다. 전역 모듈(Global modules)은 일반적으로 루트 모듈이나 코어 모듈에서 **오직 한번** 등록됩니다. 위의 예제 같이, `CatsService` 공급자(Provider)는 어디에서든 사용할 수 있게 되었으며 서비스를 주입하고 싶은 다른 모듈에서 imports 배열에 `CatsModule`을 주입할 필요가 없어졌습니다.

> 정보 **힌트** 모든 것을 전역(Global)로 만드는 것은 어플리케이션 디자인 상 좋은 결정이 아닙니다. 전역 모듈(Global)을 통해 필요한 보일러플레이트의 양을 줄일 수 있습니다. `imports` 배열은 가져온 모듈의 API를 해당 모듈에서 사용하는 방식을 더 선호합니다.

#### 동적 모듈(Dynamic modules)

Nest의 모듈 시스템은 **동적 모듈(Dynamic modules)**이라는 강력한 기능을 가지고 있습니다. 이 기능을 사용하면, 공급자(Provider)를 동적으로 등록 및 구성할 수 있는 커스터마이징이 가능한 모듈을 생성할 수 있습니다. 동적 모듈(Dynamic modules) [여기](/fundamentals/dynamic-modules)에서 광범위 하게 다루어집니다. 이번 장에서는, 모듈에 대한 설명을 끝내기 위한 간략한 개요를 다룰 것 입니다.

다음은 `DatabaseModule`을 위한 동적 모듈(Dynamic Module) 정의에 대한 예제입니다:

```typescript
@@filename()
import { Module, DynamicModule } from '@nestjs/common';
import { createDatabaseProviders } from './database.providers';
import { Connection } from './connection.provider';

@Module({
  providers: [Connection],
})
export class DatabaseModule {
  static forRoot(entities = [], options?): DynamicModule {
    const providers = createDatabaseProviders(options, entities);
    return {
      module: DatabaseModule,
      providers: providers,
      exports: providers,
    };
  }
}
@@switch
import { Module } from '@nestjs/common';
import { createDatabaseProviders } from './database.providers';
import { Connection } from './connection.provider';

@Module({
  providers: [Connection],
})
export class DatabaseModule {
  static forRoot(entities = [], options?) {
    const providers = createDatabaseProviders(options, entities);
    return {
      module: DatabaseModule,
      providers: providers,
      exports: providers,
    };
  }
}
```

> 정보 **힌트** `forRoot()` 메소드는 동기 또는 비동기식으로 동적 모듈(dynamic module)을 반환할 수 있습니다 (즉,`Promise`를 통해 수행합니다).

이 모듈은 기본적으로 `Connection` 공급자(Provider)를 정의합니다(`@Module()` 데코레이터의 메타데이터에서 확인할 수 있습니다). 게다가 레포지토리와 같은 공급자(Provider)의 컬렉션을 노출합니다(`forRoot()`메소드로 부터 전달받은 `entities` 와 `options` 객체에 의존합니다). 동적 모듈(Dynamic module)에서 반환되는 속성은  `@Module()` 데코레이터를 통해 정의된 모듈의 메타 데이터를 오버라이딩 하기 보다는 **확장**합니다. 이를 통해 정적으로 선언된 `Connection` 공급자(Provider)와 동적으로 생성된 레포지토리 공급자(providers) 를 모두 모듈에서 내보낼 수 있습니다.

만약 동적 모듈(Dynamic module)을 전역 스코프로 등록하고 싶다면, `global` 속성을 `true`로 할당하면 됩니다.

```typescript
{
  global: true,
  module: DatabaseModule,
  providers: providers,
  exports: providers,
}
```

> 주의 **주의** 위에서 업급한 것 처럼, 모든 것을 전역 모듈로 만드는 것은 **어플리케이션 디자인 상 좋은 결정이 아닙니다**.

`DatabaseModule`은 다음과 같은 방법으로 가져오고 구성될 수 있습니다:

```typescript
import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { User } from './users/entities/user.entity';

@Module({
  imports: [DatabaseModule.forRoot([User])],
})
export class AppModule {}
```

만약 동적 모듈(Dynamic module)을 다시 내보내고 싶으면, exports 배열에서 `forRoot()` 메소드 호출을 생략할 수 있습니다:

```typescript
import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { User } from './users/entities/user.entity';

@Module({
  imports: [DatabaseModule.forRoot([User])],
  exports: [DatabaseModule],
})
export class AppModule {}
```

[동적 모듈](/fundamentals/dynamic-modules) 장 에서 이 주제를 [실습 예제](https://github.com/nestjs/nest/tree/master/sample/25-dynamic-modules)와 함께 더 자세하게 다룹니다.
