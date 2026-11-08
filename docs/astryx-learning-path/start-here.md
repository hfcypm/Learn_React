# Astryx 新手导读

## 适合谁

这套路线适合已经掌握 React 与 TypeScript、准备用一套完整设计系统搭内部工具、中后台或产品界面的开发者。学习重点是包接入、主题 Token、组件组合，以及用 CLI / MCP 让人和 AI 助手读同一份契约。

## 开始前准备

- React 组件、Hooks、Props 与 TypeScript 基础，可参考 [React 学习路线](../react-learning-path/README.md) 与 [TypeScript 学习路线](../typescript-learning-path/README.md)。
- 一个 Vite 或 Next.js 工程；本路线示例默认 Vite + React + TS。
- Node.js 与 npm。
- **React 19+**：`react` / `react-dom` 是 `@astryxdesign/core` 的 peer 依赖。

```bash
node --version
npm --version
```

## 核心观念

- **设计系统**：Token（颜色、间距、圆角、字体）+ 组件 + 主题 + 文档，一起约束界面。
- **包依赖**：从 `@astryxdesign/core/<Name>` 导入现成组件，升级跟 npm 版本走。
- **主题包**：如 `@astryxdesign/theme-neutral`，提供 CSS 自定义属性；`<Theme>` 在运行时挂上当前主题。
- **CLI**：查组件 props、Token、模板、升级；也是 AI 助手的入口。
- **Agent-ready**：`astryx init` 写入 `AGENTS.md`，MCP 让模型按契约生成代码。

一句话：组件用 Core，观感用主题，不懂就问 CLI。

## 学习顺序

| 阶段 | 先回答的问题 | 阶段产出 |
| --- | --- | --- |
| 入门零 | 怎么安装、怎么包 Theme、第一个 Button 从哪导入？ | 能运行的 Hello Astryx |
| 入门一 | 布局、卡片、Token、明暗色怎么组织？ | 主题统一的页面骨架 |
| 进阶二 | 表单校验和对话框怎么做？ | 带校验的新建流程 |
| 进阶三 | 表格、模板、CLI/MCP 怎么用？ | 数据页 + 可查询的组件契约 |
| 进阶四 | 品牌主题、升级、生产样式层怎么处理？ | 可定制、可升级的工程 |
| 实战 | 如何串成一个后台？ | 内部订阅管理后台 |

## 每阶段学习方法

1. 先跑 `npx @astryxdesign/cli component <Name>` 看 props 与示例。
2. 在本地最小页面验证，再写入综合实战。
3. 从阶段一就改 Token / `defineTheme`，少写死颜色。
4. 键盘走一遍主按钮、输入框、对话框。
5. 让 AI 写 UI 时先接 CLI 或 MCP，再生成代码。

## 常见排错顺序

```text
安装失败 / 类型报错
  -> React 是否 19+
  -> @stylexjs/stylex 是否作为 peer 安装
  -> Core 与主题包是否同一发版
样式全无 / 像未重置的 HTML
  -> 是否导入 reset.css 与 astryx.css
  -> 是否用 CLI theme add 并包 <Theme>
  -> 与 Tailwind 共存时 cascade layer 顺序
组件找不到
  -> 是否从 @astryxdesign/core/<Name> 导入
  -> 名称是否与 CLI component --list 一致
主题不切换
  -> <Theme> 是否在根上
  -> mode 与 Token 的 [light, dark] 元组
输入框样式无效
  -> 是否误用根级 xstyle（应 controlXstyle / containerXstyle）
AI 写出不存在的 variant
  -> MCP / CLI 是否接入，契约是否过期
```

## 阶段项目路线

从 [综合实战：内部订阅管理后台](./project-practice.md) 开始，每完成一个阶段增加一块：初始化、主题与导航、表单对话框、表格与模板、品牌主题与升级。

## 完成标准

- 能安装 Core、主题、CLI，并用 `<Theme>` 渲染首个页面。
- 能用 Token 与主题包统一明暗色。
- 能实现带校验状态的表单与对话框。
- 能用 Table 与官方模板搭数据页。
- 能用 CLI 查询组件，完成一次主题定制与升级演练。
- 完成 [学习评估](./learning-assessment.md) 中的阶段考核与项目评分。

下一步：[阶段零：设计系统与工具链](./stage-0-foundations.md)。
