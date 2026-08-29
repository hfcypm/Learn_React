# LangChain 综合实战：知识库客服机器人

## 1. 项目目标

用 LangChain.js 构建一个知识库客服机器人，覆盖模型接入、提示工程、RAG 检索、工具调用、会话记忆、流式输出与部署。项目按阶段扩展，最终交付一个可对话的客服服务。

```text
初始化 -> 最小对话链 -> 知识库 RAG -> 客服工具 Agent
    -> 记忆与流式 -> 生产部署
```

## 2. 需求

- 基于自有知识库回答产品与售后问题。
- 查不到资料时如实说明，不编造。
- 能查询订单状态、天气等外部信息。
- 支持多轮对话，记住上下文。
- 输出流式呈现，降低首字延迟。
- 生产环境可追踪、可回退、可部署。

## 3. 技术选择

| 技术 | 用途 |
|---|---|
| TypeScript | 语言 |
| LangChain.js | LLM 应用框架 |
| `@langchain/openai` | 模型与向量化 |
| `@langchain/classic` | 内存向量库（学习用） |
| `@langchain/langgraph` | 会话记忆 checkpointer |
| LangSmith | 链路追踪 |

## 4. 项目结构

```text
llm-cs-bot/
├── src/
│   ├── model.ts          # 模型实例
│   ├── rag.ts            # 知识库加载、切分、检索
│   ├── tools.ts          # 客服工具定义
│   ├── agent.ts          # 客服 Agent 组装
│   └── server.ts         # HTTP 流式接口
├── docs/faq.txt          # 知识库语料
├── .env                  # 模型 Key（不进仓库）
└── .env.example          # 占位符模板
```

## 5. 初始化

```bash
mkdir llm-cs-bot && cd llm-cs-bot
npm init -y
npm install langchain @langchain/core @langchain/openai @langchain/classic @langchain/langgraph dotenv
npm install -D typescript tsx @types/node
```

```env
# .env.example
OPENAI_API_KEY=your-api-key-here
LANGCHAIN_TRACING_V2=true
LANGCHAIN_API_KEY=your-langsmith-key
LANGCHAIN_PROJECT=llm-cs-bot
```

## 6. 模型与最小对话链

```ts
// src/model.ts
import { ChatOpenAI } from "@langchain/openai";
import "dotenv/config";

export const model = new ChatOpenAI({
  model: "gpt-4o-mini",
  temperature: 0.2,
});
```

```ts
// src/chain.ts
import { ChatPromptTemplate } from "@langchain/core/prompts";
import { StringOutputParser } from "@langchain/core/output_parsers";
import { model } from "./model";

const prompt = ChatPromptTemplate.fromMessages([
  ["system", "你是客服助手，回答简洁，最多三句。"],
  ["human", "{question}"],
]);

export const basicChain = prompt.pipe(model).pipe(new StringOutputParser());
```

## 7. 知识库 RAG

```ts
// src/rag.ts
import { TextLoader } from "langchain/document_loaders/fs/text";
import { RecursiveCharacterTextSplitter } from "@langchain/textsplitters";
import { OpenAIEmbeddings } from "@langchain/openai";
import { MemoryVectorStore } from "@langchain/classic/vectorstores/memory";
import { formatDocumentsAsString } from "langchain/util/document";

const loader = new TextLoader("./docs/faq.txt");
const docs = await loader.load();

const splitter = new RecursiveCharacterTextSplitter({
  chunkSize: 800,
  chunkOverlap: 120,
});
const splits = await splitter.splitDocuments(docs);

const vectorStore = new MemoryVectorStore(new OpenAIEmbeddings());
await vectorStore.addDocuments(splits);

export const retriever = vectorStore.asRetriever({ k: 4 });
export { formatDocumentsAsString };
```

