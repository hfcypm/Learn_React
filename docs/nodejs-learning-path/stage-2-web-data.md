# 阶段二：Web 服务与数据层

**目标**：把 Node.js 核心能力组织成可维护的分层 API 服务，掌握 Controller、Service、Repository 的职责边界，并完成校验、认证、数据库和事务。

## 1. 服务分层

推荐使用以下边界：

```text
HTTP Controller -> Application Service -> Repository -> Database
```

- Controller 负责协议解析、状态码和响应格式。
- Application Service 负责业务用例和事务边界。
- Repository 负责数据库读写，不向 Controller 暴露 ORM 细节。
- Schema/Validator 负责运行时校验外部数据。

依赖方向是单向的：Controller 依赖 Service，Service 依赖 Repository。反向依赖会破坏边界。

## 2. 统一错误模型

所有错误通过一个 `AppError` 类型表达，包含稳定的 `code` 和 HTTP 状态码：

```js
// src/shared/errors.js
export class AppError extends Error {
  constructor(statusCode, code, message, options = {}) {
    super(message, options);
    this.statusCode = statusCode;
    this.code = code;
    this.fields = options.fields;
  }
}

export function isAppError(error) {
  return error instanceof AppError;
}
```

## 3. 校验：运行时规则

TypeScript 类型无法替代运行时校验。请求体、查询参数、环境变量和数据库返回值都属于不可信边界。

本阶段先用手写校验函数理解原理，生产项目再切换到 Zod、Valibot 等库。

```js
// src/modules/users/user.schema.js
export function parseCreateUser(input) {
  if (!input || typeof input !== 'object') {
    throw new AppError(400, 'VALIDATION_ERROR', '请求体格式错误');
  }

  const { name, email, password } = input;

  if (typeof name !== 'string' || name.trim().length < 2) {
    throw new AppError(400, 'VALIDATION_ERROR', 'name 至少 2 个字符', {
      fields: { name: '长度不足' },
    });
  }

  if (typeof email !== 'string' || !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
    throw new AppError(400, 'VALIDATION_ERROR', 'email 格式错误', {
      fields: { email: '邮箱格式错误' },
    });
  }

  return {
    name: name.trim(),
    email: email.trim().toLowerCase(),
    password: typeof password === 'string' ? password : '',
  };
}

export function parsePage(query) {
  const page = Number.parseInt(query.page ?? '1', 10);
  const pageSize = Number.parseInt(query.pageSize ?? '20', 10);

  if (!Number.isInteger(page) || page < 1) {
    throw new AppError(400, 'VALIDATION_ERROR', 'page 必须是正整数');
  }
  if (!Number.isInteger(pageSize) || pageSize < 1 || pageSize > 100) {
    throw new AppError(400, 'VALIDATION_ERROR', 'pageSize 必须在 1 到 100 之间');
  }

  return { page, pageSize, offset: (page - 1) * pageSize };
}
```

## 4. Repository：数据访问

Repository 只做数据读写，把数据库结果映射成业务对象。它不读取 HTTP 对象，也不做业务判断。

```js
// src/modules/users/user.repository.js
export class UserRepository {
  constructor(db) {
    this.db = db;
  }

  findByEmail(email) {
    return this.db
      .prepare('SELECT * FROM users WHERE email = ?')
      .get(email);
  }

  findById(id) {
    return this.db
      .prepare('SELECT * FROM users WHERE id = ?')
      .get(id);
  }

  findPage({ limit, offset }) {
    const rows = this.db
      .prepare(`
        SELECT id, name, email, role, created_at AS createdAt
        FROM users
        ORDER BY created_at DESC, id DESC
        LIMIT ? OFFSET ?
      `)
      .all(limit, offset);

    const total = this.db
      .prepare('SELECT COUNT(*) AS count FROM users')
      .get().count;

    return { rows, total };
  }

  insert(user) {
    this.db.prepare(`
      INSERT INTO users (id, name, email, password_hash, role, created_at, updated_at)
      VALUES (@id, @name, @email, @passwordHash, @role, @createdAt, @updatedAt)
    `).run(user);
    return user;
  }

  deleteById(id) {
    const result = this.db
      .prepare('DELETE FROM users WHERE id = ?')
      .run(id);
    return result.changes > 0;
  }
}
```

所有值使用参数绑定，禁止字符串拼接 SQL。分页必须有稳定排序（`created_at DESC, id DESC`），`limit` 需要经过上限校验。

