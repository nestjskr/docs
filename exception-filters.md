### Exception filters

Nest에는 애플리케이션 전체에서 처리되지 않은 모든 예외를 처리하는 **예외 레이어**가 내장되어 있습니다. 애플리케이션 코드에서 예외를 처리하지 않으면 이 레이어에서 예외를 포착하여 적절한 사용자 친화적인 응답을 자동으로 보냅니다.

<figure>
  <img src="/assets/Filter_1.png" />
</figure>

기본적으로 이 작업은 `HttpException` 유형의 예외(및 그 하위 클래스)를 처리하는 내장 **전역 예외필터**에 의해 수행됩니다. 예외가 **인식되지 않음**(`HttpException`도 `HttpException`에서 상속되는 클래스도 아님)이면 내장된 예외필터가 다음과 같은 기본 JSON 응답을 생성합니다.

```json
{
  "statusCode": 500,
  "message": "Internal server error"
}
```

#### Throwing standard exceptions

Nest는 `@nestjs/common` 패키지에서 노출된 내장 `HttpException` 클래스를 제공합니다. 일반적인 HTTP REST/GraphQL API 기반 애플리케이션의 경우 특정 오류 조건이 발생할 때 표준 HTTP 응답객체를 보내는 것이 가장 좋습니다.

예를 들어 `CatsController`에는 `findAll()` 메서드(`GET` 라우트 핸들러)가 있습니다. 이 라우트 핸들러가 어떤 이유로 예외를 던진다고 가정해 봅시다. 이를 증명하기 위해 다음과 같이 하드코딩합니다.

```typescript
@@filename(cats.controller)
@Get()
async findAll() {
  throw new HttpException('Forbidden', HttpStatus.FORBIDDEN);
}
```

> info **힌트** 여기서는 `HttpStatus`를 사용했습니다. 이것은 `@nestjs/common` 패키지에서 가져온 헬퍼 열거(enum)형입니다.

클라이언트가 이 엔드포인트를 호출하면 응답은 다음과 같습니다.

```json
{
  "statusCode": 403,
  "message": "Forbidden"
}
```

`HttpException` 생성자(constructor)는 응답을 결정하는 두개의 필수인수를 사용합니다.

- `response` 인수는 JSON 응답 본문을 정의합니다. 아래에 설명된대로 `string`또는 `object`일 수 있습니다.
- `status` 인수는 [HTTP 상태코드](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)를 정의합니다.

기본적으로 JSON 응답 본문에는 두가지 속성이 포함됩니다.

- `statusCode`: `status` 인수에 제공된 HTTP 상태코드가 기본값입니다.
- `message`: `status`에 따른 HTTP 오류에 대한 간단한 설명

JSON 응답 본문의 메시지 부분만 재정의하려면 `response` 인수에 문자열을 제공하세요. 전체 JSON 응답 본문을 재정의하려면 `response` 인수에 객체를 전달하세요. Nest는 객체를 직렬화하고 JSON 응답 본문으로 반환합니다.

두번째 생성자 인수 `status`는 유효한 HTTP 상태코드여야합니다. 모범사례는 `@nestjs/common`에서 가져온 `HttpStatus` 열거(enum)형을 사용하는 것입니다.

다음은 전체 응답 본문을 재정의하는 예입니다.

```typescript
@@filename(cats.controller)
@Get()
async findAll() {
  throw new HttpException({
    status: HttpStatus.FORBIDDEN,
    error: 'This is a custom message',
  }, HttpStatus.FORBIDDEN);
}
```

위의 내용을 사용하면 응답이 다음과 같이 표시됩니다.

```json
{
  "status": 403,
  "error": "This is a custom message"
}
```

#### Custom exceptions

대부분의 경우 커스텀 예외를 작성할 필요가 없으며 다음 섹션에 설명된대로 기본제공 Nest HTTP 예외를 사용할 수 있습니다. 커스텀 예외를 만들어야하는 경우 커스텀 예외가 기본 `HttpException` 클래스에서 상속되는 고유한 **예외 계층**을 만드는 것이 좋습니다. 이 접근방식을 사용하면 Nest가 예외를 인식하고 오류 응답을 자동으로 처리합니다. 이러한 커스텀 예외를 구현해 보겠습니다.

