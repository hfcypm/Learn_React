# 阶段二：表单、浮层与交互

**目标**：掌握 TextInput 受控 API、校验状态、必填标记，以及对话框 / 浮层的焦点与关闭行为，完成“新建订阅”闭环。

> 适用范围：Astryx Beta + React 19+ + TypeScript
> 需要的工具：Node.js、npm、已完成阶段一的工程
> 验证命令：`npx @astryxdesign/cli component TextInput`、`npx @astryxdesign/cli search dialog`
> 最后复核日期：2026-11-08

**前置知识**：阶段一的布局、Button、Card、Theme。

## 1. TextInput 受控模式

```tsx
import {useState} from 'react';
import {TextInput, type TextInputStatus} from '@astryxdesign/core/TextInput';
import {VStack} from '@astryxdesign/core/VStack';
import {Button} from '@astryxdesign/core/Button';

export function EmailField() {
  const [email, setEmail] = useState('');
  const status: TextInputStatus | undefined = email.includes('@')
    ? undefined
    : email.length === 0
      ? undefined
      : {type: 'error', message: '请输入有效邮箱'};

  return (
    <VStack gap={3}>
      <TextInput
        label="邮箱"
        description="用于账单通知"
        value={email}
        onChange={setEmail}
        status={status}
        isRequired
        hasClear
      />
      <Button label="保存" variant="primary" isDisabled={Boolean(status)} />
    </VStack>
  );
}
```

要点：

- `label` / `description` 是组件的一等 props，不要自己再叠一个裸 `<label>`。
- `isRequired` 与 `isOptional` 二选一。
- 还有 `isDisabled`、`isReadOnly`、`isLoading`；没有 `busy`。
- `status.type` 为 `error` 时提供 `message`，给屏幕阅读器与可见提示同一份文案。

查当前完整 props：

```bash
npx @astryxdesign/cli component TextInput
```

## 2. 输入类组件的样式入口

v0.7 起 `TextInput`、`TextArea`、`NumberInput`、`FileInput` **不再接受根级** `xstyle` / `className` / `style`。

| 属性 | 作用 |
|---|---|
| `containerXstyle` / `containerClassName` | 字段容器（含 label 的那一层） |
| `controlXstyle` / `controlClassName` | 真正的输入控件 |

```tsx
<TextInput
  label="项目名"
  value={name}
  onChange={setName}
  controlClassName="max-w-sm"
/>
```

旧代码升级：

```bash
npx @astryxdesign/cli upgrade --from <旧 Core 版本> --apply
```

codemod 会改 JSX 上的根级样式属性；经 wrapper 再 spread 的不会改，需手改。

## 3. 表单布局

多个字段用 `VStack` 或官方 Form 布局组件（名称以 CLI 为准）：

```bash
npx @astryxdesign/cli search form
npx @astryxdesign/cli component --list
```

提交时：

1. 每个字段自己算 `status`。
2. 有任一 error 则主按钮 `isDisabled` 或提交时聚焦第一个错误字段。
3. 服务端错误写回对应字段的 `status`，不要只 `alert`。

## 4. 对话框与浮层

新建 / 编辑用 Dialog（或官方 Overlay / Modal 名，以 CLI 打印的入口为准）：

```bash
npx @astryxdesign/cli search dialog
npx @astryxdesign/cli docs principles --dense
```

实现时守这几条：

- 触发器是真实 `Button`，有 `label`。
- 打开后焦点进入对话框，关闭后回到触发器。
- Esc 与点遮罩关闭；Astryx 从 0.5 起用共享 dismissal 栈，多层浮层不要自己再绑一套 Esc。
- 同一时间只开一层确认框。删除用 `variant="destructive"` 的确认按钮。

最小结构（组件名以你安装版本的文档为准）：

```tsx
<Button label="新建订阅" variant="primary" onClick={() => setOpen(true)} />
{/* Dialog：title、描述、表单、取消 / 保存 */}
```

移动端注意 v0.7 的 bottom sheet / 更清晰的输入行为，真机或窄视口走一遍。

## 5. 可访问性基线

- 可见文本与 `label` 一致，图标按钮必须有无障碍名。
- 错误不只靠颜色，要有 `status.message`。
- 不要用 `div onClick` 冒充按钮。
- 键盘：Tab 顺序、Enter 提交、Esc 关层。

```bash
npx @astryxdesign/cli docs principles
```

## 阶段验收

- 能写出受控 TextInput，含 label、必填、error status。
- 能区分 `container*` 与 `control*` 样式入口。
- 能完成对话框内新建表单：打开、校验、保存、关闭、焦点返回。
- 能说明多层浮层为什么共用 dismissal 栈。

## 动手任务

1. 做“新建订阅”表单：名称（必填）、客户邮箱、套餐备注。
2. 名称为空时展示 error status，主按钮在无效时禁用。
3. 把表单放进对话框，取消关闭且不提交。
4. 给名称字段加 `hasClear`。
5. 仅用键盘走完打开 -> 填写 -> 保存。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 给 TextInput 传 `className` 类型报错 | 改 `controlClassName` / `containerClassName` |
| 校验信息不出现 | 是否传了 `status={{type:'error', message}}` |
| 对话框打开后焦点在背后页面 | 是否用官方浮层，而不是自写 `position:fixed` |
| Esc 关掉了错误的一层 | 是否多层各绑各的 keydown |
| `isRequired` 与 `isOptional` 冲突 | 只保留一个 |

## 进入下一阶段的条件

新建流程可键盘完成，错误信息可见且与输入关联。此时进入 [阶段三：数据展示与 AI 工作流](./stage-3-data-and-agents.md)。
