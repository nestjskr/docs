### Database

Nest는 데이터베이스에 구애받지 않으므로 모든 SQL 또는 NoSQL 데이터베이스와 쉽게 통합할 수 있습니다. 선호도에 따라 다양한 옵션을 사용할 수 있습니다. 가장 일반적인 수준에서 Nest를 데이터베이스에 연결하는 것은 [Express](https://expressjs.com/en/guide/database-integration.html) 또는 Fastify에서와 마찬가지로 데이터베이스에 적합한 Node.js 드라이버를 로드하기만 하면됩니다.

또한 [Sequelize](https://sequelize.org/)([Sequelize 통합](/techniques/database#sequelize-integration) 섹션으로 이동), [Knex.js](https://knexjs.org/)([tutorial](https://dev.to/nestjs/build-a-nestjs-module-for-knex-js-or-other-resource-based-libraries-in-5-minutes-12an)), [TypeORM](https://github.com/typeorm/typeorm) 및 [Prisma](https://www.github.com/prisma/prisma)([recipe](/recipes/prisma))와 같은 범용 Node.js 데이터베이스 통합 **라이브러리** 또는 ORM을 직접 사용하여 더 높은 수준의 추상화에서 작동 할 수 있습니다.

편의를 위해 Nest는 현재 장에서 다룰 `@nestjs/typeorm` 및 `@nestjs/sequelize` 패키지를 사용하여 TypeORM 및 Sequelize와 기본적으로 긴밀한 통합을 제공하며, Mongoose는 `@nestjs/mongoose`, [이 장](/techniques/mongodb)에서 다룹니다. 이러한 통합은 모델/리포지토리 삽입, 테스트 가능성 및 비동기 구성과 같은 추가 NestJS 관련 기능을 제공하여 선택한 데이터베이스에 더욱 쉽게 액세스할 수 있도록 합니다.

### TypeORM Integration

SQL 및 NoSQL 데이터베이스와의 통합을 위해 Nest는 `@nestjs/typeorm` 패키지를 제공합니다. Nest는 TypeScript에 사용할 수 있는 가장 성숙된 ORM(Object Relational Mapper)이기 때문에 [TypeORM](https://github.com/typeorm/typeorm)을 사용합니다. TypeScript로 작성되었으므로 Nest 프레임워크와 잘 통합됩니다.

사용을 시작하려면 먼저 필요한 종속성을 설치합니다. 이 장에서는 널리 사용되는 [MySQL](https://www.mysql.com/) 관계형 DBMS를 사용하는 방법을 설명하지만 TypeORM은 PostgreSQL, Oracle, Microsoft SQL Server, SQLite, MongoDB와 같은 NoSQL 데이터베이스도 있습니다. 이 장에서 설명하는 절차는 TypeORM에서 지원하는 모든 데이터베이스에 대해 동일합니다. 선택한 데이터베이스에 연결된 클라이언트 API 라이브러리를 설치하기만 하면 됩니다.

```bash
$ npm install --save @nestjs/typeorm typeorm mysql2
```

설치 프로세스가 완료되면 `TypeOrmModule`을 루트 `AppModule`로 가져올 수 있습니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'root',
      database: 'test',
      entities: [],
      synchronize: true,
    }),
  ],
})
export class AppModule {}
```

> warning **경고** `synchronize: true` 설정은 프로덕션에서 사용해서는 안됩니다. 그렇지 않으면 프로덕션 데이터가 손실될 수 있습니다.

`forRoot()` 메소드는 [TypeORM](https://typeorm.io/#/connection-options) 패키지의 `createConnection()` 함수에 의해 노출되는 모든 구성 속성을 지원합니다. 또한 아래에 설명된 몇가지 추가 구성 속성이 있습니다.

<table>
  <tr>
    <td><code>retryAttempts</code></td>
    <td>데이터베이스 연결 시도 횟수 (기본값: <code>10</code>)</td>
  </tr>
  <tr>
    <td><code>retryDelay</code></td>
    <td>연결 재시도 간격(밀리 초)(기본값: <code>3000</code>)</td>
  </tr>
  <tr>
    <td><code>autoLoadEntities</code></td>
    <td><code>true</code>이면 엔티티가 자동으로 로드됩니다(기본값: <code>false</code>).</td>
  </tr>
  <tr>
    <td><code>keepConnectionAlive</code></td>
    <td><code>true</code>이면 애플리케이션 종료시 연결이 닫히지 않습니다(기본값: <code>false</code>).</td>
  </tr>
</table>

> info **힌트** 연결 옵션에 대한 자세한 내용은 [여기](https://typeorm.io/#/connection-options)를 참조하세요.

또는 구성 객체를 `forRoot()`에 전달하는 대신 프로젝트 루트 디렉토리에 `ormconfig.json` 파일을 만들 수 있습니다.

```json
{
  "type": "mysql",
  "host": "localhost",
  "port": 3306,
  "username": "root",
  "password": "root",
  "database": "test",
  "entities": ["dist/**/*.entity{.ts,.js}"],
  "synchronize": true
}
```

그런 다음 옵션없이 `forRoot()`를 호출 할 수 있습니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [TypeOrmModule.forRoot()],
})
export class AppModule {}
```

