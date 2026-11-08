# Astryx 版本边界与迁移

**目标**：给学习内容和项目一套可执行的版本边界，降低 Beta 期 API 变动、主题与 Core 错位、以及和 Tailwind 混用的风险。

## 1. 当前主线

| 层级 | 学习主线 |
|---|---|
| 设计系统 | Astryx **Beta**（`@astryxdesign/*`） |
| UI 运行时 | React **19+** / react-dom **19+** |
| 样式 peer | `@stylexjs/stylex` |
| 默认主题 | `@astryxdesign/theme-neutral` |
| 工具 | `@astryxdesign/cli` |
| 语言 | TypeScript |

公开文档：[astryx.atmeta.com](https://astryx.atmeta.com/docs/getting-started)，源码：[github.com/meta/astryx](https://github.com/meta/astryx)。

Beta 意味着小版本可含 breaking change。学习与生产都把 **Core、主题、CLI 锁在同一发版**。

## 2. 每个示例应记录的元数据

```markdown
> 适用范围：Astryx Beta + React 19+ + TypeScript
> 需要的工具：Node.js、npm、Vite 或 Next.js
> 验证命令：npx @astryxdesign/cli component Button、npm run dev
> 最后复核日期：YYYY-MM-DD
```

组件 props 以当时 CLI 输出为准，文档示例只保证结构正确。

## 3. 包配套关系

| 包 | 角色 | 错位症状 |
|---|---|---|
| `@astryxdesign/core` | 组件与 Theme | 缺组件、类型对不上 |
| `@astryxdesign/theme-*` | Token 与主题 CSS | 视觉与组件“两张皮” |
| `@astryxdesign/cli` | 文档 / 模板 / upgrade | 模板 API 过期 |
| `@astryxdesign/richtext` | 可选富文本 | 与 Core 行为不一致 |
| `@stylexjs/stylex` | peer | xstyle 无法编译或运行 |

主题包声明它所匹配的 Core。只升其中一方就会出现文档有、运行无或颜色错乱。

## 4. 已知代际（学习期需记住的）

以官方 changelog 为准，学习中反复出现的几类：

| 变化 | 影响 |
|---|---|
| React 19 peer | 18 工程不要硬装 |
| 输入组件去掉根级 `xstyle`/`className` | 改 `control*` / `container*` |
| Banner 等 API + codemod | 先 `upgrade --apply` |
| 共享 dismissal 栈 | 不要多层各绑 Esc |
| 多语言 / RTL 进入 core | 产品文案仍要自管 i18n |
| 响应式主题规则（0.6+） | Token 可随断点变化 |

```bash
npx @astryxdesign/cli docs migration
npx @astryxdesign/cli upgrade --from <旧 Core 版本> --apply
```

## 5. 升级步骤

1. 记下当前 Core 版本（`--from` 需要）。
2. 同一命令升级所有 `@astryxdesign/*`。
3. 跑 `upgrade --from <旧版本> --apply`。
4. 手修 wrapper spread、codemod 覆盖不到的调用。
5. `tsc` + 主路径（列表、表单、对话框、暗色）。
6. pnpm 发布延迟设置需排除整个 `@astryxdesign/*`。

## 6. 与 Tailwind / 其它样式共存

- 官方支持：组件 Astryx，布局 Tailwind。
- 必须管理 `@layer`：reset、`astryx-base`、Tailwind utilities。
- 无 layer 的全局 CSS 会压过组件层。
- 深度定制顺序：Token -> `className`/`xstyle` -> 单组件 swizzle。不要 fork 整个 Core。

## 7. 升级前检查

1. 阅读当前 Getting Started 与 changelog。
2. 确认 React 19。
3. 列出全部 `@astryxdesign/*` 版本。
4. 列出 swizzle 过的组件。
5. 确认全局 CSS 与 layer。
6. 分支上升级，保留可回滚点。

## 8. 升级后检查

- Hello 按钮、表单校验、对话框焦点、Table 空态。
- 明暗色 Token 仍成对。
- CLI `component Button` 与代码 import 路径一致。
- 无控制台关于被忽略的根级输入样式警告。

## 9. 常见坑

- 在 React 18 项目里当普通 UI 库安装。
- 只装 Core、不装主题、不包 `<Theme>`。
- 从包根 import 全部组件。
- Core 与 Neutral 差一个小版本。
- TextInput 继续传根级 `className`。
- 助手未接 CLI/MCP，编造 variant。
- 为改品牌色 swizzle 了半个组件树。
- 全局旧 reset 不在 layer 里，盖掉 astryx-base。
