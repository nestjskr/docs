### Standalone applications

Nest 애플리케이션을 마운트하는 방법에는 여러가지가 있습니다. 웹 앱, 마이크로 서비스 또는 베어 Nest **독립형 애플리케이션** (네트워크 리스너없이)만 만들 수 있습니다. Nest 독립형 애플리케이션은 모든 인스턴스화된 클래스를 보유하는 Nest **IoC 컨테이너**를 둘러싼 래퍼입니다. 독립 실행형 애플리케이션 객체를 사용하여 가져온 모듈 내에서 직접 기존 인스턴스에 대한 참조를 얻을 수 있습니다. 따라서 스크립팅된 **CRON** 작업을 포함하여 어디서나 Nest 프레임워크를 활용할 수 있습니다. 그 위에 **CLI**를 구축할 수도 있습니다.

#### Getting started

Nest 독립형 애플리케이션을 만들려면 다음 구성을 사용하세요.

```typescript
@@filename()
async function bootstrap() {
  const app = await NestFactory.createApplicationContext(AppModule);
  // application logic...
}
bootstrap();
```

독립형 애플리케이션 객체를 사용하면 Nest 애플리케이션에 등록된 모든 인스턴스에 대한 참조를 얻을 수 있습니다. `TasksModule`에 `TasksService`가 있다고 가정해 봅시다. 이 클래스는 CRON 작업 내에서 호출하려는 메서드 집합을 제공합니다.

```typescript
@@filename()
const app = await NestFactory.create(AppModule);
const tasksService = app.get(TasksService);
```

`TasksService` 인스턴스에 액세스하려면 `get()` 메소드를 사용합니다. `get()` 메서드는 등록된 각 모듈에서 인스턴스를 검색하는 **쿼리**처럼 작동합니다. 또는 엄격한 컨텍스트 확인을 위해 `strict: true` 속성과 함께 옵션 객체를 전달합니다. 이 옵션이 적용되면 선택한 컨텍스트에서 특정 인스턴스를 가져 오려면 특정 모듈을 탐색해야 합니다.

```typescript
@@filename()
const app = await NestFactory.create(AppModule);
const tasksService = app.select(TasksModule).get(TasksService, { strict: true });
```

다음은 독립 실행형 애플리케이션 객체에서 인스턴스 참조를 검색하는 데 사용할 수 있는 메서드를 요약한 것입니다.

<table>
  <tr>
    <td>
      <code>get()</code>
    </td>
    <td>
      애플리케이션 컨텍스트에서 사용 가능한 컨트롤러 또는 프로바이더(가드, 필터 등 포함)의 인스턴스를 검색합니다.
    </td>
  </tr>
  <tr>
    <td>
      <code>select()</code>
    </td>
    <td>
      모듈 그래프를 탐색하여 선택한 모듈에서 특정 인스턴스를 가져옵니다 (위에서 설명한대로 엄격(strict) 모드와 함께 사용됨).
    </td>
  </tr>
</table>

> info **힌트** 엄격하지 않은(not-strict) 모드에서는 기본적으로 루트 모듈이 선택됩니다. 다른 모듈을 선택하려면 모듈 그래프를 단계별로 수동으로 탐색해야 합니다.

스크립트가 완료된 후 노드 애플리케이션을 닫으려면(예: CRON 작업을 실행하는 스크립트의 경우) `bootstrap` 함수 끝에 `await app.close()`를 추가합니다.

```typescript
@@filename()
async function bootstrap() {
  const app = await NestFactory.createApplicationContext(AppModule);
  // application logic...
  await app.close();
}
bootstrap();
```

#### Example

실제 예제는 [여기](https://github.com/nestjs/nest/tree/master/sample/18-context)에서 확인할 수 있습니다.
