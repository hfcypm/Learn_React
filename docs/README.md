# 前端学习文档

当前文档按技术主题拆分为独立学习包。不确定从哪开始，先看 [技术栈地图：按项目目标选择学习路线](./technology-roadmap.md)。

- [React 学习路线](./react-learning-path/README.md)
- [TypeScript 学习路线](./typescript-learning-path/README.md)
- [Next.js 学习路线](./nextjs-learning-path/README.md)
- [Tailwind CSS 学习路线](./tailwindcss-learning-path/README.md)
- [shadcn/ui 学习路线](./shadcn-learning-path/README.md)
- [Node.js 学习路线](./nodejs-learning-path/README.md)
- [NestJS 学习路线](./nestjs-learning-path/README.md)
- [PostgreSQL 学习路线](./postgresql-learning-path/README.md)
- [Prisma 学习路线](./prisma-learning-path/README.md)
- [AI 应用前端学习路线](./ai-learning-path/README.md)
- [LangChain 学习路线](./langchain-learning-path/README.md)
- [Electron 学习路线](./electron-learning-path/README.md)
- [Tauri 学习路线](./tauri-learning-path/README.md)
- [HarmonyOS 学习路线](./harmonyos-learning-path/README.md)
- [iOS 应用开发学习路线（Objective-C）](./ios-learning-path/README.md)
- [Perfetto 学习路线](./perfetto-learning-path/README.md)
- [Node.js 与 NestJS 服务端学习计划](./server-learning-study-plan.md)
- [桌面框架选型：Electron vs Tauri](./electron-vs-tauri.md)
- [综合实战：团队任务看板](./fullstack-kanban/README.md)

React 学习路线中的“TypeScript + React”章节保留 React 场景下的类型实践；完整 TypeScript 类型系统、泛型、条件类型、声明文件和类型安全 API 内容位于独立的 TypeScript 学习包。Next.js 学习路线覆盖全栈 React 应用的 App Router、渲染、数据获取、缓存和鉴权。Tailwind CSS 学习路线覆盖工具类系统、主题定制、响应式和工程化。Node.js 文档覆盖运行时与服务端基础，NestJS 文档覆盖基于 Node.js 的模块化企业应用开发。PostgreSQL 学习路线覆盖关系模型、SQL 查询、数据建模、事务、索引和生产运维。Prisma 学习路线覆盖基于 PostgreSQL 的 ORM Schema 建模、迁移、类型安全查询与工程化。

LangChain 学习路线覆盖基于 LangChain.js 的 LLM 应用开发：模型接入、提示与结构化输出、LCEL 链式组合、RAG 检索增强、Agent 与工具，以及记忆、流式与生产可观测性。

shadcn/ui 学习路线覆盖基于 Tailwind CSS v4 的 React 组件开发：源码分发模式、CSS 变量主题、Radix 交互、表单校验、数据表图表与生产工程化。

Electron 学习路线覆盖基于 Web 技术的跨平台桌面应用开发：双进程模型、preload 与安全 IPC、窗口与系统集成、文件持久化、打包签名与自动更新。Tauri 学习路线覆盖基于 Rust 后端的轻量桌面应用开发：架构、tauri.conf.json 与 capabilities 权限、Rust 命令、官方插件、打包发布与移动端支持，两路线可对照学习桌面框架选型。

HarmonyOS 学习路线覆盖纯血鸿蒙（HarmonyOS NEXT）原生应用开发：ArkTS 语言与严格模式、ArkUI 声明式 UI 与状态管理 V2、Stage 模型与 UIAbility、Navigation 路由导航、网络与 Preferences/RDB 持久化、HAR/HSP 模块化、TaskPool/Worker 并发、分布式协同、多设备自适应、性能发布，以及元服务与服务卡片、ExtensionAbility 家族、启动模式与 AppStartup、ArkWeb、多语言与深色模式、通知与后台任务、安全隐私等进阶专题，并提供从入门到精通的五阶段路径与多设备记账元服务综合实战。

Perfetto 学习路线覆盖 Android 系统级 tracing：System Trace 录制、Perfetto UI、FrameTimeline 红帧、Trace Processor SQL、自定义 atrace 埋点，以及与 Studio Profiler / LeakCanary / APA 的分工。适合客户端卡顿与掉帧排查，和仓库 Web 主线并行。

iOS 应用开发学习路线以 Objective-C + UIKit 为主线，面向既有老工程的接手与维护：OC 语法与 ARC、属性修饰符、block 与循环引用、UIViewController 与 SceneDelegate 生命周期、Storyboard/XIB 与 Auto Layout、UITableView 复用、NSURLSession 与 JSON、GCD/NSOperationQueue、Core Data 与 Keychain、CocoaPods、OC/Swift 混编、XCTest、崩溃排查与签名上架，并提供三周快速入门路径与老版习惯打卡 App 综合实战。

综合实战 `fullstack-kanban` 以「团队任务看板」应用为线索，将 Next.js、Tailwind CSS、Prisma、PostgreSQL 四条路线的知识点穿插进初始化、建模、鉴权、取数、界面、查询事务与生产交付七个阶段，适合在完成各路线后串联验证。
