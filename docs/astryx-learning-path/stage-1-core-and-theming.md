# 阶段一：核心组件与主题

**目标**：用布局、按钮、卡片、标题搭出页面骨架，理解 Token 与明暗色，能用 `className` / `xstyle` 做局部覆盖。

> 适用范围：Astryx Beta + React 19+ + TypeScript
> 需要的工具：Node.js、npm、已完成阶段零的工程
> 验证命令：`npx @astryxdesign/cli docs tokens --dense`、`npx @astryxdesign/cli component Card`
> 最后复核日期：2026-11-08

**前置知识**：阶段零的安装、`<Theme>` 与按子路径导入。

## 1. 布局

页面结构优先用 Astryx 的 stack，而不是一上来写大量 CSS。

```tsx
import {VStack} from '@astryxdesign/core/VStack';
import {HStack} from '@astryxdesign/core/HStack';
import {Heading} from '@astryxdesign/core/Heading';
import {Button} from '@astryxdesign/core/Button';

export function PageHeader() {
  return (
    <HStack gap={3} justify="space-between">
      <Heading level={1}>订阅</Heading>
      <Button label="新建" variant="primary" />
    </HStack>
  );
}

export function PageBody({children}: {children: React.ReactNode}) {
  return (
    <VStack gap={4}>
      <PageHeader />
      {children}
    </VStack>
  );
}
```

`gap` 走间距 Token，不要写死 `16px`。更完整的布局规则：

```bash
npx @astryxdesign/cli docs layout
npx @astryxdesign/cli docs tokens --dense
```

与 Tailwind 并存时：Astryx 管组件，Tailwind 管页面级 `min-h-screen`、栅格。官方模式见 `apps/example-vite-tailwind`。

## 2. Button、Card、Badge

```tsx
import {Button} from '@astryxdesign/core/Button';
import {Card} from '@astryxdesign/core/Card';
import {VStack} from '@astryxdesign/core/VStack';
import {Heading} from '@astryxdesign/core/Heading';

<VStack gap={3}>
  <HStack gap={2}>
    <Button label="保存" variant="primary" />
    <Button label="取消" variant="secondary" />
    <Button label="删除" variant="destructive" />
  </HStack>
  <Card>
    <VStack gap={2}>
      <Heading level={2}>本月收入</Heading>
      <Button label="查看明细" variant="ghost" />
    </VStack>
  </Card>
</VStack>
```

实际 `variant` / `size` 以 CLI 为准：

```bash
npx @astryxdesign/cli component Button
npx @astryxdesign/cli component Card
```

按钮文案用 `label`，不要把可见文字只放在 `children` 里却漏掉无障碍名（以当前组件文档为准）。

## 3. Token 而不是魔法色

自定义样式时用 CSS 自定义属性：

```tsx
import * as stylex from '@stylexjs/stylex';
import {Card} from '@astryxdesign/core/Card';

const styles = stylex.create({
  surface: {
    padding: 'var(--spacing-4)',
    borderRadius: 'var(--radius-container)',
    backgroundColor: 'var(--color-background-surface)',
  },
});

<Card xstyle={styles.surface} />
```

也可用 `className` 接 Tailwind 或普通 CSS：

```tsx
<Card className="max-w-md">
  <Heading level={2}>Dashboard</Heading>
</Card>
```

| 做法 | 适用 |
|---|---|
| Token / 主题 | 品牌色、间距、圆角，跨明暗色 |
| `className` | 布局宽度、一次性偏移，可与 Tailwind 并存 |
| `xstyle` | StyleX 原子样式，精确覆盖组件 |

颜色写成 `[light, dark]` 元组的工作在主题层完成，组件只消费变量。

## 4. 明暗色

```tsx
<Theme theme={themes[defaultThemeSlug]} mode="system">
  <App />
</Theme>
```

| `mode` | 行为 |
|---|---|
| `system` | 跟随操作系统 |
| `light` / `dark` | 锁定 |

产品内切换：用状态在 light/dark 间改 `mode`，或换一套 `defineTheme` 结果。根节点会出现主题相关 data 属性，切换应是一次属性更新，而不是重挂整棵树。

## 5. 覆盖样式的边界

- 先问：这是品牌问题还是这一处布局问题？品牌走主题，一处走 `className` / `xstyle`。
- 输入类组件不要用根级 `xstyle`（阶段二细讲 `controlXstyle`）。
- 不要为了改一个圆角去 fork `@astryxdesign/core`。

```tsx
import * as stylex from '@stylexjs/stylex';
import {Button} from '@astryxdesign/core/Button';

const overrides = stylex.create({
  save: {alignSelf: 'flex-end'},
});

<Button label="Save" xstyle={overrides.save} />
```

## 阶段验收

- 能用 VStack / HStack / Heading / Button / Card 搭出带页头的骨架。
- 能说明 Token 与硬编码颜色的差别。
- 能切换 `mode` 并看到表面色、按钮色变化。
- 能用 `className` 或 `xstyle` 做一次局部覆盖且不破坏主题。

## 动手任务

1. 做页头：标题 + 主按钮 + 次按钮。
2. 放三张指标 Card（订阅数、收入、即将到期）。
3. 把一处 padding 改成 `var(--spacing-4)`。
4. 分别预览 light / dark / system。
5. 用 CLI 查 `Heading` 与 `Card` 的 props，核对你写的用法。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 间距看起来像原生块元素 | 是否用了 VStack/HStack 的 `gap` |
| 暗色下仍是浅色表面 | `<Theme>` 的 `mode`、主题 CSS 是否导入 |
| Tailwind 把 Astryx 样式盖掉 | cascade layer 顺序（阶段四） |
| `xstyle` 类型报错 | `@stylexjs/stylex` 是否安装，是否 `stylex.create` |

## 进入下一阶段的条件

骨架页在两种颜色模式下都完整，你能指认页面上哪些值来自 Token。此时进入 [阶段二：表单、浮层与交互](./stage-2-interactions-and-forms.md)。
