# 阶段四：Node.js 生产交付

**目标**：把 Node.js 服务安全、稳定地运行在生产环境，掌握安全基线、可观测性、可靠性模式和部署流程。

## 1. 安全基线

- 校验所有外部输入和文件上传。
- 设置请求体、Header、URL 和连接超时限制。
- 使用参数化查询，避免字符串拼接 SQL。
- 配置安全响应头、CORS 白名单和 HTTPS。
- 锁定依赖版本，定期审计依赖漏洞。
- 对认证、导出、搜索和写操作实施速率限制。

### 1.1 安全响应头

```js
// src/app.js 中的安全头中间件
const securityHeaders = {
  'content-security-policy': "default-src 'self'",
  'x-content-type-options': 'nosniff',
  'x-frame-options': 'DENY',
  'referrer-policy': 'no-referrer',
  'strict-transport-security': 'max-age=63072000; includeSubDomains',
};

function applySecurityHeaders(req, res, next) {
  for (const [name, value] of Object.entries(securityHeaders)) {
    res.setHeader(name, value);
  }
  next();
}
```

生产环境可以直接使用 `helmet` 包维护这些头。

### 1.2 CORS 白名单

```js
const ALLOWED_ORIGINS = new Set(['https://app.example.com']);

function applyCors(req, res, next) {
  const origin = req.headers.origin;

  if (origin && ALLOWED_ORIGINS.has(origin)) {
    res.setHeader('access-control-allow-origin', origin);
    res.setHeader('vary', 'Origin');
  }

  if (req.method === 'OPTIONS') {
    res.setHeader('access-control-allow-methods', 'GET,POST,PUT,DELETE');
    res.setHeader('access-control-allow-headers', 'content-type,authorization');
    res.writeHead(204);
    res.end();
    return;
  }

  next();
}
```

CORS 不是安全边界，只控制浏览器能否跨域读取响应。真实鉴权仍然在服务端。

### 1.3 速率限制

简单的进程内限流器，按请求 IP 和路径统计窗口内次数：

```js
// src/shared/rate-limit.js
function createRateLimiter({ windowMs, max }) {
  const buckets = new Map();

  setInterval(() => buckets.clear(), windowMs).unref();

  return function rateLimit(req, res, next) {
    const key = `${req.socket.remoteAddress}:${req.url}`;
    const now = Date.now();
    const bucket = buckets.get(key) ?? { count: 0, resetAt: now + windowMs };

    if (now > bucket.resetAt) {
      bucket.count = 0;
      bucket.resetAt = now + windowMs;
    }

    bucket.count += 1;
    buckets.set(key, bucket);

    if (bucket.count > max) {
      res.setHeader('retry-after', Math.ceil((bucket.resetAt - now) / 1000));
      res.writeHead(429, { 'content-type': 'application/json' });
      res.end(JSON.stringify({ error: { code: 'RATE_LIMITED', message: '请求过于频繁' } }));
      return;
    }

    next();
  };
}
```

单进程限流只适合学习和单实例部署。多实例或分布式场景使用 Redis 等共享存储。

### 1.4 依赖审计

```bash
npm audit
npm audit fix
```

锁定依赖版本并定期运行 `npm audit`，在 CI 中对高危漏洞设置失败阈值。

## 2. 健康检查

区分存活和就绪：

```js
// src/health.js
function createHealthHandler({ db }) {
  return {
    live(req, res) {
      res.writeHead(200, { 'content-type': 'application/json' });
      res.end(JSON.stringify({ status: 'ok' }));
    },

    async ready(req, res) {
      try {
        await db.prepare('SELECT 1').get();
        res.writeHead(200, { 'content-type': 'application/json' });
        res.end(JSON.stringify({ status: 'ready', db: 'ok' }));
      } catch (error) {
        res.writeHead(503, { 'content-type': 'application/json' });
        res.end(JSON.stringify({ status: 'not-ready', db: 'unavailable' }));
      }
    },
  };
}
```

