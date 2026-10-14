# 阶段零：HarmonyOS 与 ArkTS 基础

**目标**：建立对纯血鸿蒙、工程结构和工具链的正确认知，掌握 ArkTS 的类型系统与严格模式，跑通并读懂第一个应用。

## 1. HarmonyOS NEXT 是什么

HarmonyOS NEXT（5.x 起，即“纯血鸿蒙”）不再包含 AOSP 兼容层，不能运行 Android APK。应用侧一切从零构建：

| 维度 | 纯血鸿蒙 | 传统 Android |
|---|---|---|
| 应用语言 | ArkTS | Kotlin / Java |
| UI 框架 | ArkUI（声明式） | 传统 View / Compose |
| 应用模型 | Stage 模型 | Activity / Fragment |
| 构建工具 | Hvigor + ohpm | Gradle + Maven |
| 包格式 | HAP / HSP / HAR | APK / AAR |
| IDE | DevEco Studio | Android Studio |

系统强调“一次开发，多端部署”和分布式协同：手机、平板、折叠屏、车机、智慧屏、穿戴设备共享一套代码与能力。

## 2. 工程结构

用 DevEco Studio 新建 `Empty Ability`（ArkTS）工程后，关键文件如下：

```text
MyApplication/
├── AppScope/
│   ├── app.json5                 # 应用级：包名、版本、图标、标签
│   └── resources/                # 应用级资源
├── entry/                        # 入口 HAP 模块
│   ├── src/main/
│   │   ├── module.json5          # 模块级：Ability、权限、页面入口
│   │   ├── ets/
│   │   │   ├── entryability/     # UIAbility 入口
│   │   │   ├── entrybackupability/
│   │   │   └── pages/            # @Entry 页面（Index.ets）
│   │   └── resources/
│   │       ├── base/element/     # color、string、float 等
│   │       ├── base/media/       # 图片、图标
│   │       └── base/profile/     # main_pages.json 页面清单
│   └── oh-package.json5          # 模块依赖
├── build-profile.json5           # 构建目标（products）与签名
├── oh-package.json5              # 根依赖
└── hvigorfile.ts                 # 构建脚本入口
```

### 2.1 app.json5

```json5
{
  "app": {
    "bundleName": "com.example.myapp",
    "vendor": "example",
    "versionCode": 1000000,
    "versionName": "1.0.0",
    "icon": "$media:app_icon",
    "label": "$string:app_name"
  }
}
```

### 2.2 module.json5

```json5
{
  "module": {
    "name": "entry",
    "type": "entry",
    "description": "$string:module_desc",
    "mainElement": "EntryAbility",
    "deviceTypes": ["phone", "tablet", "2in1"],
    "deliveryWithInstall": true,
    "installationFree": false,
    "pages": "$profile:main_pages",
    "abilities": [
      {
        "name": "EntryAbility",
        "srcEntry": "./ets/entryability/EntryAbility.ets",
        "description": "$string:EntryAbility_desc",
        "icon": "$media:layered_image",
        "label": "$string:EntryAbility_label",
        "startWindowIcon": "$media:startIcon",
        "startWindowBackground": "$color:start_window_background",
        "exported": true,
        "skills": [
          {
            "entities": ["entity.system.home"],
            "actions": ["action.system.home"]
          }
        ]
      }
    ],
    "requestPermissions": [
      { "name": "ohos.permission.INTERNET" }
    ]
  }
}
```

- `type`：`entry` 为应用入口，`feature` 为特性模块，`har`/`shared` 为共享包。
- `deviceTypes`：声明可运行的设备类型。
- `requestPermissions`：声明所需权限，敏感权限还需运行时动态申请。

### 2.3 资源引用

资源统一用 `$r` 引用，避免硬编码：

```ts
Text($r('app.string.app_name'))
  .fontColor($r('app.color.primary'))
Image($r('app.media.logo'))
```

## 3. ArkTS 语言基础

ArkTS 是 TypeScript 的超集，但为了性能与可静态分析，收紧了不少动态特性。严格模式下的核心约束：

- 禁止使用 `any` 和 `unknown`，一切显式声明类型。
- 禁止运行时动态增删对象属性、改变对象结构（对象布局要稳定）。
- 只在明确的类型入口使用 `unknown` 转换。
- 变量声明使用 `let` / `const`，基本类型与 TypeScript 一致。

### 3.1 基础类型与函数

