# 阶段三：NestJS 工程化与测试

**目标**：建立可配置、可测试、可文档化的 NestJS 工程，掌握配置校验、单元测试、E2E 测试和 OpenAPI。

## 1. 配置管理

配置模块负责读取环境变量、默认值和运行环境校验。敏感配置由部署平台注入，仓库只保存 `.env.example` 占位符。配置按模块分组，避免业务代码到处读取 `process.env`。

### 1.1 配置校验函数

`ConfigModule.forRoot` 的 `validate` 选项接收校验函数，缺关键配置时直接阻止启动：

```ts
// src/config/env.validation.ts
import { plainToInstance, Type } from 'class-transformer';
import { IsEnum, IsInt, IsString, Min, MinLength, validateSync } from 'class-validator';

enum Environment {
  Development = 'development',
  Test = 'test',
  Production = 'production',
}

class EnvironmentVariables {
  @IsEnum(Environment)
  NODE_ENV!: Environment;

  @Type(() => Number)
  @IsInt()
  @Min(1)
  PORT!: number;

  @IsString()
  DATABASE_URL!: string;

  @IsString()
  @MinLength(32)
  JWT_SECRET!: string;
}

export function validate(config: Record<string, unknown>) {
  const validated = plainToInstance(EnvironmentVariables, config, {
    enableImplicitConversion: true,
  });
  const errors = validateSync(validated);

  if (errors.length > 0) {
    throw new Error(errors.toString());
  }

  return validated;
}
```

```ts
// src/app.module.ts
import { ConfigModule } from '@nestjs/config';
import { validate } from './config/env.validation';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      validate,
    }),
  ],
})
export class AppModule {}
```

缺少 `DATABASE_URL` 或 `JWT_SECRET` 时，应用在启动阶段直接失败，而不是运行到请求时才暴露。

### 1.2 按模块分组

```ts
// src/database/database.module.ts
import { Module } from '@nestjs/common';

@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: false }),
  ],
})
export class DatabaseModule {}
```

## 2. 单元测试

Nest Testing 模块可以创建 TestingModule，并通过 `overrideProvider` 替换依赖。

### 2.1 Service 单元测试

```ts
// src/users/users.service.spec.ts
import { Test } from '@nestjs/testing';
import { ConflictException, ForbiddenException } from '@nestjs/common';
import { UsersService } from './users.service';
import { UsersRepository } from './users.repository';

describe('UsersService', () => {
  let service: UsersService;

  const repositoryMock = {
    findByEmail: jest.fn(),
    create: jest.fn(),
  };

  beforeEach(async () => {
    const moduleRef = await Test.createTestingModule({
      providers: [
        UsersService,
        { provide: UsersRepository, useValue: repositoryMock },
      ],
    }).compile();

    service = moduleRef.get(UsersService);
  });

  afterEach(() => jest.clearAllMocks());

  it('邮箱已存在时抛出 ConflictException', async () => {
    repositoryMock.findByEmail.mockResolvedValue({ id: 'u1', email: 'a@example.com' });

    await expect(
      service.create({
        name: 'Alice',
        email: 'a@example.com',
        password: 'long-password',
      }),
    ).rejects.toThrow(ConflictException);
  });

  it('创建用户时写入密码哈希', async () => {
    repositoryMock.findByEmail.mockResolvedValue(null);
    repositoryMock.create.mockImplementation((data) => Promise.resolve({ id: 'u1', ...data }));

    const user = await service.create({
      name: 'Alice',
      email: 'alice@example.com',
      password: 'long-password',
    });

    expect(user.passwordHash).not.toBe('long-password');
    expect(repositoryMock.create).toHaveBeenCalledWith(
      expect.objectContaining({ email: 'alice@example.com' }),
    );
  });
});
```

### 2.2 Guard 单元测试

```ts
// src/auth/guards/roles.guard.spec.ts
import { ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { RolesGuard } from './roles.guard';
import { ROLES_KEY } from '../decorators/roles.decorator';

describe('RolesGuard', () => {
  let guard: RolesGuard;

  const reflector = {
    getAllAndOverride: jest.fn(),
  } as unknown as Reflector;

  const mockContext = (user: { role: string } | undefined) =>
    ({
      getHandler: () => ({}),
      getClass: () => ({}),
      switchToHttp: () => ({
        getRequest: () => ({ user }),
      }),
    }) as unknown as ExecutionContext;

  beforeEach(() => {
    guard = new RolesGuard(reflector);
    jest.clearAllMocks();
  });

  it('未声明角色时放行', () => {
    (reflector.getAllAndOverride as jest.Mock).mockReturnValue(undefined);
    expect(guard.canActivate(mockContext({ role: 'user' }))).toBe(true);
  });

  it('角色匹配时放行', () => {
    (reflector.getAllAndOverride as jest.Mock).mockReturnValue(['admin']);
    expect(guard.canActivate(mockContext({ role: 'admin' }))).toBe(true);
  });

  it('角色不匹配时拒绝', () => {
    (reflector.getAllAndOverride as jest.Mock).mockReturnValue(['admin']);
    expect(guard.canActivate(mockContext({ role: 'user' }))).toBe(false);
  });
});
```

单元测试关注 Service 和 Guard 的业务规则；集成测试关注模块与数据库、缓存、消息系统的协作；E2E 测试覆盖真实 HTTP 流程。

## 3. E2E 测试

E2E 测试创建完整应用，用 Supertest 发送真实 HTTP 请求：

