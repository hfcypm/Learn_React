# 阶段三：SQL、调度与自定义埋点

**目标**：用 Trace Processor SQL 量化 jank，把主线程 Sleeping 对到调度与锁，并用 `Trace.beginSection` 让业务回调出现在时间线上。

## 1. 在 UI 里跑 SQL

Perfetto UI 左侧打开 Query (SQL)。Trace Processor 把整份 trace 变成表：`slice`、`thread`、`process`、`sched`、`actual_frame_timeline_slice` 等。

先限定自己的进程，避免扫到 system_server。

## 2. 列出超时的 `doFrame`

```sql
SELECT
  s.ts,
  s.dur,
  s.dur / 1e6 AS dur_ms,
  thread.name AS thread_name,
  process.name AS process_name
FROM slice s
JOIN thread_track tt ON s.track_id = tt.id
JOIN thread USING (utid)
JOIN process USING (upid)
WHERE s.name = 'Choreographer#doFrame'
  AND process.name = 'com.tzt.android'
  AND s.dur > 16 * 1000 * 1000
ORDER BY s.dur DESC
LIMIT 50;
```

`dur` 单位是纳秒。16ms 阈值按设备刷新率改：120Hz 用 8ms。

## 3. 看 `doFrame` 内部子切片

```sql
SELECT
  child.name,
  child.dur / 1e6 AS dur_ms
FROM slice parent
JOIN slice child ON child.parent_id = parent.id
JOIN thread_track tt ON parent.track_id = tt.id
JOIN thread USING (utid)
JOIN process USING (upid)
WHERE parent.name = 'Choreographer#doFrame'
  AND process.name = 'com.tzt.android'
  AND parent.dur > 16 * 1000 * 1000
ORDER BY parent.ts, child.ts;
```

用这个表区分 animation / traversal / draw 谁在撑帧。

## 4. 标准库：帧与 jank 分类

```sql
INCLUDE PERFETTO MODULE android.frames.timeline;

SELECT *
FROM android_first_frame_after(3.42 * 1e9);
```

```sql
INCLUDE PERFETTO MODULE android.frames.jank_type;

SELECT
  android_is_app_jank_type('App Deadline Missed') AS is_app,
  android_is_sf_jank_type('SurfaceFlinger CPU Deadline Missed') AS is_sf;
```

具体模块名随 Perfetto 版本可能微调，以 UI 的 Query 报错提示和 [stdlib 文档](https://perfetto.dev/docs/analysis/stdlib-docs) 为准。

## 5. 通用 slice 查询

```sql
INCLUDE PERFETTO MODULE slices.with_context;

SELECT ts, dur, name, thread_name, process_name
FROM thread_or_process_slice
WHERE process_name = 'com.tzt.android'
  AND name IN ('animation', 'draw', 'measure', 'layout')
ORDER BY dur DESC
LIMIT 30;
```

ATrace 原始拼表：

```sql
SELECT
  s.ts,
  t.name AS thread_name,
  t.tid,
  s.name AS slice_name,
  s.dur / 1e6 AS dur_ms
FROM slice s
LEFT JOIN thread_track trk ON s.track_id = trk.id
LEFT JOIN thread t ON trk.utid = t.utid
WHERE s.name LIKE '%animation%'
ORDER BY s.dur DESC
LIMIT 20;
```

## 6. 调度：Running / Runnable / Sleeping

主线程墙钟 40ms、CPU 时间 0.5ms，差值就是没在跑。

| 状态 | 含义 |
|---|---|
| Running | 占用某个 CPU |
| Runnable | 想跑但没轮到（看 cpu_frequency、大核小核、cgroup） |
| Sleeping | 主动等待：锁、Binder、休眠、`wait` |

在 UI 中点主线程 Sleeping 段，Current Selection 给出 waker thread。SQL 侧可查 `sched` 与 `thread_state`（表名以当前 Trace Processor 为准）。

Runnable 长时间堆积：CPU 争用，行情推送线程、解码线程、GC 都可能抢核。Sleeping 长时间：先找 Binder 与 monitor contention。

## 7. 自定义埋点

Perfetto 只能显示已采集的名字。要把 `onAnimationUpdate` 对到业务，在 Java 里打 atrace：

```java
import android.os.Trace;

@Override
public void onAnimationUpdate(ValueAnimator animator) {
    Trace.beginSection("QuoteKline:onAnimationUpdate");
    try {
        // 更新分时/K 线
    } finally {
        Trace.endSection();
    }
}
```

自定义 `onDraw` 同样包一层：

```java
@Override
protected void onDraw(Canvas canvas) {
    Trace.beginSection("MinuteView.onDraw");
    try {
        super.onDraw(canvas);
    } finally {
        Trace.endSection();
    }
}
```

规则：

- 名字短、稳定、可搜索，带控件名。
- `begin` / `end` 必须成对，用 try/finally。
- 只埋热点路径：动画回调、行情 `bind(Quote)`、`onMeasure`/`onDraw`。
- 录制时 atrace 必须包含该进程（`-a com.tzt.android` 或 Studio 默认）。

对不上方法名时，再用 Studio CPU：Sample Java Methods 或 Trace Java Methods 录同一操作。System Trace 看帧，Java 采样看栈，两份对照同一时间窗。

## 8. 主线程禁止事项（对着 trace 验证）

- 动画回调里 `requestLayout()` 整棵树。
- `onDraw` 里分配大数组、解 JSON、做均线计算。
- 主线程同步 Binder 等行情服务。
- 主线程 `Sleep` / 锁等待磁盘 IO。

trace 里看到这些 slice 出现在 `doFrame` 内部，就可以当缺陷提交。

## 9. 动手任务

1. 用 SQL 列出本包所有 `doFrame > 16ms`。
2. 对耗时 Top 1 的 `doFrame` 列出子切片。
3. 在一个动画或 `onDraw` 热点加 `Trace.beginSection`，重新录制，确认 UI 能搜到该名字。
4. 找一条 Sleeping 主导的帧，写出 waker 线程名。

## 阶段三验收

- 能在 Perfetto UI 跑通两条业务 SQL。
- 能解释 Running / Runnable / Sleeping 对卡顿的意义。
- 能用 atrace 把业务函数打到时间线上。
- 知道何时加开 Java 方法采样。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| SQL 查不到 `doFrame` | 进程名带后缀 `:push` 等，先 `SELECT DISTINCT name FROM process` |
| 埋点名字搜不到 | 未包含 atrace app；begin/end 不成对；被 16 字符截断（旧 systrace 限制，Perfetto 宽松很多） |
| 只有系统 slice | `-a` 写了 `*` 但缓冲被系统事件占满，改成包名 |
| Java 采样一开就变卡 | 采样本身有开销，只对已定位的时间窗短录 |

## 进入下一阶段的条件

你能用 SQL 证明「哪类切片在超时」，并能用埋点验证猜测。此时进入 [阶段四：内存、APA 与生产流程](./stage-4-memory-and-production.md)。
