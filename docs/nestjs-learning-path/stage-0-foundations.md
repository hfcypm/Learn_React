# 阶段零：NestJS 与 TypeScript 基础

**目标**：理解 NestJS 的设计理念、运行方式和 TypeScript 基础。

## 1. NestJS 解决什么问题

NestJS 为 Node.js 应用提供模块化架构、依赖注入、装饰器和统一请求生命周期。它可以运行在 Express 或 Fastify 适配器之上，并支持 HTTP、WebSocket、GraphQL 和微服务传输层。

NestJS 的核心价值是让团队围绕模块、边界和依赖组织代码，形成可测试、松耦合、可维护的服务。

## 2. 创建项目

```bash
# 安装 Nest CLI
npm install -g @nestjs/cli

# 创建项目
nest new nest-demo

# 启动开发服务
npm run start:dev
```

项目通常包含 `main.ts`、根模块、Controller、Service、测试和配置文件。提交前执行 lint、测试和构建。

NestJS 使用平台适配器承载请求。Express 生态成熟，Fastify 通常提供更低的 HTTP 开销；切换适配器前需要验证中间件、静态资源、上传和第三方插件兼容性。

## 3. TypeScript 必备知识

- interface、type、联合类型和泛型。
- `unknown` 与运行时校验。
- 装饰器、类、访问修饰符和依赖注入类型。
- Promise、异常、模块导入和环境变量类型。

装饰器表达元数据和框架约定，业务逻辑仍应保持在方法和服务中，避免把复杂行为隐藏在装饰器里。

## 4. 第一个 Controller

```ts
import { Controller, Get } from '@nestjs/common';

@Controller('health')
export class HealthController {
  @Get()
  check() {
    return { status: 'ok' };
  }
}
```

## 阶段验收

- 能创建 NestJS 项目并解释入口文件和根模块。
- 能使用装饰器声明 Controller 和路由。
- 能为服务端输入和返回值建立 TypeScript 类型。
- 能运行 lint、测试和 production build。
- 能说明 Express 与 Fastify 适配器的选型边界。

## 动手任务

1. 使用 CLI 创建项目并运行 `npm run start:dev`。
2. 创建 `HealthModule`，让 `GET /health` 返回 `{ status: 'ok' }`。
3. 在 `main.ts` 中增加全局前缀，例如 `api`，用 curl 验证最终路径。
4. 修改端口配置，确认环境变量和默认值的行为。

## 进入下一阶段的条件

你能够从 `main.ts` 追踪到根模块、Controller 和路由，并能解释 NestJS 适配器与 Node.js HTTP 服务器的关系。
