# React 学习建议

## 核心学习原则

### 0. 先补齐前端底座

React 的组件和 Hooks 建立在 JavaScript、浏览器和 HTTP 之上。建议先完成[阶段零：前端基础与开发底座](./stage-0-foundations.md)，再进入 React 基础阶段。

学习顺序建议为：

```text
JavaScript → 浏览器与 DOM → HTTP/API → Git
    ↓
JSX → 组件 → state/effect → 表单/列表
    ↓
路由 → 服务端状态 → TypeScript → 工程化
    ↓
测试 → 性能 → 安全 → 可访问性 → 部署与监控
```

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

## 阶段验收方法

每个阶段都应完成“学习、实现、测试、复盘”四步：

1. 用自己的话解释核心概念。
2. 完成一个有真实交互的项目。
3. 为关键逻辑添加测试并处理错误状态。
4. 记录性能、可访问性和工程改进点。

阶段三开始，每个项目至少需要具备：

- TypeScript 严格模式
- loading、empty、error 状态
- 基础权限处理
- lint、类型检查和构建命令
- 至少一组组件测试和一条 E2E 流程

推荐使用[综合实战：企业级 React 中后台](./project-practice.md)贯穿阶段三和阶段四。

## 总结

| 阶段 | 核心内容 |
|------|----------|
| 基础 | JSX、组件、useState/useEffect、表单列表 |
| 进阶 | 高阶 Hooks、React Router、性能优化、自定义 Hooks |
| 工程化 | TypeScript、Redux Toolkit、Axios 封装、组件库、规范 |
| 高级 | 渲染原理、Next.js、大型项目架构、性能调优 |

记住：**多写代码、多踩坑、多总结**，才能真正掌握 React！
