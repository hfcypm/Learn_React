# NestJS 综合实战：用户权限后台 API

本项目使用 NestJS、TypeScript、PostgreSQL、Prisma、JWT、DTO 校验和 Jest，构建一个企业级用户与角色权限后台。案例展示模块、依赖注入和请求生命周期如何协作。

## 1. 项目目标

实现以下能力：

- 用户注册、登录、详情、分页和编辑。
- JWT 认证、角色授权和资源归属检查。
- DTO 校验、统一响应、异常过滤和 request ID。
- PostgreSQL 数据模型、迁移、事务和分页。
- 审计日志记录关键管理操作。
- OpenAPI 文档、单元测试、集成测试和 E2E 测试。
- Docker Compose 本地运行和生产部署检查。

项目只使用角色 `admin`、`manager`、`member` 展示基础 RBAC。真实系统还需要权限点、租户、组织树、审批和权限变更审计。

## 2. 技术选择

| 类别 | 选择 | 原因 |
|---|---|---|
| 框架 | NestJS | 模块化、依赖注入和标准生命周期 |
| 适配器 | Express 或 Fastify | 先保持默认生态，性能需求明确后评估 Fastify |
| 数据库 | PostgreSQL | 事务、约束和生产生态完整 |
| ORM | Prisma | 类型化查询和迁移工具 |
| 认证 | Passport + JWT | 与 Nest Guard 集成 |
| 校验 | class-validator + ValidationPipe | DTO 运行时校验 |
| 测试 | Jest + Supertest | 单元和 HTTP E2E 测试 |
| 文档 | Swagger | 生成 OpenAPI |

## 3. 初始化项目

```bash
npm i -g @nestjs/cli
nest new nest-admin-api
cd nest-admin-api
npm install @nestjs/config @nestjs/swagger @nestjs/passport @nestjs/jwt passport passport-jwt
npm install class-validator class-transformer @prisma/client argon2
npm install --save-dev prisma @types/passport-jwt supertest @types/supertest
npx prisma init
```

建议脚本：

```json
{
  "scripts": {
    "start:dev": "nest start --watch",
    "build": "nest build",
    "lint": "eslint \"{src,test}/**/*.ts\"",
    "test": "jest",
    "test:e2e": "jest --config ./test/jest-e2e.json",
    "prisma:generate": "prisma generate",
    "prisma:migrate": "prisma migrate dev",
    "prisma:deploy": "prisma migrate deploy"
  }
}
```

## 4. 目录结构

```text
src/
├── main.ts
├── app.module.ts
├── common/
│   ├── decorators/          # CurrentUser、Roles
│   ├── filters/             # HttpExceptionFilter
│   ├── guards/              # JwtAuthGuard、RolesGuard
│   ├── interceptors/        # RequestId、Logging
│   └── prisma/              # PrismaService
├── config/                  # 环境变量 schema
├── auth/
│   ├── auth.module.ts
│   ├── auth.controller.ts
│   ├── auth.service.ts
│   ├── jwt.strategy.ts
│   └── dto/
├── users/
│   ├── users.module.ts
│   ├── users.controller.ts
│   ├── users.service.ts
│   ├── users.repository.ts
│   └── dto/
└── audit/
    ├── audit.module.ts
    ├── audit.service.ts
    └── audit.repository.ts
prisma/
└── schema.prisma
test/
├── auth.e2e-spec.ts
└── users.e2e-spec.ts
```

依赖方向为 `Controller -> Service -> Repository -> PrismaService`。Auth Guard 负责身份识别，Roles Guard 负责粗粒度角色，UsersService 负责资源归属和业务规则。

## 5. 配置与入口

`.env.example`：

```env
NODE_ENV=development
PORT=3000
DATABASE_URL=postgresql://app:app@localhost:5432/nest_admin
JWT_SECRET=replace-with-a-local-development-secret
JWT_EXPIRES_IN=15m
```

真实 Secret 由部署环境注入。入口配置：

```ts
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.setGlobalPrefix('api');
  app.useGlobalPipes(new ValidationPipe({
    whitelist: true,
    forbidNonWhitelisted: true,
    transform: true,
  }));
  app.useGlobalFilters(new HttpExceptionFilter());
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

生产入口还应配置 CORS 白名单、Helmet、请求 ID、日志、优雅退出和 Swagger 是否启用的环境边界。

## 6. Prisma 数据模型

```prisma
enum Role {
  ADMIN
  MANAGER
  MEMBER
}

model User {
  id           String        @id @default(cuid())
  name         String
  email        String        @unique
  passwordHash String
  role         Role          @default(MEMBER)
  createdAt    DateTime      @default(now())
  updatedAt    DateTime      @updatedAt
  auditLogs    AuditLog[]
}

