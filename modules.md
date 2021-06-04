### Modules

모듈은 `@Module()` 데코레이터로 주석이 달린 클래스입니다. `@Module()` 데코레이터는 **Nest**가 애플리케이션 구조를 구성하는데 사용하는 메타데이터를 제공합니다.

<figure><img src="/assets/Modules_1.png" /></figure>

각 애플리케이션에는 **루트 모듈**이라는 하나 이상의 모듈이 있습니다. 루트 모듈은 Nest가 **애플리케이션 그래프**를 빌드하는데 사용하는 시작점입니다. Nest가 모듈과 공급자 관계 및 종속성을 해결하는데 사용하는 내부 데이터 구조입니다. 매우 작은 애플리케이션에는 이론적으로 루트 모듈만 있을 수 있지만 일반적인 경우는 아닙니다. 모듈은 구성요소를 구성하는 효과적인 방법으로 **적극** 권장된다는 점을 강조하고 싶습니다. 따라서 대부분의 애플리케이션에서 결과 아키텍처는 각각 밀접하게 관련된 **기능** 집합을 캡슐화하는 여러 모듈을 사용합니다.

`@Module()` 데코레이터는 속성이 모듈을 설명하는 단일 객체를 사용합니다.

<table>
  <tr>
    <td><code>providers</code></td>
    <td>Nest 인젝터에 의해 인스턴스화 되고 적어도 이 모듈에서 공유될 수 있는 프로바이더</td>
  </tr>
  <tr>
    <td><code>controllers</code></td>
    <td>인스턴스화 되어야 하는 이 모듈에 정의된 컨트롤러 세트</td>
  </tr>
  <tr>
    <td><code>imports</code></td>
    <td>이 모듈에 필요한 프로바이더를 내보내는 가져온 모듈 목록</td>
  </tr>
  <tr>
    <td><code>exports</code></td>
    <td>이 모듈에서 제공하고 이 모듈을 임포트하는 다른 모듈에서 사용할 수 있어야 하는 프로바이더의 하위집합</td>
  </tr>
</table>

모듈은 기본적으로 프로바이더를 **캡슐화**합니다. 즉, 현재 모듈에 직접 포함되거나 가져온(import) 모듈에서 내보내지(export) 않은 프로바이더를 삽입할 수 없습니다. 따라서 모듈에서 내보낸 프로바이더를 모듈의 공용 인터페이스 또는 API로 간주할 수 있습니다.

#### Feature modules

