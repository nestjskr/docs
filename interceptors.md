### Interceptors

인터셉터는 `@Injectable()` 데코레이터로 주석이 달린 클래스입니다. 인터셉터는 `NestInterceptor` 인터페이스를 구현해야 합니다.

<figure><img src="/assets/Interceptors_1.png" /></figure>

인터셉터에는 [AOP(Aspect Oriented Programming)](https://en.wikipedia.org/wiki/Aspect-oriented_programming) 기술에서 영감을 받은 유용한 기능 세트가 있습니다. 이를 통해 다음을 수행할 수 있습니다.

- 메소드 실행 전/후 추가 로직 바인딩
- 함수에서 반환된 결과를 변환
- 함수에서 던져진 예외 변환
- 기본 기능 동작 확장
- 특정 조건에 따라 기능을 완전히 재정의합니다(예: 캐싱 목적)

#### Basics

각 인터셉터는 두개의 인수를 취하는 `intercept()` 메소드를 구현합니다. 첫번째는 `ExecutionContext` 인스턴스입니다 ([가드](/guards)와 정확히 동일한 객체). `ExecutionContext`는 `ArgumentsHost`에서 상속됩니다. 앞서 예외필터 장에서 `ArgumentsHost`를 보았습니다. 여기에서 원래 핸들러에 전달된 인수를 둘러싼 래퍼이며 애플리케이션 유형에 따라 다른 인수 배열을 포함하고 있음을 알았습니다. 이 주제에 대한 자세한 내용은 [예외필터](/exception-filters#arguments-host)를 다시 참조할 수 있습니다.

#### Execution context

`ArgumentsHost`를 확장함으로써 `ExecutionContext`는 현재 실행 프로세스에 대한 추가 세부정보를 제공하는 몇가지 새로운 헬퍼 메서드도 추가합니다. 이러한 세부정보는 광범위한 컨트롤러, 메서드 및 실행 컨텍스트에서 작동할 수 있는 보다 일반적인 인터셉터를 빌드하는데 도움이 될 수 있습니다. [여기](/fundamentals/execution-context)에서 `ExecutionContext`에 대해 자세히 알아보세요.

#### Call handler

두번째 인수는 `CallHandler`입니다. `CallHandler` 인터페이스는 인터셉터의 특정지점에서 라우트 핸들러 메서드를 호출하는데 사용할 수 있는 `handle()` 메서드를 구현합니다. `intercept()` 메서드 구현에서 `handle()` 메서드를 호출하지 않으면 라우트 핸들러 메서드가 전혀 실행되지 않습니다.

이 접근방식은 `intercept()` 메서드가 요청/응답 스트림을 효과적으로 **포장**합니다. 결과적으로 최종 라우트 핸들러 실행 **전과 후에** 커스텀 로직을 구현할 수 있습니다. `handle()`을 호출하기 **전에** 실행되는 `intercept()` 메서드에 코드를 작성할 수 있다는 것은 분명하지만 이후에 일어나는 일에 어떤 영향을 미칠까요? `handle()` 메서드는 `Observable`을 반환하기 때문에 강력한 [RxJS](https://github.com/ReactiveX/rxjs) 연산자를 사용하여 응답을 추가로 조작할 수 있습니다. AOP(Aspect Oriented Programming) 용어를 사용하여 라우트 핸들러의 호출(즉, `handle()`호출)을 [Pointcut](https://en.wikipedia.org/wiki/Pointcut)이라고 하며, 추가 로직이 삽입됩니다.

예를 들어 들어오는 `POST /cats` 요청을 고려하십시오. 이 요청은 `CatsController` 내에 정의된 `create()` 핸들러를 대상으로 합니다. `handle()` 메서드를 호출하지 않는 인터셉터가 도중에 호출되면 `create()` 메서드가 실행되지 않습니다. `handle()`이 호출되면 (그리고 `Observable`이 반환되면) `create()` 핸들러가 트리거됩니다. 그리고 응답스트림이 `Observable`을 통해 수신되면 스트림에서 추가작업을 수행할 수 있으며 최종 결과가 호출자에게 반환됩니다.

<app-banner-shop></app-banner-shop>

#### Aspect interception

우리가 살펴볼 첫번째 사용사례는 인터셉터를 사용하여 사용자 상호작용을 기록하는 것입니다 (예: 사용자 호출저장, 이벤트를 비동기적으로 전달하거나 타임스탬프 계산). 아래에 간단한 `LoggingInterceptor`가 표시됩니다.

```typescript
@@filename(logging.interceptor)
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before...');

    const now = Date.now();
    return next
      .handle()
      .pipe(
        tap(() => console.log(`After... ${Date.now() - now}ms`)),
      );
  }
}
@@switch
import { Injectable } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor {
  intercept(context, next) {
    console.log('Before...');

    const now = Date.now();
    return next
      .handle()
      .pipe(
        tap(() => console.log(`After... ${Date.now() - now}ms`)),
      );
  }
}
```

> info **힌트** `NestInterceptor<T, R>`는 `T`가 `Observable<T>`(응답 스트림 지원)의 타입을 나타내고 `R`은 `Observable<R>`로 래핑된 값입니다.

> warning **알림** 컨트롤러, 프로바이더, 가드 등과 같은 인터셉터는 `constuctor`를 통해 **종속성을 주입**할 수 있습니다.

`handle()`은 RxJS `Observable`을 반환하므로 스트림을 조작하는 데 사용할 수 있는 다양한 연산자를 선택할 수 있습니다. 위의 예에서 우리는 관찰 가능한 스트림이 우아하거나 예외적으로 종료될 때 익명 로깅 함수를 호출하지만 응답주기를 방해하지 않는 `tap()` 연산자를 사용했습니다.

#### Binding interceptors

인터셉터를 설정하기 위해 `@nestjs/common` 패키지에서 가져온 `@UseInterceptors()` 데코레이터를 사용합니다. [파이프](/pipes) 및 [가드](/guards)와 마찬가지로 인터셉터는 컨트롤러 범위, 메서드 범위 또는 전역범위일 수 있습니다.

```typescript
@@filename(cats.controller)
@UseInterceptors(LoggingInterceptor)
export class CatsController {}
```

> info **힌트** `@UseInterceptors()` 데코레이터는 `@nestjs/common` 패키지에서 가져옵니다.

위의 구성을 사용하여 `CatsController`에 정의된 각 라우트 핸들러는 `LoggingInterceptor`를 사용합니다. 누군가 `GET /cats` 엔드포인트를 호출하면 표준출력에 다음과 같은 출력이 표시됩니다.

```typescript
Before...
After... 1ms
```

인스턴스 대신 `LoggingInterceptor` 타입을 전달하여 인스턴스화를 프레임워크에 맡기고 종속성 주입을 활성화했습니다. 파이프, 가드, 예외필터와 마찬가지로 내부 인스턴스도 전달할 수 있습니다.

```typescript
@@filename(cats.controller)
@UseInterceptors(new LoggingInterceptor())
export class CatsController {}
```

언급했듯이 위의 구성은 이 컨트롤러가 선언한 모든 핸들러에 인터셉터를 연결합니다. 인터셉터의 범위를 단일 메서드로 제한하려면 **메서드 수준**에서 데코레이터를 적용하면 됩니다.

전역 인터셉터를 설정하기 위해 Nest 애플리케이션 인스턴스의 `useGlobalInterceptors()` 메서드를 사용합니다.

```typescript
const app = await NestFactory.create(AppModule);
app.useGlobalInterceptors(new LoggingInterceptor());
```

글로벌 인터셉터는 모든 컨트롤러와 모든 라우트 핸들러에 대해 전체 애플리케이션에서 사용됩니다. 의존성 주입과 관련하여 모듈 외부에서 등록된 전역 인터셉터(위의 예에서와 같이 `useGlobalInterceptors()` 사용)는 모듈의 컨텍스트 외부에서 수행되므로 종속성을 주입할 수 없습니다. 이 문제를 해결하기 위해 다음 구성을 사용하여 **모든 모듈에서 직접** 인터셉터를 설정할 수 있습니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor,
    },
  ],
})
export class AppModule {}
```

> info **힌트** 이 접근방식을 사용하여 인터셉터에 대한 종속성 주입을 수행할 때 이 구성이 사용되는 모듈에 관계없이 인터셉터는 실제로 전역적입니다. 어디에서 해야 합니까? 인터셉터(위의 예에서는 `LoggingInterceptor`)가 정의된 모듈을 선택합니다. 또한 `useClass`가 커스텀 프로바이더 등록을 처리하는 유일한 방법은 아닙니다. [여기](/fundamentals/custom-providers)에서 자세히 알아보세요.

#### Response mapping

우리는 이미 `handle()`이 `Observable`을 반환한다는 것을 알고 있습니다. 스트림에는 라우트 핸들러에서 **반환된** 값이 포함되어 있으므로 RxJS의 `map()` 연산자를 사용하여 쉽게 변경할 수 있습니다.

> warning **경고** 응답 매핑 기능은 라이브러리별 응답 전략에서 작동하지 않습니다 (`@Res()` 객체를 직접 사용하는 것은 금지됨).

프로세스를 보여주는 간단한 방법으로 각 응답을 수정하는 `TransformInterceptor`를 만들어 보겠습니다. RxJS의 `map()` 연산자를 사용하여 새로 생성된 객체의 `data` 속성에 응답 객체를 할당하고 새 객체를 클라이언트에 반환합니다.

```typescript
@@filename(transform.interceptor)
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

