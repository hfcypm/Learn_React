# 阶段三：网络、数据持久化与工程化

**目标**：封装类型安全的网络层，掌握 Preferences / RDB 两种持久化方案，用 HAR/HSP 拆分模块，并建立测试、日志与资源管理规范。

## 1. 网络请求

鸿蒙通过 `@kit.NetworkKit` 的 `http` 模块发起 HTTP 请求。使用前必须在 `module.json5` 声明 `ohos.permission.INTERNET`。

```ts
// module.json5
"requestPermissions": [
  { "name": "ohos.permission.INTERNET" }
]
```

```ts
import { http } from '@kit.NetworkKit';
import { BusinessError } from '@kit.BasicServicesKit';

export async function request<T>(url: string, method: http.RequestMethod = http.RequestMethod.GET, body?: Object): Promise<T> {
  const client = http.createHttp();
  try {
    const res = await client.request(url, {
      method,
      header: { 'Content-Type': 'application/json' },
      extraData: body ? JSON.stringify(body) : undefined,
      connectTimeout: 10000,
      readTimeout: 10000,
      expectDataType: http.HttpDataType.STRING
    });
    if (res.responseCode !== 200) {
      throw new Error(`HTTP ${res.responseCode}`);
    }
    return JSON.parse(res.result as string) as T;
  } finally {
    client.destroy(); // 必须销毁，否则连接泄漏
  }
}
```

### 1.1 封装统一网络层

生产项目应把以下职责集中到网络模块：

- 统一请求头注入（token、设备信息、版本号）。
- 统一错误码映射（把后端业务码转为用户可读提示）。
- 统一加载态与超时重试（弱网重试一次）。
- 统一返回值类型（`ApiResult<T>`）。

```ts
interface ApiResult<T> {
  code: number;
  message: string;
  data: T;
}

class ApiError extends Error {
  constructor(public code: number, message: string) {
    super(message);
  }
}

export async function api<T>(path: string, options?: { method?: http.RequestMethod; body?: Object }): Promise<T> {
  const res = await request<ApiResult<T>>(`https://api.example.com${path}`, options?.method, options?.body);
  if (res.code !== 0) {
    throw new ApiError(res.code, res.message);
  }
  return res.data;
}
```

## 2. 数据持久化选型

| 方案 | 模块 | 适用场景 |
|---|---|---|
| Preferences | `@kit.ArkData` | 轻量键值配置：开关、字体大小、登录标记 |
| 关系型数据库 RDB | `@kit.ArkData` | 结构化数据：离线缓存、会话列表、订单 |
| KVStore | `@kit.ArkData` | 跨设备键值同步（分布式） |

### 2.1 Preferences

```ts
import { preferences } from '@kit.ArkData';
import { common } from '@kit.AbilityKit';

const context = getContext(this) as common.UIAbilityContext;

async function saveTheme(theme: string): Promise<void> {
  const store = await preferences.getPreferences(context, 'settings');
  await store.put('theme', theme);
  await store.flush(); // 落盘
}

async function loadTheme(): Promise<string> {
  const store = await preferences.getPreferences(context, 'settings');
  return (await store.get('theme', 'light')) as string;
}
```

### 2.2 关系型数据库 RDB

RDB 基于 SQLite，支持 SQL、事务、索引，适合复杂关系数据。

```ts
import { relationalStore } from '@kit.ArkData';
import { common } from '@kit.AbilityKit';

const STORE_CONFIG: relationalStore.StoreConfig = {
  name: 'ledger.db',
  securityLevel: relationalStore.SecurityLevel.S1
};

const CREATE_SQL = `CREATE TABLE IF NOT EXISTS txn (
  id TEXT PRIMARY KEY,
  amount REAL NOT NULL,
  category TEXT NOT NULL,
  created_at INTEGER NOT NULL
)`;

export class LedgerStore {
  private store?: relationalStore.RdbStore;

  async init(context: common.Context): Promise<void> {
    this.store = await relationalStore.getRdbStore(context, STORE_CONFIG);
    await this.store.executeSql(CREATE_SQL);
  }

  async insert(item: { id: string; amount: number; category: string; createdAt: number }): Promise<void> {
    await this.store?.insert('txn', {
      id: item.id,
      amount: item.amount,
      category: item.category,
      created_at: item.createdAt
    }, relationalStore.ConflictResolution.ON_CONFLICT_REPLACE);
  }

