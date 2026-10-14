# 阶段五：进阶专题（元服务 / 卡片 / 系统能力）

**目标**：补齐从“会写应用”到“精通鸿蒙”的关键专题——元服务与服务卡片、ExtensionAbility 家族、启动模式与启动框架、ArkWeb、资源与多语言、通知/后台/窗口、安全隐私与数据管理扩展。

> 适用范围：HarmonyOS NEXT（API 14+）+ ArkTS + ArkUI
> 需要的工具：DevEco Studio（配套版本）、ohpm、hvigorw
> 最后复核日期：2026-10-14

## 1. 元服务（Atomic Service）

元服务是鸿蒙的轻量化服务形态，核心是“免安装、即用即走、卡片入口”。

| 维度 | 传统应用（UIAbility 入口） | 元服务 |
|---|---|---|
| 安装方式 | 需下载安装 | 免安装，添加卡片即可使用 |
| 入口形态 | 桌面图标、全屏界面 | 服务卡片，轻交互 |
| 包体积 | 无严格限制 | 严格受限 |
| 更新 | 整包升级 | 卡片可独立更新，秒级生效 |
| 分发 | 应用市场 | 应用市场 + 卡片入口 + 负一屏等 |

工程侧的关键配置：

```json5
// module.json5：元服务标记免安装
"module": {
  "name": "entry",
  "type": "entry",
  "installationFree": true,
  "deviceTypes": ["phone", "tablet"]
}
```

要点：

- 元服务基于元服务 API 集开发，能力与普通应用共享，但受卡片运行环境约束。
- 卡片是主要入口，因此元服务几乎必然要掌握“服务卡片开发”。
- 元服务可独立上架，适合单场景闭环（点单、查询、轻工具）。

## 2. 服务卡片（Form / Form Kit）

服务卡片是提取应用核心信息与操作的轻量展示载体，由 `FormExtensionAbility` 驱动。

### 2.1 卡片类型

| 类型 | 渲染与能力 | 适用场景 | 开销 |
|---|---|---|---|
| 静态卡片 | 系统渲染最后一帧为图片，仅 UI 与布局，靠 `FormLink` 跳转 | 静态信息展示 | 低，但频繁刷新会反复创建销毁 |
| 动态卡片 | 支持通用事件与自定义动效，`postCardAction` 交互 | 需刷新/交互（列表、控制） | 较高 |
| 互动卡片 | 动态卡片基础上支持破框动效（API 20+） | 浅层交互、可玩性场景 | 较高 |

另按开发语言分为 ArkTS 卡片（`eTS`）与 JS 卡片（`JS`，均为动态卡片，能力基础）。工程中推荐 ArkTS 卡片。

### 2.2 form_config.json

```json5
{
  "forms": [
    {
      "name": "ledger_form",
      "description": "本月支出卡片",
      "src": "./ets/widget/pages/LedgerCard.ets",
      "isDefault": true,
      "colorMode": "auto",
      "supportDimensions": ["2x2", "2x4"],
      "updateDuration": 30,
      "isDynamic": true
    }
  ]
}
```

关键字段：`supportDimensions` 支持尺寸、`updateDuration` 定时刷新（分钟）、`isDynamic` 静态/动态、`colorMode` 跟随深浅色、`renderingMode` 色彩模式。

### 2.3 FormExtensionAbility 生命周期

```ts
import { formBindingData, FormExtensionAbility, formInfo, formProvider } from '@kit.FormKit';

export default class LedgerFormAbility extends FormExtensionAbility {
  // 卡片创建：返回初始数据
  onAddForm(want: Want): formBindingData.FormBindingData {
    return formBindingData.createFormBindingData({ total: '¥0.00' });
  }

  // 定时刷新（由 updateDuration 触发）
  onUpdateForm(formId: string): void {
    formProvider.updateForm(formId, formBindingData.createFormBindingData({ total: '¥128.00' }));
  }

  // 卡片内点击 message 事件
  onFormEvent(formId: string, message: string): void {
    formProvider.updateForm(formId, formBindingData.createFormBindingData({ total: '¥256.00' }));
  }

  // 卡片移除
  onRemoveForm(formId: string): void {
    // 释放该卡片相关资源
  }
}
```

### 2.4 卡片交互：postCardAction 与 FormLink

