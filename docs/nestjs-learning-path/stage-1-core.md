# 阶段一：NestJS 核心开发模型

**目标**：掌握 Module、Controller、Provider 和依赖注入的协作方式。

## 1. Module

Module 使用 `imports`、`controllers`、`providers` 和 `exports` 声明依赖边界。业务功能通常以 feature module 组织，根模块负责组合功能模块。

```ts
import { Module } from '@nestjs/common';

@Module({
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

只有导出的 Provider 才能被导入该模块的其他模块使用。共享模块需要谨慎设计，避免所有功能都依赖一个巨大模块。

## 2. Controller

Controller 负责路由、参数读取、状态码和响应协议。业务判断放在 Service，数据库访问放在 Repository。

```ts
import { Controller, Get, Param } from '@nestjs/common';

@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }
}
```

## 3. Provider 与依赖注入

Provider 可以是类、值、工厂或别名。构造函数注入让依赖显式化，测试时可以替换为 fake provider。

```ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class UsersService {
  constructor(private readonly usersRepository: UsersRepository) {}

  findOne(id: string) {
    return this.usersRepository.findById(id);
  }
}
```

## 4. 生命周期与作用域

默认 Provider 通常是单例。请求作用域会增加创建和管理成本，只有请求隔离数据确实需要时才使用。应用启动和关闭钩子适合连接数据库、启动消费者和释放资源。

## 5. Middleware、Custom Provider 与 Decorator

Middleware 适合记录原始请求、注入 request ID 和执行早期通用逻辑。认证、授权和业务前置判断使用 Guard，避免把所有逻辑堆进 Middleware。

Custom Provider 适合工厂、配置值和第三方客户端：

```ts
import { ConfigService } from '@nestjs/config';

// providers 数组中的 Provider 定义
{
  provide: 'PAYMENT_CLIENT',
  useFactory: (config: ConfigService) =>
    new PaymentClient(config.getOrThrow('PAYMENT_URL')),
  inject: [ConfigService],
}
```

Custom Decorator 用于复用参数读取或元数据声明，例如 `@CurrentUser()`。装饰器应保持轻量，权限决策仍由 Guard 或策略服务完成。

## 5. 阶段验收

- 能把一个功能拆成 Module、Controller、Service 和 Repository。
- 能使用 exports/imports 控制 Provider 可见性。
- 能使用 fake provider 测试 Service。
- 能解释单例、请求作用域和生命周期钩子的区别。
- 能区分 Middleware、Guard、Custom Decorator 和 Custom Provider 的职责。
