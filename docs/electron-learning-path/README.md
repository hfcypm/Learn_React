# Electron 学习路线图（基础 → 生产）

**目标**：掌握 Electron 的主进程与渲染进程模型、IPC 通信、窗口与系统集成、本地数据与文件能力，以及打包发布与自动更新，能够在 Web 前端基础上构建跨平台桌面应用。

## 先看这里

- [新手导读](./start-here.md)：前置条件、学习顺序、排错顺序和完成标准。
- [综合实战：桌面笔记应用](./project-practice.md)：按阶段逐步扩展同一个项目。
- [学习评估与项目评分](./learning-assessment.md)：阶段考核、排错题、项目评分和评审问题。

## 学习阶段总览

| 阶段 | 名称 | 目标 |
|---|---|---|
| 零 | Electron 基础与工具链 | 理解双进程模型，初始化项目并运行第一个窗口 |
| 一 | 进程模型与 IPC 通信 | 掌握 preload、contextBridge 与安全的进程通信 |
| 二 | 窗口与系统集成 | 掌握多窗口、菜单、托盘与系统级能力 |
| 三 | 文件、数据与前端集成 | 掌握本地文件、数据持久化与前端框架集成 |
| 四 | 打包发布与生产 | 掌握打包、签名、自动更新、安全与性能 |

## 快速导航

- [阶段零：Electron 基础与工具链](./stage-0-foundations.md)
- [阶段一：进程模型与 IPC 通信](./stage-1-processes-and-ipc.md)
- [阶段二：窗口与系统集成](./stage-2-windows-and-system.md)
- [阶段三：文件、数据与前端集成](./stage-3-file-and-data.md)
- [阶段四：打包发布与生产](./stage-4-production.md)
- [综合实战：桌面笔记应用](./project-practice.md)

## 学习原则

- 主进程负责系统能力与生命周期，渲染进程负责界面，preload 是唯一安全桥。
- 一切渲染进程需要的能力都通过 IPC 暴露，不在渲染进程直接开系统权限。
- 渲染进程默认启用上下文隔离与沙箱，remote 模块已移除。
- 长任务放主进程或子进程，避免阻塞渲染与主线程。
- 数据持久化用系统目录（app.getPath），不写死绝对路径。
- 生产必须打包、签名并接入自动更新。

## 阶段成果

- 能初始化 Electron 项目并运行窗口。
- 能通过 preload + contextBridge 安全暴露 API。
- 能用 ipcMain/ipcRenderer 完成双向通信。
- 能实现多窗口、菜单、托盘与系统通知。
- 能读写本地文件并持久化应用数据。
- 能打包出可安装产物并接入自动更新。

## 每阶段固定学习模板

每个阶段按照“目标 -> 前置知识 -> 核心问题 -> 最小示例 -> 项目增量 -> 验收 -> 排错”执行。先完成阶段项目，再阅读扩展主题；这样能够把 Electron 概念连接到可运行结果。

## 版本边界

学习主线使用 Electron 当前稳定版（Chromium + Node.js 内置）。新老 API 差异（如 remote 移除、sandbox 默认开启）参见 [版本边界与迁移](./version-governance.md)。

## 前置知识

Electron 渲染进程就是 Web 页面，建议先掌握 [React 学习路线](../react-learning-path/README.md) 与 [TypeScript 学习路线](../typescript-learning-path/README.md)，再结合 [Node.js 学习路线](../nodejs-learning-path/README.md) 理解主进程的 Node 能力。桌面界面与 UI 可对照 [Tailwind CSS 学习路线](../tailwindcss-learning-path/README.md)。若关心与 Tauri 的选型差异，可对照 [Tauri 学习路线](../tauri-learning-path/README.md)。

## 官方资源

- [Electron 官方文档](https://www.electronjs.org/docs/latest/)
- [Electron API 参考](https://www.electronjs.org/docs/latest/api/app)
- [Electron 安全文档](https://www.electronjs.org/docs/latest/tutorial/security)
- [electron-vite 文档](https://electron-vite.org/)
- [electron-builder 文档](https://www.electron.build/)