export interface Response<T> {
  data: T;
}

@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, Response<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<Response<T>> {
    return next.handle().pipe(map(data => ({ data })));
  }
}
@@switch
import { Injectable } from '@nestjs/common';
import { map } from 'rxjs/operators';

@Injectable()
export class TransformInterceptor {
  intercept(context, next) {
    return next.handle().pipe(map(data => ({ data })));
  }
}
```

> info **힌트** Nest 인터셉터는 동기 및 비동기 `intercept()` 메서드 모두에서 작동합니다. 필요한 경우 메서드를 `async`로 간단히 전환할 수 있습니다.

위의 구성에서 누군가 `GET /cats` 엔드포인트를 호출하면 응답은 다음과 같습니다 (라우트 핸들러가 빈 배열 `[]`을 반환한다고 가정).

```json
{
  "data": []
}
```

인터셉터는 전체 애플리케이션에서 발생하는 요구사항에 대한 재사용 가능한 솔루션을 만드는데 큰 가치를 둡니다.
예를 들어, `null`값의 각 항목을 빈 문자열 `''`로 변환해야 한다고 가정해보십시오. 한줄의 코드를 사용하여 이를 수행하고 인터셉터를 전역적으로 바인딩하여 등록된 각 핸들러가 자동으로 사용하도록 할 수 있습니다.

```typescript
@@filename()
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable()
export class ExcludeNullInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next
      .handle()
      .pipe(map(value => value === null ? '' : value ));
  }
}
@@switch
import { Injectable } from '@nestjs/common';
import { map } from 'rxjs/operators';

