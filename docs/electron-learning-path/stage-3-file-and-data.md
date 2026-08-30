# 阶段三：文件、数据与前端集成

**目标**：掌握本地文件读写、数据持久化与前端框架集成，构建有真实功能的桌面应用。

## 1. 系统路径

用 `app.getPath` 获取各系统正确的数据目录，不要写死路径：

```js
const { app } = require("electron");

app.getPath("userData");     // 应用私有数据目录（按平台区分）
app.getPath("documents");    // 用户文档目录
app.getPath("downloads");    // 下载目录
app.getPath("temp");         // 临时目录
```

`userData` 是存放配置与数据库的首选位置。

## 2. 文件读写

把文件能力封装进 IPC，主进程负责 I/O：

```js
// main.js
const { ipcMain, app } = require("electron");
const fs = require("node:fs/promises");
const path = require("node:path");

ipcMain.handle("note:list", async () => {
  const dir = path.join(app.getPath("documents"), "MyNotes");
  await fs.mkdir(dir, { recursive: true });
  const files = await fs.readdir(dir);
  return files.filter((f) => f.endsWith(".md"));
});

ipcMain.handle("note:save", async (_e, filename, content) => {
  const dir = path.join(app.getPath("documents"), "MyNotes");
  await fs.mkdir(dir, { recursive: true });
  const safeName = path.basename(filename); // 防路径穿越
  await fs.writeFile(path.join(dir, safeName), content, "utf-8");
  return { ok: true };
});
```

- 用 `path.basename` 过滤用户输入，防止路径穿越。
- 长 I/O 用 Promise 版 fs。

## 3. 数据持久化

轻量配置用 JSON 文件：

```js
ipcMain.handle("settings:load", async () => {
  const file = path.join(app.getPath("userData"), "settings.json");
  try {
    return JSON.parse(await fs.readFile(file, "utf-8"));
  } catch {
    return {};
  }
});

ipcMain.handle("settings:save", async (_e, settings) => {
  const file = path.join(app.getPath("userData"), "settings.json");
  await fs.writeFile(file, JSON.stringify(settings, null, 2), "utf-8");
  return { ok: true };
});
```

结构化数据可选用 SQLite（主进程内）或更低层的持久化方案，原则是数据放 `userData`。

## 4. 前端框架集成

Electron 渲染进程可以加载任何前端构建产物：

```text
Vite/Webpack 构建 -> 输出 dist -> 主进程 loadFile(dist/index.html)
```

以 Vite + React 为例，`electron-vite` 统一管理主进程、preload 与渲染进程三份构建。开发时渲染进程跑 dev server，主进程 `loadURL`；生产时 `loadFile` 构建产物。

```js
// main.js（electron-vite 风格）
const { app, BrowserWindow } = require("electron");

function createWindow() {
  const win = new BrowserWindow({
    webPreferences: {
      preload: path.join(__dirname, "../preload/index.js"),
    },
  });

  if (process.env.ELECTRON_RENDERER_URL) {
    win.loadURL(process.env.ELECTRON_RENDERER_URL);
  } else {
    win.loadFile(path.join(__dirname, "../renderer/index.html"));
  }
}
```

## 5. 集成最佳实践

- preload 暴露的 API 用 TypeScript 声明全局类型，界面端类型安全。
- 渲染进程只通过 `window.desktopAPI` 访问能力，不直接 import Electron。
- 文件变更用主进程事件推送给界面刷新。

## 6. 性能注意

- 大文件读写放主进程或子进程，避免阻塞渲染。
- 图片、富文本按需加载。
- 频繁的 IPC 调用可合并批量，降低开销。

## 7. 动手任务

1. 封装 `note:list`、`note:save`、`note:delete` 三个 IPC 通道。
2. 实现基于 `userData` 的 JSON 设置持久化。
3. 用 `electron-vite` 初始化 React + Electron 项目。
4. 让界面能列出、保存、删除 Markdown 笔记。
5. 在主进程完成全部文件 I/O，验证渲染进程无 Node 能力。

## 阶段三验收

- 能正确使用系统路径并读写文件。
- 能持久化配置与数据。
- 能集成前端框架构建渲染进程。
- 能说明 IPC 与性能注意点。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 写入路径不存在 | 先 `mkdir { recursive: true }` |
| 文件名带路径 | 用 `path.basename` 过滤 |
| 渲染进程加载空白 | dev 环境 URL 与生产文件路径切换逻辑 |
| 数据丢失 | 确认写入 `userData` 且路径正确 |
| 大文件卡顿 | 移到主进程或子进程处理 |
| preload 类型未识别 | 声明 `window.desktopAPI` 全局类型 |

## 进入下一阶段的条件

你能够构建有真实文件与数据能力的应用。此时进入 [阶段四：打包发布与生产](./stage-4-production.md)。
