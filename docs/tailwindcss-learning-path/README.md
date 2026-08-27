# Tailwind CSS 学习路线图（基础 → 生产）

**目标**：掌握 Tailwind CSS 的工具类系统、主题定制、响应式设计、组件复用和工程化实践，能够构建一致、可维护、高性能的界面。

## 先看这里

- [新手导读](./start-here.md)：前置条件、学习顺序、排错顺序和完成标准。
- [综合实战：后台管理界面](./project-practice.md)：按阶段逐步扩展同一个项目。
- [学习评估与项目评分](./learning-assessment.md)：阶段考核、排错题、项目评分和评审问题。

## 学习阶段总览

| 阶段 | 名称 | 目标 |
|---|---|---|
| 零 | 工具链与基础 | 理解 utility-first、安装配置和开发流程 |
| 一 | 核心概念与布局 | 掌握布局、间距、颜色、排版和常用工具类 |
| 二 | 响应式、主题与交互 | 掌握断点、暗色模式、状态变体和动效 |
| 三 | 组件化与进阶模式 | 掌握 @apply、组件复用、CSS-first 配置和插件 |
| 四 | 工程化与生产交付 | 掌握构建优化、可维护性、无障碍和部署 |

## 快速导航

- [阶段零：工具链与基础](./stage-0-tooling.md)
- [阶段一：核心概念与布局](./stage-1-core-layout.md)
- [阶段二：响应式、主题与交互](./stage-2-responsive-theme.md)
- [阶段三：组件化与进阶模式](./stage-3-components-advanced.md)
- [阶段四：工程化与生产交付](./stage-4-production.md)
- [综合实战：后台管理界面](./project-practice.md)

## 学习原则

- 优先使用工具类直接编写样式，减少自定义 CSS。
- 在 HTML 中看到结构时，应能直接看出布局和间距。
- 把重复的工具类组合抽取为组件或可复用样式，保持一致性。
- 通过主题变量定义颜色、字体和间距，避免散落的魔法值。
- 移动优先：默认样式写小屏，断点前缀覆盖大屏。
- 关注构建体积和无障碍，不只为写起来快。

## 阶段成果

- 能使用工具类完成页面布局、间距、排版和颜色。
- 能使用断点、暗色模式、状态变体和动效。
- 能通过 @theme、@apply 和组件抽取组织可维护样式。
- 能控制构建体积并接入主流框架。
- 能交付可访问、一致、可维护的界面。

## 每阶段固定学习模板

每个阶段按照“目标 -> 前置知识 -> 核心问题 -> 最小示例 -> 项目增量 -> 验收 -> 排错”执行。先完成阶段项目，再阅读扩展主题；这样能够把工具类知识连接到可运行结果。

## 版本边界

学习主线对应 Tailwind CSS v4 的 CSS-first 配置（`@import "tailwindcss"`、`@theme`）。旧项目中的 v3 JavaScript 配置（`tailwind.config.js`）只用于维护场景。版本细节参见 [版本边界与迁移](./version-governance.md)。

## 官方资源

- [Tailwind CSS 官方文档](https://tailwindcss.com/docs)
- [Tailwind CSS 升级指南](https://tailwindcss.com/docs/upgrade-guide)
- [Tailwind CSS GitHub](https://github.com/tailwindlabs/tailwindcss)