```typescript
@@filename(forbidden.exception)
export class ForbiddenException extends HttpException {
  constructor() {
    super('Forbidden', HttpStatus.FORBIDDEN);
  }
}
```

`ForbiddenException`은 기본 `HttpException`을 확장하므로 내장된 예외 핸들러와 원활하게 작동하므로 `findAll()` 메서드 내에서 사용할 수 있습니다.

```typescript
@@filename(cats.controller)
@Get()
async findAll() {
  throw new ForbiddenException();
}
```

#### Built-in HTTP exceptions

Nest는 기본 `HttpException`에서 상속되는 표준 예외 집합을 제공합니다. 이들은 `@nestjs/common` 패키지에서 노출되며 가장 일반적인 HTTP 예외중 대부분을 나타냅니다.

- `BadRequestException`
- `UnauthorizedException`
- `NotFoundException`
- `ForbiddenException`
- `NotAcceptableException`
- `RequestTimeoutException`
- `ConflictException`
- `GoneException`
- `HttpVersionNotSupportedException`
- `PayloadTooLargeException`
- `UnsupportedMediaTypeException`
- `UnprocessableEntityException`
- `InternalServerErrorException`
- `NotImplementedException`
- `ImATeapotException`
- `MethodNotAllowedException`
- `BadGatewayException`
- `ServiceUnavailableException`
- `GatewayTimeoutException`
- `PreconditionFailedException`

#### Exception filters

기본(내장) 예외필터가 자동으로 많은 경우를 처리할 수 있지만 예외 레이어에 대한 **완전한 제어**를 원할 수 있습니다. 예를 들어 로깅을 추가하거나 일부 동적요인을 기반으로 다른 JSON 스키마를 사용할 수 있습니다. **예외필터**는 정확히 이러한 목적을 위해 설계되었습니다. 이를 통해 정확한 제어 흐름과 클라이언트로 다시 전송되는 응답 내용을 제어할 수 있습니다.

`HttpException` 클래스의 인스턴스인 예외를 포착하고 이에 대한 커스텀 응답 로직을 구현하는 예외필터를 만들어 보겠습니다. 이렇게하려면 기본플랫폼 `Request`및 `Response`객체에 액세스해야 합니다. `Request` 객체에 액세스하여 원래 `url`을 추출하여 로깅 정보에 포함할 수 있습니다. `Response`객체를 사용하여 `response.json()`메서드를 사용하여 전송되는 응답을 직접 제어합니다.

```typescript
@@filename(http-exception.filter)
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';
import { Request, Response } from 'express';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();

    response
      .status(status)
      .json({
        statusCode: status,
        timestamp: new Date().toISOString(),
        path: request.url,
      });
  }
}
@@switch
import { Catch, HttpException } from '@nestjs/common';

@Catch(HttpException)
export class HttpExceptionFilter {
  catch(exception, host) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();
    const status = exception.getStatus();

    response
      .status(status)
      .json({
        statusCode: status,
        timestamp: new Date().toISOString(),
        path: request.url,
      });
  }
}
```

> info **힌트** 모든 예외필터는 일반 `ExceptionFilter<T>` 인터페이스를 구현해야 합니다. 이를 위해서는 표시된 서명과 함께 `catch(exception: T, host: ArgumentsHost)` 메서드를 제공해야 합니다. `T`는 예외 타입을 나타냅니다.

`@Catch(HttpException)` 데코레이터는 필요한 메타데이터를 예외필터에 바인딩하여 이 특정 필터가 `HttpException` 타입의 예외만 찾고있다는 것을 Nest에 알립니다. `@Catch()` 데코레이터는 단일 매개변수 또는 쉼표로 구분된 목록을 사용할 수 있습니다. 이를 통해 한번에 여러 타입의 예외에 대한 필터를 설정할 수 있습니다.

#### Arguments host

