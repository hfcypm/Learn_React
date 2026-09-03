# shadcn/ui 综合实战：订阅管理后台

## 1. 项目目标

用 shadcn/ui 构建一个订阅管理后台，覆盖主题定制、表单、数据表、图表与工程化。项目按阶段扩展，最终交付一个界面专业、可访问、可维护的后台应用。

```text
初始化 -> 主题与导航 -> 表单交互 -> 数据表格
    -> 图表与统计 -> 生产交付
```

## 2. 需求

- 侧边栏导航：总览、订阅、客户、设置。
- 订阅列表：分页、排序、状态徽章、行操作。
- 新建/编辑订阅：对话框 + 带校验表单。
- 收入统计：按月收入柱状图与卡片指标。
- 暗色模式切换与主题定制。
- 可访问性与基础测试。

## 3. 技术选择

| 技术 | 用途 |
|---|---|
| React + TypeScript | 界面 |
| Tailwind CSS v4 | 样式 |
| shadcn/ui | 组件 |
| Radix UI | 交互与无障碍（由组件携带） |
| react-hook-form + zod | 表单校验 |
| TanStack Table | 表格排序分页 |
| Recharts | 图表 |

## 4. 项目结构

```text
subscription-admin/
├── src/
│   ├── components/
│   │   ├── ui/            # shadcn 组件（CLI 生成）
│   │   ├── app-sidebar.tsx
│   │   ├── subscription-form.tsx
│   │   └── revenue-chart.tsx
│   ├── lib/utils.ts
│   ├── styles/globals.css # 主题变量
│   ├── types.ts
│   └── App.tsx
├── components.json
└── package.json
```

## 5. 初始化

```bash
npm create vite@latest subscription-admin -- --template react-ts
cd subscription-admin
npm install tailwindcss @tailwindcss/vite

# 配置 vite.config.ts 加入 tailwindcss 插件后
npx shadcn@latest init
npx shadcn@latest add sidebar button card input label badge dialog select dropdown-menu form table chart skeleton
```

## 6. 主题定制

```css
/* src/styles/globals.css（节选） */
:root {
  --background: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);
  --primary: oklch(0.55 0.2 259);      /* 品牌蓝 */
  --primary-foreground: oklch(0.985 0 0);
  --chart-1: oklch(0.646 0.222 41.116);
  --radius: 0.625rem;
}
.dark {
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
  --primary: oklch(0.62 0.2 259);
  --primary-foreground: oklch(0.145 0 0);
}
```

## 7. 侧边栏导航

```tsx
// src/components/app-sidebar.tsx
import {
  Sidebar,
  SidebarContent,
  SidebarGroup,
  SidebarMenu,
  SidebarMenuItem,
  SidebarMenuButton,
  SidebarProvider,
  SidebarInset,
} from "@/components/ui/sidebar";

export function AppLayout({ children }: { children: React.ReactNode }) {
  return (
    <SidebarProvider>
      <Sidebar>
        <SidebarContent>
          <SidebarGroup>
            <SidebarMenu>
              {[
                ["总览", "/"],
                ["订阅", "/subscriptions"],
                ["客户", "/customers"],
                ["设置", "/settings"],
              ].map(([label]) => (
                <SidebarMenuItem key={label}>
                  <SidebarMenuButton>{label}</SidebarMenuButton>
                </SidebarMenuItem>
              ))}
            </SidebarMenu>
          </SidebarGroup>
        </SidebarContent>
      </Sidebar>
      <SidebarInset>{children}</SidebarInset>
    </SidebarProvider>
  );
}
```

## 8. 数据模型与类型

```ts
// src/types.ts
export type Subscription = {
  id: string;
  customer: string;
  plan: "basic" | "pro" | "enterprise";
  status: "active" | "expired" | "trial";
  monthlyAmount: number;
};

export const subscriptions: Subscription[] = [
  { id: "S-1001", customer: "甲公司", plan: "pro", status: "active", monthlyAmount: 299 },
  { id: "S-1002", customer: "乙公司", plan: "basic", status: "trial", monthlyAmount: 0 },
  // ...
];
```

## 9. 新建订阅表单

```tsx
// src/components/subscription-form.tsx
"use client";

import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";
import {
  Form, FormField, FormItem, FormLabel, FormControl, FormMessage,
} from "@/components/ui/form";
import { Input } from "@/components/ui/input";
import {
  Select, SelectTrigger, SelectValue, SelectContent, SelectItem,
} from "@/components/ui/select";
import { Button } from "@/components/ui/button";

const schema = z.object({
  customer: z.string().min(1, "客户名必填"),
  plan: z.enum(["basic", "pro", "enterprise"], { message: "请选择套餐" }),
  monthlyAmount: z.coerce.number().min(0, "金额不能为负"),
});

type Schema = z.infer<typeof schema>;

export function SubscriptionForm({ onSubmit }: { onSubmit: (v: Schema) => void }) {
  const form = useForm<Schema>({ resolver: zodResolver(schema) });

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
        <FormField control={form.control} name="customer" render={({ field }) => (
          <FormItem>
            <FormLabel>客户</FormLabel>
            <FormControl><Input placeholder="客户名称" {...field} /></FormControl>
            <FormMessage />
          </FormItem>
        )} />
        <FormField control={form.control} name="plan" render={({ field }) => (
          <FormItem>
            <FormLabel>套餐</FormLabel>
            <FormControl>
              <Select onValueChange={field.onChange} value={field.value}>
                <SelectTrigger><SelectValue placeholder="选择套餐" /></SelectTrigger>
                <SelectContent>
                  <SelectItem value="basic">基础版</SelectItem>
                  <SelectItem value="pro">专业版</SelectItem>
                  <SelectItem value="enterprise">企业版</SelectItem>
                </SelectContent>
              </Select>
            </FormControl>
            <FormMessage />
          </FormItem>
        )} />
        <Button type="submit">保存</Button>
      </form>
    </Form>
  );
}
```

