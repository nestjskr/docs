### Validation

웹 애플리케이션으로 전송된 데이터의 정확성을 검증하는 것이 가장 좋습니다. 수신 요청을 자동으로 검증하기 위해 Nest는 즉시 사용할 수 있는 여러 파이프를 제공합니다.

- `ValidationPipe`
- `ParseIntPipe`
- `ParseBoolPipe`
- `ParseArrayPipe`
- `ParseUUIDPipe`

`ValidationPipe`는 강력한 [class-validator](https://github.com/typestack/class-validator) 패키지와 선언적 유효성 검사 데코레이터를 사용합니다. `ValidationPipe`는 들어오는 모든 클라이언트 페이로드에 대해 유효성 검사 규칙을 적용하는 편리한 접근 방식을 제공합니다. 여기서 특정 규칙은 각 모듈의 로컬 클래스/DTO 선언에 간단한 주석으로 선언됩니다.

#### Overview

[파이프](/pipes) 장에서 간단한 파이프를 만들고 컨트롤러, 메서드 또는 글로벌 앱에 바인딩하여 프로세스가 어떻게 작동하는지 설명하는 과정을 살펴 보았습니다. 이 장의 주제를 가장 잘 이해하려면 해당 장을 검토하십시오. 여기서는 `ValidationPipe`의 다양한 **실제** 사용 사례에 초점을 맞추고 고급 사용자 지정 기능중 일부를 사용하는 방법을 보여줍니다.

#### Using the built-in ValidationPipe

> info **힌트** `ValidationPipe`는 `@nestjs/common` 패키지에서 내보내집니다.

이 파이프는 `class-validator` 및 `class-transformer` 라이브러리를 사용하기 때문에 사용 가능한 많은 옵션이 있습니다. 파이프에 전달된 구성 객체를 통해 이러한 설정을 구성합니다. 다음은 기본 제공 옵션입니다.

```typescript
export interface ValidationPipeOptions extends ValidatorOptions {
  transform?: boolean;
  disableErrorMessages?: boolean;
  exceptionFactory?: (errors: ValidationError[]) => any;
}
```

이 외에도 모든 `class-validator` 옵션 (`ValidatorOptions` 인터페이스에서 상속됨)을 사용할 수 있습니다.

<table>
  <tr>
    <th>Option</th>
    <th>Type</th>
    <th>Description</th>
  </tr>
  <tr>
    <td><code>skipMissingProperties</code></td>
    <td><code>boolean</code></td>
    <td>true로 설정하면 유효성 검사기가 객체 유효성 검사에 누락된 모든 속성의 유효성 검사를 건너뜁니다.</td>
  </tr>
  <tr>
    <td><code>whitelist</code></td>
    <td><code>boolean</code></td>
    <td>true로 설정되면 유효성 검사기는 유효성 검사 데코레이터를 사용하지 않는 속성의 유효성 검사(반환 된) 객체를 제거합니다.</td>
  </tr>
  <tr>
    <td><code>forbidNonWhitelisted</code></td>
    <td><code>boolean</code></td>
    <td>true로 설정하면 화이트리스트에 없는 속성 검사기를 제거하는 대신 예외가 발생합니다.</td>
  </tr>
  <tr>
    <td><code>forbidUnknownValues</code></td>
    <td><code>boolean</code></td>
    <td>true로 설정하면 알 수 없는 개체의 유효성을 검사하려는 시도가 즉시 실패합니다.</td>
  </tr>
  <tr>
    <td><code>disableErrorMessages</code></td>
    <td><code>boolean</code></td>
    <td>true로 설정하면 유효성 검사 오류가 클라이언트에 반환되지 않습니다.</td>
  </tr>
  <tr>
    <td><code>errorHttpStatusCode</code></td>
    <td><code>number</code></td>
    <td>이 설정을 사용하면 오류 발생시 사용할 예외 유형을 지정할 수 있습니다. 기본적으로 <code>BadRequestException</code>이 발생합니다.</td>
  </tr>
  <tr>
    <td><code>exceptionFactory</code></td>
    <td><code>Function</code></td>
    <td>유효성 검사 오류의 배열을 가져오고 throw할 예외 개체를 반환합니다.</td>
  </tr>
  <tr>
    <td><code>groups</code></td>
    <td><code>string[]</code></td>
    <td>개체의 유효성을 검사하는 동안 사용할 그룹입니다.</td>
  </tr>
  <tr>
    <td><code>dismissDefaultMessages</code></td>
    <td><code>boolean</code></td>
    <td>true로 설정하면 유효성 검사에서 기본 메시지를 사용하지 않습니다. 오류 메시지는 항상 <code>undefined</code>입니다.
       명시 적으로 설정되지 않았습니다.</td>
  </tr>
  <tr>
    <td><code>validationError.target</code></td>
    <td><code>boolean</code></td>
    <td><code>ValidationError</code>에서 대상을 노출해야 하는지 여부를 나타냅니다.</td>
  </tr>
  <tr>
    <td><code>validationError.value</code></td>
    <td><code>boolean</code></td>
    <td>검증된 값이 <code>ValidationError</code>에 노출되어야 하는지 여부를 나타냅니다.</td>
  </tr>
</table>

> info **알림** `class-validator` 패키지에 대한 자세한 내용은 [repository](https://github.com/typestack/class-validator)에서 확인하세요.


#### Auto-validation

애플리케이션 수준에서 `ValidationPipe`를 바인딩하여 시작하여 모든 엔드 포인트가 잘못된 데이터를 수신하지 않도록 보호합니다.

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe());
  await app.listen(3000);
}
bootstrap();
```

파이프를 테스트하기 위해 기본 엔드포인트를 만들어 보겠습니다.

```typescript
@Post()
create(@Body() createUserDto: CreateUserDto) {
  return 'This action adds a new user';
}
```

> info **힌트** TypeScript는 **제네릭 또는 인터페이스**에 대한 메타데이터를 저장하지 않으므로 DTO에서 사용할 때 `ValidationPipe`가 들어오는 데이터의 유효성을 제대로 검사하지 못할 수 있습니다. 이러한 이유로 DTO에서 구체적인 클래스를 사용하는 것이 좋습니다.

> info **힌트** DTO를 가져올 때 런타임시 삭제되므로 타입 전용 가져오기를 사용할 수 없습니다. 즉, `import type {{ '{' }} CreateUserDto {{ '}' }}` 대신 `import {{ '{' }} CreateUserDto {{ '}' }}`를 기억하세요.

이제 `CreateUserDto`에 몇가지 유효성 검사 규칙을 추가할 수 있습니다. 이 작업은 [여기](https://github.com/typestack/class-validator#validation-decorators)에 자세히 설명된 `class-validator` 패키지에서 제공하는 데코레이터를 사용하여 수행합니다. 이러한 방식으로 `CreateUserDto`를 사용하는 모든 라우트는 이러한 유효성 검사 규칙을 자동으로 적용합니다.

```typescript
import { IsEmail, IsNotEmpty } from 'class-validator';

