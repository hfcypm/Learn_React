# TypeScript 从基础到进阶学习路线

**目标**：掌握 TypeScript 类型系统，能够编写类型安全的 React 应用

## 概述

TypeScript 是 JavaScript 的超集，它添加了可选的静态类型检查和面向对象特性。学习 TypeScript 可以显著提升代码质量和开发效率，尤其在使用 React 开发大型应用时。

**学习路线**：
- 阶段一：TypeScript 基础（环境、基础类型、接口、函数）
- 阶段二：TypeScript 进阶（泛型、条件类型、映射类型）
- 阶段三：TypeScript 高级（装饰器、声明文件、模块系统）
- 阶段四：React + TypeScript 实战（最佳实践、工程化配置）

---

## 阶段一：TypeScript 基础

### 1. 环境搭建与配置

- [ ] TypeScript 编译器安装
- [ ] tsconfig.json 配置详解
- [ ] IDE 配置（VSCode 类型提示）
- [ ] 构建工具集成（Vite、Webpack）

**详细概念：**

**1.1 TypeScript 编译器安装**

TypeScript 是由微软开发的开源语言，它的代码不能直接在浏览器或 Node.js 中运行，需要通过 TypeScript 编译器（tsc）编译成 JavaScript。

安装方式：
```bash
# 全局安装 TypeScript 编译器
npm install -g typescript

# 验证安装
tsc --version

# 在项目中安装（推荐）
npm install -D typescript
```

TypeScript 编译器的工作流程：
1. **解析代码**：读取 .ts/.tsx 文件，生成 AST（抽象语法树）
2. **类型检查**：根据类型注解和推导规则检查类型一致性
3. **编译转换**：将 TypeScript 代码转换为目标版本的 JavaScript
4. **输出结果**：生成 .js 文件和类型声明文件（.d.ts）

**1.2 tsconfig.json 配置详解**

tsconfig.json 是 TypeScript 项目的配置文件，定义了编译选项和文件包含关系。

核心配置项：
```json
{
  "compilerOptions": {
    "target": "ES2020",           // 编译目标 JavaScript 版本
    "module": "ESNext",           // 模块系统
    "lib": ["ES2020", "DOM"],     // 内置库类型定义
    "jsx": "react-jsx",          // JSX 处理模式
    
    "strict": true,               // 启用所有严格类型检查
    "noImplicitAny": true,        // 禁止隐式 any 类型
    "strictNullChecks": true,     // 严格 null 检查
    
    "moduleResolution": "node",    // 模块解析策略
    "esModuleInterop": true,      // 允许默认导入
    "skipLibCheck": true,         // 跳过库文件类型检查
    "forceConsistentCasingInFileNames": true,
    
    "outDir": "./dist",           // 输出目录
    "rootDir": "./src",           // 源码目录
    "declaration": true,          // 生成声明文件
    "declarationMap": true,       // 生成声明文件映射
    "sourceMap": true              // 生成源码映射
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

常见编译目标版本：
- `ES5`：兼容性最好，支持所有现代浏览器
- `ES2020`：支持最新特性，需要现代浏览器
- `ESNext`：支持最新特性，动态加载

**1.3 strict 模式详解**

`strict: true` 启用所有严格类型检查选项，这是企业项目的推荐配置。

严格模式包含的检查：
- **noImplicitAny**：禁止变量和参数使用隐式 any 类型
- **strictNullChecks**：不允许 null/undefined 作为非 nullable 类型
- **strictFunctionTypes**：函数类型检查更严格
- **strictPropertyInitialization**：类的属性必须在构造函数中初始化
- **noImplicitThis**：this 类型不明确时报错

非 strict 模式下的问题示例：
```typescript
// 非 strict 模式：不会报错
function greet(name) {
  console.log(name.toUpperCase()); // 如果 name 是 undefined，运行时会崩溃
}

// strict 模式：正确报错
function greet(name: string) {
  console.log(name.toUpperCase());
}
```

**经典案例：项目初始化与配置**

```bash
# 创建项目目录
mkdir my-ts-project && cd my-ts-project

# 初始化 npm 项目
npm init -y

# 安装 TypeScript
npm install -D typescript

# 安装 Node.js 类型定义
npm install -D @types/node

# 初始化 tsconfig.json
npx tsc --init
```

```json
// tsconfig.json - 完整配置
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

```json
// tsconfig.node.json - Node 环境配置
{
  "compilerOptions": {
    "composite": true,
    "skipLibCheck": true,
    "module": "ESNext",
    "moduleResolution": "bundler",
    "allowSyntheticDefaultImports": true
  },
  "include": ["vite.config.ts"]
}
```

---

### 2. 基础类型系统

- [ ] 基础类型（string、number、boolean）
- [ ] 数组类型
- [ ] 元组类型
- [ ] 枚举类型
- [ ] null 和 undefined
- [ ] void 和 never
- [ ] object 类型

**详细概念：**

**2.1 基础类型**

TypeScript 支持 JavaScript 的所有基础类型，并添加了类型注解机制。

类型注解语法：
```typescript
let name: string = 'Tom';
let age: number = 25;
let isStudent: boolean = true;
```

类型推断：
```typescript
// TypeScript 会自动推断类型
let name = 'Tom';  // 推断为 string
name = 25;  // 错误：不能将 number 赋值给 string
```

字面量类型：
```typescript
// 使用 const 声明时，类型就是值本身
const PI = 3.14159;  // 类型是 3.14159

// 使用字面量定义可选值
type Direction = 'north' | 'south' | 'east' | 'west';
let direction: Direction = 'north';
direction = 'center';  // 错误：center 不是有效的 Direction
```

**2.2 数组类型**

数组类型有两种声明方式：

```typescript
// 方式一：类型[]
let numbers: number[] = [1, 2, 3, 4, 5];

// 方式二：Array<类型>
let names: Array<string> = ['Tom', 'Jerry', 'Spike'];

// 只读数组
const readonlyArray: ReadonlyArray<number> = [1, 2, 3];
// readonlyArray.push(4);  // 错误：只读数组不能修改
```

**2.3 元组类型**

元组是固定长度和类型的数组：

```typescript
// 声明元组类型
let person: [string, number, boolean] = ['Tom', 25, true];

// 访问元组元素
console.log(person[0]);  // string
console.log(person[1]);  // number

// 可选元组元素
type OptionalTuple = [string, number?];
let example: OptionalTuple = 'hello';
let example2: OptionalTuple = ['hello', 42];

// 解构赋值
const [name, age, isActive] = person;
```

元组与数组的区别：
- 数组：同类型、不定长度
- 元组：定长度、不同类型（或相同类型）

**2.4 枚举类型**

枚举用于定义命名常量集合：

```typescript
// 数字枚举（默认）
enum Direction {
  North,    // 0
  South,    // 1
  East,     // 2
  West      // 3
}

let dir: Direction = Direction.North;

// 字符串枚举
enum Status {
  Pending = 'PENDING',
  Active = 'ACTIVE',
  Completed = 'COMPLETED'
}

// 反向映射（数字枚举）
console.log(Direction[0]);  // "North"

// const 枚举
const enum Color {
  Red = '#FF0000',
  Green = '#00FF00',
  Blue = '#0000FF'
}
// const 枚举在编译时会内联，提高性能
```

**2.5 null 和 undefined**

TypeScript 中 null 和 undefined 是两种不同的类型：

```typescript
// null 类型
let n: null = null;
n = undefined;  // 错误：undefined 不能赋值给 null

// undefined 类型
let u: undefined = undefined;
u = null;  // 错误：null 不能赋值给 undefined

// 可选链与空值合并
interface User {
  name: string;
  address?: {
    city: string;
    zip?: string;
  };
}

const user: User = { name: 'Tom' };
const city = user.address?.city ?? '未知';
```

**2.6 void 和 never**

- **void**：表示没有返回值（用于函数）
- **never**：表示永远不会返回（用于永不返回的函数）

```typescript
// void - 函数没有 return 或 return undefined
function logMessage(message: string): void {
  console.log(message);
}

// never - 抛出异常或无限循环
function throwError(message: string): never {
  throw new Error(message);
}

function infiniteLoop(): never {
  while (true) {
    // 无限循环
  }
}

// never 用于类型收窄
function exhaustiveCheck(x: string | number) {
  if (typeof x === 'string') {
    console.log('string');
  } else if (typeof x === 'number') {
    console.log('number');
  } else {
    // never 类型，确保所有情况都被处理
    const _exhaustive: never = x;
  }
}
```

**经典案例：基础类型使用**

```typescript
// types/basic-types.ts

// 1. 基础类型
let name: string = 'TypeScript';
let version: number = 4.9;
let isAwesome: boolean = true;

// 2. 数组
let numbers: number[] = [1, 2, 3, 4, 5];
let names: Array<string> = ['React', 'Vue', 'Angular'];

// 3. 元组 - 固定长度和类型的数组
let tuple: [string, number, boolean] = ['hello', 42, true];
const [str, num, bool] = tuple;

// 4. 枚举
enum Color {
  Red = '#FF0000',
  Green = '#00FF00',
  Blue = '#0000FF'
}

enum Permission {
  Read = 1 << 0,   // 1
  Write = 1 << 1,  // 2
  Execute = 1 << 2 // 4
}

const userPermission: Permission = Permission.Read | Permission.Write;

// 5. 可选属性与默认值的对象类型
interface User {
  id: string;
  name: string;
  email?: string;
  role: 'admin' | 'user' | 'guest';
  createdAt: Date;
}

const user: User = {
  id: '1',
  name: 'Tom',
  role: 'user',
  createdAt: new Date()
};

// 6. 联合类型
type StringOrNumber = string | number;
type Result = 'success' | 'error';
type Callback = () => void;

// 7. 字面量类型
type Direction = 'north' | 'south' | 'east' | 'west';
let direction: Direction = 'north';

// 8. any 和 unknown
// any - 任意类型，放弃类型检查
let dynamicValue: any = 'hello';
dynamicValue = 42;
dynamicValue.toUpperCase();  // 不会报错

// unknown - 未知类型，必须进行类型检查
let unknownValue: unknown = 'hello';
if (typeof unknownValue === 'string') {
  unknownValue.toUpperCase();  // 安全，需要类型检查
}

// 9. 类型别名
type Point = { x: number; y: number };
type ID = string | number;

// 10. 函数类型
type AddFn = (a: number, b: number) => number;
const add: AddFn = (x, y) => x + y;
```