```ts
// test/users.e2e-spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication, ValidationPipe } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from '../src/app.module';

describe('Users API (e2e)', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    app.useGlobalPipes(new ValidationPipe({ whitelist: true }));
    await app.init();
  });

  afterAll(async () => {
    await app.close();
  });

  it('GET /health 返回 200', () => {
    return request(app.getHttpServer())
      .get('/health')
      .expect(200)
      .expect({ status: 'ok' });
  });

  it('校验失败返回 400', () => {
    return request(app.getHttpServer())
      .post('/users')
      .send({ name: '', email: 'not-an-email' })
      .expect(400);
  });

  it('未认证访问受保护路由返回 401', () => {
    return request(app.getHttpServer())
      .get('/users')
      .expect(401);
  });
});
```

测试数据库使用独立实例或临时数据库。每组测试清理数据，避免共享状态导致顺序依赖。

## 4. OpenAPI 与文档

DTO、响应模型、认证方式和错误响应需要同步到 OpenAPI。文档既服务前端和测试，也用于发现接口边界不清、状态码不一致和字段缺失。

### 4.1 初始化 Swagger

```ts
// src/main.ts
import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';

const config = new DocumentBuilder()
  .setTitle('Nest Admin API')
  .setDescription('用户权限后台 API')
  .setVersion('1.0')
  .addBearerAuth()
  .build();

const document = SwaggerModule.createDocument(app, config);
SwaggerModule.setup('docs', app, document);
```

启动后访问 http://localhost:3000/docs 查看接口文档。

### 4.2 DTO 装饰器

```ts
import { ApiProperty } from '@nestjs/swagger';

export class CreateUserDto {
  @ApiProperty({ example: 'Alice' })
  @IsString()
  @MinLength(2)
  name!: string;

  @ApiProperty({ example: 'alice@example.com' })
  @IsEmail()
  email!: string;
}
```

### 4.3 路由级文档

```ts
import { ApiBearerAuth, ApiOperation, ApiResponse } from '@nestjs/swagger';

@ApiBearerAuth()
@ApiTags('users')
@Controller('users')
export class UsersController {
  @Get()
  @ApiOperation({ summary: '分页查询用户' })
  @ApiResponse({ status: 200, description: '成功' })
  @ApiResponse({ status: 401, description: '未认证' })
  findAll() {
    return this.usersService.findAll();
  }
}
```

使用 `@ApiBearerAuth()`、`@ApiProperty()`、`@ApiResponse()` 和 DTO 装饰器维护接口契约。公开文档不得暴露内部字段、管理接口细节或调试端点。

## 5. 日志、缓存和任务

### 5.1 结构化日志

```ts
// src/shared/logger.service.ts
import { Injectable, Logger, LoggerService } from '@nestjs/common';

@Injectable()
export class AppLogger extends Logger implements LoggerService {
  logRequest(method: string, url: string, requestId: string, durationMs: number) {
    this.log('request completed', {
      method,
      url,
      requestId,
      durationMs,
      level: 'info',
    });
  }
}
```

注册全局：

```ts
app.useLogger(app.get(AppLogger));
```

### 5.2 缓存

```ts
// 控制器中使用缓存
@Controller('users')
export class UsersController {
  constructor(private readonly cacheManager: CacheManager) {}

  @Get(':id')
  async findOne(@Param('id') id: string) {
    const cached = await this.cacheManager.get(`user:${id}`);
    if (cached) return cached;

    const user = await this.usersService.findOne(id);
    await this.cacheManager.set(`user:${id}`, user, 60_000);
    return user;
  }
}
```

CacheModule 用于明确的读取缓存，必须设计 TTL 和失效策略。用户资料变更后要主动删除对应缓存键。

### 5.3 定时任务

```ts
// src/tasks/cleanup.service.ts
import { Injectable, Logger } from '@nestjs/common';
import { Cron } from '@nestjs/schedule';

@Injectable()
export class CleanupService {
  private readonly logger = new Logger(CleanupService.name);

  @Cron('0 3 * * *') // 每天凌晨 3 点
  async cleanExpiredSessions() {
    const deleted = await this.sessionRepository.deleteExpired();
    this.logger.log(`清理过期会话 ${deleted} 条`);
  }
}
```

ScheduleModule 适合定时任务，重复执行需要幂等：同一个任务即使跑两次，也不应产生重复副作用。

### 5.4 队列

```ts
// src/tasks/tasks.processor.ts
import { Processor, Process } from '@nestjs/bull';
import { Job } from 'bull';

@Processor('email')
export class EmailProcessor {
  @Process('send')
  async send(job: Job<{ to: string; body: string }>) {
    await this.mailer.send(job.data);
    // 失败时抛出异常，BullMQ 按配置重试
  }
}
```

```ts
// 提交任务
await this.queue.add('send', { to, body }, {
  attempts: 3,
  backoff: { type: 'exponential', delay: 1000 },
});
```

BullMQ 等队列适合异步任务，消费者需要重试、死信和监控。幂等键用于防止重复消费。

## 阶段验收

- 能为核心 Service、Guard、Pipe 和 Controller 编写测试。
- 能使用配置模块校验环境变量。
- 能生成包含认证和错误模型的 OpenAPI 文档。
- 能为缓存、定时任务和队列设计幂等与失败恢复。
- 能在启动阶段拒绝无效配置，并维护与接口实现一致的 OpenAPI 文档。

## 动手任务

1. 为数据库地址和签名密钥增加启动时配置校验。
2. 使用 `TestingModule` 替换 Repository，测试 Service 和 Guard。
3. 为用户 API 生成 OpenAPI，并补充认证、分页和错误响应模型。
4. 为一个异步任务增加幂等键、重试上限和失败记录。
5. 为 `UsersService` 编写单测，覆盖邮箱冲突、权限不足和创建成功三个分支。

## 进入下一阶段的条件

你能够从测试模块追踪 Provider 替换过程，能够通过 OpenAPI 发现接口缺失，并能说明队列重试与 HTTP 重试的差异。
