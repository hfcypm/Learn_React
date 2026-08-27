# NestJS 学习评估与项目评分

**目标**：通过概念、编码、排错和项目评审确认学习者是否真正掌握了 NestJS 模块化、依赖注入和请求生命周期，避免只记住装饰器名称。

## 1. 阶段考核

### 阶段零：NestJS 与 TypeScript 基础

- 使用 Nest CLI 创建项目，解释 `main.ts`、`AppModule` 和启动流程。
- 说明装饰器如何携带元数据，以及元数据如何被框架读取。
- 完成一个可运行的健康检查模块并用 `curl` 验证。
- 对比 NestJS 的 Express 与 Fastify 适配器。

### 阶段一：核心开发模型

- 组织一个包含 Module、Controller、Service、Repository 的 feature module。
- 解释 imports、providers、exports 和依赖注入的作用。
- 使用至少三种 Provider 写法（useClass、useValue、useFactory）。
- 为请求注入 request ID，并理解生命周期钩子。

### 阶段二：请求处理与数据层

- 为 DTO 配置全局 ValidationPipe 并启用白名单。
- 实现 JWT 认证、角色授权和全局 Guard。
- 使用 Interceptor 完成响应转换，用 Filter 统一异常。
- 接入 Prisma 或等价 ORM，完成 Repository 和事务。

### 阶段三：工程化与测试

- 为 Service 和 Guard 编写单元测试并替换依赖。
- 使用 Supertest 完成 E2E 测试，覆盖 400、401、403、404、409。
- 为环境变量配置启动时校验。
- 生成 OpenAPI 文档，并补充认证与错误响应模型。

### 阶段四：企业级架构与部署

- 设计清晰的模块边界和依赖方向，说明 exports 和事件解耦。
- 接入缓存、队列或定时任务，说明幂等与失败恢复。
- 配置限流、CORS、安全响应头和健康检查。
- 完成 Docker、迁移、优雅退出和回滚验收。

## 2. 排错题

- Controller 路由未注册或与 HTTP 方法不匹配。
- Provider 未声明、未导出或未导入导致 DI 注入失败。
- Guard 返回 false 但未抛出可识别异常。
- DTO 未启用 whitelist 导致额外字段进入业务层。
- 依赖数组或闭包导致 Interceptor/Guard 读取过期上下文。
- Prisma 连接未释放或迁移未执行。
- JWT 密钥、过期时间和用户角色在 Guard 中读取不一致。
- 全局 Filter 吞掉错误或重复记录请求日志。
- `enableShutdownHooks` 缺失导致进程强制退出。

## 3. 项目评分

总分 100 分，70 分通过，90 分优秀。参考 [综合实战：用户权限后台 API](./project-practice.md)：

| 维度 | 分值 | 评分重点 |
|---|---:|---|
| 模块化设计 | 20 | 模块边界、依赖方向、exports、事件解耦 |
| 请求处理 | 15 | Pipe、Guard、Interceptor、Filter 的职责 |
| 数据层 | 15 | Repository、事务、审计、迁移 |
| 认证授权 | 15 | DTO 校验、JWT、角色、资源归属 |
| 测试质量 | 15 | 单元、Guard、E2E 的有效性 |
| 工程化 | 10 | 配置校验、OpenAPI、日志、健康检查 |
| 生产交付 | 10 | Docker、缓存队列、优雅退出、回滚 |

## 4. 项目评审问题

- Middleware、Guard、Interceptor、Pipe、Filter 分别在什么时机运行？
- 为什么 Controller 不应直接依赖 Repository？
- 哪些状态属于模块级、哪些属于请求级？
- 如何证明测试覆盖了 DTO、权限和数据层？
- 跨模块协作使用直接调用还是事件，如何选择？
- 队列或定时任务如何保证幂等和失败恢复？
- 生产部署时迁移、配置注入、优雅退出如何协作？

## 5. 学习记录模板

```markdown
## NestJS 阶段记录

- 本阶段目标：
- 已完成的模块或服务：
- 已通过的测试：
- 遇到的问题：
- 根因分析：
- 修复方式：
- 尚未掌握的概念：
- 下一阶段行动：
```
