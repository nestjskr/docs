### Extensions

> warning **경고** 이 장은 코드 우선 접근 방식에만 적용됩니다.

익스텐션은 타입 구성에서 임의의 데이터를 정의할 수 있는 **고급, 저 레벨 기능**입니다. 특정 필드에 사용자 지정 메타 데이터를 첨부하면 보다 정교하고 일반적인 솔루션을 만들 수 있습니다. 예를 들어 익스텐션을 사용하면 특정 필드에 액세스하는 데 필요한 필드 수준 역할을 정의할 수 있습니다. 이러한 역할은 런타임에 반영되어 호출자(caller)가 특정 필드를 검색할 수 있는 충분한 권한이 있는지 확인할 수 있습니다.

#### Adding custom metadata

필드에 대한 사용자 지정 메타 데이터를 첨부하려면 `@nestjs/graphql` 패키지에서 내 보낸 `@Extensions()` 데코레이터를 사용합니다.

```typescript
@Field()
@Extensions({ role: Role.ADMIN })
password: string;
```

위의 예에서는 `role` 메타 데이터 속성에 `Role.ADMIN`값을 할당했습니다. `Role`은 시스템에서 사용 가능한 모든 사용자 역할을 그룹화하는 간단한 TypeScript enum 형입니다.

필드에 메타 데이터를 설정하는 것 외에도 클래스 수준 및 메서드 수준 (예: 쿼리 핸들러에서)에서 `@Extensions()` 데코레이터를 사용할 수 있습니다.

#### Using custom metadata

사용자 지정 메타 데이터를 활용하는 논리는 필요에 따라 복잡할 수 있습니다. 예를 들어 메서드 호출당 이벤트를 저장/기록하는 간단한 인터셉터 또는 호출자 권한이 있는 필드를 검색하는 데 필요한 역할과 일치하는 [필드 미들웨어](/graphql/field-middleware)를 만들 수 있습니다 (필드 수준 권한 시스템).

설명을 위해 사용자의 역할 (여기에 하드 코딩 됨)을 대상 필드에 액세스하는 데 필요한 역할과 비교하는 `checkRoleMiddleware`를 정의하겠습니다.

```typescript
export const checkRoleMiddleware: FieldMiddleware = async (
  ctx: MiddlewareContext,
  next: NextFn,
) => {
  const { info } = ctx;
  const { extensions } = info.parentType.getFields()[info.fieldName];

  /**
   * In a real-world application, the "userRole" variable
   * should represent the caller's (user) role (for example, "ctx.user.role").
   */
  const userRole = Role.USER;
  if (userRole === extensions.role) {
    // or just "return null" to ignore
    throw new ForbiddenException(
      `User does not have sufficient permissions to access "${info.fieldName}" field.`,
    );
  }
  return next();
};
```

이를 통해 다음과 같이 `password` 필드에 대한 미들웨어를 등록할 수 있습니다.

```typescript
@Field({ middleware: [checkRoleMiddleware] })
@Extensions({ role: Role.ADMIN })
password: string;
```