---

### 3. 接口与类型别名

- [ ] 接口定义
- [ ] 可选属性与只读属性
- [ ] 方法与函数类型
- [ ] 接口继承
- [ ] 接口 vs 类型别名
- [ ] 接口合并（声明合并）

**详细概念：**

**3.1 接口基础**

接口用于定义对象的结构契约：

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  age?: number;           // 可选属性
  readonly createdAt: Date;  // 只读属性
}

// 使用接口
const user: User = {
  id: '1',
  name: 'Tom',
  email: 'tom@example.com',
  createdAt: new Date()
};

// user.createdAt = new Date();  // 错误：只读属性不能修改
```

**3.2 复杂接口定义**

接口可以定义：
- 普通属性
- 可选属性（`?`）
- 只读属性（`readonly`）
- 方法
- 函数类型
- 索引签名

```typescript
interface StringMap {
  [key: string]: string;
}

interface Config {
  // 索引签名
  [key: string]: string | number | boolean | undefined;
  
  // 固定属性
  name: string;
  version: string;
  
  // 方法
  getValue(key: string): string;
  
  // 函数类型属性
  onChange: (key: string, value: any) => void;
}
```

**3.3 接口继承**

接口可以继承一个或多个接口：

```typescript
interface Animal {
  name: string;
}

interface Mammal extends Animal {
  furColor: string;
}

interface Dog extends Mammal {
  breed: string;
}

const dog: Dog = {
  name: 'Buddy',
  furColor: 'golden',
  breed: 'Golden Retriever'
};
```

多继承：
```typescript
interface A {
  a: string;
}

interface B {
  b: number;
}

interface C extends A, B {
  c: boolean;
}

const example: C = { a: 'hello', b: 42, c: true };
```

**3.4 接口 vs 类型别名**

接口和类型别名都能定义类型，但有细微区别：

| 特性 | interface | type |
|------|-----------|------|
| 定义对象类型 | ✓ | ✓ |
| 定义基础类型 | ✗ | ✓ |
| 定义联合类型 | ✗ | ✓ |
| 定义元组 | ✗ | ✓ |
| 声明合并 | ✓ | ✗ |
| 继承 | extends | &（交叉） |

```typescript
// 接口 - 适合定义对象结构
interface User {
  name: string;
  age: number;
}

// 类型别名 - 适合定义联合类型、交叉类型
type ID = string | number;
type Point = { x: number } & { y: number };

// 接口可以声明合并
interface Animal {
  name: string;
}

interface Animal {
  breed: string;
}

const animal: Animal = {
  name: 'Tom',
  breed: 'Cat'  // 两个接口的属性都要提供
};
```

**3.5 接口合并（Declaration Merging）**

接口合并是 TypeScript 的独特特性，同名接口会自动合并：

```typescript
// 第一次定义
interface Window {
  title: string;
}

// 扩展已有接口
interface Window {
  ts: string;
}

// 合并后等价于：
interface Window {
  title: string;
  ts: string;
}

// 应用示例：扩展第三方类型
interface Math {
  random(): number;
}

Math.random();  // 现在有类型提示了
```

**经典案例：接口与类型别名**

```typescript
// types/interfaces.ts

// 1. 基础接口
interface User {
  id: string;
  name: string;
  email: string;
  age?: number;
  readonly createdAt: Date;
}

// 2. 只读数组与只读属性
interface Config {
  readonly apiUrl: string;
  readonly timeouts: ReadonlyArray<number>;
}

// 3. 方法接口
interface SearchFunc {
  (source: string, subString: string): boolean;
}

const search: SearchFunc = (source, sub) => {
  return source.includes(sub);
};

// 4. 索引接口
interface StringArray {
  [index: number]: string;
}

const myArray: StringArray = ['Apple', 'Banana', 'Orange'];

// 5. 类接口
interface ClockInterface {
  currentTime: Date;
  setTime(d: Date): void;
}

class Clock implements ClockInterface {
  currentTime: Date = new Date();
  
  setTime(d: Date): void {
    this.currentTime = d;
  }
}

// 6. 接口继承
interface Animal {
  name: string;
}

interface Bird extends Animal {
  canFly: boolean;
}

interface FlyingSquirrel extends Bird {
  glideDistance: number;
}

// 7. 混合类型接口
interface Counter {
  (): void;
  count: number;
}

function createCounter(): Counter {
  const fn = () => { fn.count++; };
  fn.count = 0;
  return fn;
}

// 8. 接口继承类
class Control {
  private state: any;
  protected enabled: boolean = true;
}

interface SelectableControl extends Control {
  select(): void;
}

class Button extends Control implements SelectableControl {
  select(): void {
    console.log('Button selected');
  }
}

// 9. 类型别名 vs 接口对比
// 类型别名 - 定义联合类型
type StringOrNumber = string | number;
type Callback<T> = (data: T) => void;

// 接口 - 定义对象结构
interface Response<T> {
  code: number;
  data: T;
  message: string;
}

// 10. 函数重载接口
interface Overloaded {
  (x: boolean): boolean;
  (x: string): string;
  (x: number): number;
}

function doSomething(x: boolean | string | number): boolean | string | number {
  if (typeof x === 'boolean') return !x;
  if (typeof x === 'string') return x.toUpperCase();
  return x * 2;
}

const overloaded: Overloaded = doSomething;
```

---

### 4. 函数类型

- [ ] 函数类型表达式
- [ ] 调用签名
- [ ] 函数重载
- [ ] this 类型
- [ ] 参数类型
- [ ] 泛型函数

**详细概念：**

**4.1 函数类型表达式**

函数类型表达式定义函数的形状：

```typescript
// 完整写法
let myAdd: (x: number, y: number) => number = 
  function(x: number, y: number): number {
    return x + y;
  };

// 简化写法（参数名可以不同）
let myAdd: (baseValue: number, increment: number) => number = 
  function(x, y) {
    return x + y;
  };
```

**4.2 调用签名**

调用签名用于描述可调用对象（包括函数、类等）的类型：

```typescript
// 函数调用签名
interface DescribeFunction {
  (name: string): string;
  description: string;
}

function doSomething(fn: DescribeFunction) {
  console.log(fn.description + ': ' + fn('Tom'));
}

// 构造签名 - 用于 new 调用
interface PointConstructor {
  new(x: number, y: number): Point;
}

function createPoint(Ctor: PointConstructor, x: number, y: number): Point {
  return new Ctor(x, y);
}
```

**4.3 函数重载**

函数重载允许一个函数有多个类型签名：

```typescript
// 重载签名
function reverse(str: string): string;
function reverse(arr: number[]): number[];
function reverse(strOrArr: string | number[]): string | number[] {
  if (typeof strOrArr === 'string') {
    return strOrArr.split('').reverse().join('');
  }
  return strOrArr.slice().reverse();
}

reverse('hello');   // 返回 "olleh" - 使用第一个重载
reverse([1, 2, 3]); // 返回 [3, 2, 1] - 使用第二个重载
reverse(123);       // 错误：没有匹配的重载
```

重载签名顺序很重要：
- 越具体的签名越靠前
- 通用签名（any/unknown）放最后

**4.4 this 类型**

TypeScript 可以限制函数中 this 的类型：

```typescript
// 在参数中指定 this 类型
function f(this: { x: number }, x: number) {
  console.log(this.x);
}

f({ x: 1 }, 2);  // 正确
f({}, 2);         // 错误：{} 缺少 x 属性

// 箭头函数保留外层 this，不需要 this 参数
const obj = {
  name: 'Tom',
  // 使用箭头函数，this 指向外层
  greet: () => {
    console.log(this.name);  // this 是外层的 this
  },
  // 使用普通函数，需要显式声明 this
  greet2(this: { name: string }) {
    console.log(this.name);
  }
};
```

**4.5 可选参数与默认参数**

```typescript
// 可选参数
function buildName(firstName: string, lastName?: string): string {
  if (lastName) {
    return firstName + ' ' + lastName;
  }
  return firstName;
}

// 默认参数
function connect(host: string, port: number = 80): void {
  console.log(`Connecting to ${host}:${port}`);
}

// 剩余参数
function sum(...numbers: number[]): number {
  return numbers.reduce((acc, n) => acc + n, 0);
}

// 解构参数
function createUser({ name, age }: { name: string; age: number }): void {
  console.log(`${name} is ${age} years old`);
}
```

**4.6 泛型函数**

泛型函数使用类型变量：

```typescript
// 基本泛型函数
function identity<T>(arg: T): T {
  return arg;
}

const num = identity<number>(42);     // 类型为 number
const str = identity('hello');        // 类型为 string（类型推断）

// 多个类型参数
function pair<K, V>(key: K, value: V): [K, V] {
  return [key, value];
}

// 泛型约束
function getLength<T extends { length: number }>(arg: T): number {
  return arg.length;
}

getLength('hello');    // 正确：string 有 length
getLength([1, 2, 3]);  // 正确：数组有 length
getLength(123);        // 错误：number 没有 length
```

**经典案例：函数类型**

```typescript
// types/functions.ts

// 1. 函数类型表达式
type Add = (a: number, b: number) => number;
const add: Add = (x, y) => x + y;

// 2. 可选参数与默认参数
function createUser(
  name: string,
  age: number = 18,
  email?: string
): { name: string; age: number; email?: string } {
  return { name, age, email };
}

// 3. 剩余参数
function sum(...values: number[]): number {
  return values.reduce((acc, val) => acc + val, 0);
}

// 4. 函数重载
function format(value: string): string;
function format(value: number, decimals?: number): string;
function format(value: string | number, decimals?: number): string | number {
  if (typeof value === 'number') {
    return value.toFixed(decimals ?? 2);
  }
  return value.trim().toUpperCase();
}

// 5. this 类型
interface Card {
  suit: string;
  rank: number;
}

function pickCard(this: Card[], x: number): Card {
  return this[x];
}

const deck: Card[] = [
  { suit: 'hearts', rank: 1 },
  { suit: 'clubs', rank: 2 }
];

// 6. 泛型函数
function firstElement<T>(arr: T[]): T | undefined {
  return arr[0];
}

function merge<T extends object, U extends object>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}

