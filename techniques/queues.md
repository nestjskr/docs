### Queues

대기열은 일반적인 애플리케이션 확장 및 성능 문제를 처리하는 데 도움이 되는 강력한 디자인 패턴입니다. 대기열 해결하는데 도움이 될 수 있는 문제의 몇가지 예는 다음과 같습니다.

- 처리 피크를 부드럽게합니다. 예를 들어 사용자가 리소스 집약적인 작업을 임의의 시간에 시작할 수 있는 경우 이러한 작업을 동기적으로 수행하는 대신 대기열에 추가할 수 있습니다. 그런 다음 작업자 프로세스가 제어된 방식으로 대기열에서 작업을 가져오도록 할 수 있습니다. 새 큐 소비자를 쉽게 추가하여 애플리케이션이 확장됨에 따라 백엔드 작업 처리를 확장할 수 있습니다.
- Node.js 이벤트 루프를 차단할 수 있는 모놀리식 작업을 분리합니다. 예를 들어 사용자 요청에 오디오 트랜스 코딩과 같은 CPU 집약적 인 작업이 필요한 경우 이 작업을 다른 프로세스에 위임하여 사용자가 직면하는 프로세스가 응답을 유지하도록 할 수 있습니다.
- 다양한 서비스에서 안정적인 커뮤니케이션 채널을 제공합니다. 예를 들어, 한 프로세스 또는 서비스에서 타스크(작업 job)을 대기열에 넣고 다른 프로세스에서 사용할 수 있습니다. 프로세스 또는 서비스에서 작업 라이프사이클의 완료, 오류 또는 기타 상태 변경시 알림(상태 이벤트 수신)을 받을 수 있습니다. 대기열 생산자 또는 소비자가 실패하면 해당 상태가 유지되고 노드가 다시 시작될 때 작업 처리가 자동으로 다시 시작될 수 있습니다.

