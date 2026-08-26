# Node.js 综合实战：用户管理 API

本项目使用 Node.js 原生 HTTP、ESM、SQLite、运行时校验和内置测试，构建一个可以本地运行的用户管理 API。案例重点是理解运行时和分层边界，生产项目可以把 HTTP 层替换为 Fastify、Express 或其他框架。

## 1. 项目目标

实现以下能力：

- 用户创建、详情、分页列表和删除。
- 邮箱唯一校验和密码安全存储。
- Bearer Token 登录与管理员权限。
- 统一错误响应和 request ID。
- SQLite 数据库初始化与迁移。
- 单元测试、HTTP 集成测试和 Docker 运行。
- 健康检查、优雅退出和结构化日志。

不在本案例中实现真实邮件发送、OAuth、文件上传和分布式刷新 Token。这些能力需要额外的供应商、密钥管理和安全评审。

## 2. 技术选择

| 类别 | 选择 | 原因 |
|---|---|---|
| 运行时 | Node.js LTS | 获得长期安全和生态支持 |
| 模块 | ESM | 使用标准 `import/export` |
| HTTP | `node:http` | 展示请求、响应和请求体边界 |
| 数据库 | SQLite | 本地学习成本低，便于测试 |
| 校验 | Zod | 运行时校验与类型推导集中维护 |
| 测试 | `node:test` | 使用 Node.js 内置测试能力 |
| 日志 | Pino 或结构化 JSON 输出 | 支持生产日志采集 |

## 3. 初始化项目

```bash
mkdir node-user-api
cd node-user-api
npm init -y
npm install better-sqlite3 zod argon2 jose
npm install --save-dev typescript @types/node
```

`package.json` 建议配置：

```json
{
  "type": "module",
  "scripts": {
    "dev": "node --watch src/server.js",
    "start": "node src/server.js",
    "test": "node --test",
    "typecheck": "tsc --noEmit",
    "build": "tsc"
  }
}
```

实际项目应提交锁定文件，并根据 Node.js 版本选择 `better-sqlite3` 的兼容版本。只使用 JavaScript 时可以省略 TypeScript 命令。

## 4. 目录结构

```text
src/
├── server.js                 # 进程启动、监听和退出
├── app.js                    # HTTP 路由和请求生命周期
├── config.js                 # 环境变量校验
├── db/
│   ├── connection.js         # 数据库连接
│   ├── migrations.js         # 建表和版本迁移
│   └── user-repository.js    # 参数化查询
├── modules/users/
│   ├── user-schema.js        # 请求和响应校验
│   ├── user-service.js       # 业务规则
│   └── user-controller.js    # HTTP 输入输出
├── auth/
│   ├── password.js           # 密码哈希
│   └── token.js              # Token 签发和校验
└── shared/
    ├── errors.js             # 应用错误
    ├── http.js               # 请求体和响应工具
    └── logger.js             # 结构化日志
test/
├── user-service.test.js
└── users-api.test.js
```

依赖方向固定为 `controller -> service -> repository`。Repository 不读取 HTTP 对象，Controller 不拼接 SQL，Service 不依赖具体响应对象。

## 5. 配置与数据库

环境变量示例：

```env
NODE_ENV=development
PORT=3000
DATABASE_FILE=./data/app.db
TOKEN_SECRET=replace-with-a-local-development-secret
TOKEN_TTL_SECONDS=3600
```

真实 Token Secret 由部署环境注入。示例值只用于本地开发，不能提交生产凭证。

配置加载需要校验：

```js
import { z } from 'zod';

const ConfigSchema = z.object({
  NODE_ENV: z.enum(['development', 'test', 'production']).default('development'),
  PORT: z.coerce.number().int().min(1).max(65535).default(3000),
  DATABASE_FILE: z.string().min(1),
  TOKEN_SECRET: z.string().min(32),
  TOKEN_TTL_SECONDS: z.coerce.number().int().positive().default(3600),
});

export const config = ConfigSchema.parse(process.env);
```

最小迁移：

