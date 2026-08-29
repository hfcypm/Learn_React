# 阶段一：模型与提示词

**目标**：掌握 Chat Model 的消息结构、提示模板与结构化输出，构建稳定、可解析的问答链。

## 1. 消息与 Chat Model

Chat Model 以消息列表为输入、以 AI 消息为输出。三种基础角色：

| 消息类 | 角色 | 用途 |
|---|---|---|
| `SystemMessage` | system | 设定助手行为与约束 |
| `HumanMessage` | human | 用户输入 |
| `AIMessage` | ai | 模型回复，可携带工具调用 |

```ts
import { ChatOpenAI } from "@langchain/openai";
import { HumanMessage, SystemMessage } from "@langchain/core/messages";

const model = new ChatOpenAI({ model: "gpt-4o-mini" });

const response = await model.invoke([
  new SystemMessage("你是一名严谨的译者，只输出译文。"),
  new HumanMessage("把这句话翻译成英文：学习 LangChain 让构建 LLM 应用变得高效。"),
]);

console.log(response.content);
```

## 2. PromptTemplate

单一字符串提示，用变量插值：

```ts
import { PromptTemplate } from "@langchain/core/prompts";

const template = PromptTemplate.fromTemplate(
  "请用 {style} 风格为产品 {product} 写一句广告语。",
);

const prompt = await template.invoke({
  style: "幽默",
  product: "降噪耳机",
});
```

## 3. ChatPromptTemplate

多消息提示模板，支持角色占位与消息占位：

```ts
import { ChatPromptTemplate, MessagesPlaceholder } from "@langchain/core/prompts";

const prompt = ChatPromptTemplate.fromMessages([
  ["system", "你是客服助手，语气友好，回答不超过 3 句。"],
  new MessagesPlaceholder("history"), // 历史消息在运行时注入
  ["human", "{question}"],
]);

const messages = await prompt.invoke({
  history: [], // 传入消息数组
  question: "我的订单多久能发货？",
});
```

`MessagesPlaceholder` 让历史消息作为整体变量注入，是多轮对话的基础。

## 4. 输出解析

```ts
import { StringOutputParser } from "@langchain/core/output_parsers";
import { ChatPromptTemplate } from "@langchain/core/prompts";

const prompt = ChatPromptTemplate.fromMessages([
  ["human", "把 {word} 翻译成英语，只输出单词。"],
]);

const chain = prompt.pipe(model).pipe(new StringOutputParser());

const result = await chain.invoke({ word: "苹果" });
// "apple"
```

`StringOutputParser` 提取消息内容为纯字符串，让下游消费简单。

## 5. 结构化输出

用 zod Schema 约束模型返回 JSON，是比字符串解析可靠的方式：

```ts
import { z } from "zod";
import { ChatOpenAI } from "@langchain/openai";

const schema = z.object({
  sentiment: z.enum(["positive", "neutral", "negative"]),
  summary: z.string(),
  score: z.number().min(0).max(10),
});

const model = new ChatOpenAI({ model: "gpt-4o-mini" });
const structured = model.withStructuredOutput(schema);

const result = await structured.invoke("这个产品很好用，就是有点贵。");
// { sentiment: "neutral", summary: "...", score: 7 }
```

- 返回值直接符合 Schema，类型安全。
- Schema 描述越具体，模型遵守越好。
- 可在字段上加 `.describe()` 提升抽取准确性。

## 6. 结合提示模板的结构化链

```ts
const prompt = ChatPromptTemplate.fromMessages([
  ["system", "你是文本分析器，严格按 Schema 返回结果。"],
  ["human", "{text}"],
]);

const chain = prompt.pipe(structured);
const result = await chain.invoke({ text: "发货很快，包装也很用心！" });
```

提示与结构化输出组合，适合情感分析、信息抽取、分类等任务。

## 7. 温度与采样参数

```ts
const model = new ChatOpenAI({
  model: "gpt-4o-mini",
  temperature: 0.2,   // 越低越确定，适合抽取与分类
  maxTokens: 512,     // 限制输出长度
});
```

- 事实抽取、分类用低温度。
- 创意写作、对话用高温度。

## 8. 动手任务

1. 用 `SystemMessage` + `HumanMessage` 直接调用模型。
2. 用 `PromptTemplate` 做一次变量插值。
3. 用 `ChatPromptTemplate` + `MessagesPlaceholder` 构建多轮提示。
4. 给阶段零的链换用 `StringOutputParser` 验证输出。
5. 用 zod 定义一个情绪分析 Schema，构建结构化输出链。
6. 对同一输入用 `temperature: 0` 与 `temperature: 1` 各调用一次，对比差异。

## 阶段一验收

- 能区分三种消息角色并正确构造输入。
- 能用模板组织提示并注入历史消息。
- 能用 `StringOutputParser` 得到纯文本。
- 能用 zod Schema 得到结构化输出。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 输出含多余文字 | 提示约束不足或未接输出解析器 |
| 结构化输出字段缺失 | Schema 描述不清晰或模型不支持 |
| 消息角色错误 | 确认 `["system", ...]` 第一个元素是合法角色 |
| MessagesPlaceholder 报错 | 确认运行时传入的是消息数组 |
| temperature 无效 | 部分模型或网关不支持该参数 |

## 进入下一阶段的条件

你能够稳定地组织提示并得到结构化输出。此时进入 [阶段二：LCEL 链与检索增强](./stage-2-chains-and-rag.md)。
