# 阶段一：Node.js 核心 API

**目标**：使用 Node.js 内置 API 完成 HTTP 服务、文件处理、事件通信和流式数据处理。

## 1. 文件、路径和 URL

路径必须使用 `node:path` 处理，文件操作优先使用 `node:fs/promises`。用户输入拼接路径前需要校验根目录，防止访问预期目录之外的文件。

```js
import path from 'node:path';
import { readFile } from 'node:fs/promises';

const root = path.resolve('public');
const filePath = path.resolve(root, 'index.html');

if (!filePath.startsWith(`${root}${path.sep}`)) {
  throw new Error('非法文件路径');
}

const html = await readFile(filePath, 'utf8');
```

## 2. 原生 HTTP 服务

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

server.listen(3000, '127.0.0.1');
```

真实项目需要路由匹配、请求体大小限制、超时、统一错误响应和日志。生产环境通常使用成熟 Web 框架承担路由与中间件组合。

## 3. EventEmitter

`EventEmitter` 适合进程内事件通知。事件处理器中的异常需要明确处理，跨进程或可靠投递应使用消息队列。

```js
import { EventEmitter } from 'node:events';

const bus = new EventEmitter();
bus.on('user.created', (user) => {
  console.log(`用户已创建: ${user.id}`);
});
bus.emit('user.created', { id: 'u1' });
```

## 4. Streams 与背压

Stream 适合处理大文件、网络响应和持续数据。背压表示消费者处理速度低于生产者时，生产者需要暂停写入。避免一次性把大文件读入内存。

```js
import { createReadStream, createWriteStream } from 'node:fs';
import { pipeline } from 'node:stream/promises';

await pipeline(
  createReadStream('input.log'),
  createWriteStream('output.log'),
);
```

`pipeline` 能够传播错误并在结束时完成清理。长时间 Stream 应支持 `AbortSignal`。

## 5. AbortController

取消是异步系统的必要能力，适用于 HTTP 请求、Stream、数据库查询和后台任务。取消后需要区分用户主动取消与真正失败。

```js
const controller = new AbortController();
const timeout = setTimeout(() => controller.abort(), 3000);

try {
  const response = await fetch('https://example.com/data', {
    signal: controller.signal,
  });
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
} finally {
  clearTimeout(timeout);
}
```

## 6. URL、Crypto 与请求体

Node.js 的 `URL` 用于可靠解析地址，`node:crypto` 提供随机数、摘要和密码学原语。密码存储应使用专用密码哈希方案和成熟库，通用摘要不适合作为密码哈希。

HTTP 请求体是 Stream，需要限制大小并处理 `aborted`、解析失败和超时：

```js
async function readJsonBody(req, maxBytes = 1_000_000) {
  const chunks = [];
  let size = 0;

  for await (const chunk of req) {
    size += chunk.length;
    if (size > maxBytes) throw new Error('请求体过大');
    chunks.push(chunk);
  }

  return JSON.parse(Buffer.concat(chunks).toString('utf8'));
}
```

真实服务还应校验 `content-type`，拒绝超大请求，并将解析异常映射为稳定的 4xx 响应。

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

## 进入下一阶段的条件

你能够用 curl 完成一次 GET 和 POST 请求，能够解释请求体为何是 Stream，并且能区分路由不存在、请求体无效和服务内部异常。
