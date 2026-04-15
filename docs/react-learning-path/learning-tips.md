# React 学习建议

## 核心学习原则

### 1. 循序渐进

先吃透基础 Hooks 和组件，再学生态和 TypeScript。

**推荐顺序**：
```
JSX 基础 → 组件 → useState/useEffect → Props → 表单/列表
    ↓
useRef/useMemo/useCallback → useContext → useReducer
    ↓
React Router → 自定义 Hooks → 性能优化
    ↓
TypeScript → 状态管理 → 样式方案 → 工程化
```

### 2. 动手为主

每个阶段做 1-2 个实战项目：

| 阶段 | 项目练习 |
|------|----------|
| 基础阶段 | TodoList、登录页、列表页 |
| 进阶基础 | 多页面博客、购物车、权限管理后台 |
| 生态与工程化 | 企业中后台系统、即时通讯应用 |
| 高级进阶 | 性能优化项目、Next.js 全栈项目 |

### 3. 重点突破

以下是企业面试必考内容：

- **Hooks**：useEffect 依赖、useCallback/useMemo 区别
- **TypeScript**：泛型、类型推断、interface vs type
- **React Router**：路由守卫、嵌套路由、路由传参
- **状态管理**：Redux 数据流、useContext vs Redux

### 4. 持续优化

写完代码后思考三个问题：

1. **如何复用？** → 抽离自定义 Hooks、组件
2. **如何避免重复渲染？** → React.memo、useMemo、useCallback
3. **如何加类型？** → 为所有 Props/State 添加类型

## 常见问题

### Q: 类组件还是函数组件？

**A**: 优先使用函数组件。所有新代码都应使用函数组件，配合 Hooks。

### Q: 何时使用 useState vs useReducer？

**A**: 
- `useState`：简单状态（计数器、开关、输入框）
- `useReducer`：复杂状态逻辑（多个相关状态、操作有副作用）

### Q: 何时使用 Context vs Redux？

**A**:
- `Context`：简单的跨组件传值（主题、语言）
- `Redux`：复杂全局状态、异步操作、需要调试工具

### Q: useEffect 如何正确处理依赖？

**A**:
```typescript
// 正确：依赖数组完整
useEffect(() => {
  fetchData(id).then(setData);
}, [id]);

// 错误：忘记依赖导致闭包陷阱
useEffect(() => {
  fetchData(id).then(setData);
}, []); // id 丢失！
```

## 学习资源推荐

### 官方文档
- [React 官方文档](https://react.dev) - 必读
- [TypeScript 官方文档](https://www.typescriptlang.org/docs/)
- [React Router 文档](https://reactrouter.com/)

### 工具文档
- [Redux Toolkit 文档](https://redux-toolkit.js.org/)
- [TanStack Query 文档](https://tanstack.com/query/)
- [Tailwind CSS 文档](https://tailwindcss.com/)

## 总结

| 阶段 | 核心内容 |
|------|----------|
| 基础 | JSX、组件、useState/useEffect、表单列表 |
| 进阶 | 高阶 Hooks、React Router、性能优化、自定义 Hooks |
| 工程化 | TypeScript、Redux Toolkit、Axios 封装、组件库、规范 |
| 高级 | 渲染原理、Next.js、大型项目架构、性能调优 |

记住：**多写代码、多踩坑、多总结**，才能真正掌握 React！