model AuditLog {
  id        String   @id @default(cuid())
  actorId   String
  action    String
  targetId  String?
  metadata  Json?
  createdAt DateTime @default(now())
  actor     User     @relation(fields: [actorId], references: [id])

  @@index([actorId, createdAt])
}
```

执行：

```bash
npx prisma migrate dev --name init
npx prisma generate
```

生产环境使用 `prisma migrate deploy`，迁移文件纳入版本控制。不要在应用启动时自动执行破坏性迁移。

## 7. Module、Repository 与 Service

```ts
import { Module } from '@nestjs/common';

@Module({
  imports: [AuthModule],
  controllers: [UsersController],
  providers: [UsersService, UsersRepository],
  exports: [UsersService],
})
export class UsersModule {}
```

Repository 封装 Prisma：

```ts
import { Injectable } from '@nestjs/common';
import { Prisma } from '@prisma/client';

@Injectable()
export class UsersRepository {
  constructor(private readonly prisma: PrismaService) {}

  findByEmail(email: string) {
    return this.prisma.user.findUnique({ where: { email } });
  }

  findPage(skip: number, take: number) {
    return this.prisma.user.findMany({
      skip,
      take,
      orderBy: [{ createdAt: 'desc' }, { id: 'desc' }],
      select: { id: true, name: true, email: true, role: true, createdAt: true },
    });
  }

  create(data: Prisma.UserCreateInput) {
    return this.prisma.user.create({ data });
  }

  findPublicById(id: string) {
    return this.prisma.user.findUnique({
      where: { id },
      select: { id: true, role: true, name: true, email: true },
    });
  }
}
```

Service 负责权限和事务：

下面的 `PasswordHasher`、`AuditService`、`CurrentActor` 和 `toPublicUser` 是项目内应定义的抽象，分别负责密码哈希、审计写入、当前身份类型和敏感字段剔除。

```ts
import { ConflictException, ForbiddenException, Injectable } from '@nestjs/common';

@Injectable()
export class UsersService {
  constructor(
    private readonly users: UsersRepository,
    private readonly audit: AuditService,
    private readonly passwordHasher: PasswordHasher,
  ) {}

  async create(dto: CreateUserDto, actor: CurrentActor) {
    if (dto.role === Role.ADMIN && actor.role !== Role.ADMIN) {
      throw new ForbiddenException('只有管理员可以创建管理员');
    }
    if (await this.users.findByEmail(dto.email)) {
      throw new ConflictException('邮箱已被使用');
    }

    const passwordHash = await this.passwordHasher.hash(dto.password);
    const user = await this.users.create({ ...dto, passwordHash });
    await this.audit.record(actor.id, 'user.created', user.id);
    return this.toPublicUser(user);
  }
}
```

Public DTO 不返回 `passwordHash`。数据库唯一约束仍必须保留，即使 Service 已预先查询邮箱，因为并发请求可能同时通过预检查。

## 8. DTO、Controller 和分页

```ts
export class CreateUserDto {
  @IsString()
  @MinLength(2)
  @MaxLength(80)
  name!: string;

  @IsEmail()
  email!: string;

  @IsString()
  @MinLength(12)
  password!: string;

  @IsEnum(Role)
  @IsOptional()
  role: Role = Role.MEMBER;
}

export class ListUsersQueryDto {
  @Type(() => Number)
  @IsInt()
  @Min(1)
  page = 1;

  @Type(() => Number)
  @IsInt()
  @Min(1)
  @Max(100)
  pageSize = 20;
}
```

```ts
import {
  Body,
  Controller,
  Get,
  Post,
  Query,
  UseGuards,
} from '@nestjs/common';

@Controller('users')
@UseGuards(JwtAuthGuard, RolesGuard)
export class UsersController {
  constructor(private readonly users: UsersService) {}

  @Get()
  list(@Query() query: ListUsersQueryDto) {
    return this.users.list(query.page, query.pageSize);
  }

  @Post()
  @Roles(Role.ADMIN, Role.MANAGER)
  create(@Body() dto: CreateUserDto, @CurrentUser() actor: CurrentActor) {
    return this.users.create(dto, actor);
  }
}
```

分页响应至少包含 `items`、`page`、`pageSize` 和 `total`。排序字段固定，避免分页期间数据变化造成重复或遗漏。

## 9. JWT 认证与角色授权

认证流程：

1. 登录 Service 校验密码并签发短期 JWT。
2. `JwtStrategy` 验证签名和过期时间，返回最小用户身份。
3. `JwtAuthGuard` 将身份放入请求。
4. `@Roles()` 写入元数据。
5. `RolesGuard` 读取元数据并判断当前角色。
6. Service 对资源归属和敏感操作再次校验。

```ts
import { CanActivate, ExecutionContext, Injectable, SetMetadata } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

export const Roles = (...roles: Role[]) => SetMetadata('roles', roles);

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private readonly reflector: Reflector) {}

  canActivate(context: ExecutionContext) {
    const required = this.reflector.getAllAndOverride<Role[]>('roles', [
      context.getHandler(),
      context.getClass(),
    ]);
    if (!required?.length) return true;
    const user = context.switchToHttp().getRequest().user as CurrentActor;
    return required.includes(user.role);
  }
}
```

角色 Guard 只解决粗粒度授权。订单归属、租户隔离、字段级权限仍由 Service 或策略层完成。

登录 Service 和 Strategy 示例：

```ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { JwtService } from '@nestjs/jwt';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';

