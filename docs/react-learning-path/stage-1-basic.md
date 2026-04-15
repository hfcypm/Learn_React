# 阶段一：React 基础阶段（入门必学）

**目标**：能独立开发简单页面、理解 React 核心思想

## 必须掌握的技能

### 1. 环境与基础工具

- [ ] 搭建 React 开发环境（Vite / Create React App）
- [ ] 熟悉 npm/yarn/pnpm 包管理器
- [ ] 理解 JSX 语法（HTML 与 JS 混合写法、规则、注意事项）

**经典案例：Vite + React 项目初始化**

```bash
# 使用 Vite 创建 React 项目
npm create vite@latest my-react-app -- --template react

# 进入项目目录
cd my-react-app

# 安装依赖
npm install

# 启动开发服务器
npm run dev
```

```jsx
// src/App.jsx - JSX 语法示例
function App() {
  const name = 'React';

  return (
    <div className="app">
      <h1>Hello, {name}!</h1>
      <p>JSX 规则：</p>
      <ul>
        <li>使用 className 而不是 class</li>
        <li>使用 {} 嵌入表达式</li>
        <li>必须有一个根元素</li>
      </ul>
    </div>
  );
}

export default App;
```

---

### 2. 组件基础

- [ ] 函数组件（优先掌握）
- [ ] 类组件（了解）
- [ ] 组件拆分、复用、嵌套、组合
- [ ] Props 传值
- [ ] Props 校验（PropTypes）

**经典案例：函数组件与 Props**

```jsx
// Button.jsx - 可复用的按钮组件
function Button({ children, variant = 'primary', onClick, disabled = false }) {
  const className = `btn btn-${variant}`;

  return (
    <button className={className} onClick={onClick} disabled={disabled}>
      {children}
    </button>
  );
}

// Card.jsx - 组合组件
function Card({ title, children }) {
  return (
    <div className="card">
      <div className="card-header">
        <h3>{title}</h3>
      </div>
      <div className="card-body">
        {children}
      </div>
    </div>
  );
}

// UserCard.jsx - 使用 Props 传值
function UserCard({ user }) {
  return (
    <Card title="用户信息">
      <p>姓名：{user.name}</p>
      <p>邮箱：{user.email}</p>
      <Button variant="secondary" onClick={() => alert(user.email)}>
        发送邮件
      </Button>
    </Card>
  );
}

// App.jsx - 组合使用
function App() {
  const user = { name: '张三', email: 'zhang@example.com' };

  return (
    <UserCard user={user} />
  );
}
```

**经典案例：类组件（了解）**

```jsx
// ClassComponent.jsx - 类组件示例
import { Component } from 'react';

class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = {
      count: props.initialCount || 0
    };
    this.handleIncrement = this.handleIncrement.bind(this);
  }

  handleIncrement() {
    this.setState({ count: this.state.count + 1 });
  }

  render() {
    return (
      <div>
        <p>计数：{this.state.count}</p>
        <button onClick={this.handleIncrement}>+1</button>
      </div>
    );
  }
}
```

---

### 3. 状态管理（基础）

- [ ] `useState`：组件内部状态管理
- [ ] `useEffect`：副作用处理（数据请求、定时器、事件监听）
- [ ] 理解 React 渲染机制
- [ ] 理解状态更新规则

**经典案例：useState 状态管理**

```jsx
// Counter.jsx - 基础 useState
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

export default Counter;
```

```jsx
// FormWithState.jsx - 多个状态
import { useState } from 'react';

function FormWithState() {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const [isLoading, setIsLoading] = useState(false);

  const handleSubmit = (e) => {
    e.preventDefault();
    setIsLoading(true);
    console.log({ username, password });
    setTimeout(() => setIsLoading(false), 1000);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={username}
        onChange={(e) => setUsername(e.target.value)}
        placeholder="用户名"
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
        placeholder="密码"
      />
      <button type="submit" disabled={isLoading}>
        {isLoading ? '登录中...' : '登录'}
      </button>
    </form>
  );
}
```

**经典案例：useEffect 副作用处理**

```jsx
// DataFetcher.jsx - 数据请求
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

```jsx
// Timer.jsx - 定时器
import { useState, useEffect } from 'react';

