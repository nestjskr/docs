### Types and parameters

`SwaggerModule`은 라우트 핸들러에서 모든 `@Body()`, `@Query()`및 `@Param()` 데코레이터를 검색하여 API 문서를 생성합니다. 또한 반사를 활용하여 해당 모델 정의를 생성합니다. 다음 코드를 고려하십시오.

```typescript
@Post()
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

> info **힌트** 본문 정의를 명시적으로 설정하려면 `@ApiBody()` 데코레이터 (`@nestjs/swagger` 패키지에서 가져옴)를 사용하세요.

`CreateCatDto`를 기반으로 다음과 같은 모델 정의 Swagger UI가 생성됩니다.

<figure><img src="/assets/swagger-dto.png" /></figure>

보시다시피, 클래스에는 몇가지 선언된 속성이 있지만 정의는 비어 있습니다. 클래스 속성을 `SwaggerModule`에 표시하려면 `@ApiProperty()` 데코레이터로 주석을 달거나 자동으로 이를 수행할 CLI 플러그인 (**Plugin** 섹션에서 자세히 알아보기)을 사용해야합니다.

```typescript
import { ApiProperty } from '@nestjs/swagger';

export class CreateCatDto {
  @ApiProperty()
  name: string;

  @ApiProperty()
  age: number;

  @ApiProperty()
  breed: string;
}
```

> info **힌트** 각 속성에 수동으로 주석을 추가하는 대신 이를 자동으로 제공하는 Swagger 플러그인([Plugin](/openapi/cli-plugin) 섹션 참조)을 사용하는 것이 좋습니다.

브라우저를 열고 생성된 `CreateCatDto` 모델을 확인하겠습니다.

<figure><img src="/assets/swagger-dto2.png" /></figure>

또한 `@ApiProperty()` 데코레이터를 사용하면 다양한 [Schema Object](https://swagger.io/specification/#schemaObject) 속성을 설정할 수 있습니다.

```typescript
@ApiProperty({
  description: 'The age of a cat',
  minimum: 1,
  default: 1,
})
age: number;
```

> info **힌트** `{{"@ApiProperty({ required: false })"}}`를 명시적으로 입력하는 대신 `@ApiPropertyOptional()` 단축 데코레이터를 사용할 수 있습니다.

속성 타입을 명시적으로 설정하려면 `type` 키를 사용하세요.

```typescript
@ApiProperty({
  type: Number,
})
age: number;
```

#### Arrays

속성이 배열인 경우 아래와 같이 배열 타입을 수동으로 표시해야 합니다.

```typescript
@ApiProperty({ type: [String] })
names: string[];
```

> info **힌트** 배열을 자동으로 감지하는 Swagger 플러그인 ([Plugin](/openapi/cli-plugin) 섹션 참조)을 사용하는 것이 좋습니다.

타입을 배열의 첫번째 요소로 포함하거나(위에 표시된대로) `isArray` 속성을 `true`로 설정합니다.

<app-banner-enterprise></app-banner-enterprise>

#### Circular dependencies

클래스간에 순환 종속성이 있는 경우 lazy 함수를 사용하여 `SwaggerModule`에 타입 정보를 제공합니다.

```typescript
@ApiProperty({ type: () => Node })
node: Node;
```

> info **힌트** 순환 종속성을 자동으로 감지하는 Swagger 플러그인([Plugin](/openapi/cli-plugin) 섹션 참조)을 사용하는 것이 좋습니다.

#### Generics and interfaces

TypeScript는 제네릭 또는 인터페이스에 대한 메타데이터를 저장하지 않으므로 DTO에서 사용할 때 `SwaggerModule`이 런타임에 모델 정의를 제대로 생성하지 못할 수 있습니다. 예를 들어 다음 코드는 Swagger 모듈에서 올바르게 검사되지 않습니다.

```typescript
createBulk(@Body() usersDto: CreateUserDto[])
```

이 제한을 극복하기 위해 타입을 명시적으로 설정할 수 있습니다.

```typescript
@ApiBody({ type: [CreateUserDto] })
createBulk(@Body() usersDto: CreateUserDto[])
```

#### Enums

`enum`을 식별하려면 값 배열을 사용하여 `@ApiProperty`의 `enum` 속성을 수동으로 설정해야합니다.

```typescript
@ApiProperty({ enum: ['Admin', 'Moderator', 'User']})
role: UserRole;
```

또는 다음과 같이 실제 TypeScript 열거형을 정의하십시오.

```typescript
export enum UserRole {
  Admin = 'Admin',
  Moderator = 'Moderator',
  User = 'User',
}
```

그런 다음 `@ApiQuery()` 데코레이터와 함께 `@Query()` 매개변수 데코레이터와 함께 열거형을 직접 사용할 수 있습니다.

```typescript
@ApiQuery({ name: 'role', enum: UserRole })
async filterByRole(@Query('role') role: UserRole = UserRole.User) {}
```

<figure><img src="/assets/enum_query.gif" /></figure>

`isArray`를 **true**로 설정하면 `enum`을 **다중 선택**으로 선택할 수 있습니다.

<figure><img src="/assets/enum_query_array.gif" /></figure>

#### Enums schema

기본적으로 `enum` 속성은 `parameter`에 [Enum](https://swagger.io/docs/specification/data-models/enums/)의 원시 정의를 추가합니다.

```yaml
- breed:
    type: 'string'
    enum:
      - Persian
      - Tabby
      - Siamese
