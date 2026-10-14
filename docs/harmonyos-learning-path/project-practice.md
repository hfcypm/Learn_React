# 综合实战：多设备记账元服务

## 1. 项目目标

用 HarmonyOS NEXT + ArkTS + ArkUI 构建一个“多设备记账元服务”，覆盖阶段零到阶段四的全部能力：工程搭建、声明式 UI 与状态管理、Stage 模型与导航、网络与数据持久化、模块化工程、并发、多设备自适应与上架。项目按阶段扩展，最终交付一款可在手机与平板运行、可上架的鸿蒙应用。

```text
初始化工程 -> 界面与状态管理 -> 多页面导航
    -> 网络与本地缓存 -> 模块拆分 -> 并发与多设备 -> 打包发布
```

选择“记账”主题的原因：数据结构清晰（一笔交易 = 金额 + 分类 + 时间），天然需要列表、表单、导航、统计计算与持久化，能自然覆盖本路线全部知识点。

## 2. 需求

**必须实现（MVP）**

- 记录一笔交易：金额、分类、备注、时间。
- 交易列表：按时间倒序，支持空态。
- 分类汇总：按分类统计支出总额。
- 本地持久化：重启后数据不丢。
- 多页面导航：首页 -> 新增/编辑 -> 详情。

**进阶**

- 网络同步：从服务端拉取分类字典，弱网时用本地缓存。
- 多设备自适应：手机单栏、平板双栏。
- 并发统计：大数据量统计放到 TaskPool。
- 分布式：在手机上查看、在平板上编辑（设计目标）。

## 3. 技术选择

| 技术 | 用途 |
|---|---|
| ArkTS | 应用开发语言 |
| ArkUI（V2 状态管理） | 声明式 UI |
| Stage 模型 + UIAbility | 应用运行时入口 |
| Navigation + NavPathStack | 路由导航 |
| `@kit.NetworkKit` http | 网络请求 |
| `@kit.ArkData` Preferences / RDB | 数据持久化 |
| HAR / HSP | 模块化 |
| TaskPool / Worker | 并发 |
| AppGallery Connect | 签名与上架 |

## 4. 项目结构

```text
LedgerApp/
├── AppScope/app.json5
├── entry/                          # 入口 HAP
│   └── src/main/
│       ├── module.json5
│       ├── ets/
│       │   ├── entryability/EntryAbility.ets
│       │   ├── pages/Index.ets
│       │   └── pages/DetailPage.ets
│       └── resources/
├── commons/                        # HAR：模型、工具、网络层
│   └── src/main/ets/
│       ├── model/Transaction.ets
│       ├── api/client.ets
│       └── utils/format.ets
├── feature-ledger/                 # HAR：记账业务与数据访问
│   └── src/main/ets/db/LedgerStore.ets
├── build-profile.json5
└── oh-package.json5
```

## 5. 阶段零：初始化（工程与 ArkTS）

创建 `Empty Ability`（ArkTS）工程，配置应用信息，用 ArkTS 定义数据模型。

```ts
// commons/src/main/ets/model/Transaction.ets
@ObservedV2
export class Transaction {
  @Trace id: string = '';
  @Trace amount: number = 0;
  @Trace category: string = '';
  @Trace note: string = '';
  @Trace createdAt: number = 0;
}
```

```json5
// AppScope/app.json5
{
  "app": {
    "bundleName": "com.example.ledger",
    "versionCode": 1000000,
    "versionName": "1.0.0",
    "icon": "$media:app_icon",
    "label": "$string:app_name"
  }
}
```

验收：工程可运行，模型类型安全，`$r` 资源引用正常。

## 6. 阶段一：界面与状态管理

首页使用 V2 状态管理，展示总支出与交易列表，支持空态。