@Injectable()
export class ExcludeNullInterceptor {
  intercept(context, next) {
    return next
      .handle()
      .pipe(map(value => value === null ? '' : value ));
  }
}
```

#### Exception mapping

또 다른 흥미로운 사용사례는 RxJS의 `catchError()` 연산자를 활용하여 던져진(throw) 예외를 재정의하는 것입니다.

```typescript
@@filename(errors.interceptor)
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  BadGatewayException,
  CallHandler,
} from '@nestjs/common';
import { Observable, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';

@Injectable()
export class ErrorsInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next
      .handle()
      .pipe(
        catchError(err => throwError(new BadGatewayException())),
      );
  }
}
@@switch
import { Injectable, BadGatewayException } from '@nestjs/common';
import { throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';

@Injectable()
export class ErrorsInterceptor {
  intercept(context, next) {
    return next
      .handle()
      .pipe(
        catchError(err => throwError(new BadGatewayException())),
      );
  }
}
```

#### Stream overriding

때때로 핸들러 호출을 완전히 방지하고 대신 다른 값을 반환하려는 몇가지 이유가 있습니다. 분명한 예는 응답시간을 개선하기 위해 캐시를 구현하는 것입니다. 캐시에서 응답을 반환하는 간단한 **캐시 인터셉터**를 살펴 보겠습니다. 현실적인 예에서는 TTL, 캐시 무효화, 캐시 크기 등과 같은 다른 요소를 고려하고 싶지만 이 논의의 범위를 벗어납니다. 여기서는 주요 개념을 보여주는 기본 예를 제공합니다.

```typescript
@@filename(cache.interceptor)
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable, of } from 'rxjs';

