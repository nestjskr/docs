### SQL (TypeORM)

##### This chapter applies only to TypeScript

> **경고** 이 문서에서는 사용자 지정 프로바이더 메커니즘을 사용하여 처음부터 **TypeORM** 패키지를 기반으로 `DatabaseModule`을 만드는 방법을 알아봅니다. 결과적으로 이 솔루션에는 바로 사용할 수 있고 즉시 사용 가능한 전용 `@nestjs/typeorm` 패키지를 사용하여 생략할 수 있는 많은 오버헤드가 포함되어 있습니다. 자세한 내용은 [여기](/techniques/sql)를 참조하십시오.

[TypeORM](https://github.com/typeorm/typeorm)은 확실히 node.js 세계에서 사용할 수 있는 가장 성숙한 객체 관계형 매퍼 (ORM)입니다. TypeScript로 작성되었기 때문에 Nest 프레임워크와 잘 작동합니다.

#### Getting started

이 라이브러리로 모험을 시작하려면 필요한 모든 종속성을 설치해야합니다.

```bash
$ npm install --save typeorm mysql
```

우리가 해야할 첫번째 단계는 `typeorm` 패키지에서 가져온 `createConnection()` 함수를 사용하여 데이터베이스와의 연결을 설정하는 것입니다. `createConnection()` 함수는 `Promise`를 반환하므로 [async provider](/fundamentals/async-components)를 생성해야합니다.

```typescript
@@filename(database.providers)
import { createConnection } from 'typeorm';

export const databaseProviders = [
  {
    provide: 'DATABASE_CONNECTION',
    useFactory: async () => await createConnection({
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'root',
      database: 'test',
      entities: [
          __dirname + '/../**/*.entity{.ts,.js}',
      ],
      synchronize: true,
    }),
  },
];
```

> warning **경고** `synchronize: true` 설정은 프로덕션에서 사용해서는 안됩니다. 그렇지 않으면 프로덕션 데이터가 손실될 수 있습니다.

> info **힌트** 모범사례에 따라 `*.providers.ts` 접미사가 있는 별도의 파일에서 사용자 지정 프로바이더를 선언했습니다.

그런 다음이 프로바이더를 내보내 나머지 애플리케이션에서 **액세스 가능**하도록 만들어야합니다.

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

이제 `@Inject()` 데코레이터를 사용하여 `Connection` 객체를 삽입할 수 있습니다. `Connection` 비동기 프로바이더에 의존하는 각 클래스는 `Promise`가 해결될 때까지 기다립니다.

#### Repository pattern

[TypeORM](https://github.com/typeorm/typeorm)은 리포지토리 디자인 패턴을 지원하므로 각 엔터티에는 자체 리포지토리가 있습니다. 이러한 저장소는 데이터베이스 연결에서 얻을 수 있습니다.

그러나 첫째, 적어도 하나의 엔티티가 필요합니다. 공식 문서에서 `Photo` 엔티티를 재사용할 것입니다.

```typescript
@@filename(photo.entity)
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class Photo {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ length: 500 })
  name: string;

  @Column('text')
  description: string;

  @Column()
  filename: string;

  @Column('int')
  views: number;

  @Column()
  isPublished: boolean;
}
```

`Photo` 엔티티는 `photo` 디렉토리에 속합니다. 이 디렉토리는 `PhotoModule`을 나타냅니다. 이제 **Repository** 프로바이더를 만들어 보겠습니다.

```typescript
@@filename(photo.providers)
import { Connection, Repository } from 'typeorm';
import { Photo } from './photo.entity';

export const photoProviders = [
  {
    provide: 'PHOTO_REPOSITORY',
    useFactory: (connection: Connection) => connection.getRepository(Photo),
    inject: ['DATABASE_CONNECTION'],
  },
];
```

> warning **경고** 실제 애플리케이션에서는 **매직 스트링**을 피해야합니다. `PHOTO_REPOSITORY`와 `DATABASE_CONNECTION`은 모두 분리 된 `constants.ts` 파일에 보관해야합니다.

이제 `@Inject()` 데코레이터를 사용하여 `Repository<Photo>`를 `PhotoService`에 삽입할 수 있습니다.

```typescript
@@filename(photo.service)
import { Injectable, Inject } from '@nestjs/common';
import { Repository } from 'typeorm';
import { Photo } from './photo.entity';

@Injectable()
export class PhotoService {
  constructor(
    @Inject('PHOTO_REPOSITORY')
    private photoRepository: Repository<Photo>,
  ) {}

  async findAll(): Promise<Photo[]> {
    return this.photoRepository.find();
  }
}
```

데이터베이스 연결은 **비동기적**이지만 Nest는 이 프로세스를 최종 사용자에게 완전히 보이지 않게합니다. `PhotoRepository`는 db 연결을 기다리고 있으며 `PhotoService`는 저장소를 사용할 준비가 될 때까지 지연됩니다. 각 클래스가 인스턴스화될 때 전체 애플리케이션이 시작될 수 있습니다.

다음은 마지막 `PhotoModule`입니다.

```typescript
@@filename(photo.module)
import { Module } from '@nestjs/common';
import { DatabaseModule } from '../database/database.module';
import { photoProviders } from './photo.providers';
import { PhotoService } from './photo.service';

@Module({
  imports: [DatabaseModule],
  providers: [
    ...photoProviders,
    PhotoService,
  ],
})
export class PhotoModule {}
```

> info **힌트** `PhotoModule`을 루트 `AppModule`로 가져오는 것을 잊지 마십시오.
