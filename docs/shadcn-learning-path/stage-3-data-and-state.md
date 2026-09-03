# 阶段三：数据展示与状态

**目标**：掌握数据表、图表与受控组件，把后台页面的核心数据展示能力落地。

## 1. Table 与 DataTable

shadcn 的 Table 是展示层，配合 TanStack Table 获得排序、筛选与分页。先添加基础组件：

```bash
npx shadcn@latest add table
npm install @tanstack/react-table
```

定义列：

```tsx
"use client";

import { ColumnDef } from "@tanstack/react-table";

type Subscription = {
  id: string;
  customer: string;
  plan: string;
  status: "active" | "expired";
};

export const columns: ColumnDef<Subscription>[] = [
  { accessorKey: "customer", header: "客户" },
  { accessorKey: "plan", header: "套餐" },
  { accessorKey: "status", header: "状态" },
];
```

表格主体：

```tsx
import {
  useReactTable,
  getCoreRowModel,
  getSortedRowModel,
  flexRender,
} from "@tanstack/react-table";

export function SubscriptionsTable({ data }: { data: Subscription[] }) {
  const table = useReactTable({
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
    getSortedRowModel: getSortedRowModel(),
  });

  return (
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
  );
}
```

- 列定义与表格逻辑分离，cell 可渲染任意 React 节点（按钮、徽章）。
- 排序、筛选由 TanStack Table 管理，UI 状态由它驱动。

## 2. 列内渲染复杂内容

```tsx
export const columns: ColumnDef<Subscription>[] = [
  { accessorKey: "customer", header: "客户" },
  { accessorKey: "plan", header: "套餐" },
  {
    accessorKey: "status",
    header: "状态",
    cell: ({ row }) => {
      const active = row.original.status === "active";
      return <Badge variant={active ? "default" : "destructive"}>
        {active ? "生效中" : "已过期"}
      </Badge>;
    },
  },
];
```

## 3. 分页与筛选

```tsx
const table = useReactTable({
  data,
  columns,
  getCoreRowModel: getCoreRowModel(),
  getPaginationRowModel: getPaginationRowModel(),
  getFilteredRowModel: getFilteredRowModel(),
  state: { sorting, columnFilters },
});
```

翻页控件用 Button 组合：上一页 / 页码 / 下一页。列筛选用 Input 绑定 `column.setFilterValue`。

## 4. 图表

shadcn 提供 Chart 组件（基于 Recharts）：

```bash
npx shadcn@latest add chart
npm install recharts
```

```tsx
"use client";

import {
  ChartContainer,
  ChartTooltip,
  ChartTooltipContent,
} from "@/components/ui/chart";
import { Bar, BarChart, CartesianGrid, XAxis } from "recharts";

const chartData = [
  { month: "一月", revenue: 1200 },
  { month: "二月", revenue: 1800 },
  { month: "三月", revenue: 1400 },
];

export function RevenueChart() {
  return (
    <ChartContainer config={{}} className="h-64 w-full">
      <BarChart data={chartData}>
        <CartesianGrid vertical={false} />
        <XAxis dataKey="month" />
        <ChartTooltip content={<ChartTooltipContent />} />
        <Bar dataKey="revenue" fill="var(--color-primary)" radius={4} />
      </BarChart>
    </ChartContainer>
  );
}
```

图表颜色引用 `--chart-1..5` 变量，随主题变化。

## 5. 受控与状态管理

- shadcn 组件多数支持受控 props（value/onValueChange）。
- 页面级状态用 React 或 TanStack Query；跨页面共享用 Zustand（可参考 [AI 应用前端学习路线](../ai-learning-path/README.md) 的状态章节）。
- 表格状态（排序、筛选、分页）由 TanStack Table 内部管理或受控提升。

## 6. 空态与加载态

```tsx
{rows.length === 0 && (
  <TableRow>
    <TableCell colSpan={columns.length} className="h-24 text-center">
      暂无数据
    </TableCell>
  </TableRow>
)}
```

加载中用 Skeleton：

```tsx
import { Skeleton } from "@/components/ui/skeleton";

<div className="space-y-2">
  <Skeleton className="h-4 w-48" />
  <Skeleton className="h-4 w-64" />
</div>;
```

## 7. 动手任务

1. 添加 table 组件，用 TanStack Table 实现订阅列表。
2. 在列内用 Badge 渲染状态。
3. 接入排序与分页。
4. 添加 chart 组件，实现收入柱状图。
5. 添加空态与 Skeleton 加载态。

## 阶段三验收

- 能用 TanStack Table 实现排序、筛选、分页。
- 能在列内渲染徽章、按钮等复杂内容。
- 能用 Chart 组件实现图表。
- 能处理空态与加载态。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 表头排序不触发 | 未启用 getSortedRowModel 或未绑定点击 |
| 图表不显示 | 确认 recharts 安装与数据格式 |
| 颜色不变主题 | 图表引用 chart 变量而非硬编码色 |
| 分页无按钮 | 需自建翻页控件绑定分页状态 |
| 空态不显示 | 判断 rowModel 长度而非 data |
| 客户端报错 | 确认表格与图表组件标了 "use client" |

## 进入下一阶段的条件

你能够实现数据驱动的后台页面。此时进入 [阶段四：工程化与生产交付](./stage-4-production.md)。
