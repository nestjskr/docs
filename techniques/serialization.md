### Serialization

직렬화는 개체가 네트워크 응답으로 반환되기 전에 발생하는 프로세스입니다. 클라이언트에 반환할 데이터를 변환하고 삭제하기 위한 규칙을 제공하기에 적합한 장소입니다. 예를 들어 비밀번호와 같은 민감한 데이터는 항상 응답에서 제외되어야합니다. 또는 특정 속성에는 엔터티 속성의 하위 집합만 보내는 것과 같은 추가 변환이 필요할 수 있습니다. 이러한 변환을 수동으로 수행하는 것은 지루하고 오류가 발생하기 쉬우며 모든 사례가 다루어 졌는지 불확실할 수 있습니다.

#### Overview

Nest는 이러한 작업을 간단한 방식으로 수행할 수 있도록 지원하는 기본 제공 기능을 제공합니다. `ClassSerializerInterceptor` 인터셉터는 강력한 [class-transformer](https://github.com/typestack/class-transformer) 패키지를 사용하여 객체를 변환하는 선언적이고 확장 가능한 방법을 제공합니다. 이것이 수행하는 기본 작업은 메소드 핸들러가 반환한 값을 가져와 [class-transformer](https://github.com/typestack/class-transformer)에서 `classToPlain()` 함수를 적용하는 것입니다. 그렇게함으로써 아래에 설명된대로 엔티티/DTO 클래스에 `class-transformer` 데코레이터로 표현된 규칙을 적용할 수 있습니다.

#### Exclude properties

사용자 엔터티에서 `password` 속성을 자동으로 제외하려고 한다고 가정해 보겠습니다. 엔티티에 다음과 같이 주석을 추가합니다.

```typescript
import { Exclude } from 'class-transformer';

export class UserEntity {
  id: number;
  firstName: string;
  lastName: string;

  @Exclude()
  password: string;

  constructor(partial: Partial<UserEntity>) {
    Object.assign(this, partial);
  }
}
```

이제이 클래스의 인스턴스를 반환하는 메서드 처리기가 있는 컨트롤러를 고려하십시오.

```typescript
@UseInterceptors(ClassSerializerInterceptor)
@Get()
findOne(): UserEntity {
  return new UserEntity({
    id: 1,
    firstName: 'Kamil',
    lastName: 'Mysliwiec',
    password: 'password',
  });
}
```

> **경고** 클래스의 인스턴스를 반환해야합니다. 일반 자바스크립트 객체(예: `{{ '{' }} user: new UserEntity() {{ '}' }}`)를 반환하면 객체가 제대로 직렬화되지 않습니다.

> info **힌트** `ClassSerializerInterceptor`는 `@nestjs/common`에서 가져옵니다.

이 엔드포인트가 요청되면 클라이언트는 다음 응답을 받습니다.

```json
{
  "id": 1,
  "firstName": "Kamil",
  "lastName": "Mysliwiec"
}
```

인터셉터는 애플리케이션 전체에 적용될 수 있습니다 ([여기](/interceptors#binding-interceptors)에서 다룹니다). 인터셉터와 엔티티 클래스 선언의 조합은 `UserEntity`를 반환하는 **모든** 메소드가 `password` 속성을 제거하도록 합니다. 이를 통해 이 비즈니스 규칙을 중앙집중식으로 시행할 수 있습니다.

#### Expose properties

아래에 표시된대로 `@Expose()` 데코레이터를 사용하여 속성의 별칭 이름을 제공하거나 속성 값을 계산하는 함수를 실행할 수 있습니다 (**getter** 함수와 유사함).

```typescript
@Expose()
get fullName(): string {
  return `${this.firstName} ${this.lastName}`;
}
```

#### Transform

`@Transform()` 데코레이터를 사용하여 추가 데이터 변환을 수행할 수 있습니다. 예를 들어 다음 구문은 전체 객체를 반환하는 대신 `RoleEntity`의 이름 속성을 반환합니다.

```typescript
@Transform(role => role.name)
role: RoleEntity;
```

#### Pass options

변환 함수의 기본동작을 수정할 수 있습니다. 기본 설정을 재정의하려면 `@SerializeOptions()` 데코레이터를 사용하여 `options` 객체에 전달합니다.

```typescript
@SerializeOptions({
  excludePrefixes: ['_'],
})
@Get()
findOne(): UserEntity {
  return new UserEntity();
}
```

> info **힌트** `@SerializeOptions()` 데코레이터는 `@nestjs/common`에서 가져옵니다.

`@SerializeOptions()`를 통해 전달된 옵션은 기본 `classToPlain()` 함수의 두번째 인수로 전달됩니다. 이 예에서는 `_` 접두사로 시작하는 모든 속성을 자동으로 제외합니다.

#### Example

작동하는 예는 [여기](https://github.com/nestjs/nest/tree/master/sample/21-serializer)에서 확인할 수 있습니다.

#### WebSockets and Microservices

이 장에서는 HTTP 스타일 애플리케이션 (예: Express 또는 Fastify)을 사용하는 예제를 보여 주지만 `ClassSerializerInterceptor`는 사용되는 전송 방법에 관계없이 WebSocket 및 Microservices에서 동일하게 작동합니다.

#### Learn more

`class-transformer` 패키지에서 제공하는 사용 가능한 데코레이터 및 옵션에 대한 자세한 내용은 [여기](https://github.com/typestack/class-transformer)를 참조하세요.