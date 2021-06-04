### Exception filters

HTTP [예외 필터](/exception-filters) 계층과 해당 마이크로서비스 계층의 유일한 차이점은 `HttpException`을 발생시키는 대신 `RpcException`을 사용해야한다는 것입니다.

```typescript
throw new RpcException('Invalid credentials.');
```

> info **힌트**`RpcException` 클래스는 `@nestjs/microservices` 패키지에서 가져옵니다.

위의 샘플에서 Nest는 던져진 예외를 처리하고 다음 구조로 `error` 객체를 반환합니다.

```json
{
  "status": "error",
  "message": "Invalid credentials."
}
```

#### Filters

마이크로서비스 예외 필터는 HTTP 예외 필터와 유사하게 작동하지만 한가지 작은 차이가 있습니다. `catch()` 메소드는 `Observable`을 반환해야합니다.

```typescript
@@filename(rpc-exception.filter)
import { Catch, RpcExceptionFilter, ArgumentsHost } from '@nestjs/common';
import { Observable, throwError } from 'rxjs';
import { RpcException } from '@nestjs/microservices';

@Catch(RpcException)
export class ExceptionFilter implements RpcExceptionFilter<RpcException> {
  catch(exception: RpcException, host: ArgumentsHost): Observable<any> {
    return throwError(exception.getError());
  }
}
@@switch
import { Catch } from '@nestjs/common';
import { throwError } from 'rxjs';

@Catch(RpcException)
export class ExceptionFilter {
  catch(exception, host) {
    return throwError(exception.getError());
  }
}
```

> warning **경고** [하이브리드 애플리케이션](/faq/hybrid-application)을 사용하는 경우 전역 마이크로서비스 예외 필터를 설정할 수 없습니다.

다음 예제에서는 수동으로 인스턴스화된 메서드 범위 필터를 사용합니다. HTTP 기반 애플리케이션과 마찬가지로 컨트롤러 범위 필터를 사용할 수도 있습니다 (예: 컨트롤러 클래스에 `@UseFilters()` 데코레이터를 접두사로 추가).

```typescript
@@filename()
@UseFilters(new ExceptionFilter())
@MessagePattern({ cmd: 'sum' })
accumulate(data: number[]): number {
  return (data || []).reduce((a, b) => a + b);
}
@@switch
@UseFilters(new ExceptionFilter())
@MessagePattern({ cmd: 'sum' })
accumulate(data) {
  return (data || []).reduce((a, b) => a + b);
}
```

#### Inheritance

일반적으로 애플리케이션 요구사항을 충족하도록 제작된 완전히 사용자 지정된 예외 필터를 만듭니다. 그러나 **핵심 예외 필터**를 간단히 확장하고 특정 요인에 따라 동작을 재정의하려는 사용 사례가 있을 수 있습니다.

예외 처리를 기본 필터에 위임하려면 `BaseExceptionFilter`를 확장하고 상속된 `catch()` 메서드를 호출해야합니다.

```typescript
@@filename()
import { Catch, ArgumentsHost } from '@nestjs/common';
import { BaseRpcExceptionFilter } from '@nestjs/microservices';

@Catch()
export class AllExceptionsFilter extends BaseRpcExceptionFilter {
  catch(exception: any, host: ArgumentsHost) {
    return super.catch(exception, host);
  }
}
@@switch
import { Catch } from '@nestjs/common';
import { BaseRpcExceptionFilter } from '@nestjs/microservices';

@Catch()
export class AllExceptionsFilter extends BaseRpcExceptionFilter {
  catch(exception, host) {
    return super.catch(exception, host);
  }
}
```

위의 구현은 접근 방식을 보여주는 셸일뿐입니다. 확장된 예외 필터의 구현에는 맞춤형 **비즈니스 로직**(예: 다양한 조건 처리)이 포함됩니다.
