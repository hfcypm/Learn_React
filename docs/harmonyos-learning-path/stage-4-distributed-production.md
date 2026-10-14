# 阶段四：并发、分布式、性能与发布

**目标**：掌握鸿蒙的并发模型与分布式能力，实现多设备自适应，能用工具定位性能问题，并完成签名打包与上架。

## 1. 并发模型

主线程只处理 UI，耗时计算、批量数据库操作、大文件处理必须放到并发任务中。鸿蒙提供两种并发机制：

| 机制 | 定位 | 生命周期 | 适用场景 |
|---|---|---|---|
| TaskPool | 任务池，系统调度 | 系统自动管理 | 短时、可并行、无需常驻的任务 |
| Worker | 独立线程 | 需手动创建与销毁 | 常驻、需要长时间运行或持有状态的任务 |

### 1.1 TaskPool

```ts
import { taskpool } from '@kit.ArkTS';

@Concurrent
function computeJank(buffer: number[]): number {
  // 在并发线程执行，需用 @Concurrent 标注
  let sum = 0;
  for (let i = 0; i < buffer.length; i++) {
    sum += buffer[i];
  }
  return sum;
}

async function run(): Promise<void> {
  const task = new taskpool.Task(computeJank, [1, 2, 3, 4, 5]);
  const result = await taskpool.execute(task) as number;
  console.info(`result=${result}`);
}
```

### 1.2 Worker

```ts
// entry/src/main/ets/workers/HeavyWorker.ets
import { worker, ThreadWorkerGlobalScope, MessageEvents } from '@kit.ArkTS';

const workerPort: ThreadWorkerGlobalScope = worker.workerPort;
workerPort.onmessage = (e: MessageEvents) => {
  const input = e.data as number;
  workerPort.postMessage(input * input);
};

// 主线程
const w = new worker.ThreadWorker('entry/ets/workers/HeavyWorker.ets');
w.onmessage = (e: MessageEvents) => { console.info(`${e.data}`); };
w.postMessage(10);
// 用完销毁
w.terminate();
```

### 1.3 跨线程数据：@Sendable

并发任务之间传递的对象必须是可安全共享的。用 `@Sendable` 标注类，保证跨线程传递时数据安全、避免共享引用：

```ts
@Sendable
class LedgerRecord {
  id: string = '';
  amount: number = 0;
}

@Concurrent
function sumRecords(records: LedgerRecord[]): number {
  return records.reduce((s, r) => s + r.amount, 0);
}
```

原则：主线程 UI 与非并发对象不跨线程传递；跨线程只传 `@Sendable` 实例或基础类型；大数据优先用 `SharedArrayBuffer` 或分批传入。

## 2. 分布式与多设备协同

鸿蒙的分布式能力是其区别于其他平台的核心。关键概念：

- **分布式软总线**：设备自动发现、连接、传输的底层能力。
- **分布式数据管理**：跨设备共享 Preferences / RDB / KVStore（同应用多设备数据同步）。
- **跨端迁移 / 流转**：任务在设备间接续，例如手机上编辑、平板上继续。
- **一次开发，多端部署**：一套代码适配手机、折叠屏、平板、车机。

### 2.1 多设备自适应布局

用断点（Breakpoints）与响应式栅格适配不同屏宽：

```ts
GridRow({
  columns: { sm: 4, md: 8, lg: 12 },
  breakpoints: { value: ['600vp', '840vp'], reference: BreakpointsReference.WindowSize }
}) {
  GridCol({ span: { sm: 4, md: 4, lg: 6 } }) {
    Text('收入')
  }
  GridCol({ span: { sm: 4, md: 4, lg: 6 } }) {
    Text('支出')
  }
}
```

配合窗口监听，在断点变化时切换单栏 / 双栏布局：

```ts
import { window } from '@kit.ArkUI';

function watchBreakpoint(win: window.Window): void {
  win.on('windowSizeChange', (size: window.Size) => {
    const widthVp = px2vp(size.width);
    const breakpoint = widthVp >= 840 ? 'lg' : widthVp >= 600 ? 'md' : 'sm';
    console.info(`breakpoint=${breakpoint}`);
  });
}
```

### 2.2 设备类型声明

`module.json5` 的 `deviceTypes` 决定应用可安装的设备，UI 需对每种设备给出可用布局，避免在大屏拉伸变形。

## 3. 性能调优

### 3.1 长列表与渲染性能

```text
长列表 -> List + LazyForEach（按需构建）
    -> 设置 cachedCount（缓存数量）
    -> @ReusableV2 复用组件
    -> 稳定 key，避免重复创建
```

```ts
LazyForEach(this.dataSource, (item: Transaction) => {
  TransactionItem({ item })
}, (item: Transaction) => item.id)
```

### 3.2 减少冗余刷新

