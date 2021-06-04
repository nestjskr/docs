### Hot Reload

애플리케이션의 부트스트랩 프로세스에 가장 큰 영향을 미치는 것은 **TypeScript 컴파일**입니다. 다행히 [webpack](https://github.com/webpack/webpack) HMR(Hot-Module Replacement)을 사용하면 변경이 발생할 때마다 전체 프로젝트를 다시 컴파일 할 필요가 없습니다. 이렇게하면 애플리케이션을 인스턴스화하는데 필요한 시간이 크게 줄어들고 반복 개발이 훨씬 쉬워집니다.

> warning **경고** `webpack`은 자산(예: `graphql` 파일)을 `dist` 폴더에 자동으로 복사하지 않습니다. 마찬가지로 `webpack`은 glob 정적 경로(예: `TypeOrmModule`의 `entities` 속성)와 호환되지 않습니다.

### With CLI

[Nest CLI](/cli/overview)를 사용하는 경우 구성 프로세스가 매우 간단합니다. CLI는 `HotModuleReplacementPlugin`의 사용을 허용하는 `webpack`을 래핑합니다.

#### Installation

먼저 필요한 패키지를 설치하십시오.

```bash
$ npm i --save-dev webpack-node-externals run-script-webpack-plugin webpack
```

> info **힌트** **Yarn Berry** (기존 Yarn 아님)를 사용하는 경우 `webpack-node-externals` 대신 `webpack-pnp-externals` 패키지를 설치하세요.

#### Configuration

설치가 완료되면 애플리케이션의 루트 디렉터리에v`webpack-hmr.config.js` 파일을 만듭니다.

```typescript
const nodeExternals = require('webpack-node-externals');
const { RunScriptWebpackPlugin } = require('run-script-webpack-plugin');

module.exports = function (options, webpack) {
  return {
    ...options,
    entry: ['webpack/hot/poll?100', options.entry],
    externals: [
      nodeExternals({
        allowlist: ['webpack/hot/poll?100'],
      }),
    ],
    plugins: [
      ...options.plugins,
      new webpack.HotModuleReplacementPlugin(),
      new webpack.WatchIgnorePlugin({
        paths: [/\.js$/, /\.d\.ts$/],
      }),
      new RunScriptWebpackPlugin({ name: options.output.filename }),
    ],
  };
};
```

> info **힌트** **Yarn Berry**(기존 Yarn 아님)를 사용하면 `externals` 구성 속성에서` nodeExternals`를 사용하는 대신 `webpack-pnp-externals` 패키지의 `WebpackPnpExternals({{ '{' }} exclude: ['webpack/hot/poll?100'] {{ '}' }})`.

이 함수는 기본 웹팩 구성이 포함된 원본 객체를 첫번째 인수로, Nest CLI에서 사용하는 기본 `webpack` 패키지에 대한 참조를 두번째 인수로 사용합니다. 또한 `HotModuleReplacementPlugin`, `WatchIgnorePlugin` 및 `RunScriptWebpackPlugin` 플러그인으로 수정된 웹팩 구성을 반환합니다.

#### Hot-Module Replacement

**HMR**을 활성화하려면 애플리케이션 항목 파일(`main.ts`)을 열고 다음 웹팩 관련 지침을 추가합니다.

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

실행 프로세스를 단순화하려면 `package.json` 파일에 스크립트를 추가하십시오.

```json
"start:dev": "nest build --webpack --webpackPath webpack-hmr.config.js --watch"
```

이제 명령 줄을 열고 다음 명령을 실행하기 만하면됩니다.

```bash
$ npm run start:dev
```

### Without CLI

[Nest CLI](/cli/overview)를 사용하지 않는 경우 구성이 약간 더 복잡해집니다(더 많은 수동 단계가 필요함).

#### Installation

먼저 필요한 패키지를 설치하십시오.

```bash
$ npm i --save-dev webpack webpack-cli webpack-node-externals ts-loader run-script-webpack-plugin
```

> info **힌트** **Yarn Berry**(기존 Yarn 아님)를 사용하는 경우 `webpack-node-externals` 대신 `webpack-pnp-externals` 패키지를 설치하세요.

#### Configuration

설치가 완료되면 애플리케이션의 루트 디렉토리에 `webpack.config.js` 파일을 만듭니다.

##### With NPM or Yarn classic

```typescript
const webpack = require('webpack');
const path = require('path');
const nodeExternals = require('webpack-node-externals');
const { RunScriptWebpackPlugin } = require('run-script-webpack-plugin');

module.exports = {
  entry: ['webpack/hot/poll?100', './src/main.ts'],
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
    new RunScriptWebpackPlugin({ name: 'server.js' }),
  ],
  output: {
    path: path.join(__dirname, 'dist'),
    filename: 'server.js',
  },
};
```

> info **힌트** **Yarn Berry**(기존 Yarn 아님)를 사용하면` externals` 구성 속성에서 `nodeExternals`를 사용하는 대신 `webpack-pnp-externals` 패키지의 `WebpackPnpExternals`를 사용합니다. `WebpackPnpExternals({{ '{' }} exclude: ['webpack/hot/poll?100'] {{ '}' }})`.

이 구성은 웹팩에 애플리케이션에 대한 몇가지 필수 사항을 알려줍니다. 엔트리 파일의 위치, **컴파일 된** 파일을 보관하는 데 사용해야하는 디렉토리, 소스 파일을 컴파일하는데 사용할 로더의 종류. 일반적으로 모든 옵션을 완전히 이해하지 못하더라도 이 파일을 있는 그대로 사용할 수 있어야 합니다.

#### Hot-Module Replacement

**HMR**을 활성화하려면 애플리케이션 항목 파일(`main.ts`)을 열고 다음 웹팩 관련 지침을 추가합니다.

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

실행 프로세스를 단순화하려면 `package.json` 파일에 스크립트를 추가하십시오.

```json
"start:dev": "webpack --config webpack.config.js --watch"
```


```bash
$ npm run start:dev
```

#### Example

작동하는 예제는 [여기](https://github.com/nestjs/nest/tree/master/sample/08-webpack)에서 확인할 수 있습니다.

### TypeORM

`@nestjs/typeorm`을 사용하는 경우 TypeORM 구성에 `keepConnectionAlive: true`를 추가해야합니다.
