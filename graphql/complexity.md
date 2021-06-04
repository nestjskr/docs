### Complexity

> warning **경고** 이 장은 코드 우선 접근 방식에만 적용됩니다.

쿼리 복잡성을 사용하면 특정 필드의 복잡성을 정의하고 **최대 복잡성**으로 쿼리를 제한할 수 있습니다. 아이디어는 간단한 숫자를 사용하여 각 필드가 얼마나 복잡한 지 정의하는 것입니다. 일반적인 기본값은 각 필드에 `1`의 복잡성을 부여하는 것입니다. 또한 GraphQL 쿼리의 복잡도 계산은 소위 복잡도 추정기로 사용자 정의할 수 있습니다. 복잡도 추정기는 필드의 복잡도를 계산하는 간단한 함수입니다. 규칙에 복잡도 추정기를 원하는만큼 추가할 수 있으며, 규칙은 차례로 실행됩니다. 숫자 복잡도 값을 반환하는 첫번째 추정기는 해당 필드의 복잡도를 결정합니다.

`@nestjs/graphql` 패키지는 비용 분석 기반 솔루션을 제공하는 [graphql-query-complexity](https://github.com/slicknode/graphql-query-complexity)와 같은 도구와 매우 잘 통합됩니다. 이 라이브러리를 사용하면 실행 비용이 너무 많이 드는 GraphQL 서버에 대한 쿼리를 거부할 수 있습니다.

#### Installation

사용을 시작하려면 먼저 필요한 종속성을 설치합니다.

```bash
$ npm install --save graphql-query-complexity
```

#### Getting started

설치 프로세스가 완료되면 `ComplexityPlugin` 클래스를 정의할 수 있습니다.

```typescript
import { GraphQLSchemaHost, Plugin } from '@nestjs/graphql';
import {
  ApolloServerPlugin,
  GraphQLRequestListener,
} from 'apollo-server-plugin-base';
import { GraphQLError } from 'graphql';
import {
  fieldExtensionsEstimator,
  getComplexity,
  simpleEstimator,
} from 'graphql-query-complexity';

@Plugin()
export class ComplexityPlugin implements ApolloServerPlugin {
  constructor(private gqlSchemaHost: GraphQLSchemaHost) {}

  requestDidStart(): GraphQLRequestListener {
    const { schema } = this.gqlSchemaHost;

    return {
      didResolveOperation({ request, document }) {
        const complexity = getComplexity({
          schema,
          operationName: request.operationName,
          query: document,
          variables: request.variables,
          estimators: [
            fieldExtensionsEstimator(),
            simpleEstimator({ defaultComplexity: 1 }),
          ],
        });
        if (complexity >= 20) {
          throw new GraphQLError(
            `Query is too complex: ${complexity}. Maximum allowed complexity: 20`,
          );
        }
        console.log('Query Complexity:', complexity);
      },
    };
  }
}
```

데모 목적으로 허용되는 최대 복잡성을 `20`으로 지정했습니다. 위의 예에서는 `simpleEstimator`와  `fieldExtensionsEstimator`라는 2 개의 추정기를 사용했습니다.

- `simpleEstimator`: 단순 추정기는 각 필드에 대해 고정된 복잡도를 반환합니다.
- `fieldExtensionsEstimator`: 필드 확장 추정기는 스키마의 각 필드에 대한 복잡성 값을 추출합니다.

> info **힌트** 이 클래스를 모든 모듈의 프로바이더 배열에 추가하는 것을 잊지 마십시오.

#### Field-level complexity

이 플러그인을 사용하면 이제 다음과 같이 `@Field()` 데코레이터에 전달된 옵션 객체에 `complexity` 속성을 지정하여 모든 필드의 복잡성을 정의할 수 있습니다.

```typescript
@Field({ complexity: 3 })
title: string;
```

또는 추정 기능을 정의할 수 있습니다.

```typescript
@Field({ complexity: (options: ComplexityEstimatorArgs) => ... })
title: string;
```

#### Query/Mutation-level complexity

또한 `@Query()` 및 `@Mutation()` 데코레이터에는 다음과 같이 지정된 `complexity` 속성이 있을 수 있습니다.

```typescript
@Query({ complexity: (options: ComplexityEstimatorArgs) => options.args.count * options.childComplexity })
items(@Args('count') count: number) {
  return this.itemsService.getItems({ count });
}
```
