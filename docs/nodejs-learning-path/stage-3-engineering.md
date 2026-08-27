# 阶段三：Node.js 工程化与并发

**目标**：建立可测试、可诊断、可扩展的 Node.js 工程，掌握测试分层、结构化日志、Worker 和优雅退出。

## 1. 测试分层

- 单元测试：纯函数、校验器、业务规则和错误映射。
- 集成测试：Repository、数据库、缓存和 HTTP 边界。
- E2E 测试：登录、权限、核心 CRUD 和异常恢复。

Node.js 提供内置 `node:test` 模块，也可以使用 Jest、Vitest 等测试框架。测试应隔离时间、随机数、网络和外部服务。

### 1.1 单元测试

用 fake Repository 测试 Service，不触碰真实数据库：

```js
// test/user-service.test.js
import test from 'node:test';
import assert from 'node:assert/strict';
import { UserService } from '../src/modules/users/user.service.js';

function createServiceWithFakes() {
  const users = new Map();

  const repository = {
    findByEmail(email) {
      for (const user of users.values()) {
        if (user.email === email) return user;
      }
      return undefined;
    },
    insert(user) {
      users.set(user.id, user);
      return user;
    },
    findPage() {
      return { rows: [...users.values()], total: users.size };
    },
  };

  const service = new UserService({
    repository,
    passwordHasher: {
      hash: async (plain) => `hashed:${plain}`,
      verify: async (hash, plain) => hash === `hashed:${plain}`,
    },
    idGenerator: () => `u${users.size + 1}`,
    clock: () => new Date('2026-01-01T00:00:00.000Z'),
  });

  return { service, repository };
}

test('创建用户成功时返回哈希后的用户', async () => {
  const { service } = createServiceWithFakes();

  const user = await service.create(
    { name: 'Alice', email: 'alice@example.com', password: 'long-password' },
    { id: 'actor-1', role: 'admin' },
  );

  assert.equal(user.email, 'alice@example.com');
  assert.equal(user.passwordHash, 'hashed:long-password');
  assert.equal(user.createdAt, '2026-01-01T00:00:00.000Z');
});

test('普通用户不能创建管理员', async () => {
  const { service } = createServiceWithFakes();

  await assert.rejects(
    service.create(
      { name: 'Admin', email: 'admin@example.com', password: 'long-password', role: 'admin' },
      { id: 'u1', role: 'user' },
    ),
    (error) => error.code === 'FORBIDDEN',
  );
});

test('邮箱重复时返回 EMAIL_TAKEN', async () => {
  const { service } = createServiceWithFakes();

  await service.create(
    { name: 'Alice', email: 'alice@example.com', password: 'long-password' },
    { id: 'actor-1', role: 'admin' },
  );

  await assert.rejects(
    service.create(
      { name: 'Bob', email: 'alice@example.com', password: 'long-password' },
      { id: 'actor-1', role: 'admin' },
    ),
    (error) => error.code === 'EMAIL_TAKEN',
  );
});
```

### 1.2 HTTP 集成测试

启动真实服务器，用 `fetch` 发起请求，验证状态码和响应结构：

```js
// test/users-api.test.js
import test from 'node:test';
import assert from 'node:assert/strict';
import { createApp } from '../src/app.js';
import { createTestDatabase } from '../src/db/test-connection.js';

let server;

test.before(async () => {
  const app = createApp({ db: await createTestDatabase() });
  server = app.listen(0); // 随机端口
  await new Promise((resolve) => server.once('listening', resolve));
});

test.after(() => server.close());

test('GET /health 返回 200', async () => {
  const base = `http://127.0.0.1:${server.address().port}`;
  const response = await fetch(`${base}/health`);

  assert.equal(response.status, 200);
  assert.deepEqual(await response.json(), { status: 'ok' });
});

test('未认证访问 /users 返回 401', async () => {
  const base = `http://127.0.0.1:${server.address().port}`;
  const response = await fetch(`${base}/users`);

  assert.equal(response.status, 401);
});
```

集成测试要求测试数据库与开发数据库分离，每个用例独立清理，避免顺序依赖。

## 2. 配置加载

配置加载时校验必填变量、类型、默认值和环境边界：

```js
// src/config.js
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

启动时缺失关键配置直接失败，避免带着错误配置运行到请求阶段才暴露。

## 3. 结构化日志

日志使用结构化格式，至少包含时间、级别、服务名、版本、路由和 request ID。内容需要过滤密码、Token、Cookie、个人隐私和完整请求体。

### 3.1 结构化输出

```js
// src/shared/logger.js
function format(level, message, fields = {}) {
  return JSON.stringify({
    time: new Date().toISOString(),
    level,
    service: 'user-api',
    message,
    ...fields,
  });
}

export const logger = {
  info: (message, fields) => console.log(format('info', message, fields)),
  error: (message, fields) => console.error(format('error', message, fields)),
};
```

### 3.2 request ID 中间件

```js
// src/app.js 中的 request ID 生成
import { randomBytes } from 'node:crypto';

export function createRequestId(req, res, next) {
  const requestId = req.headers['x-request-id'] ?? randomBytes(8).toString('hex');
  req.requestId = requestId;
  res.setHeader('x-request-id', requestId);
  next();
}
```

记录请求耗时：

```js
function logRequest(req, res, start) {
  res.on('finish', () => {
    logger.info('request completed', {
      requestId: req.requestId,
      method: req.method,
      url: req.url,
      statusCode: res.statusCode,
      durationMs: Date.now() - start,
    });
  });
}
```

