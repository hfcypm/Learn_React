# iOS 新手导读（OC + UIKit：入门 → 进阶 → 实战）

## 适合谁

这套路线适合有编程基础、准备学习 iOS 开发的工程师。技术栈是 **Objective-C + UIKit + Core Data + CocoaPods**，从入门、进阶到实战推进，最终交付一款可上架的应用。既适合从零学习的新手，也适合接手存量 OC 工程的维护者。

## 开始前准备

- 一台 macOS 电脑（iOS 开发必须在 macOS 上进行）。
- 安装 Xcode 15 或更高（本路线兼容 iOS 12+，使用 SceneDelegate 的工程以 iOS 13 为基线）。
- 一个 Apple ID。免费账号可模拟器与真机调试；上架需加入 Apple Developer Program。

```bash
# 确认开发环境
xcode-select -p
xcodebuild -version

# 首次运行需同意许可
sudo xcodebuild -license accept
```

新项目第一步：**先让它跑起来**。用 Xcode 新建 `App` 模板（Interface 选 Storyboard，Language 选 Objective-C），选一台模拟器，`Cmd + R` 看到 Hello World。

接手存量工程时同样先让它跑起来：确认 Xcode 版本、Deployment Target、`pod install` 与证书，再谈改造。

## 核心观念

- **Objective-C**：iOS 经典开发语言，C 的超集，消息发送 + 运行时，`.h` 声明接口、`.m` 实现。
- **ARC**：自动引用计数。编译器帮你插入 `retain` / `release`，但循环引用仍需用 `weak` 打破。
- **UIKit**：命令式 UI 框架。控制器搭建界面、处理回调，界面刷新靠手动调用 `reloadData` 等方法。
- **AppDelegate / SceneDelegate**：`AppDelegate` 管启动与全局事件；iOS 13+ 用 `SceneDelegate` 管理界面场景。
- **Core Data**：官方持久化框架，`.xcdatamodeld` 建模，配合 `NSFetchedResultsController` 驱动列表。
- **CocoaPods**：依赖管理，用 `Podfile` 声明、`.xcworkspace` 打开、`Podfile.lock` 锁版本。

一句话：语言学 OC，界面用 UIKit，数据用 Core Data，并发用 GCD，依赖用 CocoaPods。

OC 核心规则：属性优先 `nonatomic`；对象 `strong`；字符串 / 集合 / block 用 `copy`；代理 `weak`；网络回调回主线程更新 UI。

## 学习顺序

| 阶段 | 先回答的问题 | 阶段产出 |
| --- | --- | --- |
| 入门零 | OC 怎么写，工程由哪些文件组成，App 怎么启动？ | 跑通 Hello World，读懂 AppDelegate 与目录 |
| 入门一 | 界面怎么搭、怎么布局、怎么跳转传值？ | 列表 -> 详情 -> 编辑的多页面应用 |
| 进阶二 | 数据从网络到本地怎么落地，并发怎么写？ | 网络层 + Core Data 缓存 |
| 进阶三 | 代码怎么分层、测试、排查与发布？ | 可维护、可上 TestFlight 的工程 |
| 实战 | 如何把以上能力串成一个产品？ | 习惯打卡 App 上架 |

## 工程心智模型

```text
LegacyHabit/
  LegacyHabit.xcworkspace        使用 CocoaPods 时打开这个
  Podfile / Podfile.lock
  LegacyHabit/
    AppDelegate.h / .m           启动与全局事件
    SceneDelegate.h / .m         iOS 13+ 界面场景
    Main.storyboard              界面布局与跳转关系
    Info.plist                   权限与配置
    Assets.xcassets              图片、颜色、App 图标
    Models/                      数据模型
    Controllers/                 控制器
    Views/                       自定义视图与 Cell
    Services/                    网络、持久化、通知
  LegacyHabitTests/              单元测试
```

控制器只做调度与回调绑定，Service 不引用控制器，Model 不依赖界面。

## 常见排错顺序

```text
工程打不开 / 编译失败
  -> 是否用 .xcworkspace 打开（CocoaPods 工程）
  -> Xcode 版本是否支持当前工程
  -> Pod 是否安装成功、版本是否与 Podfile.lock 一致
  -> Deployment Target 是否低于所用 API
Storyboard 启动白屏 / 黑屏
  -> 入口配置是否正确，AppDelegate 与 SceneDelegate 是否配套
  -> IBOutlet 连线是否残留或为 nil
列表滚动后 cell 内容错乱
  -> 复用标识是否正确，prepareForReuse 是否重置状态
网络请求成功但 UI 不更新
  -> 回调是否切回主线程
  -> reloadData 是否在数据变更后被调用
页面退出后内存不释放
  -> block 是否强持 self，代理是否声明为 weak
  -> NotificationCenter / Timer 是否成对注销
真机跑不起来
  -> Signing Team、Bundle Identifier、证书与描述文件
  -> 设备是否信任开发者证书
```

## 阶段项目路线

从 [综合实战：习惯打卡 App](./project-practice.md) 开始，每完成一个阶段就扩展一块：工程与模型、界面与导航、网络与 Core Data、分层与发布。每个代码片段都标明上下文，避免把孤立片段误当成完整项目。

## 完成标准

- 能解释 Objective-C、UIKit、Core Data、CocoaPods 之间的关系。
- 能跑通新工程并读懂 AppDelegate / SceneDelegate 与目录结构。
- 能写出属性修饰符正确、无循环引用、回调规范的 OC 代码。
- 能用 Auto Layout、UITableView、UINavigationController 构建界面与导航。
- 能用 NSURLSession 请求网络并安全解析，用 NSUserDefaults / Core Data / Keychain 分层持久化。
- 能用 GCD 处理并发并在主线程更新 UI。
- 能拆分 Controller / Service / Model，接入 CocoaPods 与 XCTest。
- 能定位崩溃与内存问题，配置签名并完成 TestFlight 提交。
- 完成 [学习评估](./learning-assessment.md) 中的阶段考核与项目评分。
