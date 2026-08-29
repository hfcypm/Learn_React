# 阶段零：LangChain 基础与工具链

**目标**：理解 LangChain 的架构与定位，初始化项目，接入一个 Chat Model，用 LCEL 跑通第一条链。

## 1. LangChain 是什么

LangChain 是构建大语言模型（LLM）应用的框架，提供标准化的模型接口、提示管理、链式组合、检索、Agent 与可观测能力。它不绑定特定模型厂商，通过统一接口切换不同服务。

核心抽象：

- Runnable：一切可执行单元的通用接口，统一 `invoke`、`batch`、`stream`。
- LCEL：用 `.pipe()` 组合 Runnable 的表达式语言。
- Chat Model：基于消息对话的模型接口。
- 工具与 Agent：让模型调用外部函数完成多步任务。

## 2. 包结构与选型

| 包 | 职责 |
|---|---|
| `@langchain/core` | 核心抽象：Runnable、消息、提示模板、输出解析 |
| `langchain` | 组合生态：链、工具、Agent 创建 |
| `@langchain/openai` | OpenAI 兼容接口的模型与向量化 |
| `@langchain/langgraph` | 状态化多步执行、记忆与 Agent 运行时 |
| `@langchain/community` | 第三方集成 |

学习主线使用 `@langchain/core` 与 `langchain`，模型用 `@langchain/openai`（OpenAI 兼容即可）。

## 3. 初始化项目

```bash
mkdir llm-app && cd llm-app
npm init -y

# 安装依赖
npm install langchain @langchain/core @langchain/openai dotenv
npm install -D typescript tsx @types/node

# 初始化 TypeScript
npx tsc --init
```

配置环境变量：

```env
# .env
OPENAI_API_KEY=your-api-key-here
```

`tsconfig.json` 中启用 `moduleResolution: "bundler"` 以便正确解析包导出。

## 4. 接入 Chat Model

```ts
// src/model.ts
import { ChatOpenAI } from "@langchain/openai";
import "dotenv/config";

export const model = new ChatOpenAI({
  model: "gpt-4o-mini",
  temperature: 0.7,
});
```

- 实例化即完成配置，不发起网络请求。
- `model` 参数对应服务商的具体模型名。
- 使用兼容 OpenAI 接口的服务时，通过 `apiKey`、`baseUrl` 指向自建网关。

## 5. 第一条链：LCEL

```ts
// src/chain.ts
import { ChatPromptTemplate } from "@langchain/core/prompts";
import { StringOutputParser } from "@langchain/core/output_parsers";
import { model } from "./model";

const prompt = ChatPromptTemplate.fromMessages([
  ["system", "你是一个简洁的中文助手。"],
  ["human", "{question}"],
]);

const chain = prompt.pipe(model).pipe(new StringOutputParser());

const result = await chain.invoke({
  question: "用一句话解释什么是 LCEL。",
});
console.log(result);
```

用 `tsx` 运行：

```bash
npx tsx src/chain.ts
```

`prompt.pipe(model).pipe(parser)` 把三个 Runnable 串成一条链，`invoke` 返回最终字符串。

## 6. Runnable 三种调用

```ts
// 批量
const answers = await chain.batch([
  { question: "什么是 Agent？" },
  { question: "什么是 RAG？" },
]);

// 流式
const stream = await chain.stream({ question: "讲个三句话的笑话。" });
for await (const chunk of stream) {
  process.stdout.write(chunk);
}
```

- `invoke`：一次完整调用。
- `batch`：数组批量调用。
- `stream`：逐块返回，用于打字机效果。

## 7. 调试与错误处理

```ts
try {
  const result = await chain.invoke({ question: "你好" });
  console.log(result);
} catch (error) {
  console.error("调用失败：", error);
}
```

常见失败原因：Key 缺失、模型名错误、网络不通。所有 Runnable 都可在链首 `.withFallbacks()` 或添加超时配置（见阶段四）。

## 8. 动手任务

1. 初始化项目并配置 `.env` 中的模型 Key。
2. 实例化 `ChatOpenAI` 并直接调用一次，观察返回结构。
3. 用 `ChatPromptTemplate` + `pipe` 构建第一条链。
4. 分别用 `invoke`、`batch`、`stream` 调用同一条链。
5. 接入 `StringOutputParser`，对比有/无解析器的返回值差异。

## 阶段零验收

- 能解释 Runnable 与 LCEL 的角色。
- 能初始化项目并正确配置模型 Key。
- 能用 `.pipe()` 组合提示、模型与解析器。
- 能用 `invoke`、`batch`、`stream` 三种方式调用链。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| `OPENAI_API_KEY` 未找到 | 确认 `.env` 存在且 `dotenv/config` 已导入 |
| 401/403 | Key 错误或余额不足 |
| 404 model not found | 模型名拼写或服务不支持该模型 |
| 类型导入报错 | 确认 `moduleResolution: "bundler"` |
| `tsx` 命令不存在 | 确认开发依赖已安装 |
| 流式无输出 | 确认模型接口支持流式 |

## 进入下一阶段的条件

你能够连接模型并用 LCEL 跑通一条链。此时进入 [阶段一：模型与提示词](./stage-1-models-and-prompts.md)。
