# HarmonyOS 学习评估与项目评分

**目标**：通过概念、编码、排错和项目评审确认学习者是否真正掌握了鸿蒙的 Stage 模型、ArkTS/ArkUI、状态管理、数据链路与上架流程，避免只记住装饰器和 API 名称。

## 1. 阶段考核

### 阶段零：基础与工具链

- 解释纯血鸿蒙与传统 Android 在语言、UI、模型、构建上的差异。
- 说明 app.json5、module.json5、build-profile.json5、oh-package.json5 各自职责。
- 用 ArkTS 写出类型安全、符合严格模式的代码（禁用 any）。
- 用 $r 引用资源，用 ohpm / hvigorw 完成依赖与构建。

### 阶段一：ArkUI 声明式 UI 与状态管理

- 解释“状态驱动刷新”的原理。
- 区分并正确选择 V1 与 V2 装饰器，单组件内不混用。
- 用 @Local / @Param / @Event 组织单向数据流。
- 用 @ObservedV2 / @Trace 实现嵌套对象属性级刷新。
- 用 @Computed / @Monitor 处理派生数据与监听。
- 用 @Provider / @Consumer 与 AppStorageV2 管理跨层状态。

### 阶段二：Stage 模型、Ability 与路由导航

- 画出 UIAbility 生命周期与窗口加载节点。
- 说明 onCreate 与 onWindowStageCreate 的职责差异。
- 说明关键状态为何要在 onBackground 保存。
- 用 Navigation + NavPathStack 完成多页面跳转、传参与返回。
- 用 @Provider / @Consumer 共享 NavPathStack。

### 阶段三：网络、数据持久化与工程化

- 封装类型安全的网络层，处理超时、错误码与加载态。
- 在 Preferences 与 RDB 之间按场景选型。
- 实现“本地缓存优先、网络增量刷新”。
- 声明权限并对敏感权限做运行时申请。
- 按职责拆分 HAP / HAR / HSP。
- 对纯逻辑编写 Hypium 单元测试。

### 阶段四：并发、分布式、性能与发布

- 区分 TaskPool 与 Worker，并用 @Concurrent / @Sendable。
- 解释分布式软总线、分布式数据管理与跨端流转。
- 用断点与 GridRow / GridCol 适配多设备。
- 用 LazyForEach、缓存、@ReusableV2 优化长列表。
- 用 DevEco Profiler 与 Code Scanner 定位问题。
- 完成签名配置、构建 .app 包与上架流程。

### 阶段五：进阶专题

- 解释元服务与传统应用的区别，说明卡片为何是其入口。
- 区分静态 / 动态 / 互动卡片，写出 FormExtensionAbility 生命周期与 postCardAction。
- 列举 ExtensionAbility 常见类型，说明启动与安全规则。
- 解释三种 launchType 的行为差异与 onNewWant / onAcceptWant 的作用。
- 用 AppStartup 拆分启动任务，说明 AbilityStage.onCreate 的定位。
- 用 ArkWeb 完成页面加载与原生-H5 通信。
- 用限定词目录实现多语言、深色模式与多设备资源隔离。
- 使用通知、后台任务与窗口能力，并遵守最小权限与隐私要求。
- 说明加密、DataShare、分布式数据对象与端云一体化的适用场景。

## 2. 排错题

- 安装与项目路径含中文/空格导致 SDK 异常或构建失败。
- 页面未在 main_pages.json 注册或缺少 @Entry 导致白屏。
- 状态变量缺少装饰器或嵌套属性未加 @Trace 导致界面不刷新。
- 单组件内混用 V1 与 V2 装饰器导致编译错误。
- 子组件直接修改 @Param 无效（应通过 @Event）。
- ForEach 键不稳定导致列表错乱。
- 重初始化堆在 onWindowStageCreate 导致冷启动慢。
- 只在 onDestroy 保存状态导致数据丢失。
- Navigation 路由名不匹配或未注册导致找不到页面。
- 复杂对象跨页面传参未序列化导致参数丢失。
- 未销毁 http 客户端导致连接泄漏。
- RDB 并发多写或单条超 2M 导致异常。
- 跨线程对象未 @Sendable 导致并发报错。
- Worker 未 terminate 导致线程常驻。
- 卡片不刷新（updateDuration / postCardAction / updateForm）。
- 静态卡片误用事件（应使用 FormLink）。
- 卡片导入了不支持在卡片中使用的模块。
- singleton 复用时未在 onNewWant 更新数据。
- specified 未复用实例（onAcceptWant 返回 Key 不稳定）。
- 跨应用启动失败（exported 或权限不满足）。
- 深色模式不生效（缺少 dark 限定词资源）。
- 上架因权限与隐私说明不匹配被拒。

## 3. 项目评分

总分 100 分，70 分通过，90 分优秀。参考 [综合实战：多设备记账元服务](./project-practice.md)：

| 维度 | 分值 | 评分重点 |
|---|---:|---|
| 语言与工程结构 | 12 | ArkTS 类型安全、工程与配置规范 |
| UI 与状态管理 | 22 | V2 装饰器选型、数据流、刷新粒度、列表性能 |
| Ability 与导航 | 14 | 生命周期、窗口、Navigation 传参与返回 |
| 数据链路 | 18 | 网络封装、RDB/Preferences、缓存策略 |
| 模块化与并发 | 12 | HAR/HSP 拆分、TaskPool/Worker 使用 |
| 进阶系统能力 | 12 | 服务卡片/元服务、启动框架、通知、ArkWeb、多语言 |
| 多设备、安全与发布 | 10 | 自适应布局、安全隐私、签名打包、上架清单 |

## 4. 项目评审问题

- 纯血鸿蒙相比 Android，为什么语言、UI、模型都要换？
- Stage 模型里 UIAbility、WindowStage、页面各自的边界是什么？
- 状态管理 V1 与 V2 的本质差异是什么，为什么新项目推荐 V2？
- @Param 与 @Link 在数据流方向与性能上有什么差别？
- 为什么 Navigation 优先于 Router？传参为什么要注意序列化？
- UIAbility 生命周期中，哪些回调适合初始化，哪些适合持久化？
- 网络层应集中处理哪些横切职责？
- Preferences、RDB、KVStore 分别适合什么数据？
- HAR 与 HSP 的区别与选型依据是什么？
- TaskPool 与 Worker 如何选择？@Sendable 解决什么问题？
- “一次开发，多端部署”在布局与代码上如何落地？
- 元服务与传统应用的区别是什么，为什么以卡片为入口？
- 静态卡片与动态卡片如何选型？卡片如何与宿主应用交互？
- 三种 launchType 分别适合什么场景？specified 如何决定复用实例？
- 启动框架 AppStartup 相比在 onCreate 中初始化解决了什么问题？
- 上架前必须检查哪些项（含权限、隐私、卡片）？

## 5. 学习记录模板

```markdown
## HarmonyOS 阶段记录

- 本阶段目标：
- 已完成的页面或能力：
- 已通过的验证：
- 遇到的问题：
- 根因分析：
- 修复方式：
- 尚未掌握的概念：
- 下一阶段行动：
```
