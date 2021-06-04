### Mongo

Nest는 [MongoDB](https://www.mongodb.com/) 데이터베이스와의 통합을 위해 두가지 방법을 지원합니다. MongoDB용 커넥터가 있는 [여기](/techniques/database)에 설명된 내장 [TypeORM](https://github.com/typeorm/typeorm) 모듈을 사용하거나 가장 인기있는 MongoDB 개체 모델링 도구 인 [Mongoose](https://mongoosejs.com)를 사용할 수 있습니다. 이 장에서는 전용 `@nestjs/mongoose` 패키지를 사용하여 후자를 설명합니다.

필요한 종속성을 설치하여 시작하십시오.

```bash
$ npm install --save @nestjs/mongoose mongoose
```

설치 프로세스가 완료되면 `MongooseModule`을 루트 `AppModule`로 가져올 수 있습니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';

@Module({
  imports: [MongooseModule.forRoot('mongodb://localhost/nest')],
})
export class AppModule {}
```

`forRoot()` 메소드는 [여기](https://mongoosejs.com/docs/connections.html)에 설명된대로 Mongoose 패키지의 `mongoose.connect()`와 동일한 구성 객체를 허용합니다.

#### Model injection

Mongoose에서는 모든 것이 [스키마](http://mongoosejs.com/docs/guide.html)에서 파생됩니다. 각 스키마는 MongoDB 컬렉션에 매핑되고 해당 컬렉션 내의 문서 모양을 정의합니다. 스키마는 [모델](https://mongoosejs.com/docs/models.html)을 정의하는 데 사용됩니다. 모델은 기본 MongoDB 데이터베이스에서 문서를 만들고 읽는 역할을합니다.

스키마는 NestJS 데코레이터를 사용하거나 Mongoose 자체를 사용하여 수동으로 만들 수 있습니다. 데코레이터를 사용하여 스키마를 생성하면 상용구가 크게 줄어들고 전체 코드 가독성이 향상됩니다.

`CatSchema`를 정의해 보겠습니다.

```typescript
@@filename(schemas/cat.schema)
import { Prop, Schema, SchemaFactory } from '@nestjs/mongoose';
import { Document } from 'mongoose';

export type CatDocument = Cat & Document;

@Schema()
export class Cat {
  @Prop()
  name: string;

  @Prop()
  age: number;

  @Prop()
  breed: string;
}

export const CatSchema = SchemaFactory.createForClass(Cat);
```

> info **힌트** `DefinitionsFactory` 클래스 (`nestjs/mongoose`에서)를 사용하여 원시 스키마 정의를 생성할 수도 있습니다. 이렇게 하면 제공한 메타데이터를 기반으로 생성된 스키마 정의를 수동으로 수정할 수 있습니다. 이것은 데코레이터로 모든 것을 표현하기 어려울 수 있는 특정 엣지 케이스에 유용합니다.

`@Schema()` 데코레이터는 클래스를 스키마 정의로 표시합니다. 그것은 우리의 `Cat`클래스를 같은 이름의 MongoDB 컬렉션에 매핑하지만 끝에 추가 "s"를 추가하여 최종 mongo 컬렉션 이름은 `cats`가 됩니다. 이 데코레이터는 스키마 옵션 객체인 단일 선택적 인수를 허용합니다. 일반적으로 `mongoose.Schema` 클래스' 생성자의 두 번째 인수로 전달하는 객체로 생각하세요 (예: `new mongoose.Schema(_, options)`)). 사용 가능한 스키마 옵션에 대한 자세한 내용은 [이](https://mongoosejs.com/docs/guide.html#options) 장을 참조하세요.

`@Prop()` 데코레이터는 문서의 속성을 정의합니다. 예를 들어 위의 스키마 정의에서 `name`, `age`, `breed`의 세가지 속성을 정의했습니다. 이러한 속성의 [스키마 유형](https://mongoosejs.com/docs/schematypes.html)은 TypeScript 메타데이터 (및 리플렉션) 기능 덕분에 자동으로 유추됩니다. 그러나 형식을 암시적으로 반영할 수 없는 더 복잡한 시나리오(예: 배열 또는 중첩된 객체 구조)에서는 다음과 같이 형식을 명시적으로 표시해야합니다.

```typescript
@Prop([String])
tags: string[];
```

또는 `@Prop()` 데코레이터는 옵션 객체 인수 (사용 가능한 옵션에 대한 [자세히 알아보기](https://mongoosejs.com/docs/schematypes.html#schematype-options))를 받습니다. 이를 통해 속성이 필요한지 여부를 나타내거나 기본값을 지정하거나 변경 불가능으로 표시할 수 있습니다. 예를 들면:

```typescript
@Prop({ required: true })
name: string;
```

나중에 채우기를 위해 다른 모델과의 관계를 지정하려면 `@Prop()` 데코레이터도 사용할 수 있습니다. 예를 들어 `Cat`에 `owners`라는 다른 컬렉션에 저장된 `Owner`가 있는 경우 속성에는 타입과 참조가 있어야 합니다. 예를 들면:

```typescript
import * as mongoose from 'mongoose';
import { Owner } from '../owners/schemas/owner.schema';

// inside the class definition
@Prop({ type: mongoose.Schema.Types.ObjectId, ref: 'Owner' })
owner: Owner;
```

소유자가 여러명인 경우 속성 구성은 다음과 같아야합니다.

```typescript
@Prop({ type: [{ type: mongoose.Schema.Types.ObjectId, ref: 'Owner' }] })
owner: Owner[];
```

마지막으로 **원시 raw** 스키마 정의도 데코레이터에 전달할 수 있습니다. 예를 들어 속성이 클래스로 정의되지 않은 중첩된 객체를 나타낼 때 유용합니다. 이를 위해 다음과 같이 `@nestjs/mongoose` 패키지의 `raw()` 함수를 사용합니다.

```typescript
@Prop(raw({
  firstName: { type: String },
  lastName: { type: String }
}))
details: Record<string, any>;
```

또는 **데코레이터를 사용하지 않음**을 선호하는 경우 스키마를 수동으로 정의할 수 있습니다. 예를 들면:

```typescript
export const CatSchema = new mongoose.Schema({
  name: String,
  age: Number,
  breed: String,
});
```

`cat.schema` 파일은 `cats` 디렉토리의 폴더에 있으며 여기서 `CatsModule`도 정의합니다. 원하는 곳에 스키마 파일을 저장할 수 있지만 적절한 모듈 디렉토리의 관련 **도메인** 객체 근처에 저장하는 것이 좋습니다.

`CatsModule`을 살펴 보겠습니다.

```typescript
@@filename(cats.module)
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';
import { Cat, CatSchema } from './schemas/cat.schema';

@Module({
  imports: [MongooseModule.forFeature([{ name: Cat.name, schema: CatSchema }])],
  controllers: [CatsController],
  providers: [CatsService],
})
export class CatsModule {}
```

`MongooseModule`은 모듈을 구성하기 위한 `forFeature()` 메소드를 제공하며 현재 범위에 등록해야하는 모델을 정의합니다. 다른 모듈에서도 모델을 사용하려면 MongooseModule을 `CatsModule`의 `exports` 섹션에 추가하고 다른 모듈에서 `CatsModule`을 가져옵니다.

스키마를 등록한 후 `@InjectModel()` 데코레이터를 사용하여 `Cat` 모델을 `CatsService`에 삽입할 수 있습니다.

```typescript
@@filename(cats.service)
import { Model } from 'mongoose';
import { Injectable } from '@nestjs/common';
import { InjectModel } from '@nestjs/mongoose';
import { Cat, CatDocument } from './schemas/cat.schema';
import { CreateCatDto } from './dto/create-cat.dto';

@Injectable()
export class CatsService {
  constructor(@InjectModel(Cat.name) private catModel: Model<CatDocument>) {}

  async create(createCatDto: CreateCatDto): Promise<Cat> {
    const createdCat = new this.catModel(createCatDto);
    return createdCat.save();
  }

  async findAll(): Promise<Cat[]> {
    return this.catModel.find().exec();
  }
}
@@switch
import { Model } from 'mongoose';
import { Injectable, Dependencies } from '@nestjs/common';
import { getModelToken } from '@nestjs/mongoose';
import { Cat } from './schemas/cat.schema';

@Injectable()
@Dependencies(getModelToken(Cat.name))
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

#### Connection

때때로 기본 [Mongoose Connection](https://mongoosejs.com/docs/api.html#Connection) 객체에 액세스해야 할 수 있습니다. 예를 들어 연결 객체에 대한 기본 API 호출을 수행할 수 있습니다. 다음과 같이 `@InjectConnection()` 데코레이터를 사용하여 Mongoose Connection을 삽입할 수 있습니다.

```typescript
import { Injectable } from '@nestjs/common';
import { InjectConnection } from '@nestjs/mongoose';
import { Connection } from 'mongoose';

@Injectable()
export class CatsService {
  constructor(@InjectConnection() private connection: Connection) {}
}
```

#### Multiple databases

일부 프로젝트에는 여러 데이터베이스 연결이 필요합니다. 이것은 또한 이 모듈을 통해 달성할 수 있습니다. 여러 연결로 작업하려면 먼저 연결을 만듭니다. 이 경우 연결 이름 지정은 **필수**가 됩니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';

@Module({
  imports: [
    MongooseModule.forRoot('mongodb://localhost/test', {
      connectionName: 'cats',
    }),
    MongooseModule.forRoot('mongodb://localhost/users', {
      connectionName: 'users',
    }),
  ],
})
export class AppModule {}
```

> warning **알림** 이름이 없거나 이름이 같은 연결이 여러개 있으면 안됩니다. 그렇지 않으면 연결이 무시됩니다.

이 설정을 사용하면 `MongooseModule.forFeature()` 함수에 어떤 연결을 사용해야 하는지 알려야합니다.

```typescript
@Module({
  imports: [
    MongooseModule.forFeature([{ name: Cat.name, schema: CatSchema }], 'cats'),
  ],
})
export class AppModule {}
```

You can also inject the `Connection` for a given connection:

```typescript
import { Injectable } from '@nestjs/common';
import { InjectConnection } from '@nestjs/mongoose';
import { Connection } from 'mongoose';

@Injectable()
export class CatsService {
  constructor(@InjectConnection('cats') private connection: Connection) {}
}
```

커스텀 프로바이더(예: 팩토리 프로바이더)에 지정된 `Connection`을 삽입하려면 연결 이름을 인수로 전달하는 `getConnectionToken ()` 함수를 사용합니다.

```typescript
{
  provide: CatsService,
  useFactory: (catsConnection: Connection) => {
    return new CatsService(catsConnection);
  },
  inject: [getConnectionToken('cats')],
}
```

#### Hooks (middleware)

미들웨어(사전 및 사후 후크라고도 함)는 비동기 함수 실행중에 제어를 전달하는 함수입니다. 미들웨어는 스키마 수준에서 지정되며 플러그인([source](https://mongoosejs.com/docs/middleware.html))을 작성하는 데 유용합니다. 모델을 컴파일한 후 `pre()` 또는 `post()`를 호출하면 Mongoose에서 작동하지 않습니다. 모델 등록 **전**에 후크를 등록하려면 `MongooseModule`의 `forFeatureAsync()` 메서드를 팩토리 공급자(즉,`useFactory`)와 함께 사용하세요. 이 기술을 사용하면 스키마 객체에 액세스한 다음 `pre()` 또는 `post()` 메서드를 사용하여 해당 스키마에 후크를 등록할 수 있습니다. 아래 예를 참조하십시오.

```typescript
@Module({
  imports: [
    MongooseModule.forFeatureAsync([
      {
        name: Cat.name,
        useFactory: () => {
          const schema = CatsSchema;
          schema.pre('save', function() { console.log('Hello from pre save') });
          return schema;
        },
      },
    ]),
  ],
})
export class AppModule {}
```

다른 [팩토리 프로바이더](/fundamentals/custom-providers#factory-providers-usefactory)와 마찬가지로 팩토리 함수는 `async`일 수 있으며 `inject`을 통해 종속성을 삽입할 수 있습니다.

```typescript
@Module({
  imports: [
    MongooseModule.forFeatureAsync([
      {
        name: Cat.name,
        imports: [ConfigModule],
        useFactory: (configService: ConfigService) => {
          const schema = CatsSchema;
          schema.pre('save', function() {
            console.log(
              `${configService.get('APP_NAME')}: Hello from pre save`,
            ),
          });
          return schema;
        },
        inject: [ConfigService],
      },
    ]),
  ],
})
export class AppModule {}
```

#### Plugins

지정된 스키마에 대한 [플러그인](https://mongoosejs.com/docs/plugins.html)을 등록하려면 `forFeatureAsync()` 메서드를 사용하세요.

```typescript
@Module({
  imports: [
    MongooseModule.forFeatureAsync([
      {
        name: Cat.name,
        useFactory: () => {
          const schema = CatsSchema;
          schema.plugin(require('mongoose-autopopulate'));
          return schema;
        },
      },
    ]),
  ],
})
export class AppModule {}
```

모든 스키마에 대한 플러그인을 한번에 등록하려면 `Connection` 객체의 `.plugin()` 메서드를 호출합니다. 모델을 생성하기 전에 연결에 액세스해야 합니다. 이렇게하려면 `connectionFactory`를 사용하십시오.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';

@Module({
  imports: [
    MongooseModule.forRoot('mongodb://localhost/test', {
      connectionFactory: (connection) => {
        connection.plugin(require('mongoose-autopopulate'));
        return connection;
      }
    }),
  ],
})
export class AppModule {}
```

#### Discriminators

[판별자 Discriminators](https://mongoosejs.com/docs/discriminators.html)는 스키마 상속 메커니즘입니다. 이를 통해 동일한 기본 MongoDB 컬렉션 위에 겹치는 스키마가 있는 여러 모델을 가질 수 있습니다.

단일 컬렉션에서 다양한 유형의 이벤트를 추적하려고 한다고 가정합니다. 모든 이벤트에는 타임스탬프가 있습니다.

```typescript
@@filename(event.schema)
@Schema({ discriminatorKey: 'kind' })
export class Event {
  @Prop({
    type: String,
    required: true,
    enum: [ClickedLinkEvent.name, SignUpEvent.name],
  })
  kind: string;

  @Prop({ type: Date, required: true })
  time: Date;
}

export const EventSchema = SchemaFactory.createForClass(Event);
```

> info **힌트** mongoose가 다른 판별자 모델간의 차이를 알려주는 방식은 기본적으로 `__t`인 "판별자 키"입니다. Mongoose는 이 문서가 인스턴스인 판별자를 추적하는 데 사용하는 스키마에 `__t`라는 문자열 경로를 추가합니다. `discriminatorKey` 옵션을 사용하여 차별 경로를 정의할 수도 있습니다.

`SignedUpEvent` 및 `ClickedLinkEvent` 인스턴스는 일반 이벤트와 동일한 컬렉션에 저장됩니다.

이제 다음과 같이 `ClickedLinkEvent` 클래스를 정의해 보겠습니다.

```typescript
@@filename(click-link-event.schema)
@Schema()
export class ClickedLinkEvent {
  kind: string;
  time: Date;

  @Prop({ type: String, required: true })
  url: string;
}

export const ClickedLinkEventSchema = SchemaFactory.createForClass(ClickedLinkEvent);
```

그리고 `SignUpEvent` 클래스:

```typescript
@@filename(sign-up-event.schema)
@Schema()
export class SignUpEvent {
  kind: string;
  time: Date;

  @Prop({ type: String, required: true })
  user: string;
}

export const SignUpEventSchema = SchemaFactory.createForClass(SignUpEvent);
```

이 위치에서 `discriminators` 옵션을 사용하여 주어진 스키마에 대한 판별자를 등록하십시오. `MongooseModule.forFeature` 및 `MongooseModule.forFeatureAsync` 모두에서 작동합니다.

```typescript
@@filename(event.module)
import { Module } from '@nestjs/common';
import { MongooseModule } from '@nestjs/mongoose';

@Module({
  imports: [
    MongooseModule.forFeature([
      {
        name: Event.name,
        schema: EventSchema,
        discriminators: [
          { name: ClickedLinkEvent.name, schema: ClickedLinkEventSchema },
          { name: SignUpEvent.name, schema: SignUpEventSchema },
        ],
      },
    ]),
  ]
})
export class EventsModule {}
```

#### Testing

애플리케이션을 단위 테스트할 때 일반적으로 데이터베이스 연결을 피하여 테스트 스위트를 더 간단하게 설정하고 실행할 수 있도록합니다. 그러나 우리의 클래스는 연결 인스턴스에서 가져온 모델에 따라 달라질 수 있습니다. 이 클래스를 어떻게 해결합니까? 해결책은 모의 모델을 만드는 것입니다.

이를 보다 쉽게하기 위해 `@nestjs/mongoose` 패키지는 준비된 [주입 토큰](/fundamentals/custom-providers#di-fundamentals)을 반환하는 `getModelToken()` 함수를 노출합니다. 토큰 이름을 기반으로 합니다. 이 토큰을 사용하면 `useClass`, `useValue`, `useFactory`를 포함한 표준 [커스텀 프로바이더](/fundamentals/custom-providers) 기술을 사용하여 모의 구현을 쉽게 제공할 수 있습니다. 예를 들면:

```typescript
@Module({
  providers: [
    CatsService,
    {
      provide: getModelToken(Cat.name),
      useValue: catModel,
    },
  ],
})
export class CatsModule {}
```

이 예제에서는 소비자가 `@InjectModel()` 데코레이터를 사용하여 `Model<Cat>`을 삽입할 때마다 하드코딩된 `catModel` (객체 인스턴스)이 제공됩니다.

<app-banner-courses></app-banner-courses>

#### Async configuration

모듈 옵션을 정적으로 전달하는 대신 비동기적으로 전달해야 하는 경우 `forRootAsync()` 메서드를 사용하세요. 대부분의 동적 모듈과 마찬가지로 Nest는 비동기 구성을 처리하는 몇가지 기술을 제공합니다.

한가지 기술은 팩토리 기능을 사용하는 것입니다.

```typescript
MongooseModule.forRootAsync({
  useFactory: () => ({
    uri: 'mongodb://localhost/nest',
  }),
});
```

다른 [팩토리 프로바이더](/fundamentals/custom-providers#factory-providers-usefactory)와 마찬가지로 팩토리 함수는 `async`일 수 있으며 `inject`을 통해 종속성을 삽입할 수 있습니다.

```typescript
MongooseModule.forRootAsync({
  imports: [ConfigModule],
  useFactory: async (configService: ConfigService) => ({
    uri: configService.get<string>('MONGODB_URI'),
  }),
  inject: [ConfigService],
});
```

또는 아래와 같이 팩토리 대신 클래스를 사용하여 `MongooseModule`을 구성할 수 있습니다.

```typescript
MongooseModule.forRootAsync({
  useClass: MongooseConfigService,
});
```

위의 구성은 `MongooseModule`내에서 `MongooseConfigService`를 인스턴스화하여 필요한 옵션 객체를 만드는 데 사용합니다. 이 예에서 `MongooseConfigService`는 아래와 같이 `MongooseOptionsFactory` 인터페이스를 구현해야 합니다. `MongooseModule`은 제공된 클래스의 인스턴스화 된 객체에 대해 `createMongooseOptions()` 메서드를 호출합니다.

```typescript
@Injectable()
class MongooseConfigService implements MongooseOptionsFactory {
  createMongooseOptions(): MongooseModuleOptions {
    return {
      uri: 'mongodb://localhost/nest',
    };
  }
}
```

`MongooseModule` 내부에 비공개 사본을 만드는 대신 기존 옵션 프로바이더를 재사용하려면 `useExisting`구문을 사용하세요.

```typescript
MongooseModule.forRootAsync({
  imports: [ConfigModule],
  useExisting: ConfigService,
});
```

#### Example

작동하는 예는 [여기](https://github.com/nestjs/nest/tree/master/sample/06-mongoose)에서 확인할 수 있습니다.