```ts
// src/rag-chain.ts
import { ChatPromptTemplate } from "@langchain/core/prompts";
import { StringOutputParser } from "@langchain/core/output_parsers";
import { RunnablePassthrough, RunnableSequence } from "@langchain/core/runnables";
import { model } from "./model";
import { retriever, formatDocumentsAsString } from "./rag";

const prompt = ChatPromptTemplate.fromMessages([
  [
    "system",
    "只依据下面的资料回答客服问题，资料不足就如实说明。\n\n{context}",
  ],
  ["human", "{question}"],
]);

export const ragChain = RunnableSequence.from([
  {
    context: retriever.pipe(formatDocumentsAsString),
    question: new RunnablePassthrough(),
  },
  prompt,
  model,
  new StringOutputParser(),
]);
```

## 8. 客服工具

```ts
// src/tools.ts
import { tool } from "langchain";
import { z } from "zod";

export const getOrderStatus = tool(
  async ({ orderId }) => {
    // 生产环境改为查询真实订单系统
    return `订单 ${orderId} 已发货，预计明天送达。`;
  },
  {
    name: "get_order_status",
    description: "查询订单状态，用户询问订单、物流时调用。",
    schema: z.object({
      orderId: z.string().describe("订单号，如 SO-2026-0001"),
    }),
  },
);

export const searchKnowledge = tool(
  async ({ query }) => {
    const docs = await retriever.invoke(query);
    return docs.map((d) => d.pageContent).join("\n");
  },
  {
    name: "search_knowledge_base",
    description: "在客服知识库中检索资料，回答产品与售后政策问题前调用。",
    schema: z.object({
      query: z.string().describe("检索关键词"),
    }),
  },
);
```

## 9. 客服 Agent

```ts
// src/agent.ts
import { createAgent } from "langchain";
import { MemorySaver } from "@langchain/langgraph";
import { getOrderStatus, searchKnowledge } from "./tools";

export function createCsAgent() {
  return createAgent({
    model: "gpt-4o-mini",
    tools: [searchKnowledge, getOrderStatus],
    systemPrompt: [
      "你是公司客服助手。回答产品与售后政策问题时先检索知识库，",
      "用户询问订单时查询订单系统，资料不足就如实说明。",
    ].join(""),
    checkpointer: new MemorySaver(),
  });
}
```

## 10. 流式 HTTP 接口

```ts
// src/server.ts
import { createServer } from "node:http";
import { createCsAgent } from "./agent";
import "dotenv/config";

const agent = createCsAgent();

const server = createServer(async (req, res) => {
  if (req.method !== "POST" || req.url !== "/api/chat") {
    res.writeHead(404).end();
    return;
  }

  const body = await new Promise<string>((resolve) => {
    let data = "";
    req.on("data", (c) => (data += c));
    req.on("end", () => resolve(data));
  });
  const { threadId, question } = JSON.parse(body);

  res.writeHead(200, { "Content-Type": "text/plain; charset=utf-8" });
  const stream = await agent.streamEvents(
    { messages: [{ role: "user", content: question }] },
    {
      version: "v3",
      configurable: { thread_id: threadId },
    },
  );

  for await (const message of stream.messages) {
    for await (const delta of message.text) {
      res.write(delta);
    }
  }
  res.end();
});

server.listen(3000, () => console.log("客服机器人运行在 :3000"));
```

`threadId` 由前端按会话生成，同一会话共享记忆。

## 11. 实施顺序

1. 初始化项目并配置模型。
2. 跑通最小对话链。
3. 加载知识库，构建 RAG 链并验证检索质量。
4. 定义客服工具并组装 Agent。
5. 接入记忆与流式输出。
6. 配置 LangSmith 并部署到 Node 服务。

## 12. 验收清单

- [ ] 最小对话链可运行。
- [ ] RAG 能回答知识库内问题，范围外问题如实拒绝。
- [ ] Agent 能自主选择检索或订单工具。
- [ ] 多轮对话记忆正确。
- [ ] 流式输出逐字呈现。
- [ ] LangSmith 能看到完整调用链路。
- [ ] 模型 Key 未硬编码，环境变量管理。

## 13. 按阶段学习卡片

| 阶段 | 项目增量 |
|---|---|
| 零 | 初始化、模型接入、最小链 |
| 一 | 提示模板、结构化输出 |
| 二 | 知识库加载、切分、RAG 链 |
| 三 | 客服工具与 Agent 组装 |
| 四 | 记忆、流式、追踪与部署 |
