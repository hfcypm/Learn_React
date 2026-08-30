# Electron 与 Tauri 桌面框架选型对比

**目标**：在 Web 前端基础上选择桌面应用框架时，用一份可复用的对比表与决策流程，判断项目适合 Electron 还是 Tauri。

## 1. 定位与共同点

两者都用 Web 技术（HTML/CSS/JS）构建跨平台桌面应用，都支持 Windows/macOS/Linux，都能复用现有前端技能与代码。区别在底层实现：

```text
Electron: 前端界面 + Node.js 后端 + 内置 Chromium（每个应用自带浏览器内核）
Tauri:    前端界面 + Rust 后端 + 系统 WebView（复用操作系统自带浏览器内核）
```

- Electron 由 GitHub 维护，生态成熟，`remote` 已移除、默认安全配置收紧。
- Tauri 由 Tauri 社区维护，当前主线是 2.x，Rust 后端 + capabilities 权限模型。

## 2. 架构对比

| 维度 | Electron | Tauri |
|---|---|---|
| 界面引擎 | 内置 Chromium | 系统 WebView（WebView2/WKWebView/WebKitGTK） |
| 后端语言 | JavaScript/TypeScript（Node.js） | Rust |
| 前后端通信 | preload + contextBridge + IPC | invoke + 命令 + 事件 |
| 进程模型 | 主进程 + 渲染进程（每窗口） | 核心进程 + webview 进程 |
| 权限模型 | 靠安全配置约定（contextIsolation 等） | capabilities 显式声明（默认收敛） |

## 3. 核心维度对比

| 维度 | Electron | Tauri |
|---|---|---|
| 安装包体积 | 大（约 60-150MB+，含 Chromium） | 小（约 2-10MB） |
| 内存占用 | 高（每窗口一个 Chromium 进程） | 低（复用系统 WebView） |
| 启动速度 | 较慢 | 较快 |
| 运行性能 | Chromium 一致、可预期 | 依赖系统 WebView，跨平台可能有差异 |
| 安全性 | 需手动配置隔离/沙箱 | 权限默认收敛，粒度细 |
| 生态成熟度 | 非常成熟，库与工具多 | 快速增长，插件体系完善 |
| 学习成本 | 仅需 JS/TS（易上手） | 需 Rust 基础（所有权/错误处理） |
| 打包工具 | electron-builder / Forge | 内置打包器（bundle） |
| 自动更新 | electron-updater | 官方 updater 插件 |
| 移动端 | 不支持 | 支持 iOS/Android（Tauri 2） |
| Node 生态复用 | 直接可用 | 需在 Rust 端重写或桥接 |

## 4. 适用场景

### 优先选 Electron

- 团队以 JavaScript/TypeScript 为主，无 Rust 经验，追求快速交付。
- 需要直接复用大量 Node.js 生态（如文件、网络、原生模块）。
- 需要跨平台一致的内核行为，避免 WebView 差异（复杂渲染、老旧系统）。
- 已有成熟工具链与 CI 基建，关注生态完备度。

### 优先选 Tauri

- 对安装包体积与内存占用敏感（工具类、效率类小应用）。
- 追求默认安全的权限模型与更小的攻击面。
- 团队能投入 Rust 学习，或后端逻辑需要强类型与高性能。
- 需要同时交付桌面与移动端。

## 5. 选型决策流程

```text
是否有移动端需求？
  ├─ 是 -> 优先 Tauri
  └─ 否
      团队是否熟悉 Rust？
        ├─ 否，且重视交付速度 -> Electron
        └─ 是 或 愿意投入学习
            是否对体积/内存敏感？
              ├─ 是 -> Tauri
              └─ 否
                  是否重度依赖 Node.js 生态？
                    ├─ 是 -> Electron
                    └─ 否 -> 两者皆可，按团队偏好
```

决策建议：先用最小原型（各跑一个带文件读写与系统通知的 demo）验证关键能力，再定框架。

## 6. 各维度验证清单

用同一个 demo 需求（窗口 + 文件读写 + 系统通知 + 打包安装）在两个框架上各实现一遍：

- [ ] 初始化与首个窗口
- [ ] 前端调用系统能力（文件读写）
- [ ] 系统通知
- [ ] 打包产物体积与安装
- [ ] 自动更新链路
- [ ] 启动速度与内存占用对比

## 7. 与学习路线的配合

- 选 Electron：进入 [Electron 学习路线](./electron-learning-path/README.md)，从双进程模型学起。
- 选 Tauri：进入 [Tauri 学习路线](./tauri-learning-path/README.md)，先掌握 Rust 命令与权限。
- 两条路线的综合实战可对照阅读：[Electron 桌面笔记](./electron-learning-path/project-practice.md)、[Tauri 跨平台文件管理器](./tauri-learning-path/project-practice.md)。

## 8. 常见误区

| 误区 | 事实 |
|---|---|
| Tauri 体积必然更小 | 一般如此，但前端产物与依赖仍占空间 |
| Electron 一定更慢 | 启动偏慢，但渲染一致性强 |
| Tauri 不需要写 Rust | 需要，核心命令都是 Rust |
| Electron 默认不安全 | 新版本默认收紧，但需主动维护配置 |
| Tauri 无法复用前端 | 完全可复用，这正是它的卖点 |
| 移动端只需 Tauri | Tauri 支持，但需额外配置移动工程 |

## 9. 最终决策建议

- 面向快速交付、重度 Node 生态、统一渲染：**Electron**。
- 面向体积/内存敏感、默认安全、含移动端、团队可学 Rust：**Tauri**。
- 不确定时，先做最小原型再定，避免被框架体积与团队偏好过早锁死。
