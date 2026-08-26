# NestJS 新手导读

## 适合谁

这套路线适合已经理解 JavaScript、TypeScript、HTTP 和 Node.js 基础，准备使用结构化框架开发服务端应用的人。NestJS 的学习重点是理解模块边界、依赖注入和请求生命周期，再将这些能力用于真实业务。

## 开始前准备

- 已完成 Node.js 阶段零和阶段一，能解释异步、HTTP、模块和错误。
- 掌握 TypeScript 的类、接口、装饰器基础和 Promise。
- 能使用终端、npm、Git 和 Postman 或 curl。
- 安装受支持的 Node.js LTS 与 Nest CLI。

```bash
node --version
npm --version
npm install -g @nestjs/cli
nest --version
```

## 学习顺序

| 阶段 | 先回答的问题 | 阶段产出 |
| --- | --- | --- |
| 零 | NestJS 如何启动，模块和装饰器解决什么问题？ | `/health` 模块 |
| 一 | Module、Controller、Provider 如何协作？ | 第一个 feature module |
| 二 | 请求如何经过校验、认证、业务和数据层？ | 带 DTO 和统一错误的用户 API |
| 三 | 如何测试、配置、记录和公开服务？ | 测试覆盖、OpenAPI 和配置校验 |
| 四 | 如何设计企业级模块和生产交付？ | 权限、缓存、队列和部署方案 |

## 框架心智模型

```text
main.ts
  -> RootModule
      -> FeatureModule
          -> Controller -> Service -> Repository
```

Controller 处理协议，Service 处理用例，Repository 处理数据访问。Module 负责组装依赖，Provider 负责提供可替换的能力。装饰器描述元数据，业务决策仍然放在清晰的方法和服务中。

## 常见排错顺序

```text
启动命令与端口
  -> RootModule 是否导入 FeatureModule
  -> Controller 路由与 HTTP 方法
  -> Provider 是否声明、导出和导入
  -> DTO 与 ValidationPipe
  -> Guard 身份与权限
  -> Service 业务异常
  -> Repository、数据库连接与迁移
```

## 阶段项目路线

从 [综合实战：用户权限后台 API](./project-practice.md) 开始，每完成一个阶段就增加一个能力：健康模块、用户模块、DTO 校验、认证授权、数据库、测试和生产部署。每个代码片段都标明上下文，避免把孤立片段误当成完整项目。

## 完成标准

- 能根据请求生命周期选择 Middleware、Guard、Pipe、Interceptor 或 Filter。
- 能使用 Module 和依赖注入组织可测试的业务边界。
- 能完成 DTO、认证、授权、数据库、测试和 OpenAPI。
- 能解释 NestJS 抽象与 Node.js 原生运行时之间的关系。
