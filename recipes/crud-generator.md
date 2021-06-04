### CRUD generator

프로젝트 수명 동안 새로운 기능을 구축할 때 종종 애플리케이션에 새로운 리소스를 추가해야합니다. 이러한 리소스에는 일반적으로 새 리소스를 정의할 때마다 반복해야하는 여러 반복 작업이 필요합니다.

#### Introduction

Let's imagine a real-world scenario, where we need to expose CRUD endpoints for 2 entities, let's say **User** and **Product** entities.
Following the best practices, for each entity we would have to perform several operations, as follows:
**사용자 User** 및 **제품 Product** 항목과 같은 2개의 항목에 대한 CRUD 엔드포인트를 노출해야하는 실제 시나리오를 상상해 보겠습니다.
모범 사례에 따라 각 엔터티에 대해 다음과 같은 여러 작업을 수행해야합니다.

- 모듈(`nest g mo`)을 생성하여 코드를 구성하고 명확한 경계를 설정(관련 구성요소 그룹화)
- 컨트롤러(`nest g co`)를 생성하여 CRUD 경로(또는 GraphQL 애플리케이션의 쿼리/뮤테이션)를 정의합니다.
- 비즈니스 로직 구현 및 분리를 위한 서비스 (`nest g s`) 생성
- 리소스 데이터 형태를 나타내는 엔티티 클래스/인터페이스 생성
- 데이터 전송 객체 (또는 GraphQL 애플리케이션에 대한 입력)를 생성하여 데이터가 네트워크를 통해 전송되는 방식을 정의합니다.

그것은 많은 단계입니다!

이러한 반복적인 프로세스의 속도를 높이기 위해 [Nest CLI](/cli/overview)는 모든 상용구 코드를 자동으로 생성하는 생성기(스키매틱)를 제공하여 이 모든 작업을 피하고 개발자 경험을 훨씬 간단하게 만듭니다.

> info **참고** 스키메틱은 **HTTP** 컨트롤러, **Microservice** 컨트롤러, **GraphQL** 리졸버(코드 우선 및 스키마 우선) 및 **WebSocket** 게이트웨이 생성을 지원합니다.

#### Generating a new resource

새 리소스를 만들려면 프로젝트의 루트 디렉터리에서 다음 명령을 실행하면됩니다.

```shell
$ nest g resource
```

`nest g resource` 명령은 모든 NestJS 빌딩 블록(모듈, 서비스, 컨트롤러 클래스)뿐만 아니라 엔티티 클래스, DTO 클래스 및 테스트(`.spec`) 파일도 생성합니다.

아래에서 생성된 컨트롤러 파일(REST API용)을 볼 수 있습니다.

```typescript
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Post()
  create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }

  @Get()
  findAll() {
    return this.usersService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(+id);
  }

  @Patch(':id')
  update(@Param('id') id: string, @Body() updateUserDto: UpdateUserDto) {
    return this.usersService.update(+id, updateUserDto);
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return this.usersService.remove(+id);
  }
}
```

또한 손가락을 떼지 않고도 모든 CRUD 엔드 포인트(REST API에 대한 라우트, GraphQL에 대한 쿼리 및 뮤테이션, 마이크로서비스 및 WebSocket 게이트웨이 모두에 대한 메시지 구독)에 대한 자리표시자를 자동으로 생성합니다.

> warning **참고** 생성된 서비스 클래스는 특정 **ORM (또는 데이터 소스)**에 연결되지 **않습니다**. 이것은 모든 프로젝트의 요구를 충족시킬 수 있을 정도로 생성기를 일반화합니다. 기본적으로 모든 메서드에는 자리표시자가 포함되어 있으므로 프로젝트에 특정한 데이터 원본으로 채울 수 있습니다.

마찬가지로 GraphQL 애플리케이션에 대한 리졸버를 생성하려면 전송 레이어로 `GraphQL (코드 우선)`(또는 `GraphQL (스키마 우선)`)을 선택하면 됩니다.

이 경우 NestJS는 REST API 컨트롤러 대신 리졸버 클래스를 생성합니다.

```shell
$ nest g resource users

> ? What transport layer do you use? GraphQL (code first)
> ? Would you like to generate CRUD entry points? Yes
> CREATE src/users/users.module.ts (224 bytes)
> CREATE src/users/users.resolver.spec.ts (525 bytes)
> CREATE src/users/users.resolver.ts (1109 bytes)
> CREATE src/users/users.service.spec.ts (453 bytes)
> CREATE src/users/users.service.ts (625 bytes)
> CREATE src/users/dto/create-user.input.ts (195 bytes)
> CREATE src/users/dto/update-user.input.ts (281 bytes)
> CREATE src/users/entities/user.entity.ts (187 bytes)
> UPDATE src/app.module.ts (312 bytes)
```

> info **힌트** 테스트 파일 생성을 방지하려면 다음과 같이 `--no-spec` 플래그를 전달할 수 있습니다. `nest g resource users --no-spec`

아래에서 모든 보일러플레이트 뮤테이션과 쿼리가 생성되었을 뿐만 아니라 모든 것이 하나로 묶여있음을 알 수 있습니다. 우리는 `UsersService`, `User` Entity 및 DTO를 활용하고 있습니다.

```typescript
import { Resolver, Query, Mutation, Args, Int } from '@nestjs/graphql';
import { UsersService } from './users.service';
import { User } from './entities/user.entity';
import { CreateUserInput } from './dto/create-user.input';
import { UpdateUserInput } from './dto/update-user.input';

@Resolver(() => User)
export class UsersResolver {
  constructor(private readonly usersService: UsersService) {}

  @Mutation(() => User)
  createUser(@Args('createUserInput') createUserInput: CreateUserInput) {
    return this.usersService.create(createUserInput);
  }

  @Query(() => [User], { name: 'users' })
  findAll() {
    return this.usersService.findAll();
  }

  @Query(() => User, { name: 'user' })
  findOne(@Args('id', { type: () => Int }) id: number) {
    return this.usersService.findOne(id);
  }

  @Mutation(() => User)
  updateUser(@Args('updateUserInput') updateUserInput: UpdateUserInput) {
    return this.usersService.update(updateUserInput.id, updateUserInput);
  }

  @Mutation(() => User)
  removeUser(@Args('id', { type: () => Int }) id: number) {
    return this.usersService.remove(id);
  }
}
```
