# 阶段一：NestJS 核心开发模型

**目标**：掌握 Module、Controller、Provider 和依赖注入的协作方式，能够把一个完整功能组织成 Feature Module。

## 1. Module

Module 使用 `imports`、`controllers`、`providers` 和 `exports` 声明依赖边界。业务功能通常以 feature module 组织，根模块负责组合功能模块。

### 1.1 完整的 UsersModule

```ts
// src/users/users.module.ts
import { Module } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { UsersRepository } from './users.repository';

@Module({
  controllers: [UsersController],
  providers: [UsersService, UsersRepository],
  exports: [UsersService],
})
export class UsersModule {}
```

只有导出的 Provider 才能被导入该模块的其他模块使用。共享模块需要谨慎设计，避免所有功能都依赖一个巨大模块。

### 1.2 跨模块使用 Service

当一个模块需要另一个模块的 Service 时，必须同时满足 `exports` 和 `imports` 两个条件：

```ts
// src/auth/auth.module.ts
import { Module } from '@nestjs/common';
import { UsersModule } from '../users/users.module';
import { AuthService } from './auth.service';

@Module({
  imports: [UsersModule],      // 引入 UsersModule 以获得导出的 Provider
  providers: [AuthService],
  exports: [AuthService],
})
export class AuthModule {}
```

```ts
// src/auth/auth.service.ts
import { Injectable } from '@nestjs/common';
import { UsersService } from '../users/users.service';

@Injectable()
export class AuthService {
  constructor(private readonly usersService: UsersService) {}
}
```

`UsersService` 必须出现在 `UsersModule` 的 `exports` 里，`AuthModule` 必须在 `imports` 里声明 `UsersModule`，否则依赖解析会失败。

## 2. Controller

Controller 负责路由、参数读取、状态码和响应协议。业务判断放在 Service，数据库访问放在 Repository。

```ts
// src/users/users.controller.ts
import { Controller, Get, Param, Post, Body, HttpCode } from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';

@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Post()
  create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }

  @Get()
  findAll() {
    return this.usersService.findAll();
  }
}
```

参数装饰器把请求数据注入方法参数：`@Body()` 请求体、`@Param()` 路径参数、`@Query()` 查询参数、`@Headers()` 请求头。

## 3. Provider 与依赖注入

Provider 可以是类、值、工厂或别名。构造函数注入让依赖显式化，测试时可以替换为 fake provider。

### 3.1 类 Provider

```ts
import { Injectable } from '@nestjs/common';
import { UsersRepository } from './users.repository';

@Injectable()
export class UsersService {
  constructor(private readonly usersRepository: UsersRepository) {}

  findOne(id: string) {
    return this.usersRepository.findById(id);
  }
}
```

### 3.2 四种 Provider 写法

```ts
// providers 数组中的多种写法
@Module({
  providers: [
    // 1. 类简写（等价于 useClass）
    UsersService,

    // 2. useValue：固定值，适合配置和测试
    { provide: 'DATABASE_NAME', useValue: 'nest_demo' },

    // 3. useFactory：工厂函数，适合带配置的第三方客户端
    {
      provide: 'PAYMENT_CLIENT',
      useFactory: (config: ConfigService) =>
        new PaymentClient(config.getOrThrow('PAYMENT_URL')),
      inject: [ConfigService],
    },

    // 4. useClass：按环境切换实现
    {
      provide: StorageService,
      useClass:
        process.env.NODE_ENV === 'production'
          ? S3StorageService
          : LocalStorageService,
    },
  ],
})
export class AppModule {}
```

### 3.3 注入字符串 Token

```ts
import { Inject, Injectable } from '@nestjs/common';

@Injectable()
export class ConfigConsumer {
  constructor(@Inject('DATABASE_NAME') private readonly databaseName: string) {}
}
```

## 4. 生命周期与作用域

默认 Provider 通常是单例。请求作用域会增加创建和管理成本，只有请求隔离数据确实需要时才使用。应用启动和关闭钩子适合连接数据库、启动消费者和释放资源。

```ts
import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';

@Injectable()
export class DatabaseService implements OnModuleInit, OnModuleDestroy {
  async onModuleInit() {
    // 启动时连接数据库
    await this.connect();
  }

  async onModuleDestroy() {
    // 优雅退出时释放资源
    await this.disconnect();
  }
}
```

## 5. Middleware

Middleware 适合记录原始请求、注入 request ID 和执行早期通用逻辑。认证、授权和业务前置判断使用 Guard，避免把所有逻辑堆进 Middleware。

```ts
// src/shared/middleware/request-id.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { randomUUID } from 'node:crypto';

@Injectable()
export class RequestIdMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: () => void) {
    const requestId = req.headers['x-request-id'] ?? randomUUID();
    req.headers['x-request-id'] = String(requestId);
    res.headers.set('x-request-id', String(requestId));
    next();
  }
}
```

在模块中应用：

```ts
import { MiddlewareConsumer, Module, NestModule } from '@nestjs/common';

@Module({})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(RequestIdMiddleware).forRoutes('*');
  }
}
```

## 6. Custom Decorator

Custom Decorator 用于复用参数读取或元数据声明，例如 `@CurrentUser()`。装饰器应保持轻量，权限决策仍由 Guard 或策略服务完成。

```ts
// src/auth/decorators/current-user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export interface CurrentActor {
  id: string;
  role: 'admin' | 'user';
}

export const CurrentUser = createParamDecorator(
  (_data: unknown, ctx: ExecutionContext): CurrentActor => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  },
);
```

```ts
// 使用
@Get('me')
me(@CurrentUser() actor: CurrentActor) {
  return this.usersService.findById(actor.id);
}
```

`request.user` 由认证 Guard 填充，本阶段先理解装饰器如何读取请求，认证在阶段二实现。

## 阶段验收

- 能把一个功能拆成 Module、Controller、Service 和 Repository。
- 能使用 exports/imports 控制 Provider 可见性。
- 能使用 fake provider 测试 Service。
- 能解释单例、请求作用域和生命周期钩子的区别。
- 能区分 Middleware、Guard、Custom Decorator 和 Custom Provider 的职责。

## 动手任务

1. 创建 `UsersModule`、`UsersController` 和 `UsersService`。
2. 使用 fake Repository 测试一个查询用例。
3. 将一个配置值通过 Custom Provider 注入 Service。
4. 增加 request ID Middleware，并记录一次请求的生命周期。
5. 实现 `@CurrentUser()` 装饰器，并在一个路由中读取当前用户。

## 进入下一阶段的条件

你能够解释 `imports`、`providers`、`exports` 的依赖方向，能够替换一个 Provider 完成单元测试，并能说明请求级 Provider 的成本。
