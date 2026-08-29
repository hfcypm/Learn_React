# 阶段四：生产化交付

**目标**：为应用接入记忆、流式输出、回退与链路追踪，掌握与 Web 框架的集成和生产部署。

## 1. 记忆与会话

Agent 默认无状态，用 LangGraph 的 checkpointer 持久化会话：

```ts
import { createAgent } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const checkpointer = new MemorySaver();

const agent = createAgent({
  model: "gpt-4o-mini",
  tools: [],
  checkpointer,
});

const config = { configurable: { thread_id: "session-001" } };

await agent.invoke(
  { messages: [{ role: "user", content: "我叫小明。" }] },
  config,
);
const reply = await agent.invoke(
  { messages: [{ role: "user", content: "我叫什么？" }] },
  config,
);
console.log(reply.messages.at(-1)?.content);
// 小明
```

- `thread_id` 标识一个会话，同一会话共享历史。
- 生产环境用持久化 checkpointer（如 `PostgresSaver`）替代内存版。

## 2. 长对话压缩

历史过长会超 token 上限，用摘要中间件自动压缩：

```ts
import { createAgent, summarizationMiddleware } from "langchain";
import { MemorySaver } from "@langchain/langgraph";

const agent = createAgent({
  model: "gpt-4o-mini",
  tools: [],
  checkpointer: new MemorySaver(),
  middleware: [
    summarizationMiddleware({
      model: "gpt-4o-mini",
      trigger: { tokens: 4000 }, // 超过后触发压缩
      keep: { messages: 20 },    // 保留最近消息数
    }),
  ],
});
```

## 3. 流式输出

对话场景逐 token 输出，降低首字延迟：

```ts
const stream = await agent.streamEvents(
  { messages: [{ role: "user", content: "写一首四行诗。" }] },
  { version: "v3" },
);

for await (const message of stream.messages) {
  for await (const delta of message.text) {
    process.stdout.write(delta);
  }
}
```

在 Web 后端，把流式结果转发给前端，实现打字机效果（对接方式见 [Next.js 学习路线](../nextjs-learning-path/README.md) 与 [AI 应用前端学习路线](../ai-learning-path/README.md) 的流式章节）。

## 4. 超时与重试

```ts
const model = new ChatOpenAI({
  model: "gpt-4o-mini",
  maxRetries: 2,
  timeout: 30_000, // 毫秒
});
```

对网络不可靠场景，为整条链提供回退模型：

```ts
import { ChatOpenAI } from "@langchain/openai";

const primary = new ChatOpenAI({ model: "gpt-4o-mini" });
const fallback = new ChatOpenAI({ model: "gpt-4o" });

const resilientChain = primary.pipe(new StringOutputParser()).withFallbacks({
  fallbacks: [fallback.pipe(new StringOutputParser())],
});
```

## 5. 链路追踪：LangSmith

LangSmith 记录每次调用的输入、输出、token、延迟与工具调用：

```env
# .env
LANGCHAIN_TRACING_V2=true
LANGCHAIN_API_KEY=your-langsmith-key
LANGCHAIN_PROJECT=kanban-cs
```

```ts
import "dotenv/config";
```

开启后所有 Runnable 与 Agent 调用自动上报，可在面板按输入检索、回放与调试。

## 6. 与 Web 框架集成

Node.js HTTP 服务的典型形态：接收请求 -> 组装会话历史 -> 调用链或 Agent -> 流式返回。示例：

```ts
import { createServer } from "node:http";
import { ChatPromptTemplate } from "@langchain/core/prompts";
import { StringOutputParser } from "@langchain/core/output_parsers";
import { ChatOpenAI } from "@langchain/openai";

const prompt = ChatPromptTemplate.fromMessages([
  ["human", "{question}"],
]);
const model = new ChatOpenAI({ model: "gpt-4o-mini" });
const chain = prompt.pipe(model).pipe(new StringOutputParser());

const server = createServer(async (req, res) => {
  res.writeHead(200, { "Content-Type": "text/plain; charset=utf-8" });
  const stream = await chain.stream({ question: "讲个笑话" });
  for await (const chunk of stream) {
    res.write(chunk);
  }
  res.end();
});

server.listen(3000);
```

生产按部署形态选择：独立 Node 服务、Next.js Route Handler 或 Serverless 函数。LLM 调用是无状态的，服务实例可横向扩展，会话状态交给 checkpointer。

## 7. 部署要点

- 模型 Key 与 LangSmith Key 用 Secret 管理，不进代码仓库。
- 启动前对提示模板与工具做冒烟测试。
- 关键链路加超时、重试与回退。
- 记录每次调用的 token 消耗，监控成本。
- 会话存储与向量库选生产级方案（pgvector、PostgresSaver 等）。
- 流式接口注意超时与会话保持，避免代理层缓冲。

## 8. 动手任务

1. 用 `MemorySaver` + `thread_id` 让 Agent 记住多轮上下文。
2. 接入 `summarizationMiddleware`，构造长对话验证压缩触发。
3. 用 `streamEvents` 输出流式回复。
4. 配置 LangSmith，观察一次调用的完整追踪。
5. 在 Node HTTP 服务中暴露流式聊天接口。

## 阶段四验收

- 能用 checkpointer 实现会话记忆。
- 能实现流式输出与长对话压缩。
- 能为链配置超时、重试与回退。
- 能用 LangSmith 追踪并定位问题。
- 能集成到 Web 服务并说明部署形态。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 记忆不生效 | 每次调用传同一 `thread_id` 且 checkpointer 正确注入 |
| 历史被截断 | 调整 `keep` 与 `trigger` 参数 |
| 流式无效果 | 确认消费端未做缓冲、协议支持分块 |
| LangSmith 无数据 | 确认 `LANGCHAIN_TRACING_V2` 与 Key 配置 |
| 回退未生效 | 确认 `.withFallbacks` 挂在与调用一致的链上 |
| 并发会话串扰 | 每个会话使用独立 `thread_id` |

## 进入下一阶段的条件

你能够为应用接入记忆、流式与追踪并完成部署。此时进入 [综合实战：知识库客服机器人](./project-practice.md) 串起全流程。
