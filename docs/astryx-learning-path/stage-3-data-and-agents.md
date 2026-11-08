# 阶段三：数据展示与 AI 工作流

**目标**：用 Table 展示订阅列表，用官方模板加速页面，用 CLI / MCP 让人和编码助手读同一份组件契约。

> 适用范围：Astryx Beta + React 19+ + TypeScript
> 需要的工具：Node.js、npm、已完成阶段二的工程
> 验证命令：`npx @astryxdesign/cli component Table`、`npx @astryxdesign/cli template --list`
> 最后复核日期：2026-11-08

**前置知识**：阶段二的表单与对话框。

## 1. Table

```tsx
import {Card} from '@astryxdesign/core/Card';
import {Table} from '@astryxdesign/core/Table';

type Row = {
  id: string;
  customer: string;
  plan: string;
  status: string;
  mrr: string;
};

const columns = [
  {key: 'customer', header: '客户'},
  {key: 'plan', header: '套餐'},
  {key: 'status', header: '状态'},
  {key: 'mrr', header: '月费'},
];

export function SubscriptionTable({data}: {data: Row[]}) {
  return (
    <Card>
      <Table data={data} columns={columns} />
    </Card>
  );
}
```

列定义、排序、行操作、空态以当前文档为准：

```bash
npx @astryxdesign/cli component Table
```

要点：

- Table 放进 Card 时注意内边距与“bleed”规则，官方容器 padding 系统会让表格贴边。
- 状态用 Badge 或官方 Status 组件，不要只改文字颜色。
- 加载用骨架或 Table 自带 loading，空列表给一句行动建议（去新建）。
- 行内主操作进菜单，破坏性操作二次确认（阶段二的 Dialog）。

## 2. 模板

Astryx 把设计师打磨过的页面当成“给 AI 和人复制的高质量样本”。

```bash
npx @astryxdesign/cli template --list
npx @astryxdesign/cli template --list --type page
npx @astryxdesign/cli template --list --type block
npx @astryxdesign/cli template <name>
npx @astryxdesign/cli template <name> ./src/pages/subscriptions
npx @astryxdesign/cli template <name> --skeleton
```

- `--skeleton` 只要结构注释，适合先对齐布局再填业务。
- 模板里的导入路径、主题用法应与当前 Core 版本一致；复制后跑一遍 TypeScript。
- 仪表盘类页面优先从模板改，而不是从空白 `div` 堆。

## 3. CLI：人和 AI 的同一份说明书

```bash
npx @astryxdesign/cli docs
npx @astryxdesign/cli docs tokens
npx @astryxdesign/cli docs theme
npx @astryxdesign/cli search table
npx @astryxdesign/cli component Button --dense
npx @astryxdesign/cli docs principles --dense
```

| 命令 | 用途 |
|---|---|
| `component --list` | 按分类列出组件 |
| `component <Name>` | props、变体、示例 |
| `docs <topic>` | 主题、Token、迁移、原则 |
| `search <词>` | 按关键词找组件与文档 |
| `template` | 页面 / 区块源码 |
| `theme list` / `theme add` | 主题包 |

给助手准备固定 script，避免路径猜错：

```json
"scripts": {
  "astryx": "node node_modules/@astryxdesign/cli/clients/cli/bin/astryx.mjs"
}
```

## 4. MCP

Astryx CLI 可作 MCP Server，让 Cursor / Claude Code 等工具按需拉组件契约，而不是靠模型记忆编 `variant="ghost-primary"`。

流程：

1. 确认工具支持 MCP。
2. 按 `npx @astryxdesign/cli docs working-with-ai` 接入。
3. 生成页面前先让助手 `component Table` / `component TextInput`。
4. 生成后核对：组件名是否存在、props 是否在契约里、输入类样式是否走 `control*`。

`astryx init` 写入的 `AGENTS.md` 就是给助手的项目级约定，提交进仓库。

## 5. 列表页信息架构

后台列表通常四块：

```text
页头（标题 + 新建）
  -> 筛选 / 搜索
  -> Table
  -> 空态 / 错误 / 分页
```

筛选变更后重置页码。错误态提供重试，不要只留白 Card。

## 阶段验收

- 能用 Table 展示多行订阅，含空态。
- 能从官方模板拷贝一页并改成自己的路由 / 数据。
- 能用 CLI 查出某个组件的 props，而不是只靠记忆。
- 能说明 MCP 解决的是“助手编造 API”问题。

## 动手任务

1. 用至少 5 行 mock 数据渲染订阅表。
2. 空数组时显示“还没有订阅”和新建按钮。
3. `template --list` 选一个接近后台的页面拷进项目。
4. 对 Table 跑 `component Table --dense`，把文档里你没用过的一个 prop 接上。
5. 在 `AGENTS.md` 里加一句：生成 UI 前必须查询 Astryx CLI。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| Table 类型对不上 | 列 `key` 与 row 字段、官方 columns 形状 |
| 模板导入失败 | Core 版本与模板是否同一发版 |
| 助手写出不存在的组件 | MCP 未连上，或没用项目内 CLI |
| `--dense` 输出不够 | 人读去掉 `--dense` 再查一节 `docs <topic> <section>` |

## 进入下一阶段的条件

列表页可展示、可空态、可用 CLI 自查 API。此时进入 [阶段四：定制、升级与生产](./stage-4-production.md)。
