# 阶段一：配置与权限系统

**目标**：掌握 tauri.conf.json 的完整配置与 Tauri 2 的 capabilities 权限模型，按最小授权原则配置应用。

## 1. 应用配置全览

`tauri.conf.json` 三大块：

| 块 | 职责 |
|---|---|
| `build` | 前端命令、devUrl、产物路径 |
| `app` | 窗口、安全、托盘、窗口属性 |
| `bundle` | 打包目标、图标、分类、签名 |

窗口多实例配置：

```json
"app": {
  "windows": [
    { "label": "main", "title": "主窗口", "width": 960, "height": 650 },
    { "label": "settings", "title": "设置", "width": 480, "height": 400, "resizable": false }
  ]
}
```

## 2. 权限模型

Tauri 2 的安全模型是核心设计：前端要使用任何系统能力，必须先声明权限。

```text
能力(capabilities)
  └── 窗口/webview 范围
        └── 权限(permissions)
              └── 具体系统功能（fs、dialog、shell、http 等）
```

默认前端只能调用自定义命令与部分核心 API。访问文件、对话框等能力必须显式声明。

## 3. capabilities 文件

`src-tauri/capabilities/default.json`：

```json
{
  "$schema": "../gen/schemas/desktop-schema.json",
  "identifier": "default",
  "description": "主窗口默认能力",
  "windows": ["main"],
  "permissions": [
    "core:default",
    "core:window:allow-minimize",
    "core:window:allow-close"
  ]
}
```

- `identifier`：能力标识，可配置多个能力文件。
- `windows`：限制该能力作用于哪些窗口。
- `permissions`：显式列出允许的功能。

## 4. 插件权限

安装官方插件后需要为其声明权限，例如对话框插件：

```json
{
  "permissions": [
    "core:default",
    "dialog:default",
    "dialog:allow-open"
  ]
}
```

权限标识形如 `<plugin>:<action>`。最小授权原则：只声明用到的权限。

## 5. 自定义命令的权限

自定义命令默认可用，但可在 `permissions` 中引用自定义权限集进行限制（高级场景）。多数情况下，自定义命令本身是安全的入口，真正的敏感能力由 Rust 端校验。

## 6. 安全配置

```json
"app": {
  "security": {
    "csp": "default-src 'self'; style-src 'self' 'unsafe-inline'",
    "capabilities": null
  }
}
```

- `csp`：内容安全策略，限制可加载资源。
- `capabilities`：可覆盖 capabilities 目录（默认自动启用 `src-tauri/capabilities` 下全部文件）。

## 7. 权限调试

- 未声明权限的调用会返回拒绝错误。
- 开发时查看终端与前端 console 的权限报错。
- 用 `tauri dev` 反复调整 `permissions` 后重新编译生效。

## 8. 动手任务

1. 打开 capabilities/default.json，逐条解释每个权限。
2. 移除 `core:default` 观察影响，再恢复。
3. 安装 dialog 插件并声明 `dialog:default`。
4. 新增一个"settings"窗口，为其创建独立的能力文件，仅授予最小权限。
5. 编写一条 CSP 并验证开发环境正常。

## 阶段一验收

- 能读懂 tauri.conf.json 的 build/app/bundle。
- 能解释 capabilities 与 permissions 的作用。
- 能为插件声明最小权限。
- 能按窗口拆分能力文件。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 调用能力被拒 | 未在 permissions 声明 |
| 权限标识错误 | 检查插件版本与文档中的权限名 |
| 多窗口权限异常 | 确认能力文件的 `windows` 列表 |
| CSP 拦截资源 | 调整 csp 白名单 |
| 能力文件未生效 | 确认放在 `src-tauri/capabilities/` |

## 进入下一阶段的条件

你能够配置权限并控制应用能力。此时进入 [阶段二：Rust 命令与后端](./stage-2-commands-and-backend.md)。
