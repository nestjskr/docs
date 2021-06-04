### Operations

OpenAPI 용어에서 경로는 API가 노출하는 `/users` 또는 `/reports/summary`와 같은 엔드포인트(리소스)이고 작업은 `GET`, `POST` 또는 `DELETE`와 같이 이러한 경로를 조작하는데 사용되는 HTTP 메서드입니다.

#### Tags

컨트롤러를 특정 태그에 연결하려면 `@ApiTags(...tags)` 데코레이터를 사용하세요.

```typescript
@ApiTags('cats')
@Controller('cats')
export class CatsController {}
```

#### Headers

요청의 일부로 예상되는 커스텀 헤더를 정의하려면 `@ApiHeader()`를 사용하세요.

```typescript
@ApiHeader({
  name: 'X-MyHeader',
  description: 'Custom header',
})
@Controller('cats')
export class CatsController {}
```

#### Responses

커스텀 HTTP 응답을 정의하려면 `@ApiResponse()` 데코레이터를 사용하세요.

```typescript
@Post()
@ApiResponse({ status: 201, description: 'The record has been successfully created.'})
@ApiResponse({ status: 403, description: 'Forbidden.'})
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

Nest는 `@ApiResponse` 데코레이터에서 상속된 단축 **API 응답** 데코레이터 세트를 제공합니다.

- `@ApiOkResponse()`
- `@ApiCreatedResponse()`
- `@ApiAcceptedResponse()`
- `@ApiNoContentResponse()`
- `@ApiMovedPermanentlyResponse()`
- `@ApiBadRequestResponse()`
- `@ApiUnauthorizedResponse()`
- `@ApiNotFoundResponse()`
- `@ApiForbiddenResponse()`
- `@ApiMethodNotAllowedResponse()`
- `@ApiNotAcceptableResponse()`
- `@ApiRequestTimeoutResponse()`
- `@ApiConflictResponse()`
- `@ApiTooManyRequestsResponse()`
- `@ApiGoneResponse()`
- `@ApiPayloadTooLargeResponse()`
- `@ApiUnsupportedMediaTypeResponse()`
- `@ApiUnprocessableEntityResponse()`
- `@ApiInternalServerErrorResponse()`
- `@ApiNotImplementedResponse()`
- `@ApiBadGatewayResponse()`
- `@ApiServiceUnavailableResponse()`
- `@ApiGatewayTimeoutResponse()`
- `@ApiDefaultResponse()`

```typescript
@Post()
@ApiCreatedResponse({ description: 'The record has been successfully created.'})
@ApiForbiddenResponse({ description: 'Forbidden.'})
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

요청에 대한 반환 모델을 지정하려면 클래스를 만들고 `@ApiProperty()` 데코레이터로 모든 속성에 주석을 추가해야합니다.

```typescript
export class Cat {
  @ApiProperty()
  id: number;

  @ApiProperty()
  name: string;

  @ApiProperty()
  age: number;

  @ApiProperty()
  breed: string;
}
```

그런 다음 `Cat` 모델을 응답 데코레이터의 `type` 속성과 함께 사용할 수 있습니다.

```typescript
@ApiTags('cats')
@Controller('cats')
export class CatsController {
  @Post()
  @ApiCreatedResponse({
    description: 'The record has been successfully created.',
    type: Cat,
  })
  async create(@Body() createCatDto: CreateCatDto): Promise<Cat> {
    return this.catsService.create(createCatDto);
  }
}
```

브라우저를 열고 생성된 `Cat` 모델을 확인해 보겠습니다.

<figure><img src="/assets/swagger-response-type.png" /></figure>

#### File upload

`@ApiBody` 데코레이터와 `@ApiConsumes()`를 함께 사용하여 특정 메서드에 대한 파일 업로드를 활성화할 수 있습니다. 다음은 [파일 업로드](/techniques/file-upload) 기술을 사용한 전체 예제입니다.

```typescript
@UseInterceptors(FileInterceptor('file'))
@ApiConsumes('multipart/form-data')
@ApiBody({
  description: 'List of cats',
  type: FileUploadDto,
})
uploadFile(@UploadedFile() file) {}
```

여기서 `FileUploadDto`는 다음과 같이 정의됩니다.

```typescript
class FileUploadDto {
  @ApiProperty({ type: 'string', format: 'binary' })
  file: any;
}
```

여러 파일 업로드를 처리하기 위해 다음과 같이 `FilesUploadDto`를 정의할 수 있습니다.

```typescript
class FilesUploadDto {
  @ApiProperty({ type: 'array', items: { type: 'string', format: 'binary' } })
  files: any[];
}
```

#### Extensions

요청에 확장을 추가하려면 `@ApiExtension()` 데코레이터를 사용하십시오. 확장자 이름은 `x-`로 시작해야합니다.

```typescript
@ApiExtension('x-foo', { hello: 'world' })
```

#### Advanced: Generic `ApiResponse`

