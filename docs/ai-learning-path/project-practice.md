# AI 学习工作台综合案例

## 1. 案例目标

构建一个入门级 AI 学习工作台，完成以下闭环：

```text
选择会话 -> 读取历史消息 -> 输入问题 -> 流式接收回答
    -> 展示 Markdown/代码/图表 -> 上传学习资料
    -> 查看 Agent 步骤 -> 失败重试或停止生成
```

案例重点是前端数据流和状态边界。模型服务、文件解析和 Agent 编排可以先用 Mock API，之后替换为真实的 Node.js/NestJS 服务。

## 2. 功能清单

- 左侧会话列表：查询、切换、新建和删除。
- 中间聊天区：历史消息、输入框、流式回答和停止按钮。
- 右侧 Agent 面板：显示检索、分析、生成等步骤。
- 文件区：选择文件、检查大小和类型、展示上传状态。
- 内容渲染：Markdown、GFM 表格、代码块、复制按钮和图表。
- 异常处理：加载、空数据、401、429、网络断开和服务端错误。

## 3. 初始化项目

```bash
npx create-next-app@latest ai-learning-workbench
cd ai-learning-workbench
npm install @tanstack/react-query zustand react-markdown remark-gfm zod
```

可以后续添加：

```bash
npm install recharts shiki
```

推荐目录：

```text
src/
  app/
    page.tsx
    layout.tsx
  components/
    conversation-list.tsx
    chat-composer.tsx
    message-list.tsx
    answer-content.tsx
    agent-timeline.tsx
    file-picker.tsx
  lib/
    api.ts
    stream.ts
    schemas.ts
  stores/
    chat-store.ts
  providers/
    query-provider.tsx
```

## 4. API 契约

```text
GET    /api/conversations
GET    /api/conversations/:id/messages
POST   /api/conversations
POST   /api/chat
POST   /api/files
GET    /api/runs/:runId/events
```

`POST /api/chat` 的流式事件可以设计为：

```text
{"type":"run_started","runId":"run_123"}\n
{"type":"step_started","step":{"id":"s1","name":"检索资料"}}\n
{"type":"step_completed","stepId":"s1"}\n
{"type":"token","data":"这是"}\n
{"type":"token","data":"回答"}\n
{"type":"done","messageId":"msg_456"}\n
```

每个事件都要有 `type`。客户端对未知事件保留兼容处理，并记录诊断信息。

## 5. Zod 数据校验

```ts
import { z } from 'zod'

export const streamEventSchema = z.discriminatedUnion('type', [
  z.object({ type: z.literal('run_started'), runId: z.string() }),
  z.object({ type: z.literal('step_started'), step: z.object({
    id: z.string(), name: z.string(),
  }) }),
  z.object({ type: z.literal('step_completed'), stepId: z.string() }),
  z.object({ type: z.literal('token'), data: z.string() }),
  z.object({ type: z.literal('done'), messageId: z.string() }),
])

export type StreamEvent = z.infer<typeof streamEventSchema>
```

服务端数据经过校验后再进入 Store。错误事件可以额外加入 `code`、`message` 和 `retryable` 字段。

## 6. 流式解析器

```ts
import { streamEventSchema, type StreamEvent } from './schemas'

export async function consumeJsonLines(
  response: Response,
  onEvent: (event: StreamEvent) => void,
) {
  if (!response.body) throw new Error('响应不支持流式读取')

  const reader = response.body.getReader()
  const decoder = new TextDecoder()
  let buffer = ''

  while (true) {
    const { value, done } = await reader.read()
    buffer += decoder.decode(value ?? new Uint8Array(), { stream: !done })
    const lines = buffer.split('\n')
    buffer = lines.pop() ?? ''

    for (const line of lines) {
      if (!line.trim()) continue
      const event = streamEventSchema.parse(JSON.parse(line))
      onEvent(event)
    }

    if (done) break
  }
}
```

真实项目中还需要检查结束时的残留 `buffer`，并把解析错误转换成用户可理解的错误状态。

## 7. Store 处理事件

