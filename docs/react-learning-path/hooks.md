# React Hooks 核心概念与使用指南

**目标**：理解 Hooks 的定义、调用规则、使用边界和常见选择方式，并能够编写可复用的自定义 Hook。

## 先建立直觉：用生活类比理解 React 为什么需要 Hook

直接读 API 定义容易卡在术语上。先用一个类比把"渲染、状态、副作用"讲清楚，再看定义就会顺很多。

### 组件函数就像一个菜谱

函数组件本质是普通函数，每次"渲染"就是按菜谱重新做一遍菜：

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return <p>你点了 {count} 次</p>;
}
```

`Counter` 每次执行都会重新跑一遍函数体。Props 是菜谱上写明的食材清单，state 是你在做菜过程中"需要记住的进度"。

关键点：**函数每次执行完，函数里的局部变量就消失了**。普通变量在函数结束后不复存在：

```jsx
function Counter() {
  let count = 0; // 这个 count 每次渲染都从 0 开始
  count += 1;    // 下次渲染又变回 0，记不住
  return <p>你点了 {count} 次</p>;
}
```

这就是"组件会失忆"。`useState` 是让 React 帮你把值存进"保险箱"，每次渲染时取出来还给你：

```jsx
function Counter() {
  const [count, setCount] = useState(0); // React 记住了 count
  return <p>你点了 {count} 次</p>;
}
```

### 渲染就像重新拍一张照片

状态一变，React 就重新执行组件函数，算出一版新的 UI（新照片），再对比新旧照片只更新变化的部分。所以：

- **状态（state）** = 组件需要记住、且变化后要重画界面的数据
- **渲染（render）** = React 重新执行组件函数，得到一版新 UI
- **setCount(新值)** = 告诉 React "把保险箱里的值换成这个，然后重拍一张照片"

### 渲染之后才做的动作，放 useEffect

拍照之后，你还需要一些"画外动作"：订阅事件、发请求、设定时器。这些动作不属于画面本身，放 `useEffect`：

```jsx
useEffect(() => {
  const timer = setInterval(() => setNow(new Date()), 1000);
  return () => clearInterval(timer); // 出门前把定时器关掉
}, []);
```

类比：`useEffect` 是"照片拍完后做的事"，所以它不会卡住画面；`return` 里放"收拾现场"（清理）的逻辑。

依赖数组是"配料表"——`[a, b]` 的意思是"只有 a 或 b 变了，才重新做这个动作"：

```jsx
// 空数组：只做一次
useEffect(() => fetchData(), []);

// 有依赖：依赖变化才做
useEffect(() => fetchData(id), [id]);

