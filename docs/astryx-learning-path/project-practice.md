# 综合实战：内部订阅管理后台

## 1. 项目目标

用 Astryx 构建一个内部订阅管理后台，覆盖安装与 Theme、页面骨架、新建表单与对话框、数据表、品牌主题与升级纪律。项目按阶段扩展，最终交付主题统一、可访问、人和 AI 都能按契约改的界面。

```text
初始化 -> 主题与导航骨架 -> 表单对话框
    -> 数据表与模板 -> 品牌 Token 与生产检查
```

选择“订阅后台”是为了和 [shadcn/ui 综合实战](../shadcn-learning-path/project-practice.md) 对照：同一类产品，两套组件分发模型。

## 2. 需求

**必须实现（MVP）**

- 页头：产品名、订阅列表标题、新建按钮。
- 指标区：订阅数、月费合计、即将到期，三张 Card。
- 订阅列表：客户、套餐、状态、月费；空态可点新建。
- 新建订阅：对话框内名称 + 邮箱，名称必填校验。
- 明暗色跟随系统。

**进阶**

- Neutral 之上 `defineTheme` 品牌强调色。
- 从官方模板拷一页仪表盘再改。
- 行内删除确认。
- `AGENTS.md` + npm `astryx` script，生成 UI 前查 CLI。
- 与 Tailwind 并存时写明 layer 顺序。

## 3. 技术选型

| 技术 | 用途 |
|---|---|
| React 19 + TypeScript | 界面 |
| Vite | 开发服务器 |
| `@astryxdesign/core` | 组件与 Theme |
| `@astryxdesign/theme-neutral` | 默认主题 |
| `@astryxdesign/cli` | 文档、模板、升级 |
| `@stylexjs/stylex` | peer / 可选 xstyle |
| 可选 Tailwind | 仅页面级布局 |

## 4. 项目结构

```text
astryx-admin/
  src/
    main.tsx
    App.tsx
    index.css
    astryx-themes.ts
    theme-brand.ts
    features/
      subscriptions/
        SubscriptionPage.tsx
        SubscriptionTable.tsx
        SubscriptionCreateDialog.tsx
        types.ts
  AGENTS.md
  package.json
```

## 5. 阶段零：初始化

```bash
npm create vite@latest astryx-admin -- --template react-ts
cd astryx-admin
npm install
npm install @astryxdesign/core @stylexjs/stylex @astryxdesign/theme-neutral @astryxdesign/cli
npx @astryxdesign/cli init
npx @astryxdesign/cli theme add neutral
```

`index.css`：

```css
@import '@astryxdesign/core/reset.css';
@import '@astryxdesign/core/astryx.css';
```

`main.tsx` 用 `<Theme theme={themes[defaultThemeSlug]} mode="system">` 包 `<App />`。

验收：Hello 按钮可点，系统暗色下表面色变化。

## 6. 阶段一：骨架与指标

`SubscriptionPage`：`VStack` 包页头 `HStack`（`Heading` + `Button`），下方三张 `Card` 指标。间距只用 `gap` Token。

验收：浅色 / 深色下标题与主按钮对比度足够。

## 7. 阶段二：新建对话框

`SubscriptionCreateDialog`：

- 触发器：页头「新建」。
- `TextInput`：名称 `isRequired`，邮箱带 `description`。
- 名称为空或仅空格：`status={{type:'error', message:'名称不能为空'}}`。
- 保存写入内存列表并关闭；取消不写。

验收：键盘走完打开、填写、校验、保存、焦点回到新建按钮。

## 8. 阶段三：表格与模板

`SubscriptionTable` 使用 `Table` + mock 数据。空列表显示引导。可选：`template --list` 拷一页改路由。

`package.json`：

```json
"scripts": {
  "astryx": "node node_modules/@astryxdesign/cli/clients/cli/bin/astryx.mjs"
}
```

验收：有数据时出表，无数据时出空态；`npm run astryx -- component Table` 能跑。

## 9. 阶段四：品牌与生产

```tsx
import {defineTheme} from '@astryxdesign/core/theme';
import {themes, defaultThemeSlug} from './astryx-themes';

export const brandTheme = defineTheme({
  extends: themes[defaultThemeSlug],
  name: 'internal-admin',
  tokens: {
    '--color-accent': ['#0064E0', '#2694FE'],
  },
});
```

生产检查：Core 与主题同版本、单根 Theme、无大面积硬编码色、`AGENTS.md` 已提交。

## 10. 实施顺序

1. Vite 工程 + 四件套 + `init` + `theme add neutral`。
2. 根上 `<Theme mode="system">`。
3. 页头与三张指标 Card。
4. 内存数组 + Table。
5. 对话框新建与校验。
6. 空态与删除确认。
7. `defineTheme` 品牌色。
8. 固定 `astryx` script，键盘走查。

## 11. 验收清单

- [ ] React 19，Core / 主题 / CLI 已安装。
- [ ] reset + astryx CSS + `<Theme>` 生效。
- [ ] 列表、指标、新建主路径可走通。
- [ ] TextInput 错误状态可见。
- [ ] 系统暗色下可阅读。
- [ ] 组件均从 `@astryxdesign/core/<Name>` 导入。
- [ ] 可用 CLI 查询 Button 与 Table。
- [ ] 品牌色来自 Token，而不是组件上写死 hex。

## 12. 按阶段学习卡片

| 阶段 | 项目增量 | 关键能力 |
|---|---|---|
| 入门零 | 安装、Theme、Hello Button | 包模型、CSS、子路径导入 |
| 入门一 | 页头与指标 Card | 布局、Token、明暗色 |
| 进阶二 | 新建对话框 | TextInput、status、浮层 |
| 进阶三 | Table、空态、CLI script | 数据展示、模板、契约 |
| 进阶四 | defineTheme、检查清单 | 品牌、升级、layer |
