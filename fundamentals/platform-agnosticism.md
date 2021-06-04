### Platform agnosticism

Nest는 플랫폼에 구애받지 않는 프레임워크입니다. 즉, 다양한 유형의 애플리케이션에서 사용할 수 있는 **재사용 가능한 논리적 부분**을 개발할 수 있습니다. 예를 들어, 대부분의 구성 요소는 다양한 기본 HTTP 서버 프레임워크(예: Express 및 Fastify)에서 변경없이 재사용될 수 있으며 다른 _타입_ 애플리케이션(예: HTTP 서버 프레임워크, 다른 전송 레이어가 있는 마이크로서비스 및 웹소켓)에서도 재사용될 수 있습니다.

#### Build once, use everywhere

문서의 **개요** 섹션은 주로 HTTP 서버 프레임워크를 사용하는 코딩 기술을 보여줍니다(예: REST API를 제공하거나 MVC 스타일의 서버측 렌더링 앱을 제공하는 앱). 그러나 이러한 모든 빌딩 블록은 서로 다른 전송 계층([마이크로서비스](/microservices/basics) 또는 [웹소켓](/websockets/gateways)) 위에서 사용할 수 있습니다.

또한 Nest는 전용 [GraphQL](/graphql/quick-start) 모듈과 함께 제공됩니다. REST API를 제공하는 방식으로 GraphQL을 API 계층으로 사용할 수 있습니다.

또한 [애플리케이션 컨텍스트](/application-context) 기능은 Nest 위에 CRON 작업 및 CLI 앱과 같은 것을 포함하여 모든 종류의 Node.js 애플리케이션을 만드는 데 도움이됩니다.

Nest는 애플리케이션에 더 높은 수준의 모듈성과 재사용성을 제공하는 Node.js 앱을 위한 본격적인 플랫폼이 되고자 합니다. 한번만 빌드하면 어디서나 사용하세요!
