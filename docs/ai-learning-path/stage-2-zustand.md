# Zustand：客户端状态

## 1. 解决什么问题

Zustand 用于管理跨组件共享的客户端状态。AI 应用常见状态包括当前会话、输入草稿、生成状态、AbortController、选中的文件和 Agent 执行步骤。

服务端返回的长期数据交给 TanStack Query；短生命周期的交互状态放在 Zustand 中。

## 2. 安装与 Store

```bash
npm install zustand
```

```tsx
import { create } from 'zustand'

type RunStatus = 'idle' | 'running' | 'completed' | 'error'

type ChatStore = {
  conversationId: string | null
  draft: string
  answer: string
  runStatus: RunStatus
  setDraft: (draft: string) => void
  beginRun: (conversationId: string) => void
  appendAnswer: (chunk: string) => void
  finishRun: () => void
  resetRun: () => void
}

export const useChatStore = create<ChatStore>((set) => ({
  conversationId: null,
  draft: '',
  answer: '',
  runStatus: 'idle',
  setDraft: (draft) => set({ draft }),
  beginRun: (conversationId) => set({ conversationId, answer: '', runStatus: 'running' }),
  appendAnswer: (chunk) => set((state) => ({ answer: state.answer + chunk })),
  finishRun: () => set({ runStatus: 'completed' }),
  resetRun: () => set({ answer: '', runStatus: 'idle' }),
}))
```

## 3. 在组件中读取

```tsx
export function Composer() {
  const draft = useChatStore((state) => state.draft)
  const setDraft = useChatStore((state) => state.setDraft)

  return (
    <textarea
      value={draft}
      onChange={(event) => setDraft(event.target.value)}
      placeholder="输入问题"
    />
  )
}
```

选择器只读取需要的字段，能够减少无关状态变化带来的重新渲染。

## 4. 设计边界

- `draft`、弹窗、侧边栏、当前步骤属于客户端状态。
- 会话列表、历史消息、用户资料属于服务端数据。
- `AbortController` 适合存放在请求管理模块或组件生命周期中，避免把不可序列化对象持久化到 LocalStorage。
- 需要刷新后保留的设置可以使用 `persist`，敏感信息应保留在服务端或安全的会话机制中。
- 多个页面共同消费的状态才进入全局 Store，局部状态留在组件内部。

## 5. Agent 状态结构

```ts
type AgentStep = {
  id: string
  name: string
  status: 'queued' | 'running' | 'completed' | 'failed'
  detail?: string
  startedAt?: number
  endedAt?: number
}

type AgentRunState = {
  runId: string | null
  steps: AgentStep[]
  finalAnswer: string
  status: 'idle' | 'running' | 'completed' | 'failed'
}
```

把后端事件转换为稳定的前端状态，组件只负责渲染步骤时间线、进度条和最终结果。

## 6. 练习

1. 增加“停止生成”按钮状态。
2. 增加当前会话 ID 切换。
3. 把 Agent 步骤事件转换为步骤时间线。
4. 为草稿增加刷新后恢复功能，并评估持久化数据的安全性。
