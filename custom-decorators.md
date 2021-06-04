### Custom route decorators

Nest는 **데코레이터**라는 언어기능을 중심으로 구축되었습니다. 데코레이터는 일반적으로 사용되는 많은 프로그래밍 언어에서 잘 알려진 개념이지만 자바스크립트 세계에서는 여전히 비교적 새로운 개념입니다. 데코레이터의 작동 방식을 더 잘 이해하려면 [이 도움말](https://medium.com/google-developers/exploring-es7-decorators-76ecb65fb841)을 읽어 보시기 바랍니다. 다음은 간단한 정의입니다.

<blockquote class="external">
> ES2016 데코레이터는 함수를 반환하고 대상, 이름 및 속성 설명자를 인수로 사용할 수 있는 표현식입니다. 데코레이터 앞에 <code>@</code>문자를 붙이고 이를 데코하려는 항목의 맨 위에 배치하여 적용합니다. 데코레이터는 클래스, 메서드 또는 속성에 대해 정의할 수 있습니다.
</blockquote>

#### Param decorators

Nest는 HTTP 라우트 핸들러와 함께 사용할 수있는 유용한 **매개변수 데코레이터** 세트를 제공합니다. 아래는 제공된 데코레이터와 이들이 나타내는 일반 Express(또는 Fastify) 객체 목록입니다.

<table>
  <tbody>
    <tr>
      <td><code>@Request(), @Req()</code></td>
      <td><code>req</code></td>
    </tr>
    <tr>
      <td><code>@Response(), @Res()</code></td>
      <td><code>res</code></td>
    </tr>
    <tr>
      <td><code>@Next()</code></td>
      <td><code>next</code></td>
    </tr>
    <tr>
      <td><code>@Session()</code></td>
      <td><code>req.session</code></td>
    </tr>
    <tr>
      <td><code>@Param(param?: string)</code></td>
      <td><code>req.params</code> / <code>req.params[param]</code></td>
    </tr>
    <tr>
      <td><code>@Body(param?: string)</code></td>
      <td><code>req.body</code> / <code>req.body[param]</code></td>
    </tr>
    <tr>
      <td><code>@Query(param?: string)</code></td>
      <td><code>req.query</code> / <code>req.query[param]</code></td>
    </tr>
    <tr>
      <td><code>@Headers(param?: string)</code></td>
      <td><code>req.headers</code> / <code>req.headers[param]</code></td>
    </tr>
    <tr>
      <td><code>@Ip()</code></td>
      <td><code>req.ip</code></td>
    </tr>
    <tr>
      <td><code>@HostParam()</code></td>
      <td><code>req.hosts</code></td>
    </tr>
  </tbody>
</table>

또한 나만의 **커스텀 데코레이터**를 만들 수 있습니다. 이것이 왜 유용합니까?

node.js 세계에서는 **요청** 객체에 속성을 연결하는 것이 일반적입니다. 그런다음 다음과 같은 코드를 사용하여 각 라우트 핸들러에서 수동으로 추출합니다.

```typescript
const user = req.user;
```

코드를 더 읽기 쉽고 투명하게 만들기 위해 `@User()` 데코레이터를 만들어 모든 컨트롤러에서 재사용할 수 있습니다.

```typescript
@@filename(user.decorator)
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const User = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  },
);
```

그런 다음 요구사항에 맞는 모든 곳에서 간단히 사용할 수 있습니다.

```typescript
@@filename()
@Get()
async findOne(@User() user: UserEntity) {
  console.log(user);
}
@@switch
@Get()
@Bind(User())
async findOne(user) {
  console.log(user);
}
```

#### Passing data

데코레이터의 동작이 일부 조건에 따라 달라지는 경우 `data` 매개변수를 사용하여 데코레이터의 팩토리 함수에 인수를 전달할 수 있습니다. 이에 대한 한가지 사용사례는 키별로 요청객체에서 속성을 추출하는 커스텀 데코레이터입니다. 예를 들어, [인증 레이어](/techniques/authentication#implementing-passport-strategies)가 요청의 유효성을 검사하고 사용자 엔터티를 요청객체에 연결한다고 가정해 보겠습니다. 인증된 요청에 대한 사용자 엔티티는 다음과 같습니다.

```json
{
  "id": 101,
  "firstName": "Alan",
  "lastName": "Turing",
  "email": "alan@email.com",
  "roles": ["admin"]
}
```

속성 이름을 키로 받아 관련 값이 있는 경우(또는 존재하지 않는 경우 정의되지 않거나 `user` 객체가 생성되지 않은 경우) 반환하는 데코레이터를 정의해 보겠습니다.

```typescript
@@filename(user.decorator)
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const User = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;

    return data ? user?.[data] : user;
  },
);
@@switch
import { createParamDecorator } from '@nestjs/common';

export const User = createParamDecorator((data, ctx) => {
  const request = ctx.switchToHttp().getRequest();
  const user = request.user;

  return data ? user && user[data] : user;
});
```

컨트롤러의 `@User()` 데코레이터를 통해 특정 속성에 액세스하는 방법은 다음과 같습니다.

```typescript
@@filename()
@Get()
async findOne(@User('firstName') firstName: string) {
  console.log(`Hello ${firstName}`);
}
@@switch
@Get()
@Bind(User('firstName'))
async findOne(firstName) {
  console.log(`Hello ${firstName}`);
}
```

이 동일한 데코레이터를 다른 키와 함께 사용하여 다른 속성에 액세스할 수 있습니다. `user` 객체가 깊거나 복잡한 경우 요청 핸들러를 더 쉽고 읽기 쉽게 구현할 수 있습니다.

> info **힌트** TypeScript 사용자의 경우 `createParamDecorator<T>()`가 제네릭이라는 점에 유의하세요. 즉, `createParamDecorator<string>((data, ctx) => ...)`와 같이 형식 안전성을 명시적으로 시행할 수 있습니다. 또는 팩토리 함수에 매개변수 타입을 지정하십시오 (예: `createParamDecorator((data: string, ctx) => ...)`). 둘 다 생략하면 `data`의 타입은 `any`가 됩니다.

#### Working with pipes

Nest는 내장 매개변수 데코레이터 (`@Body()`, `@Param()` 및 `@Query()`)와 동일한 방식으로 맞춤 매개변수 데코레이터를 처리합니다. 즉, 주석이 추가된 커스텀 매개변수에 대해서도 파이프가 실행됩니다(이 예에서는 `user` 인수). 또한 파이프를 커스텀 데코레이터에 직접 적용할 수 있습니다.

```typescript
@@filename()
@Get()
async findOne(
  @User(new ValidationPipe({ validateCustomDecorators: true }))
  user: UserEntity,
) {
  console.log(user);
}
@@switch
@Get()
@Bind(User(new ValidationPipe({ validateCustomDecorators: true })))
async findOne(user) {
  console.log(user);
}
```

> info **힌트** `validateCustomDecorators` 옵션은 반드시 `true`로 설정해야 합니다. `ValidationPipe`는 기본적으로 커스텀 데코레이터로 주석이 달린 인수의 유효성을 검사하지 않습니다.

#### Decorator composition

Nest는 여러 데코레이터를 구성하는 헬퍼 메서드를 제공합니다. 예를 들어 인증과 관련된 모든 데코레이터를 단일 데코레이터로 결합한다고 가정합니다. 다음 구성으로 수행할 수 있습니다.

```typescript
@@filename(auth.decorator)
import { applyDecorators } from '@nestjs/common';

export function Auth(...roles: Role[]) {
  return applyDecorators(
    SetMetadata('roles', roles),
    UseGuards(AuthGuard, RolesGuard),
    ApiBearerAuth(),
    ApiUnauthorizedResponse({ description: 'Unauthorized' }),
  );
}
@@switch
import { applyDecorators } from '@nestjs/common';

export function Auth(...roles) {
  return applyDecorators(
    SetMetadata('roles', roles),
    UseGuards(AuthGuard, RolesGuard),
    ApiBearerAuth(),
    ApiUnauthorizedResponse({ description: 'Unauthorized' }),
  );
}
```

그런 다음이 사용자 정의 `@Auth()` 데코레이터를 다음과 같이 사용할 수 있습니다.

```typescript
@Get('users')
@Auth('admin')
findAllUsers() {}
```

이것은 단일선언으로 4개의 데코레이터를 모두 적용하는 효과가 있습니다.

> warning **경고** `@nestjs/swagger` 패키지의 `@ApiHideProperty()` 데코레이터는 구성할 수 없으며 `applyDecorators` 함수와 함께 제대로 작동하지 않습니다.
