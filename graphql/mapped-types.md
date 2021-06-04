### Mapped types

> warning **경고** 이 장은 코드 우선 접근 방식에만 적용됩니다.

CRUD (만들기(Create)/읽기(Read)/업데이트(Update)/삭제(Delete))와 같은 기능을 구축할 때 기본 엔터티 타입에 대한 변형(variants)을 구성하는 것이 종종 유용합니다. Nest는 이 작업을 보다 편리하게 만들기 위해 타입 변환을 수행하는 여러 유틸리티 함수를 제공합니다.

#### Partial

입력 유효성 검사 타입 (DTO라고도 함)을 빌드할 때 동일한 유형에서 **생성** 및 **업데이트** 변형을 빌드하는 것이 유용한 경우가 많습니다. 예를 들어 **크리에이트** 변형에는 모든 필드가 필요할 수 있지만 **업데이트** 변형은 모든 필드를 선택사항으로 만들 수 있습니다.

Nest는 이 작업을 더 쉽게 만들고 상용구를 최소화하기 위해 `PartialType()` 유틸리티 함수를 제공합니다.

`PartialType()` 함수는 입력 타입의 모든 속성이 선택 사항으로 설정된 유형 (클래스)을 반환합니다. 예를 들어 다음과 같은 **크리에이트** 타입이 있다고 가정합니다.

```typescript
@InputType()
class CreateUserInput {
  @Field()
  email: string;

  @Field()
  password: string;

  @Field()
  firstName: string;
}
```

기본적으로 이러한 필드는 모두 필수입니다. 동일한 필드를 사용하되 각 필드가 선택사항인 타입을 만들려면 클래스 참조 (`CreateUserInput`)를 인수로 전달하는 `PartialType()`을 사용합니다.

```typescript
@InputType()
export class UpdateUserInput extends PartialType(CreateUserInput) {}
```

> info **힌트** `PartialType()` 함수는 `@nestjs/graphql` 패키지에서 가져옵니다.

`PartialType()` 함수는 데코레이터 팩토리에 대한 참조인 선택적 두번째 인수를 사용합니다. 이 인수는 결과 (자식) 클래스에 적용되는 데코레이터 함수를 변경하는 데 사용할 수 있습니다. 지정하지 않으면 자식 클래스는 **부모** 클래스 (첫 번째 인수에서 참조되는 클래스)와 동일한 데코레이터를 효과적으로 사용합니다. 위의 예에서는 `@InputType()` 데코레이터로 주석이 달린 `CreateUserInput`을 확장합니다. `UpdateUserInput`도 `@InputType()`으로 장식된 것처럼 처리되기를 원하므로 두번째 인자로 `InputType`을 전달할 필요가 없습니다. 부모 타입과 자식 타입이 다른 경우 (예: 부모가 `@ObjectType`으로 장식됨) 두번째 인수로 `InputType`을 전달합니다. 예를 들면:

```typescript
@InputType()
export class UpdateUserInput extends PartialType(User, InputType) {}
```

#### Pick

`PickType()` 함수는 입력 타입에서 속성 집합을 선택하여 새로운 타입 (클래스)을 생성합니다. 예를 들어 다음과 같은 타입으로 시작한다고 가정합니다.

```typescript
@InputType()
class CreateUserInput {
  @Field()
  email: string;

  @Field()
  password: string;

  @Field()
  firstName: string;
}
```

We can pick a set of properties from this class using the `PickType()` utility function:
`PickType()` 유틸리티 함수를 사용하여 이 클래스에서 속성 집합을 선택할 수 있습니다.

```typescript
@InputType()
export class UpdateEmailInput extends PickType(CreateUserInput, ['email'] as const) {}
```

> info **힌트** `PickType()` 함수는 `@nestjs/graphql` 패키지에서 가져옵니다.

#### Omit

`OmitType()` 함수는 입력 타입에서 모든 속성을 선택한 다음 특정 키 세트를 제거하여 유형을 구성합니다. 예를 들어 다음과 같은 유형으로 시작한다고 가정합니다.

```typescript
@InputType()
class CreateUserInput {
  @Field()
  email: string;

  @Field()
  password: string;

  @Field()
  firstName: string;
}
```

아래와 같이 `email`을 **제외**한 모든 속성을 가진 파생 유형을 생성할 수 있습니다. 이 구조에서 `OmitType`의 두번째 인수는 속성 이름의 배열입니다.

```typescript
@InputType()
export class UpdateUserInput extends OmitType(CreateUserInput, ['email'] as const) {}
```

> info **힌트** `OmitType()` 함수는 `@nestjs/graphql` 패키지에서 가져옵니다.

#### Intersection

`IntersectionType()` 함수는 두 타입을 하나의 새로운 타입 (클래스)으로 결합합니다. 예를 들어 다음과 같은 두 가지 타입으로 시작한다고 가정합니다.

```typescript
@InputType()
class CreateUserInput {
  @Field()
  email: string;

  @Field()
  password: string;
}

@ObjectType()
export class AdditionalUserInfo {
  @Field()
  firstName: string;

  @Field()
  lastName: string;
}
```

두 유형의 모든 속성을 결합하는 새 유형을 생성 할 수 있습니다.

```typescript
@InputType()
export class UpdateUserInput extends IntersectionType(CreateUserInput, AdditionalUserInfo) {}
```

> info **힌트** `IntersectionType()` 함수는 `@nestjs/graphql` 패키지에서 가져옵니다.

#### Composition

타입 매핑 유틸리티 함수는 구성 가능합니다. 예를 들어, 다음은 `email`을 제외한 `CreateUserInput` 타입의 모든 속성을 가진 타입 (클래스)을 생성하며 해당 속성은 선택 사항으로 설정됩니다.

```typescript
@InputType()
export class UpdateUserInput extends PartialType(
  OmitType(CreateUserInput, ['email'] as const),
) {}
```
