### gRPC

[gRPC](https://github.com/grpc/grpc-node)는 모든 환경에서 실행할 수 있는 최신 오픈소스 고성능 RPC 프레임 워크입니다. 로드 밸런싱, 추적, 상태 확인 및 인증을 위한 플러그형 지원을 통해 데이터 센터 안팎의 서비스를 효율적으로 연결할 수 있습니다.

많은 RPC 시스템과 마찬가지로 gRPC는 원격으로 호출할 수 있는 기능(메소드) 측면에서 서비스를 정의하는 개념을 기반으로합니다. 각 메서드에 대해 매개변수와 반환타입을 정의합니다. 서비스, 매개변수, 반환타입은 Google의 오픈소스 언어 중립적인 [프로토콜 버퍼](https://developers.google.com/protocol-buffers) 메커니즘을 사용하여 `.proto` 파일에 정의됩니다.

gRPC 전송기를 통해 Nest는 `.proto` 파일을 사용하여 클라이언트와 서버를 동적으로 바인딩하여 원격 절차 호출을 쉽게 구현하고 구조화 된 데이터를 자동으로 직렬화 및 역직렬화합니다.

#### Installation

gRPC 기반 마이크로서비스 빌드를 시작하려면 먼저 필요한 패키지를 설치하세요.

```bash
$ npm i --save grpc @grpc/proto-loader
```

#### Overview

다른 Nest 마이크로서비스 전송 레이어 구현과 마찬가지로 `createMicroservice()` 메서드에 전달된 옵션 객체의 `transport` 속성을 사용하여 gRPC 전송기 메커니즘을 선택합니다. 다음 예에서는 히어로 서비스를 설정합니다. `options` 속성은 해당 서비스에 대한 메타데이터를 제공합니다. 해당 속성은 <에 href=""> [아래](microservices/grpc#options)에 설명되어 있습니다.

```typescript
@@filename(main)
const app = await NestFactory.createMicroservice<MicroserviceOptions>(AppModule, {
  transport: Transport.GRPC,
  options: {
    package: 'hero',
    protoPath: join(__dirname, 'hero/hero.proto'),
  },
});
@@switch
const app = await NestFactory.createMicroservice(AppModule, {
  transport: Transport.GRPC,
  options: {
    package: 'hero',
    protoPath: join(__dirname, 'hero/hero.proto'),
  },
});
```

> info **힌트** `join()` 함수는 `path` 패키지에서 가져옵니다. `Transport` 열거형은 `@nestjs/microservices` 패키지에서 가져옵니다.

`nest-cli.json` 파일에 비 TypeScript 파일을 배포할 수있는 `assets` 속성과 TypeScript가 아닌 모든 자산 감시를 켜는 `watchAssets` 속성을 추가합니다. 우리의 경우 `.proto` 파일이 `dist` 폴더에 자동으로 복사되기를 원합니다.

```json
{
  "compilerOptions": {
    "assets": ["**/*.proto"],
    "watchAssets": true
  }
}
```

#### Options

**gRPC** 전송자 옵션 객체는 아래 설명된 속성을 노출합니다.

<table>
  <tr>
    <td><code>package</code></td>
    <td>Protobuf 패키지 이름(<code>.proto</code> 파일의 <code>package</code> 설정과 일치). 필수항목</td>
  </tr>
  <tr>
    <td><code>protoPath</code></td>
    <td>
      <code>.proto</code> 파일의 절대(또는 루트 디렉토리에 상대적인)경로입니다. 필수항목
    </td>
  </tr>
  <tr>
    <td><code>url</code></td>
    <td>
    연결 URL. 전송자가 연결을 설정하는 주소/포트를 정의하는 <code>ip 주소/dns 이름:포트</code>(예: <code>'localhost:50051'</code>) 형식의 문자열입니다. 선택항목. 기본값은 <code>'localhost:5000'</code>입니다.
    </td>
  </tr>
  <tr>
    <td><code>protoLoader</code></td>
    <td>
    <code>.proto</code> 파일을 로드하기위한 유틸리티의 NPM 패키지 이름입니다. 선택항목. 기본값은 <code>'@grpc/proto-loader'</code>입니다.
    </td>
  </tr>
  <tr>
    <td><code>loader</code></td>
    <td>
      <code>@grpc/proto-loader</code> 옵션. 이를 통해 <code>.proto</code> 파일의 동작을 세부적으로 제어 할 수 있습니다. 선택사항. 자세한 내용은 <a
        href="https://github.com/grpc/grpc-node/blob/master/packages/proto-loader/README.md"
        rel="nofollow"
        target="_blank"
        >여기</a
      >를 참조하세요.
    </td>
  </tr>
  <tr>
    <td><code>credentials</code></td>
    <td>
      서버 자격 증명. 선택사항. 자세한 내용은 <a
        href="https://grpc.io/grpc/node/grpc.ServerCredentials.html"
        rel="nofollow"
        target="_blank"
        >여기</a
      >를 참조하세요
    </td>
  </tr>
</table>

#### Sample gRPC service

`HeroesService`라는 샘플 gRPC 서비스를 정의해 보겠습니다. 위의 `options` 객체에서 `protoPath` 속성은 `.proto` 정의 파일 `hero.proto`의 경로를 설정합니다. `hero.proto` 파일은 [프로토콜 버퍼](https://developers.google.com/protocol-buffers)를 사용하여 구성됩니다. 다음과 같이 표시됩니다.

```typescript
// hero/hero.proto
syntax = "proto3";

package hero;

service HeroesService {
  rpc FindOne (HeroById) returns (Hero) {}
}

message HeroById {
  int32 id = 1;
}

message Hero {
  int32 id = 1;
  string name = 2;
}
```

우리의 `HeroesService`는 `FindOne()`메소드를 노출합니다. 이 메소드는 `HeroById` 타입의 입력 인수를 예상하고 `Hero` 메시지를 반환합니다(프로토콜 버퍼는 `message` 요소를 사용하여 매개변수 타입과 반환 타입을 모두 정의합니다).

다음으로 서비스를 구현해야합니다. 이 정의를 충족하는 핸들러를 정의하기 위해 아래와 같이 컨트롤러에서 `@GrpcMethod()` 데코레이터를 사용합니다. 이 데코레이터는 메소드를 gRPC 서비스 메소드로 선언하는데 필요한 메타데이터를 제공합니다.

> info **힌트** 이전 마이크로서비스 장에서 소개한 `@MessagePattern()` 데코레이터 ([자세히 알아보기](/microservices/basics#request-response))는 gRPC 기반 마이크로서비스에 사용되지 않습니다. `@GrpcMethod()` 데코레이터는 gRPC 기반 마이크로서비스를 효과적으로 대신합니다.

```typescript
@@filename(heroes.controller)
@Controller()
export class HeroesController {
  @GrpcMethod('HeroesService', 'FindOne')
  findOne(data: HeroById, metadata: Metadata, call: ServerUnaryCall<any>): Hero {
    const items = [
      { id: 1, name: 'John' },
      { id: 2, name: 'Doe' },
    ];
    return items.find(({ id }) => id === data.id);
  }
}
@@switch
@Controller()
export class HeroesController {
  @GrpcMethod('HeroesService', 'FindOne')
  findOne(data, metadata, call) {
    const items = [
      { id: 1, name: 'John' },
      { id: 2, name: 'Doe' },
    ];
    return items.find(({ id }) => id === data.id);
  }
}
```

> info **힌트** `@GrpcMethod()` 데코레이터는 `@nestjs/microservices` 패키지에서 가져오고 `Metadata` 및 `ServerUnaryCall`은 `grpc` 패키지에서 가져옵니다.

위에 표시된 데코레이터는 두가지 인수를 사용합니다. 첫번째는 `hero.proto`의 `HeroesService` 서비스 정의에 해당하는 서비스 이름(예: `'HeroesService'`)입니다. 두번째(`FindOne` 문자열)는 `hero.proto` 파일의 `HeroesService`내에 정의된 `FindOne()` rpc 메서드에 해당합니다.

`findOne()` 핸들러 메소드는 호출자로부터 전달된 `data`, gRPC 요청 메타데이터를 저장하는 `metadata`, 클라이언트에 메타데이터를 보내기 위한 `sendMetadata`와 같은 `GrpcCall` 객체 속성을 가져 오는 `call`의 세가지 인수를 사용합니다.

`@GrpcMethod()` 데코레이터 인수는 모두 선택사항입니다. 두번째 인수(예: `'FindOne'`)없이 호출되면 Nest는 핸들러 이름을 대문자 카멜케이스로 변환하여 `.proto` 파일 rpc 메서드를 핸들러와 자동으로 연결합니다(예: `findOne` 핸들러는 `FindOne` rpc 호출 정의와 연관됨). 이것은 아래와 같습니다.

```typescript
@@filename(heroes.controller)
@Controller()
export class HeroesController {
  @GrpcMethod('HeroesService')
  findOne(data: HeroById, metadata: Metadata, call: ServerUnaryCall<any>): Hero {
    const items = [
      { id: 1, name: 'John' },
      { id: 2, name: 'Doe' },
    ];
    return items.find(({ id }) => id === data.id);
  }
}
@@switch
@Controller()
export class HeroesController {
  @GrpcMethod('HeroesService')
  findOne(data, metadata, call) {
    const items = [
      { id: 1, name: 'John' },
      { id: 2, name: 'Doe' },
    ];
    return items.find(({ id }) => id === data.id);
  }
}
```

첫번째 `@GrpcMethod()` 인수를 생략할 수도 있습니다. 이 경우 Nest는 핸들러가 정의된 **class** 이름을 기반으로 proto 정의 파일의 서비스 정의와 핸들러를 자동으로 연결합니다. 예를 들어, 다음 코드에서 `HeroesService` 클래스는 `'HeroesService'` 이름의 일치를 기반으로 `hero.proto` 파일의 `HeroesService` 서비스 정의와 핸들러 메소드를 연관시킵니다.

```typescript
@@filename(heroes.controller)
@Controller()
export class HeroesService {
  @GrpcMethod()
  findOne(data: HeroById, metadata: Metadata, call: ServerUnaryCall<any>): Hero {
    const items = [
      { id: 1, name: 'John' },
      { id: 2, name: 'Doe' },
    ];
    return items.find(({ id }) => id === data.id);
  }
}
@@switch
@Controller()
export class HeroesService {
  @GrpcMethod()
  findOne(data, metadata, call) {
    const items = [
      { id: 1, name: 'John' },
      { id: 2, name: 'Doe' },
    ];
    return items.find(({ id }) => id === data.id);
  }
}
```

#### Client

Nest 애플리케이션은 `.proto` 파일에 정의된 서비스를 사용하는 gRPC 클라이언트 역할을 할 수 있습니다. `ClientGrpc` 객체를 통해 원격 서비스에 액세스합니다. 여러 가지 방법으로 `ClientGrpc` 객체를 얻을 수 있습니다.

선호하는 기술은 `ClientsModule`을 가져 오는 것입니다. `register()` 메서드를 사용하여 `.proto` 파일에 정의된 서비스 패키지를 인젝션 토큰에 바인딩하고 서비스를 구성합니다. `name` 속성은 주입 토큰입니다. gRPC 서비스의 경우 `transport: Transport.GRPC`를 사용하세요. `options` 속성은 [위](/microservices/grpc#options)에 설명된 것과 동일한 속성을 가진 객체입니다.

```typescript
imports: [
  ClientsModule.register([
    {
      name: 'HERO_PACKAGE',
      transport: Transport.GRPC,
      options: {
        package: 'hero',
        protoPath: join(__dirname, 'hero/hero.proto'),
      },
    },
  ]),
];
```

> info **힌트** `register()` 메서드는 객체의 배열을 받습니다. 쉼표로 구분된 등록 객체 목록을 제공하여 여러 패키지를 등록합니다.

등록이 완료되면 구성된 `ClientGrpc` 객체를 `@Inject()`로 삽입할 수 있습니다. 그런 다음 `ClientGrpc` 객체의 `getService()` 메서드를 사용하여 아래와 같이 서비스 인스턴스를 검색합니다.

```typescript
@Injectable()
export class AppService implements OnModuleInit {
  private heroesService: HeroesService;

  constructor(@Inject('HERO_PACKAGE') private client: ClientGrpc) {}

  onModuleInit() {
    this.heroesService = this.client.getService<HeroesService>('HeroesService');
  }

  getHero(): Observable<string> {
    return this.heroesService.findOne({ id: 1 });
  }
}
```

> error **경고** gRPC 클라이언트는 프로토 로더 구성에서 `keepCase` 옵션이 `true`로 설정되어 있지 않으면 이름에 밑줄 `_`가 포함된 필드를 보내지 않습니다 (마이크로 서비스 전송기 구성의 `options.loader.keepcase`).

다른 마이크로서비스 전송 방법에 사용되는 기술과 비교하면 약간의 차이가 있습니다. `ClientProxy` 클래스 대신 `getService()` 메소드를 제공하는 `ClientGrpc` 클래스를 사용합니다. `getService()` 제네릭 메서드는 서비스 이름을 인수로 사용하고 해당 인스턴스(사용 가능한 경우)를 반환합니다.

또는 다음과 같이 `@Client()` 데코레이터를 사용하여 `ClientGrpc` 객체를 인스턴스화할 수 있습니다.

```typescript
@Injectable()
export class AppService implements OnModuleInit {
  @Client({
    transport: Transport.GRPC,
    options: {
      package: 'hero',
      protoPath: join(__dirname, 'hero/hero.proto'),
    },
  })
  client: ClientGrpc;

  private heroesService: HeroesService;

  onModuleInit() {
    this.heroesService = this.client.getService<HeroesService>('HeroesService');
  }

  getHero(): Observable<string> {
    return this.heroesService.findOne({ id: 1 });
  }
}
```

마지막으로 더 복잡한 시나리오의 경우 [여기](/microservices/basics#client)에 설명된대로 `ClientProxyFactory` 클래스를 사용하여 동적으로 구성된 클라이언트를 삽입할 수 있습니다.

두 경우 모두, 우리는 `.proto` 파일내에 정의된 동일한 메소드 세트를 노출하는 `HeroesService` 프록시 객체에 대한 참조로 끝납니다. 이제 이 프록시 객체(예: `heroesService`)에 액세스하면 gRPC 시스템이 요청을 자동으로 직렬화하고 원격 시스템에 전달하고 응답을 반환하고 응답을 역직렬화합니다. gRPC는 이러한 네트워크 통신 세부 정보로부터 우리를 보호하기 때문에 `heroesService`는 로컬 제공 업체처럼 보이고 작동합니다.

모든 서비스 방법은 **lower camel cased**입니다(언어의 자연스러운 규칙을 따르기 위해). 예를 들어 `.proto` 파일 `HeroesService` 정의에는 `FindOne()` 함수가 포함되어 있지만 `heroesService` 인스턴스는 `findOne()` 메서드를 제공합니다.

```typescript
interface HeroesService {
  findOne(data: { id: number }): Observable<any>;
}
```

메시지 핸들러는 `Observable`을 반환할 수도 있습니다.이 경우 스트림이 완료될 때까지 결과값이 내보내집니다.

```typescript
@@filename(heroes.controller)
@Get()
call(): Observable<any> {
  return this.heroesService.findOne({ id: 1 });
}
@@switch
@Get()
call() {
  return this.heroesService.findOne({ id: 1 });
}
```

요청과 함께 gRPC 메타 데이터를 전송하려면 다음과 같이 두번째 인수를 전달할 수 있습니다.

```typescript
call(): Observable<any> {
  const metadata = new Metadata();
  metadata.add('Set-Cookie', 'yummy_cookie=choco');

  return this.heroesService.findOne({ id: 1 }, metadata);
}
```

> info **힌트** `Metadata` 클래스는 `grpc` 패키지에서 가져옵니다.

이를 위해서는 몇단계 이전에 정의한 `HeroesService` 인터페이스를 업데이트해야합니다.

#### Example

작동하는 예는 [여기](https://github.com/nestjs/nest/tree/master/sample/04-grpc)에서 확인할 수 있습니다.

#### gRPC Streaming

gRPC는 자체적으로 장기 라이브 연결을 지원하며, 일반적으로 `streams`이라고 합니다. 스트림은 채팅, 관찰 또는 청크 데이터 전송과 같은 경우에 유용합니다. 자세한 내용은 공식 문서 [여기](https://grpc.io/docs/guides/concepts/)에서 확인하세요.

Nest는 두가지 방법으로 GRPC 스트림 핸들러를 지원합니다.

- RxJS `Subject` + `Observable` 핸들러: Controller 메서드내에서 바로 응답을 작성하거나 `Subject`/`Observable` 소비자에게 전달하는데 유용할 수 있습니다.
- 순수한 GRPC 호출 스트림 핸들러: Node 표준 `Duplex` 스트림 핸들러에 대한 나머지 디스패치를 처리할 일부 실행기(executor)에 전달하는 것이 유용할 수 있습니다.

<app-banner-enterprise></app-banner-enterprise>

#### Streaming sample

`HelloService`라는 새 샘플 gRPC 서비스를 정의해 보겠습니다. `hello.proto` 파일은 [프로토콜 버퍼](https://developers.google.com/protocol-buffers)를 사용하여 구성됩니다. 다음과 같이 표시됩니다.

```typescript
// hello/hello.proto
syntax = "proto3";

package hello;

service HelloService {
  rpc BidiHello(stream HelloRequest) returns (stream HelloResponse);
  rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse);
}

message HelloRequest {
  string greeting = 1;
}

message HelloResponse {
  string reply = 1;
}
```

> info **힌트** 반환된 스트림이 여러 값을 내보낼 수 있으므로 `LotsOfGreetings` 메소드는 `@GrpcMethod` 데코레이터(위의 예에서와 같이)로 간단히 구현할 수 있습니다.

이 `.proto` 파일을 기반으로 `HelloService` 인터페이스를 정의해 보겠습니다.

```typescript
interface HelloService {
  bidiHello(upstream: Observable<HelloRequest>): Observable<HelloResponse>;
  lotsOfGreetings(
    upstream: Observable<HelloRequest>,
  ): Observable<HelloResponse>;
}

interface HelloRequest {
  greeting: string;
}

interface HelloResponse {
  reply: string;
}
```

#### Subject strategy

`@GrpcStreamMethod()` 데코레이터는 함수 매개변수를 RxJS `Observable`로 제공합니다. 따라서 여러 메시지를 수신하고 처리할 수 있습니다.

```typescript
@GrpcStreamMethod()
bidiHello(messages: Observable<any>, metadata: Metadata, call: ServerDuplexStream<any, any>): Observable<any> {
  const subject = new Subject();

  const onNext = message => {
    console.log(message);
    subject.next({
      reply: 'Hello, world!'
    });
  };
  const onComplete = () => subject.complete();
  messages.subscribe({
    next: onNext,
    complete: onComplete,
  });


  return subject.asObservable();
}
```

> warning **경고** `@GrpcStreamMethod()` 데코레이터와의 전이중 상호작용을 지원하려면 컨트롤러 메서드가 RxJS `Observable`을 반환해야합니다.

> info **힌트** `Metadata` 및 `ServerUnaryCall` 클래스/인터페이스는 `grpc` 패키지에서 가져옵니다.

서비스 정의(`.proto` 파일에 있음)에 따르면 `BidiHello` 메소드는 요청을 서비스로 스트리밍해야합니다. 클라이언트에서 스트림으로 여러 비동기 메시지를 보내기 위해 RxJS `ReplySubject` 클래스를 활용합니다.

```typescript
const helloService = this.client.getService<HelloService>('HelloService');
const helloRequest$ = new ReplaySubject<HelloRequest>();

helloRequest$.next({ greeting: 'Hello (1)!' });
helloRequest$.next({ greeting: 'Hello (2)!' });
helloRequest$.complete();

return helloService.bidiHello(helloRequest$);
```

위의 예에서는 스트림에 두개의 메시지를 작성하고(`next()` 호출) 서비스에 데이터 전송을 완료했음을 알렸습니다(`complete()` 호출).

#### Call stream handler

메서드 반환 값이 `stream`으로 정의되면 `@GrpcStreamCall()` 데코레이터는 `.on('data', callback)`, `.write(message)` 또는 `.cancel()`과 같은 표준 메서드를 지원하는 `grpc.ServerDuplexStream`으로 함수 매개변수를 제공합니다. 사용 가능한 메서드에 대한 전체 문서는 [여기](https://grpc.github.io/grpc/node/grpc-ClientDuplexStream.html)에서 찾을 수 있습니다.

또는 메소드 반환 값이 `stream`이 아닌 경우 `@GrpcStreamCall()` 데코레이터는 각각 `grpc.ServerReadableStream` 및 `callback` 두개의 함수 매개변수를 제공합니다 ([여기에서 자세히 알아보기](https://grpc.github.io/grpc/node/grpc-ServerReadableStream.html))

전이중 상호작용(full-duplex interaction)을 지원해야하는 `BidiHello` 구현부터 시작하겠습니다.

```typescript
@GrpcStreamCall()
bidiHello(requestStream: any) {
  requestStream.on('data', message => {
    console.log(message);
    requestStream.write({
      reply: 'Hello, world!'
    });
  });
}
```

> info **힌트** 이 데코레이터는 특정 반환 매개변수를 제공할 필요가 없습니다. 스트림은 다른 표준 스트림 타입과 유사하게 처리될 것으로 예상됩니다.

위의 예에서는 `write()` 메서드를 사용하여 응답 스트림에 객체를 썼습니다. 두번째 매개 변수로 `.on()` 메소드에 전달된 콜백은 서비스가 새 데이터 청크를 수신할 때마다 호출됩니다.

`LotsOfGreetings` 메소드를 구현해 보겠습니다.

```typescript
@GrpcStreamCall()
lotsOfGreetings(requestStream: any, callback: (err: unknown, value: HelloResponse) => void) {
  requestStream.on('data', message => {
    console.log(message);
  });
  requestStream.on('end', () => callback(null, { reply: 'Hello, world!' }));
}
```

여기서는 `requestStream` 처리가 완료되면 `callback` 함수를 사용하여 응답을 보냅니다.

#### gRPC Metadata

메타데이터는 키-값 쌍 목록 형식의 특정 RPC 호출에 대한 정보입니다. 여기서 키는 문자열이고 값은 일반적으로 문자열이지만 이진 데이터일 수 있습니다. 메타데이터는 gRPC 자체에 대해 불투명합니다.이를 통해 클라이언트는 서버 호출과 관련된 정보를 제공할 수 있으며 그 반대의 경우도 마찬가지입니다. 메타데이터에는 인증 토큰, 모니터링 목적을 위한 요청 식별자 및 태그, 데이터 세트의 레코드 수와 같은 데이터 정보가 포함될 수 있습니다.

`@GrpcMethod()` 핸들러에서 메타데이터를 읽으려면 `Metadata` 타입(`grpc` 패키지에서 가져옴)인 두번째 인수(메타 데이터)를 사용합니다.

핸들러에서 메타데이터를 다시 보내려면 `ServerUnaryCall#sendMetadata()` 메서드(세번째 핸들러 인수)를 사용합니다.

```typescript
@@filename(heroes.controller)
@Controller()
export class HeroesService {
  @GrpcMethod()
  findOne(data: HeroById, metadata: Metadata, call: ServerUnaryCall<any>): Hero {
    const serverMetadata = new Metadata();
    const items = [
      { id: 1, name: 'John' },
      { id: 2, name: 'Doe' },
    ];

    serverMetadata.add('Set-Cookie', 'yummy_cookie=choco');
    call.sendMetadata(serverMetadata);

    return items.find(({ id }) => id === data.id);
  }
}
@@switch
@Controller()
export class HeroesService {
  @GrpcMethod()
  findOne(data, metadata, call) {
    const serverMetadata = new Metadata();
    const items = [
      { id: 1, name: 'John' },
      { id: 2, name: 'Doe' },
    ];

    serverMetadata.add('Set-Cookie', 'yummy_cookie=choco');
    call.sendMetadata(serverMetadata);

    return items.find(({ id }) => id === data.id);
  }
}
```

마찬가지로 `@GrpcStreamMethod()` 핸들러([subject strategy](microservices/grpc#subject-strategy))로 주석이 달린 핸들러에서 메타 데이터를 읽으려면 `Metadata`(`grpc` 패키지에서 가져옴).

핸들러에서 메타 데이터를 다시 보내려면 `ServerDuplexStream#sendMetadata()` 메서드(세번째 핸들러 인수)를 사용하세요.

[호출 스트림 핸들러](microservices/grpc#call-stream-handler) (`@GrpcStreamCall()` 데코레이터로 주석이 달린 핸들러)내에서 메타 데이터를 읽으려면 `requestStream` 참조에서 `metadata` 이벤트를 수신합니다. 다음과 같습니다.

```typescript
requestStream.on('metadata', (metadata: Metadata) => {
  const meta = metadata.get('X-Meta');
});
```
