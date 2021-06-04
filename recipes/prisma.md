### Prisma

[Prisma](https://www.prisma.io)는 Node.js 및 TypeScript를 위한 x[오픈 소스](https://github.com/prisma/prisma) ORM입니다. 일반 SQL을 작성하거나 SQL 쿼리 빌더(예: [knex.js](https://knexjs.org/)) 또는 ORM (예 : [TypeORM](https://typeorm.io/) 및 [Sequelize](https://sequelize.org/))과 같은 다른 데이터베이스 액세스 도구를 사용하는 **대안**으로 사용됩니다. Prisma는 현재 PostgreSQL, MySQL, SQL Server 및 SQLite를 지원합니다.

Prisma는 일반 JavaScript와 함께 사용할 수 있지만 TypeScript를 수용하고 TypeScript 에코 시스템의 다른 ORM을 보장하는 수준을 넘어서는 타입 안전성 수준을 제공합니다. Prisma와 TypeORM의 형식 안전성 보장에 대한 심층 비교는 [여기](https://www.prisma.io/docs/concepts/more/comparisons/prisma-and-typeorm#type-safety)에서 확인할 수 있습니다. .

> info **참고** Prisma 작동 방식에 대한 간략한 개요를 보려면 [빠른 시작](https://www.prisma.io/docs/getting-started/quickstart)을 따르거나 [문서](https://www.prisma.io/docs/)의 [소개](https://www.prisma.io/docs/understand-prisma/introduction)를 읽으십시오. [`prisma-examples`](https://github.com/prisma/prisma-examples/) repo에 [REST](https://github.com/prisma/prisma-examples/tree/latest/typescript/rest-nestjs) 및 [GraphQL](https://github.com/prisma/prisma-examples/tree/latest/typescript/graphql-nestjs)에 대한 바로 실행 가능한 예제가 있습니다.

#### Getting started

이 레시피에서는 NestJS 및 Prisma를 처음부터 시작하는 방법을 배웁니다. 데이터베이스에서 데이터를 읽고 쓸 수 있는 REST API로 샘플 NestJS 애플리케이션을 빌드할 것입니다.

이 가이드에서는 [SQLite](https://sqlite.org/) 데이터베이스를 사용하여 데이터베이스 서버를 설정하는 오버헤드를 줄일 수 있습니다. PostgreSQL 또는 MySQL을 사용하는 경우에도 이 가이드를 계속 따를 수 있습니다. 올바른 위치에서 이러한 데이터베이스를 사용하기 위한 추가 지침을 얻을 수 있습니다.

> info **참고** 이미 기존 프로젝트가 있고 Prisma로 마이그레이션을 고려하는 경우 [기존 프로젝트에 Prisma 추가](https://www.prisma.io/docs/getting-started/setup-prisma/add-to-existing-project-typescript-postgres) 가이드를 따를 수 있습니다. TypeORM에서 마이그레이션하는 경우 [TypeORM에서 Prisma로 마이그레이션](https://www.prisma.io/docs/guides/migrate-to-prisma/migrate-from-typeorm) 가이드를 읽을 수 있습니다.


#### Create your NestJS project

시작하려면 NestJS CLI를 설치하고 다음 명령어로 앱 스켈레톤을 만드세요.

```bash
$ npm install -g @nestjs/cli
$ nest new hello-prisma
```

이 명령으로 생성된 프로젝트 파일에 대한 자세한 내용은 [첫번째 단계](/first-steps) 페이지를 참조하세요. 이제 `npm start`를 실행하여 애플리케이션을 시작할 수도 있습니다. `http://localhost:3000/`에서 실행되는 REST API는 현재 `src/app.controller.ts`에 구현된 단일 라우트를 제공합니다. 이 가이드의 과정에서 _users_ 및 _posts_ 에 대한 데이터를 저장하고 검색하기 위한 추가 라우트를 구현합니다.

#### Set up Prisma

프로젝트에서 Prisma CLI를 개발 종속성으로 설치하여 시작하십시오.


```bash
$ cd hello-prisma
$ npm install prisma --save-dev
```

다음 단계에서는 [Prisma CLI](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-cli)를 활용합니다. 모범 사례로 `npx`를 접두사로 붙여서 CLI를 로컬에서 호출하는 것이 좋습니다.

```bash
$ npx prisma
```

<details><summary>Yarn을 사용하는 경우 확장</summary>

Yarn을 사용하는 경우 다음과 같이 Prisma CLI를 설치할 수 있습니다.

```bash
$ yarn add prisma --dev
```

일단 설치되면 `yarn` 접두사를 붙여 호출할 수 있습니다.

```bash
$ yarn prisma
```

</details>

이제 Prisma CLI의 `init` 명령을 사용하여 초기 Prisma 설정을 만듭니다.

```bash
$ npx prisma init
```

이 명령은 다음 내용으로 새 `prisma` 디렉토리를 만듭니다.

- `schema.prisma`: 데이터베이스 연결을 지정하고 데이터베이스 스키마를 포함합니다.
- `.env`: [dotenv](https://github.com/motdotla/dotenv) 파일, 일반적으로 환경 변수 그룹에 데이터베이스 자격증명을 저장하는데 사용됩니다.

#### Set the database connection

데이터베이스 연결은 `schema.prisma` 파일의 `datasource` 블록에서 구성됩니다. 기본적으로 `postgresql`로 설정되어 있지만 이 가이드에서는 SQLite 데이터베이스를 사용하고 있으므로 `datasource`블록의 `provider`필드를 `sqlite`로 조정해야합니다.

```groovy
datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}
```

이제 `.env`를 열고 `DATABASE_URL` 환경 변수를 다음과 같이 조정합니다.

```bash
DATABASE_URL="file:./dev.db"
```

SQLite 데이터베이스는 간단한 파일입니다. SQLite 데이터베이스를 사용하는데 서버가 필요하지 않습니다. 따라서 _host_ 및 _port_ 를 사용하여 연결 URL을 구성하는 대신 이 경우 `dev.db`라고 하는 로컬 파일을 가리킬 수 있습니다. 이 파일은 다음 단계에서 생성됩니다.

<details><summary>PostgreSQL 또는 MySQL을 사용하는 경우 확장</summary>

PostgreSQL 및 MySQL을 사용하는 경우 _database server_ 를 가리키도록 연결 URL을 구성해야 합니다. 필요한 연결 URL 형식에 대한 자세한 내용은 [여기](https://www.prisma.io/docs/reference/database-reference/connection-urls)를 참조하세요.

**PostgreSQL**

PostgreSQL을 사용하는 경우 `schema.prisma` 및 `.env` 파일을 다음과 같이 조정해야합니다.

**`schema.prisma`**

```groovy
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}
```

**`.env`**

```bash
DATABASE_URL="postgresql://USER:PASSWORD@HOST:PORT/DATABASE?schema=SCHEMA"
```

모두 대문자로된 자리표시자를 데이터베이스 자격증명으로 바꿉니다. `SCHEMA` 자리표시자에 무엇을 제공해야 할지 확실하지 않은 경우 기본값인 `public` 일 가능성이 높습니다.

```bash
DATABASE_URL="postgresql://USER:PASSWORD@HOST:PORT/DATABASE?schema=public"
```

PostgreSQL 데이터베이스를 설정하는 방법을 배우려면 [Heroku에서 무료 PostgreSQL 데이터베이스 설정](https://dev.to/prisma/how-to-setup-a-free-postgresql-database-on-heroku-1dc1)에 대한 이 가이드를 따르세요.

**MySQL**

MySQL을 사용하는 경우 다음과 같이 `schema.prisma` 및 `.env` 파일을 조정해야합니다.

**`schema.prisma`**

```groovy
datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}
```

**`.env`**

```bash
DATABASE_URL="mysql://USER:PASSWORD@HOST:PORT/DATABASE"
```

모두 대문자로된 자리표시자를 데이터베이스 자격증명으로 바꿉니다.

</details>

#### Create two database tables with Prisma Migrate

이 섹션에서는 [Prisma Migrate](https://www.prisma.io/docs/concepts/components/prisma-migrate)를 사용하여 데이터베이스에 두개의 새 테이블을 생성합니다. Prisma Migrate는 Prisma 스키마의 선언적 데이터 모델 정의에 대한 SQL 마이그레이션 파일을 생성합니다. 이러한 마이그레이션 파일은 완전히 사용자 정의할 수 있으므로 기본 데이터베이스의 추가기능을 구성하거나 추가명령을 포함할 수 있습니다. 파종을 위해.

`schema.prisma` 파일에 다음 두모델을 추가합니다.

```groovy
model User {
  id    Int     @default(autoincrement()) @id
  email String  @unique
  name  String?
  posts Post[]
}

model Post {
  id        Int      @default(autoincrement()) @id
  title     String
  content   String?
  published Boolean? @default(false)
  author    User?    @relation(fields: [authorId], references: [id])
  authorId  Int?
}
```

Prisma 모델을 사용하면 SQL 마이그레이션 파일을 생성하고 데이터베이스에 대해 실행할 수 있습니다. 터미널에서 다음 명령을 실행하십시오.

```bash
$ npx prisma migrate dev --name init
```

이 `prisma migrate dev` 명령은 SQL 파일을 생성하고 데이터베이스에 대해 직접 실행합니다. 이 경우 기존 `prisma` 디렉토리에 다음 마이그레이션 파일이 생성되었습니다.

```bash
$ tree prisma
prisma
├── dev.db
├── migrations
│   └── 20201207100915_init
│       └── migration.sql
└── schema.prisma
```

<details><summary>생성된 SQL 문을 보려면 확장하십시오.</summary>

SQLite 데이터베이스에 다음 테이블이 생성되었습니다.

```sql
-- CreateTable
CREATE TABLE "User" (
    "id" INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
    "email" TEXT NOT NULL,
    "name" TEXT
);

-- CreateTable
CREATE TABLE "Post" (
    "id" INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
    "title" TEXT NOT NULL,
    "content" TEXT,
    "published" BOOLEAN DEFAULT false,
    "authorId" INTEGER,

    FOREIGN KEY ("authorId") REFERENCES "User"("id") ON DELETE SET NULL ON UPDATE CASCADE
);

-- CreateIndex
CREATE UNIQUE INDEX "User.email_unique" ON "User"("email");
```

</details>

#### Install and generate Prisma Client

Prisma Client는 Prisma 모델 정의에서 _생성된_ 타입이 안전한 데이터베이스 클라이언트입니다. 이러한 접근 방식으로 인해 Prisma Client는 모델에 특별히 _맞춘_ [CRUD](https://www.prisma.io/docs/concepts/components/prisma-client/crud) 작업을 노출할 수 있습니다.

프로젝트에 Prisma Client를 설치하려면 터미널에서 다음 명령을 실행하십시오.

```bash
$ npm install @prisma/client
```

설치하는 동안 Prisma는 자동으로 `prisma generate` 명령을 호출합니다. 앞으로 생성된 Prisma 클라이언트를 업데이트하려면 Prisma 모델을 변경할 _때마다_ 이 명령을 실행해야합니다.

> info **참고** `prisma generate` 명령은 Prisma 스키마를 읽고 `node_modules/@prisma/client` 내에서 생성된 Prisma Client 라이브러리를 업데이트합니다.

#### Use Prisma Client in your NestJS services

이제 Prisma Client로 데이터베이스 쿼리를 보낼 수 있습니다. Prisma Client로 쿼리를 작성하는 방법에 대해 자세히 알아보려면 [API 문서](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-client/crud)를 확인하세요.

NestJS 애플리케이션을 설정할 때 서비스내의 데이터베이스 쿼리를 위해 Prisma Client API를 추상화하고 싶을 것입니다. 시작하려면 `PrismaClient`를 인스턴스화하고 데이터베이스에 연결하는 새로운 `PrismaService`를 만들 수 있습니다.

`src` 디렉토리 내에 `prisma.service.ts`라는 새 파일을 만들고 다음 코드를 추가합니다.

```typescript
import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient
  implements OnModuleInit, OnModuleDestroy {
  async onModuleInit() {
    await this.$connect();
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

다음으로 Prisma 스키마에서 `User` 및 `Post` 모델에 대한 데이터베이스 호출을 수행하는데 사용할 수 있는 서비스를 작성할 수 있습니다.

여전히 `src` 디렉토리 안에 `user.service.ts`라는 새 파일을 만들고 다음 코드를 추가합니다.

```typescript
import { Injectable } from '@nestjs/common';
import { PrismaService } from './prisma.service';
import {
  User,
  Prisma
} from '@prisma/client';

@Injectable()
export class UserService {
  constructor(private prisma: PrismaService) {}

  async user(userWhereUniqueInput: Prisma.UserWhereUniqueInput): Promise<User | null> {
    return this.prisma.user.findUnique({
      where: userWhereUniqueInput,
    });
  }

  async users(params: {
    skip?: number;
    take?: number;
    cursor?: Prisma.UserWhereUniqueInput;
    where?: Prisma.UserWhereInput;
    orderBy?: Prisma.UserOrderByInput;
  }): Promise<User[]> {
    const { skip, take, cursor, where, orderBy } = params;
    return this.prisma.user.findMany({
      skip,
      take,
      cursor,
      where,
      orderBy,
    });
  }

  async createUser(data: Prisma.UserCreateInput): Promise<User> {
    return this.prisma.user.create({
      data,
    });
  }

  async updateUser(params: {
    where: Prisma.UserWhereUniqueInput;
    data: Prisma.UserUpdateInput;
  }): Promise<User> {
    const { where, data } = params;
    return this.prisma.user.update({
      data,
      where,
    });
  }

  async deleteUser(where: Prisma.UserWhereUniqueInput): Promise<User> {
    return this.prisma.user.delete({
      where,
    });
  }
}
```

Prisma Client의 생성된 타입을 사용하여 서비스에 의해 노출되는 메소드가 올바르게 입력되었는지 확인하십시오. 따라서 모델을 입력하고 추가 인터페이스 또는 DTO 파일을 생성하는 상용구를 저장합니다.

이제 `Post` 모델에 대해서도 똑같이하십시오.

여전히 `src` 디렉토리 안에 `post.service.ts`라는 새 파일을 만들고 다음 코드를 추가합니다.

```typescript
import { Injectable } from '@nestjs/common';
import { PrismaService } from './prisma.service';
import {
  Post,
  Prisma,
} from '@prisma/client';

@Injectable()
export class PostService {
  constructor(private prisma: PrismaService) {}

  async post(postWhereUniqueInput: Prisma.PostWhereUniqueInput): Promise<Post | null> {
    return this.prisma.post.findUnique({
      where: postWhereUniqueInput,
    });
  }

  async posts(params: {
    skip?: number;
    take?: number;
    cursor?: Prisma.PostWhereUniqueInput;
    where?: Prisma.PostWhereInput;
    orderBy?: Prisma.PostOrderByInput;
  }): Promise<Post[]> {
    const { skip, take, cursor, where, orderBy } = params;
    return this.prisma.post.findMany({
      skip,
      take,
      cursor,
      where,
      orderBy,
    });
  }

  async createPost(data: Prisma.PostCreateInput): Promise<Post> {
    return this.prisma.post.create({
      data,
    });
  }

  async updatePost(params: {
    where: Prisma.PostWhereUniqueInput;
    data: Prisma.PostUpdateInput;
  }): Promise<Post> {
    const { data, where } = params;
    return this.prisma.post.update({
      data,
      where,
    });
  }

  async deletePost(where: Prisma.PostWhereUniqueInput): Promise<Post> {
    return this.prisma.post.delete({
      where,
    });
  }
}
```

`UserService` 및 `PostService`는 현재 Prisma Client에서 사용할 수 있는 CRUD 쿼리를 래핑합니다. 실제 애플리케이션에서 서비스는 애플리케이션에 비즈니스 로직을 추가하는 장소이기도 합니다. 예를 들어 사용자의 비밀번호를 업데이트하는 `UserService` 내부에 `updatePassword`라는 메소드가 있을 수 있습니다.

##### Implement your REST API routes in the main app controller

마지막으로 이전 섹션에서 만든 서비스를 사용하여 앱의 다양한 라우트를 구현합니다. 이 가이드의 목적에 따라 모든 라우트를 기존의 `AppController` 클래스에 넣습니다.

`app.controller.ts` 파일의 내용을 다음 코드로 바꿉니다.

```typescript
import {
  Controller,
  Get,
  Param,
  Post,
  Body,
  Put,
  Delete,
} from '@nestjs/common';
import { UserService } from './user.service';
import { PostService } from './post.service';
import { User as UserModel, Post as PostModel } from '@prisma/client';

@Controller()
export class AppController {
  constructor(
    private readonly userService: UserService,
    private readonly postService: PostService,
  ) {}

  @Get('post/:id')
  async getPostById(@Param('id') id: string): Promise<PostModel> {
    return this.postService.post({ id: Number(id) });
  }

  @Get('feed')
  async getPublishedPosts(): Promise<PostModel[]> {
    return this.postService.posts({
      where: { published: true },
    });
  }

  @Get('filtered-posts/:searchString')
  async getFilteredPosts(
    @Param('searchString') searchString: string,
  ): Promise<PostModel[]> {
    return this.postService.posts({
      where: {
        OR: [
          {
            title: { contains: searchString },
          },
          {
            content: { contains: searchString },
          },
        ],
      },
    });
  }

  @Post('post')
  async createDraft(
    @Body() postData: { title: string; content?: string; authorEmail: string },
  ): Promise<PostModel> {
    const { title, content, authorEmail } = postData;
    return this.postService.createPost({
      title,
      content,
      author: {
        connect: { email: authorEmail },
      },
    });
  }

  @Post('user')
  async signupUser(
    @Body() userData: { name?: string; email: string },
  ): Promise<UserModel> {
    return this.userService.createUser(userData);
  }

  @Put('publish/:id')
  async publishPost(@Param('id') id: string): Promise<PostModel> {
    return this.postService.updatePost({
      where: { id: Number(id) },
      data: { published: true },
    });
  }

  @Delete('post/:id')
  async deletePost(@Param('id') id: string): Promise<PostModel> {
    return this.postService.deletePost({ id: Number(id) });
  }
}
```

This controller implements the following routes:

###### `GET`

- `/post/:id`: Fetch a single post by its `id`
- `/feed`: Fetch all _published_ posts
- `/filter-posts/:searchString`: Filter posts by `title` or `content`

###### `POST`

- `/post`: Create a new post
  - Body:
    - `title: String` (required): The title of the post
    - `content: String` (optional): The content of the post
    - `authorEmail: String` (required): The email of the user that creates the post
- `/user`: Create a new user
  - Body:
    - `email: String` (required): The email address of the user
    - `name: String` (optional): The name of the user

###### `PUT`

- `/publish/:id`: Publish a post by its `id`

###### `DELETE`

- `/post/:id`: Delete a post by its `id`

#### Summary

이 레시피에서는 NestJS와 함께 Prisma를 사용하여 REST API를 구현하는 방법을 배웠습니다. API의 라우트를 구현하는 컨트롤러는 `PrismaService`를 호출하고, 이 서비스는 차례로 Prisma Client를 사용하여 들어오는 요청의 데이터 요구 사항을 충족하기 위해 데이터베이스에 쿼리를 전송합니다.

Prisma에서 NestJS를 사용하는 방법에 대해 자세히 알아 보려면 다음 리소스를 확인하세요.

- [NestJS & Prisma](https://www.prisma.io/nestjs)
- [REST 및 GraphQL을 위한 바로 실행 가능한 예제 프로젝트](https://github.com/prisma/prisma-examples/)
- [프러덕션 준비가 된 스타터 키트](https://github.com/fivethree-team/nestjs-prisma-starter#instructions)
- [비디오: Prisma와 함께 NestJS를 사용하여 데이터베이스에 액세스 (5 분)](https://www.youtube.com/watch?v=UlVJ340UEuk&ab_channel=Prisma) by [Marc Stammerjohann](https://github.com/marcjulian)
