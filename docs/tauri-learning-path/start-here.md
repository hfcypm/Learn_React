# Tauri 新手导读

## 适合谁

这套路线适合已经掌握一个前端框架、想用更小的体积与更强的安全性构建桌面应用的开发者。学习重点是 Tauri 2 的前后端架构、Rust commands、权限系统与打包发布。

## 开始前准备

- 掌握前端三件套与一个框架（React/Vue/Svelte 均可），可参考 [React 学习路线](../react-learning-path/README.md)。
- 了解 TypeScript 与异步编程，可参考 [TypeScript 学习路线](../typescript-learning-path/README.md)。
- 具备 Rust 基础语法（变量、函数、结构体、错误处理），可边学边补。
- 安装 Node.js、npm 与 Rust 工具链。

```bash
node --version
npm --version
rustc --version
cargo --version
```

## 学习顺序

| 阶段 | 先回答的问题 | 阶段产出 |
| --- | --- | --- |
| 零 | 如何初始化并运行第一个窗口？ | 可运行的最小桌面应用 |
| 一 | 如何配置应用并控制权限？ | 配置清晰、权限收敛的应用 |
| 二 | 如何让前端调用 Rust 能力？ | 类型安全的前后端连接 |
| 三 | 如何访问文件与系统能力？ | 有真实功能的桌面应用 |
| 四 | 如何打包发布并更新？ | 可安装的生产产物 |

## 每阶段学习方法

1. 每个概念先写最小命令并实际运行，观察返回值。
2. 在浏览器与 `src-tauri/target` 之间切换观察前端调试与 Rust 编译日志。
3. 从阶段一就按需声明权限，不提前放开能力。
4. 用 `tauri build` 观察最终产物体积，体会 Tauri 的优势。
5. 对照 Electron 文档理解两种框架的异同。

## 常见排错顺序

```text
安装与初始化
  -> Rust 工具链与依赖
  -> 编译错误
  -> 命令未注册
  -> invoke 参数不匹配
  -> 权限未声明
  -> 插件未安装
  -> 打包与签名失败
```

## 阶段项目路线

从 [综合实战：跨平台文件管理器](./project-practice.md) 开始，每完成一个阶段就增加一个能力：窗口、权限配置、命令后端、系统能力与打包发布。

## 完成标准

- 能初始化 Tauri 2 项目并运行窗口。
- 能配置 tauri.conf.json 与 capabilities。
- 能用 Rust commands 暴露后端能力。
- 能用插件完成文件、对话框与系统交互。
- 能打包跨平台产物并接入更新。
- 完成 [学习评估](./learning-assessment.md) 中的阶段考核与项目评分。
