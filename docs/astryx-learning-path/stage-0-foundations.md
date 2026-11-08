# 阶段零：设计系统与工具链

**目标**：理解 Astryx 的包模型与接入方式，初始化项目，包上 `<Theme>`，渲染第一个 Button。

> 适用范围：Astryx Beta + React 19+ + TypeScript
> 需要的工具：Node.js、npm、Vite 或 Next.js
> 验证命令：`npx @astryxdesign/cli component Button`、`npm run dev`
> 最后复核日期：2026-11-08

**前置知识**：React 组件与 TypeScript。不熟悉时先读 [React 学习路线](../react-learning-path/README.md)。

## 1. Astryx 是什么

Astryx 是 Meta 开源的 React 设计系统：150+ 组件、主题包、CLI 与 MCP。内部用了约八年，公开仓库在 [meta/astryx](https://github.com/meta/astryx)，目前是 Beta。

```text
@astryxdesign/core          组件与 Theme
@astryxdesign/theme-*       主题 CSS 与 Token
@astryxdesign/cli           文档、模板、升级、MCP
@stylexjs/stylex            peer：组件内部样式运行时
```

和 shadcn/ui 的关键差异：Astryx 是 **npm 依赖**，升级跟版本走；shadcn 是把源码复制进仓库。深度改某个组件时再用 `astryx swizzle`。

## 2. 前置环境

```bash
npm create vite@latest astryx-admin -- --template react-ts
cd astryx-admin
npm install
```

确认 `package.json` 里 `react` / `react-dom` 为 19 及以上。Next.js 示例见官方 `apps/example-nextjs`。

## 3. 安装

```bash
npm install @astryxdesign/core @stylexjs/stylex @astryxdesign/theme-neutral @astryxdesign/cli
npx @astryxdesign/cli init
```

`init` 无交互，会写入 AI 助手说明书（`AGENTS.md` / `CLAUDE.md`）。加 `--all` 会附带主题与页面搭建指引。

建议在 `package.json` 固定入口，避免助手找不到 CLI：

```json
{
  "scripts": {
    "astryx": "node node_modules/@astryxdesign/cli/clients/cli/bin/astryx.mjs",
    "dev": "vite"
  }
}
```

之后用 `npm run astryx -- component --list`。

## 4. 全局 CSS 与 Theme

`src/index.css`（或 Next 的全局 CSS）：

```css
@import '@astryxdesign/core/reset.css';
@import '@astryxdesign/core/astryx.css';
```

添加 Neutral 主题：

```bash
npx @astryxdesign/cli theme add neutral
```

CLI 会生成主题模块（如 `astryx-themes`），并导入该主题的生产 CSS 与字体 CSS。

应用根：

```tsx
import {StrictMode} from 'react';
import {createRoot} from 'react-dom/client';
import {Theme} from '@astryxdesign/core/theme';
import {themes, defaultThemeSlug} from './astryx-themes';
import App from './App';
import './index.css';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <Theme theme={themes[defaultThemeSlug]} mode="system">
      <App />
    </Theme>
  </StrictMode>,
);
```

- `reset.css` 在 `@layer reset`，组件样式在 `@layer astryx-base`。
- 项目已有 Tailwind 或旧 reset 时，必须显式声明 layer 顺序，见阶段四。
- `mode="system"` 跟随 `prefers-color-scheme`。

官方主题包（起步用 Neutral）：

| 包 | 观感 |
|---|---|
| `theme-neutral` | 克制、适合内部工具 |
| `theme-butter` | 暖金 + 蓝强调 |
| `theme-chocolate` | 巧克力 / 焦糖 |
| `theme-gothic` | 仅暗色，墨色 |
| `theme-matcha` | 草木绿 |
| `theme-stone` | 砂岩暖中性 |
| `theme-y2k` | 早期 2000s |

## 5. 第一个组件

按子路径导入，路径以 CLI 打印为准：

```tsx
import {Button} from '@astryxdesign/core/Button';
import {VStack} from '@astryxdesign/core/VStack';

export default function App() {
  return (
    <VStack gap={2}>
      <Button label="Hello Astryx" onClick={() => alert('Hi!')} />
    </VStack>
  );
}
```

```bash
npx @astryxdesign/cli component Button
npx @astryxdesign/cli component --list
```

不要从 `@astryxdesign/core` 根路径一次性导入全部组件。

## 6. 工程结构

```text
astryx-admin/
  src/
    App.tsx
    astryx-themes.ts      # CLI theme add 生成
    index.css             # reset + astryx.css
    main.tsx              # <Theme> 包根
  package.json
  AGENTS.md               # CLI init 生成
```

官方完整示例：

| 栈 | 仓库路径 |
|---|---|
| Next.js + 主题 CSS | `apps/example-nextjs` |
| Next.js + StyleX | `apps/example-nextjs-stylex` |
| Next.js + Tailwind | `apps/example-nextjs-tailwind` |
| Vite + Tailwind | `apps/example-vite-tailwind` |

## 7. CLI 起步命令

```bash
npx @astryxdesign/cli help
npx @astryxdesign/cli docs
npx @astryxdesign/cli docs tokens --dense
npx @astryxdesign/cli theme list
npx @astryxdesign/cli template --list
```

`--dense` 给 LLM 上下文瘦身，人读完整版即可。

## 阶段验收

- 能解释 Core、主题包、CLI、StyleX peer 各自职责。
- 能安装并 `init`，全局 CSS 含 reset 与 astryx.css。
- 能用 `<Theme>` 包住应用并渲染 Button。
- 能用 CLI 列出组件并查看 Button 文档。

## 动手任务

1. 用 Vite 建 React 19 + TS 工程并安装四件套。
2. 运行 `theme add neutral`，根上包 `<Theme mode="system">`。
3. 页面放一个 `VStack` + `Button`，点击弹出 alert。
4. 在 `package.json` 加 `astryx` script，跑 `component Button`。
5. 把系统外观切到暗色，确认按钮颜色跟着变。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| peer 依赖警告 / 运行时报 React 版本 | `react` 是否 19+ |
| 页面像没样式的原生按钮 | 是否漏了 CSS 导入或 `<Theme>` |
| 找不到 `./astryx-themes` | 是否执行 `theme add` |
| `Cannot find module '@astryxdesign/core/Button'` | 包是否装上，子路径是否与 CLI 一致 |
| CLI 命令找不到 | 用 `npx @astryxdesign/cli` 或 npm script |

## 进入下一阶段的条件

Hello Astryx 能在明暗色下运行，你能说清三个包的分工。此时进入 [阶段一：核心组件与主题](./stage-1-core-and-theming.md)。
