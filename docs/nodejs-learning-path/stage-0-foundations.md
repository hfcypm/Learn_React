# 阶段零：Node.js 运行时与 JavaScript 基础

**目标**：理解 Node.js 与浏览器的运行边界，掌握模块、异步、错误和命令行基础。

## 1. Node.js 是什么

Node.js 是基于 V8 的 JavaScript 运行时，提供文件系统、网络、进程、加密、Streams 和测试等服务端 API。Node.js 适合 I/O 密集型服务；CPU 密集型任务需要拆分、Worker 或独立计算服务。

Node.js 进程通常包含一个 JavaScript 主线程和事件循环。异步 I/O 完成后，回调会在事件循环的适当阶段执行。长时间同步计算会阻塞所有请求。

## 2. 模块系统

现代项目需要明确 ESM 和 CommonJS 的边界。ESM 使用 `import/export`，CommonJS 使用 `require/module.exports`。项目通过 `package.json` 的 `type`、文件扩展名和构建工具决定解释方式。

```js
// math.mjs
export function add(a, b) {
  return a + b;
}
```

```js
// app.mjs
import { add } from './math.mjs';

console.log(add(2, 3));
```

内置模块建议使用明确前缀：

```js
import { readFile } from 'node:fs/promises';
import path from 'node:path';
```

## 3. 异步模型

Node.js 支持回调、Promise 和 `async/await`。新代码优先使用 Promise API，并在边界处捕获异常：

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

并行任务使用 `Promise.all`，允许部分失败时使用 `Promise.allSettled`。外部请求需要设置超时或 `AbortSignal`，避免请求永久占用资源。

## 4. 环境与脚本

```bash
# 查看 Node.js 版本
node --version

# 执行脚本
node src/index.mjs

# 初始化项目
npm init -y
```

`package.json` 应记录启动、测试、类型检查和构建命令。依赖使用锁定文件，生产安装区分运行时依赖和开发依赖。

## 5. `process`、环境变量与 URL

`process.argv` 用于读取命令行参数，`process.env` 用于读取环境变量，`process.cwd()` 表示当前工作目录。环境变量默认都是字符串，启动时应完成类型转换和校验。

```js
const port = Number.parseInt(process.env.PORT ?? '3000', 10);

if (!Number.isInteger(port) || port < 1 || port > 65535) {
  throw new Error('PORT 必须是有效端口');
}

const input = process.argv[2] ?? 'https://example.com';
const url = new URL(input);
console.log({ protocol: url.protocol, host: url.host });
```

不要把密钥写入代码、日志或版本库。项目应提供 `.env.example` 说明变量名和格式，由部署环境注入真实值。

## 6. Node.js 版本治理

Node.js 版本需要与框架、原生依赖、容器镜像和 CI 保持一致。项目应明确：

- `engines.node` 或版本管理文件中的最低版本。
- CI、开发机和生产镜像使用的版本来源。
- 升级前需要运行的类型检查、测试、构建和冒烟测试。
- 原生模块重新编译、弃用 API 和 ESM/CJS 行为变化的检查项。

## 阶段验收

- 能解释事件循环、Promise 微任务和同步阻塞的影响。
- 能在 ESM 项目中拆分模块并正确导入内置 API。
- 能为文件读取和 JSON 解析设计错误处理。
- 能使用命令行参数和环境变量运行一个脚本。
- 能校验端口、解析 URL，并说明 Node.js 版本治理方式。