`CatsController`와 `CatsService`는 동일한 애플리케이션 도메인에 속합니다. 밀접하게 관련되어 있으므로 기능 모듈로 이동하는 것이 좋습니다. 기능 모듈은 단순히 특정 기능과 관련된 코드를 구성하여 코드를 체계적으로 유지하고 명확한 경계를 설정합니다. 이는 특히 애플리케이션 및/또는 팀의 규모가 커짐에 따라 복잡성을 관리하고 [SOLID](https://en.wikipedia.org/wiki/SOLID) 원칙에 따라 개발하는데 도움이 됩니다.

이를 증명하기 위해 `CatsModule`을 생성합니다.

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

> info **힌트** CLI를 사용하여 모듈을 생성하려면 `$ nest g module cats` 명령을 실행하면 됩니다.

위에서 우리는 `cats.module.ts` 파일에 `CatsModule`을 정의하고 이 모듈과 관련된 모든 것을 `cats` 디렉토리로 옮겼습니다. 마지막으로 해야할 일은 이 모듈을 루트 모듈(`app.module.ts` 파일에 정의된 `AppModule`)로 가져오는 것입니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule {}
```

다음은 디렉토리 구조가 지금 어떻게 보이는지 입니다.

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
      <div class="item">cats.controller.ts</div>
      <div class="item">cats.module.ts</div>
      <div class="item">cats.service.ts</div>
    </div>
    <div class="item">app.module.ts</div>
    <div class="item">main.ts</div>
  </div>
</div>

#### Shared modules

Nest에서 모듈은 기본적으로 **싱글톤**이므로 여러 모듈간에 쉽게 프로바이더의 동일한 인스턴스를 공유할 수 있습니다.

<figure><img src="/assets/Shared_Module_1.png" /></figure>

모든 모듈은 자동으로 **공유 모듈**입니다. 일단 생성되면 모든 모듈에서 재사용할 수 있습니다. 여러 다른 모듈간에 `CatsService`의 인스턴스를 공유하고 싶다고 가정해 봅시다. 이렇게 하려면 먼저 아래와 같이 모듈의 `exports` 배열에 `CatsService` 프로바이더를 추가하여 **내보내기**해야합니다.

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

이제 `CatsModule`을 가져오는 모든 모듈은 `CatsService`에 액세스할 수 있으며 이를 가져오는 다른 모든 모듈과 동일한 인스턴스를 공유합니다.

<app-banner-enterprise></app-banner-enterprise>

#### Module re-exporting

위에서 볼 수 있듯이 모듈은 내부 프로바이더를 내보낼 수 있습니다. 또한 가져온 모듈을 다시 내보낼 수 있습니다.
아래 예에서 `CommonModule`은 `CoreModule`로 가져**오고** 내보내므로 이 모듈을 가져오는 다른 모듈에서 사용할 수 있습니다.

```typescript
@Module({
  imports: [CommonModule],
  exports: [CommonModule],
})
export class CoreModule {}
```

#### Dependency injection

모듈 클래스는 프로바이더도 **주입**할 수 있습니다 (예: 구성 목적):

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

그러나 모듈 클래스 자체는 [순환 종속성](/fundamentals/circular-dependency)로 인해 프로바이더로 삽입될 수 없습니다.

#### Global modules

모든 곳에서 동일한 모듈 세트를 가져와야 한다면 지루할 수 있습니다. Nest와 달리 [Angular](https://angular.io) `providers`는 전역 범위에 등록됩니다. 일단 정의되면 어디서나 사용할 수 있습니다. 그러나 Nest는 모듈 범위내에서 프로바이더를 캡슐화합니다. 캡슐화 모듈을 먼저 가져오지 않으면 모듈의 프로바이더를 다른 곳에서 사용할 수 없습니다.

즉시 사용할 수 있어야하는 프로바이더 집합(예: 헬퍼, 데이터베이스 연결 등)을 제공하려면 `@Global()` 데코레이터를 사용하여 **전역** 모듈을 만듭니다. .

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

`@Global()` 데코레이터는 모듈을 전역범위로 만듭니다. 전역 모듈은 일반적으로 루트 또는 코어 모듈에서 **한번만** 등록해야 합니다. 위의 예에서 `CatsService` 프로바이더는 어디에나 있을 것이며, 서비스를 주입하려는 모듈은 가져오기 배열에서 `CatsModule`을 가져올 필요가 없습니다.

> info **힌트** 모든 것을 글로벌하게 만드는 것은 좋은 디자인 결정이 아닙니다. 필요한 보일러플레이트의 양을 줄이기 위해 전역 모듈을 사용할 수 있습니다. `imports` 배열은 일반적으로 소비자가 모듈의 API를 사용할 수 있도록 하는 선호되는 방법입니다.

#### Dynamic modules

Nest 모듈 시스템에는 **동적 모듈 Dynamic module**이라는 강력한 기능이 포함되어 있습니다. 이 기능을 사용하면 프로바이더를 동적으로 등록하고 구성할 수 있는 커스텀 가능한 모듈을 쉽게 만들 수 있습니다. 동적 모듈은 [여기](/fundamentals/dynamic-modules)에서 광범위하게 다룹니다. 이 장에서는 모듈 소개를 완료하기 위한 간략한 개요를 제공합니다.

다음은 `DatabaseModule`에 대한 동적 모듈 정의의 예입니다.


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

> info **힌트** `forRoot()` 메소드는 동기식 또는 비동기식으로 (즉, `Promise`를 통해) 동적 모듈을 반환할 수 있습니다.

이 모듈은 기본적으로 `Connection` 프로바이더를 정의하지만 (`@Module()` 데코레이터 메타데이터에서) 추가로 - `forRoot()` 메서드에 전달된 `entities` 및 `options` 객체에 따라 - 저장소와 같은 프로바이더 컬렉션을 추가로 노출합니다. 동적 모듈이 반환하는 속성은 `@Module()` 데코레이터에 정의된 기본 모듈 메타데이터를 재 정의하는 대신 **확장**합니다. 이것이 정적으로 선언된 `Connection` 프로바이더**와** 동적으로 생성된 저장소 프로바이더를 모듈에서 내보내는 방법입니다.

전역범위에 동적 모듈을 등록하려면 `global` 속성을 `true`로 설정합니다.

```typescript
{
  global: true,
  module: DatabaseModule,
  providers: providers,
  exports: providers,
}
```

> warning **경고** 위에서 언급했듯이 모든 것을 글로벌하게 만드는 것은 **좋은 디자인 결정이 아닙니다**.

`DatabaseModule`은 다음과 같은 방식으로 가져오고 구성할 수 있습니다.

```typescript
import { Module } from '@nestjs/common';
import { DatabaseModule } from './database/database.module';
import { User } from './users/entities/user.entity';

@Module({
  imports: [DatabaseModule.forRoot([User])],
})
export class AppModule {}
```

다시 동적 모듈을 다시 내보내려면 exports 배열에서 `forRoot()` 메서드 호출을 생략할 수 있습니다.

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

[동적 모듈](/fundamentals/dynamic-modules)장에서는 이 주제를 자세히 다루며 [작업 예제](https://github.com/nestjs/nest/tree/master/sample/25-dynamic-modules)를 포함합니다.