function pipe<Input, Middle, Output>(
  input: Input,
  fn1: (i: Input) => Middle,
  fn2: (m: Middle) => Output
): Output {
  return fn2(fn1(input));
}

// 7. 异步函数类型
async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) throw new Error('User not found');
  return response.json();
}

// 8. 回调函数类型
function processItems<T>(
  items: T[],
  callback: (item: T, index: number) => boolean
): T[] {
  return items.filter(callback);
}

// 9. 构造函数类型
interface AnimalConstructor {
  new(name: string): Animal;
  createDefault(): Animal;
}

function createAnimal(Ctor: AnimalConstructor, name: string): Animal {
  return new Ctor(name);
}

// 10. 链式调用（this 类型）
class StringBuilder {
  private content: string = '';
  
  add(text: string): this {
    this.content += text;
    return this;
  }
  
  appendLine(line: string): this {
    this.content += line + '\n';
    return this;
  }
  
  toString(): string {
    return this.content;
  }
}

const sb = new StringBuilder();
const result = sb.add('Hello').appendLine('World').toString();
```

---

## 阶段二：TypeScript 进阶

### 5. 泛型深度理解

- [ ] 泛型基础
- [ ] 泛型约束
- [ ] 多类型参数
- [ ] 泛型类
- [ ] 泛型接口
- [ ] 泛型工具类型

**详细概念：**

**5.1 泛型基础**

泛型允许创建可复用的组件，同时保持类型安全：

```typescript
// 泛型函数
function identity<T>(arg: T): T {
  return arg;
}

// 泛型接口
interface Container<T> {
  value: T;
  getValue(): T;
}

// 泛型类
class Box<T> {
  private content: T;
  
  constructor(value: T) {
    this.content = value;
  }
  
  get(): T {
    return this.content;
  }
}
```

类型推断：
```typescript
// TypeScript 自动推断类型
const numBox = new Box(42);       // Box<number>
const strBox = new Box('hello');   // Box<string>
const anyBox = new Box({});        // Box<{}>
```

**5.2 泛型约束**

泛型约束限制泛型的范围：

```typescript
// 使用 extends 约束
interface Lengthwise {
  length: number;
}

function logLength<T extends Lengthwise>(arg: T): number {
  return arg.length;
}

logLength('hello');    // 正确
logLength([1, 2, 3]);  // 正确
logLength({ length: 10 }); // 正确
logLength(123);       // 错误

// 约束泛型必须是另一个泛型的键
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: 'Tom', age: 25 };
const name = getProperty(user, 'name');  // string
const age = getProperty(user, 'age');    // number
getProperty(user, 'email');              // 错误：email 不在 user 中
```

**5.3 多类型参数**

可以使用多个泛型参数：

```typescript
// 交换元组中的两个元素
function swap<T, U>(tuple: [T, U]): [U, T] {
  return [tuple[1], tuple[0]];
}

// 创建键值对
function createPair<K, V>(key: K, value: V): { key: K; value: V } {
  return { key, value };
}

// 条件类型结合多泛型
type FirstOfPair<T, U> = T extends U ? T : U;
```

**5.4 泛型类**

泛型类使用类型参数定义类：

```typescript
class Stack<T> {
  private items: T[] = [];
  
  push(item: T): void {
    this.items.push(item);
  }
  
  pop(): T | undefined {
    return this.items.pop();
  }
  
  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }
  
  isEmpty(): boolean {
    return this.items.length === 0;
  }
}

const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push(2);
numberStack.pop();  // 返回 2

const stringStack = new Stack<string>();
stringStack.push('hello');
stringStack.push('world');
```

泛型类的继承：
```typescript
class MutableStack<T> extends Stack<T> {
  clear(): void {
    this.items = [];
  }
}
```

**5.5 泛型接口**

泛型接口定义通用接口契约：

```typescript
interface Pair<K, V> {
  key: K;
  value: V;
}

interface Map<K, V> {
  get(key: K): V | undefined;
  set(key: K, value: V): void;
  has(key: K): boolean;
  delete(key: K): boolean;
}

// 函数类型作为泛型
interface Transformer<T, R> {
  (input: T): R;
}

function transform<T, R>(value: T, transformer: Transformer<T, R>): R {
  return transformer(value);
}
```

**5.6 内置泛型工具类型**

TypeScript 提供了一系列内置的工具类型：

```typescript
// Partial<T> - 将所有属性变为可选
interface User {
  name: string;
  age: number;
}
type PartialUser = Partial<User>;
// 等价于 { name?: string; age?: number; }

// Required<T> - 将所有属性变为必需
type RequiredUser = Required<PartialUser>;

// Readonly<T> - 将所有属性变为只读
type ReadonlyUser = Readonly<User>;

// Pick<T, K> - 从 T 中选择属性 K
type UserPreview = Pick<User, 'name'>;

// Omit<T, K> - 从 T 中移除属性 K
type UserWithoutAge = Omit<User, 'age'>;

// Record<K, V> - 创建键值对类型
type UserMap = Record<string, User>;

// Exclude<T, U> - 从 T 中排除可分配给 U 的类型
type A = string | number | boolean;
type B = string | number;
type C = Exclude<A, B>;  // boolean

// Extract<T, U> - 从 T 中提取可分配给 U 的类型
type D = Extract<A, B>;  // string | number

// NonNullable<T> - 从 T 中排除 null 和 undefined
type E = NonNullable<string | null | undefined>;  // string

// ReturnType<T> - 获取函数返回类型
function createUser() {
  return { name: 'Tom', age: 25 };
}
type UserType = ReturnType<typeof createUser>;

// Parameters<T> - 获取函数参数类型
type UserParams = Parameters<typeof createUser>;  // []
```

**经典案例：泛型深度应用**

```typescript
// types/generics.ts

// 1. 基础泛型函数
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}

function zip<T, U>(arr1: T[], arr2: U[]): [T, U][] {
  return arr1.map((item, i) => [item, arr2[i]]);
}

// 2. 泛型约束
function getLength<T extends { length: number }>(value: T): number {
  return value.length;
}

function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// 3. 泛型类
class Repository<T extends { id: string }> {
  private items: T[] = [];
  
  add(item: T): void {
    this.items.push(item);
  }
  
  get(id: string): T | undefined {
    return this.items.find(item => item.id === id);
  }
  
  getAll(): T[] {
    return [...this.items];
  }
}

interface User { id: string; name: string; }
interface Product { id: string; name: string; price: number; }

const userRepo = new Repository<User>();
userRepo.add({ id: '1', name: 'Tom' });

// 4. 泛型接口
interface Result<T, E = Error> {
  success: true;
  data: T;
} | {
  success: false;
  error: E;
};

function fetchUser(id: string): Result<User> {
  if (id === '0') {
    return { success: false, error: new Error('User not found') };
  }
  return { success: true, data: { id, name: 'Tom' } };
}

// 5. 条件类型与泛型
type NonNullable<T> = T extends null | undefined ? never : T;

type Flatten<T> = T extends Array<infer U> ? U : T;

type ToArray<T> = T extends any ? T[] : never;

// 6. 递归泛型
type DeepReadonly<T> = T extends (infer U)[]
  ? DeepReadonlyArray<U>
  : T extends object
  ? DeepReadonlyObject<T>
  : T;

interface DeepReadonlyArray<T> extends ReadonlyArray<DeepReadonly<T>> {}

type DeepReadonlyObject<T> = {
  readonly [P in keyof T]: DeepReadonly<T[P]>;
};

// 7. 泛型工具类型实现
type MyPartial<T> = {
  [P in keyof T]?: T[P];
};

type MyRequired<T> = {
  [P in keyof T]-?: T[P];
};

type MyPick<T, K extends keyof T> = {
  [P in K]: T[P];
};

type MyOmit<T, K extends keyof T> = {
  [P in Exclude<keyof T, K>]: T[P];
};

type MyRecord<K extends keyof any, V> = {
  [P in K]: V;
};

// 8. 多泛型参数
function mapObject<K, V, R>(
  obj: Record<K, V>,
  fn: (value: V, key: K) => R
): Record<K, R> {
  const result = {} as Record<K, R>;
  for (const key in obj) {
    result[key] = fn(obj[key], key);
  }
  return result;
}

// 9. 泛型别名
type Nullable<T> = T | null;
type Optional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

type StringOrNumberArray<T extends string | number> = T[];

// 10. 泛型与默认类型
interface APIClient<Config = {}> {
  get<R>(path: string): Promise<R>;
  post<R, B = unknown>(path: string, body: B): Promise<R>;
}

const client: APIClient<{ baseURL: string }> = {
  async get(path) {
    return fetch(path).then(r => r.json());
  },
  async post(path, body) {
    return fetch(path, { method: 'POST', body: JSON.stringify(body) }).then(r => r.json());
  }
};
```

---

### 6. 条件类型与映射类型

- [ ] 条件类型基础
- [ ] infer 关键字
- [ ] 映射类型基础
- [ ] 映射类型修饰符
- [ ] 递归条件类型
- [ ] 模板字面量类型

**详细概念：**

**6.1 条件类型**

条件类型根据类型关系推断类型：

```typescript
// 基本语法：T extends U ? X : Y
type IsString<T> = T extends string ? 'yes' : 'no';

type A = IsString<string>;  // 'yes'
type B = IsString<number>;  // 'no'

// 分布式条件类型
type ToArray<T> = T extends any ? T[] : never;

type C = ToArray<string | number>;  // string[] | number[]
// 相当于：(string extends any ? string[] : never) | (number extends any ? number[] : never)
```

**6.2 infer 关键字**

infer 用于在条件类型中推断类型：

```typescript
// 推断函数返回类型
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type R1 = ReturnType<() => string>;  // string
type R2 = ReturnType<() => Promise<number>>;  // Promise<number>

// 推断数组元素类型
type ElementType<T> = T extends (infer E)[] ? E : never;

type E1 = ElementType<string[]>;  // string
type E2 = ElementType<number[]>;  // number

// 推断构造函数参数类型
type ConstructorParameters<T extends new (...args: any[]) => any> =
  T extends new (...args: infer P) => any ? P : never;

type CP = ConstructorParameters<new (name: string, age: number) => Person>;
// [string, number]
```

**6.3 映射类型**

映射类型通过遍历已有类型创建新类型：

```typescript
// 基本映射
type KeysToUppercase<T> = {
  [K in keyof T]: string;
};

