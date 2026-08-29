# LangChain 新手导读

## 适合谁

这套路线适合已经掌握 TypeScript 与异步编程，准备构建 LLM 应用的开发者。学习重点是 LangChain.js 的模型接入、LCEL 组合、RAG 与 Agent，再逐步掌握记忆、流式和生产工程化。

## 开始前准备

- 掌握 TypeScript 基本类型和异步编程，可参考 [TypeScript 学习路线](../typescript-learning-path/README.md)。
- 熟悉 Node.js 运行时、npm 与 `tsx` 运行脚本，可参考 [Node.js 学习路线](../nodejs-learning-path/README.md)。
- 了解 HTTP 与 REST 基本概念，方便理解模型 API 调用。
- 有一个可用的模型服务 Key（OpenAI 兼容接口即可），存放在环境变量中。
- 能运行 Node.js 和 npm。

```bash
node --version
npm --version
```

## 模型 Key 与安全

模型 Key 属于敏感凭据，统一放入 `.env` 并通过 `process.env` 读取，不要把真实 Key 写入代码或提交到仓库。`.env` 加入 `.gitignore`。文档示例使用占位符：

```env
OPENAI_API_KEY=your-api-key-here
```

## 学习顺序

| 阶段 | 先回答的问题 | 阶段产出 |
| --- | --- | --- |
| 零 | 如何初始化项目并跑通一条链？ | 可运行的最小链 |
| 一 | 如何组织提示并约束模型输出？ | 结构化输出的问答链 |
| 二 | 如何让模型基于自有文档回答？ | 可用的 RAG 知识库问答 |
| 三 | 如何让模型调用外部工具完成任务？ | 可调用工具的 Agent |
| 四 | 如何让应用记住上下文并生产部署？ | 生产可用的对话服务 |

## 每阶段学习方法

1. 每个概念先写最小示例并实际运行，观察输出。
2. 在环境变量中配置模型 Key，用一个常量 Topic 反复跑同一条链。
3. 对比 `.invoke()`、`.batch()`、`.stream()` 三种调用的行为。
4. 在阶段二中准备一份自己的文档（Markdown 或 TXT）作为检索语料。
5. 在生产章节接入真实调用，观察流式输出与追踪面板。

## 常见排错顺序

```text
安装与导入
  -> 环境变量与模型 Key
  -> 模型实例化与消息格式
  -> 提示模板变量缺失
  -> 结构化输出 Schema 不匹配
  -> LCEL 链类型不匹配
  -> 向量检索结果为空
  -> Agent 工具调用失败
  -> 记忆与流式问题
```

## 阶段项目路线

从 [综合实战：知识库客服机器人](./project-practice.md) 开始，每完成一个阶段就增加一个能力：最小链、提示工程、RAG、Agent 工具与生产化部署。

## 完成标准

- 能初始化 LangChain.js 项目并接入 Chat Model。
- 能用提示模板组织输入并得到结构化输出。
- 能用 LCEL 组合链并完成流式调用。
- 能基于向量检索构建 RAG 问答。
- 能定义工具并让 Agent 完成多步推理。
- 能接入记忆、流式输出与链路追踪并部署。
- 完成 [学习评估](./learning-assessment.md) 中的阶段考核与项目评分。
