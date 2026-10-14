# HarmonyOS 学习路线图（入门 → 精通实战）

**目标**：掌握鸿蒙原生应用开发的全套能力——ArkTS 语言、ArkUI 声明式 UI 与状态管理、Stage 模型与 Ability、路由导航、网络与数据持久化、模块化工程、并发与分布式协同、性能调优与上架发布。最终能独立交付一款可多设备运行、可上架的鸿蒙应用。

## 先看这里

- [新手导读](./start-here.md)：前置条件、学习顺序、框架心智模型和统一排错顺序。
- [综合实战：多设备记账元服务](./project-practice.md)：按阶段扩展同一个项目，从零到上架。
- [学习评估与项目评分](./learning-assessment.md)：阶段考核、排错题、项目评分和评审问题。

## 学习阶段总览

| 阶段 | 名称 | 目标 |
|---|---|---|
| 零 | HarmonyOS 与 ArkTS 基础 | 理解纯血鸿蒙、工程结构，掌握 ArkTS 严格类型语法 |
| 一 | ArkUI 声明式 UI 与状态管理 | 用组件、布局、状态驱动构建可交互界面 |
| 二 | Stage 模型、Ability 与路由导航 | 掌握 UIAbility 生命周期、Navigation 导航与页面栈 |
| 三 | 网络、数据持久化与工程化 | 封装网络层，掌握 Preferences/RDB、HAR/HSP 与测试 |
| 四 | 并发、分布式、性能与发布 | 掌握 TaskPool/Worker、跨端协同、调优与上架 |

## 快速导航

- [阶段零：HarmonyOS 与 ArkTS 基础](./stage-0-foundations.md)
- [阶段一：ArkUI 声明式 UI 与状态管理](./stage-1-arkui.md)
- [阶段二：Stage 模型、Ability 与路由导航](./stage-2-ability-navigation.md)
- [阶段三：网络、数据持久化与工程化](./stage-3-engineering.md)
- [阶段四：并发、分布式、性能与发布](./stage-4-distributed-production.md)
- [综合实战：多设备记账元服务](./project-practice.md)

## 架构主线

```text
应用(APP)
  -> 模块(HAP / HAR / HSP)
      -> UIAbility(入口能力) -> WindowStage -> 页面(@Entry)
                                                   |
                          Navigation / NavPathStack -> NavDestination
                                                   |
                        @ComponentV2 + 状态管理 -> ArkUI 渲染
                                                   |
                    ServiceKit / NetworkKit / ArkData(Preferences / RDB)
```

UIAbility 负责能力生命周期与窗口，页面负责 UI 组织，状态管理负责数据驱动刷新，Kit 层负责系统能力调用。每一层职责单一，业务逻辑与 UI 解耦。

## 每阶段固定学习模板

每个阶段按照“目标 -> 前置知识 -> 核心问题 -> 最小示例 -> 项目增量 -> 验收 -> 排错”执行。先理解 ArkTS 语言与 Stage 模型的运行时行为，再学习 ArkUI 的封装点，避免只记装饰器名称。

## 学习原则

- 先跑通“Hello World”，再逐层加能力；每个概念都对着一台真机或模拟器验证。
- 状态管理优先使用 V2（`@ComponentV2` / `@Local` / `@Param` / `@ObservedV2` / `@Trace`），单组件内不混用 V1 与 V2 装饰器。
- 路由优先使用 `Navigation` + `NavPathStack`，新项目避免使用已不推荐的 `Router`。
- 网络、数据库、并发默认要处理异常与线程边界，主线程只做 UI。
- 每引入一个系统能力，先在 `module.json5` 声明权限再调用。

## 阶段成果

- 能解释 HarmonyOS NEXT、ArkTS、ArkUI、DevEco Studio 各自的定位。
- 能用 ArkTS 写出类型安全、符合严格模式的代码。
- 能用 ArkUI 组件、布局和状态管理构建可交互界面。
- 能组织 UIAbility 生命周期、页面栈与页面间传参。
- 能封装网络层，用 Preferences / RDB 持久化数据，并拆出 HAR/HSP 模块。
- 能用 TaskPool / Worker 处理并发，在多设备上做协同与自适应布局。
- 能用 DevEco Profiler 定位卡顿与内存问题，并完成签名打包与上架。

## 版本边界

学习主线面向 HarmonyOS NEXT（5.x 起）及以上、API 14+ 的纯血鸿蒙应用开发与新 Stage 模型。旧版 HarmonyOS 3/4（含 AOSP 兼容层、FA 模型、`Router` 路由）与当前 ArkTS/ArkUI 差异较大，参见 [版本边界与迁移](./version-governance.md)。

## 前置知识

这是移动端/全场景应用开发路线，与仓库里的 Web 主线并行。建议已掌握至少一门静态类型语言的基础（TypeScript 最佳），理解变量、函数、类、接口、异步与面向对象。若你已学过 [TypeScript 学习路线](../typescript-learning-path/README.md)，ArkTS 的语法迁移会很顺畅；[React 学习路线](../react-learning-path/README.md) 的组件化与状态驱动心智也可迁移到 ArkUI。

## 官方资源

- [HarmonyOS 开发者官网](https://developer.huawei.com/consumer/cn/)
- [DevEco Studio 下载](https://developer.huawei.com/consumer/cn/deveco-studio/)
- [ArkTS 语言指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-get-started)
- [ArkUI 开发文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkui)
- [API 参考](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/)
- [最佳实践](https://developer.huawei.com/consumer/cn/doc/best-practices/)