export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsNotEmpty()
  password: string;
}
```

이러한 규칙을 사용하면 요청이 요청 본문에 잘못된 `email` 속성이 있는 엔드포인트에 도달하면 애플리케이션이 다음 응답 본문과 함께 `400 Bad Request` 코드로 자동 응답합니다.

```json
{
  "statusCode": 400,
  "error": "Bad Request",
  "message": ["email must be an email"]
}
```

요청 본문의 유효성을 검사하는 것 외에도 `ValidationPipe`를 다른 요청 객체 속성과 함께 사용할 수도 있습니다. 엔드포인트 경로에서 `:id`를 받아들이고 싶다고 가정 해보십시오. 이 요청 매개 변수에 숫자만 허용되도록 하기 위해 다음 구성을 사용할 수 있습니다.

```typescript
@Get(':id')
findOne(@Param() params: FindOneParams) {
  return 'This action returns a user';
}
```

`FindOneParams`는 DTO와 마찬가지로 `class-validator`를 사용하여 유효성 검사 규칙을 정의하는 클래스입니다. 다음과 같이 표시됩니다.

```typescript
import { IsNumberString } from 'class-validator';

export class FindOneParams {
  @IsNumberString()
  id: number;
}
```

#### Disable detailed errors

오류 메시지는 요청에서 무엇이 잘못되었는지 설명하는 데 도움이 될 수 있습니다. 그러나 일부 프로덕션 환경에서는 자세한 오류를 비활성화하는 것을 선호합니다. 옵션 객체를 `ValidationPipe`에 전달하면 됩니다.

```typescript
app.useGlobalPipes(
  new ValidationPipe({
    disableErrorMessages: true,
  }),
);
```

결과적으로 자세한 오류 메시지가 응답 본문에 표시되지 않습니다.

#### Stripping properties

우리의 `ValidationPipe`는 메소드 핸들러에서 수신하지 않아야 하는 속성을 필터링할 수도 있습니다. 이 경우 허용되는 속성을 **화이트리스트**할 수 있으며, 화이트리스트에 포함되지 않은 모든 속성은 결과 개체에서 자동으로 제거됩니다. 예를 들어 핸들러가 `email`및 `password`속성을 예상하지만 요청에 `age` 속성도 포함된 경우 이 속성은 결과 DTO에서 자동으로 제거될 수 있습니다. 이러한 동작을 활성화하려면 `whitelist`를 `true`로 설정하십시오.

```typescript
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,
  }),
);
```

true로 설정하면 화이트리스트에 없는 속성(유효성 검사 클래스에 데코레이터가 없는 속성)이 자동으로 제거됩니다.

또는 화이트리스트에 없는 속성이 있는 경우 요청 처리를 중지하고 사용자에게 오류 응답을 반환할 수 있습니다. 이를 활성화하려면 `whitelist`를 `true`로 설정하고 `forbidNonWhitelisted` 옵션 속성을 `true`로 설정합니다.

<app-banner-courses></app-banner-courses>

#### Transform payload objects

네트워크를 통해 들어오는 페이로드는 일반 JavaScript 객체입니다. `ValidationPipe`는 페이로드를 DTO 클래스에 따라 타입이 지정된 객체로 자동 변환할 수 있습니다. 자동 변환을 사용하려면 `transform`을 `true`로 설정하세요. 이것은 메서드 수준에서 수행할 수 있습니다.

```typescript
@@filename(cats.controller)
@Post()
@UsePipes(new ValidationPipe({ transform: true }))
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