Nest는 인기있고 잘 지원되는 고성능 Node.js 기반 대기열 시스템 구현인 [Bull](https://github.com/OptimalBits/bull) 위에 추상화/래퍼로 `@nestjs/bull` 패키지를 제공합니다. . 이 패키지를 사용하면 Bull Queues를 Nest 친화적인 방식으로 애플리케이션에 쉽게 통합할 수 있습니다.

Bull은 [Redis](https://redis.io/)를 사용하여 작업 데이터를 유지하므로 시스템에 Redis를 설치해야합니다. Redis가 지원되기 때문에 대기열 아키텍처는 완전히 분산되고 플랫폼에 독립적일 수 있습니다. 예를 들어 일부 대기열 <a href="techniques/queues#producers">제작자</a>와 <a href="techniques/queues#consumers">소비자</a> 및 <a href="techniques/queues#event-listeners">리스너</a>는 하나(또는 여러) 노드의 Nest에서 실행되고 다른 생산자, 소비자 및 다른 네트워크 노드의 다른 Node.js 플랫폼에서 실행되는 리스너입니다.

이 장에서는 `@nestjs/bull` 패키지를 다룹니다. 또한 자세한 배경 및 구체적인 구현 세부 정보를 보려면 [Bull 문서](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md)를 읽어보는 것이 좋습니다.

#### Installation

사용을 시작하려면 먼저 필요한 종속성을 설치합니다.

```bash
$ npm install --save @nestjs/bull bull
$ npm install --save-dev @types/bull
```

설치 프로세스가 완료되면 `BullModule`을 루트 `AppModule`로 가져올 수 있습니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { BullModule } from '@nestjs/bull';

@Module({
  imports: [
    BullModule.forRoot({
      redis: {
        host: 'localhost',
        port: 6379,
      },
    }),
  ],
})
export class AppModule {}
```

`forRoot()` 메소드는 애플리케이션에 등록된 모든 큐에서 사용할 `bull` 패키지 구성 객체를 등록하는 데 사용됩니다 (달리 지정하지 않는 한). 구성 객체는 다음 속성으로 구성됩니다.

- `limiter: RateLimiter` - 대기열의 작업이 처리되는 속도를 제어하는 옵션입니다. 자세한 내용은 [RateLimiter](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md#queue)를 참조하세요. 선택사항(Optional).
- `redis: RedisOpts` - Redis 연결을 구성하는 옵션입니다. 자세한 내용은 [RedisOpts](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md#queue)를 참조하세요. 선택사항.
- `prefix: string` - 모든 대기열 키의 접두사입니다. 선택사항.
- `defaultJobOptions: JobOpts` - 새 작업의 기본 설정을 제어하는 옵션입니다. 자세한 내용은 [JobOpts](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md#queueadd)를 참조하세요. 선택사항.
- `settings: AdvancedSettings` - 고급 대기열 구성 설정. 일반적으로 변경해서는 안됩니다. 자세한 내용은 [AdvancedSettings](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md#queue)를 참조하세요. 선택사항.

모든 옵션은 선택사항이며 대기열 동작을 세부적으로 제어할 수 있습니다. 이들은 Bull `Queue` 생성자에 직접 전달됩니다. 이러한 옵션에 대한 자세한 내용은 [여기](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md#queue)를 참조하세요.

대기열을 등록하려면 다음과 같이 `BullModule#registerQueue()` 동적모듈을 가져옵니다.

```typescript
BullModule.registerQueue({
  name: 'audio',
});
```

> info **힌트** 쉼표로 구분된 여러 구성 객체를 `registerQueue()` 메서드에 전달하여 여러 대기열을 만듭니다.

`registerQueue()` 메서드는 대기열을 인스턴스화 및/또는 등록하는데 사용됩니다. 대기열은 동일한 자격증명으로 동일한 기본 Redis 데이터베이스에 연결되는 모듈 및 프로세스간에 공유됩니다. 각 대기열은 이름 속성으로 고유합니다. 대기열 이름은 주입 토큰(컨트롤러/프로바이더에 대기열을 주입하기 위한)으로 사용되며 데코레이터에 대한 인수로 사용되어 소비자 클래스 및 리스너를 대기열과 연결합니다.

다음과 같이 특정 대기열에 대해 미리 구성된 일부 옵션을 재정의할 수도 있습니다.

```typescript
BullModule.registerQueue({
  name: 'audio',
  redis: {
    port: 6380,
  },
});
```

작업은 Redis에서 유지되기 때문에 특정 명명된 대기열이 인스턴스화 될 때마다(예: 앱이 시작/다시 시작될 때) 완료되지 않은 이전 세션에서 존재할 수 있는 모든 이전 작업을 처리하려고 시도합니다.

각 대기열에는 하나 이상의 생산자, 소비자 및 리스너가 있을 수 있습니다. 소비자는 FIFO(기본값), LIFO 또는 우선 순위에 따라 특정 순서로 대기열에서 작업을 검색합니다. 대기열 처리 순서 제어는 <a href="techniques/queues#consumers">여기</a>에서 설명합니다.

<app-banner-enterprise></app-banner-enterprise>

#### Named configurations

대기열이 여러 다른 Redis 인스턴스에 연결되는 경우 **명명된 구성**이라는 기술을 사용할 수 있습니다. 이 기능을 사용하면 지정된 키 아래에 여러 구성을 등록할 수 있으며 대기열 옵션에서 참조할 수 있습니다.

예를 들어 애플리케이션에 등록된 몇개의 대기열에서 사용하는 추가 Redis 인스턴스(기본 인스턴스와는 별개)가 있다고 가정하면 다음과 같이 구성을 등록할 수 있습니다.

```typescript
BullModule.forRoot('alternative-config', {
  redis: {
    port: 6381,
  },
});
```

위의 예에서 `'alternative-config'`는 구성키일 뿐입니다(임의의 문자열일 수 있음).

이제 `registerQueue()` 옵션 객체에서 이 구성을 가리킬 수 있습니다.

```typescript
BullModule.registerQueue({
  configKey: 'alternative-queue'
  name: 'video',
});
```

#### Producers

작업 생산자는 작업을 대기열에 추가합니다. 생산자는 일반적으로 애플리케이션 서비스(Nest [프로바이더](/providers))입니다. 대기열에 작업을 추가하려면 먼저 다음과 같이 대기열을 서비스에 삽입합니다.

```typescript
import { Injectable } from '@nestjs/common';
import { Queue } from 'bull';
import { InjectQueue } from '@nestjs/bull';

@Injectable()
export class AudioService {
  constructor(@InjectQueue('audio') private audioQueue: Queue) {}
}
```

> info **힌트** `@InjectQueue()` 데코레이터는 `registerQueue()` 메서드 호출(예: `'audio'`)에 제공된 이름으로 대기열을 식별합니다.

이제 대기열의 `add()` 메서드를 호출하고 사용자 정의 작업 객체를 전달하여 작업을 추가합니다. 작업은 직렬화 가능한 JavaScript 객체로 표시됩니다(이것이 Redis 데이터베이스에 저장되는 방식이므로). 전달하는 직업의 형태는 임의적입니다. 이를 사용하여 작업 객체의 의미를 나타냅니다.

```typescript
const job = await this.audioQueue.add({
  foo: 'bar',
});
```

#### Named jobs

작업에는 고유한 이름이 있을 수 있습니다. 이렇게하면 지정된 이름의 작업만 처리하는 특수 <a href="techniques/queues#consumers">소비자</a>를 만들 수 있습니다.

```typescript
const job = await this.audioQueue.add('transcode', {
  foo: 'bar',
});
```

> warning **경고** 명명된 작업을 사용하는 경우 대기열에 추가된 각 고유 이름에 대해 프로세서를 만들어야합니다. 그렇지 않으면 대기열에서 주어진 작업에 대한 프로세서가 없다고 불평합니다. 명명된 작업 사용에 대한 자세한 내용은 <a href="techniques/queues#consumers">여기</a>를 참조하세요.

#### Job options

Jobs can have additional options associated with them. Pass an options object after the `job` argument in the `Queue.add()` method. Job options properties are:
작업에는 관련된 추가 옵션이 있을 수 있습니다. `Queue.add()` 메서드에서 `job` 인수 뒤에 옵션 객체를 전달합니다. 작업 옵션 속성은 다음과 같습니다.

- `priority`: `number` - 선택적 우선 순위 값입니다. 범위는 1(가장 높은 우선 순위)에서 MAX_INT(가장 낮은 우선 순위)까지 입니다. 우선 순위를 사용하면 성능에 약간의 영향을 미치므로 주의해서 사용하십시오.
- `delay`: `number` - 이 작업을 처리할 수 있을 때까지 대기하는 시간(밀리 초)입니다. 정확한 지연을 위해 서버와 클라이언트 모두 시계를 동기화해야 합니다.
- `attempts`: `number` - 작업이 완료될 때까지 시도한 총 시도 횟수입니다.
- `repeat`: `RepeatOpts` - 크론 사양에 따라 작업을 반복합니다. [RepeatOpts](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md#queueadd)를 참조하세요.
- `backoff`: `number | BackoffOpts` - 작업이 실패할 경우 자동 재시도에 대한 백오프 설정입니다. [BackoffOpts](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md#queueadd)를 참조하세요.
- `lifo`: `boolean` - true인 경우 작업을 왼쪽 대신 대기열의 오른쪽 끝에 추가합니다(기본값 false).
- `timeout`: `number` - 작업이 시간 초과 오류와 함께 실패해야하는 시간(밀리 초)입니다.
- `jobId`: `number` | `string` - 작업 ID 재정의 - 기본적으로 작업 ID는 고유합니다.
  정수이지만 이 설정을 사용하여 재정의할 수 있습니다. 이 옵션을 사용하는 경우 jobId가 고유한지 확인하는 것은 사용자에게 달려 있습니다. 이미 존재하는 ID로 작업을 추가하려고하면 추가되지 않습니다.
- `removeOnComplete`: `boolean | number` - true이면 성공적으로 완료되면 작업을 제거합니다. 숫자는 유지할 작업의 양을 지정합니다. 기본 동작은 작업을 완료된 세트로 유지하는 것입니다.
- `removeOnFail`: `boolean | number` - true이면 모든 시도후 실패하면 작업을 제거합니다. 숫자는 유지할 작업의 양을 지정합니다. 기본 동작은 실패한 세트에 작업을 유지하는 것입니다.
- `stackTraceLimit`: `number` - 스택 트레이스에 기록될 스택 트레이스 라인의 양을 제한합니다.

다음은 작업 옵션으로 작업을 사용자 정의하는 몇가지 예입니다.

작업 시작을 지연하려면 `delay` 구성 속성을 사용하세요.

```typescript
const job = await this.audioQueue.add(
  {
    foo: 'bar',
  },
  { delay: 3000 }, // 3 seconds delayed
);
```

대기열의 오른쪽 끝에 작업을 추가하려면 (작업을 **LIFO**(Last In First Out)로 처리) 구성 개체의 `lifo` 속성을 `true`로 설정합니다.

```typescript
const job = await this.audioQueue.add(
  {
    foo: 'bar',
  },
  { lifo: true },
);
```

작업의 우선 순위를 지정하려면 `priority` 속성을 사용하세요.

```typescript
const job = await this.audioQueue.add(
  {
    foo: 'bar',
  },
  { priority: 2 },
);
```

#### Consumers

소비자는 대기열에 추가된 작업을 처리하거나 대기열에서 이벤트를 수신하거나 둘다를 수신하는 메서드를 정의하는 **클래스**입니다. 다음과 같이 `@Processor()` 데코레이터를 사용하여 소비자 클래스를 선언합니다.

```typescript
import { Processor } from '@nestjs/bull';

@Processor('audio')
export class AudioConsumer {}
```

> info **힌트** 소비자는 `@nestjs/bull` 패키지가 그들을 픽업할 수 있도록 `providers`로 등록되어야 합니다.

데코레이터의 문자열 인수(예: `'audio'`)는 클래스 메서드와 연결할 대기열의 이름입니다.

소비자 클래스 내에서 `@Process()` 데코레이터로 핸들러 메서드를 데코레이션하여 작업 핸들러를 선언합니다.

```typescript
import { Processor, Process } from '@nestjs/bull';
import { Job } from 'bull';

@Processor('audio')
export class AudioConsumer {
  @Process()
  async transcode(job: Job<unknown>) {
    let progress = 0;
    for (i = 0; i < 100; i++) {
      await doSomething(job.data);
      progress += 10;
      job.progress(progress);
    }
    return {};
  }
}
```

데코레이팅된 메서드(예: `transcode()`)는 작업자가 유휴(idle) 상태이고 대기열에 처리할 작업이 있을 때마다 호출됩니다. 이 핸들러 메서드는 `job` 객체를 유일한 인수로 받습니다. 핸들러 메소드에서 반환된 값은 작업 객체에 저장되며 나중에 예를 들어 완료된 이벤트에 대한 리스너에서 액세스할 수 있습니다.

`Job` 객체에는 상태(state)와 상호작용할 수 있는 여러 메서드가 있습니다. 예를 들어 위의 코드는 `progress()` 메서드를 사용하여 작업의 진행상황을 업데이트합니다. 전체 `Job` 객체 API 참조는 [여기](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md#job)를 참조하세요.

아래에 표시된대로 `name`을 `@Process()` 데코레이터에 전달하여 작업 핸들러 메소드가 특정 타입(특정 `name`을 가진 작업)의 **작업만** 처리하도록 지정할 수 있습니다. 주어진 소비자 클래스에 각 작업 타입(`name`)에 해당하는 여러 `@Process()` 핸들러를 가질 수 있습니다. 명명된 작업을 사용하는 경우 각 이름에 해당하는 핸들러가 있어야합니다.

```typescript
@Process('transcode')
async transcode(job: Job<unknown>) { ... }
```

#### Request-scoped consumers

소비자가 요청 범위로 플래그가 지정되면(인젝션 범위는 [여기](/fundamentals/injection-scopes#provider-scope)에 대해 자세히 알아보기) 각 작업에 대해 독점적으로 클래스의 새 인스턴스가 생성됩니다. 작업이 완료된 후 인스턴스가 가비지 수집됩니다.

```typescript
@Processor({
  name: 'audio',
  scope: Scope.REQUEST,
})
```

요청 범위의 소비자 클래스는 동적으로 인스턴스화되고 단일 작업으로 범위가 지정되므로 표준 접근 방식을 사용하여 생성자를 통해 `JOB_REF`를 삽입할 수 있습니다.

```typescript
constructor(@Inject(JOB_REF) jobRef: Job) {
  console.log(jobRef);
}
```

> info **힌트** `JOB_REF` 토큰은 `@nestjs/bull` 패키지에서 가져옵니다.

#### Event listeners

Bull은 대기열 및/또는 작업 상태가 변경될 때 유용한 이벤트 세트를 생성합니다. Nest는 핵심 표준 이벤트 세트를 구독할 수 있는 데코레이터 세트를 제공합니다. 이들은 `@nestjs/bull` 패키지에서 내 보냅니다.

이벤트 리스너는 <a href="techniques/queues#consumers">소비자</a> 클래스내 (즉, `@Processor()` 데코레이터로 장식된 클래스내)에서 선언되어야 합니다. 이벤트를 수신하려면 아래 표에 있는 데코레이터중 하나를 사용하여 이벤트 핸들러를 선언하십시오. 예를 들어 작업이 `audio` 대기열에서 활성상태가 될 때 발생하는 이벤트를 수신하려면 다음 구문을 사용합니다.

```typescript
import { Processor, Process } from '@nestjs/bull';
import { Job } from 'bull';

@Processor('audio')
export class AudioConsumer {

  @OnQueueActive()
  onActive(job: Job) {
    console.log(
      `Processing job ${job.id} of type ${job.name} with data ${job.data}...`,
    );
  }
  ...
```

Bull은 분산(다중 노드) 환경에서 작동하므로 이벤트 지역성 개념을 정의합니다. 이 개념은 이벤트가 전적으로 단일 프로세스 내에서 또는 다른 프로세스의 공유 대기열에서 트리거될 수 있음을 인식합니다. **로컬** 이벤트는 작업 또는 상태 변경이 로컬 프로세스의 대기열에서 트리거될 때 생성되는 이벤트입니다. 즉, 이벤트 생산자와 소비자가 단일 프로세스에 로컬인 경우 큐에서 발생하는 모든 이벤트는 로컬입니다.

대기열이 여러 프로세스에서 공유되면 **전역** 이벤트가 발생할 가능성이 있습니다. 한 프로세스의 리스너가 다른 프로세스에 의해 트리거된 이벤트 알림을 받으려면 전역 이벤트에 등록해야 합니다.

이벤트 핸들러는 해당 이벤트가 발생할 때마다 호출됩니다. 핸들러는 아래 표에 표시된 서명으로 호출되어 이벤트와 관련된 정보에 대한 액세스를 제공합니다. 아래에서 로컬 및 글로벌 이벤트 핸들러 서명의 주요 차이점에 대해 설명합니다.

<table>
  <tr>
    <th>Local event listeners</th>
    <th>Global event listeners</th>
    <th>Handler method signature / When fired</th>
  </tr>
  <tr>
    <td><code>@OnQueueError()</code></td><td><code>@OnGlobalQueueError()</code></td><td><code>handler(error: Error)</code> - 오류가 발생했습니다. <code>error</code>에는 트리거링 오류가 있습니다.</td>
  </tr>
  <tr>
    <td><code>@OnQueueWaiting()</code></td><td><code>@OnGlobalQueueWaiting()</code></td><td><code>handler(jobId: number | string)</code> - 작업자가 유휴 상태가 되는 즉시 작업이 처리되기를 기다리고 있습니다. <code>jobId</code>에는 이 상태에 들어간 작업의 ID가 포함됩니다.</td>
  </tr>
  <tr>
    <td><code>@OnQueueActive()</code></td><td><code>@OnGlobalQueueActive()</code></td><td><code>handler(job: Job)</code> - <code>job</code>작업이 시작되었습니다.</td>
  </tr>
  <tr>
    <td><code>@OnQueueStalled()</code></td><td><code>@OnGlobalQueueStalled()</code></td><td><code>handler(job: Job)</code> - <code>job</code>작업이 중단된 것으로 표시되었습니다. 이는 이벤트 루프를 중단하거나 일시 중지하는 작업 작업자를 디버깅하는 데 유용합니다.</td>
  </tr>
  <tr>
    <td><code>@OnQueueProgress()</code></td><td><code>@OnGlobalQueueProgress()</code></td><td><code>handler(job: Job, progress: number)</code> - 작업 <code>job</code>의 진행률이 <code>progress</code> 값으로 업데이트되었습니다.</td>
  </tr>
  <tr>
    <td><code>@OnQueueCompleted()</code></td><td><code>@OnGlobalQueueCompleted()</code></td><td><code>handler(job: Job, result: any)</code> 작업 <code>job</code>이 성공적으로 완료되었으며 결과는 <code>result</code>입니다.</td>
  </tr>
  <tr>
    <td><code>@OnQueueFailed()</code></td><td><code>@OnGlobalQueueFailed()</code></td><td><code>handler(job: Job, err: Error)</code> 작업 <code>job</code>이 <code>err</code> 이유로 실패했습니다.</td>
  </tr>
  <tr>
    <td><code>@OnQueuePaused()</code></td><td><code>@OnGlobalQueuePaused()</code></td><td><code>handler()</code> 대기열이 일시 중지되었습니다.</td>
  </tr>
  <tr>
    <td><code>@OnQueueResumed()</code></td><td><code>@OnGlobalQueueResumed()</code></td><td><code>handler(job: Job)</code> 대기열이 재개되었습니다.</td>
  </tr>
  <tr>
    <td><code>@OnQueueCleaned()</code></td><td><code>@OnGlobalQueueCleaned()</code></td><td><code>handler(jobs: Job[], type: string)</code> 대기열에서 오래된 작업이 정리되었습니다. <code>jobs</code>는 정리된 작업의 배열이고 <code>type</code>은 정리된 작업의 유형입니다.</td>
  </tr>
  <tr>
    <td><code>@OnQueueDrained()</code></td><td><code>@OnGlobalQueueDrained()</code></td><td><code>handler()</code> 대기열이 대기중인 모든 작업을 처리할 때마다 발생합니다(아직 처리되지 않은 지연된 작업이 있을 수 있는 경우에도).</td>
  </tr>
  <tr>
    <td><code>@OnQueueRemoved()</code></td><td><code>@OnGlobalQueueRemoved()</code></td><td><code>handler(job: Job)</code> Job <code>job</code> 작업이 성공적으로 제거되었습니다.</td>
  </tr>
</table>

전역 이벤트를 수신할 때 메서드 서명은 로컬 서명과 약간 다를 수 있습니다. 특히, 로컬 버전에서 `job` 객체를 수신하는 모든 메서드 서명은 대신 전역 버전에서 `jobId`(`number`)를 받습니다. 이 경우 실제 `job` 객체에 대한 참조를 얻으려면 `Queue#getJob` 메소드를 사용하십시오. 이 호출은 대기해야 하므로 핸들러는 `async`로 선언되어야합니다. 예를 들면:

```typescript
@OnGlobalQueueCompleted()
async onGlobalCompleted(jobId: number, result: any) {
  const job = await this.immediateQueue.getJob(jobId);
  console.log('(Global) on completed: job ', job.id, ' -> result: ', result);
}
```

> info **힌트** `Queue` 객체에 액세스하려면 (`getJob()` 호출을 수행하기 위해) 물론 주입해야 합니다. 또한 큐를 삽입하는 모듈에 큐를 등록해야 합니다.

특정 이벤트 리스너 데코레이터 외에도 일반 `@OnQueueEvent()` 데코레이터를 `BullQueueEvents` 또는 `BullQueueGlobalEvents` 열거형과 함께 사용할 수 도 있습니다. 이벤트에 대한 자세한 내용은 [여기](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md#events)를 참조하세요.

#### Queue management

대기열에는 일시 중지 및 재개, 다양한 상태의 작업수 검색 등과 같은 관리 기능을 수행할 수 있는 API가 있습니다. 전체 대기열 API는 [여기](https://github.com/OptimalBits/bull/blob/master/REFERENCE.md#queue)에서 찾을 수 있습니다. 일시 중지/재개 예제와 함께 아래에 표시된대로 `Queue` 객체에서 직접 이러한 메서드를 호출합니다.

`pause()` 메서드 호출로 대기열을 일시 중지합니다. 일시 중지된 대기열은 재개될 때까지 새 작업을 처리하지 않지만 현재 처리중인 작업은 완료될 때까지 계속됩니다.

```typescript
await audioQueue.pause();
```

일시 중지된 대기열을 재개하려면 다음과 같이 `resume()` 메서드를 사용하세요.

```typescript
await audioQueue.resume();
```

#### Separate processes

작업 핸들러는 별도의(포크된) 프로세스 ([source](https://github.com/OptimalBits/bull#separate-processes))에서도 실행할 수 있습니다. 여기에는 몇가지 장점이 있습니다.

- 프로세스는 샌드박스로 처리되므로 충돌이 발생해도 작업자에게 영향을 주지 않습니다.
- 대기열에 영향을 주지 않고 차단 코드를 실행할 수 있습니다(작업이 중단되지 않음).
- 멀티 코어 CPU의 훨씬 더 나은 활용.
- redis에 대한 연결이 적습니다.

```ts
@@filename(app.module)
import { Module } from '@nestjs/common';
import { BullModule } from '@nestjs/bull';
import { join } from 'path';

@Module({
  imports: [
    BullModule.registerQueue({
      name: 'audio',
      processors: [join(__dirname, 'processor.js')],
    }),
  ],
})
export class AppModule {}
```

함수가 분기된 프로세스에서 실행되기 때문에 종속성 주입(및 IoC 컨테이너)을 사용할 수 없습니다. 즉, 프로세서 함수는 필요한 모든 외부 종속성 인스턴스를 포함(또는 생성)해야 합니다.

```ts
@@filename(processor)
import { Job, DoneCallback } from 'bull';

export default function (job: Job, cb: DoneCallback) {
  console.log(`[${process.pid}] ${JSON.stringify(job.data)}`);
  cb(null, 'It works');
}
```

#### Async configuration

정적으로 대신 비동기적으로 `bull` 옵션을 전달할 수 있습니다. 이 경우 비동기 구성을 처리하는 여러 방법을 제공하는 `forRootAsync()` 메소드를 사용하십시오. 마찬가지로 대기열 옵션을 비동기적으로 전달하려면 `registerQueueAsync()` 메서드를 사용합니다.

한가지 접근 방식은 팩토리 함수를 사용하는 것입니다.

```typescript
BullModule.forRootAsync({
  useFactory: () => ({
    redis: {
      host: 'localhost',
      port: 6379,
    },
  }),
});
```

우리 팩토리는 다른 [비동기 프로바이더](/fundamentals/async-providers)처럼 작동합니다 (예: `async`일 수 있으며 `inject`를 통해 종속성을 삽입할 수 있음).

```typescript
BullModule.forRootAsync({
  imports: [ConfigModule],
  useFactory: async (configService: ConfigService) => ({
    redis: {
      host: configService.get('QUEUE_HOST'),
      port: +configService.get('QUEUE_PORT'),
    },
  }),
  inject: [ConfigService],
});
```

또는 `useClass` 구문을 사용할 수 있습니다.

```typescript
BullModule.forRootAsync({
  useClass: BullConfigService,
});
```

위의 구성은 `BullModule` 내에서 `BullConfigService`를 인스턴스화하고 `createSharedConfiguration()`을 호출하여 옵션 객체를 제공하는데 사용합니다. 이는 `BullConfigService`가 아래와 같이 `SharedBullConfigurationFactory` 인터페이스를 구현해야 함을 의미합니다.

```typescript
@Injectable()
class BullConfigService implements SharedBullConfigurationFactory {
  createSharedConfiguration(): SharedBullConfigurationFactory {
    return {
      redis: {
        host: 'localhost',
        port: 6379,
      },
    };
  }
}
```

`BullModule` 내에서 `BullConfigService` 생성을 방지하고 다른 모듈에서 가져온 프로바이더를 사용하려면 `useExisting` 구문을 사용할 수 있습니다.

```typescript
BullModule.forRootAsync({
  imports: [ConfigModule],
  useExisting: ConfigService,
});
```

이 구조는 `useClass`와 동일하게 작동하지만 한가지 중요한 차이점이 있습니다. `BullModule`은 새 모듈을 인스턴스화하는 대신 기존 `ConfigService`를 재사용하기 위해 가져온 모듈을 조회합니다.

#### Example

작동하는 예는 [여기](https://github.com/nestjs/nest/tree/master/sample/26-queues)에서 확인할 수 있습니다.
