# 阶段零：Node.js 运行时与 JavaScript 基础

**目标**：理解 Node.js 与浏览器的运行边界，掌握模块、异步、错误和命令行基础，能够从零初始化一个可运行的 Node.js 项目。

## 1. Node.js 是什么

Node.js 是基于 V8 的 JavaScript 运行时，提供文件系统、网络、进程、加密、Streams 和测试等服务端 API。Node.js 适合 I/O 密集型服务；CPU 密集型任务需要拆分、Worker 或独立计算服务。

Node.js 进程通常包含一个 JavaScript 主线程和事件循环。异步 I/O 完成后，回调会在事件循环的适当阶段执行。长时间同步计算会阻塞所有请求。

### 1.1 初始化一个项目

从零创建一个 Node.js 项目：

```bash
mkdir first-node-app
cd first-node-app
npm init -y
```

`npm init -y` 生成默认的 `package.json`。手动补齐启动、测试和类型检查命令：

```json
{
  "name": "first-node-app",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node src/index.mjs",
    "dev": "node --watch src/index.mjs",
    "test": "node --test"
  }
}
```

常用脚本命令：

```bash
npm run dev     # 开发模式，文件变化自动重启
npm start       # 运行 start 脚本
npm run test    # 运行 test 脚本
```

`--watch` 让 `dev` 模式监听文件变化并自动重启，适合学习阶段的本地开发。

## 2. 模块系统

现代项目需要明确 ESM 和 CommonJS 的边界。

| 模块系统 | 导入 | 导出 | 适用 |
| --- | --- | --- | --- |
| ESM | `import` | `export` | 新项目、浏览器兼容、异步加载 |
| CommonJS | `require` | `module.exports` | 存量项目、部分 npm 包 |

决定解释方式的三个来源：

- `package.json` 的 `"type": "module"`：整个项目按 ESM 处理。
- 文件扩展名 `.mjs`（始终 ESM）和 `.cjs`（始终 CommonJS）。
- 缺少配置时，默认按 CommonJS 处理。

```js
// math.mjs
export function add(a, b) {
  return a + b;
}

export const VERSION = '1.0.0';
```

```js
// app.mjs
import { add, VERSION } from './math.mjs';

console.log(add(2, 3));
console.log(VERSION);
```

内置模块建议使用明确前缀：

```js
import { readFile } from 'node:fs/promises';
import path from 'node:path';
```

`node:` 前缀明确表示这是内置模块，避免与第三方包同名冲突。

## 3. 异步模型

Node.js 支持回调、Promise 和 `async/await`。新代码优先使用 Promise API，并在边界处捕获异常。

### 3.1 三个 Promise 任务组合

并行任务使用 `Promise.all`，允许部分失败时使用 `Promise.allSettled`：

```js
const [users, posts] = await Promise.all([
  fetchUsers(),
  fetchPosts(),
]);
```

```js
const results = await Promise.allSettled([fetchA(), fetchB()]);

for (const result of results) {
  if (result.status === 'fulfilled') {
    console.log('成功', result.value);
  } else {
    console.log('失败', result.reason);
  }
}
```

### 3.2 边界处捕获异常

`async/await` 的异常不会自动处理，必须主动捕获并保留原因链：

```js
import { readFile } from 'node:fs/promises';

async function readConfig(filePath) {
  try {
    const content = await readFile(filePath, 'utf8');
    return JSON.parse(content);
  } catch (error) {
    throw new Error(`配置读取失败: ${filePath}`, { cause: error });
  }
}
```

`cause` 保留原始错误，排查时可以沿着原因链找到真正的失败点。

### 3.3 事件循环执行顺序

理解同步代码、微任务和宏任务的顺序，是解释许多奇怪行为的起点：

```js
console.log('1: 同步代码');

setTimeout(() => console.log('5: 宏任务 setTimeout'), 0);

Promise.resolve()
  .then(() => console.log('3: 微任务 Promise'))
  .then(() => console.log('4: 微任务 Promise 链'));

console.log('2: 同步代码结束');
```

输出顺序：

```text
1: 同步代码
2: 同步代码结束
3: 微任务 Promise
4: 微任务 Promise 链
5: 宏任务 setTimeout
```

规则：同步代码先全部执行完，再清空微任务队列，最后才执行宏任务。

## 4. `process`、环境变量与 URL

`process.argv` 用于读取命令行参数，`process.env` 用于读取环境变量，`process.cwd()` 表示当前工作目录。环境变量默认都是字符串，启动时应完成类型转换和校验。