  async listRecent(limit: number): Promise<{ id: string; amount: number }[]> {
    const predicates = new relationalStore.RdbPredicates('txn');
    predicates.orderByDesc('created_at');
    predicates.limitAs(limit);
    const rs = await this.store!.query(predicates, ['id', 'amount']);
    const out: { id: string; amount: number }[] = [];
    while (rs.goToNextRow()) {
      out.push({
        id: rs.getString(rs.getColumnIndex('id')),
        amount: rs.getDouble(rs.getColumnIndex('amount'))
      });
    }
    rs.close();
    return out;
  }
}
```

RDB 主要约束（官方文档）：

- 数据库同一时间只支持一个写操作，需合理串行化。
- 单次查询数据量建议不超过 5000 条，大数据量在 TaskPool 中执行。
- 一条数据建议不超过 2M，超出会插入成功但读取失败。
- 应用卸载后数据库文件会被自动清除。

### 2.3 缓存优先策略

列表类页面常用“先读本地、再拉网络、成功回写”的策略：

```text
onPageShow
  -> 读 RDB 立即渲染
  -> 请求网络
      -> 成功：写回 RDB + 刷新 UI
      -> 失败：保留本地数据 + 提示（弱网可用）
```

## 3. 权限与运行时申请

在 `module.json5` 声明只是第一步，敏感权限还需运行时动态申请：

```ts
import { abilityAccessCtrl, common, Permissions } from '@kit.AbilityKit';

async function ensurePermission(context: common.UIAbilityContext, permission: Permissions): Promise<boolean> {
  const atManager = abilityAccessCtrl.createAtManager();
  const result = await atManager.requestPermissionsFromUser(context, [permission]);
  return result.authResults[0] === 0;
}
```

最小授权原则：只声明并申请真正需要的能力。

## 4. 模块化工程：HAP / HAR / HSP

| 包类型 | 定位 | 加载时机 | 适用场景 |
|---|---|---|---|
| HAP | 应用主体，可独立运行 | 安装/运行 | 入口与特性模块 |
| HAR | 静态共享包 | 编译期嵌入依赖方 | 通用工具、模型、组件复用 |
| HSP | 动态共享包 | 运行期按需加载 | 体积大、使用频率低的功能 |

工程实践建议：

- 入口 `entry` 只做装配与页面，业务逻辑下沉到 HAR。
- 通用模型、网络层、工具函数抽取为 HAR，避免多模块复制。
- 体积大且低频的功能包用 HSP 动态加载，控制安装包体积。
- 通过 `oh-package.json5` 声明依赖，用 `ohpm` 管理。

```text
entry/                 # 入口 HAP，装配与页面
commons/               # HAR：模型、工具、网络层
feature-ledger/        # HAR：记账业务
feature-report/        # HSP：报表（体积大、低频）
```

## 5. 测试与质量

- **单元测试**：使用 Hypium 框架（`@ohos/hypium`），对纯逻辑（格式化、校验、计算）编写用例。
- **UI 测试**：`@ohos.UiTest` 驱动界面自动化。
- **DevEco Testing / Profiler**：性能、内存、稳定性检测；阶段四展开。
- **日志**：使用 `hilog`，按 domain/tag/level 输出，避免 `console.log` 遗留在生产代码。

```ts
// 单元测试示例（entry/src/ohosTest/ets/test/*.test.ets）
import { describe, it, expect } from '@ohos/hypium';
import { formatAmount } from '../../../main/ets/utils/format';

export default function abilityTest() {
  describe('formatAmountTest', () => {
    it('formats to two decimals', 0, () => {
      expect(formatAmount(12.5, 'CNY')).assertEqual('CNY 12.50');
    });
  });
}
```

## 阶段验收

- 能封装类型安全的网络层，处理超时、错误与加载态。
- 能按场景在 Preferences 与 RDB 之间选型，并写出可用的增删改查。
- 能实现“本地缓存优先、网络增量刷新”的取数策略。
- 能在 `module.json5` 声明权限，并对敏感权限做运行时申请。
- 能按职责把代码拆到 HAP / HAR / HSP。
- 能对纯逻辑编写 Hypium 单元测试。

## 动手任务

1. 封装一个 `api<T>()` 函数，统一处理错误码与超时。
2. 用 Preferences 保存并读取主题设置。
3. 用 RDB 建立 `txn` 表，实现插入与按时间倒序查询。
4. 实现列表页“先缓存后网络”的刷新逻辑。
5. 把网络层与数据模型抽到一个 HAR 模块并被 entry 引用。
6. 为金额格式化函数写一个通过 Hypium 执行的单元测试。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 请求报权限错误 | `module.json5` 是否声明 `INTERNET` |
| 内存/连接泄漏 | `http.createHttp()` 后是否 `destroy()` |
| RDB 写失败 | 是否并发多写；写操作需串行 |
| 查询慢/卡死 | 单次查询是否超过 5000 条；改用 TaskPool |
| 读取超 2M 失败 | 单条数据是否过大 |
| HAR 依赖冲突 | oh-package 版本是否统一，是否存在重复依赖 |
| 模块找不到 | 依赖是否 `ohpm install` 且在 `oh-package.json5` 声明 |

## 进入下一阶段的条件

你能独立完成“网络取数 -> 本地缓存 -> 类型安全渲染”的完整数据链路，并能把代码按模块职责拆分清楚。此时进入 [阶段四：并发、分布式、性能与发布](./stage-4-distributed-production.md)。
