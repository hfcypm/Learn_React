# NestJS 学习路线图（基础 → 企业级）

**目标**：掌握 NestJS 的模块化架构、依赖注入和请求生命周期，能够构建可测试、可扩展、可部署的 TypeScript 服务端应用。

## 先看这里

- [新手导读](./start-here.md)：前置条件、学习顺序、框架心智模型和排错顺序。
- [从 Node.js 到 NestJS](./node-to-nest-bridge.md)：理解每个 NestJS 抽象对应的运行时能力。
- [综合实战：用户权限后台 API](./project-practice.md)：按阶段逐步扩展同一个项目。

## 学习阶段总览

| 阶段 | 名称 | 目标 |
|---|---|---|
| 零 | NestJS 与 TypeScript 基础 | 建立 CLI、模块和装饰器认知 |
| 一 | 核心开发模型 | 掌握 Module、Controller、Provider 和 DI |
| 二 | 请求处理与数据层 | 掌握 Pipes、Guards、Interceptors、Filters 和数据库 |
| 三 | 工程化与测试 | 掌握配置、验证、文档、测试和异步集成 |
| 四 | 企业级架构与部署 | 掌握鉴权、缓存、队列、微服务和生产交付 |

## 快速导航

- [阶段零：NestJS 与 TypeScript 基础](./stage-0-foundations.md)
- [阶段一：核心开发模型](./stage-1-core.md)
- [阶段二：请求处理与数据层](./stage-2-request-data.md)
- [阶段三：工程化与测试](./stage-3-engineering.md)
- [阶段四：企业级架构与部署](./stage-4-enterprise.md)
- [综合实战：用户权限后台 API](./project-practice.md)

## 架构主线

```text
Request -> Middleware -> Guard -> Interceptor -> Pipe -> Controller -> Provider -> Repository
                                      |                                          |
                                      +---------- Exception Filter <-------------+
```

请求生命周期中的每个扩展点承担单一职责。模块负责组织依赖，Controller 负责协议，Provider 负责业务，Repository 负责数据访问。

## 每阶段固定学习模板

每个阶段按照“目标 -> 前置知识 -> 核心问题 -> 最小示例 -> 项目增量 -> 验收 -> 排错”执行。先理解 Node.js 原生行为，再学习 NestJS 的封装点，避免只记装饰器名称。

## 官方资源

- [NestJS 官方文档](https://docs.nestjs.com/)
- [NestJS GitHub](https://github.com/nestjs/nest)
- [NestJS CLI](https://docs.nestjs.com/cli/overview)
