# 阶段零：Electron 基础与工具链

**目标**：理解 Electron 的双进程模型，初始化项目，创建第一个窗口并理解应用生命周期。

## 1. Electron 是什么

Electron 用 Web 技术构建跨平台桌面应用，内嵌 Chromium 渲染页面、内置 Node.js 提供系统能力。一个 Electron 应用由三类进程组成：

| 进程 | 职责 |
|---|---|
| 主进程（Main） | 应用生命周期、窗口管理、系统能力，Node 环境 |
| 渲染进程（Renderer） | 页面 UI，即一个 Chromium 页面 |
| Preload | 主进程与渲染进程之间的安全桥 |

```text
主进程(BrowserWindow 管理)
   ├── 渲染进程 A(窗口)
   ├── 渲染进程 B(窗口)
   └── preload(每个窗口注入)
```

## 2. 初始化项目

```bash
mkdir hello-electron && cd hello-electron
npm init -y
npm install --save-dev electron
```

设置 `package.json` 的入口与启动脚本：

```json
{
  "name": "hello-electron",
  "main": "main.js",
  "scripts": {
    "start": "electron ."
  }
}
```

用 `npx electron -v` 验证安装。

## 3. 主进程：创建窗口

```js
// main.js
const { app, BrowserWindow } = require("electron");
const path = require("node:path");

function createWindow() {
  const win = new BrowserWindow({
    width: 900,
    height: 650,
    webPreferences: {
      preload: path.join(__dirname, "preload.js"),
    },
  });
  win.loadFile("index.html");
}

app.whenReady().then(() => {
  createWindow();

  app.on("activate", () => {
    if (BrowserWindow.getAllWindows().length === 0) createWindow();
  });
});

app.on("window-all-closed", () => {
  if (process.platform !== "darwin") app.quit();
});
```

- `app.whenReady()` 后创建窗口。
- `window-all-closed` 时非 macOS 退出应用。
- `activate` 处理 macOS 点击 Dock 图标重建窗口。

## 4. 渲染进程与 HTML

```html
<!-- index.html -->
<!doctype html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>Hello Electron</title>
  </head>
  <body>
    <h1>你好，Electron</h1>
    <p id="info">渲染进程运行中</p>
  </body>
</html>
```

渲染进程就是普通 Web 页面，可加载本地文件或远程 URL。

## 5. 应用生命周期

- `ready`：初始化完成，可创建窗口。
- `activate`：macOS 重新激活。
- `window-all-closed`：所有窗口关闭。
- `before-quit` / `will-quit`：退出前清理。

主进程是应用的中枢，长期运行，负责协调所有窗口。

## 6. 开发者工具与调试

```js
win.webContents.openDevTools(); // 打开渲染进程 DevTools
```

主进程日志输出到终端；渲染进程日志在 DevTools Console。用 `--inspect` 可调试主进程：

```bash
electron . --inspect
```

## 7. 进程角色小结

- 只有主进程能调用系统 API（窗口、菜单、托盘、对话框）。
- 渲染进程只能做页面能做的事，系统能力经 IPC 请求主进程。
- preload 是两者的桥梁，后面阶段详解。

## 8. 动手任务

1. 初始化项目并安装 Electron。
2. 配置 `main` 入口与启动脚本。
3. 用主进程创建窗口并加载 HTML。
4. 验证 macOS/Windows/Linux 下的生命周期行为差异（至少理解代码意图）。
5. 打开 DevTools 观察渲染进程。

## 阶段零验收

- 能解释主进程、渲染进程、preload 的职责。
- 能初始化项目并运行第一个窗口。
- 能理解 `ready`、`window-all-closed`、`activate` 的含义。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| `electron` 命令不存在 | 确认 `npm install --save-dev electron` |
| 窗口未出现 | 确认 `main` 字段与 `app.whenReady` |
| 白屏 | HTML 路径错误或 `loadFile` 路径不对 |
| 报 `module not found` | 确认从项目根目录启动 |
| 运行很慢 | 首次运行需下载 Chromium 二进制 |

## 进入下一阶段的条件

你能够运行一个窗口并理解进程模型。此时进入 [阶段一：进程模型与 IPC 通信](./stage-1-processes-and-ipc.md)。