## 5. Service：业务用例

Service 只依赖 Repository 和工具接口，不依赖 HTTP 对象，因此可以脱离服务器做单元测试。

```js
// src/modules/users/user.service.js
import { AppError } from '../../shared/errors.js';
import { hashPassword } from '../../auth/password.js';

export class UserService {
  constructor({ repository, passwordHasher, idGenerator, clock }) {
    this.repository = repository;
    this.passwordHasher = passwordHasher;
    this.idGenerator = idGenerator;
    this.clock = clock;
  }

  async create(input, actor) {
    if (input.role === 'admin' && actor?.role !== 'admin') {
      throw new AppError(403, 'FORBIDDEN', '只有管理员可以创建管理员');
    }

    if (this.repository.findByEmail(input.email)) {
      throw new AppError(409, 'EMAIL_TAKEN', '邮箱已被使用');
    }

    const now = this.clock().toISOString();
    const user = {
      id: this.idGenerator(),
      name: input.name,
      email: input.email,
      passwordHash: await this.passwordHasher.hash(input.password),
      role: input.role ?? 'user',
      createdAt: now,
      updatedAt: now,
    };

    return this.repository.insert(user);
  }

  async login(email, password) {
    const user = this.repository.findByEmail(email);

    if (!user || !(await this.passwordHasher.verify(user.password_hash, password))) {
      throw new AppError(401, 'INVALID_CREDENTIALS', '邮箱或密码错误');
    }

    return user;
  }

  list({ page, pageSize }) {
    const { rows, total } = this.repository.findPage({
      limit: pageSize,
      offset: (page - 1) * pageSize,
    });

    return {
      items: rows,
      page,
      pageSize,
      total,
      totalPages: Math.ceil(total / pageSize),
    };
  }
}
```

## 6. Controller：协议边界

Controller 读取 HTTP 输入、调用 Service、序列化响应，并处理全部异常到统一错误格式：

```js
// src/modules/users/user.controller.js
import { parseCreateUser, parsePage } from './user.schema.js';
import { toPublicUser } from './user.mapper.js';

export function createUserHandlers({ userService }) {
  return {
    async create(req, res, context) {
      const body = await context.readJsonBody(req);
      const input = parseCreateUser(body);
      const actor = await context.authenticate(req);
      const user = await userService.create(input, actor);
      context.sendJson(res, 201, {
        data: toPublicUser(user),
        requestId: context.requestId,
      });
    },

    async list(req, res, context) {
      const url = new URL(req.url, 'http://localhost');
      const { page, pageSize } = parsePage(Object.fromEntries(url.searchParams));
      const result = userService.list({ page, pageSize });
      context.sendJson(res, 200, {
        data: {
          items: result.items.map(toPublicUser),
          pagination: {
            page: result.page,
            pageSize: result.pageSize,
            total: result.total,
            totalPages: result.totalPages,
          },
        },
        requestId: context.requestId,
      });
    },

    async remove(req, res, context) {
      const id = req.url.split('/').filter(Boolean).pop();
      const actor = await context.authenticate(req);

      if (actor.role !== 'admin' && actor.id !== id) {
        throw new AppError(403, 'FORBIDDEN', '只能删除自己的账号或管理员账号');
      }

      const deleted = await userService.remove(id);
      if (!deleted) {
        throw new AppError(404, 'NOT_FOUND', '用户不存在');
      }

      context.sendJson(res, 204, { requestId: context.requestId });
    },
  };
}

// user.mapper.js：剥离内部字段
export function toPublicUser(user) {
  const { password_hash, passwordHash, ...rest } = user;
  return rest;
}
```

## 7. 认证：密码哈希与 Token

密码使用适合密码的哈希算法，本案例使用 `argon2`：

```js
// src/auth/password.js
import argon2 from 'argon2';

export async function hashPassword(plain) {
  return argon2.hash(plain);
}

export async function verifyPassword(hash, plain) {
  try {
    return await argon2.verify(hash, plain);
  } catch {
    return false;
  }
}
```

JWT 签发与验证使用 `jose`（`config` 来自阶段三的配置模块，此处省略导入）：

```js
// src/auth/token.js
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
    throw new AppError(401, 'UNAUTHORIZED', 'Token 身份信息无效');
  }
  return { id: payload.sub, role: payload.role };
}
```

Token 只放最小身份信息。JWT 签名验证成功仍需检查用户是否被禁用、租户是否有效和资源是否归属当前用户。