[Raw Definitions](/openapi/types-and-parameters#raw-definitions)를 제공하는 기능을 통해 Swagger UI에 대한 일반 스키마를 정의할 수 있습니다. 다음 DTO가 있다고 가정합니다.

```ts
export class PaginatedDto<TData> {
  @ApiProperty()
  total: number;

  @ApiProperty()
  limit: number;

  @ApiProperty()
  offset: number;

  results: TData[];
}
```

나중에 원시 정의를 제공할 것이므로 `results` 장식을 건너뜁니다. 이제 다른 DTO를 정의하고 다음과 같이 이름을 `CatDto`로 지정하겠습니다.

```ts
export class CatDto {
  @ApiProperty()
  name: string;

  @ApiProperty()
  age: number;

  @ApiProperty()
  breed: string;
}
```

이를 통해 다음과 같이 `PaginatedDto<CatDto>` 응답을 정의할 수 있습니다.

```ts
@ApiOkResponse({
  schema: {
    allOf: [
      { $ref: getSchemaPath(PaginatedDto) },
      {
        properties: {
          results: {
            type: 'array',
            items: { $ref: getSchemaPath(CatDto) },
          },
        },
      },
    ],
  },
})
async findAll(): Promise<PaginatedDto<CatDto>> {}
```

이 예제에서는 응답에 allOf `PaginatedDto`가 있고 `results` 속성이 `Array<CatDto>` 타입이되도록 지정합니다.

- 주어진 모델에 대한 OpenAPI 사양 파일 내에서 OpenAPI 스키마 경로를 반환하는 `getSchemaPath()` 함수.
- `allOf`는 OAS 3에서 다양한 상속 관련 사용 사례를 다루기 위해 제공하는 개념입니다.

마지막으로 `PaginatedDto`는 컨트롤러에 의해 직접 참조되지 않기 때문에 `SwaggerModule`은 아직 해당 모델 정의를 생성할 수 없습니다. 이 경우 [Extra Model](/openapi/types-and-parameters#extra-models)로 추가해야합니다. 예를 들어 다음과 같이 컨트롤러 수준에서 `@ApiExtraModels()` 데코레이터를 사용할 수 있습니다.

```ts
@Controller('cats')
@ApiExtraModels(PaginatedDto)
export class CatsController {}
```

지금 Swagger를 실행하면 이 특정 엔드포인트에 대해 생성된 `swagger.json`에 다음 응답이 정의되어 있어야합니다.

```json
"responses": {
  "200": {
    "description": "",
    "content": {
      "application/json": {
        "schema": {
          "allOf": [
            {
              "$ref": "#/components/schemas/PaginatedDto"
            },
            {
              "properties": {
                "results": {
                  "$ref": "#/components/schemas/CatDto"
                }
              }
            }
          ]
        }
      }
    }
  }
}
```

재사용 가능하게 만들기 위해 다음과 같이 `PaginatedDto`에 대한 사용자 정의 데코레이터를 만들 수 있습니다.

```ts
export const ApiPaginatedResponse = <TModel extends Type<any>>(
  model: TModel,
) => {
  return applyDecorators(
    ApiOkResponse({
      schema: {
        allOf: [
          { $ref: getSchemaPath(PaginatedDto) },
          {
            properties: {
              results: {
                type: 'array',
                items: { $ref: getSchemaPath(model) },
              },
            },
          },
        ],
      },
    }),
  );
};
```

> info **힌트** `Type<any>` 인터페이스 및 `applyDecorators` 함수는 `@nestjs/common` 패키지에서 가져옵니다.

이를 통해 엔드포인트에서 커스텀 `@ApiPaginatedResponse()` 데코레이터를 사용할 수 있습니다.

```ts
@ApiPaginatedResponse(CatDto)
async findAll(): Promise<PaginatedDto<CatDto>> {}
```

클라이언트 생성 도구의 경우 이 접근 방식은 클라이언트에 대해 `PaginatedResponse<TModel>`이 생성되는 방식이 모호합니다. 다음 스니펫은 위의 `GET /` 엔드포인트에 대한 클라이언트 생성기 결과의 예입니다.

```typescript
// Angular
findAll(): Observable<{ total: number, limit: number, offset: number, results: CatDto[] }>
```

보시다시피 여기서 **리턴 타입 Return Type**은 모호합니다. 이 문제를 해결하려면 `ApiPaginatedResponse`의 `schema`에 `title` 속성을 추가할 수 있습니다.

```typescript
export const ApiPaginatedResponse = <TModel extends Type<any>>(model: TModel) => {
  return applyDecorators(
    ApiOkResponse({
      schema: {
        title: `PaginatedResponseOf${model.name}`
        allOf: [
          // ...
        ],
      },
    }),
  );
};
```

이제 클라이언트 생성기 도구의 결과는 다음과 같습니다.

```ts
// Angular
findAll(): Observable<PaginatedResponseOfCatDto>
```
