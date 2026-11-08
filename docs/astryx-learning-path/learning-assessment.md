# Astryx 学习评估与项目评分

**目标**：用阶段考核、排错题与项目评分确认是否真正掌握 Astryx 的包模型、主题 Token、组件契约与升级纪律，避免只会复制 Playground。

参考 [综合实战：内部订阅管理后台](./project-practice.md)。

## 1. 阶段考核

### 入门零：设计系统与工具链

- 解释 Core、主题包、CLI、`@stylexjs/stylex` 的职责。
- 说明为何要求 React 19。
- 完成安装、`init`、`theme add`，并用 `<Theme>` 渲染 Button。
- 从 `@astryxdesign/core/Button` 导入，而不是猜测根 barrel。

### 入门一：核心组件与主题

- 用 VStack / HStack / Heading / Card 搭页头与指标区。
- 解释 Token 与硬编码颜色的差别。
- 用 `mode="system"` 验证明暗色。
- 用 `className` 或 `xstyle` 做一次局部覆盖。

### 进阶二：表单与浮层

- 写出带 `label`、`isRequired`、`status` 的受控 TextInput。
- 区分 `containerXstyle` 与 `controlXstyle`。
- 完成对话框新建流：焦点、Esc、校验、保存。
- 说明共享 dismissal 栈的意义。

### 进阶三：数据与 AI 工作流

- 用 Table 展示列表并处理空态。
- 用 `template` 拷贝或对照一页官方页面。
- 用 CLI 查出指定组件的 props。
- 说明 MCP / `AGENTS.md` 如何降低幻觉 props。

### 进阶四：定制与生产

- 用 `defineTheme({ extends })` 覆盖强调色。
- 说明 cascade layer 与 Tailwind 冲突时的排查顺序。
- 说明 swizzle 的成本与适用时机。
- 写出 Core 与主题必须同发版升级的步骤。

## 2. 排错题

给出现象，要求排查顺序与至少两个可能原因：

- `react` 18 项目安装 Core 失败或运行异常。
- 按钮完全无设计系统样式。
- 找不到 `./astryx-themes`。
- 系统暗色下页面仍是浅色。
- Tailwind 的 `bg-white` 盖住 Card。
- TextInput 上传 `className` 报类型错误。
- 校验信息没有出现。
- 对话框打开后焦点仍在背后。
- AI 生成了不存在的 `variant`。
- 只升级 Core 后组件与颜色错位。
- `upgrade` 后输入框样式丢失。

## 3. 项目评分

总分 100，70 通过，90 优秀。

| 维度 | 分值 | 评分重点 |
|---|---:|---|
| 接入与 Theme | 16 | React 19、CSS 导入、单根 Theme、子路径导入 |
| 布局与核心组件 | 16 | Stack、Heading、Button、Card、Token |
| 表单与浮层 | 18 | label、status、必填、对话框焦点 |
| 数据展示 | 14 | Table、空态、行操作确认 |
| 主题定制 | 12 | extends、明暗元组、无大面积 hex |
| CLI / Agent | 12 | script、查组件、AGENTS.md |
| 生产与升级意识 | 12 | 同发版、layer、swizzle 克制 |

## 4. 项目评审问题

- Astryx 和 shadcn/ui 在分发模型上有什么不同？
- 为什么组件要按 `@astryxdesign/core/Button` 导入？
- `<Theme>` 不包根节点时会出现什么？
- Token 写成 `[light, dark]` 元组解决什么问题？
- 为什么输入组件不能再用根级 `xstyle`？
- 人和 AI 为什么要共用 CLI 契约？
- 什么情况下才 swizzle？
- 为什么主题包必须和 Core 一起升？
- 无 `@layer` 的全局 CSS 为什么能压过组件样式？
- 如何证明新建流程对键盘用户可用？

## 5. 学习记录模板

```markdown
## Astryx 学习记录

- 本阶段目标：
- 已跑通的功能：
- 使用的组件与 CLI 命令：
- 遇到的样式或类型问题：
- 根因分析：
- 修复方式与验证：
- 尚未理解的概念：
- 下一阶段行动：
```
