### Task Scheduling

작업 예약을 사용하면 고정된 날짜/시간, 반복 간격 또는 지정된 간격후 한 번 실행되도록 임의 코드(방법/기능)를 예약할 수 있습니다. Linux 세계에서는 OS 수준에서 [cron](https://en.wikipedia.org/wiki/Cron)과 같은 패키지에서 처리하는 경우가 많습니다. Node.js 앱의 경우 cron과 유사한 기능을 에뮬레이트하는 여러 패키지가 있습니다. Nest는 인기있는 Node.js [node-cron](https://github.com/kelektiv/node-cron) 패키지와 통합되는 `@nestjs/schedule` 패키지를 제공합니다. 이 패키지는 현재 장에서 다룰 것입니다.

#### Installation

사용을 시작하려면 먼저 필요한 종속성을 설치합니다.

```bash
$ npm install --save @nestjs/schedule
$ npm install --save-dev @types/cron
```

작업 스케줄링을 활성화하려면 `ScheduleModule`을 루트 `AppModule`로 가져오고 아래와 같이 `forRoot()` 정적 메서드를 실행합니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { ScheduleModule } from '@nestjs/schedule';

@Module({
  imports: [
    ScheduleModule.forRoot()
  ],
})
export class AppModule {}
```

`.forRoot()` 호출은 스케줄러를 초기화하고 앱 내에 존재하는 선언적 [크론 작업](/techniques/task-scheduling#declarative-cron-jobs), [시간 초과](/techniques/task-scheduling#declarative-timeouts) 및 [간격](/techniques/task-scheduling#declarative-intervals)을 등록합니다. 등록은 `onApplicationBootstrap` 라이프 사이클 후크가 발생할 때 발생하여 모든 모듈이 예약된 작업을 로드하고 선언했는지 확인합니다.

#### Declarative cron jobs

크론 작업은 자동으로 실행되도록 임의의 함수(메소드 호출)를 예약합니다. Cron 작업은 다음을 실행할 수 있습니다.

- 지정된 날짜/시간에 한번.
- 반복적으로; 반복 작업은 지정된 간격내에서 지정된 순간에 실행할 수 있습니다(예: 시간당 한번, 주당 한번, 5분에 한번).

다음과 같이 실행할 코드를 포함하는 메서드 정의 앞에 `@Cron()` 데코레이터를 사용하여 크론 작업을 선언합니다.

```typescript
import { Injectable, Logger } from '@nestjs/common';
import { Cron } from '@nestjs/schedule';

@Injectable()
export class TasksService {
  private readonly logger = new Logger(TasksService.name);

  @Cron('45 * * * * *')
  handleCron() {
    this.logger.debug('Called when the current second is 45');
  }
}
```

이 예에서 `handleCron()`메서드는 현재 초가 `45`일 때마다 호출됩니다. 즉, 이 메서드는 45초일 때 매 1 분마다 한번 실행됩니다.

`@Cron()` 데코레이터는 모든 표준 [cron 패턴](http://crontab.org/)을 지원합니다.

- 별표 (예: `*`)
- 범위 (예: `1-3,5`)
- 단계 (예: `*/2`)

위의 예에서 우리는 데코레이터에 `45 * * * * *`를 전달했습니다. 다음 키는 cron 패턴 문자열의 각 위치가 해석되는 방법을 보여줍니다.

<pre class="language-javascript"><code class="language-javascript">
* * * * * *
| | | | | |
| | | | | day of week
| | | | month
| | | day of month
| | hour
| minute
second (optional)
</code></pre>

몇가지 샘플 크론 패턴은 다음과 같습니다.

<table>
  <tbody>
    <tr>
      <td><code>* * * * * *</code></td>
      <td>매 초</td>
    </tr>
    <tr>
      <td><code>45 * * * * *</code></td>
      <td>1분마다 45초에</td>
    </tr>
    <tr>
      <td><code>0 10 * * * *</code></td>
      <td>매시간, 10분이 시작될 때</td>
    </tr>
    <tr>
      <td><code>0 */30 9-17 * * *</code></td>
      <td>오전 9시부터 오후 5시까지 30분마다</td>
    </tr>
   <tr>
      <td><code>0 30 11 * * 1-5</code></td>
      <td>월요일 ~ 금요일 오전 11시 30 분</td>
    </tr>
  </tbody>
</table>

`@nestjs/schedule` 패키지는 일반적으로 사용되는 크론 패턴과 함께 편리한 열거형을 제공합니다. 이 열거형을 다음과 같이 사용할 수 있습니다.

```typescript
import { Injectable, Logger } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';

@Injectable()
export class TasksService {
  private readonly logger = new Logger(TasksService.name);

  @Cron(CronExpression.EVERY_45_SECONDS)
  handleCron() {
    this.logger.debug('Called every 45 seconds');
  }
}
```

이 예에서 `handleCron()` 메서드는 `45`초마다 호출됩니다.

또는 JavaScript `Date` 객체를 `@Cron()` 데코레이터에 제공할 수 있습니다. 이렇게하면 작업이 지정된 날짜에 정확히 한 번 실행됩니다.

> info **힌트** JavaScript 날짜 산술을 사용하여 현재 날짜를 기준으로 작업을 예약합니다. 예를 들어 `@Cron(new Date(Date.now() + 10 * 1000))`은 앱이 시작된 후 10초 후에 실행되도록 작업을 예약합니다.

또한 `@Cron()` 데코레이터에 두번째 매개변수로 추가옵션을 제공할 수 있습니다.

<table>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td>
        cron 작업이 선언된 후 액세스하고 제어하는데 유용합니다.
      </td>
    </tr>
    <tr>
      <td><code>timeZone</code></td>
      <td>
        실행 시간대를 지정합니다. 이것은 시간대에 상대적인 실제 시간을 수정합니다. 시간대가 유효하지 않으면 오류가 발생합니다. <a href="http://momentjs.com/timezone/">Moment Timezone</a> 웹사이트에서 사용 가능한 모든 시간대를 확인할 수 있습니다.
      </td>
    </tr>
    <tr>
      <td><code>utcOffset</code></td>
      <td>
        이렇게 하면 <code>timeZone</code> 매개변수를 사용하는 대신 시간대의 오프셋을 지정할 수 있습니다.
      </td>
    </tr>
  </tbody>
</table>

```typescript
import { Injectable } from '@nestjs/common';
import { Cron, CronExpression } from '@nestjs/schedule';

@Injectable()
export class NotificationService {
  @Cron('* * 0 * * *', {
    name: 'notifications',
    timeZone: 'Europe/Paris',
  })
  triggerNotifications() {}
}
```

크론 작업이 선언된 후 액세스 및 제어하거나 [동적 API](/techniques/task-scheduling#dynamic-schedule-module-api)를 사용하여 크론 작업(크론 패턴이 런타임에 정의됨)을 동적으로 생성할 수 있습니다. API를 통해 선언적 크론 작업에 액세스하려면 데코레이터의 두번째 인수로 선택적 옵션 개체의 `name` 속성을 전달하여 작업을 이름과 연결해야합니다.

#### Declarative intervals

메서드가(반복) 지정된 간격으로 실행되어야 한다고 선언하려면 메서드 정의 앞에 `@Interval()` 데코레이터를 붙입니다. 간격 값을 밀리 초 단위의 숫자로 아래와 같이 데코레이터에 전달합니다.

```typescript
@Interval(10000)
handleInterval() {
  this.logger.debug('Called every 10 seconds');
}
```

> info **힌트** 이 메커니즘은 내부적으로 자바 스크립트 `setInterval()` 함수를 사용합니다. 크론 작업을 사용하여 반복 작업을 예약할 수도 있습니다.

[동적 API](/techniques/task-scheduling#dynamic-schedule-module-api)를 통해 선언 클래스 외부에서 선언적 간격을 제어하려면 다음 구성을 사용하여 간격을 이름과 연결합니다.

```typescript
@Interval('notifications', 2500)
handleInterval() {}
```

또한 [동적 API](/content/application-context.mdtechniques/task-scheduling#dynamic-intervals)를 사용하면 런타임에 간격의 속성이 정의되는 동적 간격을 **만들고**, 이를 **나열 및 삭제**할 수 있습니다.

<app-banner-enterprise></app-banner-enterprise>

#### Declarative timeouts

메서드가 지정된 시간 제한에(한 번) 실행되어야 한다고 선언하려면 메서드 정의 앞에 `@Timeout()` 데코레이터를 붙입니다. 아래와 같이 애플리케이션 시작에서 데코레이터로 상대 시간 오프셋(밀리 초)을 전달합니다.

```typescript
@Timeout(5000)
handleTimeout() {
  this.logger.debug('Called once after 5 seconds');
}
```

> info **힌트** 이 메커니즘은 내부적으로 자바 스크립트 `setTimeout()` 함수를 사용합니다.

[동적 API](/techniques/task-scheduling#dynamic-schedule-module-api)를 통해 선언 클래스 외부에서 선언적 제한 시간을 제어하려면 다음 구성을 사용하여 제한 시간을 이름과 연결하십시오.

```typescript
@Timeout('notifications', 2500)
handleTimeout() {}
```

또한 [동적 API](/techniques/task-scheduling#dynamic-timeouts)를 사용하면 런타임에 타임아웃 속성이 정의되는 동적 타임아웃을 **만들고**,이를 **나열 및 삭제**할 수 있습니다.

#### Dynamic schedule module API

`@nestjs/schedule` 모듈은 선언적 [크론 작업](/techniques/task-scheduling#declarative-cron-jobs), [시간 초과](/techniques/task-scheduling#declarative-timeouts) 및 [간격](/techniques/task-scheduling#declarative-intervals)을 관리할 수 있는 동적 API를 제공합니다. 또한 API를 사용하면 **동적** 크론 작업, 시간 초과 및 간격을 만들고 관리할 수 있습니다. 여기서 속성은 런타임에 정의됩니다.

#### Dynamic cron jobs

`SchedulerRegistry` API를 사용하여 코드의 어느 곳에서나 이름으로 `CronJob` 인스턴스에 대한 참조를 가져옵니다. 먼저 표준 생성자 주입을 사용하여 `SchedulerRegistry`를 주입합니다.

```typescript
constructor(private schedulerRegistry: SchedulerRegistry) {}
```

> info **힌트** `@nestjs/schedule` 패키지에서 `SchedulerRegistry`를 가져옵니다.

그런 다음 다음과 같이 클래스에서 사용하십시오. 다음 선언으로 크론 작업이 생성되었다고 가정합니다.

```typescript
@Cron('* * 8 * * *', {
  name: 'notifications',
})
triggerNotifications() {}
```

다음을 사용하여 이 작업에 액세스하십시오.

```typescript
const job = this.schedulerRegistry.getCronJob('notifications');

job.stop();
console.log(job.lastDate());
```

`getCronJob()` 메소드는 명명된 크론 작업을 반환합니다. 반환된 `CronJob` 객체에는 다음 메서드가 있습니다.

- `stop()` - 실행이 예약된 작업을 중지합니다.
- `start()` - 중지된 작업을 다시 시작합니다.
- `setTime(time: CronTime)` - 작업을 중지하고 새 시간을 설정한 다음 시작
- `lastDate()`- 작업이 마지막으로 실행된 날짜의 문자열 표현을 반환합니다.
- `nextDates(count: number)` - 예정된 작업 실행 날짜를 나타내는 `moment` 객체의 배열(크기 `count`)을 반환합니다.

> info **힌트** `moment` 객체에 `toDate()`를 사용하여 사람이 읽을 수 있는 형식으로 렌더링합니다.

다음과 같이 `SchedulerRegistry.addCronJob()` 메소드를 사용하여 동적으로 새 크론 작업을 **생성**합니다.

```typescript
addCronJob(name: string, seconds: string) {
  const job = new CronJob(`${seconds} * * * * *`, () => {
    this.logger.warn(`time (${seconds}) for job ${name} to run!`);
  });

  this.schedulerRegistry.addCronJob(name, job);
  job.start();

  this.logger.warn(
    `job ${name} added for each minute at ${seconds} seconds!`,
  );
}
```

이 코드에서는 `cron` 패키지의 `CronJob` 객체를 사용하여 크론 작업을 생성합니다. `CronJob` 생성자는 첫번째 인수로 cron 패턴 (`@Cron()` [데코레이터](/techniques/task-scheduling#declarative-cron-jobs)와 유사)을 취합니다. cron 타이머가 두번째 인수로 실행될 때 실행되는 콜백입니다. `SchedulerRegistry.addCronJob()` 메소드는 `CronJob`의 이름과 `CronJob` 객체 자체의 두가지 인수를 사용합니다.

> warning **경고** 액세스하기 전에 `SchedulerRegistry`를 삽입해야합니다. `cron` 패키지에서 `CronJob`을 가져옵니다.

다음과 같이 `SchedulerRegistry.deleteCronJob()` 메소드를 사용하여 명명된 크론 작업을 **삭제**합니다.

```typescript
deleteCron(name: string) {
  this.schedulerRegistry.deleteCronJob(name);
  this.logger.warn(`job ${name} deleted!`);
}
```

다음과 같이 `SchedulerRegistry.getCronJobs()` 메소드를 사용하여 모든 크론 작업을 **나열**합니다.

```typescript
getCrons() {
  const jobs = this.schedulerRegistry.getCronJobs();
  jobs.forEach((value, key, map) => {
    let next;
    try {
      next = value.nextDates().toDate();
    } catch (e) {
      next = 'error: next fire date is in the past!';
    }
    this.logger.log(`job: ${key} -> next: ${next}`);
  });
}
```

`getCronJobs()` 메소드는 `map`을 반환합니다. 이 코드에서는 map을 반복하고 각 `CronJob`의 `nextDates()` 메소드에 액세스하려고 합니다. `CronJob` API에서 작업이 이미 실행되었고 향후 실행 날짜가 없는 경우 예외가 발생합니다.

#### Dynamic intervals

`SchedulerRegistry.getInterval()` 메서드를 사용하여 간격에 대한 참조를 가져옵니다. 위와 같이 표준 생성자 주입을 사용하여 `SchedulerRegistry`를 주입합니다.

```typescript
constructor(private schedulerRegistry: SchedulerRegistry) {}
```

다음과 같이 사용하십시오.

```typescript
const interval = this.schedulerRegistry.getInterval('notifications');
clearInterval(interval);
```

다음과 같이 `SchedulerRegistry.addInterval()` 메소드를 사용하여 동적으로 새 간격을 **생성**합니다.

```typescript
addInterval(name: string, milliseconds: number) {
  const callback = () => {
    this.logger.warn(`Interval ${name} executing at time (${milliseconds})!`);
  };

  const interval = setInterval(callback, milliseconds);
  this.schedulerRegistry.addInterval(name, interval);
}
```

이 코드에서 표준 자바 스크립트 간격(interval)을 만든 다음 `ScheduleRegistry.addInterval()` 메서드에 전달합니다.
이 메서드는 간격 이름과 간격 자체의 두 인수를 사용합니다.

다음과 같이 `SchedulerRegistry.deleteInterval()` 메소드를 사용하여 명명된 간격을 **삭제**합니다.

```typescript
deleteInterval(name: string) {
  this.schedulerRegistry.deleteInterval(name);
  this.logger.warn(`Interval ${name} deleted!`);
}
```

다음과 같이 `SchedulerRegistry.getIntervals()` 메소드를 사용하여 모든 간격을 **나열**합니다.

```typescript
getIntervals() {
  const intervals = this.schedulerRegistry.getIntervals();
  intervals.forEach(key => this.logger.log(`Interval: ${key}`));
}
```

#### Dynamic timeouts

`SchedulerRegistry.getTimeout()` 메소드를 사용하여 제한 시간에 대한 참조를 확보하십시오. 위와 같이 표준 생성자 주입을 사용하여 `SchedulerRegistry`를 주입합니다.

```typescript
constructor(private schedulerRegistry: SchedulerRegistry) {}
```

다음과 같이 사용하십시오.

```typescript
const timeout = this.schedulerRegistry.getTimeout('notifications');
clearTimeout(timeout);
```

다음과 같이 `SchedulerRegistry.addTimeout()` 메소드를 사용하여 동적으로 새 시간제한(timeout)을 **생성**합니다.

```typescript
addTimeout(name: string, milliseconds: number) {
  const callback = () => {
    this.logger.warn(`Timeout ${name} executing after (${milliseconds})!`);
  };

  const timeout = setTimeout(callback, milliseconds);
  this.schedulerRegistry.addTimeout(name, timeout);
}
```

이 코드에서 표준 JavaScript 시간제한을 생성한 다음 `ScheduleRegistry.addTimeout()` 메서드에 전달합니다.
이 메서드는 두개의 인수, 즉 시간제한 이름과 시간제한 자체를 사용합니다.

다음과 같이 `SchedulerRegistry.deleteTimeout()` 메서드를 사용하여 명명된 시간제한을 **삭제**합니다.

```typescript
deleteTimeout(name: string) {
  this.schedulerRegistry.deleteTimeout(name);
  this.logger.warn(`Timeout ${name} deleted!`);
}
```

**List** all timeouts using the `SchedulerRegistry.getTimeouts()` method as follows:
다음과 같이 `SchedulerRegistry.getTimeouts()` 메소드를 사용하여 모든 제한시간을 **나열**합니다.

```typescript
getTimeouts() {
  const timeouts = this.schedulerRegistry.getTimeouts();
  timeouts.forEach(key => this.logger.log(`Timeout: ${key}`));
}
```

#### Example

작동하는 예는 [여기](https://github.com/nestjs/nest/tree/master/sample/27-scheduling)에서 확인할 수 있습니다.
