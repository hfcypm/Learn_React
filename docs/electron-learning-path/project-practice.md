# Electron 综合实战：桌面笔记应用

## 1. 项目目标

用 Electron 构建一个跨平台桌面笔记应用，覆盖进程模型、安全 IPC、系统集成、文件持久化与打包发布。项目按阶段扩展，最终交付可安装、可更新的桌面应用。

```text
初始化 -> 窗口与外壳 -> 安全 IPC -> 文件存储
    -> 系统集成 -> 打包发布
```

## 2. 需求

- 多窗口：主窗口列出笔记，编辑窗口打开单篇。
- 本地文件存储：笔记存为 Markdown 文件。
- 系统集成：菜单、托盘、系统通知、打开文件对话框。
- 设置持久化：主题、字体等存到 `userData`。
- 打包发布：三平台安装包、签名与自动更新。

## 3. 技术选择

| 技术 | 用途 |
|---|---|
| Electron | 桌面框架 |
| electron-vite | 主进程/preload/渲染统一构建 |
| React + TypeScript | 渲染进程界面 |
| electron-builder | 打包与自动更新 |
| electron-updater | 自动更新 |

## 4. 项目结构

```text
my-notes/
├── src/
│   ├── main/          # 主进程
│   │   ├── index.ts   # 应用入口
│   │   ├── ipc.ts     # IPC 通道注册
│   │   └── store.ts   # 文件与设置持久化
│   ├── preload/
│   │   └── index.ts   # contextBridge 暴露
│   └── renderer/      # React 界面
├── build/icon.png
├── electron-builder.yml
└── package.json
```

## 5. 初始化

```bash
npm create @quick-start/electron@latest my-notes -- --template react-ts
cd my-notes
npm install
npm run dev
```

## 6. 主进程入口

```ts
// src/main/index.ts
import { app, BrowserWindow } from "electron";
import path from "node:path";
import { registerIpc } from "./ipc";

function createWindow() {
  const win = new BrowserWindow({
    width: 960,
    height: 700,
    webPreferences: {
      preload: path.join(__dirname, "../preload/index.js"),
      contextIsolation: true,
      sandbox: true,
    },
  });

  if (process.env["ELECTRON_RENDERER_URL"]) {
    win.loadURL(process.env["ELECTRON_RENDERER_URL"]);
  } else {
    win.loadFile(path.join(__dirname, "../renderer/index.html"));
  }
}

app.whenReady().then(() => {
  registerIpc();
  createWindow();
});

app.on("window-all-closed", () => {
  if (process.platform !== "darwin") app.quit();
});
```

## 7. 持久化与 IPC

```ts
// src/main/store.ts
import { app } from "electron";
import fs from "node:fs/promises";
import path from "node:path";

const NOTES_DIR = () => path.join(app.getPath("documents"), "MyNotes");

export async function listNotes() {
  await fs.mkdir(NOTES_DIR(), { recursive: true });
  return (await fs.readdir(NOTES_DIR())).filter((f) => f.endsWith(".md"));
}

export async function saveNote(filename: string, content: string) {
  await fs.mkdir(NOTES_DIR(), { recursive: true });
  const safe = path.basename(filename);
  await fs.writeFile(path.join(NOTES_DIR(), safe), content, "utf-8");
}
```

```ts
// src/main/ipc.ts
import { ipcMain, dialog, Notification, BrowserWindow } from "electron";
import { listNotes, saveNote } from "./store";

export function registerIpc() {
  ipcMain.handle("note:list", listNotes);

  ipcMain.handle("note:save", (_e, filename: string, content: string) =>
    saveNote(filename, content),
  );

  ipcMain.handle("note:pick", async () => {
    const result = await dialog.showOpenDialog({
      properties: ["openFile"],
      filters: [{ name: "Markdown", extensions: ["md"] }],
    });
    return result.filePaths[0] ?? null;
  });

  ipcMain.handle("note:openInWindow", (_e, note: string) => {
    const win = new BrowserWindow({ width: 640, height: 480 });
    win.loadFile(path.join(__dirname, "../renderer/index.html"));
    win.webContents.once("did-finish-load", () => {
      win.webContents.send("note:data", note);
    });
  });

  ipcMain.handle("notify", (_e, title: string, body: string) => {
    if (Notification.isSupported()) new Notification({ title, body }).show();
  });
}
```