이 동작을 전역적으로 활성화하려면 전역 파이프에서 옵션을 설정하십시오.

```typescript
app.useGlobalPipes(
  new ValidationPipe({
    transform: true,
  }),
);
```

자동 변환 옵션이 활성화 된 상태에서 `ValidationPipe`는 기본 유형의 변환도 수행합니다. 다음 예에서 `findOne()` 메소드는 추출 된 `id` 경로 매개변수를 나타내는 하나의 인수를 사용합니다.

```typescript
@Get(':id')
findOne(@Param('id') id: number) {
  console.log(typeof id === 'number'); // true
  return 'This action returns a user';
}
```

기본적으로 모든 경로 매개 변수와 쿼리 매개 변수는 네트워크를 통해 `string`으로 제공됩니다. 위의 예에서 `id` 유형을 `number` (메서드 서명에서)로 지정했습니다. 따라서 `ValidationPipe`는 문자열 식별자를 숫자로 자동 변환하려고 시도합니다.

#### Explicit conversion

위 섹션에서는 `ValidationPipe`가 예상 유형에 따라 쿼리 및 경로 매개 변수를 암시적으로 변환하는 방법을 보여주었습니다. 그러나 이 기능을 사용하려면 자동 변환을 활성화해야합니다.

또는 (자동 변환이 비활성화 된 상태에서) `ParseIntPipe` 또는 `ParseBoolPipe`를 사용하여 명시적으로 값을 캐스팅할 수 있습니다 (앞서 언급했듯이 모든 경로 매개변수와 쿼리 매개변수는 기본적으로 `string`로 네트워크를 통해 제공되므로 `ParseStringPipe`가 필요하지 않습니다.).

