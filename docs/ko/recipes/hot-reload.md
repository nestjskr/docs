### Hot Reload(빠른 재시작)

프로그램 시작 시간에 가장 큰 영향을 미치는 건 **타입스크립트 컴파일**입니다. 다행히도, [웹팩](https://github.com/webpack/webpack)의 HMR (Hot-Module Replacement)과 함께라면 우린 코드의 변경사항이 있을 때마다 프로젝트 전체를 다시 컴파일할 필요가 없습니다. 이는 프로그램을 시작하는 데 드는 시간을 상당히 줄여주기 때문에 변경을 빠르게 확인하며 개발하기 훨씬 쉽게 해줍니다.

> **경고** `웹팩`이 `graphql` 파일과 같은 에셋 파일들을 자동으로 `dist` 폴더로 복사해주지 않는다는걸 알아두세요. 또한, `웹팩`은 글로벌 경로와 호환되지 않습니다(예를 들어, `TypeOrmModule`의 `엔티티`)

### Nest CLI를 사용하여 시작

만약 [Nest CLI](https://docs.nestjs.com/cli/overview)을 사용중이라면, CLI가 `웹팩`의 `HotModuleReplacementPlugin`을 사용하기 쉽게 추상화하였기 때문에 Hot Reload를 사용하기 위한 설정이 꽤나 쉬울것입니다.

#### 설치

먼저, 필수 패키지들을 설치해주세요.

```bash
$ npm i --save-dev webpack-node-externals start-server-webpack-plugin
```

#### 설정

설치가 완료되면, `webpack-hmr.config.js` 파일을 프로젝트 루트 디렉터리에 만들어주세요.

```typescript
const webpack = require('webpack');
const nodeExternals = require('webpack-node-externals');
const StartServerPlugin = require('start-server-webpack-plugin');

module.exports = function(options) {
  return {
    ...options,
    entry: ['webpack/hot/poll?100', options.entry],
    watch: true,
    externals: [
      nodeExternals({
        allowlist: ['webpack/hot/poll?100'],
      }),
    ],
    plugins: [
      ...options.plugins,
      new webpack.HotModuleReplacementPlugin(),
      new webpack.WatchIgnorePlugin([/\.js$/, /\.d\.ts$/]),
      new StartServerPlugin({ name: options.output.filename }),
    ],
  };
};
```

이 함수는 기본 웹팩 설정 옵션을 담고있는 객체를 넘겨받아 여기에`HotModuleReplacementPlugin` 플러그인이 적용되어 수정된 객체를 리턴합니다.

#### Hot-Module Replacement(빠른 모듈 대체)

**HMR**을 활성화하려면, 프로젝트 엔트리 파일(`main.ts`)을 열고 다음과같이 웹팩 관련 코드를 추가해주세요.

```typescript
declare const module: any;

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);

  if (module.hot) {
    module.hot.accept();
    module.hot.dispose(() => app.close());
  }
}
bootstrap();
```

실행 명령어를 매번 입력하기 어려우므로`package.json` 파일에 스크립트를 추가해주세요.

```json
"start:dev": "nest build --webpack --webpackPath webpack-hmr.config.js"
```

이제 커맨드라인을 열고 다음 명령어를 실행하세요.

```bash
$ npm run start:dev
```

### Nest CLI 없이 시작

만약 [Nest CLI](https://docs.nestjs.com/cli/overview)를 사용하지 않는다면, 설정은 조금 더 복잡해집니다(수동으로 해주어야 하는 단계들이 필요함).

#### 설치

먼저, 필수 패키지들을 설치해주세요.

```bash
$ npm i --save-dev webpack webpack-cli webpack-node-externals ts-loader start-server-webpack-plugin
```

#### 설정

설치가 완료되면 프로젝트 루트 디렉터리에서 `webpack.config.js` 파일을 만들어주세요.

```typescript
const webpack = require('webpack');
const path = require('path');
const nodeExternals = require('webpack-node-externals');
const StartServerPlugin = require('start-server-webpack-plugin');

module.exports = {
  entry: ['webpack/hot/poll?100', './src/main.ts'],
  watch: true,
  target: 'node',
  externals: [
    nodeExternals({
      allowlist: ['webpack/hot/poll?100'],
    }),
  ],
  module: {
    rules: [
      {
        test: /.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
    ],
  },
  mode: 'development',
  resolve: {
    extensions: ['.tsx', '.ts', '.js'],
  },
  plugins: [
    new webpack.HotModuleReplacementPlugin(),
    new StartServerPlugin({ name: 'server.js' }),
  ],
  output: {
    path: path.join(__dirname, 'dist'),
    filename: 'server.js',
  },
};
```

이 설정은 웹팩에게 당신의 프로젝트에 대해 다음과 같은 몇가지 필수적인 정보를 전달합니다. 전달되는 정보는 프로젝트 시작 파일의 위치, 어떤 디렉터리에서 **컴파일**된 파일들을 갖고있어야 하는지, 어떤 로더를 사용하여 소스를 컴파일할지입니다. 일반적으로, 모든 옵션들에 대해 알지 못하더라도 위에 나와있는 예시를 가지고 그대로 사용하여도 무방합니다.

#### Hot-Module Replacement(빠른 모듈 대체)

**HMR**을 활성화하려면, 프로젝트 시작 파일(`main.ts`)을 열고 다음과 같이 웹팩 관련 코드를 삽입하세요.

```typescript
declare const module: any;

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);

  if (module.hot) {
    module.hot.accept();
    module.hot.dispose(() => app.close());
  }
}
bootstrap();
```

실행 명령어를 매번 입력하기 어려우므로`package.json` 파일에 스크립트를 추가해주세요.

```json
"start:dev": "webpack --config webpack.config.js"
```

이제 커맨드라인을 열고 다음 명령어를 실행하세요.

```bash
$ npm run start:dev
```

작동이 확인된 예시는 [여기](https://github.com/nestjs/nest/tree/master/sample/08-webpack)에서 확인할 수 있습니다.

### TypeORM

만약 `@nestjs/typeorm`을 사용중이라면, `keepConnectionAlive: true`옵션을 TypeORM 설정에 추가하여야합니다.