`catch()` 메서드의 매개변수를 살펴 보겠습니다. `exception` 매개변수는 현재 처리중인 예외 객체입니다. `host` 매개변수는 `ArgumentsHost` 객체입니다. `ArgumentsHost`는 [실행 컨텍스트 장](/fundamentals/execution-context)\* 에서 자세히 살펴볼 강력한 유틸리티 객체입니다. 이 코드 샘플에서는 이를 사용하여 원래 요청 핸들러(예외가 발생한 컨트롤러에서)로 전달되는 `Request` 및 `Response` 객체에 대한 참조를 얻습니다. 이 코드 샘플에서는 원하는 `Request` 및 `Response` 객체를 가져오기 위해 `ArgumentsHost`에 몇가지 헬퍼 메서드를 사용했습니다. [여기](/fundamentals/execution-context)에서 `ArgumentsHost`대해 자세히 알아보세요.

\*이 수준의 추상화에 대한 이유는 `ArgumentsHost`가 모든 컨텍스트(예: 현재 작업중인 HTTP 서버 컨텍스트뿐만 아니라 마이크로서비스 및 WebSockets)에서 작동하기 때문입니다. 실행 컨텍스트 장에서는 `ArgumentsHost`와 그 헬퍼 함수를 사용하여 **모든** 실행 컨텍스트에 대한 적절한 [기본 인수](/fundamentals/execution-context#host-methods)에 액세스하는 방법을 알아봅니다. 이를 통해 모든 컨텍스트에서 작동하는 일반 예외필터를 작성할 수 있습니다.

<app-banner-courses></app-banner-courses>

#### Binding filters

새로운 `HttpExceptionFilter`를 `CatsController`의 `create()`메서드에 연결해 보겠습니다.

```typescript
@@filename(cats.controller)
@Post()
@UseFilters(new HttpExceptionFilter())
async create(@Body() createCatDto: CreateCatDto) {
  throw new ForbiddenException();
}
@@switch
@Post()
@UseFilters(new HttpExceptionFilter())
@Bind(Body())
async create(createCatDto) {
  throw new ForbiddenException();
}
```

> info **힌트** `@UseFilters()` 데코레이터는 `@nestjs/common` 패키지에서 가져옵니다.

여기서는 `@UseFilters()` 데코레이터를 사용했습니다. `@Catch()` 데코레이터와 유사하게 단일 필터 인스턴스 또는 쉼표로 구분된 필터 인스턴스 목록을 사용할 수 있습니다. 여기에서 `HttpExceptionFilter`의 인스턴스를 생성했습니다. 또는 인스턴스 대신 클래스를 전달하여 프레임워크에 대한 인스턴스화 책임을 남겨 **종속성 주입**을 활성화할 수 있습니다.

```typescript
@@filename(cats.controller)
@Post()
@UseFilters(HttpExceptionFilter)
async create(@Body() createCatDto: CreateCatDto) {
  throw new ForbiddenException();
}
@@switch
@Post()
@UseFilters(HttpExceptionFilter)
@Bind(Body())
async create(createCatDto) {
  throw new ForbiddenException();
}
```

> info **힌트** 가능한 경우 인스턴스 대신 클래스를 사용하여 필터를 적용하는 것이 좋습니다. Nest는 전체 모듈에서 동일한 클래스의 인스턴스를 쉽게 재사용할 수 있으므로 **메모리 사용량**을 줄입니다.

위의 예에서 `HttpExceptionFilter`는 단일 `create()` 라우드 핸들러에만 적용되어 메소드 범위가 됩니다. 예외필터는 메서드 범위, 컨트롤러 범위 또는 전역 범위등 다양한 수준으로 범위를 지정할 수 있습니다. 예를 들어 필터를 컨트롤러 범위로 설정하려면 다음을 수행합니다.

```typescript
@@filename(cats.controller)
@UseFilters(new HttpExceptionFilter())
export class CatsController {}
```

이 구성은 `CatsController`내에 정의된 모든 라우트 핸들러에 대해 `HttpExceptionFilter`를 설정합니다.

전역범위 필터를 만들려면 다음을 수행합니다.

```typescript
@@filename(main)
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalFilters(new HttpExceptionFilter());
  await app.listen(3000);
}
bootstrap();
```

> warning **경고** `useGlobalFilters()` 메서드는 게이트웨이 또는 하이브리드 애플리케이션에 대한 필터를 설정하지 않습니다.

전역범위 필터는 모든 컨트롤러 및 모든 러유투 핸들러에 대해 전체 애플리케이션에서 사용됩니다. 의존성 주입과 관련하여 모듈 외부에서 등록된 전역 필터(위의 예에서와 같이 `useGlobalFilters()`사용)는 모듈의 컨텍스트 외부에서 수행되므로 종속성을 주입할 수 없습니다. 이 문제를 해결하기 위해 다음 구성을 사용하여 **모든 모듈에서 직접** 전역범위 필터를 등록할 수 있습니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { APP_FILTER } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_FILTER,
      useClass: HttpExceptionFilter,
    },
  ],
})
export class AppModule {}
```

> info **힌트** 이 접근방식을 사용하여 필터에 대한 종속성 주입을 수행할 때 이 구성이 사용되는 모듈에 관계없이 필터는 실제로 전역이라는 점에 유의하십시오. 어디에서 해야합니까? 필터(위 예에서는 `HttpExceptionFilter`)가 정의된 모듈을 선택합니다. 또한 `useClass`가 커스텀 프로바이더 등록을 처리하는 유일한 방법은 아닙니다. [여기](/fundamentals/custom-providers)에서 자세히 알아보세요.

이 기술을 사용하여 필요한만큼 필터를 추가할 수 있습니다. 프로바이더 배열에 각각을 추가하기만 하면 됩니다.

#### Catch everything

처리되지 않은 **모든** 예외를 잡으려면(예외 유형에 관계없이) `@Catch()` 데코레이터의 매개변수 목록을 비워둡니다(예: `@Catch()`).

```typescript
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
  HttpStatus,
} from '@nestjs/common';

