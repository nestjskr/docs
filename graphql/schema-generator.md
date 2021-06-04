### Generating SDL

> warning **경고** 이 장은 코드 우선 접근 방식에만 적용됩니다.

GraphQL SDL 스키마를 수동으로 생성하려면 (즉, 애플리케이션 실행, 데이터베이스 연결, 리졸버 연결 등)`GraphQLSchemaBuilderModule`을 사용합니다.

```typescript
async function generateSchema() {
  const app = await NestFactory.create(GraphQLSchemaBuilderModule);
  await app.init();

  const gqlSchemaFactory = app.get(GraphQLSchemaFactory);
  const schema = await gqlSchemaFactory.create([RecipesResolver]);
  console.log(printSchema(schema));
}
```

> info **힌트** `GraphQLSchemaBuilderModule` 및 `GraphQLSchemaFactory`는 `@nestjs/graphql` 패키지에서 가져옵니다. `printSchema` 함수는 `graphql` 패키지에서 가져옵니다.

#### Usage

`gqlSchemaFactory.create()` 메소드는 리졸버 클래스 참조의 배열을 받습니다. 예를 들면:

```typescript
const schema = await gqlSchemaFactory.create([
  RecipesResolver,
  AuthorsResolver,
  PostsResolvers,
]);
```

또한 스칼라 클래스 배열과 함께 두 번째 선택적 인수를 사용합니다.

```typescript
const schema = await gqlSchemaFactory.create(
  [RecipesResolver, AuthorsResolver, PostsResolvers],
  [DurationScalar, DateScalar],
);
```

마지막으로 옵션 객체를 전달할 수 있습니다.

```typescript
const schema = await gqlSchemaFactory.create([RecipesResolver], {
  skipCheck: true,
  orphanedTypes: [],
});
```

- `skipCheck`: 스키마 유효성 검사 무시; 부울, 기본값은 `false`
- `orphanedTypes`: 생성될 명시적으로 참조되지 않은 (객체 그래프의 일부가 아닌) 클래스 목록입니다. 일반적으로 클래스가 선언되었지만 그래프에서 다른 방법으로 참조되지 않으면 생략됩니다. 속성 값은 클래스 참조의 배열입니다.