@Injectable()
export class CacheInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const isCached = true;
    if (isCached) {
      return of([]);
    }
    return next.handle();
  }
}
@@switch
import { Injectable } from '@nestjs/common';
import { of } from 'rxjs';

@Injectable()
export class CacheInterceptor {
  intercept(context, next) {
    const isCached = true;
    if (isCached) {
      return of([]);
    }
    return next.handle();
  }
}
```

`CacheInterceptor`에는 하드코딩된 `isCached` 변수와 하드코딩된 응답 `[]`도 있습니다. 주목해야할 요점은 RxJS `of()` 연산자에 의해 생성된 새 스트림을 여기에 반환하므로 라우트 핸들러가 **전혀 호출되지 않습니다**. 누군가 `CacheInterceptor`를 사용하는 엔드포인트를 호출하면 응답(하드코딩된 빈 배열)이 즉시 반환됩니다. 일반적인 솔루션을 만들기 위해 `Reflector`를 활용하고 맞춤 데코레이터를 만들 수 있습니다. `Reflector`는 [가드](/guards) 장에 잘 설명되어 있습니다.

#### More operators

RxJS 연산자를 사용하여 스트림을 조작할 수 있는 가능성은 우리에게 많은 기능을 제공합니다. 또 다른 일반적인 사용사례를 살펴 보겠습니다. 라우트 요청에서 **시간 초과**를 처리하고 싶다고 가정해보십시오. 일정시간이 지나도 엔드포인트가 아무것도 반환하지 않으면 오류 응답으로 종료하려고 합니다. 다음 구성은 이를 가능하게합니다.

```typescript
@@filename(timeout.interceptor)
import { Injectable, NestInterceptor, ExecutionContext, CallHandler, RequestTimeoutException } from '@nestjs/common';
import { Observable, throwError, TimeoutError } from 'rxjs';
import { catchError, timeout } from 'rxjs/operators';

@Injectable()
export class TimeoutInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      timeout(5000),
      catchError(err => {
        if (err instanceof TimeoutError) {
          return throwError(new RequestTimeoutException());
        }
        return throwError(err);
      }),
    );
  };
};
@@switch
import { Injectable, RequestTimeoutException } from '@nestjs/common';
import { Observable, throwError, TimeoutError } from 'rxjs';
import { catchError, timeout } from 'rxjs/operators';

@Injectable()
export class TimeoutInterceptor {
  intercept(context, next) {
    return next.handle().pipe(
      timeout(5000),
      catchError(err => {
        if (err instanceof TimeoutError) {
          return throwError(new RequestTimeoutException());
        }
        return throwError(err);
      }),
    );
  };
};
```

5초후 요청 처리가 취소됩니다. `RequestTimeoutException`을 발생시키기 전에 커스텀 로직을 추가할 수도 있습니다(예: 리소스 해제).
