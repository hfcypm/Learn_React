# AI 应用前端学习路线

这套学习包面向已经掌握 HTML、CSS、JavaScript、TypeScript、React、Next.js、Tailwind CSS 和组件库的开发者，集中学习 AI 应用前端最常用的状态、数据请求、实时通信和内容渲染能力。

## 先看这里

- [新手导读](./start-here.md)：前置条件、学习顺序、排错顺序和完成标准。
- [学习评估与项目评分](./learning-assessment.md)：阶段考核、排错题、项目评分和评审问题。

## 学习梯度

从入门到进阶再到实战，每完成一个阶段就向综合案例增加一个能力：

| 阶段 | 梯度 | 名称 | 目标 |
|---|---|---|---|
| 一 | 入门 | [TanStack Query：服务端数据](./stage-1-tanstack-query.md) | 管理服务端数据、缓存和加载状态 |
| 二 | 入门 | [Zustand：客户端状态](./stage-2-zustand.md) | 管理会话、草稿和跨组件共享状态 |
| 三 | 进阶 | [SSE、WebSocket 与流式响应](./stage-3-realtime-streaming.md) | 逐字流式回答、停止、重试和错误恢复 |
| 四 | 进阶 | [文件、Markdown、代码和图表渲染](./stage-4-content-rendering.md) | 安全上传并渲染富内容 |
| 案例 | 高手 | [AI 工作台综合案例](./project-practice.md) | 独立完成完整 AI 前端应用 |

## 学习目标

完成本路线后，你能够：

- 使用 TanStack Query 管理服务端数据、缓存、加载状态和重新获取。
- 使用 Zustand 管理会话、草稿、UI 状态和跨组件共享状态。
- 理解 SSE、WebSocket 和 HTTP 流式响应的使用边界。
- 实现可逐字显示的 AI 回复、停止生成、重试和错误恢复。
- 完成文件上传、上传进度、Markdown、代码高亮和图表渲染。
- 独立设计一个适合 AI Chat 或 AI Agent 的前端页面。

## 推荐顺序

1. [TanStack Query：服务端数据](./stage-1-tanstack-query.md)
2. [Zustand：客户端状态](./stage-2-zustand.md)
3. [SSE、WebSocket 与流式响应](./stage-3-realtime-streaming.md)
4. [文件、Markdown、代码和图表渲染](./stage-4-content-rendering.md)
5. [AI 工作台综合案例](./project-practice.md)

## 技术组合

```text
React + TypeScript
  + Next.js
  + TanStack Query
  + Zustand
  + SSE / fetch streaming / WebSocket
  + react-markdown + remark-gfm
  + Shiki 或 react-syntax-highlighter
  + Recharts 或 ECharts
```

## 前置知识

- React 组件、Hooks、表单和条件渲染
- TypeScript 接口、联合类型、泛型和类型收窄
- HTTP 方法、状态码、JSON 和 Promise
- Next.js 路由、环境变量和 API 请求

## 与 LangChain 的关系

本路线负责 AI 应用的**前端交互**（数据请求、状态、流式渲染）；后端如何组织模型、提示、检索与 Agent 属于 [LangChain 学习路线](../langchain-learning-path/README.md)。两条路线互补：前端消费 LLM 的流式接口，LangChain 负责在服务端生成流式响应。建议先完成本路线掌握流式与渲染，再进入 LangChain 路线搭建后端能力，最终在 [综合实战：知识库客服机器人](../langchain-learning-path/project-practice.md) 中串起前后端。

## 学习方法

- 每章先理解数据流，再运行最小示例。
- 每个示例都观察加载、成功、空数据、失败和取消状态。
- 综合案例先完成静态界面，再接入普通请求，最后接入流式响应。
- 所有用户输入、文件类型、文件大小和服务端返回内容都进行校验。