- 只有与 UI 相关的变量才加状态装饰器。
- 频繁写入先在临时变量中累积，最后一次性写回状态。
- 用 `@ObservedV2` + `@Trace` 精准刷新嵌套属性，避免整树重刷。
- 正确使用 `@Computed`，避免在 `build` 中重复计算。

### 3.3 启动性能

- 重初始化移出 `onWindowStageCreate`，放 `onCreate` 并异步化。
- 延迟加载非首屏模块（HSP 动态加载）。
- 避免首屏做大量同步 I/O 或网络请求。

### 3.4 工具

| 工具 | 用途 |
|---|---|
| DevEco Profiler | CPU、内存、时延、能耗、网络分析，定位卡顿与泄漏 |
| Code Scanner | 检查整个项目的资源泄漏、缺陷与编码规范 |
| 应用体检 / DevEco Testing | 稳定性、性能、兼容性批量检测 |
| Inspector | 查看组件树与属性，类似 Web 的 Elements |
| Previewer | 实时预览并支持双向预览、多断点预览 |

排查卡顿主线：

```text
用户反馈卡顿
  -> Profiler 录制一段可复现操作
  -> 看帧率 / 主线程阻塞时长
  -> 定位到具体组件或任务（列表、动画、I/O、并发调度）
  -> 对应到代码（LazyForEach 键、@Trace 粒度、并发归属）
  -> 修复后复测对比
```

## 4. 打包、签名与发布

### 4.1 构建

```bash
hvigorw assembleHap --mode module -p product=default
hvigorw assembleApp --mode project -p product=default   # 生成 .app 包
```

构建产物在 `entry/build/default/outputs/` 或工程级 `build/outputs/`。

### 4.2 签名

- 调试阶段可用 DevEco Studio 自动签名。
- 发布需在 AppGallery Connect 创建应用，生成证书与 Profile，在 `build-profile.json5` 配置签名信息。

```json5
// build-profile.json5（示意）
{
  "app": {
    "signingConfigs": [
      {
        "name": "release",
        "type": "HarmonyOS",
        "material": {
          "certpath": "release.cer",
          "storePassword": "******",
          "keyAlias": "release",
          "keyPassword": "******",
          "profile": "release.p7b",
          "signAlg": "SHA256withECDSA",
          "storeFile": "release.p12"
        }
      }
    ],
    "products": [
      {
        "name": "default",
        "signingConfig": "release",
        "compatibleSdkVersion": "5.0.0(12)",
        "runtimeOS": "HarmonyOS"
      }
    ]
  }
}
```

### 4.3 发布流程

```text
完成功能与自测
  -> 应用体检 / 兼容性测试通过
  -> 生成 release 证书与 Profile
  -> 工程配置签名
  -> 构建 .app 包
  -> AppGallery Connect 上传
  -> 填写隐私、截图、描述等资料
  -> 提交审核 -> 发布
```

上架前的关键检查：隐私政策与权限说明、权限最小化、崩溃率、多设备兼容、包体积与冷启动。

## 阶段验收

- 能区分 TaskPool 与 Worker，并用 `@Concurrent` / `@Sendable` 处理并发任务。
- 能解释分布式软总线、分布式数据管理与跨端流转。
- 能用断点与 `GridRow` / `GridCol` 适配手机与平板。
- 能用 LazyForEach、缓存、`@ReusableV2`、精准刷新优化长列表。
- 能用 DevEco Profiler 与 Code Scanner 定位卡顿、泄漏与规范问题。
- 能完成签名配置、构建 `.app` 包与上架流程。

## 动手任务

1. 用 TaskPool 把一份大列表的统计计算移出主线程。
2. 用 Worker 建立一个常驻计算线程并演示消息往返。
3. 用 `@Sendable` 定义可跨线程传递的数据类。
4. 用 `GridRow` / `GridCol` 让列表在手机单栏、平板双栏切换。
5. 用 LazyForEach + `@ReusableV2` 渲染 1000 条数据，用 Profiler 对比优化前后。
6. 生成 release 证书并在 AppGallery Connect 查看上传流程。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 主线程卡顿 | 耗时计算是否在 TaskPool / Worker；列表是否用 LazyForEach |
| 并发报错 | 跨线程对象是否 `@Sendable`；是否非法共享 UI 对象 |
| Worker 不退出 | 是否调用 `terminate()` |
| 大屏拉伸变形 | `deviceTypes` 与响应式布局是否适配 |
| 列表滚动掉帧 | 键是否稳定；`cachedCount` 是否合理；是否过度刷新 |
| 上架被拒 | 权限与隐私说明是否匹配；是否申请了非必要权限 |
| 签名失败 | 证书、Profile、密码、`compatibleSdkVersion` 是否匹配 |

## 完成标准

你能在真机上交付一款多设备可运行、性能达标、签名完整并可上架的鸿蒙应用。至此进入 [综合实战：多设备记账元服务](./project-practice.md) 做整体验收。
