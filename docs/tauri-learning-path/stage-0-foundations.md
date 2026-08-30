# 阶段零：Tauri 基础与工具链

**目标**：理解 Tauri 的前后端架构，初始化项目，运行第一个窗口并理解项目结构。

## 1. Tauri 是什么

Tauri 用系统 WebView 渲染前端界面，用 Rust 编写后端能力，产物是体积小、内存占用低的原生可执行文件。相比 Electron 内置整个 Chromium，Tauri 复用系统 WebView。

```text
前端（HTML/CSS/JS，任意框架）
   │ invoke / 事件
   ▼
Rust 后端（tauri crate，系统能力）
   ▼
系统 WebView + 打包产物
```

## 2. 前置环境

```bash
# 安装 Rust 工具链（若未安装）
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# 验证
rustc --version
cargo --version
node --version
npm --version
```

各平台还需 WebView 与构建工具链（Windows 需 WebView2 + MSVC；macOS 需 Xcode 命令行工具；Linux 需 webkit2gtk 等系统依赖）。

## 3. 初始化项目

```bash
# 使用官方脚手架
npm create tauri-app@latest my-app
# 选择 TypeScript + React + Vite
cd my-app
npm install
npm run tauri dev
```

`tauri dev` 会编译 Rust 后端并打开窗口，首次编译较慢。

## 4. 项目结构

```text
my-app/
├── src/                 # 前端（Vite + React）
├── src-tauri/
│   ├── src/
│   │   ├── main.rs      # 后端入口
│   │   └── lib.rs       # 命令与 Builder
│   ├── capabilities/    # 权限声明
│   ├── icons/           # 应用图标
│   ├── tauri.conf.json  # 核心配置
│   └── Cargo.toml       # Rust 依赖
└── package.json
```

## 5. 核心配置

```json
// src-tauri/tauri.conf.json
{
  "$schema": "https://schema.tauri.app/config/2",
  "productName": "my-app",
  "version": "0.1.0",
  "identifier": "com.example.myapp",
  "build": {
    "beforeDevCommand": "npm run dev",
    "devUrl": "http://localhost:5173",
    "beforeBuildCommand": "npm run build",
    "frontendDist": "../dist"
  },
  "app": {
    "windows": [
      {
        "label": "main",
        "title": "My App",
        "width": 960,
        "height": 650
      }
    ],
    "security": {
      "csp": null
    }
  },
  "bundle": {
    "active": true,
    "targets": "all"
  }
}
```

- `identifier` 是应用唯一标识，打包与更新依赖。
- `build.devUrl` 指向前端开发服务器。
- `build.frontendDist` 指向前端构建产物。

## 6. 后端入口

```rust
// src-tauri/src/main.rs
#![cfg_attr(not(debug_assertions), windows_subsystem = "windows")]

fn main() {
    my_app_lib::run()
}
```

```rust
// src-tauri/src/lib.rs
#[tauri::command]
fn greet(name: &str) -> String {
    format!("你好，{}！", name)
}

#[cfg_attr(mobile, tauri::mobile_entry_point)]
pub fn run() {
    tauri::Builder::default()
        .invoke_handler(tauri::generate_handler![greet])
        .run(tauri::generate_context!())
        .expect("error while running tauri application");
}
```

## 7. 前端调用命令

```tsx
// src/App.tsx
import { invoke } from "@tauri-apps/api/core";

function App() {
  return (
    <button onClick={async () => {
      const msg = await invoke("greet", { name: "世界" });
      console.log(msg);
    }}>
      打招呼
    </button>
  );
}
```

`invoke("greet", { name: "世界" })` 调用 Rust 命令，参数名用 camelCase 与 Rust 参数对应。

## 8. 开发与调试

- 前端用 Vite HMR 热更新。
- Rust 端日志用 `println!` 或 `log`，编译错误看终端。
- 用 `tauri dev` 同时启动两端，`tauri build` 产出发布包。

## 9. 动手任务

1. 检查 Rust 与 Node 环境。
2. 用脚手架初始化项目并运行。
3. 阅读 tauri.conf.json，改窗口标题与尺寸。
4. 新增一个返回系统时间戳的命令并在前端调用。
5. 运行 `npm run tauri build` 观察产物与体积。

## 阶段零验收

- 能解释前端、Rust 后端、系统 WebView 的关系。
- 能初始化项目并运行窗口。
- 能读懂 tauri.conf.json 的核心字段。
- 能通过 invoke 调用一个自定义命令。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| Rust 未安装 | 安装 rustup 并重开终端 |
| 编译超慢 | 首次编译属正常，后续增量快 |
| WebView 缺失 | 按平台安装系统依赖 |
| dev 窗口空白 | devUrl 与 Vite 端口不一致 |
| invoke 失败 | 命令未注册进 `generate_handler` |

## 进入下一阶段的条件

你能够运行窗口并通过 invoke 调用命令。此时进入 [阶段一：配置与权限系统](./stage-1-config-and-permissions.md)。