```text
/health/live   -> 进程是否活着
/health/ready  -> 是否能接收流量（数据库、缓存是否就绪）
```

数据库不可用时，`live` 仍返回 200，`ready` 返回 503，让编排系统停止向该实例派流量。

## 3. 部署方式

常见方式包括容器、虚拟机、Serverless 和平台即服务。无论方式如何，应用都应通过标准输出输出日志，通过环境变量或密钥服务注入配置，并提供健康检查。

### 3.1 Dockerfile

```dockerfile
FROM node:22-bookworm-slim

WORKDIR /app

COPY package*.json ./
RUN npm ci --omit=dev

COPY src ./src
COPY data ./data

USER node
EXPOSE 3000

CMD ["node", "src/server.js"]
```

容器需要注意：

- `npm ci` 使用锁定文件，保证依赖一致。
- 非 root 用户运行，降低容器逃逸风险。
- 进程信号转发由容器运行时处理，应用需实现优雅退出。
- 数据目录通过卷挂载，镜像内不保存持久数据。

### 3.2 部署流程

部署流程应包含：

```text
安装锁定依赖
  -> 类型检查
  -> 测试
  -> 构建
  -> 镜像扫描
  -> 数据库迁移
  -> 冒烟测试
  -> 灰度发布
  -> 监控观察
  -> 回滚预案
```

## 4. 可观测性

建立日志、指标和链路追踪三类信号：

- 日志：错误、审计和关键业务事件。
- 指标：请求量、延迟、错误率、CPU、内存和事件循环延迟。
- 追踪：跨 HTTP、数据库、缓存和消息队列传递 request ID 或 trace ID。

### 4.1 指标示例

```js
// src/shared/metrics.js
const counters = new Map();

export function incrementCounter(name) {
  counters.set(name, (counters.get(name) ?? 0) + 1);
}

export function renderMetrics() {
  const lines = ['# HELP requests_total HTTP 请求总数', '# TYPE requests_total counter'];
  for (const [name, value] of counters) {
    lines.push(`${name} ${value}`);
  }
  return lines.join('\n');
}
```

```js
// 在请求中间件中
function countRequest(req, res, start) {
  res.on('finish', () => {
    incrementCounter(`http_requests_total{method="${req.method}",status="${res.statusCode}"}`);
    incrementCounter('http_request_duration_ms');
  });
}
```

### 4.2 链路追踪字段

```js
function logRequest(req, res, start) {
  res.on('finish', () => {
    logger.info('request completed', {
      requestId: req.requestId,
      traceId: req.headers['x-trace-id'] ?? req.requestId,
      method: req.method,
      url: req.url,
      statusCode: res.statusCode,
      durationMs: Date.now() - start,
    });
  });
}
```

告警需要关联阈值、负责人、处理手册和恢复验证。

## 5. 可靠性模式

### 5.1 幂等键

避免客户端重试造成重复写入：

```js
// src/shared/idempotency.js
function createIdempotencyStore(db) {
  return {
    async get(key) {
      return db.prepare('SELECT * FROM idempotency WHERE key = ?').get(key);
    },
    async set(key, response) {
      db.prepare(
        'INSERT INTO idempotency (key, response, created_at) VALUES (?, ?, ?)',
      ).run(key, JSON.stringify(response), new Date().toISOString());
    },
  };
}

export async function handleWithIdempotency(req, handler) {
  const key = req.headers['idempotency-key'];

  if (key) {
    const cached = await store.get(key);
    if (cached) {
      return JSON.parse(cached.response);
    }
  }

  const result = await handler();
  if (key) await store.set(key, result);
  return result;
}
```

### 5.2 超时和取消

限制依赖服务占用时间：

```js
function withTimeout(promise, ms) {
  const controller = new AbortController();
  const timeout = setTimeout(() => controller.abort(), ms);

  return Promise.race([
    promise(controller.signal),
    new Promise((_, reject) => {
      controller.signal.addEventListener('abort', () => {
        reject(new Error(`依赖调用超过 ${ms}ms`));
      });
    }),
  ]).finally(() => clearTimeout(timeout));
}

const result = await withTimeout((signal) => fetch(url, { signal }), 3000);
```

