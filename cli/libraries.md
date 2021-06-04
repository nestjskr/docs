### Libraries

많은 애플리케이션은 동일한 일반적인 문제를 해결하거나 여러 다른 상황에서 모듈식 구성 요소를 재사용해야 합니다. Nest에는 이 문제를 해결하는 몇가지 방법이 있지만 각각 다른 수준에서 작동하여 서로 다른 아키텍처 및 조직 목표를 충족하는 데 도움이 되는 방식으로 문제를 해결합니다.

Nest [모듈](/modules)은 단일 애플리케이션내에서 구성요소를 공유할 수 있는 실행 컨텍스트를 제공하는 데 유용합니다. 모듈은 [npm](https://npmjs.com)과 함께 패키징하여 다른 프로젝트에 설치할 수 있는 재사용 가능한 라이브러리를 만들 수도 있습니다. 이는 서로 다른, 느슨하게 연결되거나 제휴되지 않은 조직(예: 타사 라이브러리 배포/설치)에서 사용할 수 있는 구성 가능하고 재사용 가능한 라이브러리를 배포하는 효과적인 방법이 될 수 있습니다.

밀접하게 조직된 그룹(예: 회사/프로젝트 경계내)내에서 코드를 공유하려면 구성요소 공유에 대해 보다 가벼운 접근 방식을 사용하는 것이 유용할 수 있습니다. 모노레포는 이를 가능하게 하는 구조로 생겨 났으며, 모노레포내에서 **라이브러리**는 쉽고 가벼운 방식으로 코드를 공유하는 방법을 제공합니다. Nest 모노레포에서 라이브러리를 사용하면 구성요소를 공유하는 애플리케이션을 쉽게 조립할 수 있습니다. 사실, 이것은 모놀리식 애플리케이션과 개발 프로세스의 분해를 장려하여 모듈식 구성요소를 구축하고 구성하는 데 집중합니다.

#### Nest libraries

Nest 라이브러리는 자체적으로 실행할 수 없다는 점에서 애플리케이션과 다른 Nest 프로젝트입니다. 코드를 실행하려면 라이브러리를 포함하는 애플리케이션으로 가져와야 합니다. 이 섹션에 설명된 라이브러리에 대한 기본 제공 지원은 **모노레포**에서만 사용할 수 있습니다 (표준 모드 프로젝트는 npm 패키지를 사용하여 유사한 기능을 달성할 수 있음).

예를 들어 조직은 모든 내부 애플리케이션을 관리하는 회사 정책을 구현하여 인증을 관리하는 `AuthModule`을 개발할 수 있습니다. 각 애플리케이션에 대해 해당 모듈을 개별적으로 빌드하거나 npm으로 코드를 물리적으로 패키징하고 각 프로젝트에서 이를 설치하도록 요구하는 대신 모노레포는 이 모듈을 라이브러리로 정의할 수 있습니다. 이러한 방식으로 구성하면 라이브러리 모듈의 모든 소비자가 커밋 된 `AuthModule`의 최신 버전을 볼 수 있습니다. 이는 구성요소 개발 및 조립을 조정하고 종단간 테스트를 단순화하는 데 상당한 이점을 제공할 수 있습니다.

#### Creating libraries

재사용에 적합한 모든 기능은 라이브러리로 관리할 후보입니다. 라이브러리가 되어야 하고 애플리케이션의 일부가 되어야하는 것을 결정하는 것은 아키텍처 설계 결정입니다. 라이브러리를 만드는 것은 단순히 기존 애플리케이션에서 새 라이브러리로 코드를 복사하는 것 이상을 포함합니다. 라이브러리로 패키징된 경우 라이브러리 코드는 애플리케이션에서 분리되어야 합니다. 이를 위해 **더** 많은 시간이 필요할 수 있으며 더 밀접하게 결합된 코드에서는 직면하지 않을 수 있는 일부 디자인 결정을 강제할 수 있습니다. 그러나 이러한 추가 노력은 라이브러리를 사용하여 여러 애플리케이션에서 보다 빠른 애플리케이션 조립을 가능하게 할 때 효과가 있을 수 있습니다.

라이브러리 생성을 시작하려면 다음 명령을 실행하십시오.

```bash
nest g library my-library
```

명령을 실행하면 `library` 스키메틱이 라이브러리의 접두사(일명 별칭)를 입력하라는 메시지가 표시됩니다.

```bash
What prefix would you like to use for the library (default: @app)?
```

그러면 작업 공간에 `my-library`라는 새 프로젝트가 생성됩니다.
애플리케이션 유형 프로젝트와 같은 라이브러리 유형 프로젝트는 스키메틱을 사용하여 명명된 폴더에 생성됩니다. 라이브러리는 모노레포 루트의 `libs` 폴더에서 관리됩니다. Nest는 라이브러리가 처음 생성될 때 `libs` 폴더를 생성합니다.

라이브러리용으로 생성된 파일은 애플리케이션용으로 생성된 파일과 약간 다릅니다. 위의 명령을 실행한 후 `libs` 폴더의 내용은 다음과 같습니다.

<div class="file-tree">
  <div class="item">libs</div>
  <div class="children">
    <div class="item">my-library</div>
    <div class="children">
      <div class="item">src</div>
      <div class="children">
        <div class="item">index.ts</div>
        <div class="item">my-library.module.ts</div>
        <div class="item">my-library.service.ts</div>
      </div>
      <div class="item">tsconfig.lib.json</div>
    </div>
  </div>
</div>

`nest-cli.json` 파일에는 `"projects"`키 아래에 라이브러리에 대한 새 항목이 있습니다.

```javascript
...
{
    "my-library": {
      "type": "library",
      "root": "libs/my-library",
      "entryFile": "index",
      "sourceRoot": "libs/my-library/src",
      "compilerOptions": {
        "tsConfigPath": "libs/my-library/tsconfig.lib.json"
      }
}
...
```

라이브러리와 애플리케이션간에 `nest-cli.json` 메타데이터에는 두가지 차이점이 있습니다.

- `"type"` 속성이 `"application"`대신 `"library"`로 설정됨
- `"entryFile"` 속성이 `"main"`대신 `"index"`로 설정됨

이러한 차이점은 라이브러리를 적절하게 처리하기 위한 빌드 프로세스의 핵심입니다. 예를 들어 라이브러리는 `index.js` 파일을 통해 함수를 내보냅니다.

애플리케이션 유형 프로젝트와 마찬가지로 라이브러리에는 각각 루트(모노레포 전체) `tsconfig.json` 파일을 확장하는 자체 `tsconfig.lib.json` 파일이 있습니다. 필요한 경우 이 파일을 수정하여 라이브러리별 컴파일러 옵션을 제공할 수 있습니다.

CLI 명령을 사용하여 라이브러리를 빌드 할 수 있습니다.

```bash
nest build my-library
```

#### Using libraries

자동으로 생성된 구성 파일이 있으면 라이브러리를 사용하는 것이 간단합니다. `my-library` 라이브러리에서 `my-project` 애플리케이션으로 `MyLibraryService`를 가져오는 방법은 무엇입니까?

먼저 라이브러리 모듈을 사용하는 것은 다른 Nest 모듈을 사용하는 것과 동일합니다. 모노레포가 하는 일은 라이브러리를 가져오고 빌드를 생성하는 것이 이제 투명하게 경로를 관리하는 것입니다. `MyLibraryService`를 사용하려면 선언 모듈을 가져와야 합니다. `my-project/src/app.module.ts`를 다음과 같이 수정하여 `MyLibraryModule`을 가져올 수 있습니다.

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { MyLibraryModule } from '@app/my-library';

@Module({
  imports: [MyLibraryModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

위의 `nest g library` 명령으로 제공한 `접두사 prefix`인 ES 모듈 `import` 줄에서 `@app`의 경로 별칭을 사용했습니다. 내부적으로 Nest는 tsconfig 경로 매핑을 통해 이를 처리합니다. 라이브러리를 추가할 때 Nest는 전역(monorepo) `tsconfig.json` 파일의 `"paths"`키를 다음과 같이 업데이트합니다.

```javascript
"paths": {
    "@app/my-library": [
        "libs/my-library/src"
    ],
    "@app/my-library/*": [
        "libs/my-library/src/*"
    ]
}
```

요컨대, 모노레포와 라이브러리 기능의 조합은 라이브러리 모듈을 애플리케이션에 포함하는 것을 쉽고 직관적으로 만들었습니다.

이 동일한 메커니즘을 통해 라이브러리를 구성하는 애플리케이션을 빌드하고 배포할 수 있습니다. `MyLibraryModule`을 가져온 후 `nest build`를 실행하면 모든 모듈 확인이 자동으로 처리되고 배포를 위해 모든 라이브러리 종속성과 함께 앱이 번들로 제공됩니다. 모노레포의 기본 컴파일러는 **웹팩 webpack**이므로 결과 배포 파일은 트랜스파일된 모든 JavaScript 파일을 단일 파일로 묶는 단일 파일입니다. [여기](/cli/monorepo#global-compiler-options)에 설명 된대로 `tsc`로 전환할 수도 있습니다.
