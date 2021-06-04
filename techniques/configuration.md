### Configuration

애플리케이션은 종종 서로 다른 **환경**에서 실행됩니다. 환경에 따라 다른 구성 설정을 사용해야 합니다. 예를 들어, 일반적으로 로컬 환경은 로컬 DB 인스턴스에만 유효한 특정 데이터베이스 자격증명에 의존합니다. 프로덕션 환경은 별도의 DB 자격증명 세트를 사용합니다. 구성 변수가 변경되므로 환경에서 [구성 변수 저장](https://12factor.net/config)을 사용하는 것이 가장 좋습니다.

외부에서 정의된 환경 변수는 `process.env` 전역을 통해 Node.js 내부에서 볼 수 있습니다. 각 환경에서 개별적으로 환경 변수를 설정하여 여러 환경의 문제를 해결할 수 있습니다. 이는 특히 이러한 값을 쉽게 모의하거나 변경해야 하는 개발 및 테스트 환경에서 빠르게 다루기 어려울 수 있습니다.

Node.js 애플리케이션에서는 각 환경을 나타내기 위해 각 키가 특정 값을 나타내는 키-값 쌍을 보유한 `.env` 파일을 사용하는 것이 일반적입니다. 다른 환경에서 앱을 실행하는 것은 올바른 `.env` 파일에서 스와핑하면 됩니다.

Nest에서이 기술을 사용하는 좋은 방법은 적절한 `.env` 파일을 로드하는 `ConfigService`를 노출하는 `ConfigModule`을 만드는 것입니다. 이러한 모듈을 직접 작성하도록 선택할 수 있지만 편의를 위해 Nest는 즉시 사용 가능한 `@nestjs/config` 패키지를 제공합니다. 이 패키지는 현재 장에서 다룰 것입니다.

#### Installation

사용을 시작하려면 먼저 필요한 종속성을 설치합니다.

```bash
$ npm i --save @nestjs/config
```

> info **힌트** `@nestjs/config` 패키지는 내부적으로 [dotenv](https://github.com/motdotla/dotenv)를 사용합니다.

#### Getting started

설치 프로세스가 완료되면 `ConfigModule`을 가져올 수 있습니다. 일반적으로 루트 `AppModule`로 가져오고 `.forRoot()` 정적 메서드를 사용하여 동작을 제어합니다. 이 단계에서 환경 변수 키/값 쌍이 구문 분석되고 해결됩니다. 나중에 다른 기능 모듈에서 `ConfigModule`의 `ConfigService` 클래스에 액세스하기 위한 몇가지 옵션을 볼 수 있습니다.

```typescript
@@filename(app.module)
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [ConfigModule.forRoot()],
})
export class AppModule {}
```

위의 코드는 기본 위치(프로젝트 루트 디렉터리)에서 `.env` 파일을 로드 및 구문 분석하고 `.env` 파일의 키/값 쌍을 `process.env`에 할당된 환경 변수와 병합하고 결과적으로 `ConfigService`를 통해 액세스할 수 있는 개인 구조가 됩니다. `forRoot()` 메서드는 이러한 파싱/병합된 구성 변수를 읽기 위한 `get()` 메서드를 제공하는 `ConfigService` 프로바이더를 등록합니다. `@nestjs/config`는 [dotenv](https://github.com/motdotla/dotenv)에 의존하므로 환경 변수 이름의 충돌을 해결하기 위해 해당 패키지의 규칙을 사용합니다. 키가 런타임 환경에 환경 변수(예: `export DATABASE_USER=test`와 같은 OS 셸 내보내기를 통해)와 `.env` 파일로 모두 존재하는 경우 런타임 환경 변수가 우선합니다.

샘플 `.env` 파일은 다음과 같습니다.

```json
DATABASE_USER=test
DATABASE_PASSWORD=test
```

#### Custom env file path

기본적으로 패키지는 애플리케이션의 루트 디렉토리에서 `.env` 파일을 찾습니다. `.env` 파일의 다른 경로를 지정하려면 다음과 같이 `forRoot()`에 전달하는(선택 사항) 옵션 객체의 `envFilePath` 속성을 설정합니다.

```typescript
ConfigModule.forRoot({
  envFilePath: '.development.env',
});
```

다음과 같이 `.env` 파일에 여러 경로를 지정할 수도 있습니다.

```typescript
ConfigModule.forRoot({
  envFilePath: ['.env.development.local', '.env.development'],
});
```

변수가 여러 파일에서 발견되면 첫번째 파일이 우선합니다.

#### Disable env variables loading

`.env` 파일을 로드하지 않고 대신 런타임 환경에서 환경 변수에 액세스하려는 경우(`export DATABASE_USER=test`와 같은 OS 셸 내보내기와 마찬가지로) 옵션 객체의 `ignoreEnvFile`을 설정합니다. 다음과 같이 속성을 `true`로 설정합니다.

```typescript
ConfigModule.forRoot({
  ignoreEnvFile: true,
});
```

#### Use module globally

다른 모듈에서 `ConfigModule`을 사용하려면 가져와야합니다 (모든 Nest 모듈의 표준과 동일). 또는 아래와 같이 옵션 객체의 `isGlobal` 속성을 `true`로 설정하여 이를 [global module](/modules#global-modules)로 선언합니다. 이 경우 루트 모듈(예: `AppModule`)에 로드되면 다른 모듈에서 `ConfigModule`을 가져올 필요가 없습니다.

```typescript
ConfigModule.forRoot({
  isGlobal: true,
});
```

#### Custom configuration files

더 복잡한 프로젝트의 경우 사용자 지정 구성 파일을 사용하여 중첩된 구성 객체를 반환할 수 있습니다. 이를 통해 관련 구성 설정을 기능(예: 데이터베이스 관련 설정)별로 그룹화하고 관련 설정을 개별 파일에 저장하여 독립적으로 관리할 수 있습니다.

사용자 지정 구성 파일은 구성 개체를 반환하는 팩토리 함수를 내보냅니다. 구성 객체는 임의로 중첩된 일반 JavaScript 객체일 수 있습니다. `process.env` 객체에는 [위](/techniques/configuration#getting-started)에서 설명한대로 해결 및 병합 된 `.env` 파일 및 외부 정의 변수와 함께 완전히 해결된 환경 변수 키/값 쌍이 포함됩니다. 반환된 구성 객체를 제어하므로 값을 적절한 유형으로 캐스팅하고 기본값을 설정하는 데 필요한 논리를 추가할 수 있습니다. 예를 들면 다음과 같습니다.

```typescript
@@filename(config/configuration)
export default () => ({
  port: parseInt(process.env.PORT, 10) || 3000,
  database: {
    host: process.env.DATABASE_HOST,
    port: parseInt(process.env.DATABASE_PORT, 10) || 5432
  }
});
```

이 파일은 `ConfigModule.forRoot()` 메서드에 전달하는 옵션 객체의 `load` 속성을 사용하여 로드합니다.

```typescript
import configuration from './config/configuration';

@Module({
  imports: [
    ConfigModule.forRoot({
      load: [configuration],
    }),
  ],
})
export class AppModule {}
```

> info **알림** `load` 속성에 할당된 값은 배열이므로 여러 구성 파일을 로드할 수 있습니다 (예: `load: [databaseConfig, authConfig]`).

사용자 지정 구성 파일을 사용하면 YAML 파일과 같은 사용자 지정 파일도 관리할 수 있습니다. 다음은 YAML 형식을 사용하는 구성의 예입니다.

```yaml
http:
  host: 'localhost'
  port: 8080

db:
  postgres:
    url: 'localhost'
    port: 5432
    database: 'yaml-db'

  sqlite:
    database: 'sqlite.db'
```

YAML 파일을 읽고 파싱하기 위해 `js-yaml` 패키지를 활용할 수 있습니다.

```bash
$ npm i js-yaml
$ npm i -D @types/js-yaml
```

패키지가 설치되면 `yaml#load` 함수를 사용하여 방금 만든 YAML 파일을 로드합니다.

```typescript
@@filename(config/configuration)
import { readFileSync } from 'fs';
import * as yaml from 'js-yaml';
import { join } from 'path';

const YAML_CONFIG_FILENAME = 'config.yaml';

export default () => {
  return yaml.load(
    readFileSync(join(__dirname, YAML_CONFIG_FILENAME), 'utf8'),
  ) as Record<string, any>;
};
```

> warning **참고** Nest CLI는 빌드 프로세스 중에 "자산"(비 TS 파일)을 `dist`폴더로 자동으로 이동하지 않습니다. YAML 파일이 복사되었는지 확인하려면 `nest-cli.json` 파일의 `compilerOptions#assets` 객체에 이를 지정해야 합니다. 예를 들어,`config` 폴더가 `src` 폴더와 동일한 수준에 있는 경우 `"assets": [{{ '{' }}"include": "../config/*.yaml", "outDir": "./dist/config"{{ '}' }}]`를 추가합니다. [여기](/cli/monorepo#assets)에서 자세히 알아보세요.

<app-banner-enterprise></app-banner-enterprise>

#### Using the `ConfigService`

`ConfigService`에서 구성 값에 액세스하려면 먼저 `ConfigService`를 삽입해야합니다. 다른 프로바이더와 마찬가지로 포함 모듈인 `ConfigModule`을 사용할 모듈로 가져와야 합니다 (`ConfigModule.forRoot()` 메소드에 전달된 옵션 객체의 `isGlobal` 속성을 `true`로 설정하지 않는 한). 아래와 같이 기능 모듈로 가져옵니다.

```typescript
@@filename(feature.module)
@Module({
  imports: [ConfigModule],
  // ...
})
```

그런 다음 표준 생성자 주입을 사용하여 주입할 수 있습니다.

```typescript
constructor(private configService: ConfigService) {}
```

> info **힌트** `ConfigService`는 `@nestjs/config` 패키지에서 가져옵니다.

그리고 우리 클래스에서 사용하십시오.

```typescript
// get an environment variable
const dbUser = this.configService.get<string>('DATABASE_USER');

// get a custom configuration value
const dbHost = this.configService.get<string>('database.host');
```

위와 같이 `configService.get()` 메소드를 사용하여 변수 이름을 전달하여 간단한 환경 변수를 가져옵니다. 위에 표시된대로 타입을 전달하여 TypeScript 타입 힌트를 수행할 수 있습니다 (예: `get<string>(...)`). `get()`메소드는 위의 두번째 예와 같이 중첩된 사용자 정의 구성 객체 ([사용자 정의 구성 파일](/techniques/configuration#custom-configuration-files)을 통해 생성됨)를 탐색할 수도 있습니다.

인터페이스를 타입 힌트로 사용하여 전체 중첩된 사용자 지정 구성 객체를 가져올 수도 있습니다.

```typescript
interface DatabaseConfig {
  host: string;
  port: number;
}

const dbConfig = this.configService.get<DatabaseConfig>('database');

// you can now use `dbConfig.port` and `dbConfig.host`
const port = dbConfig.port;
```

`get()` 메서드는 또한 기본값을 정의하는 두번째 인수를 선택적으로 받습니다. 기본값은 아래와 같이 키가 존재하지 않을 때 반환됩니다.

```typescript
// use "localhost" when "database.host" is not defined
const dbHost = this.configService.get<string>('database.host', 'localhost');
```

`ConfigService`에는 존재하지 않는 구성 속성에 대한 액세스를 방지하는 데 도움이 되는 선택적 일반(타입 인수)이 있습니다. 아래와 같이 사용하십시오.

```typescript
interface EnvironmentVariables {
  PORT: number;
  TIMEOUT: string;
}

// somewhere in the code
constructor(private configService: ConfigService<EnvironmentVariables>) {
  // this is valid
  const port = this.configService.get<number>('PORT');

  // this is invalid as URL is not a property on the EnvironmentVariables interface
  const url = this.configService.get<string>('URL');
}
```

> warning **알림** 위의 `database.host` 예제와 같이 구성에 중첩된 속성이 있는 경우 인터페이스에 일치하는 `'database.host': string;` 속성이 있어야 합니다. 그렇지 않으면 TypeScript 오류가 발생합니다.

#### Configuration namespaces

`ConfigModule`을 사용하면 위의 [커스텀 구성 파일](techniques/configuration#custom-configuration-files)에 표시된대로 여러 사용자 정의 구성 파일을 정의하고 로드할 수 있습니다. 해당 섹션에 표시된 것처럼 중첩된 구성 개체를 사용하여 복잡한 구성 객체 계층을 관리할 수 있습니다. 또는 다음과 같이 `registerAs()` 함수를 사용하여 "네임스페이스" 구성 객체를 반환할 수 있습니다.

```typescript
@@filename(config/database.config)
export default registerAs('database', () => ({
  host: process.env.DATABASE_HOST,
  port: process.env.DATABASE_PORT || 5432
}));
```

커스텀 구성 파일과 마찬가지로 `registerAs()` 팩토리 함수 내에서 `process.env` 객체는 완전히 해결된 환경변수 키/값 쌍을 포함합니다 ([위](/techniques/configuration#getting-started)에 설명된대로 해결 및 병합된 `.env` 파일 및 외부 정의 변수 포함).

> info **힌트** `registerAs` 함수는 `@nestjs/config` 패키지에서 내보냅니다.

커스텀 구성 파일을 로드하는 것과 동일한 방식으로 `forRoot()` 메소드 옵션 객체의 `load` 속성을 사용하여 네임스페이스 구성을 로드합니다.

```typescript
import databaseConfig from './config/database.config';

@Module({
  imports: [
    ConfigModule.forRoot({
      load: [databaseConfig],
    }),
  ],
})
export class AppModule {}
```

이제 `database` 네임스페이스에서 `host` 값을 가져 오려면 점 표기법(dot notation)을 사용하십시오. 네임 스페이스의 이름에 해당하는 속성 이름의 접두사로 `'database'`를 사용합니다 (`registerAs()` 함수의 첫번째 인수로 전달됨).

```typescript
const dbHost = this.configService.get<string>('database.host');
```

합리적인 대안은 `database` 네임스페이스를 직접 삽입하는 것입니다. 이를 통해 강력한 타이핑의 이점을 얻을 수 있습니다.

```typescript
constructor(
  @Inject(databaseConfig.KEY)
  private dbConfig: ConfigType<typeof databaseConfig>,
) {}
```

> info **힌트** `ConfigType`은 `@nestjs/config` 패키지에서 내보내집니다.

#### Cache environment variables

`process.env`에 대한 액세스 속도가 느릴 수 있으므로 `ConfigModule.forRoot()`에 전달된 옵션 객체의 `cache` 속성을 설정하여 `process.env`에 저장된 변수에 대해 `ConfigService#get` 메서드의 성능을 높일 수 있습니다.

```typescript
ConfigModule.forRoot({
  cache: true,
});
```

#### Partial registration

지금까지 `forRoot()` 메서드를 사용하여 루트 모듈(예: `AppModule`)에서 구성 파일을 처리했습니다. 여러 다른 디렉토리에 기능별 구성 파일이 있는 더 복잡한 프로젝트 구조가 있을 수 있습니다. 이러한 모든 파일을 루트 모듈에 로드하는 대신 `@nestjs/config`  패키지는 각 기능 모듈과 관련된 구성 파일만 참조하는 **부분 등록**이라는 기능을 제공합니다. 기능 모듈 내에서 `forFeature()` 정적 메서드를 사용하여 다음과 같이 부분 등록을 수행합니다.

```typescript
import databaseConfig from './config/database.config';

@Module({
  imports: [ConfigModule.forFeature(databaseConfig)],
})
export class DatabaseModule {}
```

> info **경고** 경우에 따라 생성자가 아닌 `onModuleInit()` 후크를 사용하여 부분 등록을 통해 로드된 속성에 액세스해야 할 수 있습니다. 모듈 초기화 과정에서 `forFeature()` 메서드가 실행되고 모듈 초기화 순서가 정해져 있지 않기 때문입니다. 다른 모듈에 의해 이러한 방식으로 로드된 값에 액세스하는 경우 생성자에서 구성이 의존하는 모듈이 아직 초기화되지 않았을 수 있습니다. `onModuleInit()`메서드는 종속된 모든 모듈이 초기화된 후에 만 실행되므로 이 기술은 안전합니다.

#### Schema validation

필수 환경 변수가 제공되지 않았거나 특정 유효성 검사 규칙을 충족하지 않는 경우 애플리케이션 시작 중에 예외를 던지는(throw) 것이 표준 관행입니다. `@nestjs/config` 패키지를 사용하면 두가지 방법으로 이를 수행할 수 있습니다.

- [Joi](https://github.com/sideway/joi) 내장 검사기. Joi를 사용하여 객체 스키마를 정의하고 이에 대해 JavaScript 객체의 유효성을 검사합니다.
- 환경 변수를 입력으로 받는 사용자 정의 `validate()` 함수.

Joi를 사용하려면 Joi 패키지를 설치해야합니다.

```bash
$ npm install --save joi
```

> warning **알림** 최신 버전의 `joi`를 사용하려면 Node v12 이상을 실행해야 합니다. 이전 버전의 노드의 경우 `v16.1.8`을 설치하십시오. 이는 주로 빌드 시간에 오류가 발생하는 `v17.0.2`출시 이후입니다. 자세한 내용은 [17.0.0 출시 노트](https://github.com/sideway/joi/issues/2262)를 참조하세요.

이제 Joi 유효성 검사 스키마를 정의하고 아래와 같이 `forRoot()` 메서드의 옵션 객체의 `validationSchema` 속성을 통해 전달할 수 있습니다.

```typescript
@@filename(app.module)
import * as Joi from 'joi';

@Module({
  imports: [
    ConfigModule.forRoot({
      validationSchema: Joi.object({
        NODE_ENV: Joi.string()
          .valid('development', 'production', 'test', 'provision')
          .default('development'),
        PORT: Joi.number().default(3000),
      }),
    }),
  ],
})
export class AppModule {}
```

기본적으로 모든 스키마 키는 선택사항으로 간주됩니다. 여기서는 환경 (`.env` 파일 또는 프로세스 환경)에서 이러한 변수를 제공하지 않을 경우 사용할 `NODE_ENV` 및 `PORT`에 대한 기본값을 설정합니다. 또는 `required()` 유효성 검사 메서드를 사용하여 환경 (`.env` 파일 또는 프로세스 환경)에서 값을 정의해야합니다. 이 경우 환경에서 변수를 제공하지 않으면 유효성 검사 단계에서 예외가 발생합니다. 유효성 검사 스키마를 구성하는 방법에 대한 자세한 내용은 [Joi 유효성 검사 방법](https://joi.dev/api/?v=17.3.0#example)을 참조하세요.

기본적으로 알 수 없는 환경변수 (스키마에 키가 없는 환경 변수)가 허용되며 유효성 검사 예외를 트리거하지 않습니다. 기본적으로 모든 유효성 검사 오류가 보고 됩니다. `forRoot()` 옵션 객체의 `validationOptions` 키를 통해 옵션 객체를 전달하여 이러한 동작을 변경할 수 있습니다. 이 옵션 개체는 [Joi 유효성 검사 옵션](https://joi.dev/api/?v=17.3.0#anyvalidatevalue-options)에서 제공하는 표준 유효성 검사 옵션 속성을 포함할 수 있습니다. 예를 들어 위의 두 설정을 되돌리려면 다음과 같은 옵션을 전달합니다.

```typescript
@@filename(app.module)
import * as Joi from 'joi';

@Module({
  imports: [
    ConfigModule.forRoot({
      validationSchema: Joi.object({
        NODE_ENV: Joi.string()
          .valid('development', 'production', 'test', 'provision')
          .default('development'),
        PORT: Joi.number().default(3000),
      }),
      validationOptions: {
        allowUnknown: false,
        abortEarly: true,
      },
    }),
  ],
})
export class AppModule {}
```

`@nestjs/config` 패키지는 다음의 기본 설정을 사용합니다.

- `allowUnknown`: 환경변수에서 알 수 없는 키를 허용할지 여부를 제어합니다. 기본값은 `true`입니다.
- `abortEarly`: `true`이면 첫번째 오류에서 유효성 검사를 중지합니다. `false`이면 모든 오류를 반환합니다. 기본값은 `false`입니다.

일단 `validationOptions` 객체를 전달하기로 결정하면 명시적으로 전달하지 않은 모든 설정은 `Joi` 표준 기본값 (`@nestjs/config` 기본값이 아님)으로 기본 설정됩니다. 예를 들어 사용자 정의 `validationOptions` 객체에서 `allowUnknowns`를 지정하지 않은 상태로 두면 `Joi` 기본값인 `false`가 됩니다. 따라서 사용자 지정 객체에서 이러한 설정을 **둘 다** 지정하는 것이 가장 안전할 수 있습니다.

#### Custom validate function

또는 환경변수 (env 파일 및 프로세스에서)를 포함하는 객체를 가져와 필요한 경우 변환/변경할 수 있도록 검증된 환경변수가 포함된 객체를 반환하는 **동기** `validate` 함수를 지정할 수 있습니다. 함수에서 오류가 발생하면 애플리케이션이 부트스트랩되지 않습니다.

이 예에서는 `class-transformer` 및 `class-validator` 패키지를 진행합니다. 먼저 다음을 정의해야 합니다.

- 유효성 검사 제약이 있는 클래스,
- `plainToClass` 및 `validateSync` 함수를 사용하는 유효성 검사 함수.

```typescript
@@filename(env.validation)
import { plainToClass } from 'class-transformer';
import { IsEnum, IsNumber, validateSync } from 'class-validator';

enum Environment {
  Development = "development",
  Production = "production",
  Test = "test",
  Provision = "provision",
}

class EnvironmentVariables {
  @IsEnum(Environment)
  NODE_ENV: Environment;

  @IsNumber()
  PORT: number;
}

export function validate(config: Record<string, unknown>) {
  const validatedConfig = plainToClass(
    EnvironmentVariables,
    config,
    { enableImplicitConversion: true },
  );
  const errors = validateSync(validatedConfig, { skipMissingProperties: false });

  if (errors.length > 0) {
    throw new Error(errors.toString());
  }
  return validatedConfig;
}
```

이 상태에서 다음과 같이 `ConfigModule`의 구성 옵션으로 `validate` 함수를 사용합니다.

```typescript
@@filename(app.module)
import { validate } from './env.validation';

@Module({
  imports: [
    ConfigModule.forRoot({
      validate,
    }),
  ],
})
export class AppModule {}
```

<app-banner-shop></app-banner-shop>

#### Custom getter functions

`ConfigService`는 키로 구성값을 검색하는 일반적인 `get()` 메소드를 정의합니다. 좀 더 자연스러운 코딩 스타일을 사용하기 위해 `getter` 함수를 추가할 수도 있습니다.

```typescript
@@filename()
@Injectable()
export class ApiConfigService {
  constructor(private configService: ConfigService) {}

  get isAuthEnabled(): boolean {
    return this.configService.get('AUTH_ENABLED') === 'true';
  }
}
@@switch
@Dependencies(ConfigService)
@Injectable()
export class ApiConfigService {
  constructor(configService) {
    this.configService = configService;
  }

  get isAuthEnabled() {
    return this.configService.get('AUTH_ENABLED') === 'true';
  }
}
```

이제 다음과 같이 getter 함수를 사용할 수 있습니다.

```typescript
@@filename(app.service)
@Injectable()
export class AppService {
  constructor(apiConfigService: ApiConfigService) {
    if (apiConfigService.isAuthEnabled) {
      // Authentication is enabled
    }
  }
}
@@switch
@Dependencies(ApiConfigService)
@Injectable()
export class AppService {
  constructor(apiConfigService) {
    if (apiConfigService.isAuthEnabled) {
      // Authentication is enabled
    }
  }
}
```

#### Expandable variables

`@nestjs/config` 패키지는 환경변수 확장을 지원합니다. 이 기술을 사용하면 한 변수가 다른 정의내에서 참조되는 중첩 환경변수를 만들 수 있습니다. 예를 들면:

```json
APP_URL=mywebsite.com
SUPPORT_EMAIL=support@${APP_URL}
```

이 구성에서 `SUPPORT_EMAIL` 변수는 `'support@mywebsite.com'`으로 해석됩니다. `${{ '{' }}...{{ '}' }}` 구문을 사용하여 `SUPPORT_EMAIL` 정의내에서 `APP_URL` 변수의 값을 확인합니다.

> info **힌트** 이 기능을 위해 `@nestjs/config` 패키지는 내부적으로 [dotenv-expand](https://github.com/motdotla/dotenv-expand)를 사용합니다.

아래와 같이 `ConfigModule`의 `forRoot()` 메소드에 전달된 옵션 객체의 `expandVariables` 속성을 사용하여 환경변수 확장을 활성화합니다.

```typescript
@@filename(app.module)
@Module({
  imports: [
    ConfigModule.forRoot({
      // ...
      expandVariables: true,
    }),
  ],
})
export class AppModule {}
```

#### Using in the `main.ts`

우리의 설정은 서비스에 저장되지만 `main.ts` 파일에서 계속 사용할 수 있습니다. 이런식으로 애플리케이션 포트 또는 CORS 호스트와 같은 변수를 저장하는 데 사용할 수 있습니다.

액세스하려면 `app.get()` 메소드와 서비스 참조를 사용해야합니다.

```typescript
const configService = app.get(ConfigService);
```

그런 다음 구성 키로 `get` 메소드를 호출하여 평소처럼 사용할 수 있습니다.

```typescript
const port = configService.get('PORT');
```
