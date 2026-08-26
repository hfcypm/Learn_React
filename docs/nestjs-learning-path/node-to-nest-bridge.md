# 从 Node.js 到 NestJS

NestJS 不是另一套 Node.js 运行时。它在 Node.js HTTP 能力之上提供模块化、依赖注入、装饰器和请求生命周期约定。

## 概念映射

| Node.js 原生概念 | NestJS 对应概念 | 学习重点 |
| --- | --- | --- |
| `http.createServer` | `NestFactory.create` | 框架负责启动适配器 |
| 手写 URL 路由 | `@Controller` 与 `@Get` | 路由声明与参数绑定 |
| 手写请求体解析 | Pipe 与 DTO | 转换和运行时校验 |
| 中间件函数 | Middleware | 原始请求级通用逻辑 |
| `if` 权限判断 | Guard | 请求是否允许进入处理器 |
| 包装响应或计时 | Interceptor | 前后置横切逻辑 |
| `try/catch` 错误映射 | Exception Filter | 统一异常响应 |
| 手动 `new Service()` | Provider 与 DI | 依赖组装和测试替换 |
| 文件或数据库模块 | Dynamic Module 与自定义 Provider | 配置和基础设施封装 |
| `node:test` 或 Jest | `TestingModule` | 框架上下文中的单元测试 |

## 同一个健康检查

Node.js 原生实现关注请求方法、路径和响应头：

```js
if (req.method === 'GET' && req.url === '/health') {
  res.writeHead(200, { 'content-type': 'application/json' })
  res.end(JSON.stringify({ status: 'ok' }))
}
```

NestJS 把路由声明交给装饰器：

```ts
@Controller('health')
export class HealthController {
  @Get()
  check() {
    return { status: 'ok' }
  }
}
```

两段代码最终都由 HTTP 服务器接收请求并返回 JSON。NestJS 示例隐藏了适配器、路由注册和响应序列化细节，学习者需要先理解原生版本，再理解框架提供的抽象。

## 什么时候使用哪种方式

- 小型脚本、一次性工具和极简服务可以直接使用 Node.js 原生 API。
- 多模块业务、多人协作、复杂鉴权和长期维护项目适合使用 NestJS 的约定与依赖边界。
- 性能问题先通过日志、指标和 profiling 定位，再决定是否更换适配器或拆分服务。
- 使用 NestJS 仍然需要掌握 Node.js 的事件循环、Stream、HTTP、进程和错误模型。

## 桥接练习

1. 用原生 Node.js 实现 `/health` 和 `/users`。
2. 用 NestJS 重写相同路由，比较目录和职责边界。
3. 将原生请求体校验替换为 DTO 与 `ValidationPipe`。
4. 将原生权限判断替换为 `AuthGuard` 与 `RolesGuard`。
5. 为两个版本分别编写请求测试，比较测试替换依赖的方式。
