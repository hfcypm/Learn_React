# 阶段二：NestJS 请求处理与数据层

**目标**：掌握请求生命周期扩展点，并建立安全、类型化的数据访问层。

## 1. 请求生命周期

典型顺序为 Middleware、Guards、Interceptors 的前置逻辑、Pipes、Controller/Service、Interceptors 的后置逻辑，异常由 Exception Filters 处理。全局、Controller 和路由级绑定会影响执行范围。

## 2. Pipes：转换与校验

Pipe 负责把输入转换为目标类型或拒绝非法输入。生产 API 通常启用全局 `ValidationPipe`，并配置白名单、转换和拒绝未知字段策略。

```ts
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,
  forbidNonWhitelisted: true,
  transform: true,
}));
```

典型 DTO 使用 `class-validator` 描述规则，`class-transformer` 负责转换：

```ts
import { IsEmail, IsString, MinLength } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @MinLength(2)
  name!: string;

  @IsEmail()
  email!: string;
}
```

DTO 描述输入结构，运行时校验库负责真正验证数据。DTO 与数据库 Entity 需要保持边界，避免直接把持久化模型暴露给 API。

## 3. Guards：认证与授权

Guard 决定请求是否可以进入路由。认证 Guard 解析身份，授权 Guard 检查角色、权限、租户和资源归属。装饰器只声明元数据，决策逻辑集中在 Guard 或策略服务。

## 4. Interceptors：横切能力

Interceptor 可以在处理器前后执行逻辑，适合日志、计时、响应转换、缓存和超时。事务和重试需要明确边界，避免拦截器隐藏关键业务行为。

## 5. Exception Filters

Filter 将异常转换为统一 HTTP 响应，并记录 request ID、错误码和内部原因。生产响应不应直接暴露堆栈、SQL 或第三方错误细节。

## 6. 数据层

NestJS 可以集成 TypeORM、Prisma、Mongoose 等数据工具。Repository 负责查询和映射，Service 负责业务事务。数据库连接、迁移、索引、分页、事务和 N+1 查询需要单独治理。

Controller 返回的 DTO 应与 Entity 或 ORM Model 分离。响应 DTO 只暴露允许公开的字段，密码哈希、内部状态和审计字段需要在映射层剔除。

## 7. 其他传输层

- WebSocket 适合实时双向通信，需要连接认证、房间权限和断线重连策略。
- GraphQL 适合由客户端选择字段的 API，需要治理 schema、复杂查询和 N+1。
- 微服务 Transport 适合服务间消息或 RPC，需要设计超时、重试、幂等和消息版本。

## 阶段验收

- 能实现 DTO 校验、认证 Guard、统一错误 Filter。
- 能解释 Middleware、Guard、Interceptor、Pipe 的边界。
- 能完成 Controller → Service → Repository 的数据流。
- 能处理分页、事务、数据库错误和资源权限。
- 能区分 HTTP、WebSocket、GraphQL 和微服务 Transport 的适用场景。
