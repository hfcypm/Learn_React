# Electron 新手导读

## 适合谁

这套路线适合已经掌握前端三件套（HTML/CSS/JavaScript）或一个前端框架，准备用 Web 技术构建跨平台桌面应用的开发者。学习重点是双进程模型、IPC 通信、系统集成与打包发布。

## 开始前准备

- 掌握 JavaScript 与异步编程，可参考 [TypeScript 学习路线](../typescript-learning-path/README.md)。
- 熟悉一个前端框架（React/Vue 均可），可参考 [React 学习路线](../react-learning-path/README.md)。
- 了解 Node.js 与 npm，可参考 [Node.js 学习路线](../nodejs-learning-path/README.md)。
- 能运行 Node.js 与 npm。

```bash
node --version
npm --version
```

## 学习顺序

| 阶段 | 先回答的问题 | 阶段产出 |
| --- | --- | --- |
| 零 | 如何初始化并运行第一个窗口？ | 可运行的最小桌面应用 |
| 一 | 如何让界面安全地访问系统能力？ | 带安全 IPC 的窗口 |
| 二 | 如何实现多窗口、菜单与托盘？ | 完整桌面外壳 |
| 三 | 如何读写文件并持久化数据？ | 有真实功能的桌面应用 |
| 四 | 如何打包发布并自动更新？ | 可安装的生产产物 |

## 每阶段学习方法

1. 每个概念先写最小主进程脚本并实际运行。
2. 用开发者工具（Ctrl+Shift+I）观察渲染进程控制台与 DOM。
3. 先在主进程 `console.log`，再通过 IPC 把结果传到渲染进程，验证通信链路。
4. 从阶段一就开始关闭不必要的权限，默认启用 `contextIsolation` 与沙箱。
5. 在阶段三用真实文件与真实数据做读写，观察系统目录下的落盘结果。

## 常见排错顺序

```text
安装与启动
  -> 入口文件与 main 字段
  -> 窗口白屏或加载失败
  -> preload 未生效
  -> IPC 通道名不匹配
  -> contextIsolation / sandbox 冲突
  -> 系统路径与权限
  -> 打包与签名失败
```

## 阶段项目路线

从 [综合实战：桌面笔记应用](./project-practice.md) 开始，每完成一个阶段就增加一个能力：窗口、安全 IPC、系统集成、文件存储与打包发布。

## 完成标准

- 能初始化项目并运行窗口。
- 能用 preload + contextBridge 安全暴露 API。
- 能完成主渲染双向 IPC。
- 能实现菜单、托盘与系统通知。
- 能读写文件并持久化数据。
- 能打包出安装包并接入自动更新。
- 完成 [学习评估](./learning-assessment.md) 中的阶段考核与项目评分。
