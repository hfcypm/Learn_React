# Node.js 与 NestJS 服务端学习计划

这份计划把 Node.js 和 NestJS 两套路线串成一条连续路径，适合前端开发者在 8 周左右完成第一次系统学习。每天投入 60 到 90 分钟时，优先保证项目练习和阶段验收，主题扩展可以延后阅读。

## 学习顺序

```text
JavaScript/TypeScript 前置
  -> Node.js 运行时
  -> 原生 HTTP 与数据层
  -> Node.js 工程化与生产
  -> Node.js/NestJS 概念桥接
  -> NestJS 核心模型
  -> NestJS 请求处理与数据层
  -> NestJS 企业级交付
```

## 8 周计划

| 周次 | 学习主题 | 必做产出 | 验收方式 |
| --- | --- | --- | --- |
| 第 1 周 | Node.js、模块、异步、环境变量 | CLI 脚本、JSON 配置读取 | 能解释事件循环和 Promise 异常 |
| 第 2 周 | HTTP、URL、请求体、Stream、取消 | `/health` 和 `/echo` 服务 | 用 curl 完成 GET/POST 和错误请求 |
| 第 3 周 | 校验、分层、数据库、事务、分页 | 用户 CRUD API | 覆盖成功、校验失败和重复数据 |
| 第 4 周 | 认证、测试、日志、优雅退出 | 登录和权限测试 | `node:test`、request ID、SIGTERM 验证 |
| 第 5 周 | 安全、Docker、健康检查、部署 | Node.js 项目交付检查表 | 构建镜像并完成冒烟请求 |
| 第 6 周 | NestJS 启动、Module、Controller、Provider、DI | NestJS 健康模块和 UsersModule | 能追踪模块依赖和请求路由 |
| 第 7 周 | DTO、Pipe、Guard、Interceptor、Filter、Prisma | NestJS 用户权限 API | 覆盖 400、401、403、404、409 |
| 第 8 周 | OpenAPI、E2E、缓存、队列、生产交付 | NestJS 项目交付检查表 | 完成构建、测试、迁移和 Docker 验收 |

## 每周固定流程

1. 阅读对应阶段的目标、前置条件和验收标准。
2. 用最小示例验证一个核心概念。
3. 把概念加入综合项目，保持一次只增加一个能力。
4. 主动制造一次输入错误、依赖失败或权限失败。
5. 运行测试和手工请求，记录结果与未解决问题。
6. 用三到五句话解释本周的请求数据流。

## 环境检查

```bash
node --version
npm --version
git --version
docker --version
curl --version
```

Docker 只在数据库、容器和部署阶段使用。Node.js 基础阶段可以先用本地进程和 SQLite，减少同时处理多个基础设施的认知负担。

## HTTP 调试模板

```bash
# 健康检查
curl -i http://localhost:3000/health

# 发送 JSON
curl -i -X POST http://localhost:3000/users \
  -H 'content-type: application/json' \
  -d '{"name":"Alice","email":"alice@example.com"}'

# 携带 Bearer Token
curl -i http://localhost:3000/users \
  -H 'authorization: Bearer <ACCESS_TOKEN>'
```

每次调试都观察请求方法、URL、请求头、请求体、状态码、响应体和 request ID。HTTP 问题先在这一层定位，再进入业务和数据库代码。

## 何时从 Node.js 进入 NestJS

完成 Node.js 阶段二后即可开始 NestJS 阶段零和阶段一。Node.js 阶段三、四可以与 NestJS 阶段三、四并行复习；Node.js 原生能力仍然是理解 NestJS 性能、异常、Stream 和生命周期的基础。

## 暂缓学习的主题

- 微服务拆分：先完成模块化单体和真实业务边界。
- GraphQL：先掌握稳定的 HTTP API 和 OpenAPI。
- 多进程集群：先测量事件循环延迟和吞吐瓶颈。
- 复杂缓存与消息系统：先完成数据库事务、幂等和失败处理。

## 最终能力检查

- 能从请求入口追踪到业务、数据库和响应出口。
- 能区分输入校验、认证、授权、业务错误和基础设施错误。
- 能为关键业务写单元、集成和 E2E 测试。
- 能使用日志、指标、健康检查和 request ID 排查问题。
- 能完成配置注入、数据库迁移、容器启动、冒烟测试和回滚说明。

## 对应文档

- [Node.js 学习路线](./nodejs-learning-path/README.md)
- [NestJS 学习路线](./nestjs-learning-path/README.md)
- [Node.js 综合实战](./nodejs-learning-path/project-practice.md)
- [NestJS 综合实战](./nestjs-learning-path/project-practice.md)
