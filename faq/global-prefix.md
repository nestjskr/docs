### Global prefix

HTTP 애플리케이션에 등록된 **모든 라우트**에 대한 접두사를 설정하려면 `INestApplication` 인스턴스의 `setGlobalPrefix()` 메서드를 사용하세요.

```typescript
const app = await NestFactory.create(AppModule);
app.setGlobalPrefix('v1');
```
