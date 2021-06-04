### Plugins

플러그인을 사용하면 특정 이벤트에 대한 응답으로 사용자 지정 작업을 수행하여 Apollo Server의 핵심 기능을 확장할 수 있습니다. 현재 이러한 이벤트는 GraphQL 요청 수명주기의 개별 단계와 Apollo Server 자체의 시작에 해당합니다 (자세한 내용은 [여기](https://www.apollographql.com/docs/apollo-server/integrations/plugins/) 참조). 예를 들어 기본 로깅 플러그인은 Apollo 서버로 전송되는 각 요청과 관련된 GraphQL 쿼리 문자열을 로깅할 수 있습니다.

#### Custom plugins

플러그인을 생성하려면 `@nestjs/graphql` 패키지에서 내 보낸 `@Plugin` 데코레이터로 주석이 달린 클래스를 선언합니다. 또한 더 나은 코드 자동 완성을 위해 `apollo-server-plugin-base` 패키지에서 `ApolloServerPlugin` 인터페이스를 구현하십시오.

```typescript
import { Plugin } from '@nestjs/graphql';
import {
  ApolloServerPlugin,
  GraphQLRequestListener,
} from 'apollo-server-plugin-base';

@Plugin()
export class LoggingPlugin implements ApolloServerPlugin {
  requestDidStart(): GraphQLRequestListener {
    console.log('Request started');
    return {
      willSendResponse() {
        console.log('Will send response');
      },
    };
  }
}
```

이를 통해 `LoggingPlugin`을 프로바이더로 등록할 수 있습니다.

```typescript
@Module({
  providers: [LoggingPlugin],
})
export class CommonModule {}
```

Nest는 자동으로 플러그인을 인스턴스화하여 Apollo 서버에 적용합니다.

#### Using external plugins

기본적으로 제공되는 여러 플러그인이 있습니다. 기존 플러그인을 사용하려면 간단히 가져와서 `plugins` 배열에 추가하면 됩니다.

```typescript
GraphQLModule.forRoot({
  // ...
  plugins: [ApolloServerOperationRegistry({ /* options */})]
}),
```

> info **힌트** `ApolloServerOperationRegistry` 플러그인은 `apollo-server-plugin-operation-registry` 패키지에서 내보내집니다.
