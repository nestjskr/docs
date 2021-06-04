### Serve Static

SPA(단일 페이지 애플리케이션)와 같은 정적 콘텐츠를 제공하기 위해 [`@nestjs/serve-static`](https://www.npmjs.com/package/@nestjs/serve-static) 패키지의 `ServeStaticModule`을 사용할 수 있습니다..

#### Installation

먼저 필요한 패키지를 설치해야 합니다.

```bash
$ npm install --save @nestjs/serve-static
```

#### Bootstrap

설치 프로세스가 완료되면 `ServeStaticModule`을 루트 `AppModule`로 가져 와서 `forRoot()` 메소드에 구성객체를 전달하여 구성할 수 있습니다.

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { ServeStaticModule } from '@nestjs/serve-static';
import { join } from 'path';

@Module({
  imports: [
    ServeStaticModule.forRoot({
      rootPath: join(__dirname, '..', 'client'),
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

이 위치를 사용하여 정적 웹 사이트를 빌드하고 `rootPath` 속성으로 지정된 위치에 해당 콘텐츠를 배치합니다.

#### Configuration

[ServeStaticModule](https://github.com/nestjs/serve-static)은 다양한 옵션으로 구성하여 동작을 사용자 지정할 수 있습니다.
정적 앱을 렌더링하기 위한 경로를 설정하고, 제외된 경로를 지정하고, Cache-Control 응답 헤더 설정을 사용 또는 사용 중지하는 등의 작업을 수행할 수 있습니다. 전체 옵션 목록은 [여기](https://github.com/nestjs/serve-static/blob/master/lib/interfaces/serve-static-options.interface.ts)를 참조하세요.

> warning **알림** 정적 앱의 기본 `renderPath`는 `*`(모든 경로)이며 모듈은 응답으로 "index.html" 파일을 보냅니다.
> SPA에 대한 클라이언트 측 라우팅을 생성할 수 있습니다. 컨트롤러에 지정된 경로는 서버로 대체됩니다.
> 이 동작 설정 `serveRoot`, `renderPath`를 다른 옵션과 결합하여 변경할 수 있습니다.


#### Example

작동하는 예제는 [여기](https://github.com/nestjs/nest/tree/master/sample/24-serve-static)에서 확인할 수 있습니다.