```ts
// entry/src/main/ets/pages/Index.ets
import { Transaction } from 'commons';
import { formatAmount } from 'commons';

@ComponentV2
struct SummaryCard {
  @Param total: number = 0;
  @Param count: number = 0;

  build() {
    Column({ space: 6 }) {
      Text('本月支出').fontSize(14).fontColor($r('app.color.text_secondary'))
      Text(formatAmount(this.total)).fontSize(28).fontWeight(FontWeight.Bold)
      Text(`共 ${this.count} 笔`).fontSize(12)
    }
    .alignItems(HorizontalAlign.Start)
    .padding(16)
    .width('100%')
    .borderRadius(16)
    .backgroundColor($r('app.color.card_bg'))
  }
}

@ComponentV2
struct TransactionItem {
  @Param item: Transaction = new Transaction();

  build() {
    Row({ space: 12 }) {
      Column({ space: 4 }) {
        Text(this.item.category).fontSize(16)
        Text(this.item.note || '无备注').fontSize(12).fontColor($r('app.color.text_secondary'))
      }.layoutWeight(1).alignItems(HorizontalAlign.Start)

      Text(`-${this.item.amount.toFixed(2)}`).fontSize(16).fontColor(Color.Red)
    }
    .width('100%')
    .padding(12)
    .borderRadius(12)
    .backgroundColor($r('app.color.card_bg'))
  }
}

@Entry
@ComponentV2
struct Index {
  @Provider('navStack') navStack: NavPathStack = new NavPathStack();
  @Local list: Transaction[] = [];

  @Computed
  get total(): number {
    return this.list.reduce((s, t) => s + t.amount, 0);
  }

  build() {
    Navigation(this.navStack) {
      Column({ space: 12 }) {
        SummaryCard({ total: this.total, count: this.list.length })

        if (this.list.length === 0) {
          Text('还没有记录，点击右下角新增').fontColor($r('app.color.text_secondary'))
        } else {
          List({ space: 8 }) {
            ForEach(this.list, (item: Transaction) => {
              ListItem() {
                TransactionItem({ item })
              }
            }, (item: Transaction) => item.id)
          }
          .layoutWeight(1)
          .width('100%')
        }
      }
      .padding(16)
      .width('100%')
      .height('100%')
    }
    .title('记账')
    .navDestination(PageMap)
  }
}

@Builder
export function PageMap(name: string) {
  if (name === 'DetailPage') {
    DetailPage()
  } else if (name === 'EditPage') {
    EditPage()
  }
}

import { DetailPage } from './DetailPage';
import { EditPage } from './EditPage';
```

验收：状态驱动刷新正常，`@Computed` 计算总额，空态与列表切换正确。

## 7. 阶段二：多页面导航

用 `Navigation` + `NavPathStack` 实现首页 -> 详情 / 编辑的导航与返回结果。

```ts
// 首页跳转
this.navStack.pushPathByName('EditPage', null, (popInfo: PopInfo) => {
  const saved = popInfo.result as boolean;
  if (saved) {
    this.reload(); // 编辑页保存后刷新
  }
});
```

```ts
// entry/src/main/ets/pages/EditPage.ets
import { Transaction } from 'commons';

@ComponentV2
export struct EditPage {
  @Consume('navStack') navStack: NavPathStack;
  @Local amountText: string = '';
  @Local category: string = '餐饮';

  private readonly categories: string[] = ['餐饮', '交通', '购物', '其他'];

  build() {
    NavDestination() {
      Column({ space: 16 }) {
        TextInput({ placeholder: '金额' })
          .type(InputType.Number)
          .onChange((v: string) => { this.amountText = v; })

        Select(this.categories.map((c: string) => ({ value: c })))
          .onSelect((index: number) => { this.category = this.categories[index]; })

        Button('保存')
          .width('100%')
          .onClick(async () => {
            const amount = Number(this.amountText);
            if (!Number.isFinite(amount) || amount <= 0) {
              this.getUIContext().getPromptAction().showToast({ message: '请输入有效金额' });
              return;
            }
            // 保存逻辑见阶段三
            this.navStack.pop({ result: true });
          })

        Button('取消').onClick(() => this.navStack.pop({ result: false }))
      }
      .padding(16)
    }
    .title('新增交易')
  }
}
```

验收：三级导航可用，传参与返回结果正确，编辑页保存后首页刷新。

## 8. 阶段三：网络、持久化与模块拆分

### 8.1 网络层（HAR：commons）

```ts
// commons/src/main/ets/api/client.ets
import { http } from '@kit.NetworkKit';

export interface ApiResult<T> {
  code: number;
  message: string;
  data: T;
}

export async function api<T>(path: string, method: http.RequestMethod = http.RequestMethod.GET, body?: Object): Promise<T> {
  const client = http.createHttp();
  try {
    const res = await client.request(`https://api.example.com${path}`, {
      method,
      header: { 'Content-Type': 'application/json' },
      extraData: body ? JSON.stringify(body) : undefined,
      connectTimeout: 10000,
      readTimeout: 10000
    });
    const parsed = JSON.parse(res.result as string) as ApiResult<T>;
    if (parsed.code !== 0) {
      throw new Error(parsed.message);
    }
    return parsed.data;
  } finally {
    client.destroy();
  }
}
```

### 8.2 数据层（HAR：feature-ledger）

```ts
// feature-ledger/src/main/ets/db/LedgerStore.ets
import { relationalStore } from '@kit.ArkData';
import { Transaction } from 'commons';
import { common } from '@kit.AbilityKit';

const CONFIG: relationalStore.StoreConfig = {
  name: 'ledger.db',
  securityLevel: relationalStore.SecurityLevel.S1
};

export class LedgerStore {
  private store?: relationalStore.RdbStore;

  async init(context: common.Context): Promise<void> {
    this.store = await relationalStore.getRdbStore(context, CONFIG);
    await this.store.executeSql(
      `CREATE TABLE IF NOT EXISTS txn (
        id TEXT PRIMARY KEY, amount REAL NOT NULL,
        category TEXT NOT NULL, note TEXT, created_at INTEGER NOT NULL)`
    );
  }