type Readonly<T> = {
  readonly [K in keyof T]: T[K];
};

type Optional<T> = {
  [K in keyof T]?: T[K];
};
```

**6.4 映射类型修饰符**

映射类型可以使用修饰符：
- `readonly`：只读属性
- `?`：可选属性
- `-`：移除修饰符
- `+`：添加修饰符

```typescript
// 添加只读
type Readonly<T> = {
  readonly [K in keyof T]: T[K];
};

// 移除只读
type Mutable<T> = {
  -readonly [K in keyof T]: T[K];
};

// 添加可选
type Partial<T> = {
  [K in keyof T]?: T[K];
};

// 移除可选
type Required<T> = {
  [K in keyof T]-?: T[K];
};

// 组合使用
type ReadonlyOptional<T> = {
  readonly [K in keyof T]?: T[K];
};
```

**6.5 递归条件类型**

条件类型支持递归，用于处理深层嵌套结构：

```typescript
// 深可选类型
type DeepPartial<T> = T extends object
  ? { [K in keyof T]?: DeepPartial<T[K]> }
  : T;

interface Window {
  title: string;
  size: { width: number; height: number };
}

type PartialWindow = DeepPartial<Window>;
// {
//   title?: string;
//   size?: {
//     width?: number;
//     height?: number;
//   };
// }

// 深只读类型
type DeepReadonly<T> = T extends object
  ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
  : T;
```

**6.6 模板字面量类型**

TypeScript 4.1 引入了模板字面量类型：

```typescript
// 基本用法
type World = 'world';
type Greeting = `hello ${World}`;  // 'hello world'

// 联合类型
type Direction = 'north' | 'south' | 'east' | 'west';
type DirectionEvent = `on${Direction}`;  
// 'onnorth' | 'onsouth' | 'oneast' | 'onwest'

// 通用工具类型
type EventName<T extends string> = `on${Capitalize<T>}`;

type ClickEvent = EventName<'click'>;  // 'onClick'
type HoverEvent = EventName<'hover'>;  // 'onHover'
```

**经典案例：条件类型与映射类型**

```typescript
// types/advanced-types.ts

// 1. 基本条件类型
type IsArray<T> = T extends any[] ? true : false;
type IsArrayOfString<T> = T extends string[] ? true : false;

type A = IsArray<string[]>;  // true
type B = IsArray<string>;    // false

// 2. 条件类型与联合类型
type ExtractArrayParts<T> = T extends (infer U)[]
  ? { item: U; isArray: true }
  : { value: T; isArray: false };

type P1 = ExtractArrayParts<string[]>;  // { item: string; isArray: true }
type P2 = ExtractArrayParts<number>;    // { value: number; isArray: false }

// 3. 映射类型
type Stringify<T> = {
  [K in keyof T]: string;
};

type Numberify<T> = {
  [K in keyof T]: T[K] extends object ? Numberify<T[K]> : number;
};

// 4. 键重映射
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface User {
  name: string;
  age: number;
}

type UserGetters = Getters<User>;
// { getName: () => string; getAge: () => number }

// 5. 条件映射
type ConditionalPick<T, U> = {
  [K in keyof T as T[K] extends U ? K : never]: T[K];
};

interface User2 {
  name: string;
  age: number;
  address: string;
}

type StringPropsOnly = ConditionalPick<User2, string>;
// { name: string; address: string }

// 6. 递归类型
type DeepReadonly<T> = T extends (infer U)[]
  ? ReadonlyArray<DeepReadonly<U>>
  : T extends object
  ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
  : T;

type Flatten<T> = T extends Array<infer U> ? Flatten<U> : T;

// 7. 模板字面量类型
type EventName = 'click' | 'focus' | 'blur';
type HandlerName = `on${Capitalize<EventName>}`;
// 'onClick' | 'onFocus' | 'onBlur'

type ExtractRoute<T extends string> = 
  T extends `${infer Method} ${infer Path}` ? { method: Method; path: Path } : never;

type Route = ExtractRoute<'GET /api/users'>;
// { method: 'GET'; path: '/api/users' }

// 8. 分发与非分发
type ToArray<T> = T extends any ? T[] : never;

type Dist = ToArray<string | number>;  // string[] | number[]
type NonDist = [ToArray<string>] | [ToArray<number>];  // [string[]] | [number[]]

// 9. 元组与条件类型
type First<T extends any[]> = T extends [infer F, ...any[]] ? F : never;
type Rest<T extends any[]> = T extends [any, ...infer R] ? R : never;

type F = First<[string, number, boolean]>;  // string
type R = Rest<[string, number, boolean]>;   // [number, boolean]

// 10. 综合示例 - 实现一个简易版 zustand
type StateCreator<
  S extends object,
  Mps extends [string, any][] = [],
  Mcs extends [string, any][] = []
> = {
  get: () => S;
  set: (partial: Partial<S>) => void;
} & {
  [K in Mps[number] as `get${Capitalize<K>}`]: () => Mps[number][1];
} & {
  [K in Mcs[number] as `${Mcs[number][1]}`]: (
    partial: Partial<S>
  ) => void;
};

declare function createStore<
  S extends object,
  Mps extends [string, any][] = [],
  Mcs extends [string, any][] = []
>(
  initializer: StateCreator<S, Mps, Mcs>
): StateCreator<S, Mps, Mcs>;
```

---

### 7. 类型守卫与类型收窄

- [ ] typeof 类型守卫
- [ ] instanceof 类型守卫
- [ ] in 操作符
- [ ] 自定义类型守卫
- [ ] 断言函数
- [ ] 类型谓词
- [ ] 穷尽性检查

**详细概念：**

**7.1 typeof 类型守卫**

typeof 是 JavaScript 原生的类型检查，TypeScript 用来收窄类型：

```typescript
function padLeft(value: string | number, padding: string | number) {
  if (typeof padding === 'number') {
    // 这里 padding 是 number 类型
    return Array(padding + 1).join(' ') + value;
  }
  // 这里 padding 是 string 类型
  return padding + value;
}

// typeof 返回的字符串字面量类型
type TypeName<T> = T extends string ? 'string'
  : T extends number ? 'number'
  : T extends boolean ? 'boolean'
  : T extends undefined ? 'undefined'
  : T extends Function ? 'function'
  : 'object';

function getTypeName<T>(value: T): TypeName<T> {
  if (typeof value === 'string') return 'string';
  if (typeof value === 'number') return 'number';
  // ...
}
```

**7.2 instanceof 类型守卫**

instanceof 检查实例的构造函数：

```typescript
class Animal {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}

class Dog extends Animal {
  breed: string;
  constructor(name: string, breed: string) {
    super(name);
    this.breed = breed;
  }
}

class Cat extends Animal {
  color: string;
  constructor(name: string, color: string) {
    super(name);
    this.color = color;
  }
}

function speak(animal: Animal) {
  if (animal instanceof Dog) {
    // animal 被收窄为 Dog 类型
    console.log(`${animal.name} says woof! Breed: ${animal.breed}`);
  } else if (animal instanceof Cat) {
    // animal 被收窄为 Cat 类型
    console.log(`${animal.name} says meow! Color: ${animal.color}`);
  }
}
```

**7.3 in 操作符**

in 操作符检查对象是否包含某个属性：

```typescript
interface Fish {
  swim: () => void;
}

interface Bird {
  fly: () => void;
}

function move(animal: Fish | Bird) {
  if ('swim' in animal) {
    // animal 被收窄为 Fish 类型
    animal.swim();
  } else {
    // animal 被收窄为 Bird 类型
    animal.fly();
  }
}
```

**7.4 自定义类型守卫**

自定义类型守卫返回布尔值，并告诉 TypeScript 如何收窄类型：

```typescript
// 类型谓词 (Type Predicate)
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

function processValue(value: unknown) {
  if (isString(value)) {
    // value 被收窄为 string
    console.log(value.toUpperCase());
  }
}

// 类型断言函数
function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== 'string') {
    throw new Error('Not a string!');
  }
}

function processValue2(value: unknown) {
  assertIsString(value);
  // 断言后 value 是 string 类型
  console.log(value.toUpperCase());
}
```

**7.5 断言函数**

断言函数确保某个条件必须满足：

```typescript
// 断言函数签名
function assert(condition: any, msg?: string): asserts condition {
  if (!condition) {
    throw new Error(msg || 'Assertion failed');
  }
}

// 使用断言函数
function processValue(value: string | number | null) {
  assert(value !== null, 'Value cannot be null');
  // value 被收窄为 string | number
  
  if (typeof value === 'string') {
    console.log(value.toUpperCase());
  }
}

// 另一种断言模式
function assertIsDefined<T>(val: T): asserts val is NonNullable<T> {
  if (val === null || val === undefined) {
    throw new Error('Value is null or undefined');
  }
}
```

**7.6 穷尽性检查**

穷尽性检查确保所有可能的情况都被处理：

```typescript
type Shape = 
  | { kind: 'circle'; radius: number }
  | { kind: 'rectangle'; width: number; height: number }
  | { kind: 'triangle'; base: number; height: number };

function area(shape: Shape): number {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2;
    case 'rectangle':
      return shape.width * shape.height;
    case 'triangle':
      return 0.5 * shape.base * shape.height;
    default:
      // 穷尽性检查
      // 如果添加新的 Shape 类型，这里会报错
      const _exhaustive: never = shape;
      throw new Error('Unknown shape kind');
  }
}
```

**经典案例：类型守卫实战**

```typescript
// types/type-guards.ts

// 1. typeof 类型守卫
function parseInput(input: string | number) {
  if (typeof input === 'string') {
    return input.trim().toUpperCase();
  }
  return input * 2;
}

// 2. instanceof 类型守卫
class ApiError extends Error {
  constructor(public code: number, message: string) {
    super(message);
    this.name = 'ApiError';
  }
}

class NetworkError extends Error {
  constructor(public retryable: boolean, message: string) {
    super(message);
    this.name = 'NetworkError';
  }
}

function handleError(error: Error) {
  if (error instanceof ApiError) {
    console.log(`API Error ${error.code}: ${error.message}`);
  } else if (error instanceof NetworkError) {
    if (error.retryable) {
      console.log(`Retryable network error: ${error.message}`);
    }
  }
}

