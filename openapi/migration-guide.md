### Migration guide

현재 `@nestjs/swagger@3.*`를 사용중인 경우 버전 4.0의 다음 중단/API 변경 사항에 유의하세요.

#### Breaking changes

다음 데코레이터가 변경/이름이 변경되었습니다.

- `@ApiModelProperty` 는 이제 `@ApiProperty`입니다
- `@ApiModelPropertyOptional` 는 이제 `@ApiPropertyOptional`입니다
- `@ApiResponseModelProperty` 는 이제 `@ApiResponseProperty`입니다
- `@ApiImplicitQuery` 는 이제 `@ApiQuery`입니다
- `@ApiImplicitParam` 는 이제 `@ApiParam`입니다
- `@ApiImplicitBody` 는 이제 `@ApiBody`입니다
- `@ApiImplicitHeader` 는 이제 `@ApiHeader`입니다
- `@ApiOperation({{ '{' }} title: 'test' {{ '}' }})` 는 이제 `@ApiOperation({{ '{' }} summary: 'test' {{ '}' }})`입니다
- `@ApiUseTags` 는 이제 `@ApiTags`입니다

`DocumentBuilder` 주요 변경 사항(업데이트된 메서드 서명) :

- `addTag`
- `addBearerAuth`
- `addOAuth2`
- `setContactEmail` 는 이제 `setContact`입니다
- `setHost`가 제거되었습니다.
- `setSchemes`가 제거되었습니다 (대신 `addServer` 사용, 예: `addServer('http://')`)

#### New methods

다음 방법이 추가되었습니다.

- `addServer`
- `addApiKey`
- `addBasicAuth`
- `addSecurity`
- `addSecurityRequirements`

<app-banner-shop></app-banner-shop>
