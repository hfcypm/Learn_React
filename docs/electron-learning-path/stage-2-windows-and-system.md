# 阶段二：窗口与系统集成

**目标**：掌握多窗口管理、菜单、托盘与系统通知，构建完整的桌面外壳。

## 1. 多窗口管理

用工厂函数统一创建窗口，主进程持有引用：

```js
// windows.js
const { BrowserWindow } = require("electron");

function createMainWindow() {
  const win = new BrowserWindow({ width: 960, height: 700 });
  win.loadFile("index.html");
  return win;
}

function createNoteWindow(payload) {
  const win = new BrowserWindow({ width: 640, height: 480, parent: null });
  win.loadFile("note.html");
  return win;
}
```

- 每个窗口独立渲染进程。
- 窗口间通信经主进程中转，或按需用 `webContents.send` 定向推送。

## 2. 窗口间通信

主进程作为中转：

```js
// main.js
const { ipcMain, BrowserWindow } = require("electron");

ipcMain.handle("note:open", (_e, note) => {
  const noteWin = createNoteWindow(note);
  noteWin.webContents.once("did-finish-load", () => {
    noteWin.webContents.send("note:data", note);
  });
});
```

收到数据后由主进程决定投递到哪个窗口，渲染进程之间不直接通信。

## 3. 应用菜单

```js
const { Menu } = require("electron");

const template = [
  {
    label: "文件",
    submenu: [
      { label: "新建", accelerator: "CmdOrCtrl+N", click: () => createMainWindow() },
      { label: "退出", role: "quit" },
    ],
  },
  {
    label: "编辑",
    submenu: [
      { role: "undo" },
      { role: "redo" },
      { type: "separator" },
      { role: "cut" },
      { role: "copy" },
      { role: "paste" },
    ],
  },
];

Menu.setApplicationMenu(Menu.buildFromTemplate(template));
```

- `role` 提供系统内置行为，无需自实现。
- `accelerator` 设置快捷键。

## 4. 系统托盘

```js
const { Tray, Menu, nativeImage } = require("electron");

let tray;

app.whenReady().then(() => {
  tray = new Tray(nativeImage.createFromPath("icon.png"));
  tray.setToolTip("我的桌面应用");
  tray.setContextMenu(Menu.buildFromTemplate([
    { label: "打开主窗口", click: () => mainWin.show() },
    { label: "退出", role: "quit" },
  ]));
  tray.on("click", () => mainWin.show());
});
```

托盘让应用在关闭窗口后仍常驻（需配合隐藏窗口而非退出）。

## 5. 系统通知

```js
const { Notification } = require("electron");

function notify(title, body) {
  if (Notification.isSupported()) {
    new Notification({ title, body }).show();
  }
}
```

通知在主进程创建，避免渲染进程受限。

## 6. 对话框

```js
const { dialog } = require("electron");

// 主进程内
const result = await dialog.showOpenDialog(mainWin, {
  properties: ["openFile", "multiSelections"],
  filters: [{ name: "Markdown", extensions: ["md"] }],
});
console.log(result.filePaths);
```

文件打开/保存对话框经主进程弹出，得到路径后再读写。

## 7. 常用系统能力

| API | 用途 |
|---|---|
| `shell.openExternal` | 打开浏览器链接 |
| `shell.openPath` | 用默认程序打开文件/目录 |
| `clipboard` | 剪贴板读写 |
| `globalShortcut` | 全局快捷键 |
| `powerMonitor` | 电源状态监听 |

## 8. 动手任务

1. 创建主窗口与第二个窗口，实现经主进程的窗口间数据传递。
2. 用模板构建应用菜单，含自定义项与 role 项。
3. 添加系统托盘，实现常驻与显示/隐藏窗口。
4. 用 Notification 发送通知，用 dialog 选择文件。
5. 验证关闭窗口不退出（托盘常驻）的逻辑。

## 阶段二验收

- 能创建多窗口并完成窗口间通信。
- 能构建菜单、托盘与通知。
- 能用对话框与 shell 完成系统交互。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 关闭窗口后应用退出 | 未监听 window-all-closed 保留托盘，或想常驻时未 hide |
| 菜单不显示 | `Menu.setApplicationMenu` 在 ready 后调用 |
| 托盘图标空白 | 图标路径错误或格式不支持 |
| 通知不显示 | 确认 `Notification.isSupported()` 与系统通知权限 |
| 全局快捷键冲突 | 注册后检查 `globalShortcut.register` 返回值 |

## 进入下一阶段的条件

你能够构建完整的桌面外壳。此时进入 [阶段三：文件、数据与前端集成](./stage-3-file-and-data.md)。
