# LangChain 版本边界与迁移

**目标**：让学习内容、示例项目和生产项目拥有清晰的版本边界，降低大版本升级和 API 混用风险。

## 1. 版本差异

LangChain 生态更新较快，新旧 API 并存。常用 API 沿革：

| 能力 | 旧写法 | 当前主线写法 |
|---|---|---|
| 工具定义 | `DynamicStructuredTool` 类实例化 | `tool()` 函数 + zod Schema |
| Agent 创建 | `createOpenAIFunctionsAgent` / `createReactAgent` 组装 | `createAgent({ model, tools })` |
| 事件流 | `Runnable` 的 `streamEvents`（v1/v2） | `streamEvents(..., { version: "v3" })` |
| 记忆 | `BufferMemory` / `ChatMessageHistory` 链式注入 | LangGraph checkpointer + `thread_id` |
| 长对话 | 手动裁剪历史 | `summarizationMiddleware` |

## 2. 包边界

| 包 | 使用边界 |
|---|---|
| `@langchain/core` | 消息、提示模板、输出解析、Runnable |
| `langchain` | 链、工具、Agent 创建、文档加载器 |
| `@langchain/openai` | OpenAI 兼容模型与嵌入 |
| `@langchain/langgraph` | checkpointer、状态化运行时 |
| `@langchain/community` | 第三方集成，注意单独安装对应子包 |
| `@langchain/classic` | 曾属 `langchain` 的核心集成被迁移至此 |

## 3. 每个示例必须记录的元数据

```markdown
> 适用范围：LangChain.js + @langchain/openai（OpenAI 兼容接口）
> 需要的工具：Node.js、npm、tsx、模型服务 Key
> 验证命令：npx tsx <示例脚本>
> 最后复核日期：YYYY-MM-DD
```

## 4. 升级与迁移要点

- 先阅读官方 [迁移指南](https://docs.langchain.com/oss/javascript/langchain/migrations) 与 release notes。
- 工具与 Agent：把 `DynamicStructuredTool` 改写为 `tool()`；Agent 创建统一用 `createAgent`。
- 记忆：从链式 `BufferMemory` 迁移到 LangGraph `MemorySaver` + `thread_id`。
- 事件流：升级到 `streamEvents` v3，按 `stream.messages` 与 `stream.values` 消费。
- 包路径：从 `langchain` 单一包导入的集成组件迁至对应分包（如向量库、加载器）。
- 检查导入路径与废弃 API，避免新旧混用。

## 5. 升级前检查

1. 阅读官方升级指南与 breaking changes。
2. 检查目标版本对应的包与导入路径。
3. 检查项目中已废弃的 API（工具类、记忆类、事件流版本）。
4. 在分支环境完整验证链、RAG、Agent 与流式调用。

## 6. 升级后检查

- 链与 Agent 调用正常。
- 结构化输出 Schema 行为一致。
- 记忆与会话隔离正常。
- 流式输出与事件版本一致。
- 工具调用与回退行为一致。
- 构建与部署通过。
