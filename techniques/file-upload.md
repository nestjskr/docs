### File upload

파일 업로드를 처리하기 위해 Nest는 Express용 [multer](https://github.com/expressjs/multer) 미들웨어 패키지를 기반으로하는 내장 모듈을 제공합니다. Multer는 주로 HTTP `POST` 요청을 통해 파일을 업로드하는 데 사용되는 `multipart/form-data` 형식으로 게시된 데이터를 처리합니다. 이 모듈은 완전히 구성할 수 있으며 애플리케이션 요구사항에 맞게 동작을 조정할 수 있습니다.

> warning **경고** Multer는 지원되는 멀티 파트 형식(`multipart/form-data`)이 아닌 데이터를 처리할 수 없습니다. 또한 이 패키지는 `FastifyAdapter`와 호환되지 않습니다.

더 나은 타입안전을 위해 Multer 타이핑 패키지를 설치하겠습니다.

```shell
$ npm i -D @types/multer
```

이 패키지가 설치되면 이제 `Express.Multer.File` 타입을 사용할 수 있습니다 (이 타입은 다음과 같이 가져올 수 있습니다: `import {{ '{' }} Express {{ '}' }} from 'express'`). .

#### Basic example

단일 파일을 업로드하려면 `FileInterceptor()` 인터셉터를 라우트 핸들러에 연결하고 `@UploadedFile()` 데코레이터를 사용하여 `request`에서 `file`을 추출하면 됩니다.

```typescript
@@filename()
@Post('upload')
@UseInterceptors(FileInterceptor('file'))
uploadFile(@UploadedFile() file: Express.Multer.File) {
  console.log(file);
}
@@switch
@Post('upload')
@UseInterceptors(FileInterceptor('file'))
@Bind(UploadedFile())
uploadFile(file) {
  console.log(file);
}
```

> info **힌트** `FileInterceptor()` 데코레이터는 `@nestjs/platform-express` 패키지에서 내보냅니다.  `@UploadedFile()` 데코레이터는 `@nestjs/common`에서 내보내집니다.

The `FileInterceptor()` decorator takes two arguments:
`FileInterceptor()` 데코레이터는 두개의 인수를 받습니다.

- `fieldName`: 파일이 있는 HTML 양식에서 필드 이름을 제공하는 문자열
- `options`: `MulterOptions` 타입의 선택적 객체. 이것은 multer 생성자에서 사용하는 것과 동일한 객체입니다(자세한 내용은 [여기](https://github.com/expressjs/multer#multeropts)).

> warning **경고** `FileInterceptor()`는 Google Firebase 등의 타사 클라우드 제공업체와 호환되지 않을 수 있습니다.

#### Array of files

파일 배열(단일 필드 이름으로 식별됨)을 업로드하려면 `FilesInterceptor()` 데코레이터를 사용합니다(데코레이터 이름에 **Files**라고 복수 있음). 이 데코레이터는 세가지 인수를받습니다.

- `fieldName`: 위에서 설명한대로
- `maxCount`: 허용할 최대 파일 수를 정의하는 선택적 숫자
- `options`: 위에서 설명한대로 선택적 `MulterOptions` 객체

`FilesInterceptor()`를 사용하는 경우 `@UploadedFiles()` 데코레이터를 사용하여 `request`에서 파일을 추출합니다.

```typescript
@@filename()
@Post('upload')
@UseInterceptors(FilesInterceptor('files'))
uploadFile(@UploadedFiles() files: Array<Express.Multer.File>) {
  console.log(files);
}
@@switch
@Post('upload')
@UseInterceptors(FilesInterceptor('files'))
@Bind(UploadedFiles())
uploadFile(files) {
  console.log(files);
}
```

> info **힌트** `FilesInterceptor()` 데코레이터는 `@nestjs/platform-express` 패키지에서 내보냅니다. `@UploadedFiles()` 데코레이터는 `@nestjs/common`에서 내보내집니다.

#### Multiple files

여러 필드를 업로드하려면(모두 다른 필드이름 키를 사용) `FileFieldsInterceptor()` 데코레이터를 사용합니다. 이 데코레이터는 두 가지 인수를 취합니다.

- `uploadedFields`: 객체의 배열. 여기서 각 객체는 위에서 설명한대로 필드 이름을 지정하는 문자열 값으로 필수 `name` 속성을 지정하고 위에서 설명한대로 선택적 `maxCount` 속성을 지정합니다.
- `options`: 위에서 설명한대로 선택적 `MulterOptions` 객체

`FileFieldsInterceptor()`를 사용할 때 `@UploadedFiles()` 데코레이터를 사용하여 `request`에서 파일을 추출합니다.

```typescript
@@filename()
@Post('upload')
@UseInterceptors(FileFieldsInterceptor([
  { name: 'avatar', maxCount: 1 },
  { name: 'background', maxCount: 1 },
]))
uploadFile(@UploadedFiles() files) {
  console.log(files);
}
@@switch
@Post('upload')
@Bind(UploadedFiles())
@UseInterceptors(FileFieldsInterceptor([
  { name: 'avatar', maxCount: 1 },
  { name: 'background', maxCount: 1 },
]))
uploadFile(files) {
  console.log(files);
}
```

#### Any files

임의의 필드 이름 키가 있는 모든 필드를 업로드하려면 `AnyFilesInterceptor()` 데코레이터를 사용하세요. 이 데코레이터는 위에서 설명한대로 선택적 `options` 객체를 허용할 수 있습니다.

`AnyFilesInterceptor()`를 사용하는 경우 `@UploadedFiles()` 데코레이터를 사용하여 `request`에서 파일을 추출합니다.

```typescript
@@filename()
@Post('upload')
@UseInterceptors(AnyFilesInterceptor())
uploadFile(@UploadedFiles() files: Array<Express.Multer.File>) {
  console.log(files);
}
@@switch
@Post('upload')
@Bind(UploadedFiles())
@UseInterceptors(AnyFilesInterceptor())
uploadFile(files) {
  console.log(files);
}
```

#### Default options

위에서 설명한대로 파일 인터셉터에서 멀터 옵션을 지정할 수 있습니다. 기본 옵션을 설정하려면 `MulterModule`을 가져올 때 정적 `register()` 메서드를 호출하여 지원되는 옵션을 전달할 수 있습니다. [여기](https://github.com/expressjs/multer#multeropts)에 나열된 모든 옵션을 사용할 수 있습니다.

```typescript
MulterModule.register({
  dest: './upload',
});
```

> info **힌트** `MulterModule` 클래스는 `@nestjs/platform-express` 패키지에서 내보내집니다.

#### Async configuration

`MulterModule` 옵션을 정적으로 설정하는 대신 비동기적으로 설정해야하는 경우 `registerAsync()` 메서드를 사용하세요. 대부분의 동적 모듈과 마찬가지로 Nest는 비동기 구성을 처리하는 몇가지 기술을 제공합니다.

한가지 기술은 팩토리 기능을 사용하는 것입니다.

```typescript
MulterModule.registerAsync({
  useFactory: () => ({
    dest: './upload',
  }),
});
```

다른 [팩토리 프로바이더](/fundamentals/custom-providers#factory-providers-usefactory)와 마찬가지로 팩토리 함수는 `async`일 수 있으며 `inject`을 통해 종속성을 삽입할 수 있습니다.

```typescript
MulterModule.registerAsync({
  imports: [ConfigModule],
  useFactory: async (configService: ConfigService) => ({
    dest: configService.getString('MULTER_DEST'),
  }),
  inject: [ConfigService],
});
```

또는 아래와 같이 팩토리 대신 클래스를 사용하여 `MulterModule`을 구성할 수 있습니다.

```typescript
MulterModule.registerAsync({
  useClass: MulterConfigService,
});
```

위의 구성은 `MulterModule`내에서 `MulterConfigService`를 인스턴스화하여 필요한 옵션 객체를 만드는 데 사용합니다. 이 예에서 `MulterConfigService`는 아래와 같이 `MulterOptionsFactory` 인터페이스를 구현해야합니다. `MulterModule`은 제공된 클래스의 인스턴스화된 객체에서 `createMulterOptions()` 메소드를 호출합니다.

```typescript
@Injectable()
class MulterConfigService implements MulterOptionsFactory {
  createMulterOptions(): MulterModuleOptions {
    return {
      dest: './upload',
    };
  }
}
```

`MulterModule` 내부에 비공개 사본을 만드는 대신 기존 옵션 프로바이더를 재사용하려면 `useExisting`구문을 사용하세요.

```typescript
MulterModule.registerAsync({
  imports: [ConfigModule],
  useExisting: ConfigService,
});
```

#### Example

작동하는 예는 [여기](https://github.com/nestjs/nest/tree/master/sample/29-file-upload)에서 확인할 수 있습니다.