  async save(t: Transaction): Promise<void> {
    await this.store?.insert('txn', {
      id: t.id, amount: t.amount, category: t.category,
      note: t.note, created_at: t.createdAt
    }, relationalStore.ConflictResolution.ON_CONFLICT_REPLACE);
  }

  async all(): Promise<Transaction[]> {
    const list: Transaction[] = [];
    const rs = await this.store!.query(new relationalStore.RdbPredicates('txn'), []);
    while (rs.goToNextRow()) {
      const t = new Transaction();
      t.id = rs.getString(rs.getColumnIndex('id'));
      t.amount = rs.getDouble(rs.getColumnIndex('amount'));
      t.category = rs.getString(rs.getColumnIndex('category'));
      t.note = rs.getString(rs.getColumnIndex('note'));
      t.createdAt = rs.getLong(rs.getColumnIndex('created_at'));
      list.push(t);
    }
    rs.close();
    return list;
  }
}
```

### 8.3 模块依赖

```json5
// entry/oh-package.json5
{
  "dependencies": {
    "commons": "file:../commons",
    "feature-ledger": "file:../feature-ledger"
  }
}
```

### 8.4 缓存优先刷新

```ts
async reload(): Promise<void> {
  this.store = this.store ?? new LedgerStore();
  this.list = await this.store.all();     // 1. 本地先渲染
  try {
    const dict = await api<string[]>('/categories'); // 2. 网络刷新
    this.categories = dict;
  } catch (e) {
    // 3. 失败保留本地
  }
}
```

验收：网络层类型安全，RDB 增删改查可用，缓存优先策略生效，模块依赖正确。

## 9. 阶段四：并发、多设备与发布

### 9.1 并发统计

```ts
import { taskpool } from '@kit.ArkTS';

@Sendable
class StatItem {
  amount: number = 0;
  category: string = '';
}

@Concurrent
function groupSum(items: StatItem[]): Record<string, number> {
  const acc: Record<string, number> = {};
  for (const it of items) {
    acc[it.category] = (acc[it.category] ?? 0) + it.amount;
  }
  return acc;
}

async function computeStats(items: StatItem[]): Promise<Record<string, number>> {
  return await taskpool.execute(new taskpool.Task(groupSum, items)) as Record<string, number>;
}
```

### 9.2 多设备自适应

```ts
GridRow({
  columns: { sm: 4, md: 8, lg: 12 },
  breakpoints: { value: ['600vp', '840vp'], reference: BreakpointsReference.WindowSize }
}) {
  GridCol({ span: { sm: 4, md: 4, lg: 6 } }) { SummaryCard({ total: this.total, count: this.list.length }) }
  GridCol({ span: { sm: 4, md: 4, lg: 6 } }) { CategorySummary() }
}
```

### 9.3 权限声明

```json5
// entry/src/main/module.json5
"requestPermissions": [
  { "name": "ohos.permission.INTERNET" }
]
```

### 9.4 打包发布

```bash
hvigorw assembleHap --mode module -p product=default
hvigorw assembleApp --mode project -p product=default
```

配置 release 签名后，用 AppGallery Connect 上传 `.app` 包并提交审核。

验收：并发统计不阻塞主线程，手机/平板布局正确，签名完整，可上传 AppGallery Connect。

## 10. 实施顺序

1. 初始化工程，定义 `Transaction` 模型。
2. 用 V2 状态管理实现首页、汇总卡片与列表。
3. 用 Navigation 打通首页 -> 详情 / 编辑。
4. 接入网络层与 RDB，实现缓存优先刷新。
5. 拆出 `commons` 与 `feature-ledger` 两个 HAR。
6. 用 TaskPool 做分类统计，用 GridRow 适配多设备。
7. 配置签名，构建并走通上架流程。

## 11. 验收清单

- [ ] 能新增交易并持久化，重启后数据仍在。
- [ ] 列表按时间倒序，空态与有数据切换正确。
- [ ] `@Computed` 总额与分类汇总正确。
- [ ] 三级导航与返回结果正确。
- [ ] 网络失败时回退到本地缓存。
- [ ] 代码拆分到 HAP / HAR 且依赖声明正确。
- [ ] 统计计算在 TaskPool 中执行，主线程不卡顿。
- [ ] 手机单栏、平板双栏布局正确。
- [ ] `module.json5` 仅声明必要权限。
- [ ] release 签名完整，可构建 `.app` 并上传。

## 12. 按阶段学习卡片

| 阶段 | 项目增量 | 关键能力 |
|---|---|---|
| 零 | 初始化、模型、资源 | ArkTS、工程结构 |
| 一 | 首页、卡片、列表 | ArkUI、V2 状态管理 |
| 二 | 详情、编辑、返回 | Stage 模型、Navigation |
| 三 | 网络层、RDB、HAR | 网络与持久化、模块化 |
| 四 | 并发、自适应、发布 | 并发、多设备、上架 |
