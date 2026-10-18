# 学习评估与项目评分

**目标**：用阶段考核、排错题与项目评分检验 Objective-C 老工程维护能力，避免“看懂了但改不动”。

## 1. 阶段考核

### 阶段零：Objective-C 与工程

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

## 2. 排错题

给出现象，要求说出排查顺序与至少两个可能原因：

- 工程用 `.xcodeproj` 打开后编译报找不到 AFNetworking。
- Storyboard 启动后白屏。
- `IBOutlet` 在 `viewDidLoad` 中为 nil。
- 列表滚动后 cell 内容错乱。
- `cellForRowAtIndexPath` 崩溃在数组下标。
- 点击按钮没有反应。
- 网络请求成功但 UI 不更新。
- JSON 解析偶发崩溃。
- Core Data 保存后列表不刷新。
- 升级模型后老用户数据丢失。
- 通知授权成功但收不到提醒。
- 页面退出后内存不释放。
- 真机运行提示签名失败。
- 上线后崩溃日志无法符号化。

## 3. 项目评分

总分 100 分，70 分通过，90 分优秀。参考 [综合实战：维护老版习惯打卡 App](./project-practice.md)：

| 维度 | 分值 | 评分重点 |
|---|---:|---|
| OC 语言与内存 | 18 | 属性修饰符、ARC、block 循环引用、生命周期 |
| UIKit 与布局 | 22 | 控制器职责、Auto Layout、列表复用、导航 |
| 网络与持久化 | 20 | 请求封装、错误处理、Core Data、缓存策略 |
| 并发与线程 | 12 | 主线程纪律、GCD/NSOperation、竞态处理 |
| 架构改造 | 13 | 分层、依赖注入、可测试性 |
| 测试与排查 | 8 | XCTest、LLDB、Instruments、崩溃定位 |
| 发布 | 7 | 签名、隐私清单、Archive、TestFlight |

## 4. 项目评审问题

- Objective-C 的消息发送与 C 函数调用有什么本质差异？
- 为什么代理用 `weak`、block 用 `copy`、字符串用 `copy`？
- ARC 下还会出现循环引用吗？常见有哪几类？
- `UIViewController` 的 `viewDidLoad` 与 `viewWillAppear` 分别适合做什么？
- Storyboard、XIB、纯代码布局各自的取舍是什么？
- `UITableView` 的复用机制如何工作，为什么要在 `prepareForReuse` 重置状态？
- delegate、block、通知三种回调方式如何选型？
- NSURLSession 回调在哪个线程？为什么必须回主线程更新 UI？
- Core Data 的上下文有哪些类型，跨线程使用要注意什么？
- Core Data 模型加字段时如何做轻量迁移，复杂迁移怎么办？
- 本地通知的授权、触发与取消分别怎么处理？
- 如何把 Massive ViewController 安全地拆分而不破坏现有功能？
- CocoaPods 与 SPM 在存量工程中的取舍是什么？
- OC 与 Swift 混编的成本与迁移顺序如何控制？
- 崩溃日志符号化依赖什么，为什么必须保留 dSYM？
- 老工程上架前最容易忽略的隐私与权限问题有哪些？

## 5. 学习记录模板

```markdown
## Objective-C 老工程维护记录

- 本阶段目标：
- 已跑通的功能：
- 修改的文件与原因：
- 遇到的崩溃或异常：
- 根因分析：
- 修复方式与验证：
- 尚未理解的概念：
- 下一阶段行动：
```
