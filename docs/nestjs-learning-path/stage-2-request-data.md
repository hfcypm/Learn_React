# 阶段二：NestJS 请求处理与数据层

**目标**：掌握请求生命周期扩展点，并建立安全、类型化的数据访问层，能够实现 DTO 校验、认证授权、统一错误和数据访问。

## 1. 请求生命周期

典型顺序为 Middleware、Guards、Interceptors 的前置逻辑、Pipes、Controller/Service、Interceptors 的后置逻辑，异常由 Exception Filters 处理。全局、Controller 和路由级绑定会影响执行范围。

```text
Request
  -> Middleware
  -> Guard
  -> Interceptor (before)
  -> Pipe
  -> Controller -> Service -> Repository
  -> Interceptor (after)
  -> Exception Filter (on error)
  -> Response
```

## 2. Pipes：转换与校验

Pipe 负责把输入转换为目标类型或拒绝非法输入。生产 API 通常启用全局 `ValidationPipe`，并配置白名单、转换和拒绝未知字段策略。

```ts
// src/main.ts
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,                 // 剥离 DTO 中未声明的字段
  forbidNonWhitelisted: true,      // 存在未知字段时返回 400
  transform: true,                 // 把查询参数转换为类型
}));
```

### 2.1 DTO 完整示例

```ts
// src/users/dto/create-user.dto.ts
import { IsEmail, IsString, MinLength, Matches } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @MinLength(2)
  name!: string;

  @IsEmail()
  email!: string;

  @IsString()
  @MinLength(12, { message: '密码至少 12 个字符' })
  password!: string;

  @Matches(/^[a-z0-9_]+$/)
  username!: string;
}
```

```ts
// src/users/dto/query-users.dto.ts
import { Type } from 'class-transformer';
import { IsInt, IsOptional, Max, Min } from 'class-validator';

export class QueryUsersDto {
  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  page?: number = 1;

  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @Max(100)
  pageSize?: number = 20;
}
```

DTO 描述输入结构，运行时校验库负责真正验证数据。DTO 与数据库 Entity 需要保持边界，避免直接把持久化模型暴露给 API。

### 2.2 方法参数使用

```ts
@Get()
findAll(@Query() query: QueryUsersDto) {
  return this.usersService.findAll(query);
}
```

## 3. Guards：认证与授权

Guard 决定请求是否可以进入路由。认证 Guard 解析身份，授权 Guard 检查角色、权限、租户和资源归属。装饰器只声明元数据，决策逻辑集中在 Guard 或策略服务。

### 3.1 JWT 认证 Guard

```ts
// src/auth/guards/jwt-auth.guard.ts
import { Injectable } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {}
```

```ts
// 使用
@UseGuards(JwtAuthGuard)
@Get('me')
me() {
  return this.usersService.findById(this.actor.id);
}
```

### 3.2 Passport JWT Strategy

```ts
// src/auth/strategies/jwt.strategy.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(config: ConfigService) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: config.getOrThrow<string>('JWT_SECRET'),
    });
  }

  validate(payload: { sub: string; role: string }) {
    return { id: payload.sub, role: payload.role };
  }
}
```

`validate` 的返回值会被注入到 `request.user`，供 Controller 和 Service 使用。

### 3.3 角色 Guard

角色装饰器声明所需角色，Guard 读取元数据并决策：

```ts
// src/auth/decorators/roles.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const ROLES_KEY = 'roles';
export const Roles = (...roles: string[]) => SetMetadata(ROLES_KEY, roles);
```

```ts
// src/auth/guards/roles.guard.ts
import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private readonly reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<string[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!requiredRoles?.length) return true;

    const { user } = context.switchToHttp().getRequest();
    return requiredRoles.includes(user?.role);
  }
}
```

```ts
// 使用
@Roles('admin')
@Post()
create(@Body() dto: CreateUserDto) {
  return this.usersService.create(dto);
}
```

角色 Guard 只解决粗粒度授权。订单归属、租户隔离、字段级权限仍由 Service 或策略层完成。

### 3.4 全局认证

在模块中注册全局 Guard：

```ts
@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: JwtAuthGuard,
    },
  ],
})
export class AuthModule {}
```

受保护路由需要认证，公开路由使用 `@Public()` 装饰器标记，并让 Guard 放行标记了元数据的路由。

## 4. Interceptors：横切能力

Interceptor 可以在处理器前后执行逻辑，适合日志、计时、响应转换、缓存和超时。事务和重试需要明确边界，避免拦截器隐藏关键业务行为。

### 4.1 计时日志

```ts
// src/shared/interceptors/logging.interceptor.ts
import { CallHandler, ExecutionContext, Injectable, NestInterceptor } from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<unknown> {
    const start = Date.now();

    return next.handle().pipe(
      tap(() => {
        const durationMs = Date.now() - start;
        const req = context.switchToHttp().getRequest();
        console.log(`${req.method} ${req.url} ${durationMs}ms`);
      }),
    );
  }
}
```

### 4.2 响应转换

```ts
@Injectable()
export class ResponseInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<unknown> {
    return next.handle().pipe(
      map((data) => ({
        data,
        timestamp: new Date().toISOString(),
      })),
    );
  }
}
```

## 5. Exception Filters

Filter 将异常转换为统一 HTTP 响应，并记录 request ID、错误码和内部原因。生产响应不应直接暴露堆栈、SQL 或第三方错误细节。

