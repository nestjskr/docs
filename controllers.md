### Controllers

컨트롤러는 들어오는 **요청 request**을 처리하고 **응답 response**을 클라이언트에 반환할 책임이 있습니다.

<figure><img src="/assets/Controllers_1.png" /></figure>

컨트롤러의 목적은 애플리케이션에 대한 특정 요청을 수신하는 것입니다. **라우팅** 메커니즘은 어떤 컨트롤러가 어떤 요청을 수신하는지 제어합니다. 종종 각 컨트롤러에는 둘 이상의 라우트가 있으며 다른 라우트는 다른 작업을 수행할 수 있습니다.

기본 컨트롤러를 만들기 위해 클래스와 **데코레이터**를 사용합니다. 데코레이터는 클래스를 필수 메타데이터와 연결하고 Nest가 라우팅 맵을 만들 수 있도록 합니다(요청을 해당 컨트롤러에 연결).

> info **힌트** [validation](/techniques/validation)이 내장된 CRUD 컨트롤러를 빠르게 생성하려면 CLI의 [CRUD 생성기](/recipes/crud-generator#crud-generator): `nest g resource [name]`을 사용할 수 있습니다.

#### Routing

다음 예제에서는 기본 컨트롤러를 정의하는 데 **필수**인 `@Controller()` 데코레이터를 사용합니다. 선택적 라우트 경로(path) 접두사 `cats`를 지정합니다. `@Controller()` 데코레이터에서 경로(path) 접두사를 사용하면 관련 라우트 집합을 쉽게 그룹화하고 반복코드를 최소화할 수 있습니다. 예를 들어 `/customers` 라우트 아래에서 고객 엔터티와의 상호작용을 관리하는 라우트 집합을 그룹화하도록 선택할 수 있습니다. 이 경우 `@Controller()` 데코레이터에서 경로(path) 접두사 `customers`를 지정하여 파일의 각 라우트에 대해 경로(path)의 해당부분을 반복할 필요가 없습니다.

```typescript
@@filename(cats.controller)
import { Controller, Get } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Get()
  findAll(): string {
    return 'This action returns all cats';
  }
}
@@switch
import { Controller, Get } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Get()
  findAll() {
    return 'This action returns all cats';
  }
}
```

> info **힌트** CLI를 사용하여 컨트롤러를 만들려면 `$ nest g controller cats` 명령을 실행하면 됩니다.

`findAll()` 메서드 앞에 있는 `@Get()` HTTP 요청 메서드 데코레이터는 Nest에 HTTP 요청에 대한 특정 엔드포인트에 대한 핸들러를 생성하도록 지시합니다. 엔드포인트는 HTTP 요청 메서드(이 경우 GET) 및 라우트 경로에 해당합니다. 라우트 경로는 무엇입니까? 핸들러의 라우트 경로는 컨트롤러에 대해 선언된(선택 사항) 접두사와 요청 데코레이터에 지정된 경로를 연결하여 결정됩니다. 모든 라우트 (`cats`)에 대한 접두사를 선언하고 데코레이터에 경로 정보를 추가하지 않았으므로 Nest는 `GET /cats` 요청을 이 핸들러에 매핑합니다. 언급했듯이 경로에는 선택적 컨트롤러 경로 접두사 **및** 요청 메서드 데코레이터에서 선언된 모든 경로 문자열이 모두 포함됩니다. 예를 들어, 데코레이터 `@Get('profile')`과 결합된 `customers`의 경로 접두사는 `GET /customers/profile`과 같은 요청에 대한 라우트 매핑을 생성합니다.

위의 예에서 이 엔드포인트에 GET 요청이 있을 때 Nest는 요청을 커스텀 `findAll()` 메서드로 라우팅합니다. 여기서 선택한 메서드 이름은 완전히 임의적입니다. 분명히 경로를 바인딩할 메서드를 선언해야 하지만 Nest는 선택한 메서드 이름에 어떤 의미도 부여하지 않습니다.

이 메서드는 200 상태 코드와 관련 응답을 반환합니다. 이 경우에는 문자열일 뿐입니다. 왜 그럴까요? 설명하기 위해 먼저 Nest가 응답을 조작하기 위해 **다른** 두가지 옵션을 사용한다는 개념을 소개하겠습니다.

<table>
  <tr>
    <td>표준 (권장)</td>
    <td>
이 내장 메서드를 사용하면 요청 핸들러가 자바스크립트 객체 또는 배열을 반환할 때 <strong>자동</strong>으로 JSON으로 직렬화됩니다. 그러나 자바스크립트 기본 타입(예: <code>string</code>, <code>number</code>, <code>boolean</code>)을 반환하면 Nest는 직렬화를 시도하지 않고 값만 보냅니다. 이렇게 하면 응답처리가 간단해집니다. 값을 반환하기만 하면 Nest가 나머지 작업을 처리합니다.
<br />
<br /> 또한 응답의 <strong>상태 코드</strong>는 201을 사용하는 POST 요청을 제외하고는 항상 기본적으로 200입니다. 핸들러 수준에서 <code>@HttpCode(...)</code> 데코레이터를 추가하여 이 동작을 쉽게 변경할 수 있습니다(참조: <a href='controllers#status-code'>상태 코드</a>).
    </td>
  </tr>
  <tr>
    <td>라이브별 Library-specific</td>
    <td>
메소드 핸들러 시그니처(예: <code>findAll(@Res() response)</code>)에서 <code>@Res()</code> 데코레이터를 사용하여 삽입할 수 있는 라이브러리별(예: Express) <a href="https://expressjs.com/en/api.html#res" rel="nofollow" target="_blank">응답객체</a>를 사용할 수 있습니다. 예를 들어 Express에서는 <code>response.status(200).send()</code>와 같은 코드를 사용하여 응답을 구성할 수 있습니다.
    </td>
  </tr>
</table>

> warning **경고** Nest는 핸들러가 `@Res()` 또는 `@Next()`를 사용할 때 이를 감지하여 라이브러리별 옵션을 선택했음을 나타냅니다. 두 접근 방식을 동시에 사용하는 경우 이 단일 라우트에 대해 표준 접근방식이 **자동으로 비활성화**되고 더 이상 예상대로 작동하지 않습니다. 두 접근 방식을 동시에 사용하려면 (예: 쿠키/헤더만 설정하고 나머지는 프레임워크에 남겨 두도록 응답객체를 삽입) `@Res({{ '{' }} passthrough: true {{ '}' }})` 데코레이터에서 `passthrough` 옵션을 `true`로 설정해야 합니다.

<app-banner-enterprise></app-banner-enterprise>

#### Request object

핸들러는 종종 클라이언트 **요청** 세부정보에 액세스해야 합니다. Nest는 기본 플랫폼(기본적으로 Express)의 [요청객체](https://expressjs.com/en/api.html#req)에 대한 액세스를 제공합니다. 핸들러의 시그니처에 `@Req()` 데코레이터를 추가하여 Nest에 주입하도록 지시하여 요청객체에 액세스할 수 있습니다.

```typescript
@@filename(cats.controller)
import { Controller, Get, Req } from '@nestjs/common';
import { Request } from 'express';

@Controller('cats')
export class CatsController {
  @Get()
  findAll(@Req() request: Request): string {
    return 'This action returns all cats';
  }
}
@@switch
import { Controller, Bind, Get, Req } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Get()
  @Bind(Req())
  findAll(request) {
    return 'This action returns all cats';
  }
}
```

> info **힌트** `express` 타입(위의 `request: Request` 매개변수 예제 참조)을 활용하려면 `@types/express` 패키지를 설치합니다.

요청객체는 HTTP 요청을 나타내며 요청 쿼리 문자열, 매개변수, HTTP 헤더 및 본문에 대한 속성을 포함합니다(자세한 내용은 [여기](https://expressjs.com/en/api.html#req) 참조). 대부분의 경우 이러한 속성을 수동으로 가져올 필요가 없습니다. 바로 사용할 수 있는 `@Body()` 또는 `@Query()`와 같은 전용 데코레이터를 대신 사용할 수 있습니다. 아래는 제공된 데코레이터와 이들이 나타내는 일반 플랫폼별 객체 목록입니다.

<table>
  <tbody>
    <tr>
      <td><code>@Request(), @Req()</code></td>
      <td><code>req</code></td></tr>
    <tr>
      <td><code>@Response(), @Res()</code><span class="table-code-asterisk">*</span></td>
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
      <td><code>@Param(key?: string)</code></td>
      <td><code>req.params</code> / <code>req.params[key]</code></td>
    </tr>
    <tr>
      <td><code>@Body(key?: string)</code></td>
      <td><code>req.body</code> / <code>req.body[key]</code></td>
    </tr>
    <tr>
      <td><code>@Query(key?: string)</code></td>
      <td><code>req.query</code> / <code>req.query[key]</code></td>
    </tr>
    <tr>
      <td><code>@Headers(name?: string)</code></td>
      <td><code>req.headers</code> / <code>req.headers[name]</code></td>
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

<sup>\*</sup> 기본 HTTP 플랫폼(예: Express 및 Fastify)에서 입력과의 호환성을 위해 Nest는 `@Res()` 및 `@Response()` 데코레이터를 제공합니다. `@Res()`는 단순히 `@Response()`의 별칭입니다. 둘 다 기본 네이티브 플랫폼 `response` 객체 인터페이스를 직접 노출합니다. 이를 사용하는 경우 기본 라이브러리(예: `@types/express`)에 대한 타입도 가져와야 최대한 활용할 수 있습니다. 메소드 핸들러에 `@Res()` 또는 `@Response()`를 삽입할 때 해당 핸들러에 대해 Nest를 **라이브러리별 모드**로 설정하고 응답을 관리해야 합니다. 그렇게 할 때 `응답객체`(예: `res.json(...)` 또는 `res.send(...)`)를 호출하여 일종의 응답을 발행해야합니다. 그렇지 않으면 HTTP 서버가 중단됩니다.

> info **힌트** 나만의 데코레이터를 만드는 방법을 배우려면 [이 장](/custom-decorators)을 방문하세요.

#### Resources

이전에는 cats 리소스(**GET** 라우트)를 가져오는 엔드포인트를 정의했습니다. 일반적으로 새 레코드를 생성하는 엔드포인트도 제공하려고 합니다. 이를 위해 **POST** 핸들러를 만들어 보겠습니다.

```typescript
@@filename(cats.controller)
import { Controller, Get, Post } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Post()
  create(): string {
    return 'This action adds a new cat';
  }

  @Get()
  findAll(): string {
    return 'This action returns all cats';
  }
}
@@switch
import { Controller, Get, Post } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Post()
  create() {
    return 'This action adds a new cat';
  }

  @Get()
  findAll() {
    return 'This action returns all cats';
  }
}
```

그렇게 간단합니다. Nest는 모든 표준 HTTP 메소드에 대한 데코레이터를 제공합니다: `@Get()`, `@Post()`, `@Put()`, `@Delete()`, `@Patch()`, `@Options()` 및 `@Head()`. 또한 `@All()`은 이들 모두를 처리하는 엔드 포인트를 정의합니다.

#### Route wildcards

패턴 기반 라우트도 지원됩니다. 예를 들어 별표는 와일드카드로 사용되며 모든 문자조합과 일치합니다.

```typescript
@Get('ab*cd')
findAll() {
  return 'This route uses a wildcard';
}
```

`'ab*cd'` 라우트 경로는 `abcd`, `ab_cd`, `abecd` 등과 일치합니다. `?`, `+`, `*` 및 `()` 문자는 라우트 경로에 사용될 수 있으며 해당 정규표현식 대응 부분의 하위집합입니다. 하이픈(`-`)과 점(`.`)은 문자열 기반 경로로 문자 그대로 해석됩니다.

#### Status code

언급했듯이 **201**인 POST 요청을 제외하고 응답 **상태코드**는 기본적으로 항상 **200**입니다. 핸들러 레벨에서 `@HttpCode(...)` 데코레이터를 추가하여 이 동작을 쉽게 변경할 수 있습니다.

```typescript
@Post()
@HttpCode(204)
create() {
  return 'This action adds a new cat';
}
```

> info **힌트** `@nestjs/common` 패키지에서 `HttpCode`를 가져옵니다.

종종 상태코드는 정적이 아니지만 다양한 요인에 따라 달라집니다. 이 경우 라이브러리별 **응답**(`@Res()`를 사용하여 주입)객체를 사용할 수 있습니다 (또는 오류가 발생한 경우 예외 발생).

#### Headers

커스텀 응답헤더를 지정하려면 `@Header()` 데코레이터 또는 라이브러리별 응답객체를 사용할 수 있습니다(그리고 `res.header()`를 직접 호출).

```typescript
@Post()
@Header('Cache-Control', 'none')
create() {
  return 'This action adds a new cat';
}
```

> info **힌트** `@nestjs/common` 패키지에서 `Header`를 가져옵니다.

#### Redirection

응답을 특정 URL로 리디렉션하려면 `@Redirect()` 데코레이터 또는 라이브러리별 응답객체를 사용할 수 있습니다 (그리고 `res.redirect()`를 직접 호출).

`@Redirect()`는 `url`과 `statusCode`라는 두개의 인수를 취하며 둘 다 선택사항입니다. 생략된 경우 `statusCode`의 기본값은 `302`(`Found`)입니다.

```typescript
@Get()
@Redirect('https://nestjs.com', 301)
```

때때로 HTTP 상태코드 또는 리디렉션 URL을 동적으로 확인해야 할 수 있습니다. 다음과 같은 형태로 라우트 핸들러 메서드에서 객체를 반환하면 됩니다.

```json
{
  "url": string,
  "statusCode": number
}
```

반환된 값은 `@Redirect()` 데코레이터에 전달된 모든 인수를 재정의합니다. 예를 들면:

```typescript
@Get('docs')
@Redirect('https://docs.nestjs.com', 302)
getDocs(@Query('version') version) {
  if (version && version === '5') {
    return { url: 'https://docs.nestjs.com/v5/' };
  }
}
```

#### Route parameters

요청의 일부로 **동적 데이터**를 수락해야 하는 경우 정적 경로가 있는 라우트가 작동하지 않습니다 (예: `GET /cats/1`는 ID가 `1`인 cat을 가져옴). 매개변수가 있는 라우트를 정의하기 위해 라우트 경로에 라우트 매개변수 **토큰 tokens**을 추가하여 요청 URL의 해당 위치에서 동적값을 캡처할 수 있습니다. 아래 `@Get()` 데코레이터 예제의 경로 매개변수 토큰은 이 사용법을 보여줍니다. 이렇게 선언된 라우트 매개변수는 `@Param ()` 데코레이터를 사용하여 액세스할 수 있으며, 이는 메소드 서명에 추가되어야 합니다.

```typescript
@@filename()
@Get(':id')
findOne(@Param() params): string {
  console.log(params.id);
  return `This action returns a #${params.id} cat`;
}
@@switch
@Get(':id')
@Bind(Param())
findOne(params) {
  console.log(params.id);
  return `This action returns a #${params.id} cat`;
}
```

`@Param()`은 메서드 매개변수(위의 예에서는 `params`)를 장식하는데 사용되며, **라우트** 매개변수를 메서드 본문내에서 장식된 메서드 매개변수의 속성으로 사용할 수 있도록 합니다. 위 코드에서 볼 수 있듯이 `params.id`를 참조하여 `id` 매개변수에 액세스할 수 있습니다. 특정 매개변수 토큰을 데코레이터에 전달한 다음 메서드 본문에서 이름으로 직접 라우트 매개변수를 참조할 수도 있습니다.

> info **힌트** `@nestjs/common` 패키지에서 `Param`을 가져옵니다.

```typescript
@@filename()
@Get(':id')
findOne(@Param('id') id: string): string {
  return `This action returns a #${id} cat`;
}
@@switch
@Get(':id')
@Bind(Param('id'))
findOne(id) {
  return `This action returns a #${id} cat`;
}
```

#### Sub-Domain Routing

`@Controller` 데코레이터는 들어오는 요청의 HTTP 호스트가 특정값과 일치하도록 `host` 옵션을 사용할 수 있습니다.

```typescript
@Controller({ host: 'admin.example.com' })
export class AdminController {
  @Get()
  index(): string {
    return 'Admin page';
  }
}
```

> warning **경고** **Fastify**에는 중첩 라우터(nested route)에 대한 지원이 없기 때문에 하위 도메인 라우팅을 사용할 때, (기본) Express 어댑터를 대신 사용해야합니다.

라우트 `path`와 유사하게 `host` 옵션은 토큰을 사용하여 호스트 이름의 해당 위치에서 동적값을 캡처할 수 있습니다. 아래 `@Controller()` 데코레이터 예제의 호스트 매개변수 토큰은 이 사용법을 보여줍니다. 이러한 방식으로 선언된 호스트 매개변수는 `@HostParam()` 데코레이터를 사용하여 액세스할 수 있으며 메소드 서명에 추가해야 합니다.

```typescript
@Controller({ host: ':account.example.com' })
export class AccountController {
  @Get()
  getInfo(@HostParam('account') account: string) {
    return account;
  }
}
```

#### Scopes

다른 프로그래밍 언어 배경을 가진 사람들의 경우 Nest에서 거의 모든 것이 들어오는 요청에서 공유된다는 사실을 배우는 것은 예상치 못한 일입니다. 데이터베이스에 대한 연결 풀, 전역상태의 싱글톤 서비스 등이 있습니다. Node.js는 모든 요청이 별도의 스레드에서 처리되는 요청/응답 다중스레드 상태 비저장(Multi-Threaded Stateless) 모델을 따르지 않습니다. 따라서 싱글톤 인스턴스를 사용하는 것은 애플리케이션에 완전히 **안전**합니다.

그러나 컨트롤러의 요청 기반 수명이 바람직한 동작일 수 있는 경우가 있습니다. 예를 들어 GraphQL 애플리케이션의 요청별 캐싱, 요청 추적 또는 [멀티테넌시](https://ko.wikipedia.org/wiki/%EB%A9%80%ED%8B%B0%ED%85%8C%EB%84%8C%EC%8B%9C)가 있습니다. [여기](/fundamentals/injection-scopes)에서 범위를 제어하는 방법을 알아보세요.

#### Asynchronicity

우리는 최신 자바스크립트를 좋아하며 데이터 추출이 대부분 **비동기적**이라는 것을 알고 있습니다. 이것이 Nest가 `비동기(async)` 기능을 지원하고 잘 작동하는 이유입니다.

> info **힌트** [여기](https://kamilmysliwiec.com/typescript-2-1-introduction-async-await)에서 `async/await` 기능에 대해 자세히 알아보기

모든 비동기 함수는 `Promise`를 반환해야 합니다. 즉, Nest가 자체적으로 해결할 수 있는 지연된 값을 반환할 수 있습니다. 이것의 예를 보겠습니다:

```typescript
@@filename(cats.controller)
@Get()
async findAll(): Promise<any[]> {
  return [];
}
@@switch
@Get()
async findAll() {
  return [];
}
```

위의 코드는 완전히 유효합니다. 또한 Nest 라우트 핸들러는 RxJS [관찰가능한 스트림](http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html)를 반환할 수 있으므로 훨씬 더 강력합니다. Nest는 자동으로 아래의 소스를 구독하고 마지막으로 내보낸 값을 가져옵니다(스트림이 완료되면).

> translation_note **역주** [Promise vs Observable](https://stackoverflow.com/questions/37364973/what-is-the-difference-between-promises-and-observables) 차이점 알아보기

```typescript
@@filename(cats.controller)
@Get()
findAll(): Observable<any[]> {
  return of([]);
}
@@switch
@Get()
findAll() {
  return of([]);
}
```

위의 두가지 접근방식이 모두 작동하며 요구사항에 맞는 것을 사용할 수 있습니다.

#### Request payloads

POST 라우트 핸들러의 이전 예제는 클라이언트 매개변수를 허용하지 않았습니다. 여기에 `@Body()` 데코레이터를 추가하여 이 문제를 해결하겠습니다.

그러나 먼저(TypeScript를 사용하는 경우) **DTO**(데이터 전송 개체) 스키마를 결정해야 합니다. DTO는 데이터가 네트워크를 통해 전송되는 방식을 정의하는 객체입니다. **TypeScript** 인터페이스를 사용하거나 간단한 클래스를 사용하여 DTO 스키마를 확인할 수 있습니다. 흥미롭게도 여기서 **클래스**을 사용하는 것이 좋습니다. 왜? 클래스는 자바스크립트 ES6 표준의 일부이므로 컴파일된 자바스크립트에서 실제 엔티티로 유지됩니다. 반면에 TypeScript 인터페이스는 트랜스 파일중에 제거되기 때문에 Nest는 런타임에 이를 참조할 수 없습니다. **파이프**와 같은 기능은 런타임에 변수의 메타타입에 액세스할 수 있을 때 추가 가능성을 제공하기 때문에 중요합니다.

`CreateCatDto` 클래스를 만들어 보겠습니다.

```typescript
@@filename(create-cat.dto)
export class CreateCatDto {
  name: string;
  age: number;
  breed: string;
}
```

기본 속성은 세가지뿐입니다. 그런 다음 `CatsController` 내부에서 새로 생성된 DTO를 사용할 수 있습니다.

```typescript
@@filename(cats.controller)
@Post()
async create(@Body() createCatDto: CreateCatDto) {
  return 'This action adds a new cat';
}
@@switch
@Post()
@Bind(Body())
async create(createCatDto) {
  return 'This action adds a new cat';
}
```

#### Handling errors

[여기](/exception-filters)에 오류처리(예: 예외 작업)에 대한 별도의 장이 있습니다.

#### Full resource sample

다음은 사용 가능한 여러 데코레이터를 사용하여 기본 컨트롤러를 만드는 예제입니다. 이 컨트롤러는 내부 데이터에 액세스하고 조작하는 몇가지 방법을 제공합니다.

```typescript
@@filename(cats.controller)
import { Controller, Get, Query, Post, Body, Put, Param, Delete } from '@nestjs/common';
import { CreateCatDto, UpdateCatDto, ListAllEntities } from './dto';

@Controller('cats')
export class CatsController {
  @Post()
  create(@Body() createCatDto: CreateCatDto) {
    return 'This action adds a new cat';
  }

  @Get()
  findAll(@Query() query: ListAllEntities) {
    return `This action returns all cats (limit: ${query.limit} items)`;
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return `This action returns a #${id} cat`;
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() updateCatDto: UpdateCatDto) {
    return `This action updates a #${id} cat`;
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return `This action removes a #${id} cat`;
  }
}
@@switch
import { Controller, Get, Query, Post, Body, Put, Param, Delete, Bind } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Post()
  @Bind(Body())
  create(createCatDto) {
    return 'This action adds a new cat';
  }

  @Get()
  @Bind(Query())
  findAll(query) {
    console.log(query);
    return `This action returns all cats (limit: ${query.limit} items)`;
  }

  @Get(':id')
  @Bind(Param('id'))
  findOne(id) {
    return `This action returns a #${id} cat`;
  }

  @Put(':id')
  @Bind(Param('id'), Body())
  update(id, updateCatDto) {
    return `This action updates a #${id} cat`;
  }

  @Delete(':id')
  @Bind(Param('id'))
  remove(id) {
    return `This action removes a #${id} cat`;
  }
}
```

> info **힌트** Nest CLI는 **모든 상용구(boilerplate) 코드**를 자동으로 생성하는 생성기(스키메틱)를 제공하여 이 모든 작업을 피하도록 돕고, 개발자 경험을 훨씬 더 간단하게 만들 수 있습니다. 이 기능에 대한 자세한 내용은 [여기](/recipes/crud-generator)를 참조하세요.

#### Getting up and running

위의 컨트롤러가 완전히 정의된 상태에서 Nest는 여전히 `CatsController`가 존재하는지 알지 못하므로 결과적으로 이 클래스의 인스턴스를 만들지 않습니다.

컨트롤러는 항상 모듈에 속하므로 `@Module()` 데코레이터 내에 `controllers` 배열을 포함합니다. 루트 `AppModule`을 제외한 다른 모듈을 아직 정의하지 않았으므로 이를 사용하여 `CatsController`를 소개합니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';

@Module({
  controllers: [CatsController],
})
export class AppModule {}
```

`@Module()` 데코레이터를 사용하여 모듈 클래스에 메타데이터를 첨부했으며 Nest는 이제 어떤 컨트롤러를 마운트해야 하는지 쉽게 반영할 수 있습니다.

<app-banner-shop></app-banner-shop>

#### Library-specific approach

지금까지 응답을 조작하는 Nest 표준방식에 대해 논의했습니다. 응답을 조작하는 두번째 방법은 라이브러리별 [응답객체](https://expressjs.com/en/api.html#res)를 사용하는 것입니다. 특정 응답객체를 삽입하려면 `@Res()` 데코레이터를 사용해야 합니다. 차이점을 보여주기 위해 `CatsController`를 다음과 같이 다시 작성해 보겠습니다.

```typescript
@@filename()
import { Controller, Get, Post, Res, HttpStatus } from '@nestjs/common';
import { Response } from 'express';

@Controller('cats')
export class CatsController {
  @Post()
  create(@Res() res: Response) {
    res.status(HttpStatus.CREATED).send();
  }

  @Get()
  findAll(@Res() res: Response) {
     res.status(HttpStatus.OK).json([]);
  }
}
@@switch
import { Controller, Get, Post, Bind, Res, Body, HttpStatus } from '@nestjs/common';

@Controller('cats')
export class CatsController {
  @Post()
  @Bind(Res(), Body())
  create(res, createCatDto) {
    res.status(HttpStatus.CREATED).send();
  }

  @Get()
  @Bind(Res())
  findAll(res) {
     res.status(HttpStatus.OK).json([]);
  }
}
```

이 접근 방식이 작동하고 실제로 응답객체(헤더 조작, 라이브러리별 기능등)에 대한 전체제어를 제공함으로써 어떤 방식으로든 더 많은 유연성을 허용하지만 주의해서 사용해야 합니다. 일반적으로 접근방식은 훨씬 덜 명확하고 몇가지 단점이 있습니다. 가장 큰 단점은 코드가 플랫폼에 종속되고(기본 라이브러리가 응답객체에 대해 다른 API를 가질 수 있기 때문에) 테스트하기가 더 어렵다는 것입니다(응답객체를 모의 처리해야하는 등).

또한 위의 예에서 인터셉터 및 `@HttpCode()`/`@Header()` 데코레이터와 같은 Nest 표준 응답처리에 의존하는 Nest 기능과의 호환성이 손실됩니다. 이 문제를 해결하려면 다음과 같이 `passthrough` 옵션을 `true`로 설정할 수 있습니다.

```typescript
@@filename()
@Get()
findAll(@Res({ passthrough: true }) res: Response) {
  res.status(HttpStatus.OK);
  return [];
}
@@switch
@Get()
@Bind(Res({ passthrough: true }))
findAll(res) {
  res.status(HttpStatus.OK);
  return [];
}
```

이제 네이티브 응답객체와 상호작용할 수 있지만(예: 특정조건에 따라 쿠키 또는 헤더설정) 나머지는 프레임워크에 맡깁니다.