```sql
CREATE TABLE IF NOT EXISTS users (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  email TEXT NOT NULL UNIQUE,
  password_hash TEXT NOT NULL,
  role TEXT NOT NULL CHECK (role IN ('admin', 'user')),
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL
);

CREATE INDEX IF NOT EXISTS idx_users_created_at ON users(created_at);
```

迁移需要记录版本号。生产发布遵循“先发布兼容代码，再迁移，再启用新字段，最后清理旧字段”的顺序。

## 6. Repository

```js
export class UserRepository {
  constructor(db) {
    this.db = db;
  }

  findByEmail(email) {
    return this.db
      .prepare('SELECT * FROM users WHERE email = ?')
      .get(email);
  }

  findPage({ limit, offset }) {
    return this.db
      .prepare(`
        SELECT id, name, email, role, created_at AS createdAt
        FROM users
        ORDER BY created_at DESC, id DESC
        LIMIT ? OFFSET ?
      `)
      .all(limit, offset);
  }

  insert(user) {
    this.db.prepare(`
      INSERT INTO users (id, name, email, password_hash, role, created_at, updated_at)
      VALUES (@id, @name, @email, @passwordHash, @role, @createdAt, @updatedAt)
    `).run(user);
    return user;
  }
}
```

所有值使用参数绑定。分页必须有稳定排序，`limit` 需要经过上限校验。删除、更新和查询详情应继续采用同样的参数化方式。

## 7. Schema 与 Service

```js
import { z } from 'zod';

export const CreateUserSchema = z.object({
  name: z.string().trim().min(2).max(80),
  email: z.string().trim().toLowerCase().email(),
  password: z.string().min(12).max(128),
  role: z.enum(['admin', 'user']).default('user'),
});

export const PageSchema = z.object({
  page: z.coerce.number().int().min(1).default(1),
  pageSize: z.coerce.number().int().min(1).max(100).default(20),
});
```

Service 负责邮箱唯一性、密码哈希、角色创建权限和事务边界：

```js
export class UserService {
  constructor({ repository, passwordHasher, idGenerator, clock }) {
    this.repository = repository;
    this.passwordHasher = passwordHasher;
    this.idGenerator = idGenerator;
    this.clock = clock;
  }

  async create(input, actor) {
    if (input.role === 'admin' && actor?.role !== 'admin') {
      throw new AppError('FORBIDDEN', '只有管理员可以创建管理员');
    }
    if (this.repository.findByEmail(input.email)) {
      throw new AppError('EMAIL_TAKEN', '邮箱已被使用');
    }

    const now = this.clock().toISOString();
    const user = {
      id: this.idGenerator(),
      name: input.name,
      email: input.email,
      passwordHash: await this.passwordHasher.hash(input.password),
      role: input.role,
      createdAt: now,
      updatedAt: now,
    };
    return this.repository.insert(user);
  }
}
```

密码验证使用 `argon2` 等专用密码哈希库。登录、创建管理员和删除用户都需要审计记录。

## 8. HTTP 路由设计

| 方法 | 路径 | 权限 | 说明 |
|---|---|---|---|
| `GET` | `/health` | 公开 | 存活检查 |
| `POST` | `/auth/login` | 公开 | 邮箱密码登录 |
| `POST` | `/users` | 管理员 | 创建用户 |
| `GET` | `/users` | 已认证 | 分页列表 |
| `GET` | `/users/:id` | 已认证 | 用户详情 |
| `DELETE` | `/users/:id` | 管理员或本人 | 删除用户 |

响应约定：

```json
{
  "data": {},
  "requestId": "req_123"
}
```

