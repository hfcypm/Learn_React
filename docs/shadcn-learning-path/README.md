# shadcn/ui 学习路线图（基础 → 生产）

**目标**：掌握 shadcn/ui 的组件分发模式、主题定制、表单与数据展示，以及与 React + Tailwind CSS v4 的工程化集成，能够用可访问、可定制的组件快速搭建专业界面。

## 先看这里

- [新手导读](./start-here.md)：前置条件、学习顺序、排错顺序和完成标准。
- [综合实战：订阅管理后台](./project-practice.md)：按阶段逐步扩展同一个项目。
- [学习评估与项目评分](./learning-assessment.md)：阶段考核、排错题、项目评分和评审问题。

## 学习阶段总览

| 阶段 | 名称 | 目标 |
|---|---|---|
| 零 | shadcn/ui 基础与工具链 | 理解组件分发模式，初始化项目并添加第一个组件 |
| 一 | 核心组件与主题定制 | 掌握基础组件、CSS 变量与主题体系 |
| 二 | 交互组件与表单 | 掌握对话框、下拉、弹出层与表单校验 |
| 三 | 数据展示与状态 | 掌握数据表、图表、暗色模式与受控组件 |
| 四 | 工程化与生产交付 | 掌握组件变体、可访问性、测试与工程化 |

## 快速导航

- [阶段零：shadcn/ui 基础与工具链](./stage-0-foundations.md)
- [阶段一：核心组件与主题定制](./stage-1-core-and-theming.md)
- [阶段二：交互组件与表单](./stage-2-interactions-and-forms.md)
- [阶段三：数据展示与状态](./stage-3-data-and-state.md)
- [阶段四：工程化与生产交付](./stage-4-production.md)
- [综合实战：订阅管理后台](./project-practice.md)

## 学习原则

- shadcn/ui 不是组件依赖，而是把组件源码复制进你的项目，源码可读、可改、可维护。
- 主题用 CSS 变量驱动，改主题就是改变量，不逐组件覆盖。
- 先掌握基础组件与组合方式，再进入表单、数据表等复杂组件。
- 组件默认基于 Radix UI，可访问性开箱即用，不要在改样式时破坏语义。
- 组件样式基于 Tailwind CSS v4，先理解工具类与 CSS-first 配置。
- 一切组件源码都在项目里，需要时直接修改实现而不是另写包装层。

## 阶段成果

- 能初始化项目并安装 shadcn/ui。
- 能用 CSS 变量定制主题并切换暗色模式。
- 能用基础与交互组件搭建界面。
- 能实现带校验的表单与数据表。
- 能修改组件源码、管理变体并保证可访问性。
- 能把 shadcn/ui 纳入工程化流程并生产交付。

## 每阶段固定学习模板

每个阶段按照“目标 -> 前置知识 -> 核心问题 -> 最小示例 -> 项目增量 -> 验收 -> 排错”执行。先完成阶段项目，再阅读扩展主题；这样能够把 shadcn/ui 概念连接到可运行结果。

## 版本边界

学习主线使用 shadcn/ui 当前稳定版（面向 Tailwind CSS v4 与 React），依赖 `components.json` 与 CLI。历史版本配置差异参见 [版本边界与迁移](./version-governance.md)。

## 前置知识

shadcn/ui 构建在 React 与 Tailwind CSS 之上，建议先完成 [React 学习路线](../react-learning-path/README.md) 与 [TypeScript 学习路线](../typescript-learning-path/README.md)，再掌握 [Tailwind CSS 学习路线](../tailwindcss-learning-path/README.md) 的工具类与主题机制。在 Next.js 中使用可对照 [Next.js 学习路线](../nextjs-learning-path/README.md)，在 AI 应用中使用可对照 [AI 应用前端学习路线](../ai-learning-path/README.md)。

## 官方资源

- [shadcn/ui 官方文档](https://ui.shadcn.com/)
- [shadcn/ui 组件列表](https://ui.shadcn.com/docs/components/button)
- [shadcn/ui 主题](https://ui.shadcn.com/docs/theming)
- [shadcn/ui GitHub](https://github.com/shadcn-ui/ui)
- [Radix UI 文档](https://www.radix-ui.com/)
