# 阶段一：Node.js 核心 API

**目标**：使用 Node.js 内置 API 完成 HTTP 服务、文件处理、事件通信和流式数据处理，能够独立运行一个多路由的 JSON 服务。

## 1. 文件、路径和 URL

路径必须使用 `node:path` 处理，文件操作优先使用 `node:fs/promises`。用户输入拼接路径前需要校验根目录，防止访问预期目录之外的文件。

### 1.1 路径拼接与目录穿越防护

```js
import path from 'node:path';
import { readFile } from 'node:fs/promises';

const root = path.resolve('public');

function resolveSafe(relativePath) {
  const filePath = path.resolve(root, relativePath);

  if (!filePath.startsWith(`${root}${path.sep}`)) {
    throw new Error('非法文件路径');
  }

  return filePath;
}

const html = await readFile(resolveSafe('index.html'), 'utf8');
```

`startsWith(root + sep)` 检查确保解析后的路径仍然位于 `public` 目录内，防止 `../` 越权访问其他目录。

### 1.2 URL 解析

`URL` 用于可靠解析地址，比字符串拼接更安全：

```js
const url = new URL('https://example.com/api/users?page=2&pageSize=20');

console.log(url.pathname);             // /api/users
console.log(url.searchParams.get('page'));   // 2
console.log(url.searchParams.get('pageSize')); // 20
```

## 2. 原生 HTTP 服务

### 2.1 第一个 HTTP 服务

```js
import http from 'node:http';

const server = http.createServer((req, res) => {
  if (req.method === 'GET' && req.url === '/health') {
    res.writeHead(200, { 'content-type': 'application/json' });
    res.end(JSON.stringify({ status: 'ok' }));
    return;
  }

  res.writeHead(404, { 'content-type': 'application/json' });
  res.end(JSON.stringify({ error: 'Not Found' }));
});

server.listen(3000, '127.0.0.1', () => {
  console.log('服务已启动: http://127.0.0.1:3000');
});
```

运行后验证：

```bash
node src/server.mjs
curl -i http://127.0.0.1:3000/health
```

### 2.2 完整可运行的多路由服务

把路由匹配、统一响应和错误处理放进一个文件，形成阶段一的完整案例：

```js
// src/server.mjs
import http from 'node:http';

function sendJson(res, statusCode, data) {
  res.writeHead(statusCode, { 'content-type': 'application/json' });
  res.end(JSON.stringify(data));
}

const routes = [
  { method: 'GET', path: '/health', handler: () => ({ status: 'ok' }) },
  {
    method: 'GET',
    path: '/echo',
    handler: (req) => ({
      method: req.method,
      url: req.url,
      userAgent: req.headers['user-agent'] ?? null,
    }),
  },
];

const server = http.createServer(async (req, res) => {
  try {
    const route = routes.find(
      (r) => r.method === req.method && r.path === req.url,
    );

    if (!route) {
      sendJson(res, 404, { error: 'Not Found' });
      return;
    }

    const data = route.handler(req);
    sendJson(res, 200, data);
  } catch (error) {
    console.error(error);
    sendJson(res, 500, { error: 'Internal Server Error' });
  }
});

server.listen(3000, '127.0.0.1', () => {
  console.log('服务已启动: http://127.0.0.1:3000');
});
```

验证：

```bash
curl -i http://127.0.0.1:3000/health
curl -i http://127.0.0.1:3000/echo
curl -i http://127.0.0.1:3000/not-exist   # 404
```

真实项目需要路由匹配、请求体大小限制、超时、统一错误响应和日志。生产环境通常使用成熟 Web 框架承担路由与中间件组合，本阶段先理解原生机制。

## 3. Buffer 与编码

Buffer 用于处理二进制数据，是文件、网络和加密的基础：

```js
import { Buffer } from 'node:buffer';

const utf8 = Buffer.from('你好', 'utf8');
console.log(utf8.length);          // 6（3 个汉字 = 6 字节）
console.log(utf8.toString('hex')); // 十六进制表示
console.log(utf8.toString('base64'));
console.log(Buffer.from('aGVsbG8=', 'base64').toString('utf8')); // hello
```

请求体、Stream 和加密模块产生的都是 Buffer。解码用 `toString('utf8')`，需要注意多字节字符可能跨 chunk 断裂，`TextDecoder` 可以缓解这一问题：

```js
const decoder = new TextDecoder();
const text = decoder.decode(buffer);
```

## 4. Crypto 基础

`node:crypto` 提供随机数和摘要。通用摘要（如 SHA-256）适合校验和数据指纹，**不适合**作为密码哈希，密码哈希需要使用专用算法。

### 4.1 随机数

```js
import { randomBytes } from 'node:crypto';

const token = randomBytes(16).toString('hex');
console.log(token); // 32 位十六进制随机串
```

适合生成请求 ID、重置令牌和盐值。

### 4.2 摘要

```js
import { createHash } from 'node:crypto';

function hashContent(content) {
  return createHash('sha256').update(content).digest('hex');
}

console.log(hashContent('hello')); // 相同输入得到相同输出
```