```

위의 사양은 대부분의 경우 잘 작동합니다. 그러나 사양을 **입력**으로 사용하고 **클라이언트 측** 코드를 생성하는 도구를 사용하는 경우 중복 된 `enum`을 포함하는 생성된 코드에 문제가 발생할 수 있습니다. 다음 코드 스니펫을 고려하십시오.

```typescript
// generated client-side code
export class CatDetail {
  breed: CatDetailEnum;
}

export class CatInformation {
  breed: CatInformationEnum;
}

export enum CatDetailEnum {
  Persian = 'Persian',
  Tabby = 'Tabby',
  Siamese = 'Siamese',
}

export enum CatInformationEnum {
  Persian = 'Persian',
  Tabby = 'Tabby',
  Siamese = 'Siamese',
}
```

> info **힌트** 위의 스니펫은 [NSwag](https://github.com/RicoSuter/NSwag)라는 도구를 사용하여 생성되었습니다.

이제 정확히 동일한 두개의 `enum`이 있음을 알 수 있습니다.
이 문제를 해결하기 위해 데코레이터에서 `enum` 속성과 함께 `enumName`을 전달할 수 있습니다.

```typescript
export class CatDetail {
  @ApiProperty({ enum: CatBreed, enumName: 'CatBreed' })
  breed: CatBreed;
}
```

`enumName` 속성을 사용하면 `@nestjs/swagger`가 `CatBreed`를 자체 `schema`로 전환하여 `CatBreed` 열거형을 재사용할 수 있습니다. 사양은 다음과 같습니다.

```yaml
CatDetail:
  type: 'object'
  properties:
    ...
    - breed:
        schema:
          $ref: '#/components/schemas/CatBreed'
CatBreed:
  type: string
  enum:
    - Persian
    - Tabby
    - Siamese
```

> info **힌트** `enum`을 속성으로 사용하는 모든 **데코레이터**도 `enumName`을 사용합니다.

#### Raw definitions

일부 특정 시나리오(예: 깊게 중첩된 배열, 행렬)에서는 타입을 직접 설명할 수 있습니다.

```typescript
@ApiProperty({
  type: 'array',
  items: {
    type: 'array',
    items: {
      type: 'number',
    },
  },
})
coords: number[][];
```

마찬가지로 컨트롤러 클래스에서 입력/출력 콘텐츠를 수동으로 정의하려면 `schema` 속성을 사용하십시오.

```typescript
@ApiBody({
  schema: {
    type: 'array',
    items: {
      type: 'array',
      items: {
        type: 'number',
      },
    },
  },
})
async create(@Body() coords: number[][]) {}
```

#### Extra models

컨트롤러에서 직접 참조되지 않지만 Swagger 모듈에서 검사해야하는 추가 모델을 정의하려면 `@ApiExtraModels()` 데코레이터를 사용하십시오.

```typescript
@ApiExtraModels(ExtraModel)
export class CreateCatDto {}
```

> info **힌트** 특정 모델 클래스에 대해 `@ApiExtraModels()`를 한번만 사용하면 됩니다.

또는 다음과 같이 `extraModels` 속성이 `SwaggerModule#createDocument()` 메서드에 지정된 옵션 객체를 전달할 수 있습니다.

```typescript
const document = SwaggerModule.createDocument(app, options, {
  extraModels: [ExtraModel],
});
```

모델에 대한 참조(`$ref`)를 얻으려면 `getSchemaPath(ExtraModel)` 함수를 사용하세요.

```typescript
'application/vnd.api+json': {
   schema: { $ref: getSchemaPath(ExtraModel) },
},
```

#### oneOf, anyOf, allOf

스키마를 결합하려면 `oneOf`, `anyOf` 또는 `allOf` 키워드를 사용할 수 있습니다([자세히 알아보기](https://swagger.io/docs/specification/data-models/oneof-anyof-allof-not/)).

```typescript
@ApiProperty({
  oneOf: [
    { $ref: getSchemaPath(Cat) },
    { $ref: getSchemaPath(Dog) },
  ],
})
pet: Cat | Dog;
```

다형성 배열(즉, 멤버가 여러 스키마에 걸쳐있는 배열)을 정의하려면 원시 정의(위 참조)를 사용하여 타입을 직접 정의해야합니다.

```typescript
type Pet = Cat | Dog;

@ApiProperty({
  type: 'array',
  items: {
    oneOf: [
      { $ref: getSchemaPath(Cat) },
      { $ref: getSchemaPath(Dog) },
    ],
  },
})
pets: Pet[];
```

> info **힌트** `@nestjs/swagger`에서 `getSchemaPath()` 함수를 가져옵니다.

`Cat`과 `Dog`는 모두 `@ApiExtraModels()` 데코레이터를 사용하여 추가 모델로 정의해야합니다(클래스 수준에서).
