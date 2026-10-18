# iOS 新手导读（Objective-C 老项目维护）

## 适合谁

这套路线适合有编程基础、需要接手或维护既有 iOS 老工程的工程师。主线是 **Objective-C + UIKit + Storyboard/XIB**，不要求先学 SwiftUI，也不要求重写项目。

如果你熟悉 C 或 Java，Objective-C 的语法（`@interface`、消息发送、指针）会需要几天适应；如果你熟悉前端框架，请把 `UIViewController` 理解为“页面 + 生命周期”，把 `Storyboard` 理解为可视化布局文件，把 Delegate/Block 理解为回调。

## 开始前准备

- 一台 macOS 电脑（iOS 开发必须在 macOS 上进行）。
- 安装 Xcode（建议与工程当前使用的版本接近，老工程升级 IDE 需谨慎）。
- 一个 Apple ID。免费账号可模拟器与真机调试；上架需加入 Apple Developer Program。
- 能运行工程：拿到源码、`Podfile` 与依赖说明，能执行 `pod install`。

```bash
# 确认开发环境
xcode-select -p
xcodebuild -version
clang --version

# 首次运行需同意许可
sudo xcodebuild -license accept

# CocoaPods（按团队版本要求安装）
pod --version
```

老工程接手第一步：**先让它跑起来**。确认 Xcode 版本、Deployment Target、依赖管理方式（CocoaPods / SPM / 手动拖入）、证书与 Bundle Identifier，再谈改代码。

## 核心观念

- **Objective-C**：C 的超集，面向对象通过消息发送实现，运行时动态性强。
- **UIKit**：命令式 UI 框架，控制器管理页面与生命周期，视图用 Auto Layout 布局。
- **Storyboard / XIB**：可视化界面文件，通过 IBOutlet / IBAction 与代码连接。
- **CocoaPods**：老工程最常见的依赖管理工具，用 `.xcworkspace` 打开工程。
- **Core Data**：对象图与持久化框架，常用于本地数据库。
- **ARC**：自动引用计数，绝大多数现代 OC 文件默认开启。

一句话：先把老工程跑起来、读懂生命周期，再按“界面 -> 数据 -> 架构”的顺序改造。

## 学习顺序

| 阶段 | 先回答的问题 | 阶段产出 |
| --- | --- | --- |
| 零 | OC 的类、属性、ARC 怎么用，工程由哪些文件组成？ | 跑通工程，读懂 AppDelegate 与目录 |
| 一 | UIKit 如何组织页面、布局、列表与跳转？ | 能改界面、加页面、连通列表 |
| 二 | 网络数据怎么取、怎么存、怎么回主线程？ | 网络层 + Core Data 持久化 |
| 三 | 老代码怎么分层、测试、排查与发布？ | 可维护、可上 TestFlight 的工程 |

## 工程心智模型

```text
LegacyHabit/
  LegacyHabit.xcodeproj          工程文件
  LegacyHabit.xcworkspace        使用 CocoaPods 时打开这个
  Podfile / Podfile.lock         依赖声明与锁定
  LegacyHabit/
    AppDelegate.h / .m           应用生命周期（iOS 13 前）
    SceneDelegate.h / .m         场景生命周期（iOS 13+）
    Info.plist                   权限与配置
    Main.storyboard              主界面与入口
    Assets.xcassets              图片、颜色、App 图标
    Controllers/                 视图控制器
    Views/                       自定义视图与 Cell
    Models/                      数据模型
    Services/                    网络、数据库、Keychain
    Categories/                  分类（工具方法）
    Supporting Files/
  LegacyHabitTests/              单元测试
  LegacyHabitUITests/            UI 测试
```

`AppDelegate` 负责应用级事件，`SceneDelegate` 负责窗口与场景（iOS 13+），控制器负责页面，Storyboard 声明界面结构，`Podfile` 声明三方依赖。

## 常见排错顺序

```text
工程打不开 / 编译失败
  -> 是否用 .xcworkspace 打开（CocoaPods 工程）
  -> 是否执行过 pod install，Podfile.lock 是否与 Pods 一致
  -> Xcode 版本与 Deployment Target 是否匹配
  -> 是否缺少 Bridging Header 或库搜索路径
界面空白 / 崩溃在 Storyboard
  -> Main Interface 是否指向正确的 Storyboard
  -> Storyboard 的入口箭头与 Initial View Controller 是否设置
  -> IBOutlet 是否连接到了已删除的控件（旧连接残留）
列表不显示
  -> dataSource / delegate 是否设置
  -> numberOfRows / cellForRow 是否返回正确数据
  -> cell 复用标识符（reuseIdentifier）是否与 Storyboard 一致
点击无响应
  -> IBAction 是否连接，target/action 是否成对
  -> 视图是否被遮挡或 userInteractionEnabled = NO
网络请求失败
  -> ATS 是否允许 http（生产应使用 https）
  -> 是否检查 NSHTTPURLResponse.statusCode
  -> 回调是否回到主线程更新 UI
崩溃 / 内存问题
  -> 查看崩溃日志与符号化（dSYM）
  -> Instruments 的 Leaks / Allocations / Zombies
  -> block 是否强引用 self 导致循环引用
真机跑不起来
  -> Signing Team、Bundle Identifier、证书与描述文件
  -> 设备是否信任开发者证书
```

## 阶段项目路线

从 [综合实战：维护老版习惯打卡 App](./project-practice.md) 开始，每完成一个阶段就改造一块：跑通工程、修复界面与列表、接入网络与持久化、分层与发布。每个代码片段都标明上下文，避免把孤立片段误当成完整项目。

## 完成标准

- 能解释 Objective-C、UIKit、Storyboard、CocoaPods、Core Data 之间的关系。
- 能跑通老工程并读懂 `AppDelegate` / `SceneDelegate` 与目录结构。
- 能写出属性修饰符正确、无循环引用的 OC 代码。
- 能用 UIKit、Auto Layout、UITableView 与导航控制器改造界面。
- 能用 NSURLSession 请求网络，用 Core Data / NSUserDefaults / Keychain 持久化。
- 能处理 Core Data 轻量迁移与本地通知授权。
- 能用 GCD / NSOperationQueue 处理并发并回主线程更新 UI。
- 能拆分 Massive ViewController，接入 CocoaPods 与 OC/Swift 混编。
- 能定位崩溃与内存问题，配置签名并完成 TestFlight 提交。
- 完成 [学习评估](./learning-assessment.md) 中的阶段考核与项目评分。