```ts
// 动态卡片：点击按钮触发 message，拉起 FormExtensionAbility 更新
@Entry
@Component
struct LedgerCard {
  @LocalStorageProp('total') total: string = '¥0.00';

  build() {
    Column() {
      Text(`本月支出 ${this.total}`)
      Button('刷新').onClick(() => {
        postCardAction(this, {
          action: 'message',
          params: { message: 'refresh' }
        });
      })
    }
  }
}
```

- `postCardAction`（动态卡片）支持 `router`（跳 UIAbility）、`message`（拉起 FormExtensionAbility）、`call`（拉起 UIAbility 到后台）。
- 静态卡片用 `FormLink` 组件实现同类交互。
- 卡片状态用 `@LocalStorageProp` 接收 `updateForm` 推送的数据。

约束：卡片仅支持 ArkUI 开发，只能导入标注“支持在 ArkTS 卡片中使用”的模块，支持 HAR、不支持 HSP，不支持 Native 与部分 API。

## 3. ExtensionAbility 家族

ExtensionAbility 是应用的扩展能力，与 UIAbility 并列。常见类型：

| 类型 | 用途 |
|---|---|
| `FormExtensionAbility` | 服务卡片 |
| `WorkSchedulerExtensionAbility` | 延迟任务（系统择机调度） |
| `ServiceExtensionAbility` | 后台服务（可被连接/启动） |
| `DataShareExtensionAbility` | 跨应用数据共享 |
| `StaticSubscriberExtensionAbility` | 静态订阅公共事件 |
| `InputMethodExtensionAbility` | 输入法 |
| `AccessibilityExtensionAbility` | 无障碍 |

启动与安全规则（Stage 模型）：

- 跨应用启动组件前，被启动组件 `exported` 必须为 `true`。
- 后台应用拉起其他应用 UIAbility 受限制，需要系统级权限 `ohos.permission.START_ABILITIES_FROM_BACKGROUND`。
- 跨设备启动用 `startAbilityByCall`，需 `ohos.permission.DISTRIBUTED_DATASYNC`。
- 违反启动规则会报服务异常，排查时先看 `exported` 与权限。

## 4. UIAbility 启动模式与启动流程

### 4.1 launchType

在 `module.json5` 的 ability 上配置：

| 模式 | 行为 | 适用场景 |
|---|---|---|
| `singleton`（默认） | 复用唯一实例，再次启动走 `onNewWant` | 主界面、设置中心 |
| `multiton`（曾用 `standard`） | 每次启动创建新实例 | 分屏、多窗口 |
| `specified` | 由 `AbilityStage.onAcceptWant` 返回的 Key 决定复用或新建 | 文档类应用 |

```json5
{
  "abilities": [
    { "name": "EntryAbility", "launchType": "singleton" }
  ]
}
```

注意：`singleton` 实例启动中再次拉起会收到错误码 `16000082`；复用时只回调 `onNewWant`，不会重走 `onCreate` / `onWindowStageCreate`。

### 4.2 AbilityStage 与启动顺序

```text
进程创建 -> AbilityStage.onCreate（每个 HAP 一次，模块级初始化）
        -> UIAbility.onCreate -> onWindowStageCreate -> onForeground
```

- `AbilityStage.onCreate`：模块级资源预加载、注册环境变量监听（语言、深浅色）。
- `AbilityStage.onAcceptWant`：`specified` 模式的路由决策。
- 不要把与具体 UIAbility 强相关的逻辑放这里，也不要阻塞主线程。

### 4.3 AppStartup 启动框架

当启动任务多、依赖复杂时，把初始化从 `onCreate` 中抽到启动框架：

- 在资源目录定义启动任务配置，并在 `module.json5` 引用。
- 实现 `StartupTask` 接口，框架按依赖顺序执行。
- 支持自动模式（构造 AbilityStage 时执行）与手动模式（`startupManager.run`）。
- 支持 HSP/HAR 内配置启动任务（API 18+），不支持在 ExtensionAbility 启动时拉起。

### 4.4 启动入口

应用启动入口包括：桌面图标、快捷方式（`shortcuts`，静态或 `shortcutManager` 动态发布）、快捷栏、跨应用 Want/openLink、意图框架。不同入口传入的 Want 不同，可在 `onCreate` / `onNewWant` 中解析。

