# 阶段四：NestJS 企业级架构与部署

**目标**：完成大型 NestJS 应用的边界设计、异步通信、安全治理和生产交付。

## 1. 模块边界

按业务能力组织模块，例如 Identity、Users、Orders、Billing 和 Audit。模块之间通过公开接口和事件协作，避免跨模块直接访问 Repository。复杂业务可以分为 API、Application、Domain 和 Infrastructure 层。

### 1.1 依赖方向

```text
ApiModule (Controller/DTO)
  -> ApplicationModule (Service/用例)
  -> DomainModule (实体/规则)
  -> InfrastructureModule (Repository/ORM/外部服务)
```

依赖只能向下，不能反向。Controller 不直接依赖 Repository，Service 不感知 HTTP 细节。

### 1.2 Feature Module 组织示例

```ts
// src/orders/orders.module.ts
import { Module } from '@nestjs/common';
import { OrdersController } from './orders.controller';
import { OrdersService } from './orders.service';
import { OrdersRepository } from './orders.repository';
import { BillingModule } from '../billing/billing.module';
import { AuditModule } from '../audit/audit.module';

@Module({
  imports: [BillingModule, AuditModule],
  controllers: [OrdersController],
  providers: [OrdersService, OrdersRepository],
  exports: [OrdersService],
})
export class OrdersModule {}
```

`OrdersModule` 只通过 `BillingModule` 和 `AuditModule` 导出的公开接口协作，不直接访问它们的 Repository。

### 1.3 共享事件

跨模块解耦使用事件而不是直接调用：

```ts
// src/events/order.events.ts
export class OrderCreatedEvent {
  constructor(
    public readonly orderId: string,
    public readonly total: number,
  ) {}
}
```

```ts
// src/orders/orders.service.ts
import { EventEmitter2 } from '@nestjs/event-emitter';

@Injectable()
export class OrdersService {
  constructor(private readonly eventEmitter: EventEmitter2) {}

  async create(dto: CreateOrderDto) {
    const order = await this.ordersRepository.create(dto);
    this.eventEmitter.emit('order.created', new OrderCreatedEvent(order.id, order.total));
    return order;
  }
}
```

```ts
// src/audit/audit.service.ts
import { OnEvent } from '@nestjs/event-emitter';

@Injectable()
export class AuditService {
  @OnEvent('order.created')
  handleOrderCreated(event: OrderCreatedEvent) {
    // 记录审计日志，不阻塞订单主流程
  }
}
```

事件让订单模块不再直接依赖审计模块，两个模块可以独立演化和测试。

## 2. 微服务与消息通信

NestJS 支持 TCP、Redis、NATS、MQTT、RabbitMQ 和 Kafka 等传输方式。选择通信方式时评估投递语义、顺序、重试、幂等、消息版本和可观测性。

### 2.1 微服务消费者

```ts
// src/analytics/analytics.controller.ts
import { Controller } from '@nestjs/common';
import { MessagePattern, Payload } from '@nestjs/microservices';

interface UserCreatedMessage {
  userId: string;
  email: string;
}

@Controller()
export class AnalyticsController {
  @MessagePattern('user.created')
  handleUserCreated(@Payload() message: UserCreatedMessage) {
    // 幂等处理：同一消息重复投递时不产生重复副作用
    const key = `user.created:${message.userId}`;
    if (await this.redis.setnx(key, '1')) {
      await this.analyticsRepository.recordUser(message);
    }
  }
}
```

### 2.2 微服务生产者

```ts
// src/users/users.service.ts
import { Inject } from '@nestjs/common';
import { ClientProxy } from '@nestjs/microservices';

@Injectable()
export class UsersService {
  constructor(@Inject('ANALYTICS_SERVICE') private readonly client: ClientProxy) {}

  async create(dto: CreateUserDto) {
    const user = await this.usersRepository.create(dto);
    await this.client.emit('user.created', { userId: user.id, email: user.email });
    return user;
  }
}
```

消息消费者必须能够安全重复处理。幂等键（如 `user.created:<id>`）是防止重复消费的基本手段。

HTTP 适合同步查询和用户交互；消息适合异步任务、领域事件和服务解耦。

微服务拆分需要先确认团队边界、数据所有权和独立发布需求。单体模块化应用可以先提供清晰边界，再在真实吞吐、组织协作或隔离需求出现时拆分服务。

## 3. 安全与权限

### 3.1 全局安全配置

```ts
// src/main.ts
app.enableCors({
  origin: ['https://app.example.com'],
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
});
app.use(helmet());
app.useGlobalPipes(new ValidationPipe({ whitelist: true, forbidNonWhitelisted: true }));
```