function Timer() {
  const [seconds, setSeconds] = useState(0);
  const [isRunning, setIsRunning] = useState(true);

  useEffect(() => {
    let interval = null;

    if (isRunning) {
      interval = setInterval(() => {
        setSeconds(prev => prev + 1);
      }, 1000);
    }

    return () => {
      if (interval) clearInterval(interval);
    };
  }, [isRunning]);

  const formatTime = (totalSeconds) => {
    const mins = Math.floor(totalSeconds / 60);
    const secs = totalSeconds % 60;
    return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
  };

  return (
    <div>
      <p>计时器：{formatTime(seconds)}</p>
      <button onClick={() => setIsRunning(!isRunning)}>
        {isRunning ? '暂停' : '继续'}
      </button>
      <button onClick={() => setSeconds(0)}>重置</button>
    </div>
  );
}
```

---

### 4. 事件处理

- [ ] React 合成事件
- [ ] 事件绑定、传参
- [ ] 阻止默认行为、阻止冒泡

**经典案例：事件处理**

```jsx
// EventExamples.jsx
function EventExamples() {
  const handleClick = (e) => {
    console.log('按钮点击', e);
  };

  const handleClickWithParam = (name, e) => {
    console.log(`你好，${name}！`, e);
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('表单提交');
  };

  const handleBubble = (label, e) => {
    console.log(`${label} 被点击`);
  };

  const handleStopBubble = (e) => {
    e.stopPropagation();
    console.log('停止冒泡');
  };

  return (
    <div onClick={() => handleBubble('父容器', e)}>
      <button onClick={handleClick}>基本点击</button>

      <button onClick={(e) => handleClickWithParam('张三', e)}>
        带参数点击
      </button>

      <form onSubmit={handleSubmit}>
        <button type="submit">提交表单（阻止默认行为）</button>
      </form>

      <div onClick={() => handleBubble('外层', event)}>
        <button onClick={handleStopBubble}>停止冒泡</button>
      </div>
    </div>
  );
}
```

---

### 5. 列表与条件渲染

- [ ] `map` 渲染列表
- [ ] `key` 属性的作用
- [ ] 三元运算条件渲染
- [ ] `&&` 逻辑实现条件渲染

**经典案例：列表渲染**

```jsx
// UserList.jsx
function UserList() {
  const users = [
    { id: 1, name: '张三', age: 25, email: 'zhang@example.com' },
    { id: 2, name: '李四', age: 30, email: 'li@example.com' },
    { id: 3, name: '王五', age: 28, email: 'wang@example.com' },
  ];

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          <strong>{user.name}</strong> - {user.age}岁 - {user.email}
        </li>
      ))}
    </ul>
  );
}
```

**经典案例：条件渲染**

```jsx
// ConditionalRendering.jsx
function ConditionalRendering() {
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  const [user, setUser] = useState({ name: '张三', role: 'admin' });

  return (
    <div>
      {/* 三元运算 */}
      {isLoggedIn ? (
        <p>欢迎，{user.name}！</p>
      ) : (
        <p>请登录</p>
      )}

      {/* && 逻辑 */}
      {user.role === 'admin' && (
        <button>管理后台</button>
      )}

      {/* 多种状态 */}
      {user.role === 'vip' ? (
        <span>VIP 用户</span>
      ) : user.role === 'admin' ? (
        <span>管理员</span>
      ) : (
        <span>普通用户</span>
      )}

      <button onClick={() => setIsLoggedIn(!isLoggedIn)}>
        {isLoggedIn ? '退出' : '登录'}
      </button>
    </div>
  );
}
```

**经典案例：综合列表与条件渲染**

```jsx
// FilterableUserList.jsx
function FilterableUserList() {
  const [users, setUsers] = useState([
    { id: 1, name: '张三', age: 25, vip: true },
    { id: 2, name: '李四', age: 30, vip: false },
    { id: 3, name: '王五', age: 35, vip: true },
    { id: 4, name: '赵六', age: 28, vip: false },
  ]);
  const [showVipOnly, setShowVipOnly] = useState(false);
  const [searchName, setSearchName] = useState('');

  const filteredUsers = users.filter(user => {
    const matchVip = showVipOnly ? user.vip : true;
    const matchName = user.name.includes(searchName);
    return matchVip && matchName;
  });

  return (
    <div>
      <label>
        <input
          type="checkbox"
          checked={showVipOnly}
          onChange={(e) => setShowVipOnly(e.target.checked)}
        />
        只看 VIP 用户
      </label>

      <input
        type="text"
        value={searchName}
        onChange={(e) => setSearchName(e.target.value)}
        placeholder="搜索用户名..."
      />

      <p>共 {filteredUsers.length} 位用户</p>

      <ul>
        {filteredUsers.map(user => (
          <li key={user.id}>
            {user.vip && <span>[VIP]</span>}
            {user.name} - {user.age}岁
          </li>
        ))}
      </ul>

      {filteredUsers.length === 0 && (
        <p>没有找到匹配的用户</p>
      )}
    </div>
  );
}
```

---

### 6. 表单处理

- [ ] 受控组件（输入框、单选、多选、下拉框）
- [ ] 非受控组件（useRef 获取 DOM 值）

**经典案例：受控组件**

```jsx
// ControlledForm.jsx - 受控表单
function ControlledForm() {
  const [form, setForm] = useState({
    username: '',
    password: '',
    gender: '',
    hobbies: [],
    country: ''
  });
  const [errors, setErrors] = useState({});

  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    setForm(prev => ({
      ...prev,
      [name]: type === 'checkbox'
        ? checked
          ? [...prev.hobbies, value]
          : prev.hobbies.filter(h => h !== value)
        : value
    }));
  };

  const validate = () => {
    const newErrors = {};
    if (!form.username) newErrors.username = '用户名不能为空';
    if (!form.password) newErrors.password = '密码不能为空';
    if (form.password.length < 6) newErrors.password = '密码至少6位';
    if (!form.gender) newErrors.gender = '请选择性别';
    if (form.hobbies.length === 0) newErrors.hobbies = '请至少选择一个爱好';
    if (!form.country) newErrors.country = '请选择国家';
    return newErrors;
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    const newErrors = validate();
    setErrors(newErrors);
    if (Object.keys(newErrors).length === 0) {
      console.log('表单数据：', form);
      alert('提交成功！');
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
        {errors.username && <span style={{ color: 'red' }}>{errors.username}</span>}
      </div>

      <div>
        <input
          name="password"
          type="password"
          value={form.password}
          onChange={handleChange}
          placeholder="密码"
        />
        {errors.password && <span style={{ color: 'red' }}>{errors.password}</span>}
      </div>

      <div>
        <label>
          <input
            type="radio"
            name="gender"
            value="male"
            checked={form.gender === 'male'}
            onChange={handleChange}
          />
          男
        </label>
        <label>
          <input
            type="radio"
            name="gender"
            value="female"
            checked={form.gender === 'female'}
            onChange={handleChange}
          />
          女
        </label>
        {errors.gender && <span style={{ color: 'red' }}>{errors.gender}</span>}
      </div>

      <div>
        <label>
          <input
            type="checkbox"
            name="hobbies"
            value="reading"
            checked={form.hobbies.includes('reading')}
            onChange={handleChange}
          />
          阅读
        </label>
        <label>
          <input
            type="checkbox"
            name="hobbies"
            value="sports"
            checked={form.hobbies.includes('sports')}
            onChange={handleChange}
          />
          运动
        </label>
        <label>
          <input
            type="checkbox"
            name="hobbies"
            value="music"
            checked={form.hobbies.includes('music')}
            onChange={handleChange}
          />
          音乐
        </label>
        {errors.hobbies && <span style={{ color: 'red' }}>{errors.hobbies}</span>}
      </div>

      <div>
        <select
          name="country"
          value={form.country}
          onChange={handleChange}
        >
          <option value="">请选择国家</option>
          <option value="cn">中国</option>
          <option value="us">美国</option>
          <option value="jp">日本</option>
        </select>
        {errors.country && <span style={{ color: 'red' }}>{errors.country}</span>}
      </div>

      <button type="submit">提交</button>
    </form>
  );
}
```

**经典案例：非受控组件**

```jsx
// UncontrolledForm.jsx - 非受控组件
import { useRef } from 'react';

function UncontrolledForm() {
  const usernameRef = useRef(null);
  const fileRef = useRef(null);

  const handleSubmit = (e) => {
    e.preventDefault();

    console.log('用户名：', usernameRef.current.value);
    console.log('文件：', fileRef.current.files[0]);

    usernameRef.current.focus();
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        ref={usernameRef}
        type="text"
        placeholder="用户名"
        defaultValue="张三"
      />

      <input
        ref={fileRef}
        type="file"
      />

      <button type="submit">提交</button>
    </form>
  );
}
```

---

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

## 下一阶段预告

完成基础阶段后，将进入 [React 进阶基础](./stage-2-advanced-basic.md)，你将学习：
- 高阶 Hooks（useMemo、useCallback、useReducer）
- 组件通信全方案
- React Router 路由管理
- 性能优化基础
