### Providers

프로바이더는 Nest의 기본개념입니다. 서비스, 리포지토리, 팩토리, 헬퍼등 기본 Nest 클래스의 대부분은 프로바이더로 취급될 수 있습니다. 프로바이더의 주요 아이디어는 종속성을 **주입**할 수 있다는 것입니다. 즉, 객체는 서로 다양한 관계를 만들 수 있으며 객체의 인스턴스를 "연결"하는 기능은 대부분 Nest 런타임 시스템에 위임될 수 있습니다.

<figure><img src="/assets/Components_1.png" /></figure>

이전 장에서 우리는 간단한 `CatsController`를 만들었습니다. 컨트롤러는 HTTP 요청을 처리하고 더 복잡한 작업을 **프로바이더**에게 위임해야 합니다. 공급자는 [모듈](/module)에서 `provider`로 선언된 일반 자바스크립트 클래스입니다.

> info **힌트** Nest를 사용하면 보다 객체지향적인 방식(OO-way)으로 종속성을 설계하고 구성할 수 있으므로 [SOLID](https://en.wikipedia.org/wiki/SOLID) 원칙을 따르는 것이 좋습니다.

#### Services

간단한 `CatsService`를 만들어 보겠습니다. 이 서비스는 데이터 저장 및 검색을 담당하고 `CatsController`에서 사용하도록 설계되었으므로 프로바이더로 정의하기에 좋은 후보입니다.

```typescript
@@filename(cats.service)
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';

@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];

  create(cat: Cat) {
    this.cats.push(cat);
  }

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

  create(cat) {
    this.cats.push(cat);
  }

  findAll() {
    return this.cats;
  }
}
```

> info **힌트** CLI를 사용하여 서비스를 생성하려면 `$ nest g service cats` 명령을 실행하면 됩니다.

우리의 `CatsService`는 하나의 속성과 두개의 메서드가 있는 기본 클래스입니다. 유일한 새로운 기능은 `@Injectable()` 데코레이터를 사용한다는 것입니다. `@Injectable()` 데코레이터는 `CatsService`가 Nest IoC 컨테이너에서 관리할 수 있는 클래스임을 선언하는 메타데이터를 첨부합니다. 그건 그렇고, 이 예제는 아마도 다음과 같은 `Cat` 인터페이스를 사용합니다 :

```typescript
@@filename(interfaces/cat.interface)
export interface Cat {
  name: string;
  age: number;
  breed: string;
}
```

이제 고양이(cat)를 검색하는 서비스 클래스가 있으므로 `CatsController` 내에서 사용하겠습니다.

```typescript
@@filename(cats.controller)
import { Controller, Get, Post, Body } from '@nestjs/common';
import { CreateCatDto } from './dto/create-cat.dto';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Post()
  async create(@Body() createCatDto: CreateCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
@@switch
import { Controller, Get, Post, Body, Bind, Dependencies } from '@nestjs/common';
import { CatsService } from './cats.service';

@Controller('cats')
@Dependencies(CatsService)
export class CatsController {
  constructor(catsService) {
    this.catsService = catsService;
  }

  @Post()
  @Bind(Body())
  async create(createCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll() {
    return this.catsService.findAll();
  }
}
```

`CatsService`는 클래스 생성자(constructor)를 통해 **주입**됩니다. `private` 구문의 사용에 주목하십시오. 이 약식(shorthand)을 사용하면 동일한 위치에서 즉시 `catsService` 멤버를 선언하고 초기화할 수 있습니다.

#### Dependency injection

Nest는 일반적으로 **종속성 주입**으로 알려진 강력한 디자인 패턴을 기반으로 구축되었습니다. 공식 [Angular](https://angular.io/guide/dependency-injection) 문서에서 이 개념에 대한 훌륭한 기사를 읽는 것이 좋습니다.

Nest에서는 TypeScript 기능 덕분에 종속성이 유형별로 해결되기 때문에 매우 쉽게 관리할 수 있습니다. 아래 예에서 Nest는 `CatsService`의 인스턴스를 만들고 반환하여 `catsService`를 해결합니다(또는 일반적인 경우 싱글톤의 경우 기존 인스턴스가 이미 다른 곳에서 요청된 경우 반환). 이 종속성은 해결되어 컨트롤러의 생성자(또는 표시된 속성에 할당됨)로 전달됩니다.

```typescript
constructor(private catsService: CatsService) {}
```

#### Scopes

프로바이더는 일반적으로 애플리케이션 수명주기와 동기화된 수명("범위(scope)")을 갖습니다. 애플리케이션이 부트스트랩되면 모든 종속성을 해결해야하므로 모든 프로바이더를 인스턴스화해야 합니다. 마찬가지로 애플리케이션이 종료되면 각 프로바이더가 삭제됩니다. 그러나 프로바이더 수명을 **요청 범위**로 만드는 방법도 있습니다. 이러한 기술에 대한 자세한 내용은 [여기](/fundamentals/injection-scopes)를 참조하세요.

<app-banner-courses></app-banner-courses>

#### Custom providers

Nest에는 프로바이더간의 관계를 해결하는 내장된 제어반전("IoC") 컨테이너가 있습니다. 이 기능은 위에서 설명한 종속성 주입기능의 기본이되지만 실제로 지금까지 설명한 것보다 훨씬 강력합니다. 프로바이더를 정의하는 방법에는 여러가지가 있습니다. 일반 값, 클래스 및 비동기 또는 동기 팩토리를 사용할 수 있습니다. 더 많은 예가 [여기](/fundamentals/dependency-injection)에 제공됩니다.

#### Optional providers

경우에 따라 반드시 해결하지 않아도 되는 종속성이 있을 수 있습니다. 예를 들어, 클래스는 **구성 객체**에 종속될 수 있지만 전달되는 것이 없으면 기본값을 사용해야 합니다. 이러한 경우 구성 프로바이더가 없어도 오류가 발생하지 않으므로 종속성이 선택사항이 됩니다.

프로바이더가 선택사항임을 나타내려면 생성자의 서명에 `@Optional()` 데코레이터를 사용하십시오.

```typescript
import { Injectable, Optional, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  constructor(@Optional() @Inject('HTTP_OPTIONS') private httpClient: T) {}
}
```

위의 예에서는 커스텀 프로바이더를 사용하고 있으므로 `HTTP_OPTIONS` 커스텀 **토큰**을 포함합니다. 이전 예제에서는 생성자의 클래스를 통해 종속성을 나타내는 생성자 기반 주입을 보여주었습니다. 커스텀 프로바이더 및 관련 토큰에 대한 자세한 내용은 [여기](/fundamentals/custom-providers)를 참조하세요.

#### Property-based injection

지금까지 사용한 기술을 생성자 기반 주입이라고 합니다. 프로바이더는 생성자 메서드를 통해 주입되기 때문입니다. 매우 특정한 경우 **속성 기반 주입**이 유용할 수 있습니다. 예를 들어 최상위 클래스가 하나 또는 여러 프로바이더에 의존하는 경우 생성자의 하위 클래스에서 `super()`를 호출하여 모든 프로바이더를 전달하는 것은 매우 지루할 수 있습니다. 이를 방지하기 위해 속성 수준에서 `@Inject()` 데코레이터를 사용할 수 있습니다.

```typescript
import { Injectable, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  @Inject('HTTP_OPTIONS')
  private readonly httpClient: T;
}
```

> warning **경고** 클래스가 다른 프로바이더를 확장하지 않는 경우 항상 **생성자 기반** 주입 사용을 선호해야 합니다.

#### Provider registration

이제 프로바이더(`CatsService`)를 정의하고 해당 서비스의 소비자(`CatsController`)를 확보했으므로 주입을 수행할 수 있도록 서비스를 Nest에 등록해야 합니다. 모듈 파일(`app.module.ts`)을 편집하고 `@Module()` 데코레이터의 `providers` 배열에 서비스를 추가하여 이를 수행합니다.

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

Nest는 이제 `CatsController` 클래스의 종속성을 해결할 수 있습니다.

이것이 우리의 디렉토리 구조가 이제 어떻게 보이는지입니다 :

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
<div class="item">cats.service.ts</div>
</div>
<div class="item">app.module.ts</div>
<div class="item">main.ts</div>
</div>
</div>

#### Manual instantiation

 지금까지 Nest가 종속성 해결에 대한 대부분의 세부사항을 자동으로 처리하는 방법에 대해 논의했습니다. 특정 상황에서는 내장된 종속성 주입 시스템을 벗어나 프로바이더를 수동으로 검색하거나 인스턴스화해야할 수 있습니다. 아래에서 이러한 두 가지 주제에 대해 간략하게 설명합니다.

  기존 인스턴스를 가져오거나 프로바이더를 동적으로 인스턴스화하려면 [모듈 참조](/fundamentals/module-ref)를 사용할 수 있습니다.

  `bootstrap()` 함수내에서 프로바이더를 가져오려면(예: 컨트롤러가 없는 독립형 애플리케이션 또는 부트스트랩중에 구성 서비스를 활용하려면) [독립형 애플리케이션](/standalone-applications)을 참조하세요.