错误响应：

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "请求参数无效",
    "fields": { "email": "邮箱格式错误" }
  },
  "requestId": "req_123"
}
```

路由组装时先生成 request ID，再读取请求体、校验输入、调用 Service，最后统一序列化响应：

```js
export async function handleCreateUser(req, res, context) {
  try {
    const body = await readJsonBody(req);
    const input = CreateUserSchema.parse(body);
    const actor = await context.authenticate(req);
    const user = await context.userService.create(input, actor);
    sendJson(res, 201, { data: toPublicUser(user), requestId: context.requestId });
  } catch (error) {
    sendAppError(res, error, context.requestId);
  }
}
```

`sendAppError` 负责把 Zod 错误、认证错误、权限错误、唯一约束错误和未知异常映射为 400、401、403、409、500。未知异常只记录服务端日志，不把堆栈返回给客户端。

## 9. 认证流程

1. `POST /auth/login` 校验邮箱和密码。
2. 密码哈希验证成功后签发短期 Token。
3. 受保护路由读取 `Authorization: Bearer <token>`。
4. Token 验证器只把最小身份信息放入请求上下文。
5. Service 再检查资源归属和角色权限。

Token Secret 轮换时需要支持旧密钥短暂验证窗口，或使用会话存储和撤销列表。生产环境应评估 HttpOnly Cookie、CSRF 和跨域策略。

使用 `jose` 签发和验证 JWT：

```js
import { SignJWT, jwtVerify } from 'jose';

const secret = new TextEncoder().encode(config.TOKEN_SECRET);

export async function signAccessToken(user) {
  return new SignJWT({ role: user.role })
    .setProtectedHeader({ alg: 'HS256' })
    .setSubject(user.id)
    .setIssuedAt()
    .setExpirationTime(`${config.TOKEN_TTL_SECONDS}s`)
    .sign(secret);
}

export async function verifyAccessToken(token) {
  const { payload } = await jwtVerify(token, secret);
  if (typeof payload.sub !== 'string' || typeof payload.role !== 'string') {
    throw new AppError('UNAUTHORIZED', 'Token 身份信息无效');
  }
  return { id: payload.sub, role: payload.role };
}
```

Token 只放最小身份信息。JWT 签名验证成功仍需检查用户是否被禁用、租户是否有效和资源是否归属当前用户。

## 10. 测试计划

单元测试：

- 邮箱重复时返回 `EMAIL_TAKEN`。
- 非管理员创建管理员被拒绝。
- 密码只保存哈希，不保存明文。
- 分页参数被限制在合法范围。
- Service 使用 fake Repository 时不触碰真实数据库。

集成测试：

- 测试数据库执行迁移。
- 创建用户后可以登录。
- 未认证访问返回 401。
- 普通用户创建用户返回 403。
- 删除不存在的用户返回 404。
- 重复邮箱返回 409。

```js
import test from 'node:test';
import assert from 'node:assert/strict';

test('普通用户不能创建管理员', async () => {
  const service = createUserServiceWithFakes();

  await assert.rejects(
    service.create(
      { name: 'Admin', email: 'a@example.com', password: 'long-password', role: 'admin' },
      { id: 'u1', role: 'user' },
    ),
    { code: 'FORBIDDEN' },
  );
});
```

## 11. Docker 与部署

```dockerfile
FROM node:22-bookworm-slim
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY src ./src
USER node
EXPOSE 3000
CMD ["node", "src/server.js"]
```

容器需要挂载或托管数据库文件，生产环境更适合使用 PostgreSQL 等独立数据库。部署前执行迁移、健康检查和冒烟请求；停止时等待在途请求并关闭数据库连接。

## 12. 验收清单

- [ ] `npm ci` 可以安装锁定依赖。
- [ ] `npm test` 覆盖 Service 和 HTTP 核心流程。
- [ ] `npm run typecheck` 通过，或项目明确采用纯 JavaScript。
- [ ] `/health` 能区分存活和数据库就绪状态。
- [ ] 所有 SQL 使用参数绑定。
- [ ] 密码、Token 和 Cookie 不进入日志。
- [ ] API 文档列出状态码、错误码和认证要求。
- [ ] Docker 启动后可以完成迁移和冒烟测试。
- [ ] 生产配置通过环境变量注入并完成校验。

## 13. 实施顺序

1. 创建 ESM 项目、配置校验和健康检查。
2. 完成 SQLite 连接、迁移和 Repository 单元测试。
3. 完成用户 Schema、Service 和用户 CRUD。
4. 加入密码哈希、登录和 JWT 验证。
5. 加入统一错误、request ID、审计日志和限流边界。
6. 补齐 HTTP 集成测试、Docker 和优雅退出。
7. 运行验收清单并记录未实现的生产能力。