## 5. ArkWeb（Web 组件）

在应用内嵌入网页，使用 `@kit.ArkWeb` 的 `Web` 组件与 `webview.WebviewController`。

```ts
import { webview } from '@kit.ArkWeb';

@Entry
@ComponentV2
struct WebPage {
  controller: webview.WebviewController = new webview.WebviewController();

  build() {
    Column() {
      Web({ src: 'https://developer.huawei.com/consumer/cn/', controller: this.controller })
        .javaScriptAccess(true)
        .onPageEnd(() => {
          this.controller.runJavaScript('document.title').then((title: string) => {
            console.info(`title=${title}`);
          });
        })
    }
  }
}
```

- 前后端通信：`registerJavaScriptProxy` / `javaScriptProxy` 暴露原生方法给 H5，`runJavaScript` 执行页面脚本。
- 安全：配置 `onInterceptRequest`、域名白名单，谨慎开启混合内容。
- 权限：加载网络页面需 `ohos.permission.INTERNET`。

## 6. 资源与限定词目录、多语言与深色模式

### 6.1 资源目录与限定词

```text
resources/
├── base/                 # 默认资源
│   ├── element/          # string.json、color.json、float.json
│   ├── media/            # 图片
│   └── profile/          # main_pages.json、form_config.json
├── zh_CN/element/        # 中文
├── en_US/element/        # 英文
├── dark/element/         # 深色模式颜色
├── phone/ ... tablet/    # 设备限定词
```

系统按“语言 / 深浅色 / 设备类型 / 屏幕方向 / 分辨率”等限定词自动匹配最合适资源，无需手写判断。

### 6.2 多语言

```json5
// resources/base/element/string.json
{ "string": [ { "name": "app_name", "value": "记账" } ] }
// resources/en_US/element/string.json
{ "string": [ { "name": "app_name", "value": "Ledger" } ] }
```

页面统一用 `$r('app.string.app_name')` 引用，语言切换时自动生效。

### 6.3 深色模式

```ts
@ComponentV2
struct ThemedText {
  @Env('colorMode') colorMode: ColorMode = ColorMode.LIGHT;
  build() {
    Text('自适应文本').fontColor($r('app.color.text_primary'))
  }
}
```

颜色放到 `base/element/color.json` 与 `dark/element/color.json`，跟随系统自动切换；也可用 `EnvironmentCallback` 监听变化。

### 6.4 一多适配三种能力

| 能力 | 手段 |
|---|---|
| 自适应布局 | `Column`/`Row` + `layoutWeight`、`Flex`、`RelativeContainer` |
| 响应式布局 | 断点 + `GridRow`/`GridCol`，按窗口宽度切换结构 |
| 能力差异 | 按 `deviceTypes` 提供差异化功能，用限定词目录隔离资源 |

## 7. 通知、后台任务与窗口

### 7.1 通知（Notification Kit）

```ts
import { notificationManager } from '@kit.NotificationKit';

async function notify(title: string, text: string): Promise<void> {
  const request: notificationManager.NotificationRequest = {
    id: 1,
    content: { notificationContentType: notificationManager.ContentType.NOTIFICATION_CONTENT_BASIC_TEXT, normal: { title, text } }
  };
  await notificationManager.publish(request);
}
```

发布通知可能需要 `ohos.permission.NOTIFICATION_CONTROLLER`（系统应用）或用户授权，普通通知按 API 版本要求处理。

### 7.2 后台任务

| 类型 | 机制 | 场景 |
|---|---|---|
| 短时任务 | `backgroundTaskManager` 申请短暂延后挂起 | 保存数据、完成一次请求 |
| 长时任务 | `backgroundTaskManager.startBackgroundRunning` | 音乐、导航、录音 |
| 延迟任务 | `WorkSchedulerExtensionAbility` | 系统择机的后台作业 |

后台能力必须声明对应权限，长时任务需要合适的后台模式类型。

### 7.3 窗口管理

- 多窗口、分屏：配合 `launchType: multiton`。
- 悬浮窗、画中画（PiP）、窗口尺寸与安全区域适配，通过 `@kit.ArkUI` 的 `window` API 操作。
- 沉浸式：处理状态栏/导航栏安全区域，避免内容被遮挡。

## 8. 安全与隐私

