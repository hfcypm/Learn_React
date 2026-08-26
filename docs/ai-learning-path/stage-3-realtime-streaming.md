# SSE、WebSocket 与流式响应

## 1. 三种通信方式

| 方式 | 特点 | 适合场景 |
| --- | --- | --- |
| 普通 HTTP | 一次请求一次完整响应 | 会话列表、文件列表、用户设置 |
| SSE | 服务端单向持续推送，基于 HTTP | AI Token 流、Agent 进度、通知 |
| WebSocket | 客户端与服务端双向持续通信 | 协作编辑、语音状态、实时控制台 |
| `fetch` Stream | 读取 HTTP Response Body 的字节流 | 自定义 AI 流协议、需要 POST 请求的生成接口 |

## 2. 使用 fetch 读取流

```ts
export async function readAnswerStream(
  question: string,
  onChunk: (chunk: string) => void,
  signal?: AbortSignal,
) {
  const response = await fetch('/api/chat', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ question }),
    signal,
  })

  if (!response.ok || !response.body) throw new Error('生成请求失败')

  const reader = response.body.getReader()
  const decoder = new TextDecoder()

  try {
    while (true) {
      const { value, done } = await reader.read()
      if (done) break
      onChunk(decoder.decode(value, { stream: true }))
    }
  } finally {
    reader.releaseLock()
  }
}
```

生产协议需要定义事件边界。JSON 行协议示例：

```text
{"type":"token","data":"你好"}\n
{"type":"token","data":"，世界"}\n
{"type":"done","messageId":"msg_123"}\n
```

解析器需要处理半个 JSON 跨 chunk 的情况，不能假设每次 `reader.read()` 都返回一条完整消息。

## 3. SSE 客户端

GET SSE 可以使用浏览器原生 `EventSource`：

```ts
const source = new EventSource(`/api/runs/${runId}/events`)

source.addEventListener('step', (event) => {
  const step = JSON.parse(event.data)
  console.log(step.name, step.status)
})

source.addEventListener('done', () => source.close())
source.onerror = () => source.close()
```

需要携带自定义请求头或使用 POST 时，可以使用支持 fetch 的 SSE 客户端，或自行读取响应流。

## 4. WebSocket 客户端

```ts
const socket = new WebSocket('wss://example.com/api/runs')

socket.addEventListener('open', () => {
  socket.send(JSON.stringify({ type: 'subscribe', runId }))
})

socket.addEventListener('message', (event) => {
  const message = JSON.parse(event.data)
  console.log(message.type, message.data)
})

socket.addEventListener('close', () => {
  // 根据业务决定是否重连，并避免重复订阅
})
```

## 5. 取消、重试和生命周期

```ts
const controller = new AbortController()
readAnswerStream('介绍 SSE', (chunk) => {
  useChatStore.getState().appendAnswer(chunk)
}, controller.signal)

// 用户点击停止生成
controller.abort()
```

- 组件卸载时关闭 EventSource、WebSocket 或 AbortController。
- 重试前确认服务端是否支持幂等键，避免重复创建消息或重复执行 Agent 工具。
- 断线重连需要携带 `runId` 和最后事件 ID，服务端据此补发事件。
- 页面显示“生成中”时，仍要处理网络错误、服务端错误、用户取消和服务端完成四种结果。

## 6. 练习

1. 用 `fetch` 实现逐字显示回答。
2. 增加停止生成和重新生成。
3. 把 Token 事件和 Agent 步骤事件统一成事件类型。
4. 模拟网络断开，设计重连和重复事件去重。