```ts
// src/shared/filters/all-exceptions.filter.ts
import {
  ArgumentsHost,
  Catch,
  ExceptionFilter,
  HttpException,
  HttpStatus,
} from '@nestjs/common';
import { Response } from 'express';

@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const requestId = ctx.getRequest().headers['x-request-id'];

    if (exception instanceof HttpException) {
      const status = exception.getStatus();
      const body = exception.getResponse();
      response.status(status).json({
        error: typeof body === 'string' ? { message: body } : body,
        requestId,
      });
      return;
    }

    console.error('未处理异常:', exception);
    response.status(HttpStatus.INTERNAL_SERVER_ERROR).json({
      error: { code: 'INTERNAL_ERROR', message: '服务器内部错误' },
      requestId,
    });
  }
}
```

在 `main.ts` 注册全局 Filter：

```ts
app.useGlobalFilters(new AllExceptionsFilter());
```

## 6. 数据层

NestJS 可以集成 TypeORM、Prisma、Mongoose 等数据工具。Repository 负责查询和映射，Service 负责业务事务。数据库连接、迁移、索引、分页、事务和 N+1 查询需要单独治理。

### 6.1 Prisma 完整接入

```ts
// src/prisma/prisma.service.ts
import { Injectable, OnModuleDestroy, OnModuleInit } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService
  extends PrismaClient
  implements OnModuleInit, OnModuleDestroy
{
  async onModuleInit() {
    await this.$connect();
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

```ts
// src/prisma/prisma.module.ts
import { Global, Module } from '@nestjs/common';
import { PrismaService } from './prisma.service';

@Global()
@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

### 6.2 Repository

```ts
// src/users/users.repository.ts
import { Injectable } from '@nestjs/common';
import { User } from '@prisma/client';
import { PrismaService } from '../prisma/prisma.service';

@Injectable()
export class UsersRepository {
  constructor(private readonly prisma: PrismaService) {}

  findByEmail(email: string): Promise<User | null> {
    return this.prisma.user.findUnique({ where: { email } });
  }

  findById(id: string): Promise<User | null> {
    return this.prisma.user.findUnique({ where: { id } });
  }

  async findPage(page: number, pageSize: number) {
    const [items, total] = await Promise.all([
      this.prisma.user.findMany({
        orderBy: [{ createdAt: 'desc' }, { id: 'desc' }],
        skip: (page - 1) * pageSize,
        take: pageSize,
      }),
      this.prisma.user.count(),
    ]);
    return { items, total };
  }

  create(data: { name: string; email: string; passwordHash: string; role: string }) {
    return this.prisma.user.create({ data });
  }
}
```

### 6.3 Service 与事务

```ts
@Injectable()
export class UsersService {
  constructor(private readonly repository: UsersRepository) {}

  async create(dto: CreateUserDto) {
    const existing = await this.repository.findByEmail(dto.email);
    if (existing) {
      throw new ConflictException('邮箱已被使用');
    }

    return this.repository.create({
      name: dto.name,
      email: dto.email,
      passwordHash: await hashPassword(dto.password),
      role: 'user',
    });
  }

  // 事务：创建用户 + 写入审计记录，任一失败则回滚
  async createWithAudit(dto: CreateUserDto, actorId: string) {
    return this.prisma.$transaction(async (tx) => {
      const user = await tx.user.create({ data: this.buildUser(dto) });
      await tx.audit.create({
        data: { actorId, action: 'user.created', targetId: user.id },
      });
      return user;
    });
  }
}
```

Controller 返回的 DTO 应与 Entity 或 ORM Model 分离。响应 DTO 只暴露允许公开的字段，密码哈希、内部状态和审计字段需要在映射层剔除：

```ts
// src/users/users.mapper.ts
export function toPublicUser(user: User) {
  const { passwordHash, ...rest } = user;
  return rest;
}
```

## 7. 其他传输层

- WebSocket 适合实时双向通信，需要连接认证、房间权限和断线重连策略。
- GraphQL 适合由客户端选择字段的 API，需要治理 schema、复杂查询和 N+1。
- 微服务 Transport 适合服务间消息或 RPC，需要设计超时、重试、幂等和消息版本。

## 阶段验收

- 能实现 DTO 校验、认证 Guard、统一错误 Filter。
- 能解释 Middleware、Guard、Interceptor、Pipe 的边界。
- 能完成 Controller → Service → Repository 的数据流。
- 能处理分页、事务、数据库错误和资源权限。
- 能区分 HTTP、WebSocket、GraphQL 和微服务 Transport 的适用场景。

## 动手任务

1. 为创建用户编写 DTO，并开启 `whitelist`、`forbidNonWhitelisted` 和 `transform`。
2. 为受保护路由增加认证 Guard，再增加角色 Guard。
3. 编写统一 Exception Filter，把校验、认证、权限和未知异常映射成稳定响应。
4. 让 Service 调用 Repository 完成一次事务，并为唯一约束错误编写测试。
5. 实现 `@Roles('admin')` 装饰器与 `RolesGuard`，验证管理员与普通用户权限差异。

## 进入下一阶段的条件

你能够根据问题选择 Pipe、Guard、Interceptor 或 Filter，能够从 Controller 追踪到数据库，并能解释 DTO 与持久化模型为何需要分离。
