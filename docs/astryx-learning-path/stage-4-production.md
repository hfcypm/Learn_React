# 阶段四：定制、升级与生产

**目标**：用 `defineTheme` 做品牌覆盖，处理与 Tailwind 共存的 cascade layer，按需 swizzle，并用官方升级命令保持 Core 与主题同发版。

> 适用范围：Astryx Beta + React 19+ + TypeScript
> 需要的工具：Node.js、npm、已完成阶段三的工程
> 验证命令：`npx @astryxdesign/cli docs theme`、`npx @astryxdesign/cli docs migration`
> 最后复核日期：2026-11-08

**前置知识**：阶段三的列表、模板与 CLI。

## 1. 品牌主题：extends 而不是 fork

```tsx
import {defineTheme} from '@astryxdesign/core/theme';
import {themes, defaultThemeSlug} from './astryx-themes';

const brandTheme = defineTheme({
  extends: themes[defaultThemeSlug],
  name: 'acme',
  tokens: {
    '--color-accent': ['#0064E0', '#2694FE'],
    '--radius-container': '12px',
  },
});

<Theme theme={brandTheme} mode="system">
  <App />
</Theme>
```

- Token 的 `[light, dark]` 元组：第一项浅色，第二项深色。
- 单值表示两模式相同。
- 组件级覆盖走 `defineTheme` 的 `components` 字段，先查：

```bash
npx @astryxdesign/cli theme targets
npx @astryxdesign/cli theme template
npx @astryxdesign/cli docs theme
```

`theme eject` 会变成独立源码分叉，只在明确要脱离主题包时使用。日常品牌色用 `extends`。

SSR / 生产可改用主题的 `/built` 导入并带上对应 `theme.css`，减少运行时编译。细节见：

```bash
npx @astryxdesign/cli docs theme
```

## 2. Cascade layer 与 Tailwind

Astryx 的 reset 在 `@layer reset`，组件在 `@layer astryx-base`。**未分层的样式和更后的 layer 都会压过 astryx-base**，与选择器权重无关。

项目已有 Tailwind 或旧全局 CSS 时：

1. 显式声明 `@layer` 顺序。
2. 每一份全局表都放进某个 layer，不要让无 layer 的重置文件垫后。
3. 读迁移文档的 cascade 两节：

```bash
npx @astryxdesign/cli docs migration
```

布局用 Tailwind、组件用 Astryx 是官方支持的组合（`apps/example-vite-tailwind`）。冲突时先查 layer，再查是否用 Token。

## 3. swizzle：按需弹出源码

```bash
npx @astryxdesign/cli swizzle Button
```

只在主题与 `xstyle` 不够时弹出单个组件。弹出后：

- 该组件升级靠你自己合并。
- 尽量少 swizzle；能 Token 解决的不要弹。
- 弹出文件纳入 Code Review，注明为什么包内能力不够。

## 4. 升级

主题包绑定确切的 Core 发版。只升 Core 不升主题会错位。

```bash
npm install @astryxdesign/core@latest @astryxdesign/theme-neutral@latest @astryxdesign/cli@latest
npx @astryxdesign/cli upgrade --from <旧 Core 版本> --apply
```

应用里其它 `@astryxdesign/*` 一并写进同一条 install。pnpm 若开了最低发布年龄，排除整个 `@astryxdesign/*` 作用域，避免同一仓库里多个发版并存。

升级后必做：

1. 跑 TypeScript。
2. 过一遍主按钮、表单、对话框、表格。
3. 读 changelog：输入组件样式入口、Banner API 等都曾做过 breaking change。
4. 有 codemod 的先 apply，再手修 spread 出去的 props。

## 5. i18n、RTL 与富文本

- 0.5 起核心带多语言目录与 RTL 相关能力，产品文案仍要走你自己的 i18n。
- 表单富文本用独立包 `@astryxdesign/richtext`（基于 Lexical，Markdown 往返），需要时再装，保持与 Core 同发版。

```bash
npx @astryxdesign/cli docs internationalization
```

## 6. 生产检查清单

- [ ] `react` / `react-dom` >= 19
- [ ] Core、主题、CLI 同一发版
- [ ] 根上只有一处 `<Theme>`
- [ ] 全局 CSS 含 reset + astryx + 主题 CSS
- [ ] 品牌改动在 `defineTheme`，无大面积硬编码色
- [ ] 与 Tailwind 共存时 layer 顺序已声明
- [ ] `AGENTS.md` 已提交，助手走项目内 CLI
- [ ] 键盘与一条屏幕阅读器路径走过新建 / 列表
- [ ] 升级演练：`--from` 有记录

## 阶段验收

- 能用 `extends` 做出品牌强调色，明暗色都正确。
- 能说明为什么无 layer 的 CSS 会盖掉组件。
- 能描述 swizzle 的成本。
- 能执行一次 `upgrade --from` 并解释为何主题必须一起升。

## 动手任务

1. `defineTheme` 覆盖 `--color-accent` 为品牌色。
2. 若项目有 Tailwind，写下当前 `@layer` 顺序注释。
3. 对一个非关键展示组件做一次 swizzle，看生成文件后还原（或另开分支丢弃）。
4. 记录当前 Core 版本，走一遍 upgrade 文档（可在分支上）。
5. 用键盘 + 系统暗色再验收订阅列表与新建对话框。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 改了 Token 组件不变 | 是否 `extends` 了正在使用的那份 theme，是否包在 `<Theme>` |
| Tailwind `bg-white` 盖掉 Card | unlayered 工具类 vs `astryx-base` |
| 升级后 TextInput 样式丢失 | 根级 `xstyle` 已被忽略 |
| 主题与组件视觉分裂 | Core 与 `theme-*` 版本不一致 |
| swizzle 后无法升级 | 预期行为，减少弹出面 |

## 进入实战的条件

品牌主题能切换，升级命令能说清，layer 不再是黑盒。按 [综合实战](./project-practice.md) 把后台做完整。