@Injectable()
export class AuthService {
  constructor(
    private readonly users: UsersRepository,
    private readonly jwt: JwtService,
    private readonly passwordHasher: PasswordHasher,
  ) {}

  async login(email: string, password: string) {
    const user = await this.users.findByEmail(email);
    if (!user || !(await this.passwordHasher.verify(user.passwordHash, password))) {
      throw new UnauthorizedException('邮箱或密码错误');
    }

    return {
      accessToken: await this.jwt.signAsync({ sub: user.id, role: user.role }),
    };
  }
}
```

```ts
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(config: ConfigService, private readonly users: UsersRepository) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: config.getOrThrow<string>('JWT_SECRET'),
    });
  }

  async validate(payload: { sub: string; role: Role }) {
    const user = await this.users.findPublicById(payload.sub);
    if (!user) throw new UnauthorizedException();
    return { id: user.id, role: user.role } satisfies CurrentActor;
  }
}
```

## 10. 统一错误和审计

全局 Filter 将 `ValidationError`、`UnauthorizedException`、`ForbiddenException`、`NotFoundException`、`ConflictException` 和未知异常映射为稳定响应：

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "请求参数无效",
    "fields": { "email": ["必须是有效邮箱"] }
  },
  "requestId": "req_123"
}
```

审计日志记录 actor、action、target、request ID、时间和必要 metadata。密码、Token 和完整请求体不进入审计日志。

## 11. OpenAPI

```ts
import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';

const document = SwaggerModule.createDocument(app, new DocumentBuilder()
  .setTitle('Nest Admin API')
  .setVersion('1.0')
  .addBearerAuth()
  .build());

SwaggerModule.setup('docs', app, document);
```

使用 `@ApiBearerAuth()`、`@ApiProperty()`、`@ApiResponse()` 和 DTO 装饰器维护接口契约。公开文档不得暴露内部字段、管理接口细节或调试端点。

## 12. 测试计划

单元测试：

- UsersService 的邮箱冲突、角色限制和公开字段映射。
- JwtStrategy 的过期、签名失败和用户不存在。
- RolesGuard 的无角色、匹配和拒绝分支。
- ExceptionFilter 的业务错误和未知错误映射。

E2E 测试：

```ts
describe('Users API', () => {
  it('普通成员不能创建用户', async () => {
    await request(app.getHttpServer())
      .post('/api/users')
      .set('Authorization', `Bearer ${memberToken}`)
      .send({ name: 'New User', email: 'new@example.com', password: 'long-password' })
      .expect(403);
  });
});
```

测试数据库使用独立实例或临时数据库。每组测试清理数据，避免共享状态导致顺序依赖。

## 13. Docker Compose 与部署

```yaml
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: nest_admin
      POSTGRES_USER: app
      POSTGRES_PASSWORD: local-only-password
    ports:
      - '5432:5432'
  api:
    build: .
    environment:
      DATABASE_URL: postgresql://app:local-only-password@postgres:5432/nest_admin
      JWT_SECRET: replace-with-a-local-development-secret
    depends_on:
      - postgres
```

Compose 中的密码只用于本地演示，生产环境改用 Secret 管理。生产部署执行 `npm ci`、`npm run build`、`npx prisma migrate deploy`、健康检查和 E2E/冒烟测试。

## 14. 验收清单

- [ ] `npm run build`、`npm run lint` 和 `npm test` 通过。
- [ ] E2E 覆盖登录、401、403、404、409 和成功 CRUD。
- [ ] DTO 开启白名单和未知字段拒绝。
- [ ] JWT Secret、数据库地址和环境配置经过启动校验。
- [ ] 响应 DTO 不泄露密码哈希和内部字段。
- [ ] 角色授权与资源归属分别测试。
- [ ] Prisma 迁移可以在空数据库和已有数据库执行。
- [ ] OpenAPI 文档与真实状态码、响应和鉴权一致。
- [ ] 审计日志覆盖用户创建、角色修改和删除。
- [ ] Docker Compose 可以启动数据库和 API 并完成健康检查。

## 15. 实施顺序

1. 创建 NestJS 项目、配置模块和健康检查。
2. 建立 Prisma Schema、迁移和 PrismaService。
3. 完成 UsersModule、Repository、Service、DTO 和分页接口。
4. 完成 AuthModule、密码哈希、登录、JWT Strategy 和 Guard。
5. 加入 Roles Decorator、RolesGuard、资源归属检查和审计日志。
6. 接入全局 Pipe、Exception Filter、request ID、Swagger 和日志。
7. 编写单元测试与 E2E 测试，最后补充 Docker、迁移和部署检查。