适合内容校验和缓存键，不适合密码存储。

## 5. EventEmitter

`EventEmitter` 适合进程内事件通知。事件处理器中的异常需要明确处理，跨进程或可靠投递应使用消息队列。

```js
import { EventEmitter } from 'node:events';

const bus = new EventEmitter();

bus.on('user.created', (user) => {
  console.log(`用户已创建: ${user.id}`);
});

bus.once('app.started', () => {
  console.log('只触发一次');
});

bus.emit('user.created', { id: 'u1' });
bus.emit('user.created', { id: 'u2' });
bus.emit('app.started');
bus.emit('app.started'); // 不输出，once 只处理一次
```

`once` 只监听一次，适合初始化通知和一次性预热逻辑。

## 6. Streams 与背压

Stream 适合处理大文件、网络响应和持续数据。背压表示消费者处理速度低于生产者时，生产者需要暂停写入。避免一次性把大文件读入内存。

### 6.1 文件复制

```js
import { createReadStream, createWriteStream } from 'node:fs';
import { pipeline } from 'node:stream/promises';

await pipeline(
  createReadStream('input.log'),
  createWriteStream('output.log'),
);
```

`pipeline` 能够传播错误并在结束时完成清理。

### 6.2 流式转换

```js
import { createReadStream } from 'node:fs';
import { Transform } from 'node:stream';

const upperCase = new Transform({
  transform(chunk, encoding, callback) {
    callback(null, chunk.toString('utf8').toUpperCase());
  },
});

for await (const chunk of createReadStream('input.txt').pipe(upperCase)) {
  process.stdout.write(chunk);
}
```

### 6.3 处理不可信输入的安全版本

本阶段先用带 `statusCode` 和 `code` 的普通错误对象区分错误类型；阶段二会把它统一为 `AppError` 类：

```js
function bodyError(statusCode, code, message) {
  const error = new Error(message);
  error.statusCode = statusCode;
  error.code = code;
  return error;
}

async function readJsonBody(req, maxBytes = 1_000_000) {
  const contentType = req.headers['content-type'] ?? '';

  if (!contentType.includes('application/json')) {
    throw bodyError(415, 'UNSUPPORTED_MEDIA_TYPE', '需要 application/json');
  }

  const chunks = [];
  let size = 0;

  for await (const chunk of req) {
    size += chunk.length;
    if (size > maxBytes) {
      throw bodyError(413, 'PAYLOAD_TOO_LARGE', '请求体过大');
    }
    chunks.push(chunk);
  }

  try {
    return JSON.parse(Buffer.concat(chunks).toString('utf8'));
  } catch (error) {
    throw bodyError(400, 'INVALID_JSON', '请求体不是合法 JSON');
  }
}
```

真实服务还应校验 `content-type`，拒绝超大请求，并将解析异常映射为稳定的 4xx 响应。区分客户端错误（400/413/415）与服务器错误（500）是这一阶段的关键能力。

## 7. AbortController

取消是异步系统的必要能力，适用于 HTTP 请求、Stream、数据库查询和后台任务。取消后需要区分用户主动取消与真正失败。

### 7.1 带超时的外部请求

```js
const controller = new AbortController();
const timeout = setTimeout(() => controller.abort(), 3000);

try {
  const response = await fetch('https://example.com/data', {
    signal: controller.signal,
  });
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
} catch (error) {
  if (error.name === 'AbortError') {
    console.log('请求被取消（超时）');
  } else {
    throw error;
  }
} finally {
  clearTimeout(timeout);
}
```

### 7.2 可取消的服务器响应

```js
async function sendWithTimeout(res, data, ms) {
  const controller = new AbortController();
  const timeout = setTimeout(() => controller.abort(), ms);

  try {
    await new Promise((resolve, reject) => {
      res.on('close', resolve);
      controller.signal.addEventListener('abort', () => {
        reject(new Error('响应超时'));
      });
    });
  } finally {
    clearTimeout(timeout);
  }
}
```

## 阶段验收

- 能写带健康检查的原生 HTTP 服务。
- 能使用 `pipeline` 处理大文件并传播错误。
- 能解释 EventEmitter、Stream 和消息队列的边界。
- 能为外部请求增加超时和取消。
- 能安全读取有限大小的 JSON 请求体，并区分解析失败与服务错误。

## 动手任务

1. 为 `/health` 增加 `content-type`、状态码和 JSON 响应。
2. 增加一个只读 `/files/:name` 路由，限制文件根目录和响应大小。
3. 使用 `readJsonBody` 接收 POST 请求，并处理空请求体、错误 JSON 和超大请求体。
4. 使用 `AbortController` 为一个外部请求增加 3 秒超时。
5. 为多路由服务增加一条 `/sum?a=1&b=2` 路由，校验参数并返回计算结果。

## 进入下一阶段的条件

你能够用 curl 完成一次 GET 和 POST 请求，能够解释请求体为何是 Stream，并且能区分路由不存在、请求体无效和服务内部异常。
