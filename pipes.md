### Pipes

파이프는 `@Injectable()` 데코레이터로 주석이 달린 클래스입니다. 파이프는 `PipeTransform` 인터페이스를 구현해야 합니다.

<figure>
  <img src="/assets/Pipe_1.png" />
</figure>

파이프에는 두가지 일반적인 사용사례가 있습니다.

- **변환**(transformation): 입력 데이터를 원하는 형식으로 변환(예: 문자열에서 정수로)
- **유효성 검사**(validation): 입력 데이터를 평가하고 유효한 경우 변경하지 않고 전달합니다. 그렇지 않으면 데이터가 올바르지 않을 때 예외를 발생시킵니다.

두경우 모두 파이프는 [컨트롤러 라우트 핸들러](/controllers#route-parameters)가 처리하는 `인수(arguments)`에서 작동합니다. Nest는 메소드가 호출되기 직전에 파이프를 삽입하고 파이프는 메소드로 향하는 인수를 수신하고 이에 대해 작동합니다. 모든 변환 또는 유효성 검사 작업은 해당 시간에 발생하며 그 후 라우트 핸들러가(잠재적으로) 변환된 인수와 함께 호출됩니다.

Nest에는 기본적으로 사용할 수 있는 여러 내장된 파이프가 함께 제공됩니다. 고유한 커스텀 파이프를 만들 수도 있습니다. 이 장에서는 내장된 파이프를 소개하고 라우트 핸들러에 바인딩하는 방법을 보여줍니다. 그런 다음 몇가지 커스텀 파이프를 조사하여, 처음부터 빌드하는 방법을 보여줍니다.

> info **힌트** 파이프는 예외 영역내에서 실행됩니다. 이것은 파이프가 예외를 던질 때 예외 계층(전역 예외필터 및 현재 컨텍스트에 적용되는 모든 [예외필터](/exception-filters))에 의해 처리된다는 것을 의미합니다. 위의 내용을 고려할 때 파이프에서 예외가 발생하면 이후에 컨트롤러 메서드가 실행되지 않음을 분명히해야 합니다. 이는 시스템 경계의 외부 소스에서 애플리케이션으로 들어오는 데이터를 검증하기 위한 모범사례 기술을 제공합니다.

#### Built-in pipes

Nest에는 즉시 사용할 수 있는 6 개의 파이프가 함께 제공됩니다.

- `ValidationPipe`
- `ParseIntPipe`
- `ParseBoolPipe`
- `ParseArrayPipe`
- `ParseUUIDPipe`
- `DefaultValuePipe`

`@nestjs/common` 패키지에서 내보내집니다.

`ParseIntPipe` 사용에 대해 간단히 살펴 보겠습니다. 이것은 파이프가 메소드 핸들러 매개변수가 자바스크립트 정수로 변환되도록 하는 **변환** 사용 사례의 예입니다(또는 변환에 실패하면 예외가 발생함). 이 장의 뒷부분에서 `ParseIntPipe`에 대한 간단한 커스텀 구현을 보여줄 것입니다. 아래의 예제 기술은 다른 내장 변환 파이프에도 적용됩니다(`ParseBoolPipe`, `ParseArrayPipe` 및 `ParseUUIDPipe`, 이 장에서 `Parse*` 파이프라고 함).

#### Binding pipes

파이프를 사용하려면 파이프 클래스의 인스턴스를 적절한 컨텍스트에 바인딩해야 합니다. `ParseIntPipe` 예제에서 파이프를 특정 라우트 핸들러 메소드와 연관시키고 메소드가 호출되기 전에 실행되는지 확인하려고 합니다. 메서드 매개변수 수준에서 파이프를 바인딩하는 것으로 참조할 다음 구성으로 이를 수행합니다.

```typescript
@Get(':id')
async findOne(@Param('id', ParseIntPipe) id: number) {
  return this.catsService.findOne(id);
}
```

이렇게 하면 다음 두조건중 하나가 참이 됩니다. `findOne()` 메서드에서 수신하는 매개변수가 숫자(`this.catsService.findOne()` 호출에서 예상한대로)이거나 예외는 다음과 같습니다. 라우트 핸들러가 호출되기 전에 발생합니다.

예를 들어 라우터가 다음과 같이 호출되었다고 가정합니다.

```bash
GET localhost:3000/abc
```


```json
{
  "statusCode": 400,
  "message": "Validation failed (numeric string is expected)",
  "error": "Bad Request"
}
```

예외는 `findOne()` 메서드의 본문이 실행되지 않도록 합니다.

위의 예에서는 인스턴스가 아닌 클래스(`ParseIntPipe`)를 전달하여 인스턴스화를 프레임워크에 맡기고 종속성 주입을 활성화합니다. 파이프 및 가드와 마찬가지로 대신 내부 인스턴스를 전달할 수 있습니다. 내부(in-place) 인스턴스를 전달하는 것은 옵션을 전달하여 내장 파이프의 동작을 커스텀하려는 경우 유용합니다.

```typescript
@Get(':id')
async findOne(
  @Param('id', new ParseIntPipe({ errorHttpStatusCode: HttpStatus.NOT_ACCEPTABLE }))
  id: number,
) {
  return this.catsService.findOne(id);
}
```

다른 변환 파이프(모든 **Parse\*** 파이프)를 바인딩하는 방법도 비슷합니다. 이러한 파이프는 모두 라우트 매개변수, 쿼리 문자열 매개변수 및 요청 본문 값을 확인하는 컨텍스트에서 작동합니다.

예를 들어 쿼리 문자열 매개변수를 사용하는 경우:

```typescript
@Get()
async findOne(@Query('id', ParseIntPipe) id: number) {
  return this.catsService.findOne(id);
}
```

다음은 `ParseUUIDPipe`를 사용하여 문자열 매개변수를 구문분석하고 UUID인지 확인하는 예입니다.

```typescript
@@filename()
@Get(':uuid')
async findOne(@Param('uuid', new ParseUUIDPipe()) uuid: string) {
  return this.catsService.findOne(uuid);
}
@@switch
@Get(':uuid')
@Bind(Param('uuid', new ParseUUIDPipe()))
async findOne(uuid) {
  return this.catsService.findOne(uuid);
}
```

> info **힌트** `ParseUUIDPipe()`를 사용하는 경우 버전 3, 4 또는 5에서 UUID를 구문분석합니다. 특정 버전의 UUID만 필요한 경우 파이프 옵션에서 버전을 전달할 수 있습니다.

위에서 우리는 내장 파이프의 다양한 `Parse*` 계열을 바인딩하는 예를 보았습니다. 바인딩 유효성 검사 파이프는 약간 다릅니다. 다음 섹션에서 논의할 것입니다.

> info **힌트** 또한 검증 파이프의 광범위한 예는 [검증기법](/techniques/validation)을 참조하십시오.

#### Custom pipes

언급했듯이 커스텀 파이프를 만들 수 있습니다. Nest는 강력한 내장 `ParseIntPipe` 및 `ValidationPipe`를 제공하지만, 각각의 간단한 커스텀 버전을 처음부터 만들어 커스텀 파이프가 어떻게 구성되는지 살펴 보겠습니다.

간단한 `ValidationPipe`로 시작합니다. 처음에는 단순히 입력값을 취하고 즉시 동일한 값을 반환하여 식별함수처럼 동작하도록 합니다.

```typescript
@@filename(validation.pipe)
import { PipeTransform, Injectable, ArgumentMetadata } from '@nestjs/common';

@Injectable()
export class ValidationPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    return value;
  }
}
@@switch
import { Injectable } from '@nestjs/common';

@Injectable()
export class ValidationPipe {
  transform(value, metadata) {
    return value;
  }
}
```

> info **힌트** `PipeTransform<T, R>`은 파이프로 구현해야하는 일반 인터페이스입니다. 일반 인터페이스는 `T`를 사용하여 입력 `value`의 유형을 나타내고 `R`을 사용하여 `transform()`메서드의 반환유형을 나타냅니다.

모든 파이프는 `PipeTransform` 인터페이스 계약을 이행하기 위해 `transform()`메서드를 구현해야 합니다. 이 메소드에는 두개의 매개변수가 있습니다.

- `value`
- `metadata`

`value` 매개변수는 현재 처리된 메서드 인수(라우트 처리 메서드에 의해 수신되기 전)이고 `metadata`는 현재 처리된 메서드 인수의 메타데이터입니다. 메타데이터 객체에는 다음과 같은 속성이 있습니다.

```typescript
export interface ArgumentMetadata {
  type: 'body' | 'query' | 'param' | 'custom';
  metatype?: Type<unknown>;
  data?: string;
}
```

이러한 속성은 현재 처리된 인수를 설명합니다.

<table>
  <tr>
    <td>
      <code>type</code>
    </td>
    <td>인수가 본문
      <code>@Body()</code>, 쿼리
      <code>@Query()</code>, param
      <code>@Param()</code> 또는 커스텀 매개변수인지 여부를 나타냅니다(자세한 내용은
      <a routerLink="/custom-decorators">여기</a>) 참조).</td>
  </tr>
  <tr>
    <td>
      <code>metatype</code>
    </td>
    <td>
      인수의 메타타입을 제공합니다(예:
      <code>String</code>). 참고: 라우트 핸들러 메소드 서명에서 타입 선언을 생략하거나 바닐라 자바스크립트를 사용하면 값이 <code>정의되지 않습니다</code>.
    </td>
  </tr>
  <tr>
    <td>
      <code>data</code>
    </td>
    <td>데코레이터에 전달된 문자열(예: <code>@Body('string')</code>). 데코레이터 괄호를 비워두면 <code>정의되지 않습니다</code>.</td>
  </tr>