```ts
let count: number = 0;
const title: string = '记账';
let enabled: boolean = true;

class Transaction {
  id: string = '';
  amount: number = 0;
  category: string = '';
}

interface Category {
  key: string;
  label: string;
}

function formatAmount(value: number, currency: string = 'CNY'): string {
  return `${currency} ${value.toFixed(2)}`;
}

const arrowFormat = (v: number): string => `-${v}`;
```

### 3.2 接口、泛型与联合类型

```ts
interface ApiResult<T> {
  code: number;
  message: string;
  data: T;
}

type LoadState = 'idle' | 'loading' | 'success' | 'error';

function unwrap<T>(res: ApiResult<T>): T {
  if (res.code !== 0) {
    throw new Error(res.message);
  }
  return res.data;
}
```

### 3.3 类与继承

```ts
abstract class LedgerBase {
  abstract summary(): string;
}

class IncomeLedger extends LedgerBase {
  constructor(private readonly total: number) {
    super();
  }
  summary(): string {
    return `收入合计 ${this.total}`;
  }
}
```

### 3.4 异步

```ts
async function delay(ms: number): Promise<void> {
  return new Promise<void>((resolve) => setTimeout(() => resolve(), ms));
}

async function load(): Promise<Transaction[]> {
  await delay(200);
  return [];
}
```

## 4. 第一个页面

`entry/src/main/ets/pages/Index.ets`：

```ts
@Entry
@Component
struct Index {
  @State message: string = 'Hello HarmonyOS';

  build() {
    Column({ space: 12 }) {
      Text(this.message)
        .fontSize(24)
        .fontWeight(FontWeight.Bold)
      Button('点击更新')
        .onClick(() => {
          this.message = '你好，鸿蒙';
        })
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
  }
}
```

- `@Entry`：标记页面入口，一个页面有且仅有一个。
- `@Component`：声明自定义组件。
- `@State`：组件内部状态，变化驱动 UI 刷新。
- `build()`：声明式地描述 UI 结构，必须返回组件树。

页面需要在 `entry/src/main/resources/base/profile/main_pages.json` 注册：

```json
{
  "src": ["pages/Index"]
}
```

## 5. 工具链

### 5.1 ohpm 包管理

```bash
ohpm install                # 安装 oh-package.json5 中的依赖
ohpm install dayjs          # 安装三方库（OpenHarmony 三方库中心仓）
ohpm list                   # 查看依赖
```

### 5.2 Hvigor 构建

Hvigor 是鸿蒙的构建工具，任务与配置在 `hvigorfile.ts` 与 `build-profile.json5` 中：

```bash
hvigorw assembleHap --mode module -p product=default   # 构建 HAP
hvigorw clean                                           # 清理
```

多数情况下由 DevEco Studio 的 Build / Run 按钮触发，理解任务名有助于排查构建问题。

## 阶段验收

- 能解释纯血鸿蒙与传统 Android 在语言、UI、模型、构建上的差异。
- 能读懂 app.json5、module.json5、build-profile.json5、oh-package.json5。
- 能用 ArkTS 写出类型安全、符合严格模式的代码。
- 能创建工程并跑通首个页面，理解 `@Entry` / `@Component` / `@State` / `build()`。
- 能用 `$r` 引用资源，能执行 ohpm 安装与 hvigorw 构建。

## 动手任务

1. 新建 `Empty Ability` 工程并运行到模拟器或真机。
2. 修改 `app.json5` 的 `bundleName`、`versionName`，观察签名与安装变化。
3. 在 `resources/base/element` 新增一个颜色和一个字符串，用 `$r` 引用到页面。
4. 用 ArkTS 写一个 `Transaction` 类与 `formatAmount` 函数，并在页面显示。
5. 在 `module.json5` 声明 `ohos.permission.INTERNET`，理解权限声明位置。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| SDK 加载异常 / 构建失败 | 安装与项目路径是否含中文、空格 |
| 页面找不到 | `main_pages.json` 是否注册、`@Entry` 是否存在 |
| 编译报 `any` 错误 | 严格模式禁止 `any`，显式声明类型 |
| 资源引用失败 | `$r('app.string.xxx')` 名称与资源文件是否一致 |
| 依赖安装失败 | 检查网络与 ohpm 仓库配置，重试 `ohpm install` |
| 真机安装失败 | 自动签名是否配置、设备调试是否开启 |

## 进入下一阶段的条件

你能独立创建并运行工程，能解释每个配置文件的作用，并能用 ArkTS 写出符合严格模式的基础代码。此时进入 [阶段一：ArkUI 声明式 UI 与状态管理](./stage-1-arkui.md)。