## 8. 认证中间件

把身份解析抽成 `authenticate`，供受保护路由复用：

```js
// src/auth/authenticate.js
import { verifyAccessToken } from './token.js';
import { AppError } from '../shared/errors.js';

export async function authenticate(req) {
  const header = req.headers.authorization ?? '';

  if (!header.startsWith('Bearer ')) {
    throw new AppError(401, 'UNAUTHORIZED', '缺少 Bearer Token');
  }

  return verifyAccessToken(header.slice(7));
}
```

Controller 可以组合认证与业务判断：

```js
async update(req, res, context) {
  const actor = await context.authenticate(req);
  const id = req.url.split('/').filter(Boolean).pop();

  if (actor.id !== id && actor.role !== 'admin') {
    throw new AppError(403, 'FORBIDDEN', '无权修改他人资料');
  }

  // 继续执行业务
}
```

## 9. 统一错误处理与响应

`sendAppError` 负责把 Zod 错误、认证错误、权限错误、唯一约束错误和未知异常映射为 400、401、403、404、409、500。未知异常只记录服务端日志，不把堆栈返回给客户端：

```js
// src/shared/http.js
import { AppError } from './errors.js';

export function sendJson(res, statusCode, data) {
  res.writeHead(statusCode, { 'content-type': 'application/json' });
  res.end(JSON.stringify(data));
}

export function sendAppError(res, error, requestId) {
  if (error instanceof AppError) {
    sendJson(res, error.statusCode, {
      error: {
        code: error.code,
        message: error.message,
        ...(error.fields ? { fields: error.fields } : {}),
      },
      requestId,
    });
    return;
  }

  console.error('未处理异常:', error);
  sendJson(res, 500, {
    error: { code: 'INTERNAL_ERROR', message: '服务器内部错误' },
    requestId,
  });
}
```

## 10. 数据库与事务

数据库访问需要连接池、超时、参数化查询和迁移管理。事务覆盖一个完整业务用例，并根据隔离级别处理并发更新。

事务示例（创建用户并写入审计记录，任一失败则回滚）：

```js
export class UserService {
  async createWithAudit(input, actor) {
    return this.db.transaction(() => {
      const user = this.repository.insert(this.buildUser(input));
      this.auditRepository.insert({
        actorId: actor.id,
        action: 'user.created',
        targetId: user.id,
        createdAt: this.clock().toISOString(),
      });
      return user;
    })();
  }
}
```

分页优先使用稳定排序和游标分页；列表接口限制最大 page size。缓存需要明确 key、TTL、失效和击穿保护。

## 11. Web 服务案例结构

完整项目参考：

```text
src/
├── server.js                 # 进程启动、监听和退出
├── app.js                    # HTTP 路由组装和 request ID
├── config.js                 # 环境变量校验
├── db/
│   ├── connection.js         # 数据库连接
│   ├── migrations.js         # 建表和版本迁移
│   └── user-repository.js    # 参数化查询
├── modules/users/
│   ├── user.schema.js        # 请求和响应校验
│   ├── user.service.js       # 业务规则
│   ├── user.controller.js    # HTTP 输入输出
│   └── user.mapper.js        # 内部字段剥离
├── auth/
│   ├── password.js           # 密码哈希
│   ├── token.js              # Token 签发和校验
│   └── authenticate.js       # Bearer Token 解析
└── shared/
    ├── errors.js             # 应用错误
    ├── http.js               # 请求体和响应工具
    └── logger.js             # 结构化日志
```

## 阶段验收

- 能实现 CRUD、分页、校验、统一错误和认证。
- 能解释事务、连接池、缓存失效和幂等键。
- 能把业务规则从 HTTP 层和数据库层隔离出来。
- 能为成功、校验失败、未认证、无权限和服务错误设计响应。

## 动手任务

1. 将用户功能拆成 Controller、Service、Repository 和 Schema 四个模块。
2. 为创建用户增加字段校验、重复邮箱冲突和统一错误响应。
3. 为列表接口增加稳定排序、最大 page size 和分页元数据。
4. 为一个完整业务用例划定事务边界，并写出成功和回滚场景。
5. 为 `PUT /users/:id` 增加更新逻辑，Service 层校验资源归属。

## 进入下一阶段的条件

你能够画出一次创建用户请求的完整数据流，能够在不修改 Controller 的情况下替换 Repository，并能为每类错误写出状态码和错误码。