</table>

> warning **경고** TypeScript 인터페이스는 변환중에 사라집니다. 따라서 메소드 매개변수의 타입이 클래스가 아닌 인터페이스로 선언되면 `metatype`값은 `Object`가 됩니다.

#### Schema based validation

유효성 검사 파이프를 좀 더 유용하게 만들어 보겠습니다. `CatsController`의 `create()` 메소드를 자세히 살펴보면 서비스 메소드를 실행하기 전에 게시물 본문 객체가 유효한지 확인하고 싶을 것입니다.

```typescript
@@filename()
@Post()
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
@@switch
@Post()
async create(@Body() createCatDto) {
  this.catsService.create(createCatDto);
}
```

`createCatDto` 본문 매개변수에 초점을 맞춥니다. 타입은 `CreateCatDto`입니다.

```typescript
@@filename(create-cat.dto)
export class CreateCatDto {
  name: string;
  age: number;
  breed: string;
}
```

create 메서드로 들어오는 모든 요청에 유효한 본문이 포함되어 있는지 확인하고 싶습니다. 그래서 우리는 `createCatDto` 객체의 세 멤버를 검증해야 합니다. 라우트 핸들러 메소드내에서 이를 수행할 수 있지만 그렇게 하는 것은 **단일 책임 규칙**(SRP single responsibility rule)을 위반하므로 이상적이지 않습니다.

