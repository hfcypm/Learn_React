# 文件、Markdown、代码和图表渲染

## 1. 文件上传

客户端需要在选择文件时做第一层检查，服务端必须再次验证。

```tsx
function FilePicker({ onSelect }: { onSelect: (file: File) => void }) {
  function handleChange(event: React.ChangeEvent<HTMLInputElement>) {
    const file = event.target.files?.[0]
    if (!file) return
    if (file.size > 10 * 1024 * 1024) throw new Error('文件不能超过 10 MB')
    if (!['application/pdf', 'text/plain'].includes(file.type)) {
      throw new Error('当前只支持 PDF 和 TXT')
    }
    onSelect(file)
  }

  return <input type="file" accept="application/pdf,text/plain" onChange={handleChange} />
}
```

上传使用 `FormData`：

```ts
const formData = new FormData()
formData.append('file', file)
await fetch('/api/files', { method: 'POST', body: formData })
```

大文件应使用分片上传、断点续传和服务端对象存储。上传进度可以使用 `XMLHttpRequest` 或专门的上传客户端获取。

## 2. Markdown 渲染

```bash
npm install react-markdown remark-gfm
```

```tsx
import ReactMarkdown from 'react-markdown'
import remarkGfm from 'remark-gfm'

export function Answer({ content }: { content: string }) {
  return <ReactMarkdown remarkPlugins={[remarkGfm]}>{content}</ReactMarkdown>
}
```

Markdown 来自模型或用户输入时，要根据渲染库的默认安全策略进行配置。HTML、链接、图片和脚本内容需要经过安全审查；业务页面应限制允许的协议和外部资源。

## 3. 代码高亮

代码块渲染需要识别语言、处理未知语言，并避免把代码当成 HTML 执行。可以选择 Shiki 或 `react-syntax-highlighter`。

```tsx
type CodeBlockProps = { code: string; language: string }

export function CodeBlock({ code, language }: CodeBlockProps) {
  return (
    <pre data-language={language}>
      <code>{code}</code>
    </pre>
  )
}
```

先实现纯文本代码块，再加入高亮、复制、换行和下载功能，便于定位问题。

## 4. 图表渲染

AI 返回图表数据时，优先约束为结构化 JSON：

```ts
type ChartData = {
  title: string
  labels: string[]
  series: Array<{ name: string; values: number[] }>
}
```

前端需要验证 `labels` 和每个 `values` 长度一致，再传给 Recharts 或 ECharts。模型输出的图表配置属于不可信输入，禁止直接执行任意 JavaScript 或直接注入未经检查的配置。

## 5. 富文本与交互内容

- 表格需要支持横向滚动和小屏幕布局。
- 长回答使用虚拟列表或分段渲染，降低大文本重新渲染成本。
- 图片、附件和代码块提供复制、下载、预览状态。
- 生成中的 Markdown 可能处于不完整语法状态，渲染器需要容忍半截代码块和未闭合列表。
- 内容区域支持键盘操作、焦点管理和屏幕阅读器。

## 6. 练习

1. 实现 PDF/TXT 文件选择和上传状态。
2. 渲染包含表格、链接和代码块的 AI 回复。
3. 为代码块增加复制按钮。
4. 使用固定 JSON 数据渲染一张统计图。
