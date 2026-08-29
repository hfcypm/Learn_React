# LangChain 学习路线图（基础 → 生产）

**目标**：掌握 LangChain.js 的模型与提示、LCEL 链式组合、检索增强（RAG）、Agent 与工具，以及记忆、流式与可观测的生产工程化，能够在 TypeScript/Node.js 项目中构建可上线的 LLM 应用。

## 先看这里

- [新手导读](./start-here.md)：前置条件、学习顺序、排错顺序和完成标准。
- [综合实战：知识库客服机器人](./project-practice.md)：按阶段逐步扩展同一个项目。
- [学习评估与项目评分](./learning-assessment.md)：阶段考核、排错题、项目评分和评审问题。

## 学习阶段总览

| 阶段 | 名称 | 目标 |
|---|---|---|
| 零 | LangChain 基础与工具链 | 理解框架定位与包结构，初始化项目，用 LCEL 跑通第一条链 |
| 一 | 模型与提示词 | 掌握 Chat Model、提示模板、消息结构与结构化输出 |
| 二 | LCEL 链与检索增强 | 掌握链式组合、Runnable 并行与 RAG 检索问答 |
| 三 | Agent 与工具 | 掌握工具定义、工具调用与多步推理 Agent |
| 四 | 生产化交付 | 掌握记忆、流式、可观测、回退与部署 |

## 快速导航

- [阶段零：LangChain 基础与工具链](./stage-0-foundations.md)
- [阶段一：模型与提示词](./stage-1-models-and-prompts.md)
- [阶段二：LCEL 链与检索增强](./stage-2-chains-and-rag.md)
- [阶段三：Agent 与工具](./stage-3-agents-and-tools.md)
- [阶段四：生产化交付](./stage-4-production.md)
- [综合实战：知识库客服机器人](./project-practice.md)

## 学习原则

- LCEL（LangChain Expression Language）是组合一切的统一方式，链、检索器、工具都能 pipe。
- 一切可运行对象都是 Runnable，统一实现 invoke、batch、stream 三种调用。
- 先跑通最小示例，再叠加记忆、流式与可观测性。
- 用结构化输出（Schema）约束模型返回，少做字符串解析。
- 检索增强优先于把文档塞进上下文，控制 token 与相关度。
- 工具是 Agent 与外部世界交互的唯一通道，Schema 要精确。
- 生产环境必须做回退、超时、流式与链路追踪。

## 阶段成果

- 能初始化 LangChain.js 项目并连接一个 Chat Model。
- 能用 PromptTemplate、ChatPromptTemplate 组织提示并约束输出。
- 能用 LCEL 组合提示、模型、解析器与检索器形成完整链路。
- 能构建基于向量检索的知识库问答。
- 能用工具与 Agent 完成多步推理任务。
- 能为应用接入记忆、流式输出与 LangSmith 追踪并部署。

## 每阶段固定学习模板

每个阶段按照“目标 -> 前置知识 -> 核心问题 -> 最小示例 -> 项目增量 -> 验收 -> 排错”执行。先完成阶段项目，再阅读扩展主题；这样能够把 LangChain 概念连接到可运行结果。

## 版本边界

学习主线使用 LangChain.js（TypeScript）生态，包结构、模型接口与 Agent 创建方式以官方 JavaScript 文档为准。新老 API 差异、Python 版差异参见 [版本边界与迁移](./version-governance.md)。

## 前置知识

LangChain.js 是 TypeScript/Node.js 生态，建议先完成 [Node.js 学习路线](../nodejs-learning-path/README.md) 掌握运行时与异步编程，再结合 [TypeScript 学习路线](../typescript-learning-path/README.md) 使用类型安全。需要调用模型，请自备一个兼容 OpenAI 接口的模型服务 Key。与 Next.js 集成可对照 [Next.js 学习路线](../nextjs-learning-path/README.md) 与 [AI 应用前端学习路线](../ai-learning-path/README.md)。

## 官方资源

- [LangChain 官方文档（JavaScript）](https://docs.langchain.com/oss/javascript/langchain/overview)
- [LangChain.js API 参考](https://api.js.langchain.com/)
- [LangChain JavaScript GitHub](https://github.com/langchain-ai/langchainjs)
- [LangGraph.js 文档](https://langchain-ai.github.io/langgraphjs/)
- [LangSmith 文档](https://docs.smith.langchain.com/)