```ts
import { create } from 'zustand'
import type { StreamEvent } from '@/lib/schemas'

type ChatState = {
  answer: string
  runId: string | null
  status: 'idle' | 'running' | 'completed' | 'failed'
  steps: Record<string, { name: string; status: string }>
  applyEvent: (event: StreamEvent) => void
}

export const useChatStore = create<ChatState>((set) => ({
  answer: '',
  runId: null,
  status: 'idle',
  steps: {},
  applyEvent: (event) => set((state) => {
    if (event.type === 'run_started') {
      return { ...state, runId: event.runId, answer: '', status: 'running' }
    }
    if (event.type === 'step_started') {
      return { ...state, steps: { ...state.steps, [event.step.id]: {
        name: event.step.name, status: 'running',
      } } }
    }
    if (event.type === 'step_completed') {
      return { ...state, steps: { ...state.steps, [event.stepId]: {
        ...state.steps[event.stepId], status: 'completed',
      } } }
    }
    if (event.type === 'token') return { ...state, answer: state.answer + event.data }
    if (event.type === 'done') return { ...state, status: 'completed' }
    return state
  }),
}))
```

组件根据 `status` 决定显示发送、停止、重试或完成状态。历史消息仍由 TanStack Query 管理，流式草稿由 Zustand 管理。

## 8. TanStack Query 接入

```tsx
const messagesQuery = useQuery({
  queryKey: ['messages', conversationId],
  queryFn: () => fetch(`/api/conversations/${conversationId}/messages`).then((r) => r.json()),
  enabled: Boolean(conversationId),
})
```

回答完成后：

```ts
await consumeJsonLines(response, (event) => {
  useChatStore.getState().applyEvent(event)
})

queryClient.invalidateQueries({ queryKey: ['messages', conversationId] })
```

这样可以把最终已保存的消息同步回服务端缓存，避免刷新后丢失。

## 9. 文件上传流程

```ts
export async function uploadFile(file: File, signal?: AbortSignal) {
  const allowedTypes = ['application/pdf', 'text/plain']
  if (!allowedTypes.includes(file.type)) throw new Error('文件类型不支持')
  if (file.size > 10 * 1024 * 1024) throw new Error('文件不能超过 10 MB')

  const formData = new FormData()
  formData.append('file', file)
  const response = await fetch('/api/files', { method: 'POST', body: formData, signal })
  if (!response.ok) throw new Error('上传失败')
  return response.json() as Promise<{ id: string; name: string; status: string }>
}
```

界面状态可以设计为：`idle`、`validating`、`uploading`、`processing`、`ready`、`failed`。文件解析完成后，服务端再把文件加入 RAG 知识库。

## 10. 页面拆分

```tsx
export default function WorkbenchPage() {
  return (
    <main className="grid min-h-screen grid-cols-[240px_1fr_280px]">
      <ConversationList />
      <section className="flex min-w-0 flex-col">
        <MessageList />
        <ChatComposer />
      </section>
      <AgentTimeline />
    </main>
  )
}
```

移动端将三列改为抽屉或 Tab：会话列表和 Agent 步骤面板通过按钮打开，聊天区保持主要内容。

## 11. 验收清单

- 首次加载显示骨架屏或加载状态。
- 没有会话时显示空状态和新建入口。
- 发送空问题时阻止请求。
- AI 回复可以逐步显示。
- 生成中可以停止，停止后可以重新生成。
- 网络失败时保留已经收到的内容，并显示重试入口。
- 会话切换后不会串显示其他会话的流式内容。
- 文件类型和大小在客户端、服务端分别校验。
- Markdown 表格、代码块和链接可以安全渲染。
- 图表数据经过 Schema 校验后才渲染。
- WebSocket/SSE 断开后能够释放连接并更新状态。
- 键盘操作、焦点状态和移动端布局可用。

## 12. 分阶段练习

1. 只使用本地数组完成静态聊天界面。
2. 使用 TanStack Query 加载会话和历史消息。
3. 使用 Zustand 管理输入框、当前回答和生成状态。
4. 使用 `fetch` Stream 实现 Token 流式输出。
5. 加入停止、重试和错误恢复。
6. 加入 Markdown、代码块、文件上传和固定图表数据。
7. 使用 Mock Agent 事件渲染右侧步骤时间线。
8. 将 Mock API 替换为 NestJS 或 Node.js 后端。
