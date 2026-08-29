# 阶段二：LCEL 链与检索增强（RAG）

**目标**：掌握 Runnable 组合方式与并行执行，构建文档切分、向量化、检索与生成的完整 RAG 链路，让模型基于自有知识回答。

## 1. Runnable 组合

LCEL 用 `.pipe()` 串联，输出作为下一步输入：

```ts
import { ChatPromptTemplate } from "@langchain/core/prompts";
import { StringOutputParser } from "@langchain/core/output_parsers";
import { ChatOpenAI } from "@langchain/openai";

const prompt = ChatPromptTemplate.fromMessages([
  ["system", "你是产品经理，输出要点列表。"],
  ["human", "{topic}"],
]);

const model = new ChatOpenAI({ model: "gpt-4o-mini" });
const chain = prompt.pipe(model).pipe(new StringOutputParser());
```

## 2. RunnableParallel

并行分支，适合多个独立任务同时执行：

```ts
import { RunnableParallel, RunnablePassthrough } from "@langchain/core/runnables";

const branches = RunnableParallel.from({
  要点: prompt.pipe(model).pipe(new StringOutputParser()),
  一句话版本: ChatPromptTemplate.fromMessages([
    ["human", "用一句话概括 {topic}"],
  ]).pipe(model).pipe(new StringOutputParser()),
});

const result = await branches.invoke({ topic: "大模型应用开发" });
```

## 3. 文档加载与切分

```ts
import { TextLoader } from "langchain/document_loaders/fs/text";
import { RecursiveCharacterTextSplitter } from "@langchain/textsplitters";

const loader = new TextLoader("./docs/faq.txt");
const docs = await loader.load();

const splitter = new RecursiveCharacterTextSplitter({
  chunkSize: 800,
  chunkOverlap: 120,
});

const splits = await splitter.splitDocuments(docs);
console.log(splits.length, "个分块");
```

- `chunkSize` 控制块大小，`chunkOverlap` 保留上下文衔接。
- 切分粒度影响检索质量：过大召回不精确，过小上下文不足。

## 4. 向量化与向量库

```ts
import { OpenAIEmbeddings } from "@langchain/openai";
import { MemoryVectorStore } from "@langchain/classic/vectorstores/memory";

const embeddings = new OpenAIEmbeddings();
const vectorStore = new MemoryVectorStore(embeddings);

await vectorStore.addDocuments(splits);

const results = await vectorStore.similaritySearch("如何退款？", 3);
```

- Embeddings 把文本转为向量，语义相近的文本向量距离近。
- `MemoryVectorStore` 内存向量库适合学习与原型；生产改用数据库向量存储（如 pgvector、Pinecone）。
- `similaritySearch` 返回最相似的文档块。

## 5. 把向量库变成检索器

```ts
const retriever = vectorStore.asRetriever({ k: 4 });

const docs = await retriever.invoke("发货时间是多久？");
```

检索器是 Runnable，可以 `.pipe()` 进链，是 RAG 的标准入口。

## 6. 组装 RAG 链

```ts
import { ChatPromptTemplate } from "@langchain/core/prompts";
import { StringOutputParser } from "@langchain/core/output_parsers";
import { RunnablePassthrough, RunnableSequence } from "@langchain/core/runnables";
import { formatDocumentsAsString } from "langchain/util/document";

const prompt = ChatPromptTemplate.fromMessages([
  [
    "system",
    "只依据下面的资料回答问题，资料不足就如实说明。\n\n{context}",
  ],
  ["human", "{question}"],
]);

const ragChain = RunnableSequence.from([
  {
    context: retriever.pipe(formatDocumentsAsString),
    question: new RunnablePassthrough(),
  },
  prompt,
  model,
  new StringOutputParser(),
]);

const answer = await ragChain.invoke("退货需要什么条件？");
console.log(answer);
```

流程：`question` 同时传给检索器（得到 context）与提示模板，模型基于检索资料作答。

## 7. RAG 质量三要素

- 语料质量：文档权威、格式干净。
- 切分策略：块大小与重叠匹配问题粒度。
- 检索相关度：`k` 值、相似度阈值、重排序。

## 8. 动手任务

1. 准备一份自己的文档（FAQ 或产品说明 TXT）。
2. 用 `TextLoader` 加载并用 `RecursiveCharacterTextSplitter` 切分。
3. 用 `OpenAIEmbeddings` + `MemoryVectorStore` 建索引。
4. 用 `similaritySearch` 验证检索相关度，调整切分参数。
5. 组装完整 RAG 链，测试在资料范围内回答与范围外回答。
6. 用 `RunnableParallel` 同时输出检索来源与最终答案。

## 阶段二验收

- 能加载并切分文档。
- 能向量化、建索引并检索相关分块。
- 能组装提示、检索与模型的完整 RAG 链。
- 能解释切分参数对检索质量的影响。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 检索结果为空 | 确认已 `addDocuments`，且查询词与语料语言一致 |
| 答案编造 | 提示约束未生效，或检索块未正确注入 |
| 上下文超限 | 减小 `chunkSize` 或 `k` |
| 相关度差 | 调整切分粒度、增大 `chunkOverlap`、换模型 |
| MemoryVectorStore 清理 | 进程重启后索引消失，生产用持久化存储 |
| 嵌入维度不一致 | 同一模型的嵌入向量维度需一致 |

## 进入下一阶段的条件

你能够构建并调优一条 RAG 链路。此时进入 [阶段三：Agent 与工具](./stage-3-agents-and-tools.md)。
