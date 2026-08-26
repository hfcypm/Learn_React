# TanStack Query：服务端数据

## 1. 解决什么问题

TanStack Query 管理来自服务器的数据。它负责请求状态、缓存、去重、重新获取、失效和分页，让组件专注于展示数据。

服务端数据包括：用户资料、会话列表、消息记录、知识库文件列表和 Agent 执行记录。输入框内容、弹窗开关和当前 Tab 属于客户端状态，应交给 Zustand 或组件自身管理。

## 2. 安装与 Provider

```bash
npm install @tanstack/react-query
```

```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'

const queryClient = new QueryClient({
  defaultOptions: {
    queries: { staleTime: 30_000, retry: 1 },
  },
})

export function AppProviders({ children }: { children: React.ReactNode }) {
  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  )
}
```

## 3. 查询数据

```tsx
import { useQuery } from '@tanstack/react-query'

type Conversation = { id: string; title: string; updatedAt: string }

async function fetchConversations(): Promise<Conversation[]> {
  const response = await fetch('/api/conversations')
  if (!response.ok) throw new Error('加载会话失败')
  return response.json()
}

export function ConversationList() {
  const query = useQuery({
    queryKey: ['conversations'],
    queryFn: fetchConversations,
  })

  if (query.isPending) return <p>加载中...</p>
  if (query.isError) return <button onClick={() => query.refetch()}>重新加载</button>

  return <ul>{query.data.map((item) => <li key={item.id}>{item.title}</li>)}</ul>
}
```

## 4. 创建和更新

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query'

export function CreateConversationButton() {
  const client = useQueryClient()
  const mutation = useMutation({
    mutationFn: async (title: string) => {
      const response = await fetch('/api/conversations', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ title }),
      })
      if (!response.ok) throw new Error('创建失败')
      return response.json()
    },
    onSuccess: () => client.invalidateQueries({ queryKey: ['conversations'] }),
  })

  return <button onClick={() => mutation.mutate('新会话')}>创建会话</button>
}
```

## 5. AI 应用中的关键模式

- `queryKey` 必须包含影响结果的参数，例如 `['messages', conversationId]`。
- 切换会话时清理或切换对应缓存，避免显示上一会话内容。
- 发送消息后可以先更新缓存，再等待服务端确认，这叫 optimistic update。
- 流式消息通常由 Zustand 管理实时草稿，流结束后再通过 Query 重新同步服务端数据。
- 401、403、429 和 5xx 应显示不同的用户提示。
- `staleTime` 决定数据多久视为新鲜，`gcTime` 决定未使用缓存保留多久。

## 6. 练习

1. 为会话列表增加搜索参数。
2. 为消息列表增加分页或无限滚动。
3. 实现删除会话后的缓存失效。
4. 模拟接口失败，完成重试按钮和错误提示。
