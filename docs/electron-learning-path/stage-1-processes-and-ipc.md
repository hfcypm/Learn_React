# 阶段一：进程模型与 IPC 通信

**目标**：掌握 preload、contextBridge 与 ipcMain/ipcRenderer，构建安全、可维护的主渲染通信。

## 1. 为什么需要 IPC

渲染进程不能直接调用 Node 与系统 API（安全设计）。需要系统能力时，渲染进程通过 IPC 请求主进程执行，主进程把结果回传。

```text
渲染进程 --invoke--> preload --ipcRenderer.invoke--> 主进程 ipcMain.handle
渲染进程 <--返回--- preload <--返回值------------- 主进程
```

## 2. preload 与 contextBridge

preload 运行在隔离上下文（Isolated World），用 `contextBridge` 把安全 API 暴露给页面主世界：

```js
// preload.js
const { contextBridge, ipcRenderer } = require("electron");

contextBridge.exposeInMainWorld("desktopAPI", {
  getAppVersion: () => ipcRenderer.invoke("app:version"),
  saveFile: (name, content) => ipcRenderer.invoke("file:save", name, content),
});
```

页面中通过 `window.desktopAPI.getAppVersion()` 调用。preload 是渲染进程唯一能接触 Node 的地方，必须只暴露最小 API。

## 3. 主进程处理请求

```js
// main.js
const { ipcMain } = require("electron");
const { app } = require("electron");

ipcMain.handle("app:version", () => app.getVersion());

ipcMain.handle("file:save", async (_event, name, content) => {
  // 主进程拥有 fs 能力，这里执行写入
  return { ok: true, name };
});
```

- `ipcMain.handle(channel, handler)` 处理来自渲染进程的请求。
- channel 命名约定：`作用域:动作`（如 `file:save`），避免冲突。
- handler 可以是 async，返回 Promise。

## 4. 渲染进程调用

```js
// renderer.js（页面主世界）
const version = await window.desktopAPI.getAppVersion();
const result = await window.desktopAPI.saveFile("notes.md", "# 标题");
```

```html
<button id="save">保存</button>
<script src="./renderer.js"></script>
```

## 5. 主进程向渲染进程推送事件

需要推送（如文件变化、进度）时，主进程用 `webContents.send`：

```js
// main.js
ipcMain.handle("job:start", (event) => {
  let p = 0;
  const timer = setInterval(() => {
    p += 10;
    event.sender.send("job:progress", p);
    if (p >= 100) clearInterval(timer);
  }, 200);
  return "started";
});
```

preload 包装监听：

```js
// preload.js
contextBridge.exposeInMainWorld("desktopAPI", {
  onJobProgress: (callback) =>
    ipcRenderer.on("job:progress", (_e, p) => callback(p)),
});
```

## 6. 安全基线

- 开启上下文隔离与沙箱（新版本默认开启）：

```js
webPreferences: {
  preload: path.join(__dirname, "preload.js"),
  contextIsolation: true,
  nodeIntegration: false,
  sandbox: true,
}
```

- 不要在渲染进程开启 `nodeIntegration`。
- 不要用 `ipcRenderer` 暴露给页面全局，只通过 contextBridge 暴露窄接口。
- 主进程校验所有传入参数，不信任渲染进程输入。

## 7. 通信模式选择

| 场景 | 模式 |
|---|---|
| 请求/响应 | `ipcRenderer.invoke` + `ipcMain.handle` |
| 单向通知 | `ipcRenderer.send` + `ipcMain.on` |
| 主进程推送 | `webContents.send` + preload 包装监听 |

## 8. 动手任务

1. 在 preload 中暴露一个返回应用版本的 API。
2. 主进程处理该请求并返回版本号。
3. 增加一个 `file:save` 通道，主进程用 `fs` 写入 `app.getPath("documents")`。
4. 用 `webContents.send` 推送进度事件并在页面显示。
5. 验证 `contextIsolation` 开启时无法在页面直接 `require`。

## 阶段一验收

- 能解释 preload 与 contextBridge 的作用。
- 能完成 invoke/handle 请求响应与 send/on 推送。
- 能按安全基线配置 webPreferences。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| `window.desktopAPI` 未定义 | preload 路径错误或 contextBridge 未执行 |
| invoke 无返回 | channel 名不一致或 handler 未注册 |
| 页面报 `require is not defined` | 正常现象，说明沙箱生效；应走 preload |
| 监听不到推送 | send 的窗口与监听窗口是同一个实例 |
| contextBridge 暴露函数未生效 | 确认 preload 在 `webPreferences.preload` 注册 |

## 进入下一阶段的条件

你能够安全地完成双向通信。此时进入 [阶段二：窗口与系统集成](./stage-2-windows-and-system.md)。