> warning **경고** 정적 glob 경로 (예: `dist/**/*.entity{{ '{' }} .ts,.js{{ '}' }}`)는 [webpack에서 제대로 작동하지 않습니다.](https://webpack.js.org/).

> info **힌트** `ormconfig.json` 파일은 `typeorm` 라이브러리에 의해 로드됩니다. 따라서 위에서 설명한 추가 속성( `forRoot()` 메서드를 통해 내부적으로 지원되는 속성(예: `autoLoadEntities` 및 `retryDelay`))은 적용되지 않습니다. 다행히 TypeORM은 `ormconfig` 파일 또는 환경 변수에서 연결 옵션을 읽는 [`getConnectionOptions`](https://typeorm.io/#/using-ormconfig/overriding-options-defined-in-ormconfig)함수를 제공합니다. 이를 통해 다음과 같이 구성 파일을 계속 사용하고 Nest 관련 옵션을 설정할 수 있습니다.
> ```typescript
> TypeOrmModule.forRootAsync({
>   useFactory: async () =>
>     Object.assign(await getConnectionOptions(), {
>       autoLoadEntities: true,
>     }),
> })
> ```

이 작업이 완료되면 TypeORM `Connection` 및 `EntityManager` 객체를 사용하여 전체 프로젝트에 삽입할 수 있습니다(모듈을 가져올 필요없이). 예:

```typescript
@@filename(app.module)
import { Connection } from 'typeorm';

@Module({
  imports: [TypeOrmModule.forRoot(), UsersModule],
})
export class AppModule {
  constructor(private connection: Connection) {}
}
@@switch
import { Connection } from 'typeorm';

@Dependencies(Connection)
@Module({
  imports: [TypeOrmModule.forRoot(), UsersModule],
})
export class AppModule {
  constructor(connection) {
    this.connection = connection;
  }
}
```

#### Repository pattern

[TypeORM](https://github.com/typeorm/typeorm)은 **저장소 디자인 패턴**을 지원하므로 각 항목에 자체 저장소가 있습니다. 이러한 저장소는 데이터베이스 연결에서 얻을 수 있습니다.

예제를 계속하려면 하나 이상의 엔티티가 필요합니다. `User` 엔터티를 정의해 보겠습니다.

```typescript
@@filename(user.entity)
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column({ default: true })
  isActive: boolean;
}
```

> info **힌트** [TypeORM 문서](https://typeorm.io/#/entities)에서 항목에 대해 자세히 알아보세요.

`User` 엔티티 파일은 `users` 디렉토리에 있습니다. 이 디렉토리는 `UsersModule`과 관련된 모든 파일을 포함합니다. 모델 파일을 보관할 위치를 결정할 수 있지만 해당 모듈 디렉토리의 **도메인** 근처에 파일을 만드는 것이 좋습니다.

`User` 엔티티 사용을 시작하려면 모듈 `forRoot()` 메소드 옵션의 `entities` 배열에 삽입하여 TypeORM에 알려야합니다(정적 glob 경로를 사용하지 않는 한) :

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { User } from './users/user.entity';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'root',
      database: 'test',
      entities: [User],
      synchronize: true,
    }),
  ],
})
export class AppModule {}
```

다음으로 `UsersModule`을 살펴 보겠습니다.

```typescript
@@filename(users.module)
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersService } from './users.service';
import { UsersController } from './users.controller';
import { User } from './user.entity';

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  providers: [UsersService],
  controllers: [UsersController],
})
export class UsersModule {}
```

이 모듈은 `forFeature()` 메소드를 사용하여 현재 범위에 등록된 저장소를 정의합니다. 그런 다음 `@InjectRepository()` 데코레이터를 사용하여 `UsersRepository`를 `UsersService`에 삽입할 수 있습니다.

```typescript
@@filename(users.service)
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private usersRepository: Repository<User>,
  ) {}

  findAll(): Promise<User[]> {
    return this.usersRepository.find();
  }

  findOne(id: string): Promise<User> {
    return this.usersRepository.findOne(id);
  }

  async remove(id: string): Promise<void> {
    await this.usersRepository.delete(id);
  }
}
@@switch
import { Injectable, Dependencies } from '@nestjs/common';
import { getRepositoryToken } from '@nestjs/typeorm';
import { User } from './user.entity';

@Injectable()
@Dependencies(getRepositoryToken(User))
export class UsersService {
  constructor(usersRepository) {
    this.usersRepository = usersRepository;
  }

  findAll() {
    return this.usersRepository.find();
  }

  findOne(id) {
    return this.usersRepository.findOne(id);
  }

  async remove(id) {
    await this.usersRepository.delete(id);
  }
}
```

> warning **알림** `UsersModule`을 루트 `AppModule`로 가져오는 것을 잊지 마십시오.

`TypeOrmModule.forFeature`를 가져오는 모듈 외부의 저장소를 사용하려면 해당 저장소에서 생성한 프로바이더를 다시 내 보내야합니다.
다음과 같이 전체 모듈을 내보내면 됩니다.

```typescript
@@filename(users.module)
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { User } from './user.entity';

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  exports: [TypeOrmModule]
})
export class UsersModule {}
```

이제 `UserHttpModule`에서 `UsersModule`을 가져오면 후자의 모듈 공급자에서 `@InjectRepository(User)`를 사용할 수 있습니다.

```typescript
@@filename(users-http.module)
import { Module } from '@nestjs/common';
import { UsersModule } from './users.module';
import { UsersService } from './users.service';
import { UsersController } from './users.controller';

@Module({
  imports: [UsersModule],
  providers: [UsersService],
  controllers: [UsersController]
})
export class UserHttpModule {}
```

### Relations

관계는 두개 이상의 테이블간에 설정된 연결입니다. 관계는 종종 기본 및 외래 키를 포함하는 각 테이블의 공통 필드를 기반으로 합니다.

관계에는 세가지 유형이 있습니다.

<table>
  <tr>
    <td><code>One-to-one</code></td>
    <td>기본 테이블의 모든 행에는 외부 테이블에 연결된 행이 하나뿐입니다. 이러한 유형의 관계를 정의하려면 <code>@OneToOne()</code> 데코레이터를 사용하십시오.</td>
  </tr>
  <tr>
    <td><code>One-to-many / Many-to-one</code></td>
    <td>기본 테이블의 모든 행에는 외부 테이블에 하나 이상의 관련 행이 있습니다. 이러한 유형의 관계를 정의하려면 <code>@OneToMany()</code> 및 <code>@ManyToOne()</code> 데코레이터를 사용하십시오.</td>
  </tr>
  <tr>
    <td><code>Many-to-many</code></td>
    <td>기본 테이블의 모든 행에는 외부 테이블에 많은 관련 행이 있고 외부 테이블의 모든 레코드에는 기본 테이블에 많은 관련 행이 있습니다. 이러한 유형의 관계를 정의하려면 <code>@ManyToMany()</code> 데코레이터를 사용하십시오.</td>
  </tr>
</table>

엔티티에서 관계를 정의하려면 해당 **데코레이터**를 사용합니다. 예를 들어 각 `User`가 여러장의 사진을 가질 수 있도록 정의하려면 `@OneToMany()` 데코레이터를 사용합니다.

```typescript
@@filename(user.entity)
import { Entity, Column, PrimaryGeneratedColumn, OneToMany } from 'typeorm';
import { Photo } from '../photos/photo.entity';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column({ default: true })
  isActive: boolean;

  @OneToMany(type => Photo, photo => photo.user)
  photos: Photo[];
}
```

> info **힌트** TypeORM의 관계에 대해 자세히 알아 보려면 [TypeORM 문서](https://typeorm.io/#/relations)를 방문하세요.

#### Auto-load entities

연결 옵션의 `entities` 배열에 엔티티를 수동으로 추가하는 것은 지루할 수 있습니다. 또한 루트 모듈에서 엔티티를 참조하면 애플리케이션 도메인 경계가 깨지고 구현 세부 사항이 애플리케이션의 다른 부분으로 유출됩니다. 이 문제를 해결하기 위해 정적 glob 경로를 사용할 수 있습니다(예: `dist/**/*. entity{{ '{' }} .ts,.js{{ '}' }}`).

그러나 glob 경로는 webpack에서 지원되지 않으므로 모노레포내에서 애플리케이션을 빌드하는 경우 사용할 수 없습니다. 이 문제를 해결하기 위해 대체 솔루션이 제공됩니다. 항목을 자동으로 로드하려면 아래와 같이 구성 개체의 `autoLoadEntities` 속성(`forRoot()` 메서드로 전달됨)을 `true`로 설정합니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      ...
      autoLoadEntities: true,
    }),
  ],
})
export class AppModule {}
```

