### Exception filters

HTTP [예외 필터](/exception-filters) 레이어와 해당 웹소켓 레이어의 유일한 차이점은 `HttpException`을 던지는 대신 `WsException`을 사용해야한다는 것입니다.

```typescript
throw new WsException('Invalid credentials.');
```

> info **힌트** `WsException` 클래스는 `@nestjs/websockets` 패키지에서 가져옵니다.

위의 샘플을 사용하여 Nest는 던져진 예외를 처리하고 다음 구조로 `exception` 메시지를 내 보냅니다.

```typescript
{
  status: 'error',
  message: 'Invalid credentials.'
}
```

#### Filters

웹소켓 예외 필터는 HTTP 예외 필터와 동일하게 작동합니다. 다음 예제에서는 수동으로 인스턴스화된 메서드 범위 필터를 사용합니다. HTTP 기반 애플리케이션과 마찬가지로 게이트웨이 범위 필터를 사용할 수도 있습니다(즉, 게이트웨이 클래스 앞에 `@UseFilters()` 데코레이터를 붙임).

```typescript
@UseFilters(new WsExceptionFilter())
@SubscribeMessage('events')
onEvent(client, data: any): WsResponse<any> {
  const event = 'events';
  return { event, data };
}
```

#### Inheritance

일반적으로 애플리케이션 요구 사항을 충족하도록 제작된 완전히 사용자 지정된 예외 필터를 만듭니다. 그러나 **핵심 예외 필터**를 간단히 확장하고 특정 요인에 따라 동작을 재정의하려는 사용 사례가 있을 수 있습니다.

예외 처리를 기본 필터에 위임하려면 `BaseWsExceptionFilter`를 확장하고 상속된 `catch()` 메서드를 호출해야합니다.

```typescript
@@filename()
import { Catch, ArgumentsHost } from '@nestjs/common';
import { BaseWsExceptionFilter } from '@nestjs/websockets';

@Catch()
export class AllExceptionsFilter extends BaseWsExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    super.catch(exception, host);
  }
}
@@switch
import { Catch } from '@nestjs/common';
import { BaseWsExceptionFilter } from '@nestjs/websockets';

@Catch()
export class AllExceptionsFilter extends BaseWsExceptionFilter {
  catch(exception, host) {
    super.catch(exception, host);
  }
}
```

위의 구현은 접근 방식을 보여주는 셸일 뿐입니다. 확장된 예외 필터의 구현에는 맞춤형 **비즈니스 로직**(예: 다양한 조건 처리)이 포함됩니다.
