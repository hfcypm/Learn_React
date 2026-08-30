# 阶段二：Rust 命令与后端

**目标**：掌握 Rust 命令的定义、参数、状态管理与错误处理，构建类型安全的后端能力。

## 1. 命令基础

```rust
// src-tauri/src/lib.rs
#[tauri::command]
fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[tauri::command]
fn greet(name: String) -> String {
    format!("你好，{}", name)
}
```

注册到 Builder：

```rust
pub fn run() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![add, greet])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

前端调用：

```ts
import { invoke } from "@tauri-apps/api/core";

const sum = await invoke<number>("add", { a: 1, b: 2 });
const msg = await invoke<string>("greet", { name: "世界" });
```

## 2. 参数与序列化

- 命令参数用 camelCase，前端 invoke 传同名对象。
- 复杂类型用 `serde` 派生 `Serialize` / `Deserialize`：

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
struct Note {
    id: i32,
    title: String,
    content: String,
}

#[tauri::command]
fn create_note(note: Note) -> Note {
    // 处理并返回
    note
}
```

前端传入对象自动映射为 Rust 结构体，字段名保持一致。

## 3. 返回 Result 与错误处理

命令返回 `Result<T, String>`，错误信息能传到前端：

```rust
#[tauri::command]
fn read_file(path: String) -> Result<String, String> {
    std::fs::read_to_string(&path)
        .map_err(|e| format!("读取失败：{}", e))
}
```

```ts
try {
  const content = await invoke<string>("read_file", { path });
} catch (err) {
  console.error("读取失败：", err);
}
```

用自定义错误类型提高可读性：

```rust
#[derive(Debug, thiserror::Error)]
enum AppError {
    #[error("文件不存在: {0}")]
    NotFound(String),
    #[error("IO 错误: {0}")]
    Io(#[from] std::io::Error),
}
```

## 4. 状态管理

用 `tauri::State` 在命令间共享状态：

```rust
use tauri::State;
use std::sync::Mutex;

struct AppState {
    notes: Mutex<Vec<String>>,
}

#[tauri::command]
fn add_note(state: State<AppState>, text: String) -> Result<Vec<String>, String> {
    let mut notes = state.notes.lock().map_err(|e| e.to_string())?;
    notes.push(text);
    Ok(notes.clone())
}
```

注册状态：

```rust
pub fn run() {
    tauri::Builder::default()
        .manage(AppState { notes: Mutex::new(Vec::new()) })
        .invoke_handler(tauri::generate_handler![add_note])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

- `Mutex` 保证线程安全。
- 状态在应用生命周期内共享，适合配置与内存缓存。

## 5. 异步命令

命令可用 `async fn`，配合 `tokio`：

```rust
#[tauri::command]
async fn fetch_remote(url: String) -> Result<String, String> {
    let body = reqwest::get(&url)
        .await
        .map_err(|e| e.to_string())?
        .text()
        .await
        .map_err(|e| e.to_string())?;
    Ok(body)
}
```

耗时操作放异步命令，避免阻塞主线程。

## 6. 访问窗口与应用句柄

命令可注入 `Window` 或 `AppHandle`：

```rust
use tauri::Window;

#[tauri::command]
fn set_title(window: Window, title: String) {
    let _ = window.set_title(&title);
}
```

## 7. 动手任务

1. 定义 `add`、`greet` 两个命令并注册。
2. 定义 `Note` 结构体命令，验证对象映射。
3. 让命令返回 `Result`，在前端 try/catch 错误。
4. 用 `State` + `Mutex` 实现内存笔记列表。
5. 写一个异步命令调用远程接口。

## 阶段二验收

- 能定义并注册命令。
- 能序列化复杂参数与返回值。
- 能用 `Result` 正确传递错误。
- 能用 `State` 共享数据。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| invoke 报"命令不存在" | 未注册进 `generate_handler` |
| 参数缺失 | 前端参数名与 Rust 参数名不一致 |
| 序列化错误 | 结构体未派生 Serialize/Deserialize |
| 状态锁失败 | Mutex 中毒，检查是否有 panic |
| 编译错误 | 确认新增依赖写入 Cargo.toml |

## 进入下一阶段的条件

你能够用 Rust 命令提供类型安全的后端能力。此时进入 [阶段三：系统能力与插件](./stage-3-system-and-plugins.md)。
