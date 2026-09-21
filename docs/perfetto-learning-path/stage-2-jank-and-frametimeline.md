# 阶段二：卡顿与 FrameTimeline

**目标**：用 Expected / Actual Timeline 判定红帧，把一帧拆成 `doFrame`、`draw`、RenderThread，并能解释「主线程 Sleeping 40ms」这类现象。

## 1. 两条时间线

每个有上屏帧的应用会多出两条轨道（SurfaceFlinger 自己也有一对）：

| 轨道 | 含义 |
|---|---|
| Expected Timeline | 系统预测这一帧应该在哪个 vsync 窗口完成 |
| Actual Timeline | 这一帧真正完成呈现的时间 |

Actual 的终点取「应用 post 缓冲」与「GPU 做完」中较晚的那个。所以 Actual 很长，不一定等于主线程 `doFrame` 很长。

## 2. 颜色怎么读

| 颜色 | 含义 | 下一步 |
|---|---|---|
| 绿 | 按时完成 | 通常忽略 |
| 浅绿 | 高延迟但仍赶上 | 关注连续出现时的趋势 |
| 红 | 本进程导致的 jank | 进主线程 / RenderThread |
| 黄 | SurfaceFlinger 侧 jank | 看 SF 进程，应用侧次要 |
| 蓝 | 丢帧 | 看是否来不及提交或被 SF 丢掉 |

选中一条 Actual slice，Current Selection 里关键字段：

- Present Type：early / on time / late
- Jank Type：App Deadline Missed、Buffer Stuffing、SurfaceFlinger 等
- On time finish：CPU 工作是否在截止前结束
- GPU composition：是否走 GPU 合成
- Layer name：哪一层（帮助确认是不是你的 Activity）

官方分类函数：`android_is_app_jank_type`、`android_is_sf_jank_type`、`android_is_missed_frame_type`。红帧先确认是 App 侧再改代码。

## 3. 从红帧跳到 `doFrame`

1. 点红的 Actual Timeline slice，记下起止时间。
2. 同一时间窗看主线程的 `Choreographer#doFrame`。
3. 展开 `doFrame` 内部：

```text
Choreographer#doFrame
  ├── input
  ├── animation          <- ValueAnimator / ObjectAnimator 回调
  ├── traversal
  │     ├── measure
  │     ├── layout
  │     └── draw         <- View.onDraw / 构建 DisplayList
  └── ...
```

4. 并列看 RenderThread 的 `DrawFrame`。主线程 `draw` 结束只代表 DisplayList 建完，GPU 工作在 RenderThread 之后。

标准库函数 `android_first_frame_after(ts)` 可把时间戳映射到 `do_frame_id`、`draw_frame_id`、`actual_frame_timeline_id`。

## 4. 典型模式

### 4.1 `doFrame` 很长，内部 `draw` 很长

View 树过重、自定义 `onDraw` 算路径 / 量价、一次 invalidate 整页。对策：减少每帧绘制、分层缓存、把计算移出绘制路径。

### 4.2 `doFrame` 很长，内部几乎全是 `animation`

`ValueAnimator` 的 `onAnimationUpdate` 里做了布局、请求行情、刷新多个自定义 View。一条 animation slice 包住整帧甚至连续多帧时，去搜 `addUpdateListener`。

实战里出现过：`animation` 233ms 且只包 1 个 `doFrame`。这表示动画回调本身把这一帧撑爆，或回调里又同步触发了 traversal。需要对 Java 方法采样或加 `Trace.beginSection`。

### 4.3 `doFrame` 不长，Actual 却是红的

主线程按时画完，但 GPU / SF / 缓冲堆积晚了。看 RenderThread、GPU 计数、SF 的 FrameTimeline。股票 App 相对少见，先排除主线程再下这个结论。

### 4.4 主线程 `draw` 显示 Sleeping

例如 `draw` 约 40.7ms，其中 Sleeping 40.22ms。墙钟时间长，CPU 时间短：线程在等。

点 Sleeping 段看：

- waker 是 RenderThread：往往在等渲染相关同步。
- waker 是 Binder：主线程在等跨进程（行情服务、系统服务）。
- 无 wank 或等锁：看 `monitor contention` / `Lock contention` slice。

不要把 Sleeping 理解成「onDraw 算了 40ms」。

### 4.5 ViewPager 邻页

ViewPager 会保留左右页。邻页 Fragment 仍 attach 时，它们的 View 仍可能参与绘制或动画。红帧时间窗里搜索邻页相关自定义 View 的 atrace 名。

## 5. 帧预算对照

| 刷新率 | 预算 |
|---|---|
| 60Hz | ≈16.67ms |
| 90Hz | ≈11.11ms |
| 120Hz | ≈8.33ms |

设备是 120Hz 时，16ms 的 `doFrame` 已经可能红。先看 Expected Timeline 的窗口长度，再和 `doFrame.dur` 比。

## 6. 最小分析记录模板

```markdown
## 帧记录

- 操作：打开个股详情并停留分时图
- 红帧时间：trace 内 3.420s – 3.461s
- Jank Type：
- doFrame dur：
- 其中 draw / animation / traversal：
- 主线程状态：Running / Sleeping（waker：）
- RenderThread DrawFrame dur：
- 结论（一句话）：
- 下一步验证：
```

## 7. 动手任务

1. 在已有 trace 里找出 3 条红 Actual。
2. 各写一条「帧记录」。
3. 至少找出一条 Sleeping 主导的帧，记下 waker。
4. 至少找出一条 animation 或 draw 主导的帧。

## 阶段二验收

- 能解释 Expected 与 Actual 的差别。
- 能根据颜色判断 App jank vs SF jank。
- 能把红帧对到 `doFrame` 子切片。
- 能区分「算得慢」和「等得久」。

## 常见排错

| 现象 | 排查方向 |
|---|---|
| 没有 FrameTimeline | 系统过旧；config 漏了 `android.surfaceflinger.frametimeline` |
| 满屏红但操作很轻 | 录到了掉帧风暴或缓冲 stuffing，先找连续红帧的第一帧 |
| 只盯 GPU | 先看主线程；行情 App 多数是 CPU / 主线程 |
| 把邻页动画算到当前页 | 看 layer name 与 ViewPager 离屏页是否仍 attach |

## 进入下一阶段的条件

你能对一条红帧写出「谁超时、时间花在算还是等」。此时进入 [阶段三：SQL、调度与自定义埋点](./stage-3-sql-and-cpu.md)。
