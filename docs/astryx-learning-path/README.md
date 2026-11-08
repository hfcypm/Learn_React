# Astryx 学习路线图（入门 → 进阶 → 实战）

**目标**：掌握 Meta 开源设计系统 Astryx 的接入方式、主题与 Token、核心组件、表单与浮层、表格与模板，以及面向人和 AI 助手的 CLI / MCP 工作流。学完后能用 React 19 + TypeScript 交付一套主题统一、可访问、可升级的界面。

Astryx 是 **React 组件库**（设计系统），不是 Vue / Svelte 方案。内部样式用 StyleX 编写，使用侧导入预构建 CSS，不强制项目改用 StyleX。

## 先看这里

- [新手导读](./start-here.md)：前置条件、学习顺序、排错顺序和完成标准。
- [综合实战：内部订阅管理后台](./project-practice.md)：按阶段扩展同一个项目。
- [学习评估与项目评分](./learning-assessment.md)：阶段考核、排错题、项目评分和评审问题。
- [版本边界与迁移](./version-governance.md)：Beta 基线、主题与 Core 配套、升级 codemod。

## 学习阶段总览

| 阶段 | 名称 | 目标 |
|---|---|---|
| 入门零 | 设计系统与工具链 | 理解包模型，初始化项目并跑通第一个组件 |
| 入门一 | 核心组件与主题 | 掌握布局、按钮、卡片、Token 与明暗色 |
| 进阶二 | 表单、浮层与交互 | 掌握 TextInput、校验状态、对话框与焦点管理 |
| 进阶三 | 数据展示与 AI 工作流 | 掌握 Table、模板、CLI 查询与 MCP |
| 进阶四 | 定制、升级与生产 | 掌握 defineTheme、swizzle、cascade layer 与升级 |
| 实战 | 内部订阅管理后台 | 把入门到进阶串成一个可运行后台 |

## 快速导航

**入门**

- [阶段零：设计系统与工具链](./stage-0-foundations.md)
- [阶段一：核心组件与主题](./stage-1-core-and-theming.md)

**进阶**

- [阶段二：表单、浮层与交互](./stage-2-interactions-and-forms.md)
- [阶段三：数据展示与 AI 工作流](./stage-3-data-and-agents.md)
- [阶段四：定制、升级与生产](./stage-4-production.md)

**实战**

- [综合实战：内部订阅管理后台](./project-practice.md)
- [学习评估与项目评分](./learning-assessment.md)
- [版本边界与迁移](./version-governance.md)

## 架构主线

```text
React 19 App
  -> 全局 CSS（reset.css + astryx.css + 主题 CSS）
  -> <Theme theme={...} mode="system">
      -> Layout（VStack / HStack）
          -> Button / TextInput / Card / Dialog / Table
              -> Token（CSS 自定义属性）
              -> className / xstyle 覆盖
  -> CLI / MCP（文档、模板、升级、给 AI 的契约）
```

组件从 `@astryxdesign/core/<Name>` 按需导入。主题决定 Token，组件消费 Token。CLI 是人和编码助手共用的说明书。

## 学习原则

- Astryx 是 **npm 包依赖**，不是把源码复制进仓库；深度定制再用 `swizzle`。
- 组件按子路径导入，保持打包体积与意图清晰。
- 先接 Neutral 主题跑通，再 `defineTheme({ extends })` 改品牌，不要一上来 fork 主题包。
- 颜色、间距、圆角写 Token，不写死十六进制。
- 样式覆盖优先 `className`（可与 Tailwind 并存）或 `xstyle`；输入类组件用 `controlXstyle` / `containerXstyle`。
- 查 API 先跑 CLI：`npx @astryxdesign/cli component Button`，再写代码。
- Core 与主题包升到同一发版；升级用 `astryx upgrade --from <旧版本> --apply`。

## 阶段成果

- 能在 Vite 或 Next.js 中安装 Core、主题与 CLI，并用 `<Theme>` 包住应用。
- 能用 VStack、Button、Card、Heading 搭出页面骨架。
- 能切换明暗色，并用 Token 做品牌色覆盖。
- 能实现带 label、校验状态与必填的表单，以及对话框确认流。
- 能用 Table 展示数据，用模板加速页面搭建。
- 能用 CLI / MCP 查询组件契约，减少 AI 幻觉 props。
- 能定制主题、处理 cascade layer，并完成版本升级。

## 每阶段固定学习模板

每个阶段按照“目标 -> 前置知识 -> 核心问题 -> 最小示例 -> 项目增量 -> 验收 -> 排错”执行。先在官方 Playground 或本地跑通最小示例，再把能力加进综合实战。

## 版本边界

学习主线面向 **Astryx Beta + React 19+ + TypeScript**，包名为 `@astryxdesign/*`。当前仍是 Beta，API 可能随小版本调整；升级以 CLI codemod 与 [版本边界与迁移](./version-governance.md) 为准。

## 与 shadcn/ui 的分工

| 维度 | Astryx | shadcn/ui |
|---|---|---|
| 分发 | npm 包 | 源码复制进项目 |
| 样式内核 | StyleX（对使用者可不可见） | Tailwind 工具类 |
| 主题 | 主题包 + `defineTheme` + CSS 变量 | CSS 变量 + `components.json` |
| 查询方式 | CLI / MCP 一等公民 | 文档站 + CLI add |
| 适用 | 要一套完整设计系统、人和 AI 共用契约 | 要完全掌控源码、深度改组件 |

两条路线都建立在 React 上，可对照学习，不必绑在同一个项目里。

## 前置知识

建议先完成 [React 学习路线](../react-learning-path/README.md) 与 [TypeScript 学习路线](../typescript-learning-path/README.md)。用 Tailwind 做布局时可对照 [Tailwind CSS 学习路线](../tailwindcss-learning-path/README.md)。在 Next.js 中接入可对照 [Next.js 学习路线](../nextjs-learning-path/README.md)。已学过 [shadcn/ui 学习路线](../shadcn-learning-path/README.md) 时，重点对比“包依赖 vs 源码分发”和主题机制。

## 官方资源

- [Astryx 文档](https://astryx.atmeta.com/docs/getting-started)
- [组件列表](https://astryx.atmeta.com/components)
- [主题](https://astryx.atmeta.com/themes)
- [模板](https://astryx.atmeta.com/templates)
- [Playground](https://astryx.atmeta.com/playground)
- [GitHub：meta/astryx](https://github.com/meta/astryx)
