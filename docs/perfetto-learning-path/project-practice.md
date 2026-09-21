# Perfetto 综合实战：个股详情页卡顿排查

## 1. 项目目标

用同一条业务路径练完录制、读帧、SQL、埋点与回归对比。默认场景是证券 Android 个股详情：多层自定义 View + Fragment + ViewPager，主线程动画与行情刷新并存。

```text
复现操作 -> 录 15 秒 System Trace -> FrameTimeline 标红帧
    -> 拆 doFrame -> SQL 定量 -> 埋点复录 -> 对比
```

包名示例使用 `com.tzt.android`，换成你的应用包名即可。

## 2. 需求

- 能稳定复现：进入个股详情，停留分时 / K 线，下滑打开 F10。
- 产出一份 `.perfetto-trace` 与一份书面帧记录。
- 至少定位一种根因类型：重绘制、动画回调过重、主线程等待、邻页仍在跑。
- 加最少埋点后复录，证明切片名字出现且 dur 可对比。

## 3. 技术选择

| 技术 | 用途 |
|---|---|
| Android Studio System Trace | 日常录制 |
| ui.perfetto.dev | 拆帧与 SQL |
| `record_android_trace` | 固定探针、复录对比 |
| `android.os.Trace` | 业务切片 |
| Studio Sample Java Methods | 对不上方法名时短录 |
| LeakCanary | 若同时怀疑泄漏，分开单走 |

## 4. 场景拆解

| 步骤 | 用户动作 | 系统上可能发生的事 |
|---|---|---|
| A | 从列表点进个股 | Activity/Fragment 创建、View 树 inflate、首次 measure/layout |
| B | 分时图滚动或十字光标 | 自定义 View `onDraw`、手势、可能每帧 invalidate |
| C | 行情推送到达 | 主线程 `bind(Quote)`、多层 View 刷新 |
| D | ViewPager 左右滑 | 邻页 attach，离屏页动画是否仍跑 |
| E | 滑到底加载 F10 | 此时才 `new WebView`，观察是否提前创建 |

每次 trace 只覆盖其中一条，避免 A+E 混在 15 秒里无法归因。

## 5. 阶段增量

### 阶段零

写清本应用的进程名（主进程 / 推送进程）、详情页所在 Fragment、关键自定义 View 类名。

### 阶段一

热身到股票列表 -> 开始录制 -> 点进一只股票 -> 停 3 秒看分时 -> 结束。导出 `quote-detail-enter.perfetto-trace`。

确认 UI 中有 `com.tzt.android`、主线程、RenderThread、FrameTimeline。

### 阶段二

在进入详情后的 2 秒窗口找红 Actual。填写至少 3 条帧记录。重点看：

- 第一条红帧是不是 inflate / 首次 layout。
- 之后是否周期性红帧（像动画或推送节奏）。
- `draw` 是否 Sleeping。

已知对照模式：`draw` ≈40.7ms 且 Sleeping ≈40.22ms，表示在等；`animation` 233ms 只包 1 个 `doFrame`，表示动画回调把帧撑爆。

### 阶段三

SQL 导出进入详情后所有 `doFrame > 16ms`。对 Top 1 列子切片。

在嫌疑回调加埋点，例如：

```java
Trace.beginSection("QuotePager:bindQuote");
try {
    minuteView.bind(quote);
    klineView.bind(quote);
} finally {
    Trace.endSection();
}
```

复录 `quote-detail-enter-v2.perfetto-trace`，搜索 `QuotePager:bindQuote`。

若埋点已出现但内部仍是黑盒，再短录一次 Sample Java Methods，只抓同一操作。

### 阶段四

把结论写成缺陷单：

- 现象：进入详情后前 2 秒掉帧。
- 证据：trace 文件 + 红帧时间 + SQL Top N。
- 根因：例如 `onAnimationUpdate` 内 `requestLayout`。
- 修复：回调里只改属性，布局移出动画。
- 回归：同一操作红帧从 N 降到 M，p99 `doFrame` 从 Xms 到 Yms。

泄漏疑点另开 Memory 单，不写进这份卡顿结论。

## 6. 命令备忘

```bash
python3 record_android_trace \
    -o quote-detail-enter.perfetto-trace \
    -t 15s \
    -b 32mb \
    -a com.tzt.android \
    sched freq gfx view ss input binder_driver
```

```sql
SELECT s.dur / 1e6 AS dur_ms, s.ts
FROM slice s
JOIN thread_track tt ON s.track_id = tt.id
JOIN thread USING (utid)
JOIN process USING (upid)
WHERE s.name = 'Choreographer#doFrame'
  AND process.name = 'com.tzt.android'
  AND s.dur > 16 * 1000 * 1000
ORDER BY s.dur DESC
LIMIT 20;
```

## 7. 验收演示

向同事投屏时按这个顺序，3–5 分钟讲完：

1. 复现步骤与文件名。
2. FrameTimeline 颜色。
3. 一条红帧展开到 `doFrame` 子切片。
4. SQL Top N。
5. 埋点前后对比。
6. 修复建议与回归指标。

## 8. 扩展

- 把 Macrobenchmark 接到同一操作，CI 出 jank 计数。
- 对推送进程单独录一份，确认主进程卡顿不是 `:push` 抢 CPU。
- 120Hz 设备用 8ms 阈值重跑 SQL。