### 3.2 速率限制

使用 `@nestjs/throttler`：

```ts
// src/app.module.ts
import { ThrottlerModule } from '@nestjs/throttler';

@Module({
  imports: [
    ThrottlerModule.forRoot([{ ttl: 60_000, limit: 100 }]),
  ],
})
export class AppModule {}
```

```ts
// 针对登录接口更严格
@Throttle({ default: { limit: 5, ttl: 60_000 } })
@Post('login')
login(@Body() dto: LoginDto) {
  return this.authService.login(dto);
}
```

### 3.3 全局认证与资源归属

资源级权限（如"只能删除自己的订单"）由 Service 检查：

```ts
@Injectable()
export class OrdersService {
  async remove(orderId: string, actor: CurrentActor) {
    const order = await this.ordersRepository.findById(orderId);

    if (!order) {
      throw new NotFoundException('订单不存在');
    }

    if (order.userId !== actor.id && actor.role !== 'admin') {
      throw new ForbiddenException('只能删除自己的订单');
    }

    return this.ordersRepository.deleteById(orderId);
  }
}
```

对密码、Token、Cookie 和个人数据进行脱敏。日志中不记录完整请求体、密码和 Token。

## 4. 性能与稳定性

### 4.1 健康检查

```ts
// src/health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import { HealthCheck, HealthCheckService, TypeOrmHealthIndicator } from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private readonly health: HealthCheckService,
    private readonly db: TypeOrmHealthIndicator,
  ) {}

  @Get('live')
  live() {
    return { status: 'ok' };
  }

  @Get('ready')
  @HealthCheck()
  ready() {
    return this.health.check([
      () => this.db.pingCheck('database'),
    ]);
  }
}
```

`/health/live` 表示进程存活，`/health/ready` 检查数据库是否就绪，用于编排系统流量控制。

### 4.2 优雅退出

```ts
// src/main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableShutdownHooks(); // 启用 SIGTERM/SIGINT 处理

  await app.listen(3000);
}

void bootstrap();
```

```ts
// PrismaService 实现 OnModuleDestroy 释放连接
@Injectable()
export class PrismaService extends PrismaClient implements OnModuleDestroy {
  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

`enableShutdownHooks()` 让 NestJS 在收到终止信号时按依赖顺序调用各 Provider 的 `onModuleDestroy`，关闭 HTTP、数据库、缓存和消息消费者。

### 4.3 缓存与查询优化

- 使用缓存降低数据库压力，明确 key、TTL 和失效。
- 分页、批量查询和索引减少查询成本。
- 使用队列隔离耗时任务，使用超时、重试和熔断保护依赖。
- 使用 Fastify 适配器前先验证插件和生态兼容性。

## 5. 部署与验收

### 5.1 Dockerfile

```dockerfile
FROM node:22-bookworm-slim AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:22-bookworm-slim
WORKDIR /app
ENV NODE_ENV=production
COPY package*.json ./
RUN npm ci --omit=dev
COPY --from=builder /app/dist ./dist
COPY prisma ./prisma
USER node
EXPOSE 3000
CMD ["sh", "-c", "npx prisma migrate deploy && node dist/main.js"]
```

### 5.2 CI 流程

CI 流程至少包含安装锁定依赖、lint、类型检查、单元测试、E2E、构建和安全扫描：

```text
npm ci
  -> npm run lint
  -> npm run typecheck
  -> npm test
  -> npm run test:e2e
  -> npm run build
  -> npm audit
```

部署需要管理迁移顺序、配置注入、日志采集、指标告警和回滚。生产迁移使用 `prisma migrate deploy`，不在应用启动时自动执行破坏性迁移。

### 5.3 生产验收

- 模块化业务边界和清晰依赖方向。
- 认证、授权、校验、错误和审计能力。
- 数据库、缓存、队列和外部服务的失败处理。
- OpenAPI、测试、健康检查、结构化日志和部署手册。

## 动手任务

1. 将 Users、Identity 和 Audit 拆成独立 Feature Module，明确 exports。
2. 为管理员和普通用户编写角色、资源归属和审计测试。
3. 选择一个异步操作接入队列，定义重试、死信和重复消费处理。
4. 编写容器启动、数据库迁移、readiness 和回滚验收步骤。
5. 用 `@nestjs/event-emitter` 实现订单创建事件，让审计模块异步响应。

## 路线终点

完成 [综合实战：用户权限后台 API](./project-practice.md) 的验收项后，你应能够解释模块边界、依赖方向、异步通信和生产部署之间的关系。
