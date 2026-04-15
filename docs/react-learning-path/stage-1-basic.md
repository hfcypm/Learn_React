# 阶段一：React 基础阶段（入门必学）

**目标**：能独立开发简单页面、理解 React 核心思想

## 必须掌握的技能

### 1. 环境与基础工具

- [ ] 搭建 React 开发环境（Vite / Create React App）
- [ ] 熟悉 npm/yarn/pnpm 包管理器
- [ ] 理解 JSX 语法（HTML 与 JS 混合写法、规则、注意事项）

### 2. 组件基础

- [ ] 函数组件（优先掌握）
- [ ] 类组件（了解）
- [ ] 组件拆分、复用、嵌套、组合
- [ ] Props 传值
- [ ] Props 校验（PropTypes）

### 3. 状态管理（基础）

- [ ] `useState`：组件内部状态管理
- [ ] `useEffect`：副作用处理（数据请求、定时器、事件监听）
- [ ] 理解 React 渲染机制
- [ ] 理解状态更新规则

### 4. 事件处理

- [ ] React 合成事件
- [ ] 事件绑定、传参
- [ ] 阻止默认行为、阻止冒泡

### 5. 列表与条件渲染

- [ ] `map` 渲染列表
- [ ] `key` 属性的作用
- [ ] 三元运算条件渲染
- [ ] `&&` 逻辑实现条件渲染

### 6. 表单处理

- [ ] 受控组件（输入框、单选、多选、下拉框）
- [ ] 非受控组件（useRef 获取 DOM 值）

## 了解即可

- 类组件的生命周期概念
- create-react-app vs Vite 的区别

## 阶段成果

能开发以下应用：
- 登录页
- 列表页
- 简单表单
- TodoList

## 练习项目建议

1. **TodoList**：实现添加、删除、完成待办事项
2. **登录页**：实现用户名密码验证、表单提交
3. **列表页**：展示数据列表，支持筛选和搜索

## 知识点速查

| 概念 | 核心要点 |
|------|----------|
| JSX | 1. 只能返回一个根元素 2. className 代替 class 3. 表达式用 {} 包裹 |
| useState | 1. 接收初始值 2. 返回 [state, setState] 3. 更新是异步的 |
| useEffect | 1. 第二个参数控制执行时机 2. [] 表示仅执行一次 3. 返回函数用于清理 |
| key | 1. 列表中唯一标识 2. 帮助 React 高效更新 3. 不推荐使用索引 |

## 经典学习案例

### 案例 1：计数器组件

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>计数：{count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
      <button onClick={() => setCount(count - 1)}>-1</button>
      <button onClick={() => setCount(0)}>重置</button>
    </div>
  );
}
```

### 案例 2：条件渲染与列表

```jsx
function UserList() {
  const [users, setUsers] = useState([
    { id: 1, name: '张三', age: 25 },
    { id: 2, name: '李四', age: 30 },
  ]);
  const [show vip, setShowVip] = useState(false);

  const filteredUsers = showVip ? users.filter(u => u.age > 28) : users;

  return (
    <div>
      <label>
        <input
          type="checkbox"
          checked={showVip}
          onChange={(e) => setShowVip(e.target.checked)}
        />
        只看 VIP 用户
      </label>

      <ul>
        {filteredUsers.map(user => (
          <li key={user.id}>
            {user.name} - {user.age}岁
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### 案例 3：受控表单组件

```jsx
function LoginForm() {
  const [form, setForm] = useState({ username: '', password: '' });
  const [errors, setErrors] = useState({});

  const handleChange = (e) => {
    const { name, value } = e.target;
    setForm(prev => ({ ...prev, [name]: value }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    const newErrors = {};

    if (!form.username) newErrors.username = '用户名不能为空';
    if (!form.password) newErrors.password = '密码不能为空';
    if (form.password.length < 6) newErrors.password = '密码至少6位';

    setErrors(newErrors);
    if (Object.keys(newErrors).length === 0) {
      console.log('提交登录：', form);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          name="username"
          value={form.username}
          onChange={handleChange}
          placeholder="用户名"
        />
        {errors.username && <span>{errors.username}</span>}
      </div>
      <div>
        <input
          name="password"
          type="password"
          value={form.password}
          onChange={handleChange}
          placeholder="密码"
        />
        {errors.password && <span>{errors.password}</span>}
      </div>
      <button type="submit">登录</button>
    </form>
  );
}
```

### 案例 4：副作用与数据请求

```jsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;

    async function fetchUser() {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) throw new Error('获取用户失败');
        const data = await response.json();
        if (!cancelled) setUser(data);
      } catch (err) {
        if (!cancelled) setError(err.message);
      } finally {
        if (!cancelled) setLoading(false);
      }
    }

    fetchUser();

    return () => {
      cancelled = true;
    };
  }, [userId]);

  if (loading) return <div>加载中...</div>;
  if (error) return <div>错误：{error}</div>;
  if (!user) return null;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.bio}</p>
    </div>
  );
}
```

### 案例 5：组件组合与 Props

```jsx
function Card({ title, children }) {
  return (
    <div className="card">
      <h3>{title}</h3>
      <div>{children}</div>
    </div>
  );
}

function Avatar({ src, alt, size = 'medium' }) {
  return (
    <img
      src={src}
      alt={alt}
      className={`avatar avatar-${size}`}
      style={{ width: size === 'small' ? 32 : 64 }}
    />
  );
}

function UserCard({ user }) {
  return (
    <Card title="用户信息">
      <Avatar src={user.avatar} alt={user.name} />
      <p>姓名：{user.name}</p>
      <p>邮箱：{user.email}</p>
    </Card>
  );
}
```

## 下一阶段预告

完成基础阶段后，将进入 [React 进阶基础](./stage-2-advanced-basic.md)，你将学习：
- 高阶 Hooks（useMemo、useCallback、useReducer）
- 组件通信全方案
- React Router 路由管理
- 性能优化基础
