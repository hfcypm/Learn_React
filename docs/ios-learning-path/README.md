# iOS 应用开发学习路线（Objective-C + UIKit：入门 → 进阶 → 实战）

**目标**：掌握 iOS 原生应用开发的完整链路：Objective-C 语言与 Xcode 工程、UIKit 界面、数据网络与并发、工程化与发布。既适合从零学习 OC + UIKit 的新手，也适合接手存量工程的维护者。学完能独立交付一款可上架的 iOS 应用。

## 先看这里

- [新手导读](./00-start-here.md)：前置条件、学习顺序、工程心智模型和统一排错顺序。
- [综合实战：习惯打卡 App](./05-project-practice.md)：从零构建或接手维护，从 Hello World 到上架。
- [学习评估与项目评分](./06-learning-assessment.md)：阶段考核、排错题、项目评分和评审问题。
- [版本边界与升级](./07-version-governance.md)：iOS / Xcode / CocoaPods / Core Data 的版本差异与升级检查。

## 学习阶段总览

| 阶段 | 名称 | 目标 |
|---|---|---|
| 入门零 | Objective-C 与 Xcode 工程 | 掌握 OC 类、属性、内存管理与生命周期，跑通第一个工程 |
| 入门一 | UIKit 与 Storyboard/XIB | 用控制器、Auto Layout、UITableView 构建界面并传值 |
| 进阶二 | 数据、网络、持久化与并发 | 用 NSURLSession、Core Data、GCD 落地数据 |
| 进阶三 | 工程化、测试与发布 | 分层、CocoaPods、测试、崩溃定位与签名发布 |
| 实战 | 习惯打卡 App | 把入门到进阶串成一个可上架项目 |

## 快速入门路径（5 周）

| 时间 | 内容 | 阶段产出 |
|---|---|---|
| 第 1 周 | 阶段零：OC 与 Xcode | 跑通 Hello World，读懂类 / 属性 / 内存管理 |
| 第 2 周 | 阶段一：UIKit 界面 | 列表页 + 详情页，Auto Layout 可用 |
| 第 3 周 | 阶段一后半：导航与多页面传值 | 列表 -> 详情 -> 编辑，能返回刷新 |
| 第 4 周 | 阶段二：数据、网络与并发 | Core Data 落地 + 网络同步骨架 |
| 第 5 周 | 阶段三 + 实战 MVP | 可测试、签名、上 TestFlight |

时间充裕时继续实战进阶项与存量改造专题。

## 快速导航

**入门**

- [阶段零：Objective-C 语言与 Xcode 工程](./01-stage-0-foundations.md)
- [阶段一：UIKit 与 Storyboard/XIB](./02-stage-1-uikit.md)

**进阶**

- [阶段二：数据、网络、持久化与并发](./03-stage-2-data-network.md)
- [阶段三：工程化、测试与发布](./04-stage-3-engineering-release.md)

**实战**

- [综合实战：习惯打卡 App](./05-project-practice.md)
- [学习评估与项目评分](./06-learning-assessment.md)
- [版本边界与升级](./07-version-governance.md)

## 架构主线

```text
AppDelegate / SceneDelegate（入口与生命周期）
  -> UIWindow / UIWindowScene
      -> UINavigationController / UITabBarController
          -> UIViewController（命令式界面与回调）
              -> Model（NSObject 数据模型）
                  -> Service（NSURLSession / Core Data / NSUserDefaults / Keychain）
```

`AppDelegate` 负责启动与全局事件，iOS 13+ 的工程用 `SceneDelegate` 管理界面场景。`UIViewController` 命令式地搭建与刷新界面，Model 保存领域数据，Service 封装网络、持久化与系统能力。控制器只做调度，不直接拼 URL、不直接操作数据库。

## 每阶段固定学习模板

每个阶段按照“目标 -> 前置知识 -> 核心问题 -> 最小示例 -> 项目增量 -> 验收 -> 排错”执行。先在模拟器跑通最小示例，再把能力加进综合实战，一次只增加一个能力。

## 学习原则

- 属性修饰符默认规则：默认 `nonatomic`；对象用 `strong`；字符串 / 集合 / block 用 `copy`；代理与父对象用 `weak`；基本类型用 `assign`。
- 所有 UI 更新回到主线程；耗时任务用 GCD 移出主线程。
- 使用 CocoaPods 的工程用 `.xcworkspace` 打开；依赖版本用 `Podfile.lock` 锁定。
- 每引入一个系统能力，在 `Info.plist` 与 `PrivacyInfo.xcprivacy` 补齐权限与用途说明。
- 改动先补测试或可复现步骤，再扩展功能。

## 阶段成果

- 能解释 Objective-C、Xcode、UIKit、Core Data、CocoaPods 各自的定位。
- 能写出属性修饰符正确、ARC 下无循环引用、block 使用规范的 OC 代码。
- 能用控制器、Auto Layout、UITableView、UINavigationController 构建多页面应用。
- 能封装网络层并安全解析，处理错误分支与缓存降级。
- 能用 Core Data 完成增删改查与轻量迁移，驱动列表刷新。
- 能用 GCD / NSOperation 处理并发，避免主线程卡顿。
- 能拆分 Controller / ViewModel / Service / Model，接入 CocoaPods 与 XCTest。
- 能用 Instruments 与崩溃日志定位问题，完成签名打包与 TestFlight。

## 版本边界

本路线兼容 **iOS 12 及以上**；使用 SceneDelegate 的工程以 **iOS 13** 为基线。各系统版本差异、MRC/ARC、Core Data 迁移与升级检查，参见 [版本边界与升级](./07-version-governance.md)。

## 前置知识

这是移动端原生开发路线，与仓库里的 Web / 桌面 / 鸿蒙主线并行。建议具备：

- 一门编程语言基础：变量、函数、类与对象、流程控制。
- 面向对象的基本概念：封装、继承、多态。
- 命令行、Git 与依赖管理的基本概念。
- 有 Java 背景？各阶段文档开头内置「Java 对照速查」，总表与心智差异见 [新手导读](./00-start-here.md)。

OC 的指针用法与内存管理会在阶段零结合实例讲解。有 [HarmonyOS 学习路线](../harmonyos-learning-path/README.md) 基础的读者，可以把 `UIViewController` 生命周期对照 UIAbility。

## 官方资源

- [Apple Developer](https://developer.apple.com/)
- [Programming with Objective-C](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Introduction/Introduction.html)
- [UIKit 文档](https://developer.apple.com/documentation/uikit)
- [Core Data 文档](https://developer.apple.com/documentation/coredata)
- [Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/)
