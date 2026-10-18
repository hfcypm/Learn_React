# iOS 应用开发学习路线（Objective-C 老项目维护 → 实战）

**目标**：以 Objective-C + UIKit 为主线，掌握接手、维护、改造既有 iOS 老工程所需的完整链路——Objective-C 语法与内存管理、UIKit 与 Storyboard/XIB、网络与持久化、GCD 并发、CocoaPods、OC/Swift 混编、测试、崩溃排查与上架发布。

本路线面向**快速入门 + 老项目维护**：先读懂并跑通一个既有工程，再逐步改造，不要求你先学 SwiftUI 或重写项目。

## 先看这里

- [新手导读](./start-here.md)：前置条件、学习顺序、工程心智模型和统一排错顺序。
- [综合实战：维护老版习惯打卡 App](./project-practice.md)：按阶段改造同一个 Objective-C 工程，从跑通到上架。
- [学习评估与项目评分](./learning-assessment.md)：阶段考核、排错题、项目评分和评审问题。
- [版本边界与迁移](./version-governance.md)：iOS / Xcode / ARC / SceneDelegate 的代际差异。

## 快速入门路径（3 周）

| 时间 | 内容 | 阶段产出 |
|---|---|---|
| 第 1-2 天 | 阶段零：Objective-C 与 Xcode | 跑通工程，读懂 AppDelegate 与目录结构 |
| 第 3-7 天 | 阶段一：UIKit 与 Storyboard | 能改界面、加页面、连通列表与跳转 |
| 第 2 周 | 阶段二：数据、网络与并发 | 网络请求 + 本地持久化 + 主线程更新 |
| 第 3 周 | 阶段三 + 综合实战 | 能定位崩溃、补测试、签名上 TestFlight |

时间紧张时按此顺序推进；有余力再回头补阶段三的架构改造与性能排查。

## 学习阶段总览

| 阶段 | 名称 | 目标 |
|---|---|---|
| 零 | Objective-C 语言与 Xcode 工程 | 掌握 OC 语法、属性与 ARC，读懂老工程 |
| 一 | UIKit 与 Storyboard/XIB | 用控制器、约束、列表与导航改造界面 |
| 二 | 数据、网络、持久化与并发 | 用 NSURLSession 取数，用 Core Data 落地 |
| 三 | 老工程维护、测试与发布 | 分层改造、CocoaPods、混编、排查与上架 |

## 快速导航

- [阶段零：Objective-C 语言与 Xcode 工程](./stage-0-foundations.md)
- [阶段一：UIKit 与 Storyboard/XIB](./stage-1-uikit.md)
- [阶段二：数据、网络、持久化与并发](./stage-2-data-network.md)
- [阶段三：老工程维护、测试与发布](./stage-3-engineering-release.md)
- [综合实战：维护老版习惯打卡 App](./project-practice.md)
- [学习评估与项目评分](./learning-assessment.md)
- [版本边界与迁移](./version-governance.md)

## 架构主线

```text
UIApplication
  -> AppDelegate / SceneDelegate（生命周期）
      -> UIWindow
          -> UINavigationController / UITabBarController
              -> UIViewController（Storyboard / XIB / 纯代码）
                  -> View（Auto Layout）
                  -> Model（NSObject 子类）
                      -> Service（NSURLSession / Core Data / Keychain）
```

控制器负责页面生命周期与交互，View 负责布局与展示，Model 负责数据，Service 封装网络与持久化。老工程常见问题是控制器过重，改造时把网络、缓存、格式化逐步下沉到 Service 与 Model。

## 学习原则

- 界面主线用 UIKit + Storyboard/XIB，与老工程保持一致；新页面可以逐步改为 XIB 或纯代码。
- 内存管理默认 ARC，遇到 `-fno-objc-arc` 的老文件先识别再改造。
- 属性优先 `nonatomic`，对象用 `strong`，字符串/集合/block 用 `copy`，代理用 `weak`。
- 网络与数据库回调统一回主线程再更新 UI。
- 依赖优先 CocoaPods；新引入的库再评估 SPM。
- 改动老代码先补测试或可复现步骤，再动手重构。
- 每引入一个系统能力，在 `Info.plist` 与 `PrivacyInfo.xcprivacy` 补齐权限与用途说明。

## 阶段成果

- 能解释 Objective-C、UIKit、Xcode、CocoaPods、Core Data 各自的定位。
- 能用 OC 写出属性正确、内存安全、block 无循环引用的代码。
- 能用 UIKit、Auto Layout、UITableView 与导航控制器改造界面。
- 能用 NSURLSession 请求网络，用 Core Data / NSUserDefaults / Keychain 持久化。
- 能处理 Core Data 轻量迁移与后台上下文写入，并接入本地通知。
- 能用 GCD / NSOperationQueue 处理并发，避免主线程卡顿。
- 能拆分 Massive ViewController，接入 CocoaPods 与 OC/Swift 混编。
- 能用 LLDB、Instruments、View Debugger、静态分析与命令行构建排查问题。
- 能定位崩溃与内存问题，配置签名并完成 TestFlight 与 App Store 提交。

## 版本边界

学习主线面向仍在维护的 **Objective-C + UIKit** 工程，兼容 iOS 12 及以上；使用 `SceneDelegate` 的工程以 iOS 13 及以上为基线。新项目若从零开始，官方推荐 Swift + SwiftUI，本路线聚焦存量工程维护与渐进式改造，迁移策略参见 [版本边界与迁移](./version-governance.md)。

## 前置知识

这是一条移动端原生开发路线，与仓库里的 Web / 桌面 / 鸿蒙主线并行。建议具备：

- C 语言基础：指针、结构体、函数、编译与链接的基本概念。
- 面向对象基础：类、继承、封装、多态、协议/接口。
- 命令行、Git 与依赖管理的基本概念。

有 [TypeScript 学习路线](../typescript-learning-path/README.md) 或 [React 学习路线](../react-learning-path/README.md) 基础时，理解 MVC、组件化与状态驱动会更快；有 [HarmonyOS 学习路线](../harmonyos-learning-path/README.md) 基础的读者，可以对照 UIKit 与 ArkUI 的页面生命周期与状态管理。
