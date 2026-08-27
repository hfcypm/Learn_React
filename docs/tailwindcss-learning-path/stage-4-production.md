# 阶段四：Tailwind CSS 工程化与生产交付

**目标**：掌握构建体积控制、无障碍、可维护性和生产交付，能够交付高性能、一致、可维护的界面。

## 1. 构建体积控制

### 1.1 按需生成

Tailwind 只为扫描到的类生成 CSS。控制体积的第一原则是确保扫描范围正确、不引入无用类：

```css
@import "tailwindcss";

/* 只扫描这些目录，避免扫描整个 node_modules */
@source "../components";
@source "../app";
```

### 1.2 检查产物体积

```bash
# 生产构建
npm run build

# 查看输出 CSS 文件大小
ls -lh dist/*.css
```

使用工具分析 CSS 体积：

```bash
npx tailwindcss --help
```

### 1.3 减少体积的实践

- 用语义化类名代替重复工具类组合，避免多次生成相同规则。
- 不手动导入整个主题，只定义需要的变量。
- 生产环境确认没有开启开发工具类或调试模式。
- 避免用任意值创建大量一次性类，能复用就复用。

### 1.4 动态类名处理

动态拼接类名会生成失败，需要维护一个完整类名列表：

```ts
// 错误：运行时拼接，Tailwind 扫描不到
const classes = `text-${size}`;

// 正确：完整类名写出来，供扫描
const sizeClasses = {
  sm: 'text-sm',
  md: 'text-base',
  lg: 'text-lg',
};
```

## 2. 无障碍

### 2.1 语义与焦点

工具类不能替代语义化 HTML。按钮用 `<button>`，导航用 `<nav>`，输入框用 `<label>` 关联：

```html
<label class="block text-sm font-medium text-gray-700" for="email">
  邮箱
</label>
<input
  id="email"
  name="email"
  type="email"
  class="mt-1 block w-full rounded-md border border-gray-300 px-3 py-2"
/>
```

### 2.2 focus-visible

用 `focus-visible:` 只为键盘焦点显示轮廓，避免鼠标点击时也出现轮廓：

```html
<button
  class="rounded-md px-4 py-2 bg-blue-600 text-white
         focus-visible:outline-2 focus-visible:outline-offset-2
         focus-visible:outline-blue-600"
>
  提交
</button>
```

### 2.3 颜色对比度

使用足够深的文字色和浅色背景组合，检查对比度：

```html
<!-- 对比度不足 -->
<p class="text-gray-300">次要文字</p>

<!-- 对比度足够 -->
<p class="text-gray-600">次要文字</p>
```

正文至少用 `text-gray-700` 或更深的颜色，配合 `bg-white` 保证可读性。

### 2.4 prefers-reduced-motion

尊重用户的减少动画偏好：

```css
@import "tailwindcss";

@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

## 3. 测试与验证

### 3.1 视觉回归

用 Playwright 或 Chromatic 做视觉回归测试，防止工具类改动破坏界面：

```ts
// e2e/visual.spec.ts
import { test, expect } from '@playwright/test';

test('首页截图', async ({ page }) => {
  await page.goto('/');
  await expect(page).toHaveScreenshot('home.png');
});
```

### 3.2 组件测试

用 Testing Library 验证组件在不同变体和状态下正确渲染：

```tsx
// components/Button.test.tsx
import { render, screen } from '@testing-library/react';
import { Button } from './Button';

describe('Button', () => {
  it('渲染 primary 变体', () => {
    render(<Button variant="primary">保存</Button>);
    const button = screen.getByRole('button', { name: '保存' });
    expect(button.className).toContain('bg-blue-600');
  });
});
```

### 3.3 手动检查清单

- 在浏览器调整窗口宽度，检查断点行为。
- 切换系统暗色模式，检查 dark 变体。
- 用键盘 Tab 遍历，检查 focus 轮廓。
- 打开浏览器对比度检查，验证可读性。
- 检查构建产物 CSS 大小。

## 4. 可维护性

### 4.1 样式组织

```text
styles/
├── globals.css        # @import tailwindcss
├── theme.css          # @theme 设计变量
└── components.css     # @apply 组件类
```

### 4.2 组件边界

- 组件通过 props 控制变体和状态。
- 公共样式抽取为组件，避免在多个页面复制长类名。
- 用 `cn` 工具合并条件类名：

```ts
// lib/cn.ts
export function cn(...classes: (string | false | undefined)[]) {
  return classes.filter(Boolean).join(' ');
}
```

```tsx
<Button className={cn(isActive && 'bg-brand-600 text-white')}>
  {label}
</Button>
```

### 4.3 审查与自动化

- 用 prettier 插件自动排序类名。
- 用 ESLint 规则约束类名使用。
- 在代码评审中检查：变体是否集中、类名是否重复、主题变量是否被魔法值替代。

## 5. 部署与 CI

### 5.1 CI 检查

```text
npm ci
  -> lint
  -> typecheck
  -> test
  -> build（确认 CSS 体积）
```

### 5.2 部署

Tailwind 构建产物是纯 CSS，可部署到任意静态托管或随框架部署。生产构建默认启用优化：

```bash
npm run build
```

检查部署后的 CSS 是否包含源映射和开发注释，生产应剔除。

## 6. 常见排错

| 现象 | 排查方向 |
|---|---|
| 生产 CSS 巨大 | 检查扫描范围、重复类、任意值数量 |
| 生产与开发样式不一致 | 对比开发/生产构建的扫描配置 |
| 动态类不生效 | 改为完整类名列表 |
| 对比度不足 | 加深文字色或调亮背景 |
| 键盘看不到焦点 | 使用 `focus-visible:` 显示轮廓 |
| 动画被误触发 | 配置 `prefers-reduced-motion` |

## 阶段四验收

- 能控制生产 CSS 体积并解释构成。
- 能保证基本无障碍（焦点、对比度、语义）。
- 能组织可维护的样式与组件体系。
- 能配置视觉回归和组件测试。
- 能完成生产构建与部署。

## 动手任务

1. 记录项目生产 CSS 大小并尝试减少 20%。
2. 为所有可交互元素添加 focus-visible 样式。
3. 检查并修复一个对比度不足的文本。
4. 配置 prettier 插件和视觉回归测试。
5. 编写一份样式维护与审查检查清单。

## 进入下一阶段的条件

你能够交付体积可控、可访问、可维护的生产界面。此时进入 [综合实战：后台管理界面](./project-practice.md)。
