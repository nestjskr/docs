### Common errors

NestJS로 개발하는 동안 프레임워크를 배우면서 다양한 오류가 발생할 수 있습니다.

#### "Cannot resolve dependency" error

아마도 가장 일반적인 오류 메시지는 Nest가 프로바이더의 종속성을 해결할 수 없다는 것입니다. 오류 메시지는 일반적으로 다음과 같습니다.

```bash
Nest는 <provider>(?)의 종속성을 확인할 수 없습니다. 인덱스 [<index>]의 <unknown_token> 인수가 <module> 컨텍스트에서 사용 가능한지 확인하십시오.

잠재적인 해결책 :
- <unknown_token>이 프로바이더인 경우 현재 <module>의 일부입니까?
- 별도의 @Module에서 <unknown_token>을 내보내면 해당 모듈을 <module>내에서 가져오나요?
  @Module({
    imports: [ /* <unknown_token>을 포함하는 모듈 */ ]
  })
```


오류의 가장 일반적인 원인은 모듈의 `providers` 배열에 `provider`가 없는 것입니다. 프로바이더가 실제로 `providers` 배열에 있고 [표준 NestJS 프로바이더 관행](/fundamentals/custom-providers#di-fundamentals)을 따르는 지 확인하세요.

일반적인 몇가지 문제가 있습니다. 하나는 프로바이더를 `imports` 배열에 넣는 것입니다. 이 경우 오류에는 `<module>`이 있어야하는 프로바이더 이름이 표시됩니다.

개발중에 이 오류가 발생하면 오류 메시지에 언급된 모듈을 살펴보고 `providers`를 살펴보십시오. `providers` 배열의 각 프로바이더에 대해 모듈이 모든 종속성에 액세스할 수 있는지 확인합니다. 종종 `providers`는 "기능 모듈"과 "루트 모듈"에 중복됩니다. 즉, Nest가 프로바이더를 두번 인스턴스화하려고 시도합니다. 중복되는 `provider`를 포함하는 모듈은 대신 "루트 모듈"의 `imports` 배열에 추가되어야 합니다.

#### "Circular dependency" error

때때로 애플리케이션에서 [순환 종속성](/fundamentals/circular-dependency)을 피하는 것이 어려울 수 있습니다. Nest가 이 문제를 해결하는 데 도움이 되도록 몇가지 조치를 취해야합니다. 순환 종속성에서 발생하는 오류는 다음과 같습니다.

```bash
Nest는 <module> 인스턴스를 만들 수 없습니다.
<module> "imports" 배열의 색인 [<index>]에 있는 모듈이 정의되지 않았습니다.

잠재적 원인:
- 모듈간의 순환 종속성. 그것을 피하려면 forwardRef()를 사용하십시오. 자세히보기: /fundamentals/circular-dependency
- 색인 [<index>]의 모듈이 "undefined" 타입입니다. import 문과 모듈 타입을 확인하십시오.

Scope [<module_import_chain>]
# example chain AppModule -> FooModule
```

순환 종속성은 서로 종속된 두 프로바이더 또는 모듈 파일에서 상수를 내보내고 서비스 파일로 가져오는 것과 같이 상수에 대해 서로 종속된 typescript 파일에서 발생할 수 있습니다. 후자의 경우 상수에 대해 별도의 파일을 만드는 것이 좋습니다. 전자의 경우 순환 종속성에 대한 가이드를 따르고 모듈 **과** 프로바이더 모두 `forwardRef`로 표시되어 있는지 확인하세요.