// 3. in 操作符类型守卫
interface Admin {
  role: 'admin';
  permissions: string[];
}

interface User {
  name: string;
  email: string;
}

function greet(person: Admin | User) {
  if ('permissions' in person) {
    console.log(`Admin with ${person.permissions.length} permissions`);
  } else {
    console.log(`User: ${person.name}`);
  }
}

// 4. 自定义类型守卫
interface Cat {
  meow: () => void;
}

interface Dog {
  bark: () => void;
}

function isCat(animal: Cat | Dog): animal is Cat {
  return (animal as Cat).meow !== undefined;
}

function isDog(animal: Cat | Dog): animal is Dog {
  return (animal as Dog).bark !== undefined;
}

function makeSound(animal: Cat | Dog) {
  if (isCat(animal)) {
    animal.meow();
  } else {
    animal.bark();
  }
}

// 5. 类型谓词
function isNonNullable<T>(val: T): val is NonNullable<T> {
  return val !== null && val !== undefined;
}

function processArray<T>(arr: (T | null | undefined)[]): T[] {
  return arr.filter(isNonNullable);
}

// 6. 断言函数
function assertIsArray(val: unknown): asserts val is unknown[] {
  if (!Array.isArray(val)) {
    throw new Error('Not an array!');
  }
}

function processData(data: unknown) {
  assertIsArray(data);
  // data 现在是 unknown[]
  data.push(1);
}

// 7. 可辨识联合类型
interface SuccessState {
  type: 'success';
  data: User;
}

interface ErrorState {
  type: 'error';
  error: string;
}

interface LoadingState {
  type: 'loading';
}

type AsyncState<T> = SuccessState | ErrorState | { type: 'loading' };

function handleState<T extends AsyncState<any>>(state: T) {
  switch (state.type) {
    case 'success':
      console.log(state.data);
      break;
    case 'error':
      console.error(state.error);
      break;
    case 'loading':
      console.log('Loading...');
      break;
  }
}

// 8. 穷尽性检查
type Vehicle = 
  | { type: 'car'; doors: number }
  | { type: 'bike'; hasBasket: boolean }
  | { type: 'plane'; wingspan: number };

function describeVehicle(vehicle: Vehicle): string {
  switch (vehicle.type) {
    case 'car':
      return `Car with ${vehicle.doors} doors`;
    case 'bike':
      return `Bike with basket: ${vehicle.hasBasket}`;
    case 'plane':
      return `Plane with ${vehicle.wingspan}m wingspan`;
    default:
      const _exhaustive: never = vehicle;
      throw new Error('Unknown vehicle type');
  }
}

// 9. 基于 in 的收窄
type Props = 
  | { show: true; visibleContent: string }
  | { show: false; hiddenReason: string };

function toggleContent(props: Props) {
  if (props.show) {
    return props.visibleContent.toUpperCase();
  } else {
    return `Hidden because: ${props.hiddenReason}`;
  }
}

// 10. 组合类型守卫
function isPromise<T>(val: unknown): val is Promise<T> {
  return (
    !!val &&
    typeof val === 'object' &&
    'then' in val &&
    typeof (val as Promise<T>).then === 'function'
  );
}

async function handleValue(val: unknown) {
  if (isPromise<string>(val)) {
    const result = await val;
    console.log(result);
  } else {
    console.log(val);
  }
}
```

---

## 阶段三：TypeScript 高级

### 8. 装饰器

- [ ] 装饰器基础
- [ ] 装饰器工厂
- [ ] 类装饰器
- [ ] 方法装饰器
- [ ] 属性装饰器
- [ ] 参数装饰器
- [ ] 装饰器组合

**详细概念：**

**8.1 装饰器基础**

装饰器是一种特殊类型的声明，能够附加到类声明、方法、属性或参数上。装饰器使用 `@expression` 形式，其中 `expression` 必须在运行时被调用。

**注意**：装饰器是实验性功能，需要在 tsconfig.json 中启用：
```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

装饰器求值顺序：
1. 参数装饰器，其次是方法装饰器，然后是类装饰器
2. 同一类别中，从左到右求值，从上到下应用

**8.2 装饰器工厂**

装饰器工厂是一个返回装饰器函数的函数：

```typescript
// 装饰器工厂
function logger(message: string) {
  return function(target: any) {
    console.log(`Logger: ${message} - ${target}`);
  };
}

@logger('Application started')
class MyClass {}

// 等价于
class MyClass {}
MyClass = logger('Application started')(MyClass) || MyClass;
```

**8.3 类装饰器**

类装饰器应用于类构造函数：

```typescript
// 简单类装饰器
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class BugReport {
  title: string;
}

// 带参数的类装饰器工厂
function reportable(filename: string) {
  return function(constructor: any) {
    constructor.reportFilename = filename;
  };
}

@reportable('bug-report.txt')
class BugReport2 {
  title: string;
}

console.log(BugReport2.reportFilename);  // 'bug-report.txt'
```

**8.4 方法装饰器**

方法装饰器应用于类方法：

```typescript
function readonly(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  descriptor.writable = false;
  return descriptor;
}

function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    console.log(`Calling ${propertyKey} with`, args);
    return original.apply(this, args);
  };
  
  return descriptor;
}

class Calculator {
  @readonly
  result: number = 0;
  
  @log
  add(a: number, b: number): number {
    return a + b;
  }
}
```

**8.5 属性装饰器**

属性装饰器应用于类的属性：

```typescript
function format(formatString: string) {
  return function(target: any, propertyKey: string) {
    let value: any;
    
    Object.defineProperty(target, propertyKey, {
      get: () => value,
      set: (v: any) => {
        value = formatString.replace('{}', String(v));
      },
      enumerable: true,
      configurable: true
    });
  };
}

class User {
  @format('User: {}')
  name: string = '';
}

const user = new User();
user.name = 'Tom';
console.log(user.name);  // 'User: Tom'
```

**8.6 参数装饰器**

参数装饰器应用于构造函数或方法的参数：

```typescript
function required(target: any, propertyKey: string, parameterIndex: number) {
  console.log(`Required parameter at index ${parameterIndex} in ${propertyKey}`);
}

class UserService {
  createUser(
    @required name: string,
    @required email: string
  ): User {
    return { name, email };
  }
}
```

**经典案例：装饰器实战**

```typescript
// decorators/index.ts

// 1. 启用装饰器配置需要的类型声明
declare module 'reflect-metadata' {
  function decorate(
    target: Object,
    targetKey?: string | symbol,
    parameterIndex?: number
  ): any;
}

// 2. 简单装饰器
function timestamp<T extends { new(...args: any[]): {} }>(
  constructor: T
) {
  return class extends constructor {
    createdAt = new Date();
  };
}

@timestamp
class Created {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}

// 3. 方法装饰器 - 防抖
function debounce(wait: number) {
  return function(
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    let timeout: ReturnType<typeof setTimeout>;
    
    const original = descriptor.value;
    
    descriptor.value = function(...args: any[]) {
      clearTimeout(timeout);
      timeout = setTimeout(() => original.apply(this, args), wait);
    };
    
    return descriptor;
  };
}

class ApiService {
  @debounce(300)
  fetchData(id: string) {
    console.log(`Fetching data for ${id}`);
  }
}

// 4. 属性装饰器 - 验证
function minLength(min: number) {
  return function(target: any, propertyKey: string) {
    let value: string;
    
    Object.defineProperty(target, propertyKey, {
      get: () => value,
      set: (v: string) => {
        if (v.length < min) {
          throw new Error(`${propertyKey} must be at least ${min} characters`);
        }
        value = v;
      },
      enumerable: true,
      configurable: true
    });
  };
}

class User {
  @minLength(3)
  username: string = '';
}

// 5. 参数装饰器 - 注入
const injects: Map<string, string> = new Map();

function inject(token: string) {
  return function(target: any, propertyKey: string, parameterIndex: number) {
    injects.set(`${String(propertyKey)}_${parameterIndex}`, token);
  };
}

// 6. 组合装饰器 - 路由装饰器
const METHOD_METADATA = 'method';
const PATH_METADATA = 'path';

function method(method: string) {
  return function(path: string) {
    return function(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
      Reflect.defineMetadata(METHOD_METADATA, method, descriptor.value);
      Reflect.defineMetadata(PATH_METADATA, path, descriptor.value);
      return descriptor;
    };
  };
}

const Get = method('GET');
const Post = method('POST');

class UserController {
  @Get('/users')
  getUsers() {
    return 'All users';
  }
  
  @Post('/users')
  createUser() {
    return 'User created';
  }
}

// 7. 装饰器工厂组合
function compose(...decorators: Function[]) {
  return function(target: Function) {
    decorators.forEach(fn => fn(target));
  };
}

@Component('button')
@Styles('btn btn-primary')
class ButtonComponent {
  text: string = 'Click';
}

// 8. 泛型与装饰器
function memoize<T extends (...args: any[]) => any>(
  target: T,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const original = target;
  const cache = new Map<string, ReturnType<T>>();
  
  descriptor.value = function(...args: Parameters<T>): ReturnType<T> {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key)!;
    }
    const result = original.apply(this, args);
    cache.set(key, result);
    return result;
  };
  
  return descriptor;
}

class MathService {
  @memoize
  fibonacci(n: number): number {
    if (n <= 1) return n;
    return this.fibonacci(n - 1) + this.fibonacci(n - 2);
  }
}
```

---

### 9. 模块系统与声明文件

- [ ] ES 模块与 CommonJS
- [ ] 模块导出导入
- [ ] 声明文件基础
- [ ] .d.ts 声明文件
- [ ]declare module 扩展
- [ ] 全局声明
- [ ] 命名空间

**详细概念：**

**9.1 模块导出导入**

TypeScript 支持 ES 模块和 CommonJS 模块：

```typescript
// ES 模块导出
// types/math.ts
export interface Point {
  x: number;
  y: number;
}

export function add(a: number, b: number): number {
  return a + b;
}

export default function subtract(a: number, b: number): number {
  return a - b;
}

// ES 模块导入
import { add, Point } from './math';
import subtract from './math';
import * as math from './math';

// CommonJS 导出
module.exports = { add, Point };

// CommonJS 导入
const { add, Point } = require('./math');
```

**9.2 模块声明**

模块声明用于声明模块的结构：

