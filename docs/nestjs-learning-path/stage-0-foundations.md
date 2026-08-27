# 阶段零：NestJS 与 TypeScript 基础

**目标**：理解 NestJS 的设计理念、运行方式和 TypeScript 基础，能够从零创建一个可运行的健康检查模块。

## 1. NestJS 解决什么问题

NestJS 为 Node.js 应用提供模块化架构、依赖注入、装饰器和统一请求生命周期。它可以运行在 Express 或 Fastify 适配器之上，并支持 HTTP、WebSocket、GraphQL 和微服务传输层。

NestJS 的核心价值是让团队围绕模块、边界和依赖组织代码，形成可测试、松耦合、可维护的服务。

### 1.1 与 Node.js 的关系

NestJS 不是另一套运行时。它运行在 Node.js 之上，把 Node.js 的 `http.createServer` 封装为平台适配器。你仍然需要掌握 Node.js 的事件循环、Stream 和错误模型。

## 2. 创建项目

```bash
# 安装 Nest CLI
npm install -g @nestjs/cli

# 创建项目
nest new nest-demo

# 启动开发服务
npm run start:dev
```

启动后访问 http://localhost:3000 应看到默认页面。用 `--strict` 选项开启 TypeScript 严格模式，适合新项目。

### 2.1 生成的项目结构

```text
nest-demo/
├── src/
│   ├── main.ts              # 入口：创建应用并监听端口
│   ├── app.module.ts        # 根模块
│   ├── app.controller.ts    # 根 Controller
│   ├── app.service.ts       # 根 Service
│   └── app.controller.spec.ts
├── test/
│   ├── app.e2e-spec.ts      # E2E 测试
│   └── jest-e2e.json
├── nest-cli.json
├── package.json
└── tsconfig.json
```

### 2.2 CLI 常用命令

```bash
nest generate module health      # 生成 HealthModule
nest generate controller users   # 生成 UsersController
nest generate service users      # 生成 UsersService
nest generate guard auth         # 生成 AuthGuard
nest generate middleware request-id
nest generate pipe parse-id
```

## 3. 入口文件与根模块

`main.ts` 是应用入口，`NestFactory` 负责创建应用实例并监听端口：

```ts
// src/main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.setGlobalPrefix('api');
  await app.listen(process.env.PORT ?? 3000);
}

void bootstrap();
```

```ts
// src/app.module.ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

`NestFactory.create(AppModule)` 之后，框架内部会创建 Express 适配器和 HTTP 服务器。`setGlobalPrefix('api')` 让所有路由统一带有 `/api` 前缀。

## 4. TypeScript 必备知识

- interface、type、联合类型和泛型。
- `unknown` 与运行时校验。
- 装饰器、类、访问修饰符和依赖注入类型。
- Promise、异常、模块导入和环境变量类型。

装饰器表达元数据和框架约定，业务逻辑仍应保持在方法和服务中，避免把复杂行为隐藏在装饰器里。

## 5. 第一个 Controller

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

NestJS 会自动把返回值序列化为 JSON 响应。`@Controller('health')` 声明路由前缀，`@Get()` 声明 GET 方法。

### 5.1 完整可运行的健康检查模块

把 Controller 注册到 Module，让路由真正生效：

```ts
// src/health/health.controller.ts
import { Controller, Get } from '@nestjs/common';

@Controller('health')
export class HealthController {
  @Get()
  check() {
    return { status: 'ok' };
  }
}
```

```ts
// src/health/health.module.ts
import { Module } from '@nestjs/common';
import { HealthController } from './health.controller';

@Module({
  controllers: [HealthController],
})
export class HealthModule {}
```

```ts
// src/app.module.ts
import { Module } from '@nestjs/common';
import { HealthModule } from './health/health.module';

@Module({
  imports: [HealthModule],
})
export class AppModule {}
```

验证：

```bash
curl -i http://localhost:3000/api/health
# HTTP/1.1 200
# {"status":"ok"}
```

没有 `@Get()` 会怎样？`@Controller('health')` 只声明前缀，方法上的 `@Get()`、`@Post()` 声明具体方法和路由。两者缺一不可。

## 6. 平台适配器

NestJS 使用平台适配器承载请求。Express 生态成熟，Fastify 通常提供更低的 HTTP 开销；切换适配器前需要验证中间件、静态资源、上传和第三方插件兼容性。

```ts
import { NestFactory } from '@nestjs/core';
import { FastifyAdapter, NestFastifyApplication } from '@nestjs/platform-fastify';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create<NestFastifyApplication>(
    AppModule,
    new FastifyAdapter(),
  );
  await app.listen(3000);
}

void bootstrap();
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
5. 用 `nest generate` 分别生成 controller、service 和 module，观察目录变化。

## 进入下一阶段的条件

你能够从 `main.ts` 追踪到根模块、Controller 和路由，并能解释 NestJS 适配器与 Node.js HTTP 服务器的关系。
