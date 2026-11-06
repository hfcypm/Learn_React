# iOS 应用开发学习路线（入门 → 进阶 → 实战）

**目标**：掌握 iOS 原生应用开发的完整链路。新项目主线是 Swift + SwiftUI：语言与工程、声明式 UI、导航与数据流、网络与持久化、架构测试与上架、系统能力进阶。学完后能独立交付一款可上架的 iOS 应用。存量 Objective-C + UIKit 工程走维护支线。

## 先看这里

- [新手导读](./start-here.md)：前置条件、学习顺序、工程心智模型和统一排错顺序。
- [综合实战：习惯打卡 App](./swift-project-practice.md)：按阶段从零扩展同一个 SwiftUI 项目，从 Hello World 到上架。
- [学习评估与项目评分](./learning-assessment.md)：阶段考核、排错题、项目评分和评审问题。
- [版本边界与迁移](./version-governance.md)：iOS / Xcode / SwiftUI / SwiftData 的代际差异。

老工程接手请改走 [Objective-C 维护支线](#老工程维护支线objective-c--uikit)。

## 学习阶段总览

| 阶段 | 名称 | 目标 |
|---|---|---|
| 入门零 | Swift 语言与 Xcode 工程 | 掌握 Swift 语法、App 生命周期，跑通第一个工程 |
| 入门一 | SwiftUI 声明式 UI 与状态 | 用视图、修饰符和状态驱动构建可交互界面 |
| 入门二 | 导航、生命周期与数据流 | 掌握 NavigationStack、Sheet、Tab 与单向数据流 |
| 进阶三 | 网络、持久化与并发 | 用 URLSession、SwiftData、Swift Concurrency 落地数据 |
| 进阶四 | 架构、测试、性能与发布 | 分层、SPM、测试、隐私清单与 TestFlight |
| 进阶五 | 系统能力专题 | Widget、App Intents、Live Activity、后台、本地化与 UIKit 互操作 |
| 实战 | 习惯打卡 App | 把入门到进阶串成一个可上架项目 |

## 快速入门路径（4 周）

| 时间 | 内容 | 阶段产出 |
|---|---|---|
| 第 1-3 天 | 阶段零：Swift 与 Xcode | 跑通工程，读懂 `App` / `WindowGroup` |
| 第 4-7 天 | 阶段一：SwiftUI | 一个多状态可交互的列表页 |
| 第 2 周 | 阶段二：导航与数据流 | 列表 -> 详情 -> 编辑，能返回刷新 |
| 第 3 周 | 阶段三：网络与持久化 | 本地 SwiftData + 网络同步骨架 |
| 第 4 周 | 阶段四 + 综合实战 MVP | 能测试、签名、上 TestFlight |

时间充裕时继续阶段五与实战进阶项。

## 快速导航

**入门**

- [阶段零：Swift 语言与 Xcode 工程](./swift-stage-0-foundations.md)
- [阶段一：SwiftUI 声明式 UI 与状态](./swift-stage-1-swiftui.md)
- [阶段二：导航、生命周期与数据流](./swift-stage-2-navigation.md)

**进阶**

- [阶段三：网络、持久化与并发](./swift-stage-3-data-network.md)
- [阶段四：架构、测试、性能与发布](./swift-stage-4-engineering.md)
- [阶段五：系统能力专题](./swift-stage-5-advanced.md)

**实战**

- [综合实战：习惯打卡 App](./swift-project-practice.md)
- [学习评估与项目评分](./learning-assessment.md)
- [版本边界与迁移](./version-governance.md)

## 架构主线

```text
App (@main)
  -> WindowGroup / Scene
      -> NavigationStack / TabView
          -> View（SwiftUI 声明式界面）
              -> @Observable / @State / @Binding / @Environment
                  -> Model（struct / @Model）
                      -> Service（URLSession / SwiftData / Keychain）
```

`App` 负责入口与场景，`View` 描述界面，状态装饰器决定数据从哪来、谁能改、刷新谁。Model 保存领域数据，Service 封装网络、持久化与系统能力。视图只做绑定与展示。

## 每阶段固定学习模板

每个阶段按照“目标 -> 前置知识 -> 核心问题 -> 最小示例 -> 项目增量 -> 验收 -> 排错”执行。先在模拟器跑通最小示例，再把能力加进综合实战，一次只增加一个能力。

## 学习原则

- 新项目用 Swift + SwiftUI，Deployment Target 以 iOS 17 为基线，状态管理优先 `@Observable`。
- 导航优先 `NavigationStack` + `navigationDestination`，列表优先 `List` / `ForEach` 稳定 `id`。
- 网络与数据库回调统一回到主线程；异步用 `async/await`，共享可变状态用 `actor` 或 `@MainActor`。
- 每引入一个系统能力，在 `Info.plist` 与 `PrivacyInfo.xcprivacy` 补齐权限与用途说明。
- 改动先补测试或可复现步骤，再扩展功能。

## 阶段成果

- 能解释 Swift、SwiftUI、Xcode、SwiftData、SPM 各自的定位。
- 能用 Swift 写出类型安全、可选值处理正确、无数据竞争的代码。
- 能用 SwiftUI 组件、布局和状态驱动构建可交互界面。
- 能组织 `NavigationStack`、Sheet、Tab 与页面间传值。
- 能封装网络层，用 SwiftData / AppStorage / Keychain 持久化数据。
- 能用 Swift Concurrency 处理并发，避免主线程卡顿。
- 能拆分视图与数据层，接入 SPM 与单元测试。
- 能用 Instruments 定位卡顿与内存问题，完成签名打包与 TestFlight。
- 能开发 Widget、App Intent，处理本地化、深色模式与 UIKit 互操作。

## 版本边界

学习主线面向 **Swift 5.9+ / iOS 17+ / Xcode 15+** 的新项目开发，使用 Observation（`@Observable`）与 SwiftData。iOS 16 及更早的 `ObservableObject` / Core Data / `NavigationView` 作为对照，迁移策略参见 [版本边界与迁移](./version-governance.md)。

存量 Objective-C + UIKit 工程走下方维护支线，兼容 iOS 12 及以上。

## 前置知识

这是移动端原生开发路线，与仓库里的 Web / 桌面 / 鸿蒙主线并行。建议具备：

- 一门静态类型语言基础：变量、函数、结构体/类、可选值、异步。
- 面向对象与值类型的基本概念。
- 命令行、Git 与依赖管理的基本概念。

有 [TypeScript 学习路线](../typescript-learning-path/README.md) 或 [React 学习路线](../react-learning-path/README.md) 基础时，理解组件化与状态驱动会更快；有 [HarmonyOS 学习路线](../harmonyos-learning-path/README.md) 基础的读者，可以把 SwiftUI 对照 ArkUI，把 `App` 生命周期对照 UIAbility。

## 老工程维护支线（Objective-C + UIKit）

接手既有 OC 工程时使用这条支线：先跑通，再按界面 -> 数据 -> 架构改造。

| 阶段 | 名称 | 文档 |
|---|---|---|
| 预备 | C 语言基础 | [附录：C 语言基础](./c-foundations.md) |
| 零 | Objective-C 与 Xcode | [阶段零](./stage-0-foundations.md) |
| 一 | UIKit 与 Storyboard/XIB | [阶段一](./stage-1-uikit.md) |
| 二 | 数据、网络、持久化与并发 | [阶段二](./stage-2-data-network.md) |
| 三 | 维护、测试与发布 | [阶段三](./stage-3-engineering-release.md) |
| 实战 | 维护老版习惯打卡 App | [综合实战](./project-practice.md) |

OC 支线的 3 周快速路径、属性规则与排错顺序写在 [新手导读](./start-here.md) 的「老工程维护」一节。

## 官方资源

- [Apple Developer](https://developer.apple.com/)
- [Swift 语言指南](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/)
- [SwiftUI 文档](https://developer.apple.com/documentation/swiftui/)
- [SwiftData 文档](https://developer.apple.com/documentation/swiftdata)
- [Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/)