日志内容需要过滤密码、Token、Cookie、个人隐私和完整请求体。错误日志记录 cause 和堆栈，用户响应使用稳定错误码。

## 4. Worker 与多进程

Worker Threads 适合 CPU 密集型 JavaScript 工作，例如图片处理、压缩和复杂计算。I/O 密集型任务通常通过异步 API 完成。

### 4.1 对比：主线程阻塞

```js
// compute-sync.js —— 会阻塞事件循环
function heavyTask(iterations) {
  let result = 0;
  for (let i = 0; i < iterations; i++) {
    result += Math.sqrt(i) * Math.sin(i);
  }
  return result;
}

console.log(heavyTask(1_000_000));
console.log('这行要等上面计算完才执行');
```

### 4.2 Worker 版本

```js
// compute-worker.js —— 主线程调用 Worker
import { Worker } from 'node:worker_threads';

const worker = new Worker('./worker-task.js');
worker.postMessage(1_000_000);

worker.on('message', (result) => {
  console.log(`计算结果: ${result}`);
  worker.terminate();
});

worker.on('error', (error) => {
  console.error('Worker 错误:', error);
});
```

```js
// worker-task.js
import { parentPort } from 'node:worker_threads';

parentPort.on('message', (iterations) => {
  let result = 0;
  for (let i = 0; i < iterations; i++) {
    result += Math.sqrt(i) * Math.sin(i);
  }
  parentPort.postMessage(result);
});
```

Worker 之间通过消息通信，数据复制和序列化也有成本。小任务直接使用异步 API，只有真实 CPU 瓶颈才需要 Worker。

多进程可以利用多个 CPU 核心并提高隔离性，但需要处理共享状态、会话、优雅退出和进程重启。容器编排环境通常负责副本扩展。

## 5. 性能分析

- 先用指标确认瓶颈，再决定优化方向。
- 使用 `performance` API、CPU Profile、Heap Snapshot 和事件循环延迟观测。
- 关注 P50、P95、P99 延迟、吞吐、错误率和内存增长。
- 大响应使用分页、压缩和 Stream；避免无界缓存和同步大计算。

```js
// 测量函数耗时
import { performance } from 'node:perf_hooks';

const start = performance.now();
await heavyAsyncWork();
console.log(`耗时: ${(performance.now() - start).toFixed(2)}ms`);
```

```js
// 测量事件循环延迟
import { monitorEventLoopDelay } from 'node:perf_hooks';

const histogram = monitorEventLoopDelay({ resolution: 20 });
histogram.enable();
// 运行一段时间后
histogram.disable();
console.log(`P95 事件循环延迟: ${histogram.percentile(95).toFixed(0)}ms`);
```

Node.js 诊断工具包括 `node --inspect`、CPU Profile、Heap Snapshot、`node:perf_hooks` 和事件循环延迟监控。排查内存问题时记录采样时间、请求流量、堆大小和发布版本，区分真实泄漏与正常缓存增长。

覆盖率应服务于风险判断，重点覆盖鉴权、金额、状态转换、重试和错误恢复等分支。测试通过后再进行构建和容器镜像验证。

## 6. 优雅退出

收到 `SIGTERM` 后停止接收新请求，等待在途请求完成，关闭数据库连接和消息消费者，并在超时后退出。健康检查需要区分存活和就绪状态。

```js
// src/server.js
import http from 'node:http';
import { app } from './app.js';

const server = http.createServer(app);
const connections = new Set();

server.on('connection', (socket) => {
  connections.add(socket);
  socket.on('close', () => connections.delete(socket));
});

function shutdown(signal) {
  console.log(`收到 ${signal}，开始优雅退出`);

  // 停止接收新连接
  server.close(async () => {
    for (const socket of connections) {
      socket.destroy();
    }
    await db.close();
    console.log('资源已释放，退出');
    process.exit(0);
  });

  // 超时保护：10 秒后强制退出
  setTimeout(() => {
    console.error('优雅退出超时，强制退出');
    process.exit(1);
  }, 10_000).unref();
}

process.on('SIGTERM', () => shutdown('SIGTERM'));
process.on('SIGINT', () => shutdown('SIGINT'));

server.listen(config.PORT, () => {
  console.log(`服务已启动: http://127.0.0.1:${config.PORT}`);
});
```

验证优雅退出：

```bash
node src/server.js &
PID=$!
kill -TERM $PID
# 观察日志：收到 SIGTERM -> 资源已释放 -> 退出
```

`server.close()` 停止接收新连接，`connections` 集合用于主动关闭仍在途的 socket，超时保护防止进程永远不退出。

## 阶段验收

- 能使用 `node:test` 覆盖核心业务函数。
- 能输出带 request ID 的结构化日志。
- 能判断任务应使用异步 API、Worker 还是多进程。
- 能实现超时、优雅退出和健康检查。

## 动手任务

1. 为用户 Service 编写成功、校验失败和重复数据单元测试。
2. 为 HTTP 层增加 request ID 和结构化日志，过滤 Token 与密码。
3. 写一个 CPU 密集型示例，比较主线程计算和 Worker 的响应差异。
4. 模拟 `SIGTERM`，验证服务停止接收新请求并释放资源。
5. 为 `/health` 增加 `live` 和 `ready` 两个端点，`ready` 依赖数据库连接状态。

## 进入下一阶段的条件

你能够根据指标判断性能瓶颈，能够区分单元、集成和 E2E 测试边界，并能解释服务收到终止信号后的完整退出顺序。