```typescript
// 声明一个模块（通常在 .d.ts 文件中）
declare module 'my-module' {
  export function doSomething(): void;
  export interface Config {
    name: string;
  }
}

// 使用时
import { doSomething, Config } from 'my-module';
```

**9.3 声明文件基础**

声明文件（.d.ts）用于为 JavaScript 库提供类型信息：

```typescript
// types/lodash.d.ts
declare module 'lodash' {
  export function chunk<T>(array: T[], size?: number): T[][];
  export function compact<T>(array: (T | null | undefined)[]): T[];
  export function flatten<T>(array: any[]): T[];
  
  export interface LoDashStatic {
    chunk<T>(array: T[], size?: number): T[][];
    chunk<T>(array: T[], size: number): T[][];
  }
}
```

**9.4 全局声明**

全局声明可以在任何文件中使用：

```typescript
// global.d.ts
declare global {
  interface Window {
    analytics: Analytics;
  }
  
  namespace NodeJS {
    interface ProcessEnv {
      NODE_ENV: 'development' | 'production';
      API_URL: string;
    }
  }
}

interface Analytics {
  track(event: string, properties?: Record<string, any>): void;
}

export {};
```

**9.5 扩展已有模块**

可以扩展第三方模块的类型：

```typescript
// 扩展 express 模块
declare module 'express' {
  interface Request {
    userId?: string;
  }
}

// 扩展 react 模块
declare module 'react' {
  interface InputHTMLAttributes<T> {
    customAttribute?: string;
  }
}
```

**经典案例：声明文件与模块系统**

```typescript
// types/declarations.d.ts

// 1. 全局声明
declare const VERSION: string;
declare const DEBUG: boolean;

declare function log(message: string, level?: 'info' | 'warn' | 'error'): void;

// 2. 全局接口
interface GlobalConfig {
  apiUrl: string;
  timeout: number;
}

declare var CONFIG: GlobalConfig;

// 3. 模块声明
declare module 'my-custom-lib' {
  export interface Options {
    debug?: boolean;
    timeout?: number;
  }
  
  export class Client {
    constructor(options: Options);
    request<T>(url: string): Promise<T>;
  }
  
  export default Client;
}

// 4. 函数重载声明
declare function parseJSON(json: string): any;
declare function parseJSON<T>(json: string): T;

// 5. 可调用接口
interface CallSignature {
  (name: string): string;
  description: string;
}

declare const createCallable: {
  new(description: string): CallSignature;
  (description: string): CallSignature;
};

// 6. 扩展已有模块
declare module 'express' {
  interface Application {
    locals: {
      user?: User;
      session?: Session;
    };
  }
}

// 7. UMD 声明
declare function createDialog(options?: DialogOptions): Dialog;
declare namespace createDialog {
  const defaultOptions: DialogOptions;
}

// 8. 命名空间声明
declare namespace MyLib {
  const VERSION: string;
  function greet(name: string): string;
  
  namespace Utils {
    function formatDate(date: Date): string;
  }
}

// 9. 模板字符串声明
declare module '*.svg' {
  const content: string;
  export default content;
}

declare module '*.png' {
  const content: string;
  export default content;
}

// 10. 环境变量声明
interface ImportMeta {
  readonly env: ImportMetaEnv;
}

interface ImportMetaEnv {
  readonly VITE_API_URL: string;
  readonly VITE_APP_TITLE: string;
}

declare global {
  interface Window {
    importMeta: ImportMeta;
  }
}

export {};
```

---

### 10. 工具类型实现原理

- [ ] 内置工具类型源码解析
- [ ] 实现自己的工具类型库
- [ ] 类型编程最佳实践

**详细概念：**

**10.1 常用工具类型实现**

TypeScript 内置的工具类型都有对应的实现逻辑：

```typescript
// Partial<T> - 将所有属性变为可选
type Partial<T> = {
  [P in keyof T]?: T[P];
};

// Required<T> - 移除可选
type Required<T> = {
  [P in keyof T]-?: T[P];
};

// Readonly<T> - 添加只读
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

// Pick<T, K> - 选择属性
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

// Omit<T, K> - 排除属性
type Omit<T, K extends keyof T> = {
  [P in Exclude<keyof T, K>]: T[P];
};

// Record<K, V> - 创建对象类型
type Record<K extends keyof any, V> = {
  [P in K]: V;
};
```

**10.2 高级工具类型**

```typescript
// NonNullable<T> - 排除 null 和 undefined
type NonNullable<T> = T extends null | undefined ? never : T;

// Extract<T, U> - 提取 T 中可赋值给 U 的类型
type Extract<T, U> = T extends U ? T : never;

// Exclude<T, U> - 从 T 中排除可赋值给 U 的类型
type Exclude<T, U> = T extends U ? never : T;

// ReturnType<T> - 获取函数返回类型
type ReturnType<T extends (...args: any) => any> = 
  T extends (...args: any) => infer R ? R : any;

// Parameters<T> - 获取函数参数类型
type Parameters<T extends (...args: any) => any> = 
  T extends (...args: infer P) => any ? P : never;

// ConstructorParameters<T> - 获取构造函数参数
type ConstructorParameters<T extends new (...args: any) => any> = 
  T extends new (...args: infer P) => any ? P : never;

// InstanceType<T> - 获取实例类型
type InstanceType<T extends new (...args: any) => any> = 
  T extends new (...args: any) => infer R ? R : any;
```

**10.3 实战工具类型**

```typescript
// DeepPartial - 深可选
type DeepPartial<T> = T extends object
  ? { [P in keyof T]?: DeepPartial<T[P]> }
  : T;

// DeepReadonly - 深只读
type DeepReadonly<T> = T extends (infer U)[]
  ? ReadonlyArray<DeepReadonly<U>>
  : T extends object
  ? { readonly [P in keyof T]: DeepReadonly<T[P]> }
  : T;

// DeepRequired - 深必需
type DeepRequired<T> = T extends (infer U)[]
  ? Required<Required<U>>[]
  : T extends object
  ? { [P in keyof T]-?: DeepRequired<T[P]> }
  : T;

// Mutable<T> - 移除只读
type Mutable<T> = {
  -readonly [P in keyof T]: T[P];
};

// Writable<T> - 同 Mutable

// AtLeast - 至少提供某些属性
type AtLeast<T, K extends keyof T> = 
  Partial<T> & Pick<T, K>;

// ExactlyOne - 只能提供其中一个属性
type ExactlyOne<T, K extends keyof T = keyof T> = 
  { [P in K]?: T[P] } & Partial<Omit<T, K>>;
```

**经典案例：手写工具类型库**

```typescript
// types/custom-utils.ts

// ==================== 基础工具类型 ====================

// Awaited<T> - 获取 Promise 的值类型
type Awaited<T> = T extends null | undefined
  ? T
  : T extends object & { then(onfulfilled: infer F, ...args: any): any }
  ? F extends (value: infer V, ...args: any) => any
    ? Awaited<V>
    : never
  : T;

// Partial<T> - 可选
type MyPartial<T> = {
  [P in keyof T]?: T[P];
};

// Required<T> - 必需
type MyRequired<T> = {
  [P in keyof T]-?: T[P];
};

// Readonly<T> - 只读
type MyReadonly<T> = {
  readonly [P in keyof T]: T[P];
};

// Pick<T, K> - 选择
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P];
};

// Omit<T, K> - 排除
type MyOmit<T, K extends keyof T> = {
  [P in Exclude<keyof T, K>]: T[P];
};

// Record<K, V> - 记录
type MyRecord<K extends keyof any, V> = {
  [P in K]: V;
};

// ==================== 高级工具类型 ====================

// NonNullable<T> - 非空
type MyNonNullable<T> = T extends null | undefined ? never : T;

// Extract<T, U> - 提取
type MyExtract<T, U> = T extends U ? T : never;

// Exclude<T, U> - 排除
type MyExclude<T, U> = T extends U ? never : T;

// ReturnType<T> - 返回类型
type MyReturnType<T extends (...args: any) => any> = 
  T extends (...args: any) => infer R ? R : any;

// Parameters<T> - 参数类型
type MyParameters<T extends (...args: any) => any> = 
  T extends (...args: infer P) => any ? P : never;

// ConstructorParameters<T> - 构造函数参数
type MyConstructorParameters<T extends new (...args: any) => any> = 
  T extends new (...args: infer P) => any ? P : never;

// InstanceType<T> - 实例类型
type MyInstanceType<T extends new (...args: any) => any> = 
  T extends new (...args: any) => infer R ? R : any;

// ==================== 字符串工具类型 ====================

// Uppercase / Lowercase / Capitalize / Uncapitalize 内置

// 字符串模板
type EventName<T extends string> = `on${Capitalize<T>}`;

type StringToUnion<T extends string> = 
  T extends `${infer F}${infer Rest}` 
    ? F | StringToUnion<Rest> 
    : never;

// ==================== 数组工具类型 ====================

// ElementOf<T> - 数组元素类型
type ElementOf<T> = T extends (infer E)[] ? E : never;

// First<T> - 第一个元素
type First<T extends any[]> = T extends [infer F, ...any[]] ? F : never;

// Last<T> - 最后一个元素
type Last<T extends any[]> = T extends [...any[], infer L] ? L : never;

// Push<T, V> - 添加元素到末尾
type Push<T extends any[], V> = [...T, V];

// Unshift<T, V> - 添加元素到开头
type Unshift<T extends any[], V> = [V, ...T];

// ==================== 对象工具类型 ====================

// DeepPartial - 深可选
type DeepPartial<T> = T extends object
  ? { [P in keyof T]?: DeepPartial<T[P]> }
  : T;

// DeepReadonly - 深只读
type DeepReadonly<T> = T extends object
  ? { readonly [P in keyof T]: DeepReadonly<T[P]> }
  : T;

// Merge<T, U> - 合并类型
type Merge<T extends object, U extends object> = 
  Omit<T, keyof U> & U;

// Overwrite<T, U> - 覆盖类型
type Overwrite<T extends object, U extends object> = 
  Merge<T, { [P in keyof U]: U[P] }>;

// ==================== 条件类型工具 ====================

// IsAny<T> - 判断是否为 any
type IsAny<T> = 0 extends (1 & T) ? true : false;

// IsNever<T> - 判断是否为 never
type IsNever<T> = [T] extends [never] ? true : false;

// If<C, T, F> - 条件类型
type If<C extends boolean, T, F> = C extends true ? T : F;

// ==================== 函数工具类型 ====================

// ThisParameterType<T> - 获取 this 类型
type MyThisParameterType<T> = 
  T extends (this: infer This, ...args: any) => any ? This : unknown;

// OmitThisParameter<T> - 移除 this 类型
type MyOmitThisParameter<T> = 
  T extends (...args: infer P) => infer R 
    ? (...args: P) => R 
    : T;
```

