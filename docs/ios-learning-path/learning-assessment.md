# 学习评估与项目评分

**目标**：用阶段考核、排错题与项目评分检验 iOS 开发能力。主线检验 Swift + SwiftUI 从入门到上架；OC 支线检验老工程维护，避免“看懂了但改不动”。

主线实战参考 [综合实战：习惯打卡 App](./swift-project-practice.md)。接手存量工程时加考 OC 支线，参考 [维护老版习惯打卡 App](./project-practice.md)。

## 1. 阶段考核（Swift 主线）

### 入门零：Swift 与工程

- 说明 `@main`、`App`、`WindowGroup`、`View.body` 各自职责。
- 写出带可选值、`struct` / `enum` / `protocol` 的类型安全代码。
- 解释值类型与引用类型的拷贝差异。
- 说明 Bundle Identifier 与 Deployment Target 的作用。
- 用 `#Preview` 跑通首个页面。

### 入门一：SwiftUI 与状态

- 解释“状态驱动刷新”的原理。
- 区分 `@State` / `@Binding` / `@Observable` / `@Bindable` 的数据流方向。
- 用 `List` / `ForEach` 展示数据，稳定 `id`，处理删除。
- 拆分子视图，用闭包保持单向数据流。
- 处理空态、加载态与失败态，深色模式与动态字体可用。

### 入门二：导航与数据流

- 画出 `App` -> `scenePhase` -> `View.task` 的生命周期。
- 用 `NavigationStack` + `navigationDestination` 完成列表到详情。
- 用 Sheet 完成新增，用 alert 完成删除确认。
- 用 `@Observable` 环境对象作为单一数据源，返回后列表自动刷新。
- 组织 Tab + 各 Tab 内导航栈。

### 进阶三：网络、持久化与并发

- 用 URLSession + Codable 拉取并解码，检查状态码。
- 实现本地优先、网络刷新的数据链路。
- 用 SwiftData 完成增删改查，Preview 使用内存容器。
- 区分 AppStorage、SwiftData、Keychain 的适用场景。
- 用 `async/await` 与 `@MainActor` 更新 UI。
- 申请通知权限并安排、取消本地通知。

### 进阶四：架构、测试与发布

- 按 Feature / Model / Service 拆分，视图不直接打网络。
- 用协议注入假客户端并写出 XCTest。
- 用 SPM 添加并锁定依赖。
- 用 Instruments 指出一次卡顿或一次泄漏的根因。
- 完成 Archive 并上传 TestFlight。

### 进阶五：系统能力

- 解释 Widget timeline 与 App Group 共享数据。
- 写一个可被快捷指令调用的 App Intent。
- 说明 BGTask 与推送刷新的适用边界。
- 完成中英字符串目录与深色模式预览。
- 用 `UIViewRepresentable` 或 `UIHostingController` 做单向互操作。

## 2. 阶段考核（OC 维护支线）

接手存量工程时使用。C 语言预备不计入总分，任一项答不上来先读 [附录：C 语言基础](./c-foundations.md)。

### 前置：C 语言

- 解释 `&`、`*`、`NULL` 与指针类型的关系。
- 说清栈与堆的区别，以及 `malloc` / `free` 的配对规则。
- 读写 `char *` 字符串，说明 `'\0'` 的作用。
- 看懂结构体定义与按值传递，能识别数组越界与野指针。

### 阶段零：Objective-C 与工程

- 说明 `.h` / `.m` 的职责以及从源码到可执行文件的编译、链接过程。
- 写出一个带指定初始化器、属性修饰符正确的 OC 类。
- 解释 `strong` / `weak` / `copy` / `assign` 的使用场景。
- 用 block 写回调，并用 `__weak` 避免循环引用。
- 说清 `AppDelegate`、`SceneDelegate`、`UIViewController` 的生命周期顺序。
- 用 `.xcworkspace` 打开并运行 CocoaPods 工程。

### 阶段一：UIKit 与 Storyboard

- 新建控制器并完成 IBOutlet / IBAction 连线。
- 用 Auto Layout 完成贴边、居中与自适应。
- 用 UITableView 展示数据，正确处理复用与 `prepareForReuse`。
- 用 `UINavigationController` 完成 push / pop 与模态。
- 用 `UIAlertController` 弹确认框，说清连续 present 的崩溃场景。
- 处理键盘收起与遮挡。
- 用 delegate 或 block 完成反向传值。

### 阶段二：数据、网络与并发

- 用 NSURLSession 发起请求并检查错误与状态码。
- 用 NSJSONSerialization 解析并安全地转成模型。
- 用 GCD 把耗时任务移出主线程，并回主线程更新 UI。
- 用 Core Data 完成增删改查并驱动列表刷新。
- 能处理 Core Data 轻量迁移与后台上下文写入。
- 能申请通知权限并添加、取消本地通知。
- 区分 NSUserDefaults、文件、Core Data、Keychain 的适用场景。
- 设计缓存优先的数据链路。

### 阶段三：维护与发布

