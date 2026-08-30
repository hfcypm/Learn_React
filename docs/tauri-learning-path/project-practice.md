# Tauri 综合实战：跨平台文件管理器

## 1. 项目目标

用 Tauri 2 构建一个跨平台文件管理器，覆盖权限配置、Rust 命令、文件系统插件、事件通知与打包发布。项目按阶段扩展，最终交付可安装、可更新的桌面应用。

```text
初始化 -> 权限配置 -> 文件命令 -> 界面交互
    -> 系统能力 -> 打包发布
```

## 2. 需求

- 浏览指定目录的文件与子目录。
- 预览文件内容（文本）。
- 新建文件与目录、重命名、删除（需确认）。
- 复制文件路径、用默认程序打开。
- 文件操作进度用事件通知前端。
- 打包发布并接入更新。

## 3. 技术选择

| 技术 | 用途 |
|---|---|
| Tauri 2 | 桌面框架 |
| React + TypeScript | 前端界面 |
| tauri-plugin-fs | 文件系统 |
| tauri-plugin-dialog | 目录选择与确认 |
| tauri-plugin-updater | 自动更新 |

## 4. 项目结构

```text
file-manager/
├── src/                 # React 前端
├── src-tauri/
│   ├── src/lib.rs       # 命令与状态
│   ├── capabilities/default.json
│   ├── tauri.conf.json
│   └── Cargo.toml
└── package.json
```

## 5. 初始化

```bash
npm create tauri-app@latest file-manager -- --template react-ts
cd file-manager
npm install
npm run tauri dev
```

## 6. 权限配置

```json
// src-tauri/capabilities/default.json
{
  "identifier": "default",
  "description": "主窗口能力",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "fs:default",
    "dialog:default",
    "dialog:allow-confirm"
  ]
}
```

## 7. Rust 命令

```rust
// src-tauri/src/lib.rs
use serde::Serialize;
use std::path::PathBuf;
use tauri::Manager;

#[derive(Serialize)]
struct Entry {
    name: String,
    path: String,
    is_dir: bool,
}

#[tauri::command]
fn list_dir(dir: String) -> Result<Vec<Entry>, String> {
    let entries = std::fs::read_dir(&dir)
        .map_err(|e| format!("读取目录失败：{}", e))?
        .filter_map(|e| e.ok())
        .map(|e| {
            let path = e.path();
            Entry {
                name: e.file_name().to_string_lossy().into_owned(),
                path: path.to_string_lossy().into_owned(),
                is_dir: path.is_dir(),
            }
        })
        .collect();
    Ok(entries)
}

#[tauri::command]
fn read_text(path: String) -> Result<String, String> {
    std::fs::read_to_string(&path).map_err(|e| format!("读取文件失败：{}", e))
}

#[tauri::command]
fn create_file(dir: String, name: String) -> Result<(), String> {
    let safe = PathBuf::from(name)
        .file_name()
        .map(|s| s.to_string_lossy().into_owned())
        .unwrap_or_default();
    let path = PathBuf::from(dir).join(safe);
    std::fs::write(&path, "").map_err(|e| format!("创建失败：{}", e))
}

#[tauri::command]
fn create_dir(dir: String, name: String) -> Result<(), String> {
    let safe = PathBuf::from(name)
        .file_name()
        .map(|s| s.to_string_lossy().into_owned())
        .unwrap_or_default();
    std::fs::create_dir(PathBuf::from(dir).join(safe)).map_err(|e| e.to_string())
}

#[tauri::command]
fn remove_entry(path: String) -> Result<(), String> {
    let p = PathBuf::from(path);
    if p.is_dir() {
        std::fs::remove_dir_all(&p).map_err(|e| e.to_string())
    } else {
        std::fs::remove_file(&p).map_err(|e| e.to_string())
    }
}

#[tauri::command]
fn emit_notice(app: tauri::AppHandle, message: String) {
    use tauri::Emitter;
    let _ = app.emit("notice", message);
}
```

注册：

```rust
pub fn run() {
    tauri::Builder::default()
        .plugin(tauri_plugin_fs::init())
        .plugin(tauri_plugin_dialog::init())
        .invoke_handler(tauri::generate_handler![
            list_dir,
            read_text,
            create_file,
            create_dir,
            remove_entry,
            emit_notice
        ])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

## 8. 前端界面

```tsx
// src/App.tsx
import { useEffect, useState } from "react";
import { invoke } from "@tauri-apps/api/core";
import { listen } from "@tauri-apps/api/event";

type Entry = { name: string; path: string; is_dir: boolean };

function App() {
  const [dir, setDir] = useState("");
  const [entries, setEntries] = useState<Entry[]>([]);
  const [notice, setNotice] = useState("");

  async function refresh() {
    if (!dir) return;
    setEntries(await invoke<Entry[]>("list_dir", { dir }));
  }

  useEffect(() => {
    const unlisten = listen<string>("notice", (e) => setNotice(e.payload));
    return () => {
      unlisten.then((fn) => fn());
    };
  }, []);

  useEffect(() => {
    refresh();
  }, [dir]);

  return (
    <div className="p-6">
      <div className="flex gap-2">
        <input
          className="flex-1 rounded border p-2"
          value={dir}
          onChange={(e) => setDir(e.target.value)}
          placeholder="输入目录路径"
        />
        <button
          className="rounded bg-blue-600 px-4 py-2 text-white"
          onClick={async () => {
            await invoke("emit_notice", { message: "目录已刷新" });
            refresh();
          }}
        >
          刷新
        </button>
      </div>

      <ul className="mt-4 space-y-1">
        {entries.map((e) => (
          <li key={e.path} className="flex items-center justify-between rounded border p-2">
            <button
              onClick={() => {
                if (e.is_dir) setDir(e.path);
                else invoke("read_text", { path: e.path }).then((t) => alert(t));
              }}
            >
              {e.is_dir ? "📁" : "📄"} {e.name}
            </button>
            <button
              className="text-red-600"
              onClick={async () => {
                await invoke("remove_entry", { path: e.path });
                refresh();
              }}
            >
              删除
            </button>
          </li>
        ))}
      </ul>

      {notice && <p className="mt-4 text-sm text-gray-600">{notice}</p>}
    </div>
  );
}

export default App;
```

## 9. 打包配置

```json
{
  "bundle": {
    "active": true,
    "targets": "all",
    "createUpdaterArtifacts": true,
    "category": "Utility"
  }
}
```

```bash
npm run tauri build
```

## 10. 实施顺序

1. 初始化项目并跑通窗口。
2. 配置 capabilities 与必要插件。
3. 实现目录与文件命令。
4. 实现前端浏览、预览与删除交互。
5. 用事件通知操作结果。
6. 配置打包、签名与更新。

## 11. 验收清单

- [ ] 能浏览目录并进入子目录。
- [ ] 能新建文件/目录与删除（带确认）。
- [ ] 能预览文本文件内容。
- [ ] 事件通知正确显示。
- [ ] capabilities 只含必要权限。
- [ ] 路径经 basename 过滤防穿越。
- [ ] 打包产物可安装运行。

## 12. 按阶段学习卡片

| 阶段 | 项目增量 |
|---|---|
| 零 | 初始化、窗口、invoke 首调用 |
| 一 | capabilities 与插件权限 |
| 二 | 文件命令与错误处理 |
| 三 | fs/dialog 插件与事件 |
| 四 | 打包、签名、更新 |