이 옵션을 지정하면 `forFeature()` 메소드를 통해 등록된 모든 엔티티가 구성 객체의 `entities` 배열에 자동으로 추가됩니다.

> warning **경고** `forFeature()` 메소드를 통해 등록되지 않았지만 엔티티에서만 참조되는 엔티티(관계를 통해)는 `autoLoadEntities` 설정을 통해 포함되지 않습니다.

#### Separating entity definition

데코레이터를 사용하여 모델에서 바로 엔티티와 해당열을 정의할 수 있습니다. 그러나 일부 사람들은 ["엔티티 스키마"](https://typeorm.io/#/separating-entity-definition)를 사용하여 별도의 파일내에 엔터티와 해당열을 정의하는 것을 선호합니다.

```typescript
import { EntitySchema } from 'typeorm';
import { User } from './user.entity';

export const UserSchema = new EntitySchema<User>({
  name: 'User',
  target: User,
  columns: {
    id: {
      type: Number,
      primary: true,
      generated: true,
    },
    firstName: {
      type: String,
    },
    lastName: {
      type: String,
    },
    isActive: {
      type: Boolean,
      default: true,
    },
  },
  relations: {
    photos: {
      type: 'one-to-many',
      target: 'Photo', // the name of the PhotoSchema
    },
  },
});
```

> warning error **경고** `target` 옵션을 제공하는 경우 `name` 옵션 값은 대상 클래스의 이름과 동일해야 합니다.
> `target`을 제공하지 않으면 아무 이름이나 사용할 수 있습니다.

Nest를 사용하면 `Entity`가 예상되는 모든 위치에서 `EntitySchema`인스턴스를 사용할 수 있습니다. 예를 들면 다음과 같습니다.

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UserSchema } from './user.schema';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

@Module({
  imports: [TypeOrmModule.forFeature([UserSchema])],
  providers: [UsersService],
  controllers: [UsersController],
})
export class UsersModule {}
```

#### Transactions

데이터베이스 트랜잭션은 데이터베이스에 대해 데이터베이스 관리 시스템내에서 수행되는 작업 단위를 상징하며 다른 트랜잭션과 독립적으로 일관되고 신뢰할 수 있는 방식으로 처리됩니다. 트랜잭션은 일반적으로 데이터베이스의 모든 변경을 나타냅니다([자세히 알아보기](https://en.wikipedia.org/wiki/Database_transaction)).

[TypeORM 트랜잭션](https://typeorm.io/#/transactions)를 처리하기 위한 다양한 전략이 있습니다. 트랜잭션을 완전히 제어 할 수 있는 `QueryRunner` 클래스를 사용하는 것이 좋습니다.

먼저 일반적인 방법으로 `Connection` 객체를 클래스에 삽입해야 합니다.

```typescript
@Injectable()
export class UsersService {
  constructor(private connection: Connection) {}
}
```

> info **힌트** `Connection` 클래스는 `typeorm` 패키지에서 가져옵니다.

이제 이 개체를 사용하여 트랜잭션을 만들 수 있습니다.

```typescript
async createMany(users: User[]) {
  const queryRunner = this.connection.createQueryRunner();

  await queryRunner.connect();
  await queryRunner.startTransaction();
  try {
    await queryRunner.manager.save(users[0]);
    await queryRunner.manager.save(users[1]);

    await queryRunner.commitTransaction();
  } catch (err) {
    // since we have errors lets rollback the changes we made
    await queryRunner.rollbackTransaction();
  } finally {
    // you need to release a queryRunner which was manually instantiated
    await queryRunner.release();
  }
}
```

> info **힌트** `connection`은 `QueryRunner`를 생성하는 데만 사용됩니다. 그러나 이 클래스를 테스트하려면 전체 `Connection` 객체(여러 메서드를 노출함)를 모의해야(mocking) 합니다. 따라서 헬퍼 팩토리 클래스(예: `QueryRunnerFactory`)를 사용하고 트랜잭션을 유지하는 데 필요한 제한된 메서드 집합으로 인터페이스를 정의하는 것이 좋습니다. 이 기술은 이러한 방법을 매우 간단하게 모의합니다.

또는 `Connection` 객체의 `transaction` 메소드와 함께 콜백 스타일 접근 방식을 사용할 수 있습니다.([자세히 알아보기](https://typeorm.io/#/transactions/creating-and-using-transactions))

```typescript
async createMany(users: User[]) {
  await this.connection.transaction(async manager => {
    await manager.save(users[0]);
    await manager.save(users[1]);
  });
}
```

데코레이터를 사용하여 트랜잭션 (`@Transaction()` 및 `@TransactionManager()`)을 제어하는 것은 권장되지 않습니다.

<app-banner-shop></app-banner-shop>

#### Subscribers

TypeORM [구독자 subscribers](https://typeorm.io/#/listeners-and-subscribers/what-is-a-subscriber)를 사용하면 특정 엔티티 이벤트를 수신할 수 있습니다.

```typescript
import {
  Connection,
  EntitySubscriberInterface,
  EventSubscriber,
  InsertEvent,
} from 'typeorm';
import { User } from './user.entity';

