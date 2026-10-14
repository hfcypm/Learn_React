# HarmonyOS 版本边界与迁移

**目标**：让学习内容、示例项目和生产项目拥有清晰的版本边界，降低大版本升级和 API 混用风险。

## 1. 版本与 API 的配套关系

鸿蒙的版本由“设备系统版本 + API 版本 + DevEco Studio + SDK”四者配套决定。使用配套关系一致的开发套件，才能获得稳定体验。

| 层级 | 说明 |
|---|---|
| 设备系统版本 | 用户设备上的系统版本（如 HarmonyOS NEXT 5.x / 6.x）|
| API 版本 | SDK 提供的能力范围，设备支持能力以 API 版本为准 |
| DevEco Studio | 官方 IDE，SDK 已内嵌，安装即配套 |
| SDK | 随 DevEco Studio 自动安装，含 Toolchains、System Image 等 |

查询方式：

- 设备：设置 -> 关于本机，查看设备 API 版本。
- IDE 版本：DevEco Studio 菜单 Help -> About DevEco Studio。
- SDK 版本：DevEco Studio 菜单 Help -> About HarmonyOS SDK。

## 2. 应用模型的代际差异

| 维度 | 旧版（FA 模型） | 当前（Stage 模型） |
|---|---|---|
| 入口单元 | PageAbility / ServiceAbility | UIAbility / ExtensionAbility |
| 页面描述 | config.json + pages | module.json5 + pages + Navigation |
| 路由 | `Router`（页面级） | `Navigation` + `NavPathStack`（推荐） |
| 进程模型 | 单进程为主 | 支持多进程与多实例 |
| 多设备 | 支持有限 | 原生支持元服务与多端部署 |

新项目一律使用 Stage 模型；`Router` 已不推荐，新代码用 `Navigation`。

## 3. 语言与编译的代际差异

| 维度 | 说明 |
|---|---|
| ArkTS 1.0 | 当前主流，TypeScript 超集，运行时支持部分动态特性 |
| ArkTS 1.2 / 静态模式 | 更严格的静态类型与并发安全约束，逐步推广 |
| ArkCompiler | AOT 编译，提前编译为机器码，提升启动与运行性能 |

迁移时关注：严格模式是否禁止 `any`、是否限制运行时改对象结构、并发对象是否需要 `@Sendable`。这些约束会影响既有代码能否直接运行。

## 4. 状态管理 V1 与 V2 的迁移

V2 提供了更精准的观测与刷新控制。迁移建议采用渐进式策略，避免一次性重写：

| V1 | V2 | 迁移要点 |
|---|---|---|
| `@Observed` / `@ObjectLink` | `@ObservedV2` / `@Trace` | 用 `@Trace` 标注需观测的属性，支持深层观测 |
| `@State` | `@Local` / `@Param`+`@Once` | `@Local` 不能外部初始化 |
| `@Prop` | `@Param` | `@Param` 传引用，`@Prop` 深拷贝 |
| `@Link` | `@Param`+`@Event` | 手动实现双向同步 |
| `@Provide` / `@Consume` | `@Provider` / `@Consumer` | 跨层双向同步 |
| `@Watch` | `@Monitor` | `@Monitor` 可获取新旧值 |

约束：单个组件内不混用 V1 与 V2 装饰器。建议新组件直接写 V2，旧组件在改动时逐步迁移。

## 5. 每个示例必须记录的元数据

```markdown
> 适用范围：HarmonyOS NEXT 5.x+（API 14+）+ ArkTS + ArkUI
> 需要的工具：DevEco Studio（配套版本）、ohpm、hvigorw
> 验证命令：hvigorw assembleHap --mode module -p product=default
> 最后复核日期：YYYY-MM-DD
```

## 6. 升级前检查

1. 阅读官方版本说明与变更日志，确认配套关系。
2. 检查工程配置：`build-profile.json5` 的 `compatibleSdkVersion` / `targetSdkVersion`、`runtimeOS`。
3. 检查是否仍在使用 `Router`，规划迁移到 `Navigation`。
4. 检查状态管理是否混用 V1/V2，规划 V2 迁移。
5. 检查权限声明与运行时申请是否调整。
6. 检查三方库对目标 API 版本的兼容性。
7. 在分支环境完整验证 dev 与 build。

## 7. 升级后检查

- 冷启动与页面跳转正常。
- 状态刷新范围符合预期，无过度刷新或漏刷新。
- 权限与系统能力调用正常。
- RDB / Preferences 数据读写正常，无迁移丢失。
- 多设备布局与分布式能力正常。
- 构建产物可签名、可安装、可上架。

## 8. 迁移常见坑

- 直接升级 IDE 与 SDK 但未更新 `build-profile.json5`，导致构建目标错位。
- 新工程模板已用 V2/Stage，旧代码仍用 V1/FA，混用报错。
- 跨版本传递可观察对象，序列化后属性名被加前缀导致反序列化异常。
- 并发对象未加 `@Sendable`，在更严格的运行时下报错。
- 依赖三方库未适配目标 API 版本，运行期能力缺失。