// 不写数组：每次渲染都做（默认不推荐）
useEffect(() => console.log('每次渲染都跑'));
```

### 其余 Hook 一句话类比

| Hook | 一句话类比 |
|---|---|
| `useRef` | 便签纸：随手记，改了不会重拍照片，适合存定时器 ID、DOM 节点 |
| `useMemo` | 速算表：算一遍存起来，食材（依赖）没变就用旧答案 |
| `useCallback` | 同一张名片：保证传给子组件的是同一个函数引用 |
| `useContext` | 公告栏：不用挨个传话，大家看同一个公告栏 |
| `useReducer` | 升级版记账本：把复杂的"改状态"规则集中到一张表里 |

### 术语大白话速查

- **渲染**：React 重新执行组件函数，算出一版新的 UI
- **状态**：组件记住、变化后要重画界面的数据
- **副作用**：渲染之外、影响外部世界的动作（请求、订阅、定时器、改标题）
- **闭包**：函数"出生"时记住了当时的环境变量；这既是 Hook 依赖数组的基础，也是"读取旧值"问题的来源
- **批量更新**：把同一次事件里的多次 `setState` 攒到一起，只重画一次
- **虚拟 DOM**：React 先在内存里算出一版 UI，再对比找出真正要改的 DOM 节点

理解这些之后，再读下面的正式定义和调用规则，会顺畅得多。

## Hooks 总体概念

Hook 是 React 提供的特殊函数，用于让函数组件接入 React 的状态、生命周期关联能力、Context、缓存和其他运行时能力。Hook 名称通常以 `use` 开头，例如 `useState`、`useEffect` 和 `useContext`。

Hook 的核心价值是把组件逻辑按“状态、同步、副作用、复用能力”组织起来。它让相关逻辑可以放在一起，也让多个组件能够通过自定义 Hook 复用同一套行为。

Hook 有三个重要认识：

- Hook 调用由 React 在组件渲染过程中管理，调用顺序会影响状态与副作用的对应关系。
- Hook 返回值可能是状态值、更新函数、引用对象、上下文值或缓存结果。
- Hook 解决逻辑组织问题，组件仍然需要清晰的 Props、状态边界、错误处理和测试。

## Hooks 规则

### 规则一：只在函数组件或自定义 Hook 顶层调用

```jsx
function Profile({ enabled }) {
  const [name, setName] = useState('');

  if (!enabled) {
    return null;
  }

  return <input value={name} onChange={event => setName(event.target.value)} />;
}
```

Hook 调用位于条件判断之前，因此每次渲染的调用顺序保持一致。以下写法会导致调用顺序变化：

```jsx
function Profile({ enabled }) {
  if (enabled) {
    const [name] = useState('');
    return <span>{name}</span>;
  }

  return null;
}
```

### 规则二：不要在循环、条件和嵌套函数中调用

```jsx
function Users({ users }) {
  const [selectedId, setSelectedId] = useState(null);

  return users.map(user => (
    <button key={user.id} onClick={() => setSelectedId(user.id)}>
      {user.name}
    </button>
  ));
}
```

列表中的每一项需要独立状态时，应拆分为 `UserRow` 子组件，让每个子组件在自己的顶层调用 Hook。

### 规则三：自定义 Hook 也必须以 `use` 开头

命名规则让 ESLint 和开发者识别可复用的 Hook 逻辑，并帮助检查 Hook 规则。

## Hook 选择速查

| 需求 | 首选 Hook | 核心问题 |
|---|---|---|
| 保存会影响 UI 的值 | `useState` | 状态变化后是否需要重新渲染 |
| 管理多个相关状态和动作 | `useReducer` | 状态转换是否需要集中建模 |
| 与外部系统同步 | `useEffect` | 是否需要清理连接、订阅或定时器 |
| 保存 DOM 或可变引用 | `useRef` | 更新引用是否需要触发渲染 |
| 跨层级共享稳定数据 | `useContext` | 是否真的需要跨组件读取 |
| 缓存昂贵计算结果 | `useMemo` | 是否有测量证据证明计算成本高 |
| 稳定传递给子组件的函数 | `useCallback` | 子组件是否依赖引用稳定性 |
| 复用一组状态与行为 | 自定义 Hook | 是否存在可复用的逻辑边界 |

## Hook 使用决策

写 Hook 前先回答：

1. 这个值是否属于 UI 状态？属于 UI 状态时使用 `useState` 或 `useReducer`。
2. 这个逻辑是否与浏览器、网络、订阅或定时器等外部系统同步？是时考虑 `useEffect`。
3. 这个值变化后是否需要重新渲染？不需要时考虑 `useRef`。
4. 这个计算是否存在实际性能瓶颈？有 Profile 数据时再使用 `useMemo`。
5. 这段逻辑是否需要多个组件复用？需要时抽取自定义 Hook。

## 常见错误

- 把 `useEffect` 当作所有业务逻辑的默认入口。
- 用 `useEffect` 同步本可直接计算的派生值。
- 依赖数组遗漏外部变量，造成闭包读取旧值。
- 用 `useRef` 保存需要展示在页面上的状态，导致界面不更新。
- 为所有函数和计算都添加 `useCallback` 或 `useMemo`，增加维护成本。
- 在一个 Hook 中同时承担请求、表单、订阅和多个无关职责。
- 自定义 Hook 返回过多内部细节，导致调用方与实现耦合。

## 自定义 Hook

自定义 Hook 是一个以 `use` 开头的函数，可以调用其他 Hook，并返回调用方需要的状态、动作和结果。它复用的是逻辑，不会共享同一个状态实例；每个组件调用自定义 Hook 时，都会拥有自己的 Hook 状态。

```jsx
import { useEffect, useState } from 'react';

function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return isOnline;
}
```

自定义 Hook 的设计要求：

- 参数表达外部输入，返回值表达调用方需要的能力。
- 内部负责订阅、清理和状态转换，调用方负责页面展示。
- 只暴露稳定且必要的 API。
- 对加载、成功、失败和取消状态建立明确模型。
- 使用独立测试验证边界行为和清理逻辑。
