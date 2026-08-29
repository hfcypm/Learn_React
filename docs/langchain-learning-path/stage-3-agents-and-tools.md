# 阶段三：Agent 与工具

**目标**：掌握工具定义、工具调用与多步推理 Agent，让模型能自主决定调用外部能力完成复杂任务。

## 1. 链与 Agent 的边界

- 链：执行流程固定，适合流程确定的场景。
- Agent：由模型决定下一步调用哪个工具、何时停止，适合流程不固定的多步任务。

决策标准：调用步骤是否预先确定。确定用链，不确定用 Agent。

## 2. 定义工具

用 `tool()` 把函数声明为工具，配 zod Schema 让模型知道如何调用：

```ts
import { tool } from "langchain";
import { z } from "zod";

const getWeather = tool(
  async ({ city }) => {
    // 真实场景中在此调用天气服务
    return `当前城市 ${city} 晴天，26 度。`;
  },
  {
    name: "get_weather",
    description: "查询指定城市的天气情况，参数为城市中文名。",
    schema: z.object({
      city: z.string().describe("城市名，如 上海"),
    }),
  },
);
```

- `description` 决定模型何时使用该工具，要写清楚触发条件。
- `schema` 决定参数抽取质量，字段用 `.describe()` 说明含义。
- 返回值是字符串或可字符串化的数据。

## 3. 创建 Agent

```ts
import { createAgent } from "langchain";

const agent = createAgent({
  model: "gpt-4o-mini",
  tools: [getWeather],
});

const response = await agent.invoke({
  messages: [{ role: "user", content: "上海今天天气怎么样？" }],
});

console.log(response.messages.at(-1)?.content);
```

模型会自主决定：调用 `get_weather("上海")`，拿到结果后再组织回答。

## 4. 观察工具调用过程

用 `streamEvents` 逐步查看模型每一步的决策：

```ts
const stream = await agent.streamEvents(
  {
    messages: [{ role: "user", content: "北京和广州今天都下雨吗？" }],
  },
  { version: "v3" },
);

for await (const snapshot of stream.values) {
  const latest = snapshot.messages.at(-1);
  if (latest?.tool_calls?.length) {
    const calls = latest.tool_calls.map((c) => c.name).join(", ");
    console.log("正在调用工具：", calls);
  } else if (latest?.content) {
    console.log("输出：", latest.content);
  }
}

const finalState = await stream.output;
console.log(finalState.messages.at(-1)?.content);
```

`streamEvents` v3 按消息快照推送，能实时观察工具调用与最终答案。

## 5. 多工具 Agent

```ts
const getOrderStatus = tool(
  async ({ orderId }) => {
    return `订单 ${orderId} 状态：已发货，预计明天送达。`;
  },
  {
    name: "get_order_status",
    description: "查询订单状态，需要用户提供订单号。",
    schema: z.object({
      orderId: z.string().describe("订单号，如 SO-2026-0001"),
    }),
  },
);

const agent = createAgent({
  model: "gpt-4o-mini",
  tools: [getWeather, getOrderStatus],
});
```

工具多时，模型的工具选择准确性与 `description` 质量直接相关。

## 6. 工具与 RAG 结合

检索器也可以包装成工具，让 Agent 决定是否需要查知识库：

```ts
const searchDocs = tool(
  async ({ query }) => {
    const docs = await retriever.invoke(query);
    return docs.map((d) => d.pageContent).join("\n");
  },
  {
    name: "search_knowledge_base",
    description: "在知识库中检索资料，回答产品与售后问题前调用。",
    schema: z.object({
      query: z.string().describe("检索关键词"),
    }),
  },
);
```

## 7. 工具设计原则

- 一个工具只做一件事，职责单一。
- Schema 精确，让参数抽取稳定。
- 返回结构化数据，便于下游处理。
- 工具内部做好错误处理，失败返回明确信息而不是抛异常。
- 不把敏感操作暴露为工具，或至少加确认流程。

## 8. 动手任务

1. 用 `tool()` 定义一个工具并创建 Agent，验证它能自主调用。
2. 用 `streamEvents` 观察工具调用全过程。
3. 增加第二个工具，测试模型在多个工具间选择。
4. 把阶段二的检索器包装成工具，构建带知识库的 Agent。
5. 故意让工具抛错，观察 Agent 如何恢复。

## 阶段三验收

- 能用 `tool()` 定义带 Schema 的工具。
- 能用 `createAgent` 创建可自主调用工具的 Agent。
- 能用 `streamEvents` 观察推理过程。
- 能解释工具 `description` 与 `schema` 对决策质量的影响。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 工具从未被调用 | `description` 不够明确，或模型未声明支持工具调用 |
| 参数抽取错误 | 收紧 Schema，字段加 `.describe()` |
| 循环调用停不下来 | 工具内逻辑异常或返回格式问题，检查输出约束 |
| 返回值无法解析 | 返回非字符串且不可序列化 |
| 工具抛异常导致中断 | 在工具内捕获并返回错误信息 |
| 与 RAG 混合时不检索 | 提示词需说明何时使用检索工具 |

## 进入下一阶段的条件

你能够构建自主调用工具的 Agent。此时进入 [阶段四：生产化交付](./stage-4-production.md)。
