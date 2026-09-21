# Perfetto 学习评估与项目评分

**目标**：通过概念、实操、排错和项目评审确认学习者能独立把红帧拆到线程与切片，避免只会拖时间轴。

## 1. 阶段考核

### 阶段零：基础与工具链

- 解释 Perfetto 采集端与 UI / Trace Processor 的分工。
- 默写 vsync -> `doFrame` -> RenderThread -> 上屏。
- 正确选择卡顿、Java 泄漏、native 堆各自的工具。

### 阶段一：录制与 UI

- 用 Studio 或 `record_android_trace` 录 10–20 秒。
- 说明 `-a`、`gfx/view`、FrameTimeline 数据源。
- 在 UI 中找到包名、主线程、RenderThread。

### 阶段二：FrameTimeline

- 解释 Expected 与 Actual。
- 根据颜色区分 App jank、SF jank、丢帧。
- 把红帧对到 `doFrame` 的 animation / draw / traversal。
- 解释 Sleeping 与 waker。

### 阶段三：SQL 与埋点

- 写出列出超时 `doFrame` 的 SQL。
- 解释 Running / Runnable / Sleeping。
- 用 `Trace.beginSection` 打出可搜索切片。
- 说明何时加 Java 方法采样。

### 阶段四：内存与生产

- 说明为什么 Java 泄漏主用 LeakCanary / Studio。
- 说明 heapprofd 适用场景。
- 走完复录对比清单。

## 2. 排错题

- 进程列表没有自己的包。
- 没有 FrameTimeline 轨道。
- `doFrame` 很短但 Actual 是红的。
- `draw` 40ms 几乎全是 Sleeping。
- `animation` 切片覆盖整帧但对不上代码。
- SQL 查不到 `doFrame`（进程名带后缀）。
- 埋点名字搜不到。
- 把 Perfetto hprof 当成跳源码工具。
- 修复后仍红，刷新率其实是 120Hz。

## 3. 项目评分

总分 100 分，70 分通过，90 分优秀。参考 [综合实战：个股详情页卡顿排查](./project-practice.md)：

| 维度 | 分值 | 评分重点 |
|---|---:|---|
| 录制质量 | 15 | 时长、单场景、文件可打开、探针齐全 |
| 拆帧 | 25 | 颜色、Actual、`doFrame` 子切片、Sleeping/waker |
| 定量 | 20 | SQL Top N、阈值与刷新率一致 |
| 埋点与验证 | 20 | 切片能搜到，修复前后可对比 |
| 结论与边界 | 20 | 根因一句话、工具分工正确、回归指标 |

## 4. 项目评审问题

- 为什么 Actual Timeline 可能比 `doFrame` 长？
- 红、黄、蓝分别改谁的代码？
- 主线程 Sleeping 40ms 时，优化方向为什么先找 waker？
- ViewPager 邻页仍 attach 会怎样污染时间线？
- `-a '*'` 有什么风险？
- 什么时候必须上 Java 方法采样？
- heapprofd 和 LeakCanary 各回答什么问题？
- APA 打开同一份 trace 后，分析步骤是否改变？

## 5. 学习记录模板

```markdown
## Perfetto 阶段记录

- 本阶段目标：
- 使用的 trace 文件：
- 红帧时间窗：
- doFrame / draw / animation 数据：
- SQL 结论：
- 遇到的问题：
- 根因分析：
- 修复与复录对比：
- 尚未掌握的概念：
- 下一阶段行动：
```