```typescript
@Get(':id')
findOne(
  @Param('id', ParseIntPipe) id: number,
  @Query('sort', ParseBoolPipe) sort: boolean,
) {
  console.log(typeof id === 'number'); // true
  console.log(typeof sort === 'boolean'); // true
  return 'This action returns a user';
}
```

> info **힌트** `ParseIntPipe` 및 `ParseBoolPipe`는 `@nestjs/common` 패키지에서 내보내집니다.

#### Mapped types

**CRUD**(만들기/읽기/업데이트/삭제)와 같은 기능을 구축할 때 기본 항목 유형에 대한 변형을 구성하는 것이 종종 유용합니다. Nest는 이 작업을 보다 편리하게 만들기 위해 유형 변환을 수행하는 여러 유틸리티 함수를 제공합니다.

> warning **경고** 애플리케이션에서 `@nestjs/swagger` 패키지를 사용하는 경우 매핑된 유형에 대한 자세한 내용은 [이 장](/openapi/mapped-types)을 참조하세요. 마찬가지로 `@nestjs/graphql` 패키지를 사용하는 경우 [이 장](/graphql/mapped-types)을 참조하십시오. 두 패키지 모두 유형에 크게 의존하므로 다른 가져오기를 사용해야합니다. 따라서 `@nestjs/mapped-types` (앱 유형에 따라 적절한 `@nestjs/swagger` 또는 `@nestjs/graphql` 대신)를 사용했다면 문서화되지 않은 다양한 측면에 직면할 수 있습니다.

입력 유효성 검사 타입(DTO라고도 함)을 빌드할 때 동일한 유형에서 **생성** 및 **업데이트** 변형을 빌드하는 것이 유용한 경우가 많습니다. 예를 들어 **create** 변형에는 모든 필드가 필요할 수 있지만 **update** 변형은 모든 필드를 선택 사항으로 만들 수 있습니다.

Nest는 이 작업을 더 쉽게 만들고 상용구를 최소화하기 위해 `PartialType()` 유틸리티 함수를 제공합니다.

`PartialType()` 함수는 입력 유형의 모든 속성이 선택 사항으로 설정된 유형 (클래스)을 반환합니다. 예를 들어 다음과 같은 **create** 유형이 있다고 가정합니다.

```typescript
export class CreateCatDto {
  name: string;
  age: number;
  breed: string;
}
```

기본적으로 이러한 필드는 모두 필수입니다. 동일한 필드를 사용하되 각 필드가 선택사항인 유형을 만들려면 클래스 참조(`CreateCatDto`)를 인수로 전달하는 `PartialType()`을 사용합니다.

```typescript
export class UpdateCatDto extends PartialType(CreateCatDto) {}
```

> info **힌트** `PartialType()` 함수는 `@nestjs/mapped-types` 패키지에서 가져옵니다.

`PickType()` 함수는 입력 유형에서 속성 집합을 선택하여 새로운 유형(클래스)을 생성합니다. 예를 들어 다음과 같은 유형으로 시작한다고 가정합니다.


```typescript
export class CreateCatDto {
  name: string;
  age: number;
  breed: string;
}
```

`PickType()`유틸리티 함수를 사용하여 이 클래스에서 속성 집합을 선택할 수 있습니다.

```typescript
export class UpdateCatAgeDto extends PickType(CreateCatDto, ['age'] as const) {}
```

> info **힌트** `PickType()` 함수는 `@nestjs/mapped-types` 패키지에서 가져옵니다.