---

## 阶段四：React + TypeScript 实战

### 11. React TypeScript 最佳实践

- [ ] 组件类型定义
- [ ] Props 类型定义
- [ ] Hooks 类型定义
- [ ] 事件处理类型
- [ ] 泛型组件

**详细概念：**

**11.1 组件类型定义**

React 函数组件的返回类型：

```typescript
import React from 'react';

// 方式一：直接指定返回类型
function Component(): React.ReactNode {
  return <div>Hello</div>;
}

// 方式二：使用 JSX.Element
function Component2(): JSX.Element {
  return <div>Hello</div>;
}

// 方式三：使用 React.FC（不推荐）
const Component3: React.FC = () => {
  return <div>Hello</div>;
};
```

**推荐**：使用 `React.ReactNode` 或直接返回 JSX，不使用 `React.FC`。

原因：
- `React.FC` 会自动添加 `children` prop，可能不符合预期
- `React.FC` 的类型推断有时不够精确
- 直接返回类型更简单直观

**11.2 Props 类型定义**

Props 类型定义的最佳实践：

```typescript
// 基础 Props
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
}

function Button({ label, onClick, variant = 'primary', disabled = false }: ButtonProps) {
  return (
    <button className={`btn btn-${variant}`} onClick={onClick} disabled={disabled}>
      {label}
    </button>
  );
}

// 必需 Props 与可选 Props
interface UserCardProps {
  user: User;                    // 必需
  onEdit?: (user: User) => void; // 可选
  onDelete?: (id: string) => void;
  isSelected?: boolean;          // 带默认值的可选
}

// Props with children
interface CardProps {
  title: string;
  children: React.ReactNode;
}

// 使用 React.PropsWithChildren
type CardProps2 = React.PropsWithChildren<{ title: string }>;
```

**11.3 Hooks 类型定义**

React Hooks 的类型定义：

```typescript
// useState - 显式指定类型
const [count, setCount] = useState<number>(0);
const [user, setUser] = useState<User | null>(null);
const [loading, setLoading] = useState(false);

// useRef
const inputRef = useRef<HTMLInputElement>(null);
const timerRef = useRef<ReturnType<typeof setTimeout> | null>(null);

// useCallback
const handleClick = useCallback<React.MouseEventHandler<HTMLButtonElement>>(() => {
  console.log('clicked');
}, []);

// useMemo
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(a, b);
}, [a, b]);

// useContext
const theme = useContext<ThemeContextType>(ThemeContext);
```

**11.4 事件处理类型**

React 事件的类型定义：

```typescript
// 鼠标事件
onClick: React.MouseEventHandler<HTMLButtonElement>
onMouseEnter: React.MouseEventHandler<HTMLDivElement>
onMouseDown: React.MouseEventHandler<Element>

// 获取事件目标
onClick: (e: React.MouseEvent<HTMLButtonElement>) => {
  const button = e.currentTarget;
  const value = button.value;
}

// 表单事件
onChange: React.ChangeEventHandler<HTMLInputElement>
onSubmit: React.FormEventHandler<HTMLFormElement>

// 键盘事件
onKeyDown: React.KeyboardEventHandler<HTMLInputElement>

// 焦点事件
onFocus: React.FocusEventHandler<HTMLInputElement>
onBlur: React.FocusEventHandler<HTMLInputElement>

// 通用事件类型
onClick: (e: React.SyntheticEvent) => void
```

**11.5 泛型组件**

泛型组件可以处理多种数据类型：

```typescript
// 泛型列表组件
interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  keyExtractor: (item: T) => string;
  emptyText?: string;
}

function List<T>({ items, renderItem, keyExtractor, emptyText = 'No items' }: ListProps<T>) {
  if (items.length === 0) {
    return <div>{emptyText}</div>;
  }
  
  return (
    <div>
      {items.map((item, index) => (
        <div key={keyExtractor(item)}>
          {renderItem(item, index)}
        </div>
      ))}
    </div>
  );
}

// 使用泛型组件
interface User {
  id: string;
  name: string;
}

const users: User[] = [
  { id: '1', name: 'Tom' },
  { id: '2', name: 'Jerry' }
];

<List
  items={users}
  keyExtractor={(user) => user.id}
  renderItem={(user) => <div>{user.name}</div>}
/>
```

**经典案例：React + TypeScript 完整示例**

```tsx
// types/react-types.tsx

// ==================== 组件类型 ====================

import React, { useState, useCallback, useMemo, useRef, useEffect } from 'react';

// 基础组件 Props
interface ButtonProps {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  onClick?: React.MouseEventHandler<HTMLButtonElement>;
}

function Button({ 
  children, 
  variant = 'primary', 
  size = 'medium',
  disabled = false, 
  onClick 
}: ButtonProps) {
  return (
    <button 
      className={`btn btn-${variant} btn-${size}`}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  );
}

// ==================== 表单组件 ====================

interface InputProps {
  value: string;
  onChange: React.ChangeEventHandler<HTMLInputElement>;
  placeholder?: string;
  type?: 'text' | 'email' | 'password' | 'number';
  error?: string;
  label?: string;
}

function Input({ value, onChange, placeholder, type = 'text', error, label }: InputProps) {
  return (
    <div className="input-wrapper">
      {label && <label>{label}</label>}
      <input
        type={type}
        value={value}
        onChange={onChange}
        placeholder={placeholder}
        className={error ? 'input-error' : ''}
      />
      {error && <span className="error-text">{error}</span>}
    </div>
  );
}

// ==================== 泛型列表组件 ====================

interface Column<T> {
  key: keyof T;
  title: string;
  render?: (value: T[keyof T], record: T) => React.ReactNode;
}

interface DataTableProps<T> {
  data: T[];
  columns: Column<T>[];
  rowKey: keyof T;
  loading?: boolean;
  onRowClick?: (record: T, index: number) => void;
  emptyText?: string;
}

function DataTable<T extends Record<string, unknown>>({
  data,
  columns,
  rowKey,
  loading,
  onRowClick,
  emptyText = '暂无数据'
}: DataTableProps<T>) {
  if (loading) {
    return <div className="loading">加载中...</div>;
  }
  
  if (data.length === 0) {
    return <div className="empty">{emptyText}</div>;
  }
  
  return (
    <table>
      <thead>
        <tr>
          {columns.map(col => (
            <th key={String(col.key)}>{col.title}</th>
          ))}
        </tr>
      </thead>
      <tbody>
        {data.map((record, index) => (
          <tr 
            key={String(record[rowKey])}
            onClick={() => onRowClick?.(record, index)}
          >
            {columns.map(col => (
              <td key={String(col.key)}>
                {col.render 
                  ? col.render(record[col.key], record)
                  : String(record[col.key])
                }
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}

// ==================== 泛型表单组件 ====================

interface FormFieldProps<T> {
  name: keyof T;
  label: string;
  value: T[keyof T];
  onChange: (name: keyof T, value: T[keyof T]) => void;
  type?: 'text' | 'email' | 'password' | 'number';
  error?: string;
  required?: boolean;
}

function FormField<T extends Record<string, unknown>>({
  name,
  label,
  value,
  onChange,
  type = 'text',
  error,
  required
}: FormFieldProps<T>) {
  const handleChange: React.ChangeEventHandler<HTMLInputElement> = (e) => {
    onChange(name, e.target.value as T[keyof T]);
  };
  
  return (
    <div className="form-field">
      <label>
        {required && <span className="required">*</span>}
        {label}
      </label>
      <input
        type={type}
        value={value as string}
        onChange={handleChange}
      />
      {error && <span className="error">{error}</span>}
    </div>
  );
}

// ==================== Context 类型 ====================

interface ThemeContextValue {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
  colors: {
    primary: string;
    background: string;
    text: string;
  };
}

const ThemeContext = React.createContext<ThemeContextValue | null>(null);

function useTheme(): ThemeContextValue {
  const context = React.useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// ==================== 自定义 Hooks 类型 ====================

function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });
  
  const setValue = useCallback((value: T | ((val: T) => T)) => {
    const valueToStore = value instanceof Function ? value(storedValue) : value;
    setStoredValue(valueToStore);
    window.localStorage.setItem(key, JSON.stringify(valueToStore));
  }, [key, storedValue]);
  
  return [storedValue, setValue] as const;
}

// ==================== 异步 Hooks ====================

interface AsyncState<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
}

function useAsync<T>(
  asyncFunction: () => Promise<T>,
  dependencies: unknown[] = []
) {
  const [state, setState] = useState<AsyncState<T>>({
    data: null,
    loading: true,
    error: null
  });
  
  useEffect(() => {
    let mounted = true;
    
    asyncFunction()
      .then(data => {
        if (mounted) {
          setState({ data, loading: false, error: null });
        }
      })
      .catch(error => {
        if (mounted) {
          setState({ data: null, loading: false, error });
        }
      });
    
    return () => { mounted = false; };
  }, dependencies);
  
  return state;
}
```

---

### 12. 实战：构建类型安全的 API 层

- [ ] API 响应类型定义
- [ ] 请求参数类型化
- [ ] 错误处理类型
- [ ] Axios 类型封装
- [ ] React Query 集成

**详细概念：**

**12.1 API 响应类型定义**

统一 API 响应格式的类型定义：

```typescript
// 统一响应格式
interface ApiResponse<T = unknown> {
  code: number;
  data: T;
  message: string;
  timestamp: number;
}

// 分页响应
interface PaginatedResponse<T> {
  list: T[];
  total: number;
  page: number;
  pageSize: number;
  totalPages: number;
}

// API 错误响应
interface ApiError {
  code: number;
  message: string;
  details?: Record<string, string[]>;
}
```

**12.2 请求参数类型化**

每个 API 的请求参数都应该类型化：