### 5.3 重试退避

只对可恢复错误重试，并设置上限：

```js
async function retryWithBackoff(fn, { retries = 3, baseDelay = 200 } = {}) {
  let attempt = 0;

  while (true) {
    try {
      return await fn();
    } catch (error) {
      attempt += 1;

      const retryable =
        error.status === 429 ||
        error.status >= 500 ||
        error.name === 'AbortError' ||
        error.code === 'ECONNRESET';

      if (!retryable || attempt > retries) {
        throw error;
      }

      const delay = baseDelay * 2 ** (attempt - 1);
      console.log(`第 ${attempt} 次重试，等待 ${delay}ms`);
      await new Promise((resolve) => setTimeout(resolve, delay));
    }
  }
}
```

不要对 4xx（参数错误、权限不足）重试，这些错误重试也不会成功。

### 5.4 熔断和降级

防止故障依赖拖垮主服务：

```js
class CircuitBreaker {
  constructor({ failureThreshold = 5, resetTimeoutMs = 30_000 } = {}) {
    this.failureThreshold = failureThreshold;
    this.resetTimeoutMs = resetTimeoutMs;
    this.failures = 0;
    this.state = 'closed';
    this.lastFailureAt = 0;
  }

  async call(fn) {
    if (this.state === 'open') {
      if (Date.now() - this.lastFailureAt > this.resetTimeoutMs) {
        this.state = 'half-open';
      } else {
        throw new Error('熔断器开启，快速失败');
      }
    }

    try {
      const result = await fn();
      if (this.state === 'half-open') {
        this.state = 'closed';
        this.failures = 0;
      }
      return result;
    } catch (error) {
      this.failures += 1;
      this.lastFailureAt = Date.now();
      if (this.failures >= this.failureThreshold) {
        this.state = 'open';
      }
      throw error;
    }
  }
}
```

优雅降级明确缓存、只读和异步处理策略，让主流程在依赖故障时仍能提供部分能力。

## 6. 数据发布与备份

数据发布需要考虑迁移和备份：

- 数据库迁移应支持向前兼容，先发布兼容代码再执行破坏性清理。
- 迁移顺序：先发布兼容代码 -> 迁移 -> 启用新字段 -> 清理旧字段。
- 备份需要验证恢复演练、保留周期、加密和访问权限。
- 备份不是备份了就结束，必须定期演练恢复流程。

```bash
# 备份
pg_dump app > backup-$(date +%F).sql

# 恢复演练
createdb app-restore
psql app-restore < backup-$(date +%F).sql
```

## 7. 生产验收

- 能在 staging 环境验证迁移、配置和健康检查。
- 能定位一次请求从入口到数据库的完整日志链路。
- 能解释内存泄漏、事件循环阻塞和连接池耗尽的排查路径。
- 能执行回滚，并验证旧版本和数据兼容性。

排查路径模板：

```text
请求失败
  -> 查看结构化日志（requestId）
  -> 确认错误码：4xx（客户端）还是 5xx（服务器）
  -> 检查指标：错误率、延迟、事件循环延迟
  -> 检查依赖：数据库、缓存、外部 API
  -> 检查资源：CPU、内存、连接池
  -> 对比最近发布：回滚候选
```

## 动手任务

1. 为 API 设置请求体、连接和外部依赖超时。
2. 使用环境变量注入配置，准备 staging 配置检查表。
3. 为一次发布写出迁移、健康检查、冒烟测试和回滚步骤。
4. 模拟数据库不可用，确认服务的就绪状态、日志和用户提示。
5. 为写操作接入幂等键和速率限制，编写对应的 409 和 429 响应。

## 路线终点

完成 [综合实战：用户管理 API](./project-practice.md) 的全部验收项后，再进入 NestJS 路线，使用相同业务主题学习框架化实现。
