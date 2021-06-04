### Security

특정 작업에 사용할 보안 메커니즘을 정의하려면 `@ApiSecurity()` 데코레이터를 사용하세요.

```typescript
@ApiSecurity('basic')
@Controller('cats')
export class CatsController {}
```

애플리케이션을 실행하기 전에 `DocumentBuilder`를 사용하여 기본 문서에 보안 정의를 추가해야합니다.

```typescript
const options = new DocumentBuilder().addSecurity('basic', {
  type: 'http',
  scheme: 'basic',
});
```

가장 널리 사용되는 인증 기술중 일부가 내장되어 있으므로(예: `basic` 및 `bearer`) 위에 표시된대로 보안 메커니즘을 수동으로 정의할 필요가 없습니다.

#### Basic authentication

기본 인증을 사용하려면 `@ApiBasicAuth()`를 사용하십시오.

```typescript
@ApiBasicAuth()
@Controller('cats')
export class CatsController {}
```

애플리케이션을 실행하기 전에 `DocumentBuilder`를 사용하여 기본 문서에 보안 정의를 추가해야 합니다.

```typescript
const options = new DocumentBuilder().addBasicAuth();
```

#### Bearer authentication

Bearer 인증을 사용하려면 `@ApiBearerAuth()`를 사용하십시오.

```typescript
@ApiBearerAuth()
@Controller('cats')
export class CatsController {}
```

애플리케이션을 실행하기 전에 `DocumentBuilder`를 사용하여 기본 문서에 보안 정의를 추가해야 합니다.

```typescript
const options = new DocumentBuilder().addBearerAuth();
```

#### OAuth2 authentication

OAuth2를 활성화하려면 `@ApiOAuth2()`를 사용하십시오.

```typescript
@ApiOAuth2(['pets:write'])
@Controller('cats')
export class CatsController {}
```

애플리케이션을 실행하기 전에 `DocumentBuilder`를 사용하여 기본 문서에 보안 정의를 추가해야 합니다.

```typescript
const options = new DocumentBuilder().addOAuth2();
```

#### Cookie authentication

쿠키 인증을 활성화하려면 `@ApiCookieAuth()`를 사용하십시오.

```typescript
@ApiCookieAuth()
@Controller('cats')
export class CatsController {}
```

애플리케이션을 실행하기 전에 `DocumentBuilder`를 사용하여 기본 문서에 보안 정의를 추가해야 합니다.

```typescript
const options = new DocumentBuilder().addCookieAuth('optional-session-id');
```
