# 阶段二：交互组件与表单

**目标**：掌握对话框、下拉、弹出层等交互组件，以及带校验的表单实现。

## 1. Dialog

```tsx
import {
  Dialog,
  DialogTrigger,
  DialogContent,
  DialogHeader,
  DialogTitle,
  DialogDescription,
  DialogFooter,
  DialogClose,
} from "@/components/ui/dialog";
import { Button } from "@/components/ui/button";

<Dialog>
  <DialogTrigger asChild>
    <Button>新建订阅</Button>
  </DialogTrigger>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>新建订阅</DialogTitle>
      <DialogDescription>填写客户与套餐信息。</DialogDescription>
    </DialogHeader>
    <DialogFooter>
      <DialogClose asChild>
        <Button variant="ghost">取消</Button>
      </DialogClose>
      <Button>保存</Button>
    </DialogFooter>
  </DialogContent>
</Dialog>
```

Dialog 由 Radix 提供焦点陷阱、Esc 关闭与无障碍标注，`asChild` 让触发器复用现有按钮。

## 2. DropdownMenu、Popover、Select

```tsx
import {
  DropdownMenu,
  DropdownMenuTrigger,
  DropdownMenuContent,
  DropdownMenuItem,
} from "@/components/ui/dropdown-menu";

<DropdownMenu>
  <DropdownMenuTrigger asChild>
    <Button variant="ghost">更多</Button>
  </DropdownMenuTrigger>
  <DropdownMenuContent>
    <DropdownMenuItem>编辑</DropdownMenuItem>
    <DropdownMenuItem variant="destructive">删除</DropdownMenuItem>
  </DropdownMenuContent>
</DropdownMenu>
```

```tsx
import {
  Select,
  SelectTrigger,
  SelectValue,
  SelectContent,
  SelectItem,
} from "@/components/ui/select";

<Select onValueChange={(v) => console.log(v)}>
  <SelectTrigger>
    <SelectValue placeholder="选择套餐" />
  </SelectTrigger>
  <SelectContent>
    <SelectItem value="basic">基础版</SelectItem>
    <SelectItem value="pro">专业版</SelectItem>
  </SelectContent>
</Select>
```

- DropdownMenu 与 Select 用于选择类交互。
- Popover 用于浮层内容（如日期选择、自定义筛选）。

## 3. Form 组件

shadcn 的 Form 基于 react-hook-form + zod：

```bash
npx shadcn@latest add form
npm install react-hook-form zod @hookform/resolvers
```

```tsx
"use client";

import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";
import {
  Form,
  FormField,
  FormItem,
  FormLabel,
  FormControl,
  FormMessage,
} from "@/components/ui/form";
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";

const schema = z.object({
  email: z.string().email("邮箱格式不正确"),
  plan: z.string().min(1, "请选择套餐"),
});

type Schema = z.infer<typeof schema>;

export function SubscribeForm() {
  const form = useForm<Schema>({
    resolver: zodResolver(schema),
    defaultValues: { email: "", plan: "" },
  });

  function onSubmit(values: Schema) {
    console.log(values);
  }

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
        <FormField
          control={form.control}
          name="email"
          render={({ field }) => (
            <FormItem>
              <FormLabel>邮箱</FormLabel>
              <FormControl>
                <Input placeholder="you@example.com" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <Button type="submit">提交</Button>
      </form>
    </Form>
  );
}
```

- Schema 是校验的唯一事实来源。
- `FormField` 把字段状态与错误注入 UI。
- `FormMessage` 显示校验错误。

## 4. 组合：对话框内的表单

把表单放进 Dialog 常用于新建/编辑弹窗：

```tsx
<Dialog>
  <DialogTrigger asChild>
    <Button>新建订阅</Button>
  </DialogTrigger>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>新建订阅</DialogTitle>
    </DialogHeader>
    <SubscribeForm />
  </DialogContent>
</Dialog>
```

## 5. 动手任务

1. 添加 dialog、dropdown-menu、select、form 组件。
2. 实现一个新建对话框，含取消与保存。
3. 用 dropdown-menu 实现行操作菜单。
4. 定义 zod Schema，实现带校验的新建订阅表单。
5. 让表单在提交成功后关闭对话框。

## 阶段二验收

- 能使用 Dialog、DropdownMenu、Select。
- 能用 react-hook-form + zod 实现校验表单。
- 能组合对话框与表单。
- 能解释 Radix 提供的可访问性能力。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 校验不触发 | 表单 onSubmit 未用 handleSubmit |
| 错误不显示 | FormMessage 未放在 FormField 内 |
| Select 值取不到 | 用 onValueChange 而非 onChange |
| asChild 报错 | 子元素需支持 ref 转发 |
| 对话框不能关 | DialogClose 未包裹 Button |
| Schema 类型漂移 | 用 z.infer 派生类型 |

## 进入下一阶段的条件

你能够实现带校验的表单。此时进入 [阶段三：数据展示与状态](./stage-3-data-and-state.md)。
