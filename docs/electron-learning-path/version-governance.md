# Electron 版本边界与迁移

**目标**：让学习内容、示例项目和生产项目拥有清晰的版本边界，降低大版本升级和 API 混用风险。

## 1. 版本差异

Electron 迭代较快，主要 API 沿革：

| 能力 | 旧写法 | 当前主线写法 |
|---|---|---|
| 主进程访问 | `remote` 模块 | 已移除，全部改用 IPC |
| 渲染进程 Node | `nodeIntegration: true` | 默认关闭，用 preload 桥 |
| 脚本模块 | CommonJS 为主 | ESM 支持增强，仍兼容 CJS |
| 渲染进程模块 | `require("electron")` 任意模块 | 推荐 `electron/renderer` 显式导入 |
| 桌面截屏 | 渲染进程 `desktopCapturer` | 移入主进程，经 IPC 桥接 |
| 构建工具 | 手工 Webpack | electron-vite / Electron Forge 一体化 |

## 2. 每个示例必须记录的元数据

```markdown
> 适用范围：Electron 当前稳定版（Chromium + Node.js 内置）
> 需要的工具：Node.js、npm、electron
> 验证命令：npm start、npm run dist
> 最后复核日期：YYYY-MM-DD
```

## 3. 迁移要点

- `remote` 已移除：原 `remote.BrowserWindow` 等调用改为主进程 IPC。
- 默认安全配置：`contextIsolation: true`、`sandbox: true`、`nodeIntegration: false`。
- preload 用 `electron/renderer` 导入 `contextBridge` 与 `ipcRenderer`。
- 渲染进程不再能直接 `require` Electron 系统模块，全部经 contextBridge。
- 构建迁移到 electron-vite：主进程、preload、渲染进程三份构建配置。

## 4. 升级前检查

1. 阅读官方 [breaking changes](https://www.electronjs.org/docs/latest/breaking-changes)。
2. 检查 `remote`、`nodeIntegration`、`desktopCapturer` 等废弃用法。
3. 检查 preload 与安全配置是否齐全。
4. 检查依赖与内置 Chromium 版本兼容性。
5. 在分支环境完整验证窗口、IPC 与打包。

## 5. 升级后检查

- 窗口正常创建与加载。
- preload API 在页面可用且沙箱正常。
- IPC 双向通信行为一致。
- 打包产物在各平台可运行。
- 自动更新链路正常。
