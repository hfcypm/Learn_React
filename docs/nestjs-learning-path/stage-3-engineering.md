# 阶段三：NestJS 工程化与测试

**目标**：建立可配置、可测试、可文档化的 NestJS 工程。

## 1. 配置管理

配置模块负责读取环境变量、默认值和运行环境校验。敏感配置由部署平台注入，仓库只保存 `.env.example` 占位符。配置按模块分组，避免业务代码到处读取 `process.env`。

`ConfigModule.forRoot({ isGlobal: true })` 可以提供全局配置服务；更严格的项目应在启动时使用 schema 或自定义工厂校验变量，缺少数据库地址、签名密钥等关键配置时直接阻止启动。

## 2. 单元测试

Nest Testing 模块可以创建 TestingModule，并通过 `overrideProvider` 替换依赖：

```ts
const moduleRef = await Test.createTestingModule({
  providers: [UsersService, UsersRepository],
})
  .overrideProvider(UsersRepository)
  .useValue({ findById: jest.fn() })
  .compile();
```

单元测试关注 Service 和 Guard 的业务规则；集成测试关注模块与数据库、缓存、消息系统的协作；E2E 测试覆盖真实 HTTP 流程。

## 3. OpenAPI 与文档

DTO、响应模型、认证方式和错误响应需要同步到 OpenAPI。文档既服务前端和测试，也用于发现接口边界不清、状态码不一致和字段缺失。

OpenAPI 装饰器描述 DTO、响应、状态码和鉴权要求。生成文档时避免把内部 Entity 当作公共响应模型，并为分页、错误和空结果建立统一约定。

## 4. 日志、缓存和任务

- 日志记录结构化字段、request ID、版本和耗时。
- CacheModule 用于明确的读取缓存，必须设计 TTL 和失效策略。
- ScheduleModule 适合定时任务，重复执行需要幂等。
- BullMQ 等队列适合异步任务，消费者需要重试、死信和监控。

## 5. 阶段验收

- 能为核心 Service、Guard、Pipe 和 Controller 编写测试。
- 能使用配置模块校验环境变量。
- 能生成包含认证和错误模型的 OpenAPI 文档。
- 能为缓存、定时任务和队列设计幂等与失败恢复。
- 能在启动阶段拒绝无效配置，并维护与接口实现一致的 OpenAPI 文档。

## 动手任务

1. 为数据库地址和签名密钥增加启动时配置校验。
2. 使用 `TestingModule` 替换 Repository，测试 Service 和 Guard。
3. 为用户 API 生成 OpenAPI，并补充认证、分页和错误响应模型。
4. 为一个异步任务增加幂等键、重试上限和失败记录。

## 进入下一阶段的条件

你能够从测试模块追踪 Provider 替换过程，能够通过 OpenAPI 发现接口缺失，并能说明队列重试与 HTTP 重试的差异。