@EventSubscriber()
export class UserSubscriber implements EntitySubscriberInterface<User> {
  constructor(connection: Connection) {
    connection.subscribers.push(this);
  }

  listenTo() {
    return User;
  }

  beforeInsert(event: InsertEvent<User>) {
    console.log(`BEFORE USER INSERTED: `, event.entity);
  }
}
```

> error **경고** 이벤트 구독자는 [요청 범위](/fundamentals/injection-scopes)일 수 없습니다.

이제 `providers` 배열에 `UserSubscriber` 클래스를 추가합니다.

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { User } from './user.entity';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { UserSubscriber } from './user.subscriber';

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  providers: [UsersService, UserSubscriber],
  controllers: [UsersController],
})
export class UsersModule {}
```

> info **힌트** [여기](https://typeorm.io/#/listeners-and-subscribers/what-is-a-subscriber)에서 항목 구독자에 대해 자세히 알아보세요.

#### Migrations

[마이그레이션](https://typeorm.io/#/migrations)은 데이터베이스 스키마를 점진적으로 업데이트하여 데이터베이스의 기존 데이터를 보존하면서 애플리케이션의 데이터 모델과 동기화 상태를 유지하는 방법을 제공합니다. 마이그레이션을 생성, 실행 및 되돌리기 위해 TypeORM은 전용 [CLI](https://typeorm.io/#/migrations/creating-a-new-migration)를 제공합니다.

마이그레이션 클래스는 Nest 애플리케이션 소스 코드와 별개입니다. 라이프 사이클은 TypeORM CLI에 의해 유지됩니다. 따라서 마이그레이션과 함께 종속성 주입 및 기타 Nest 특정 기능을 활용할 수 없습니다. 마이그레이션에 대해 자세히 알아 보려면 [TypeORM 문서](https://typeorm.io/#/migrations/creating-a-new-migration)의 가이드를 따르세요.

#### Multiple databases

일부 프로젝트에는 여러 데이터베이스 연결이 필요합니다. 이것은 또한 이 모듈을 통해 달성할 수 있습니다. 여러 연결로 작업하려면 먼저 연결을 만듭니다. 이 경우 연결 이름 지정은 **필수**가 됩니다.

자체 데이터베이스에 `Album` 항목이 저장되어 있다고 가정합니다.

```typescript
const defaultOptions = {
  type: 'postgres',
  port: 5432,
  username: 'user',
  password: 'password',
  database: 'db',
  synchronize: true,
};

@Module({
  imports: [
    TypeOrmModule.forRoot({
      ...defaultOptions,
      host: 'user_db_host',
      entities: [User],
    }),
    TypeOrmModule.forRoot({
      ...defaultOptions,
      name: 'albumsConnection',
      host: 'album_db_host',
      entities: [Album],
    }),
  ],
})
export class AppModule {}
```

> warning **주의 알림** 연결에 `name`을 설정하지 않으면 연결 이름이 `기본값 default`으로 설정됩니다. 이름이 없거나 이름이 같은 연결이 여러개 있으면 안됩니다. 그렇지 않으면 연결이 재정의됩니다.

이 시점에서 `User` 및 `Album` 엔티티가 자체 연결에 등록되어 있습니다. 이 설정으로 `TypeOrmModule.forFeature()` 메소드와 `@InjectRepository()` 데코레이터에게 어떤 연결을 사용해야하는지 알려 주어야합니다. 연결 이름을 전달하지 않으면 `default` 연결이 사용됩니다.

```typescript
@Module({
  imports: [
    TypeOrmModule.forFeature([User]),
    TypeOrmModule.forFeature([Album], 'albumsConnection'),
  ],
})
export class AppModule {}
```

주어진 연결에 대해 `Connection` 또는 `EntityManager`를 삽입할 수도 있습니다.

```typescript
@Injectable()
export class AlbumsService {
  constructor(
    @InjectConnection('albumsConnection')
    private connection: Connection,
    @InjectEntityManager('albumsConnection')
    private entityManager: EntityManager,
  ) {}
}
```

프로바이더에게 `Connection`을 삽입하는 것도 가능합니다.

```typescript
@Module({
  providers: [
    {
      provide: AlbumsService,
      useFactory: (albumsConnection: Connection) => {
        return new AlbumsService(albumsConnection);
      },
      inject: [getConnectionToken('albumsConnection')],
    },
  ],
})
export class AlbumsModule {}
```

#### Testing

애플리케이션의 단위 테스트에 관해서는 일반적으로 데이터베이스 연결을 피하고 테스트 스위트를 독립적으로 유지하고 실행 프로세스를 가능한 한 빨리 유지하려고 합니다. 그러나 우리의 클래스는 연결 인스턴스에서 가져온 저장소에 따라 달라질 수 있습니다. 어떻게 처리합니까? 해결책은 모의 저장소를 만드는 것입니다. 이를 달성하기 위해 [커스텀 프로바이더](/fundamentals/custom-providers)를 설정했습니다. 등록된 각 저장소는 자동으로 `<EntityName>Repository` 토큰으로 표시됩니다. 여기서 `EntityName`은 엔티티 클래스의 이름입니다.

`@nestjs/typeorm` 패키지는 주어진 엔티티를 기반으로 준비된 토큰을 반환하는 `getRepositoryToken()` 함수를 노출합니다.

```typescript
@Module({
  providers: [
    UsersService,
    {
      provide: getRepositoryToken(User),
      useValue: mockRepository,
    },
  ],
})
export class UsersModule {}
```

이제 대체 `mockRepository`가 `UsersRepository`로 사용됩니다. 클래스가 `@InjectRepository()` 데코레이터를 사용하여 `UsersRepository`를 요청할 때마다 Nest는 등록된 `mockRepository` 객체를 사용합니다.

#### Custom repository

TypeORM은 **사용자 지정 저장소**라는 기능을 제공합니다. 커스텀 리포지토리를 사용하면 기본 리포지토리 클래스를 확장하고 몇 가지 특수 메서드를 사용하여 강화할 수 있습니다. 이 기능에 대해 자세히 알아 보려면 [이 페이지](https://typeorm.io/#/custom-repository)를 방문하세요.

커스텀 리포지토리를 만들려면 `@EntityRepository()` 데코레이터를 사용하고 `Repository` 클래스를 확장합니다.

```typescript
@EntityRepository(Author)
export class AuthorRepository extends Repository<Author> {}
```

> info **힌트** `@EntityRepository()`와 `Repository`는 모두 `typeorm` 패키지에서 가져옵니다.

클래스가 생성되면 다음 단계는 인스턴스화 책임을 Nest에 위임하는 것입니다. 이를 위해 `AuthorRepository` 클래스를 `TypeOrm.forFeature()` 메소드에 전달해야 합니다.

```typescript
@Module({
  imports: [TypeOrmModule.forFeature([AuthorRepository])],
  controller: [AuthorController],
  providers: [AuthorService],
})
export class AuthorModule {}
```

그런 다음 다음 구성을 사용하여 저장소를 삽입하십시오.

```typescript
@Injectable()
export class AuthorService {
  constructor(private authorRepository: AuthorRepository) {}
}
```

#### Async configuration

저장소 모듈 옵션을 정적으로 전달하는 대신 비동기 적으로 전달할 수 있습니다. 이 경우 비동기 구성을 처리하는 여러 방법을 제공하는 `forRootAsync()` 메소드를 사용하십시오.

한가지 접근 방식은 팩토리 함수를 사용하는 것입니다.

```typescript
TypeOrmModule.forRootAsync({
  useFactory: () => ({
    type: 'mysql',
    host: 'localhost',
    port: 3306,
    username: 'root',
    password: 'root',
    database: 'test',
    entities: [__dirname + '/**/*.entity{.ts,.js}'],
    synchronize: true,
  }),
});
```

우리 공장은 다른 [비동기 프로바이더](/fundamentals/async-providers)처럼 작동합니다 (예: `async`일 수 있으며 `inject`를 통해 종속성을 삽입할 수 있음).

```typescript
TypeOrmModule.forRootAsync({
  imports: [ConfigModule],
  useFactory: (configService: ConfigService) => ({
    type: 'mysql',
    host: configService.get('HOST'),
    port: +configService.get<number>('PORT'),
    username: configService.get('USERNAME'),
    password: configService.get('PASSWORD'),
    database: configService.get('DATABASE'),
    entities: [__dirname + '/**/*.entity{.ts,.js}'],
    synchronize: true,
  }),
  inject: [ConfigService],
});
```

또는 `useClass` 구문을 사용할 수 있습니다.

```typescript
TypeOrmModule.forRootAsync({
  useClass: TypeOrmConfigService,
});
```

위의 구성은 `TypeOrmModule` 내에서 `TypeOrmConfigService`를 인스턴스화하고 `createTypeOrmOptions()`를 호출하여 옵션 객체를 제공하는 데 사용합니다. 이는 `TypeOrmConfigService`가 아래와 같이 `TypeOrmOptionsFactory` 인터페이스를 구현해야 함을 의미합니다.

```typescript
@Injectable()
class TypeOrmConfigService implements TypeOrmOptionsFactory {
  createTypeOrmOptions(): TypeOrmModuleOptions {
    return {
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'root',
      database: 'test',
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: true,
    };
  }
}
```

`TypeOrmModule` 내에서 `TypeOrmConfigService` 생성을 방지하고 다른 모듈에서 가져온 프로바이더를 사용하려면 `useExisting` 구문을 사용할 수 있습니다.

```typescript
TypeOrmModule.forRootAsync({
  imports: [ConfigModule],
  useExisting: ConfigService,
});
```

이 구성은 `useClass`와 동일하게 작동하지만 한가지 중요한 차이점이 있습니다. `TypeOrmModule`은 새 모듈을 인스턴스화하는 대신 기존 `ConfigService`를 재사용하기 위해 가져온 모듈을 조회합니다.

> info **힌트** `name` 속성이 `useFactory`, `useClass` 또는 `useValue` 속성과 동일한 수준에서 정의되었는지 확인하세요. 이렇게하면 Nest가 적절한 인젝션 토큰 아래에 연결을 올바르게 등록할 수 있습니다.

#### Example

작동하는 예제는 [여기](https://github.com/nestjs/nest/tree/master/sample/05-sql-typeorm)에서 확인할 수 있습니다.

<app-banner-enterprise></app-banner-enterprise>

### Sequelize Integration

TypeORM 사용에 대한 대안은 `@nestjs/sequelize` 패키지와 함께 [Sequelize](https://sequelize.org/) ORM을 사용하는 것입니다. 또한 엔티티를 선언적으로 정의하기 위한 추가 데코레이터 세트를 제공하는 [sequelize-typescript](https://github.com/RobinBuschmann/sequelize-typescript) 패키지를 활용합니다.

사용을 시작하려면 먼저 필요한 종속성을 설치합니다. 이 장에서는 널리 사용되는 [MySQL](https://www.mysql.com/) 관계형 DBMS를 사용하는 방법을 시연하지만 Sequelize는 PostgreSQL, MySQL, Microsoft SQL Server, SQLite 및 MariaDB와 같은 많은 관계형 데이터베이스를 지원합니다. 이 장에서 살펴 보는 절차는 Sequelize에서 지원하는 모든 데이터베이스에 대해 동일합니다. 선택한 데이터베이스에 연결된 클라이언트 API 라이브러리를 설치하기만하면 됩니다.

```bash
$ npm install --save @nestjs/sequelize sequelize sequelize-typescript mysql2
$ npm install --save-dev @types/sequelize
```

설치 프로세스가 완료되면 `SequelizeModule`을 루트 `AppModule`로 가져올 수 있습니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { SequelizeModule } from '@nestjs/sequelize';

@Module({
  imports: [
    SequelizeModule.forRoot({
      dialect: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'root',
      database: 'test',
      models: [],
    }),
  ],
})
export class AppModule {}
```

`forRoot()` 메서드는 Sequelize 생성자에 의해 노출된 모든 구성 속성을 지원합니다 ([자세히 알아보기](https://sequelize.org/v5/manual/getting-started.html#setting-up-a-connection)). 또한 아래에 설명된 몇가지 추가 구성 속성이 있습니다.

<table>
  <tr>
    <td><code>retryAttempts</code></td>
    <td>데이터베이스 연결 시도 횟수 (기본값: <code>10</code>)</td>
  </tr>
  <tr>
    <td><code>retryDelay</code></td>
    <td>연결 재시도 간격 (밀리 초) (기본값: <code>3000</code>)</td>
  </tr>
  <tr>
    <td><code>autoLoadModels</code></td>
    <td><code>true</code>이면 모델이 자동으로 로드됩니다.(기본값: <code>false</code>)</td>
  </tr>
  <tr>
    <td><code>keepConnectionAlive</code></td>
    <td><code>true</code>인 경우 애플리케이션 종료시 연결이 닫히지 않습니다.(기본값: <code>false</code>)</td>
  </tr>
  <tr>
    <td><code>synchronize</code></td>
    <td><code>true</code>이면 자동 로드된 모델이 동기화됩니다.(기본값: <code>true</code>)</td>
  </tr>
</table>

이 작업이 완료되면 `Sequelize` 객체를 사용하여 전체 프로젝트에 삽입할 수 있습니다.(모듈을 가져올 필요없이) 예:

```typescript
@@filename(app.service)
import { Injectable } from '@nestjs/common';
import { Sequelize } from 'sequelize-typescript';

@Injectable()
export class AppService {
  constructor(private sequelize: Sequelize) {}
}
@@switch
import { Injectable } from '@nestjs/common';
import { Sequelize } from 'sequelize-typescript';

@Dependencies(Sequelize)
@Injectable()
export class AppService {
  constructor(sequelize) {
    this.sequelize = sequelize;
  }
}
```

#### Models

Sequelize는 Active Record 패턴을 구현합니다. 이 패턴에서는 모델 클래스를 직접 사용하여 데이터베이스와 상호 작용합니다. 예제를 계속하려면 하나 이상의 모델이 필요합니다. `User` 모델을 정의해 보겠습니다.

```typescript
@@filename(user.model)
import { Column, Model, Table } from 'sequelize-typescript';

@Table
export class User extends Model {
  @Column
  firstName: string;

  @Column
  lastName: string;

  @Column({ defaultValue: true })
  isActive: boolean;
}
```

> info **힌트** 사용 가능한 데코레이터에 대한 자세한 내용은 [여기](https://github.com/RobinBuschmann/sequelize-typescript#column)를 참조하세요.

`User` 모델 파일은 `users` 디렉토리에 있습니다. 이 디렉토리는 `UsersModule`과 관련된 모든 파일을 포함합니다. 모델 파일을 보관할 위치를 결정할 수 있지만 해당 모듈 디렉토리의 **도메인** 근처에 파일을 만드는 것이 좋습니다.

`User` 모델 사용을 시작하려면 모듈 `forRoot()` 메소드 옵션의 `models` 배열에 삽입하여 Sequelize에 알려야 합니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { SequelizeModule } from '@nestjs/sequelize';
import { User } from './users/user.model';

@Module({
  imports: [
    SequelizeModule.forRoot({
      dialect: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'root',
      database: 'test',
      models: [User],
    }),
  ],
})
export class AppModule {}
```

다음으로 `UsersModule`을 살펴 보겠습니다.

```typescript
@@filename(users.module)
import { Module } from '@nestjs/common';
import { SequelizeModule } from '@nestjs/sequelize';
import { User } from './user.model';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

@Module({
  imports: [SequelizeModule.forFeature([User])],
  providers: [UsersService],
  controllers: [UsersController],
})
export class UsersModule {}
```

이 모듈은 `forFeature()` 메소드를 사용하여 현재 범위에 등록된 모델을 정의합니다. 그런 다음 `@InjectModel()` 데코레이터를 사용하여 `UserModel`을 `UsersService`에 삽입할 수 있습니다.

```typescript
@@filename(users.service)
import { Injectable } from '@nestjs/common';
import { InjectModel } from '@nestjs/sequelize';
import { User } from './user.model';

@Injectable()
export class UsersService {
  constructor(
    @InjectModel(User)
    private userModel: typeof User,
  ) {}

  async findAll(): Promise<User[]> {
    return this.userModel.findAll();
  }

  findOne(id: string): Promise<User> {
    return this.userModel.findOne({
      where: {
        id,
      },
    });
  }

  async remove(id: string): Promise<void> {
    const user = await this.findOne(id);
    await user.destroy();
  }
}
@@switch
import { Injectable, Dependencies } from '@nestjs/common';
import { getModelToken } from '@nestjs/sequelize';
import { User } from './user.model';

@Injectable()
@Dependencies(getModelToken(User))
export class UsersService {
  constructor(usersRepository) {
    this.usersRepository = usersRepository;
  }

  async findAll() {
    return this.userModel.findAll();
  }

  findOne(id) {
    return this.userModel.findOne({
      where: {
        id,
      },
    });
  }

  async remove(id) {
    const user = await this.findOne(id);
    await user.destroy();
  }
}
```

> warning **알림** `UsersModule`을 루트 `AppModule`로 가져오는 것을 잊지 마십시오.

`SequelizeModule.forFeature`를 가져오는 모듈 외부의 리포지토리를 사용하려면 생성된 프로바이더를 다시 내보내야합니다.
다음과 같이 전체 모듈을 내보내면 됩니다.

```typescript
@@filename(users.module)
import { Module } from '@nestjs/common';
import { SequelizeModule } from '@nestjs/sequelize';
import { User } from './user.entity';

@Module({
  imports: [SequelizeModule.forFeature([User])],
  exports: [SequelizeModule]
})
export class UsersModule {}
```

이제 `UserHttpModule`에서 `UsersModule`을 가져오면 후자의 모듈 공급자에서 `@InjectModel(User)`를 사용할 수 있습니다.

```typescript
@@filename(users-http.module)
import { Module } from '@nestjs/common';
import { UsersModule } from './users.module';
import { UsersService } from './users.service';
import { UsersController } from './users.controller';

@Module({
  imports: [UsersModule],
  providers: [UsersService],
  controllers: [UsersController]
})
export class UserHttpModule {}
```

### Relations

관계는 두개 이상의 테이블간에 설정된 연결입니다. 관계는 종종 기본 및 외래 키를 포함하는 각 테이블의 공통 필드를 기반으로 합니다.

관계에는 세 가지 유형이 있습니다.

<table>
  <tr>
    <td><code>One-to-one</code></td>
    <td>기본 테이블의 모든 행에는 외부 테이블에 하나의 연관된 행만 있습니다.</td>
  </tr>
  <tr>
    <td><code>One-to-many / Many-to-one</code></td>
    <td>기본 테이블의 모든 행에는 외부 테이블에 하나 이상의 관련 행이 있습니다.</td>
  </tr>
  <tr>
    <td><code>Many-to-many</code></td>
    <td>기본 테이블의 모든 행에는 외부 테이블에 많은 관련 행이 있고 외부 테이블의 모든 레코드에는 기본 테이블에 많은 관련 행이 있습니다.</td>
  </tr>
</table>

엔티티에서 관계를 정의하려면 해당 **데코레이터**를 사용합니다. 예를 들어 각 `User`가 여러장의 사진을 가질 수 있도록 정의하려면 `@HasMany()` 데코레이터를 사용합니다.

```typescript
@@filename(user.entity)
import { Column, Model, Table, HasMany } from 'sequelize-typescript';
import { Photo } from '../photos/photo.model';

@Table
export class User extends Model {
  @Column
  firstName: string;

  @Column
  lastName: string;

  @Column({ defaultValue: true })
  isActive: boolean;

  @HasMany(() => Photo)
  photos: Photo[];
}
```

> info **힌트** Sequelize의 연결에 대한 자세한 내용은 [이](https://github.com/RobinBuschmann/sequelize-typescript#model-association)장을 참조하세요.

#### Auto-load models

연결 옵션의 `models` 배열에 모델을 수동으로 추가하는 것은 지루할 수 있습니다. 또한 루트 모듈에서 모델을 참조하면 애플리케이션 도메인 경계가 깨지고 구현 세부 정보가 애플리케이션의 다른 부분으로 유출됩니다. 이 문제를 해결하려면 아래와 같이 구성 객체의 `autoLoadModels` 및 `synchronize` 속성 (`forRoot()` 메서드로 전달됨)을 모두 `true`로 설정하여 모델을 자동으로 로드합니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { SequelizeModule } from '@nestjs/sequelize';

@Module({
  imports: [
    SequelizeModule.forRoot({
      ...
      autoLoadModels: true,
      synchronize: true,
    }),
  ],
})
export class AppModule {}
```

이 옵션을 지정하면 `forFeature()` 메소드를 통해 등록된 모든 모델이 구성 객체의 `models` 배열에 자동으로 추가됩니다.

> warning **경고** `forFeature()` 메서드를 통해 등록되지 않았지만 연결을 통해 모델에서만 참조되는 모델은 포함되지 않습니다.

#### Transactions

데이터베이스 트랜잭션은 데이터베이스에 대해 데이터베이스 관리 시스템 내에서 수행되는 작업 단위를 상징하며 다른 트랜잭션과 독립적으로 일관되고 신뢰할 수 있는 방식으로 처리됩니다. 트랜잭션은 일반적으로 데이터베이스의 모든 변경을 나타냅니다 ([자세히 알아보기](https://en.wikipedia.org/wiki/Database_transaction)).

[Sequelize 트랜잭션](https://sequelize.org/v5/manual/transactions.html)을 처리하기위한 다양한 전략이 있습니다. 다음은 관리 트랜잭션(자동 콜백)의 샘플 구현입니다.

먼저 일반적인 방법으로 `Sequelize` 객체를 클래스에 삽입해야합니다.

```typescript
@Injectable()
export class UsersService {
  constructor(private sequelize: Sequelize) {}
}
```

> info **힌트** `Sequelize` 클래스는 `sequelize-typescript` 패키지에서 가져옵니다.

이제이 객체를 사용하여 트랜잭션을 만들 수 있습니다.

```typescript
async createMany() {
  try {
    await this.sequelize.transaction(async t => {
      const transactionHost = { transaction: t };

      await this.userModel.create(
          { firstName: 'Abraham', lastName: 'Lincoln' },
          transactionHost,
      );
      await this.userModel.create(
          { firstName: 'John', lastName: 'Boothe' },
          transactionHost,
      );
    });
  } catch (err) {
    // Transaction has been rolled back
    // err is whatever rejected the promise chain returned to the transaction callback
  }
}
```

> info **힌트** `Sequelize` 인스턴스는 트랜잭션을 시작하는 데만 사용됩니다. 그러나 이 클래스를 테스트하려면 전체 `Sequelize` 객체(여러 메서드를 노출함)를 모의해야합니다. 따라서 헬퍼 팩토리 클래스(예: `TransactionRunner`)를 사용하고 트랜잭션을 유지하는 데 필요한 제한된 메서드 집합으로 인터페이스를 정의하는 것이 좋습니다. 이 기술은 이러한 방법을 매우 간단하게 모의합니다.

#### Migrations

[마이그레이션](https://sequelize.org/v5/manual/migrations.html)은 데이터베이스 스키마를 점진적으로 업데이트하여 데이터베이스의 기존 데이터를 유지하면서 애플리케이션의 데이터 모델과 동기화 상태를 유지하는 방법을 제공합니다. 마이그레이션을 생성, 실행 및 되돌리기 위해 Sequelize는 전용 [CLI](https://sequelize.org/v5/manual/migrations.html#the-cli)를 제공합니다.

마이그레이션 클래스는 Nest 애플리케이션 소스 코드와 별개입니다. 수명주기는 Sequelize CLI에 의해 유지됩니다. 따라서 마이그레이션과 함께 종속성 주입 및 기타 Nest 특정 기능을 활용할 수 없습니다. 이전에 대해 자세히 알아 보려면 [Sequelize 문서](https://sequelize.org/v5/manual/migrations.html#the-cli)의 가이드를 따르세요.

<app-banner-courses></app-banner-courses>

#### Multiple databases

일부 프로젝트에는 여러 데이터베이스 연결이 필요합니다. 이것은 또한 이 모듈을 통해 달성할 수 있습니다. 여러 연결로 작업하려면 먼저 연결을 만듭니다. 이 경우 연결 이름 지정은 **필수**가 됩니다.

자체 데이터베이스에 `Album` 항목이 저장되어 있다고 가정합니다.

```typescript
const defaultOptions = {
  dialect: 'postgres',
  port: 5432,
  username: 'user',
  password: 'password',
  database: 'db',
  synchronize: true,
};

@Module({
  imports: [
    SequelizeModule.forRoot({
      ...defaultOptions,
      host: 'user_db_host',
      models: [User],
    }),
    SequelizeModule.forRoot({
      ...defaultOptions,
      name: 'albumsConnection',
      host: 'album_db_host',
      models: [Album],
    }),
  ],
})
export class AppModule {}
```

> warning **알림** 연결에 `name`을 설정하지 않으면 연결 이름이 `기본값 default`으로 설정됩니다. 이름이 없거나 이름이 같은 연결이 여러개 있으면 안됩니다. 그렇지 않으면 연결이 재정의됩니다.

이 시점에서 `User` 및 `Album` 모델이 자체 연결로 등록되어 있습니다. 이 설정으로 `SequelizeModule.forFeature()` 메소드와 `@InjectModel()` 데코레이터에게 어떤 연결을 사용해야 하는지 알려주어야 합니다. 연결 이름을 전달하지 않으면 `default` 연결이 사용됩니다.

```typescript
@Module({
  imports: [
    SequelizeModule.forFeature([User]),
    SequelizeModule.forFeature([Album], 'albumsConnection'),
  ],
})
export class AppModule {}
```

주어진 연결에 `Sequelize` 인스턴스를 삽입할 수도 있습니다.

```typescript
@Injectable()
export class AlbumsService {
  constructor(
    @InjectConnection('albumsConnection')
    private sequelize: Sequelize,
  ) {}
}
```

`Sequelize` 인스턴스를 프로바이더에 삽입할 수도 있습니다.

```typescript
@Module({
  providers: [
    {
      provide: AlbumsService,
      useFactory: (albumsSequelize: Sequelize) => {
        return new AlbumsService(albumsSequelize);
      },
      inject: [getConnectionToken('albumsConnection')],
    },
  ],
})
export class AlbumsModule {}
```

#### Testing

애플리케이션의 단위 테스트에 관해서는 일반적으로 데이터베이스 연결을 피하고 테스트 스위트를 독립적으로 유지하고 실행 프로세스를 가능한 한 빨리 유지하려고 합니다. 그러나 우리의 클래스는 연결 인스턴스에서 가져온 모델에 따라 달라질 수 있습니다. 어떻게 처리합니까? 해결책은 모의 모델을 만드는 것입니다. 이를 달성하기 위해 [커스텀 프로바이더](/fundamentals/custom-providers)를 설정했습니다. 등록된 각 모델은 자동으로 `<ModelName>Model` 토큰으로 표시됩니다. 여기서 `ModelName`은 모델 클래스의 이름입니다.

`@estjs/sequelize` 패키지는 주어진 모델을 기반으로 준비된 토큰을 반환하는 `getModelToken ()` 함수를 노출합니다.

```typescript
@Module({
  providers: [
    UsersService,
    {
      provide: getModelToken(User),
      useValue: mockModel,
    },
  ],
})
export class UsersModule {}
```

이제 대체 `mockModel`이 `UserModel`로 사용됩니다. 클래스가 `@InjectModel()` 데코레이터를 사용하여 `UserModel`을 요청할 때마다 Nest는 등록된 `mockModel` 객체를 사용합니다.

#### Async configuration

`SequelizeModule` 옵션을 정적으로 전달하는 대신 비동기 적으로 전달할 수 있습니다. 이 경우 비동기 구성을 처리하는 여러 방법을 제공하는 `forRootAsync()` 메소드를 사용하십시오.

한가지 접근 방식은 팩토리 함수를 사용하는 것입니다.

```typescript
SequelizeModule.forRootAsync({
  useFactory: () => ({
    dialect: 'mysql',
    host: 'localhost',
    port: 3306,
    username: 'root',
    password: 'root',
    database: 'test',
    models: [],
  }),
});
```

우리 팩토리는 다른 [비동기 프로바이더](/fundamentals/async-providers)처럼 작동합니다 (예: `async`일 수 있으며 `inject`를 통해 종속성을 삽입할 수 있음).

```typescript
SequelizeModule.forRootAsync({
  imports: [ConfigModule],
  useFactory: (configService: ConfigService) => ({
    dialect: 'mysql',
    host: configService.get('HOST'),
    port: +configService.get('PORT'),
    username: configService.get('USERNAME'),
    password: configService.get('PASSWORD'),
    database: configService.get('DATABASE'),
    models: [],
  }),
  inject: [ConfigService],
});
```

또는 `useClass` 구문을 사용할 수 있습니다.

```typescript
SequelizeModule.forRootAsync({
  useClass: SequelizeConfigService,
});
```

위의  `SequelizeModule` 내에서 `SequelizeConfigService`를 인스턴스화하고 `createSequelizeOptions()`를 호출하여 옵션 객체를 제공하는 데 사용합니다. 이는 `SequelizeConfigService`가 아래와 같이 `SequelizeOptionsFactory` 인터페이스를 구현해야 함을 의미합니다.

```typescript
@Injectable()
class SequelizeConfigService implements SequelizeOptionsFactory {
  createSequelizeOptions(): SequelizeModuleOptions {
    return {
      dialect: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'root',
      database: 'test',
      models: [],
    };
  }
}
```

`Sequelize Module` 내에서 `Sequelize Config Service` 생성을 방지하고 다른 모듈에서 가져온 프로바이더를 사용하려면 `useExisting` 구문을 사용할 수 있습니다.

```typescript
SequelizeModule.forRootAsync({
  imports: [ConfigModule],
  useExisting: ConfigService,
});
```

이 구성은 `useClass`와 동일하게 작동하지만 한가지 중요한 차이점이 있습니다. `SequelizeModule`은 새 모듈을 인스턴스화하는 대신 기존 `ConfigService`를 재사용하기 위해 가져온 모듈을 조회합니다.

#### Example

작동하는 예는 [여기](https://github.com/nestjs/nest/tree/master/sample/07-sequelize)에서 확인할 수 있습니다.
