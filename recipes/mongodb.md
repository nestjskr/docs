### MongoDB (Mongoose)

> **경고** 이 문서에서는 사용자 지정 구성요소를 사용하여 처음부터 **Mongoose** 패키지를 기반으로 `DatabaseModule`을 만드는 방법을 알아봅니다. 결과적으로 이 솔루션에는 바로 사용할 수 있고 즉시 사용 가능한 전용 `@nestjs/mongoose` 패키지를 사용하여 생략 할 수 있는 많은 오버헤드가 포함되어 있습니다. 자세한 내용은 [여기](/techniques/mongodb)를 참조하세요.

[Mongoose](https://mongoosejs.com)는 가장 인기있는 [MongoDB](https://www.mongodb.org/) 객체 모델링 도구입니다.

#### Getting started

이 라이브러리로 모험을 시작하려면 필요한 모든 종속성을 설치해야합니다.

```typescript
$ npm install --save mongoose
```

우리가 해야할 첫번째 단계는 `connect()` 함수를 사용하여 데이터베이스와의 연결을 설정하는 것입니다. `connect()` 함수는 `Promise`를 반환하므로 [async provider](/fundamentals/async-components)를 만들어야합니다.

```typescript
@@filename(database.providers)
import * as mongoose from 'mongoose';

export const databaseProviders = [
  {
    provide: 'DATABASE_CONNECTION',
    useFactory: (): Promise<typeof mongoose> =>
      mongoose.connect('mongodb://localhost/nest'),
  },
];
@@switch
import * as mongoose from 'mongoose';

export const databaseProviders = [
  {
    provide: 'DATABASE_CONNECTION',
    useFactory: () => mongoose.connect('mongodb://localhost/nest'),
  },
];
```

> info **힌트** 모범 사례에 따라 `*.providers.ts` 접미사가 있는 별도의 파일에서 사용자 지정 공급자를 선언했습니다.

그런 다음 애플리케이션의 나머지 부분에서 **액세스 가능**하도록 이러한 프로바이더를 내보내야합니다.

```typescript
@@filename(database.module)
import { Module } from '@nestjs/common';
import { databaseProviders } from './database.providers';

@Module({
  providers: [...databaseProviders],
  exports: [...databaseProviders],
})
export class DatabaseModule {}
```

이제 `@Inject()` 데코레이터를 사용하여 `Connection` 객체를 삽입할 수 있습니다. `Connection` 비동기 공급자에 의존하는 각 클래스는 `Promise`가 해결될 때까지 기다립니다.

#### Model injection

Mongoose를 사용하면 모든 것이 [Schema](https://mongoosejs.com/docs/guide.html)에서 파생됩니다. `CatSchema`를 정의해 보겠습니다.

```typescript
@@filename(schemas/cat.schema)
import * as mongoose from 'mongoose';

export const CatSchema = new mongoose.Schema({
  name: String,
  age: Number,
  breed: String,
});
```

`CatsSchema`는 `cats` 디렉토리에 속합니다. 이 디렉토리는 `CatsModule`을 나타냅니다.

이제 **모델** 프로바이더를 만들 차례입니다.

```typescript
@@filename(cats.providers)
import { Connection } from 'mongoose';
import { CatSchema } from './schemas/cat.schema';

export const catsProviders = [
  {
    provide: 'CAT_MODEL',
    useFactory: (connection: Connection) => connection.model('Cat', CatSchema),
    inject: ['DATABASE_CONNECTION'],
  },
];
@@switch
import { CatSchema } from './schemas/cat.schema';

export const catsProviders = [
  {
    provide: 'CAT_MODEL',
    useFactory: (connection) => connection.model('Cat', CatSchema),
    inject: ['DATABASE_CONNECTION'],
  },
];
```

> warning **경고** 실제 애플리케이션에서는 **매직 스트링**을 피해야합니다. `CAT_MODEL`과 `DATABASE_CONNECTION`은 모두 분리된 `constants.ts` 파일에 보관되어야합니다.

이제 `@Inject()` 데코레이터를 사용하여 `CatsService`에 `CAT_MODEL`을 삽입할 수 있습니다.

```typescript
@@filename(cats.service)
import { Model } from 'mongoose';
import { Injectable, Inject } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';
import { CreateCatDto } from './dto/create-cat.dto';

@Injectable()
export class CatsService {
  constructor(
    @Inject('CAT_MODEL')
    private catModel: Model<Cat>,
  ) {}

  async create(createCatDto: CreateCatDto): Promise<Cat> {
    const createdCat = new this.catModel(createCatDto);
    return createdCat.save();
  }

  async findAll(): Promise<Cat[]> {
    return this.catModel.find().exec();
  }
}
@@switch
import { Injectable, Dependencies } from '@nestjs/common';

@Injectable()
@Dependencies('CAT_MODEL')
export class CatsService {
  constructor(catModel) {
    this.catModel = catModel;
  }

  async create(createCatDto) {
    const createdCat = new this.catModel(createCatDto);
    return createdCat.save();
  }

  async findAll() {
    return this.catModel.find().exec();
  }
}
```

위의 예에서는 `Cat` 인터페이스를 사용했습니다. 이 인터페이스는 mongoose 패키지의 `Document`를 확장합니다.

```typescript
import { Document } from 'mongoose';

export interface Cat extends Document {
  readonly name: string;
  readonly age: number;
  readonly breed: string;
}
```

데이터베이스 연결은 **비동기적**이지만 Nest는 이 프로세스를 최종 사용자에게 완전히 보이지 않게합니다. `CatModel` 클래스는 db 연결을 기다리고 있으며 `CatsService`는 모델을 사용할 준비가 될 때까지 지연됩니다. 각 클래스가 인스턴스화될 때 전체 애플리케이션이 시작될 수 있습니다.

다음은 마지막 `CatsModule`입니다.

```typescript
@@filename(cats.module)
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';
import { catsProviders } from './cats.providers';
import { DatabaseModule } from '../database/database.module';

@Module({
  imports: [DatabaseModule],
  controllers: [CatsController],
  providers: [
    CatsService,
    ...catsProviders,
  ],
})
export class CatsModule {}
```

> info **힌트** `CatsModule`을 루트 `AppModule`로 가져오는 것을 잊지 마십시오.