@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();

    const status =
      exception instanceof HttpException
        ? exception.getStatus()
        : HttpStatus.INTERNAL_SERVER_ERROR;

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
    });
  }
}
```

위의 예에서 필터는 타입(클래스)에 관계없이 발생한 각 예외를 포착합니다.

#### Inheritance

일반적으로 애플리케이션 요구사항을 충족하도록 제작된 완전히 커스텀 예외필터를 만듭니다. 그러나 기본 제공되는 디폴트 **전역 예외필터**를 확장하고 특정요인에 따라 동작을 재정의하려는 경우 사용사례가 있을 수 있습니다.

예외 처리를 기본필터에 위임하려면 `BaseExceptionFilter`를 확장하고 상속된 `catch()` 메서드를 호출해야 합니다.

```typescript
@@filename(all-exceptions.filter)
import { Catch, ArgumentsHost } from '@nestjs/common';
import { BaseExceptionFilter } from '@nestjs/core';

@Catch()
export class AllExceptionsFilter extends BaseExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    super.catch(exception, host);
  }
}
@@switch
import { Catch } from '@nestjs/common';
import { BaseExceptionFilter } from '@nestjs/core';

@Catch()
export class AllExceptionsFilter extends BaseExceptionFilter {
  catch(exception, host) {
    super.catch(exception, host);
  }
}
```

> warning **경고** `BaseExceptionFilter`를 확장하는 메서드 범위 및 컨트롤러 범위 필터는 `new`로 인스턴스화 하면 안됩니다. 대신 프레임워크가 자동으로 인스턴스화하도록 합니다.

위의 구현은 접근방식을 보여주는 셸일뿐입니다. 확장된 예외필터의 구현에는 맞춤형 **비즈니스** 로직(예: 다양한 조건 처리)이 포함됩니다.

전역필터는 기본필터를 **확장할 수 있습니다**. 이는 두가지 방법중 하나로 수행할 수 있습니다.

첫번째 방법은 커스텀 전역필터를 인스턴스화할 때 `HttpServer` 참조를 삽입하는 것입니다.

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const { httpAdapter } = app.get(HttpAdapterHost);
  app.useGlobalFilters(new AllExceptionsFilter(httpAdapter));

  await app.listen(3000);
}
bootstrap();
```

두번째 방법은 여기에 [표시된대로](/exception-filters#binding-filters) `APP_FILTER` 토큰을 사용하는 것입니다.