- 把 Massive ViewController 拆成 Controller / ViewModel / Service / Model。
- 用 CocoaPods 管理依赖并锁定版本。
- 配置 OC / Swift 混编的 Bridging Header 与 `-Swift.h`。
- 用 XCTest 写出模型与失败分支测试。
- 用 LLDB、Instruments 与 View Debugger 定位崩溃、泄漏与卡顿。
- 能用静态分析与命令行构建接入持续集成。
- 完成签名、Archive 与 TestFlight 上传。

## 3. 排错题

给出现象，要求说出排查顺序与至少两个可能原因。

**Swift 主线**

- Deployment Target 为 iOS 16 时使用 `@Observable` 编译失败。
- Preview 空白或崩溃。
- `TextField` 输入后界面不更新。
- `ForEach` 列表行错乱。
- 点了 `NavigationLink` 不跳转。
- 详情打卡后列表不刷新。
- JSON 解码失败但请求成功。
- `@Query` 始终为空。
- `.task` 里的请求在离开页面后仍回调。
- 主线程卡顿发生在滑动列表时。
- Archive 签名失败。
- 审核被拒隐私清单。
- 小组件不刷新。

**OC 支线**

- 工程用 `.xcodeproj` 打开后编译报找不到 AFNetworking。
- Storyboard 启动后白屏。
- `IBOutlet` 在 `viewDidLoad` 中为 nil。
- 列表滚动后 cell 内容错乱。
- 网络请求成功但 UI 不更新。
- Core Data 保存后列表不刷新。
- 升级模型后老用户数据丢失。
- 页面退出后内存不释放。
- 真机运行提示签名失败。
- 上线后崩溃日志无法符号化。

## 4. 项目评分

总分 100 分，70 分通过，90 分优秀。

### Swift 主线（习惯打卡 App）

| 维度 | 分值 | 评分重点 |
|---|---:|---|
| Swift 语言与类型 | 12 | 可选值、值类型、错误处理 |
| SwiftUI 与状态 | 20 | 属性包装器、列表、空态、单向数据流 |
| 导航与数据流 | 12 | NavigationStack、Sheet、共享 Store |
| 网络与持久化 | 18 | URLSession、SwiftData、缓存优先、Keychain |
| 并发 | 10 | async/await、@MainActor、取消 |
| 架构与测试 | 14 | 分层、协议注入、XCTest |
| 发布 | 8 | 隐私清单、Archive、TestFlight |
| 系统能力（加分可计入此项） | 6 | Widget、Intent、本地化 |

### OC 支线（老版习惯打卡 App）

| 维度 | 分值 | 评分重点 |
|---|---:|---|
| OC 语言与内存 | 18 | 属性修饰符、ARC、block 循环引用、生命周期 |
| UIKit 与布局 | 22 | 控制器职责、Auto Layout、列表复用、导航 |
| 网络与持久化 | 20 | 请求封装、错误处理、Core Data、缓存策略 |
| 并发与线程 | 12 | 主线程纪律、GCD/NSOperation、竞态处理 |
| 架构改造 | 13 | 分层、依赖注入、可测试性 |
| 测试与排查 | 8 | XCTest、LLDB、Instruments、崩溃定位 |
| 发布 | 7 | 签名、隐私清单、Archive、TestFlight |

## 5. 项目评审问题

**Swift 主线**

- Swift 的值类型与引用类型在 SwiftUI 刷新里分别意味着什么？
- 为什么优先 `@Observable` 而不是 `ObservableObject`？
- `@State`、`@Binding`、`@Environment` 如何选型？
- 为什么 `ForEach` 不能用数组下标当 `id`？
- `NavigationStack` 的 `path` 放在哪一层，才能支持深链？
- URLSession 与 `.task` 如何配合取消？
- SwiftData 的 `@Query` 为什么能自动刷新？
- 同一天重复打卡如何做成幂等？
- `@MainActor` 解决了什么问题，什么时候需要 `actor`？
- 如何在不改视图的前提下给网络层写测试？
- Widget 为什么要用 App Group？
- 上架前隐私清单最容易漏哪些 API？

**OC 支线**

- Objective-C 的消息发送与 C 函数调用有什么本质差异？
- 为什么代理用 `weak`、block 用 `copy`、字符串用 `copy`？
- ARC 下还会出现循环引用吗？常见有哪几类？
- `UIViewController` 的 `viewDidLoad` 与 `viewWillAppear` 分别适合做什么？
- `UITableView` 的复用机制如何工作，为什么要在 `prepareForReuse` 重置状态？
- NSURLSession 回调在哪个线程？为什么必须回主线程更新 UI？
- Core Data 模型加字段时如何做轻量迁移？
- 如何把 Massive ViewController 安全地拆分而不破坏现有功能？
- 崩溃日志符号化依赖什么，为什么必须保留 dSYM？

## 6. 学习记录模板

```markdown
## iOS 学习记录

- 本阶段目标：
- 已跑通的功能：
- 修改的文件与原因：
- 遇到的崩溃或异常：
- 根因分析：
- 修复方式与验证：
- 尚未理解的概念：
- 下一阶段行动：
```
