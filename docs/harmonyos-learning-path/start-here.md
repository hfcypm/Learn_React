# HarmonyOS 新手导读

## 适合谁

这套路线适合有一定编程基础、准备进入鸿蒙原生应用开发的工程师。你不需要有 Android 经验——HarmonyOS NEXT 已经去掉了 AOSP 兼容层，不再运行 Android APK，语言、UI 框架和工具链都是新的。学习重点是：理解纯血鸿蒙的运行时与工程结构，用 ArkTS 写类型安全的代码，用 ArkUI 的状态管理把数据驱动到 UI。

如果你熟悉 TypeScript 与 React，ArkTS 的语法与 ArkUI 的组件化、状态驱动心智会迁移得很快；如果你来自 Android 原生开发，请主动放下 `Activity` / `XML 布局` / `Gradle` 的旧习惯，对照本路线的 Stage 模型与 Hvigor 构建重新建立映射。

## 开始前准备

- 掌握一门静态类型语言基础：变量、函数、类、接口、泛型、异步（`Promise` / `async-await`）。
- 能使用终端、Git，理解依赖管理与构建的基本概念。
- 注册华为开发者账号并完成实名认证（真机调试与上架需要）。
- 安装 DevEco Studio 最新稳定版，SDK 已内嵌，安装时自动配置。

```bash
# 安装与项目路径不要包含中文和空格，否则 SDK 加载或构建会失败
# 首次启动 DevEco Studio 会自动下载 SDK 组件与工具链，保持网络畅通
# 打开 Settings -> SDK 确认 Toolchains、System Image 等组件齐全
# 如需本地模拟器，在 SDK 页下载对应设备的系统镜像
```

命令行工具（DevEco Studio 自带）：

```bash
node --version      # DevEco 内置 Node
ohpm --version      # 鸿蒙包管理器，类似 npm
hvigorw --version   # 鸿蒙构建工具
```

## 核心观念

HarmonyOS NEXT 是面向全场景的分布式操作系统。应用开发从这里出发：

- **ArkTS**：开发语言，是 TypeScript 的超集，扩展了声明式 UI 能力，同时收紧动态特性（严格模式下禁止 `any`、限制运行时动态改对象结构）。
- **ArkUI**：声明式 UI 框架，组件化、状态驱动，与 Flutter / Compose 同代。
- **Stage 模型**：应用以 UIAbility 为入口，配合窗口与页面组织运行时。
- **Kit 化 API**：系统能力按 Kit 组织（`@kit.ArkUI`、`@kit.ArkData`、`@kit.NetworkKit`、`@kit.AbilityKit` 等）。

一句话：工具只装一个 DevEco Studio，语言只学 ArkTS，UI 只用 ArkUI，不用在旧 Android 知识上犹豫。

## 学习顺序

| 阶段 | 先回答的问题 | 阶段产出 |
| --- | --- | --- |
| 零 | 纯血鸿蒙解决什么，工程由哪些文件组成？ | 跑通 Hello World，读懂工程结构 |
| 一 | ArkUI 如何用状态驱动 UI？组件和布局怎么组合？ | 一个多状态可交互的页面 |
| 二 | UIAbility 生命周期、窗口和页面栈如何协作？ | 多页面带传参的导航应用 |
| 三 | 数据从网络到本地怎么落地，代码怎么拆模块？ | 网络层 + RDB 缓存 + HAR/HSP |
| 四 | 如何并发、跨设备协同、调优并上架？ | 可多设备运行、可上架的应用 |

## 工程心智模型

```text
AppScope/app.json5          应用级配置（包名、图标、版本）
entry/                      入口 HAP 模块
  src/main/
    module.json5            模块配置（Ability、权限、页面）
    ets/
      entryability/         UIAbility 入口
      pages/                @Entry 页面
    resources/              资源目录（$r 引用）
build-profile.json5         构建目标与产物配置
oh-package.json5            依赖声明（类似 package.json）
```

AppScope 描述整个应用，entry 是入口模块，`module.json5` 声明能力与权限，`ets/pages` 放页面，`resources` 放可被 `$r` 引用的资源。

## 常见排错顺序

```text
构建失败 / SDK 报错
  -> 安装与项目路径是否含中文、空格
  -> DevEco Studio 与 SDK 版本是否配套
  -> oh-package 依赖是否安装（ohpm install）
页面白屏 / 找不到页面
  -> pages 是否在 module.json5 的 pages 列表注册
  -> @Entry 是否标注在入口组件
  -> 路由名称与跳转参数是否匹配
界面不刷新
  -> 状态变量是否加了正确的装饰器（V2 优先）
  -> 是否单组件内混用了 V1 与 V2 装饰器
  -> 是否修改了未被观察的嵌套属性（需 @ObservedV2 + @Trace）
能力调用被拒
  -> 是否在 module.json5 声明权限
  -> 敏感权限是否做了运行时动态申请
  -> 是否使用了正确的 Kit 导入路径
真机连不上
  -> 开发者选项与 USB 调试是否开启
  -> 设备是否已签名（自动签名或手动配置证书）
  -> hdc list targets 是否能看到设备
```

## 阶段项目路线

从 [综合实战：多设备记账元服务](./project-practice.md) 开始，每完成一个阶段就增加一个能力：跑通工程、构建界面、接入导航、落地网络与数据、协同与发布。每个代码片段都标明上下文，避免把孤立片段误当成完整项目。

## 完成标准

- 能解释 HarmonyOS NEXT、ArkTS、ArkUI、Stage 模型、Kit 之间的关系。
- 能创建工程并读懂 app.json5 / module.json5 / build-profile.json5。
- 能用 ArkUI 状态管理（V2 优先）实现数据驱动界面。
- 能组织 UIAbility 生命周期并用 Navigation 完成多页面导航与传参。
- 能封装网络层并用 Preferences / RDB 持久化数据。
- 能拆出 HAR/HSP 模块，用 TaskPool / Worker 处理并发。
- 能完成多设备自适应、性能调优与签名上架。
- 完成 [学习评估](./learning-assessment.md) 中的阶段考核与项目评分。
