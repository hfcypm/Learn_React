# 阶段三：系统能力与插件

**目标**：掌握事件系统与官方插件（文件系统、对话框、Shell、HTTP），用最小权限完成系统交互。

## 1. 事件系统

前后端互相通知用事件，与命令的请求/响应互补。

### 前端监听后端事件

```ts
import { listen } from "@tauri-apps/api/event";

const unlisten = await listen("progress", (event) => {
  console.log("进度：", event.payload);
});
// 不再需要时取消监听
unlisten();
```

### 后端发事件

```rust
use tauri::Emitter;

#[tauri::command]
fn start_job(app: tauri::AppHandle) {
    for i in 0..=10 {
        let _ = app.emit("progress", i * 10);
        std::thread::sleep(std::time::Duration::from_millis(100));
    }
}
```

命令用于"我要结果"，事件用于"后端主动通知"。

## 2. 安装官方插件

以文件系统插件为例：

```bash
cargo add tauri-plugin-fs
```

Rust 端注册：

```rust
use tauri_plugin_fs::FsExt;

pub fn run() {
    tauri::Builder::default()
        .plugin(tauri_plugin_fs::init())
        // ...
        .run(tauri::generate_context!())
}
```

前端安装 API 包：

```bash
npm install @tauri-apps/plugin-fs
```

声明权限：

```json
{
  "permissions": ["fs:default"]
}
```

## 3. 文件系统插件

```ts
import { readTextFile, writeTextFile } from "@tauri-apps/plugin-fs";
import { join } from "@tauri-apps/api/path";
import { appDataDir } from "@tauri-apps/api/path";

const dir = await appDataDir();
const file = await join(dir, "notes.json");

await writeTextFile(file, JSON.stringify({ hello: "world" }));
const data = await readTextFile(file);
```

- 用 `appDataDir()`、`documentDir()` 等系统目录 API，不写死路径。
- 路径由前端传入后端统一解析，避免路径拼接错误。

## 4. 对话框插件

```bash
cargo add tauri-plugin-dialog
```

```rust
.plugin(tauri_plugin_dialog::init())
```

```json
{ "permissions": ["dialog:default"] }
```

```ts
import { open, save } from "@tauri-apps/plugin-dialog";

const selected = await open({ multiple: true, directory: false });
const target = await save({ defaultPath: "untitled.md" });
```

## 5. Shell 插件

谨慎使用，默认关闭，仅按需开启：

```json
{ "permissions": ["shell:allow-open"] }
```

```ts
import { open as shellOpen } from "@tauri-apps/plugin-shell";
await shellOpen("https://example.com");
```

`shell:allow-open` 只允许用默认程序打开，限制执行任意命令的范围。

## 6. 其他常用插件

| 插件 | 能力 |
|---|---|
| `tauri-plugin-store` | 键值存储 |
| `tauri-plugin-http` | HTTP 请求（替代受限的 fetch） |
| `tauri-plugin-sql` | SQLite 数据库 |
| `tauri-plugin-notification` | 系统通知 |
| `tauri-plugin-clipboard-manager` | 剪贴板 |

## 7. 动手任务

1. 用事件实现后端进度通知，前端实时显示。
2. 安装 fs 插件，读写 `appDataDir` 下的 JSON。
3. 用 dialog 插件选择并打开文件。
4. 安装 store 插件保存设置。
5. 用 notification 插件发送系统通知。

## 阶段三验收

- 能用事件实现前后端通知。
- 能安装并配置官方插件。
- 能读写系统目录文件与对话框。
- 能按最小权限使用插件能力。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 插件 API 不存在 | 确认 Rust 注册 + 前端包 + 权限声明齐全 |
| 权限拒绝 | 检查 permissions 是否声明对应能力 |
| 事件收不到 | 确认同一事件名与 emit/listen 匹配 |
| 路径错误 | 用系统目录 API，不用硬编码路径 |
| shell 调用被拒 | 权限未开或配置了不允许的 scope |

## 进入下一阶段的条件

你能够用插件完成系统能力。此时进入 [阶段四：打包发布与生产](./stage-4-production.md)。