- **加密**：`@kit.CryptoArchitectureKit` 提供对称/非对称加解密、摘要、签名、密钥派生，用于敏感数据本地保护与传输安全。
- **权限最小化**：只申请业务必要的权限，敏感权限运行时动态申请并说明用途。
- **应用沙箱**：应用只能访问自身沙箱目录，访问用户文件需通过 `filePicker` / 媒体库（`photoAccessHelper`）等受控接口。
- **隐私合规**：上架需提供隐私政策，权限使用需在隐私声明中如实描述，避免申请与功能无关的权限。

```ts
// 本地敏感数据建议加密后再落盘
// 使用 Crypto Architecture Kit 进行对称加解密，密钥存入安全存储
```

## 9. 数据管理扩展

| 能力 | 用途 |
|---|---|
| 用户文件 | `filePicker` 选择/保存用户文件、`photoAccessHelper` 访问媒体 |
| DataShare | 通过 `DataShareExtensionAbility` 向其他应用安全共享数据 |
| 分布式数据对象 | `distributedDataObject` 跨设备实时同步同一对象 |
| 分布式 KVStore / RDB | 同应用跨设备数据同步 |
| 端云一体化 | AppGallery Connect 云函数、云数据库，无需自建后端 |

```ts
import { distributedDataObject } from '@kit.ArkData';

// 跨设备实时共享一个对象（需同一账号登录设备）
const session = distributedDataObject.create(this.context, { total: 0 });
session.setSessionId('ledger-session');
session.on('change', (change) => { /* 同步到 UI */ });
```

## 阶段验收

- 能解释元服务与传统应用的区别，并说明卡片为何是其入口。
- 能区分静态 / 动态 / 互动卡片，能写出 `FormExtensionAbility` 生命周期与 `postCardAction`。
- 能列举 ExtensionAbility 常见类型并说明启动与安全规则。
- 能解释三种 `launchType` 的行为差异与 `onNewWant` / `onAcceptWant` 的作用。
- 能用 `AppStartup` 拆分启动任务，说明 `AbilityStage.onCreate` 的定位。
- 能用 ArkWeb 完成页面加载与原生-H5 通信。
- 能用限定词目录实现多语言、深色模式与多设备资源隔离。
- 能使用通知、后台任务与窗口能力，并遵守最小权限与隐私要求。
- 能说明加密、DataShare、分布式数据对象与端云一体化的适用场景。

## 动手任务

1. 新建元服务工程（Empty Form Ability），给记账应用添加一张 2x2 动态卡片并在桌面刷新数据。
2. 用 `postCardAction` 的 `router` 事件从卡片跳转到应用详情页。
3. 写一个 `AbilityStage.onAcceptWant`，用 `specified` 模式实现“同一文档复用实例”。
4. 用 AppStartup 把“分类字典预加载”拆成启动任务。
5. 用 Web 组件加载一个页面并注入一个原生方法来读取标题。
6. 为应用补充 `en_US` 与 `dark` 资源，验证切换语言与深色模式。
7. 发布一条通知并在点击后跳转到页面。（可选）用分布式数据对象在多设备上同步一条记录。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 卡片不刷新 | `updateDuration` 配置、`postCardAction` 事件类型、`formProvider.updateForm` 调用 |
| 卡片导入 API 报错 | 是否使用了“不支持在 ArkTS 卡片中使用”的模块；误用 HSP |
| 卡片点击无响应 | 静态卡片应用 `FormLink`，动态卡片用 `postCardAction` |
| 二次启动不刷新数据 | `singleton` 复用时只走 `onNewWant`，需在该回调更新 |
| `specified` 未复用实例 | `onAcceptWant` 返回的 Key 是否唯一且稳定 |
| 跨应用启动失败 | `exported` 是否为 `true`、权限是否满足 |
| Web 白屏 | `ohos.permission.INTERNET`、域名/混合内容策略 |
| 深色模式不生效 | 是否提供 `dark` 限定词资源；颜色是否用了 `$r` |
| 后台任务被回收 | 是否申请长时任务与权限、后台模式类型是否匹配 |

## 进入下一步

你可以回到 [综合实战：多设备记账元服务](./project-practice.md)，把本阶段的卡片、通知、启动框架、安全与数据扩展作为进阶增量接入；也可以到此为止，完成 [学习评估](./learning-assessment.md) 的评分与评审。