## 8. preload

```ts
// src/preload/index.ts
import { contextBridge, ipcRenderer } from "electron";

contextBridge.exposeInMainWorld("desktopAPI", {
  listNotes: () => ipcRenderer.invoke("note:list"),
  saveNote: (filename: string, content: string) =>
    ipcRenderer.invoke("note:save", filename, content),
  pickNote: () => ipcRenderer.invoke("note:pick"),
  openInWindow: (note: string) => ipcRenderer.invoke("note:openInWindow", note),
  notify: (title: string, body: string) =>
    ipcRenderer.invoke("notify", title, body),
  onNoteData: (cb: (data: string) => void) =>
    ipcRenderer.on("note:data", (_e, data) => cb(data)),
});
```

## 9. 渲染进程界面

```tsx
// src/renderer/App.tsx
import { useEffect, useState } from "react";

declare global {
  interface Window {
    desktopAPI: {
      listNotes: () => Promise<string[]>;
      saveNote: (f: string, c: string) => Promise<void>;
      pickNote: () => Promise<string | null>;
      openInWindow: (n: string) => Promise<void>;
      notify: (t: string, b: string) => Promise<void>;
      onNoteData: (cb: (d: string) => void) => void;
    };
  }
}

export function App() {
  const [notes, setNotes] = useState<string[]>([]);
  const [current, setCurrent] = useState("");

  useEffect(() => {
    window.desktopAPI.listNotes().then(setNotes);
    window.desktopAPI.onNoteData((data) => setCurrent(data));
  }, []);

  return (
    <div className="p-6">
      <h1 className="text-xl font-bold">我的笔记</h1>
      <ul className="mt-4 space-y-2">
        {notes.map((n) => (
          <li key={n}>
            <button onClick={() => window.desktopAPI.openInWindow(n)}>{n}</button>
          </li>
        ))}
      </ul>
      <button
        className="mt-4 rounded bg-blue-600 px-4 py-2 text-white"
        onClick={async () => {
          await window.desktopAPI.saveNote("新笔记.md", "# 新笔记");
          setNotes(await window.desktopAPI.listNotes());
        }}
      >
        新建笔记
      </button>
    </div>
  );
}
```

## 10. 打包配置

```yaml
# electron-builder.yml
appId: com.example.mynotes
productName: MyNotes
directories:
  output: release
files:
  - dist/**
win:
  target: nsis
mac:
  target: dmg
linux:
  target: AppImage
```

```bash
npm run build
npx electron-builder
```

## 11. 实施顺序

1. 初始化 electron-vite + React 项目并跑通窗口。
2. 实现文件持久化与笔记列表 IPC。
3. 实现编辑窗口与窗口间数据传递。
4. 添加菜单、托盘、通知与文件对话框。
5. 配置打包与自动更新。
6. 复查安全基线并发布。

## 12. 验收清单

- [ ] 笔记能保存为 Markdown 文件并重新列出。
- [ ] 多窗口编辑数据正确传递。
- [ ] 菜单、托盘与通知可用。
- [ ] 设置与数据写入系统目录。
- [ ] 安全基线满足（隔离、沙箱、入参校验）。
- [ ] 打包产物可安装运行。
- [ ] 自动更新链路可用。

## 13. 按阶段学习卡片

| 阶段 | 项目增量 |
|---|---|
| 零 | 初始化、窗口、生命周期 |
| 一 | preload、安全 IPC、通道注册 |
| 二 | 多窗口、菜单、托盘、通知 |
| 三 | 文件持久化、React 集成、设置 |
| 四 | 打包、签名、自动更新、安全检查 |
