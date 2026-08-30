# Tauri 学习路线图（基础 → 生产）

**目标**：掌握 Tauri 2 的前后端架构、Rust commands、权限系统、系统能力插件与打包发布，能够在 Web 前端基础上构建体积小、性能高、安全强的跨平台桌面应用。

## 先看这里

- [新手导读](./start-here.md)：前置条件、学习顺序、排错顺序和完成标准。
- [综合实战：跨平台文件管理器](./project-practice.md)：按阶段逐步扩展同一个项目。
- [学习评估与项目评分](./learning-assessment.md)：阶段考核、排错题、项目评分和评审问题。

## 学习阶段总览

| 阶段 | 名称 | 目标 |
|---|---|---|
| 零 | Tauri 基础与工具链 | 理解前后端架构，初始化项目并运行第一个窗口 |
| 一 | 配置与权限系统 | 掌握 tauri.conf.json、capabilities 与权限模型 |
| 二 | Rust 命令与后端 | 掌握 commands、invoke、状态与错误处理 |
| 三 | 系统能力与插件 | 掌握文件系统、对话框、事件与官方插件 |
| 四 | 打包发布与生产 | 掌握打包、签名、更新、移动端与性能 |

## 快速导航

- [阶段零：Tauri 基础与工具链](./stage-0-foundations.md)
- [阶段一：配置与权限系统](./stage-1-config-and-permissions.md)
- [阶段二：Rust 命令与后端](./stage-2-commands-and-backend.md)
- [阶段三：系统能力与插件](./stage-3-system-and-plugins.md)
- [阶段四：打包发布与生产](./stage-4-production.md)
- [综合实战：跨平台文件管理器](./project-practice.md)

## 学习原则

- 前端负责界面，Rust 后端负责能力，二者通过 invoke 通信。
- 安全默认：每个系统能力都要显式声明权限，最小授权。
- 命令入参必须经过类型校验，返回明确错误结构。
- 优先用官方插件获取系统能力，少手写不安全代码。
- 事件用于前后端通知，命令用于请求响应。
- 生产必须打包、签名并接入更新机制。

## 阶段成果

- 能初始化 Tauri 2 项目并运行窗口。
- 能读懂并配置 tauri.conf.json 与 capabilities。
- 能用 Rust 命令暴露类型安全的后端能力。
- 能用插件完成文件、对话框与系统交互。
- 能打包出跨平台产物并接入更新。
- 理解 Tauri 与 Electron 的差异与选型。

## 每阶段固定学习模板

每个阶段按照“目标 -> 前置知识 -> 核心问题 -> 最小示例 -> 项目增量 -> 验收 -> 排错”执行。先完成阶段项目，再阅读扩展主题；这样能够把 Tauri 概念连接到可运行结果。

## 版本边界

学习主线使用 Tauri 2（Rust 后端 + 任意前端框架）。Tauri 1 与 Tauri 2 的配置、权限与插件 API 存在差异，参见 [版本边界与迁移](./version-governance.md)。

## 前置知识

前端部分建议先掌握 [React 学习路线](../react-learning-path/README.md) 与 [TypeScript 学习路线](../typescript-learning-path/README.md)。Rust 部分需要基础语法与所有权概念，从阶段二逐步深入。若已从 Web 全栈学习过来，可先完成 [综合实战：团队任务看板](../fullstack-kanban/README.md) 再桌面化；若关心桌面框架选型，可对照 [Electron 学习路线](../electron-learning-path/README.md) 与 [桌面框架选型](../electron-vs-tauri.md)。

## 官方资源

- [Tauri 官方文档（2.x）](https://tauri.app/zh-cn/start/)
- [Tauri 插件仓库](https://v2.tauri.app/plugin/)
- [Tauri GitHub](https://github.com/tauri-apps/tauri)
- [Rust 官方书](https://doc.rust-lang.org/book/)