### 4.1 端口校验与 URL 解析

```js
const port = Number.parseInt(process.env.PORT ?? '3000', 10);

if (!Number.isInteger(port) || port < 1 || port > 65535) {
  throw new Error('PORT 必须是有效端口');
}

const input = process.argv[2] ?? 'https://example.com';
const url = new URL(input);
console.log({
  protocol: url.protocol,
  host: url.host,
  pathname: url.pathname,
  searchParams: Object.fromEntries(url.searchParams),
});
```

运行方式：

```bash
PORT=8080 node src/url-tool.mjs 'https://example.com/api?page=2'
```

### 4.2 完整 CLI 案例

一个读取 JSON 配置并输出报告的命令行工具，串起本阶段所有知识点：

```js
// src/file-report.mjs
import { readFile } from 'node:fs/promises';
import path from 'node:path';

async function main() {
  const filePath = process.argv[2];

  if (!filePath) {
    console.error('用法: node src/file-report.mjs <json 文件路径>');
    process.exit(1);
  }

  try {
    const resolved = path.resolve(process.cwd(), filePath);
    const content = await readFile(resolved, 'utf8');
    const data = JSON.parse(content);
    console.log(`键名列表: ${Object.keys(data).join(', ')}`);
    console.log(`条目数量: ${Array.isArray(data) ? data.length : 1}`);
  } catch (error) {
    console.error(`读取失败: ${filePath}`);
    console.error(`原因: ${error.cause?.message ?? error.message}`);
    process.exit(1);
  }
}

main();
```

运行和验证：

```bash
node src/file-report.mjs config.json        # 正常输出
node src/file-report.mjs missing.json       # 错误退出码 1
node src/file-report.mjs                    # 用法提示
```

三个入口场景分别对应：解析成功、文件不存在、缺少参数。

### 4.3 环境变量安全

不要把密钥写入代码、日志或版本库。项目应提供 `.env.example` 说明变量名和格式，由部署环境注入真实值：

```env
# .env.example
PORT=3000
DATABASE_URL=postgresql://user:password@localhost/db
```

真实值由本地文件或部署平台注入，`.env` 加入 `.gitignore`。

## 5. 错误处理模式

### 5.1 区分错误类型

```js
async function safeCall(fn) {
  try {
    const value = await fn();
    return { ok: true, value };
  } catch (error) {
    if (error.name === 'AbortError') {
      return { ok: false, reason: 'cancelled' };
    }
    if (error.code === 'ENOENT') {
      return { ok: false, reason: 'not-found' };
    }
    return { ok: false, reason: 'unknown', error };
  }
}
```

`error.code` 是 Node.js 系统错误的稳定标识，`ENOENT` 表示文件不存在。

### 5.2 顶层 await

ESM 支持在模块顶层直接 `await`：

```js
// top-level-await.mjs
const data = await fetchConfig();
console.log(data);
```

这要求文件属于 ESM（`"type": "module"` 或 `.mjs` 扩展名）。

## 6. Node.js 版本治理

Node.js 版本需要与框架、原生依赖、容器镜像和 CI 保持一致。项目应明确：

- `engines.node` 或版本管理文件中的最低版本。

```json
{
  "engines": {
    "node": ">=20"
  }
}
```

- CI、开发机和生产镜像使用的版本来源。
- 升级前需要运行的类型检查、测试、构建和冒烟测试。
- 原生模块重新编译、弃用 API 和 ESM/CJS 行为变化的检查项。

## 阶段验收

- 能解释事件循环、Promise 微任务和同步阻塞的影响。
- 能在 ESM 项目中拆分模块并正确导入内置 API。
- 能为文件读取和 JSON 解析设计错误处理。
- 能使用命令行参数和环境变量运行一个脚本。
- 能校验端口、解析 URL，并说明 Node.js 版本治理方式。

## 动手任务

1. 创建 `src/cli.mjs`，读取一个命令行参数并输出规范化 URL。
2. 创建 `src/config.mjs`，读取 `PORT`，对空值、非数字和越界值分别报错。
3. 创建 `src/file-report.mjs`，异步读取一个 JSON 文件并输出键名列表。
4. 故意传入不存在的文件，记录错误原因和用户可读提示。
5. 运行一次事件循环顺序示例，写出你预测的输出，再与实际对比。

## 进入下一阶段的条件

你能够说明一段代码运行在 Node.js 还是浏览器、Promise 异常如何传播、同步计算为何会阻塞请求，并且能够独立运行上述三个脚本。
