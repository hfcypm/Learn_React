# iOS 新手导读（Swift 入门 → 进阶 → 实战）

## 适合谁

这套路线适合有编程基础、准备从零开发 iOS 应用的工程师。主线是 **Swift + SwiftUI + SwiftData**，按入门、进阶、实战推进，最终交付一款可上架的应用。

熟悉 TypeScript 或 React 时，把 `View` 理解为组件，把 `@State` / `@Observable` 理解为状态，把 `NavigationStack` 理解为路由栈。熟悉 HarmonyOS ArkUI 时，SwiftUI 的声明式刷新与状态装饰器可直接对照。

需要接手既有 Objective-C 工程时，读本文「老工程维护」一节，再进入 OC 支线文档。

## 开始前准备

- 一台 macOS 电脑（iOS 开发必须在 macOS 上进行）。
- 安装 Xcode 15 或更高（学习主线以 iOS 17 SDK 为基线）。
- 一个 Apple ID。免费账号可模拟器与真机调试；上架需加入 Apple Developer Program。

```bash
# 确认开发环境
xcode-select -p
xcodebuild -version
swift --version

# 首次运行需同意许可
sudo xcodebuild -license accept
```

新项目第一步：**先让它跑起来**。用 Xcode 新建 `App` 模板（Interface 选 SwiftUI，Language 选 Swift），选一台模拟器，`Cmd + R` 看到 Hello World。

## 核心观念

- **Swift**：静态类型语言，值类型优先，可选值与协议驱动设计。
- **SwiftUI**：声明式 UI 框架，界面是状态的函数，状态变化后框架差量刷新。
- **App / Scene**：`@main` 的 `App` 是入口，`WindowGroup` 管理窗口场景。
- **Observation**：iOS 17 起用 `@Observable` 做可观察模型，替代 `ObservableObject`。
- **SwiftData**：官方持久化框架，用 `@Model` 声明实体，与 SwiftUI 查询绑定。
- **SPM**：新项目默认依赖管理，用 `Package.swift` 或 Xcode 的 Package Dependencies。

一句话：语言学 Swift，界面用 SwiftUI，数据用 SwiftData，异步用 `async/await`。

## 学习顺序

| 阶段 | 先回答的问题 | 阶段产出 |
| --- | --- | --- |
| 入门零 | Swift 怎么写，工程由哪些文件组成，App 怎么启动？ | 跑通 Hello World，读懂 `App` 与目录 |
| 入门一 | SwiftUI 如何用状态驱动 UI？列表和表单怎么组合？ | 一个多状态可交互的页面 |
| 入门二 | 导航栈、Sheet、Tab 如何协作，数据怎么单向流动？ | 多页面带传参的导航应用 |
| 进阶三 | 数据从网络到本地怎么落地，并发怎么写？ | 网络层 + SwiftData 缓存 |
| 进阶四 | 代码怎么分层、测试、排查与发布？ | 可维护、可上 TestFlight 的工程 |
| 进阶五 | Widget、Intent、后台、本地化怎么做？ | 一张可刷新的小组件 + 系统能力 |
| 实战 | 如何把以上能力串成一个产品？ | 习惯打卡 App 上架 |

## 工程心智模型

```text
HabitTracker/
  HabitTracker.xcodeproj
  HabitTracker/
    HabitTrackerApp.swift        @main 入口与 WindowGroup
    ContentView.swift            根视图
    Info.plist                   权限与配置（部分项在 target 的 Info 面板）
    Assets.xcassets              图片、颜色、App 图标
    PrivacyInfo.xcprivacy        隐私清单
    Models/                      SwiftData @Model 与普通 struct
    Views/                       SwiftUI 页面与子视图
    Services/                    网络、Keychain、通知
    Resources/
  HabitTrackerTests/             单元测试
  HabitTrackerUITests/           UI 测试
```

`HabitTrackerApp` 负责注入 `modelContainer` 与环境对象，页面通过 `@Query` / `@Environment` 取数据，Service 不引用视图。

## 常见排错顺序

```text
工程打不开 / 编译失败
  -> Xcode 版本是否支持当前 Swift / iOS SDK
  -> Package 依赖是否解析成功
  -> Deployment Target 是否低于所用 API
预览空白 / Preview 崩溃
  -> Preview 是否提供了 @Query / @Environment 所需依赖
  -> modelContainer 是否用 in-memory 配置
  -> 预览宏是否写在与 View 相同的文件
界面不刷新
  -> 模型是否标记 @Observable 或 @Model
  -> 变更是否发生在 @MainActor
  -> ForEach 的 id 是否稳定
列表错乱
  -> id 是否用 UUID / 持久主键，避免用数组下标
导航无效
  -> 是否包在 NavigationStack 内
  -> navigationDestination 的类型是否与 value 一致
网络请求失败
  -> ATS 是否允许 http（生产使用 https）
  -> 是否检查 HTTP 状态码
  -> 异步任务是否在视图消失后被取消
真机跑不起来
  -> Signing Team、Bundle Identifier、证书与描述文件
  -> 设备是否信任开发者证书
```

## 阶段项目路线

从 [综合实战：习惯打卡 App](./swift-project-practice.md) 开始，每完成一个阶段就扩展一块：工程与模型、列表与状态、导航、网络与 SwiftData、分层与发布、小组件。每个代码片段都标明上下文，避免把孤立片段误当成完整项目。

## 完成标准

- 能解释 Swift、SwiftUI、SwiftData、SPM 之间的关系。
- 能跑通新工程并读懂 `App` / `WindowGroup` 与目录结构。
- 能写出可选值处理正确、并发安全的 Swift 代码。
- 能用 SwiftUI、`List`、`NavigationStack` 构建界面。
- 能用 URLSession 请求网络，用 SwiftData / AppStorage / Keychain 持久化。
- 能用 `async/await` 与 `@MainActor` 处理并发并更新 UI。
- 能拆分 View / Model / Service，接入 SPM 与 XCTest。
- 能定位崩溃与内存问题，配置签名并完成 TestFlight 提交。
- 完成 [学习评估](./learning-assessment.md) 中的阶段考核与项目评分。

## 老工程维护

接手 Objective-C + UIKit 工程时：

1. 先让它跑起来：Xcode 版本、Deployment Target、`pod install`、证书。
2. 读 [附录：C 语言基础](./c-foundations.md)（指针与内存不熟时）。
3. 按 OC 支线阶段零到阶段三推进，实战用 [维护老版习惯打卡 App](./project-practice.md)。

OC 核心规则：属性优先 `nonatomic`；对象 `strong`；字符串 / 集合 / block 用 `copy`；代理 `weak`；网络回调回主线程更新 UI。

```text
LegacyHabit/
  LegacyHabit.xcworkspace        使用 CocoaPods 时打开这个
  Podfile / Podfile.lock
  LegacyHabit/
    AppDelegate.h / .m
    SceneDelegate.h / .m
    Main.storyboard
    Controllers / Views / Models / Services
```

OC 排错优先看：是否用 `.xcworkspace` 打开、IBOutlet 是否残留、cell 复用标识、ATS、block 循环引用、签名。
