# 阶段四：工程化与生产交付

**目标**：掌握组件变体管理、可访问性、测试与工程化集成，交付可维护、可访问、可测试的界面。

## 1. 深入组件源码

添加组件后源码就在项目里。以 Button 为例，核心是变体表：

```tsx
// components/ui/button.tsx（节选）
import { cva, type VariantProps } from "class-variance-authority";

const buttonVariants = cva(
  "inline-flex items-center justify-center gap-2 whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-2 focus-visible:outline-ring/50 disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        primary: "bg-primary text-primary-foreground hover:bg-primary/90",
        secondary: "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        outline: "border border-input bg-background hover:bg-accent",
        ghost: "hover:bg-accent hover:text-accent-foreground",
        destructive: "bg-destructive text-white hover:bg-destructive/90",
      },
      size: {
        default: "h-10 px-4",
        sm: "h-8 rounded-md px-3",
        lg: "h-11 rounded-md px-8",
      },
    },
    defaultVariants: { variant: "primary", size: "default" },
  },
);
```

- 用 cva 管理变体，样式集中、类型安全。
- 颜色全部引用语义变量。

## 2. 定制与扩展

组件源码在你的项目里，需要新增变体直接改 cva：

```tsx
// 给 Button 增加 success 变体
variant: {
  // ...现有变体
  success: "bg-emerald-600 text-white hover:bg-emerald-700",
}
```

不写包装层，直接改源码是 shadcn/ui 的设计意图。

## 3. 可访问性基线

- 交互组件来自 Radix UI，自带键盘导航、焦点管理与 ARIA。
- 图标按钮必须加 `aria-label`。
- 对话框用 `DialogTitle`/`DialogDescription` 提供可访问名称与描述。
- 用 `asChild` 保持语义元素（按钮仍是按钮）。
- 保持对比度与焦点可见，遵循 [Tailwind CSS 学习路线](../tailwindcss-learning-path/README.md) 的无障碍章节。

## 4. 布局与排版

后台界面常用 Sidebar 组件：

```bash
npx shadcn@latest add sidebar
```

```tsx
import {
  Sidebar,
  SidebarContent,
  SidebarGroup,
  SidebarMenu,
  SidebarMenuItem,
  SidebarMenuButton,
} from "@/components/ui/sidebar";

<Sidebar>
  <SidebarContent>
    <SidebarGroup>
      <SidebarMenu>
        <SidebarMenuItem>
          <SidebarMenuButton>总览</SidebarMenuButton>
        </SidebarMenuItem>
      </SidebarMenu>
    </SidebarGroup>
  </SidebarContent>
</Sidebar>;
```

## 5. 测试

组件是 React 组件，用 Testing Library 做交互测试：

```tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { SubscribeForm } from "./subscribe-form";

test("邮箱为空时提示校验错误", async () => {
  render(<SubscribeForm />);
  await userEvent.click(screen.getByRole("button", { name: "提交" }));
  expect(await screen.findByText("邮箱格式不正确")).toBeInTheDocument();
});
```

- 交互组件走真实用户路径：点击、输入、断言错误。
- Radix 组件在 jsdom 中需配置 matchMedia/ResizeObserver 等 polyfill。

## 6. 工程化注意

- Server Component 项目中，交互组件需标 `"use client"`，纯展示可留在服务端。
- 按需 add 组件，避免无关组件进入包体。
- 用 components.json 统一别名，团队一致。
- 依赖（Radix、lucide、react-hook-form、TanStack）按组件安装，无全局组件库包袱。

## 7. 动手任务

1. 阅读 button 与 badge 源码，解释 cva 变体结构。
2. 为 Button 新增一个自定义 variant 并全局使用。
3. 用 sidebar 组件搭一个后台导航骨架。
4. 为订阅表单写一条校验失败的测试。
5. 用键盘走一遍对话框，验证可访问性。

## 阶段四验收

- 能读懂并修改 cva 变体。
- 能维护可访问性基线。
- 能用 sidebar 搭建布局。
- 能为组件写交互测试。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 新增变体不生效 | 确认修改的组件被实际使用且无缓存 |
| 测试找不到元素 | 用 role 查询而非文本 |
| Radix 在测试中报错 | 补充 matchMedia 等 jsdom polyfill |
| 服务端渲染报错 | 交互组件标 "use client" |
| 图标不显示 | 确认 lucide-react 依赖与图标名 |

## 进入下一阶段的条件

你能够工程化地使用与扩展组件。此时进入 [综合实战：订阅管理后台](./project-practice.md) 串起全流程。