```typescript
// 用户相关
interface GetUsersParams {
  page?: number;
  pageSize?: number;
  keyword?: string;
  role?: 'admin' | 'user';
}

interface CreateUserData {
  name: string;
  email: string;
  password: string;
  role: 'admin' | 'user';
}

interface UpdateUserData {
  name?: string;
  email?: string;
  role?: 'admin' | 'user';
}

// 产品相关
interface GetProductsParams {
  category?: string;
  minPrice?: number;
  maxPrice?: number;
  inStock?: boolean;
  sortBy?: 'price' | 'name' | 'createdAt';
  sortOrder?: 'asc' | 'desc';
}
```

**12.3 错误处理类型**

类型化的错误处理：

```typescript
// 业务错误码
enum ApiCode {
  Success = 200,
  BadRequest = 400,
  Unauthorized = 401,
  Forbidden = 403,
  NotFound = 404,
  InternalError = 500
}

// 自定义 API 错误类
class ApiError extends Error {
  constructor(
    public code: ApiCode,
    message: string,
    public details?: Record<string, string[]>
  ) {
    super(message);
    this.name = 'ApiError';
  }
}

// Result 类型 - 替代 try/catch
type Result<T, E = ApiError> = 
  | { success: true; data: T }
  | { success: false; error: E };

// 使用示例
async function fetchUser(id: string): Promise<Result<User, ApiError>> {
  try {
    const response = await api.get(`/users/${id}`);
    return { success: true, data: response };
  } catch (error) {
    return { success: false, error: error as ApiError };
  }
}
```

**12.4 Axios 类型封装**

类型安全的 Axios 封装：

```typescript
import axios, { AxiosInstance, AxiosRequestConfig } from 'axios';

// 创建实例
const api: AxiosInstance = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 10000,
});

// 请求拦截器
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token && config.headers) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// 响应拦截器
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // 处理未授权
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

// 类型化的请求方法
const request = {
  get<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    return api.get<ApiResponse<T>>(url, config).then(res => res.data.data);
  },
  
  post<T>(url: string, data?: unknown, config?: AxiosRequestConfig): Promise<T> {
    return api.post<ApiResponse<T>>(url, data, config).then(res => res.data.data);
  },
  
  put<T>(url: string, data?: unknown, config?: AxiosRequestConfig): Promise<T> {
    return api.put<ApiResponse<T>>(url, data, config).then(res => res.data.data);
  },
  
  delete<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    return api.delete<ApiResponse<T>>(url, config).then(res => res.data.data);
  }
};

// 服务层类型定义
export const userService = {
  getUsers: (params?: GetUsersParams) => 
    request.get<PaginatedResponse<User>>('/users', { params }),
  
  getUserById: (id: string) => 
    request.get<User>(`/users/${id}`),
  
  createUser: (data: CreateUserData) => 
    request.post<User>('/users', data),
  
  updateUser: (id: string, data: UpdateUserData) => 
    request.put<User>(`/users/${id}`, data),
  
  deleteUser: (id: string) => 
    request.delete<void>(`/users/${id}`)
};
```

**12.5 React Query 集成**

与 React Query 深度集成：

```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { userService, CreateUserData, UpdateUserData } from './api';

// 查询 Hooks
export function useUsers(params?: GetUsersParams) {
  return useQuery({
    queryKey: ['users', params],
    queryFn: () => userService.getUsers(params),
    staleTime: 5 * 60 * 1000,  // 5 分钟内不刷新
  });
}

export function useUser(id: string) {
  return useQuery({
    queryKey: ['user', id],
    queryFn: () => userService.getUserById(id),
    enabled: !!id,  // id 不为空时才执行
  });
}

// 变更 Hooks
export function useCreateUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: (data: CreateUserData) => userService.createUser(data),
    onSuccess: () => {
      // 创建成功后刷新用户列表
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}

export function useUpdateUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: UpdateUserData }) =>
      userService.updateUser(id, data),
    onSuccess: (_, variables) => {
      // 更新成功后刷新用户列表和单个用户
      queryClient.invalidateQueries({ queryKey: ['users'] });
      queryClient.invalidateQueries({ queryKey: ['user', variables.id] });
    },
  });
}

export function useDeleteUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: (id: string) => userService.deleteUser(id),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}
```

**经典案例：完整类型安全 API 层**

```typescript
// types/api-client.ts

import axios, { AxiosInstance, AxiosRequestConfig } from 'axios';

// ==================== 类型定义 ====================

interface ApiResponse<T = unknown> {
  code: number;
  data: T;
  message: string;
  timestamp: number;
}

interface PaginatedResponse<T> {
  list: T[];
  total: number;
  page: number;
  pageSize: number;
}

class ApiException extends Error {
  constructor(
    public code: number,
    message: string,
    public details?: Record<string, string[]>
  ) {
    super(message);
    this.name = 'ApiException';
  }
}

type Result<T> = Promise<{ success: true; data: T } | { success: false; error: ApiException }>;

// ==================== API 客户端 ====================

class ApiClient {
  private instance: AxiosInstance;
  
  constructor(baseURL: string) {
    this.instance = axios.create({
      baseURL,
      timeout: 10000,
      headers: { 'Content-Type': 'application/json' }
    });
    
    this.setupInterceptors();
  }
  
  private setupInterceptors() {
    this.instance.interceptors.request.use(
      (config) => {
        const token = localStorage.getItem('token');
        if (token) {
          config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
      },
      (error) => Promise.reject(error)
    );
    
    this.instance.interceptors.response.use(
      (response) => {
        const { code, data, message } = response.data;
        if (code !== 200) {
          return Promise.reject(new ApiException(code, message));
        }
        return response;
      },
      (error) => {
        if (error.response) {
          const { status, data } = error.response;
          if (status === 401) {
            localStorage.removeItem('token');
            window.location.href = '/login';
          }
          return Promise.reject(
            new ApiException(status, data.message || 'Request failed')
          );
        }
        return Promise.reject(error);
      }
    );
  }
  
  async get<T>(url: string, config?: AxiosRequestConfig): Result<T> {
    try {
      const response = await this.instance.get<ApiResponse<T>>(url, config);
      return { success: true, data: response.data.data };
    } catch (error) {
      return { success: false, error: error as ApiException };
    }
  }
  
  async post<T>(url: string, data?: unknown, config?: AxiosRequestConfig): Result<T> {
    try {
      const response = await this.instance.post<ApiResponse<T>>(url, data, config);
      return { success: true, data: response.data.data };
    } catch (error) {
      return { success: false, error: error as ApiException };
    }
  }
  
  async put<T>(url: string, data?: unknown, config?: AxiosRequestConfig): Result<T> {
    try {
      const response = await this.instance.put<ApiResponse<T>>(url, data, config);
      return { success: true, data: response.data.data };
    } catch (error) {
      return { success: false, error: error as ApiException };
    }
  }
  
  async delete<T>(url: string, config?: AxiosRequestConfig): Result<T> {
    try {
      const response = await this.instance.delete<ApiResponse<T>>(url, config);
      return { success: true, data: response.data.data };
    } catch (error) {
      return { success: false, error: error as ApiException };
    }
  }
}

export const apiClient = new ApiClient(import.meta.env.VITE_API_BASE_URL || '');

// ==================== API 服务层 ====================

// 用户相关类型
interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user' | 'guest';
  createdAt: string;
  updatedAt: string;
}

interface GetUsersParams {
  page?: number;
  pageSize?: number;
  keyword?: string;
  role?: User['role'];
}

interface CreateUserData {
  name: string;
  email: string;
  password: string;
  role: User['role'];
}

interface UpdateUserData {
  name?: string;
  email?: string;
  role?: User['role'];
}

// 用户服务
export const userService = {
  getUsers: (params?: GetUsersParams) =>
    apiClient.get<PaginatedResponse<User>>('/users', { params }),
  
  getUserById: (id: string) =>
    apiClient.get<User>(`/users/${id}`),
  
  createUser: (data: CreateUserData) =>
    apiClient.post<User>('/users', data),
  
  updateUser: (id: string, data: UpdateUserData) =>
    apiClient.put<User>(`/users/${id}`, data),
  
  deleteUser: (id: string) =>
    apiClient.delete<void>(`/users/${id}`)
};

// ==================== React Query 集成 ====================

import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

export function useUsers(params?: GetUsersParams) {
  return useQuery({
    queryKey: ['users', params],
    queryFn: async () => {
      const result = await userService.getUsers(params);
      if (!result.success) throw result.error;
      return result.data;
    },
    select: (data) => data.list,
  });
}

export function useUser(id: string) {
  return useQuery({
    queryKey: ['user', id],
    queryFn: async () => {
      const result = await userService.getUserById(id);
      if (!result.success) throw result.error;
      return result.data;
    },
    enabled: !!id,
  });
}

export function useCreateUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: async (data: CreateUserData) => {
      const result = await userService.createUser(data);
      if (!result.success) throw result.error;
      return result.data;
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}

export function useUpdateUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: async ({ id, data }: { id: string; data: UpdateUserData }) => {
      const result = await userService.updateUser(id, data);
      if (!result.success) throw result.error;
      return result.data;
    },
    onSuccess: (_, variables) => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
      queryClient.invalidateQueries({ queryKey: ['user', variables.id] });
    },
  });
}

export function useDeleteUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: async (id: string) => {
      const result = await userService.deleteUser(id);
      if (!result.success) throw result.error;
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}
```

---

## 学习建议

### 阶段一（基础）- 建议 1-2 周
- 每天花 1-2 小时学习
- 重点理解类型注解、接口、函数类型
- 多做类型推导练习

### 阶段二（进阶）- 建议 2-3 周
- 泛型是核心难点，需要大量练习
- 条件类型和映射类型用于高级类型操作
- 尝试实现自己的工具类型

### 阶段三（高级）- 建议 1-2 周
- 装饰器在 Angular/NestJS 中常用
- 声明文件用于为 JS 库补全类型
- 深入理解 TypeScript 编译器原理

### 阶段四（实战）- 持续学习
- 在实际项目中应用 TypeScript
- 参考 React + TypeScript 最佳实践
- 持续优化类型设计

## 持续学习资源

- [TypeScript 官方文档](https://www.typescriptlang.org/docs/)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)
- [React TypeScript Cheatsheets](https://react-typescript-cheatsheet.netlify.app/)
- [Type Challenges](https://github.com/type-challenges/type-challenges)