또 다른 접근방식은 **유효성 검사기 클래스**를 만들고 여기에 작업을 위임하는 것입니다. 이것은 우리가 각 메서드의 시작부분에서 이 유효성 검사기를 호출해야 한다는 것을 기억해야 한다는 단점이 있습니다.

유효성 검사 미들웨어를 만드는 것은 어떻습니까? 이것은 작동할 수 있지만 불행히도 전체 애플리케이션의 모든 컨텍스트에서 사용할 수 있는 **일반 미들웨어**를 만드는 것은 불가능합니다. 이는 미들웨어가 호출될 핸들러 및 매개변수를 포함하여 **실행 컨텍스트**를 인식하지 못하기 때문입니다.

물론 이것은 파이프가 설계된 사용사례입니다. 이제 계속해서 검증 파이프를 개선해 보겠습니다.

<app-banner-courses></app-banner-courses>

#### Object schema validation

깨끗하고 [건조한](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) 방식으로 객체 유효성 검사를 수행하는데 사용할 수 있는 몇가지 방법이 있습니다. 한가지 일반적인 접근 방식은 **스키마 기반** 유효성 검사를 사용하는 것입니다. 계속해서 그 접근방식을 시도해 봅시다.

[Joi](https://github.com/sideway/joi) 라이브러리를 사용하면 읽기 쉬운 API를 사용하여 간단한 방식으로 스키마를 만들 수 있습니다. Joi 기반 스키마를 사용하는 유효성 검사 파이프를 구축해 보겠습니다.

필요한 패키지를 설치하여 시작하십시오.

```bash
$ npm install --save joi
$ npm install --save-dev @types/joi
```

아래 코드 샘플에서는 스키마를 `constructor` 인수로 사용하는 간단한 클래스를 만듭니다. 그런 다음 제공된 스키마에 대해 들어오는 인수의 유효성을 검사하는 `schema.validate()` 메서드를 적용합니다.

앞서 언급했듯이 **유효성 검사 파이프**는 값을 변경하지 않고 반환하거나 예외를 던집니다(throw).

다음 섹션에서는 `@UsePipes()` 데코레이터를 사용하여 주어진 컨트롤러 메소드에 적절한 스키마를 제공하는 방법을 볼 수 있습니다. 이렇게 하면 검증 파이프를 컨텍스트 전체에서 다시 사용할 수 있습니다.

```typescript
@@filename()
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';
import { ObjectSchema } from 'joi';

@Injectable()
export class JoiValidationPipe implements PipeTransform {
  constructor(private schema: ObjectSchema) {}

  transform(value: any, metadata: ArgumentMetadata) {
    const { error } = this.schema.validate(value);
    if (error) {
      throw new BadRequestException('Validation failed');
    }
    return value;
  }
}
@@switch
import { Injectable, BadRequestException } from '@nestjs/common';

@Injectable()
export class JoiValidationPipe {
  constructor(schema) {
    this.schema = schema;
  }

  transform(value, metadata) {
    const { error } = this.schema.validate(value);
    if (error) {
      throw new BadRequestException('Validation failed');
    }
    return value;
  }
}
```

#### Binding validation pipes

앞서 `ParseIntPipe` 및 나머지 `Parse*` 파이프와 같은 변환 파이프를 바인딩하는 방법을 살펴 보았습니다.

바인딩 유효성 검사 파이프도 매우 간단합니다.

이 경우 메서드 호출 수준에서 파이프를 바인딩하려고 합니다. 현재 예제에서 `JoiValidationPipe`를 사용하려면 다음을 수행해야 합니다.

1. `JoiValidationPipe`의 인스턴스를 만듭니다.
2. 파이프의 클래스 생성자에 컨텍스트별 Joi 스키마를 전달합니다.
3. 파이프를 메서드에 바인딩

아래와 같이 `@UsePipes()` 데코레이터를 사용합니다.

```typescript
@@filename()
@Post()
@UsePipes(new JoiValidationPipe(createCatSchema))
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
@@switch
@Post()
@Bind(Body())
@UsePipes(new JoiValidationPipe(createCatSchema))
async create(createCatDto) {
  this.catsService.create(createCatDto);
}
```

> info **힌트** `@UsePipes()` 데코레이터는 `@nestjs/common` 패키지에서 가져옵니다.

#### Class validator

> warning **경고** 이 섹션의 기술에는 TypeScript가 필요하며 앱이 바닐라 자바스크립트를 사용하여 작성된 경우 사용할 수 없습니다.

유효성 검사 기술의 대체 구현을 살펴 보겠습니다.

Nest는 [class-validator](https://github.com/typestack/class-validator) 라이브러리와 잘 작동합니다. 이 강력한 라이브러리를 사용하면 데코레이터 기반 유효성 검사를 사용할 수 있습니다. 데코레이터 기반 유효성 검사는 특히 처리된 속성의 `metatype`에 액세스할 수 있으므로 Nest의 **파이프** 기능과 결합할 때 매우 강력합니다. 시작하기 전에 필요한 패키지를 설치해야 합니다.

```bash
$ npm i --save class-validator class-transformer
```

이것들이 설치되면 `CreateCatDto` 클래스에 데코레이터 몇개를 추가할 수 있습니다. 여기에서 이 기법의 중요한 이점을 볼 수 있습니다.`CreateCatDto` 클래스는 Post 본문 객체에 대한 단일 소스로 남아 있습니다 (별도의 유효성 검사 클래스를 만들 필요가 없음).

```typescript
@@filename(create-cat.dto)
import { IsString, IsInt } from 'class-validator';

export class CreateCatDto {
  @IsString()
  name: string;

  @IsInt()
  age: number;

  @IsString()
  breed: string;
}
```

> info **힌트** 클래스 유효성 검사기 데코레이터에 대한 자세한 내용은 [여기](https://github.com/typestack/class-validator#usage)를 참조하세요.

이제 이러한 주석을 사용하는 `ValidationPipe` 클래스를 만들 수 있습니다.

```typescript
@@filename(validation.pipe)
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';
import { validate } from 'class-validator';
import { plainToClass } from 'class-transformer';

@Injectable()
export class ValidationPipe implements PipeTransform<any> {
  async transform(value: any, { metatype }: ArgumentMetadata) {
    if (!metatype || !this.toValidate(metatype)) {
      return value;
    }
    const object = plainToClass(metatype, value);
    const errors = await validate(object);
    if (errors.length > 0) {
      throw new BadRequestException('Validation failed');
    }
    return value;
  }

  private toValidate(metatype: Function): boolean {
    const types: Function[] = [String, Boolean, Number, Array, Object];
    return !types.includes(metatype);
  }
}
```

> warning **알림** 위에서 우리는 [class-transformer](https://github.com/typestack/class-transformer) 라이브러리를 사용했습니다. **class-validator** 라이브러리와 동일한 작성자가 만들었으며 결과적으로 서로 잘 어울립니다.

이 코드를 살펴보겠습니다. 먼저 `transform()` 메서드가 `async`로 표시되어 있습니다. Nest가 동기 및 **비동기** 파이프를 모두 지원하기 때문에 가능합니다. 일부 `class-validator` 유효성 검사가 [비동기화될 수 있음](https://github.com/typestack/class-validator#custom-validation-classes)(Promise 활용)때문에 이 메서드를 `async`로 만듭니다.

다음으로, 우리는 메타타입 필드 (`ArgumentMetadata`에서 이 멤버만 추출)를 `metatype` 매개변수로 추출하기 위해 디스트럭처링을 사용하고 있습니다. 이것은 전체 `ArgumentMetadata`를 가져온 다음 메타타입 변수를 할당하는 추가 명령문을 갖는 것에 대한 속기일 뿐입니다.

다음으로 헬퍼 함수 `toValidate()`를 확인합니다. 처리중인 현재 인수가 네이티브 자바스크립트 타입인 경우 유효성 검사 단계를 건너 뛰는 역할을 합니다(이러한 인수는 유효성 검사 데코레이터를 연결할 수 없으므로 유효성 검사 단계를 통해 실행할 이유가 없습니다).

다음으로 클래스 변환기 함수 `plainToClass()`를 사용하여 일반 자바스크립트 인수 객체를 타입이 지정된 객체로 변환하여 유효성 검사를 적용할 수 있습니다. 이 작업을 수행해야 하는 이유는 네트워크 요청에서 역직렬화될 때 들어오는 포스트(post) 본문 객체가 **아무 타입 정보도 가지고 있지 않기 때문입니다**(이것이 Express와 같은 기본 플랫폼이 작동하는 방식입니다). 클래스 유효성 검사기는 이전에 DTO에 대해 정의한 유효성 검사 데코레이터를 사용해야 하므로 들어오는 본문을 단순한 바닐라 객체가 아닌 적절하게 장식된 객체로 처리하기 위해 이 변환을 수행해야 합니다.

마지막으로 앞서 언급했듯이 이것은 **유효성 검사 파이프**이므로 변경되지 않은 값을 반환하거나 예외를 던집니다(throw).

마지막 단계는 `ValidationPipe`를 바인딩하는 것입니다. 파이프는 매개변수 범위, 메서드 범위, 컨트롤러 범위 또는 전역 범위일 수 있습니다. 앞서 Joi 기반 유효성 검사 파이프를 사용하여 메서드 수준에서 파이프를 바인딩하는 예를 보았습니다.
아래 예제에서는 파이프 인스턴스를 라우트 핸들러 `@Body()` 데코레이터에 바인딩하여 파이프가 포스트(post) 본문의 유효성을 검사하도록 호출합니다.

```typescript
@@filename(cats.controller)
@Post()
async create(
  @Body(new ValidationPipe()) createCatDto: CreateCatDto,
) {
  this.catsService.create(createCatDto);
}
```

매개변수 범위 파이프는 유효성 검증 로직이 지정된 매개변수 하나만 관련될 때 유용합니다.

#### Global scoped pipes

`ValidationPipe`는 가능한 한 일반적으로 생성되었으므로 전체 애플리케이션의 모든 라우트 핸들러에 적용되도록 **전역 범위**(global-scoped) 파이프로 설정하여 완전한 유틸리티임을 실현할 수 있습니다.

```typescript
@@filename(main)
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe());
  await app.listen(3000);
}
bootstrap();
```

> warning **알림** [하이브리드 앱](/faq/hybrid-application)의 경우 `useGlobalPipes()` 메서드는 게이트웨이 및 마이크로서비스에 대한 파이프를 설정하지 않습니다. "표준"(비 하이브리드) 마이크로서비스 앱의 경우 `useGlobalPipes()`는 파이프를 전역으로 마운트합니다.

전역 파이프는 모든 컨트롤러 및 모든 라우트 핸들러에 대해 애플리케이션에서 사용됩니다.

의존성 주입과 관련하여 모듈 외부에서 등록된 전역 파이프(위의 예에서와 같이 `useGlobalPipes()` 사용)는 바인딩이 모듈 컨텍스트 외부에서 수행되었으므로 종속성을 주입할 수 없습니다. 이 문제를 해결하기 위해 다음 구성을 사용하여 **모든 모듈에서 직접** 전역 파이프를 설정할 수 있습니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { APP_PIPE } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_PIPE,
      useClass: ValidationPipe,
    },
  ],
})
export class AppModule {}
```

> info **힌트** 이 접근방식을 사용하여 파이프에 대한 종속성 주입을 수행할 때 이 구성이 사용되는 모듈에 관계없이 파이프는 실제로 전역이라는 점에 유의하십시오. 어디에서해야 합니까? 파이프(위의 예에서는 `ValidationPipe`)가 정의된 모듈을 선택합니다. 또한 `useClass`가 커스텀 프로바이더 등록을 처리하는 유일한 방법은 아닙니다. [여기](/fundamentals/custom-providers)에서 자세히 알아보세요.

#### The built-in ValidationPipe

참고로 `ValidationPipe`는 Nest에서 즉시 제공되므로 일반 유효성 검사 파이프를 직접 빌드할 필요가 없습니다. 빌트인 `ValidationPipe`는 이 장에서 빌드한 샘플보다 더 많은 옵션을 제공합니다. 이 샘플은 커스텀 빌드 파이프의 메커니즘을 설명하기 위해 기본으로 유지되었습니다. 많은 예제와 함께 자세한 내용은 [여기](/techniques/validation)에서 찾을 수 있습니다.

#### Transformation use case

커스텀 파이프의 유일한 사용 사례는 유효성 검사가 아닙니다. 이 장의 시작부분에서 파이프가 입력 데이터를 원하는 형식으로 **변환**할 수도 있다고 언급했습니다. 이는 `transform` 함수에서 반환된 값이 인수의 이전값을 완전히 덮어쓰기 때문에 가능합니다.

이것이 언제 유용합니까? 때때로 클라이언트에서 전달된 데이터는 라우트 핸들러 메소드에 의해 적절하게 처리되기 전에 문자열을 정수로 변환하는 것과 같이 약간의 변경이 필요하다는 점을 고려하십시오. 또한 일부 필수 데이터 필드가 누락되었을 수 있으며 기본값을 적용하려고합니다. **변환 파이프**는 클라이언트 요청과 요청 핸들러 사이에 처리 기능을 삽입하여 이러한 기능을 수행할 수 있습니다.

다음은 문자열을 정수값으로 파싱하는 간단한 `ParseIntPipe`입니다.(위에서 언급했듯이 Nest에는 더 정교한 내장 `ParseIntPipe`가 있습니다. 이를 커스텀 변환 파이프의 간단한 예로 포함합니다).

```typescript
@@filename(parse-int.pipe)
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';

@Injectable()
export class ParseIntPipe implements PipeTransform<string, number> {
  transform(value: string, metadata: ArgumentMetadata): number {
    const val = parseInt(value, 10);
    if (isNaN(val)) {
      throw new BadRequestException('Validation failed');
    }
    return val;
  }
}
@@switch
import { Injectable, BadRequestException } from '@nestjs/common';

@Injectable()
export class ParseIntPipe {
  transform(value, metadata) {
    const val = parseInt(value, 10);
    if (isNaN(val)) {
      throw new BadRequestException('Validation failed');
    }
    return val;
  }
}
```

그런 다음 이 파이프를 아래와 같이 선택한 매개변수에 바인딩할 수 있습니다.

```typescript
@@filename()
@Get(':id')
async findOne(@Param('id', new ParseIntPipe()) id) {
  return this.catsService.findOne(id);
}
@@switch
@Get(':id')
@Bind(Param('id', new ParseIntPipe()))
async findOne(id) {
  return this.catsService.findOne(id);
}
```

또 다른 유용한 변환사례는 요청에 제공된 ID를 사용하여 데이터베이스에서 **기존 사용자** 항목을 선택하는 것입니다.

```typescript
@@filename()
@Get(':id')
findOne(@Param('id', UserByIdPipe) userEntity: UserEntity) {
  return userEntity;
}
@@switch
@Get(':id')
@Bind(Param('id', UserByIdPipe))
findOne(userEntity) {
  return userEntity;
}
```

이 파이프의 구현은 독자에게 맡기지만 다른 모든 변환 파이프와 마찬가지로 입력값(`id`)을 받고 출력값(`UserEntity` 객체)을 반환합니다. 이렇게하면 핸들러에서 공통 파이프로 상용구 코드를 추상화하여 코드를 더 선언적이고 [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)하게 만들 수 있습니다.

#### Providing defaults

`Parse*` 파이프는 매개변수 값이 정의될 것으로 예상합니다. `null` 또는 `undefined` 값을 받으면 예외가 발생합니다. 엔드포인트가 누락된 쿼리 문자열 매개변수 값을 처리할 수 있도록하려면 `Parse*` 파이프가 이러한 값에 대해 작동하기 전에 삽입할 기본값을 제공해야합니다. `DefaultValuePipe`는 그 목적에 부합합니다. 아래와 같이 관련 `Parse*` 파이프 앞에 `@Query()` 데코레이터에서 `DefaultValuePipe`를 인스턴스화하면됩니다.

```typescript
@@filename()
@Get()
async findAll(
  @Query('activeOnly', new DefaultValuePipe(false), ParseBoolPipe) activeOnly: boolean,
  @Query('page', new DefaultValuePipe(0), ParseIntPipe) page: number,
) {
  return this.catsService.findAll({ activeOnly, page });
}
```