`OmitType()` 함수는 입력 타입에서 모든 속성을 선택한 다음 특정 키 세트를 제거하여 타입을 구성합니다. 예를 들어 다음과 같은 타입으로 시작한다고 가정합니다.


```typescript
export class CreateCatDto {
  name: string;
  age: number;
  breed: string;
}
```

아래와 같이 `name`을 **제외**한 모든 속성을 가진 파생 타입을 생성할 수 있습니다. 이 구조에서 `OmitType`의 두번째 인수는 속성 이름의 배열입니다.

```typescript
export class UpdateCatDto extends OmitType(CreateCatDto, ['name'] as const) {}
```

> info **힌트** `OmitType()` 함수는 `@nestjs/mapped-types` 패키지에서 가져옵니다.

`IntersectionType()` 함수는 두 타입을 하나의 새로운 타입 (클래스)으로 결합합니다. 예를 들어 다음과 같은 두 가지 타입으로 시작한다고 가정합니다.

```typescript
export class CreateCatDto {
  name: string;
  breed: string;
}

export class AdditionalCatInfo {
  color: string;
}
```

두 타입의 모든 속성을 결합하는 새 타입을 생성할 수 있습니다.

```typescript
export class UpdateCatDto extends IntersectionType(
  CreateCatDto,
  AdditionalCatInfo,
) {}
```

> info **힌트** `IntersectionType()` 함수는 `@nestjs/mapped-types` 패키지에서 가져옵니다.

타입 매핑 유틸리티 함수는 구성 가능합니다. 예를 들어, 다음은 `name`을 제외한 `CreateCatDto` 타입의 모든 속성을 가진 타입 (클래스)을 생성하며 이러한 속성은 선택 사항으로 설정됩니다.

```typescript
export class UpdateCatDto extends PartialType(
  OmitType(CreateCatDto, ['name'] as const),
) {}
```

#### Parsing and validating arrays

TypeScript는 제네릭 또는 인터페이스에 대한 메타 데이터를 저장하지 않으므로 DTO에서 사용할 때 `ValidationPipe`가 들어오는 데이터의 유효성을 제대로 검사하지 못할 수 있습니다. 예를 들어, 다음 코드에서 `createUserDtos`는 올바르게 검증되지 않습니다.

```typescript
@Post()
createBulk(@Body() createUserDtos: CreateUserDto[]) {
  return 'This action adds new users';
}
```

배열의 유효성을 검사하려면 배열을 래핑하는 속성을 포함하는 전용 클래스를 만들거나 `ParseArrayPipe`를 사용합니다.

```typescript
@Post()
createBulk(
  @Body(new ParseArrayPipe({ items: CreateUserDto }))
  createUserDtos: CreateUserDto[],
) {
  return 'This action adds new users';
}
```

또한 `ParseArrayPipe`는 쿼리 매개변수를 구문 분석할 때 유용할 수 있습니다. 쿼리 매개변수로 전달된 식별자를 기반으로 사용자를 반환하는 `findByIds()` 메서드를 살펴 보겠습니다.

```typescript
@Get()
findByIds(
  @Query('id', new ParseArrayPipe({ items: Number, separator: ',' }))
  ids: number[],
) {
  return 'This action returns users by ids';
}
```

이 구성은 다음과 같이 HTTP `GET` 요청에서 들어오는 쿼리 매개변수의 유효성을 검사합니다.

```bash
GET /?ids=1,2,3
```

#### WebSockets and Microservices

이 장에서는 HTTP 스타일 애플리케이션 (예: Express 또는 Fastify)을 사용하는 예제를 보여 주지만 `ValidationPipe`는 사용되는 전송 방법에 관계없이 WebSocket 및 마이크로 서비스에 대해 동일하게 작동합니다.

#### Learn more

`class-validator` 패키지에서 제공하는 커스텀 유효성 검사기, 오류 메시지, 사용 가능한 데코레이터에 대한 자세한 내용은 [여기](https://github.com/typestack/class-validator)를 참조하세요.