## 10. 订阅数据表

```tsx
// src/components/subscriptions-table.tsx
import { ColumnDef, useReactTable, getCoreRowModel, getSortedRowModel, getPaginationRowModel, flexRender } from "@tanstack/react-table";
import { Badge } from "@/components/ui/badge";
import { Button } from "@/components/ui/button";
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from "@/components/ui/table";

const statusMap = { active: "生效中", trial: "试用", expired: "已过期" } as const;

const columns: ColumnDef<Subscription>[] = [
  { accessorKey: "id", header: "编号" },
  { accessorKey: "customer", header: "客户" },
  { accessorKey: "plan", header: "套餐" },
  {
    accessorKey: "status",
    header: "状态",
    cell: ({ row }) => (
      <Badge variant={row.original.status === "expired" ? "destructive" : "default"}>
        {statusMap[row.original.status]}
      </Badge>
    ),
  },
];

export function SubscriptionsTable({ data }: { data: Subscription[] }) {
  const table = useReactTable({
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
    getSortedRowModel: getSortedRowModel(),
    getPaginationRowModel: getPaginationRowModel(),
  });

  return (
    <div>
      <div className="overflow-hidden rounded-md border">
        <Table>
          <TableHeader>
            {table.getHeaderGroups().map((group) => (
              <TableRow key={group.id}>
                {group.headers.map((header) => (
                  <TableHead key={header.id}>
                    {header.isPlaceholder ? null : (
                      <button onClick={header.column.getToggleSortingHandler()}>
                        {flexRender(header.column.columnDef.header, header.getContext())}
                      </button>
                    )}
                  </TableHead>
                ))}
              </TableRow>
            ))}
          </TableHeader>
          <TableBody>
            {table.getRowModel().rows.map((row) => (
              <TableRow key={row.id}>
                {row.getVisibleCells().map((cell) => (
                  <TableCell key={cell.id}>
                    {flexRender(cell.column.columnDef.cell, cell.getContext())}
                  </TableCell>
                ))}
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </div>
      <div className="mt-4 flex items-center gap-2">
        <Button size="sm" onClick={() => table.previousPage()} disabled={!table.getCanPreviousPage()}>上一页</Button>
        <Button size="sm" onClick={() => table.nextPage()} disabled={!table.getCanNextPage()}>下一页</Button>
      </div>
    </div>
  );
}
```

## 11. 新建对话框与页面组装

```tsx
// src/App.tsx
import { useState } from "react";
import { Dialog, DialogTrigger, DialogContent, DialogHeader, DialogTitle } from "@/components/ui/dialog";
import { Button } from "@/components/ui/button";
import { SubscriptionForm } from "./components/subscription-form";
import { SubscriptionsTable } from "./components/subscriptions-table";
import { RevenueChart } from "./components/revenue-chart";
import { subscriptions } from "./types";

export function App() {
  const [rows, setRows] = useState(subscriptions);

  return (
    <div className="p-6 space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-xl font-bold">订阅管理</h1>
        <Dialog>
          <DialogTrigger asChild>
            <Button>新建订阅</Button>
          </DialogTrigger>
          <DialogContent>
            <DialogHeader><DialogTitle>新建订阅</DialogTitle></DialogHeader>
            <SubscriptionForm
              onSubmit={(v) =>
                setRows((prev) => [
                  { ...v, id: `S-${Date.now()}`, status: "trial", monthlyAmount: v.monthlyAmount } as never,
                  ...prev,
                ])
              }
            />
          </DialogContent>
        </Dialog>
      </div>

      <RevenueChart />
      <SubscriptionsTable data={rows} />
    </div>
  );
}
```

## 12. 实施顺序

1. 初始化项目与 shadcn/ui。
2. 定制主题并接入暗色切换。
3. 搭建 Sidebar 布局。
4. 实现订阅表单与对话框。
5. 实现数据表排序分页。
6. 实现收入图表与统计卡片。
7. 补测试与可访问性复查。

## 13. 验收清单

- [ ] 主题变量统一，暗色可切换。
- [ ] 新建订阅对话框与校验表单可用。
- [ ] 表格排序与分页正常。
- [ ] 图表引用主题色。
- [ ] 空态与加载态存在。
- [ ] 键盘可操作对话框与下拉。
- [ ] 组件变体可扩展，无散落样式。
- [ ] 关键表单有测试覆盖。

## 14. 按阶段学习卡片

| 阶段 | 项目增量 |
|---|---|
| 零 | 初始化、添加组件 |
| 一 | 主题定制、导航骨架 |
| 二 | 表单与对话框 |
| 三 | 数据表、图表、状态 |
| 四 | 变体扩展、测试、工程化 |
