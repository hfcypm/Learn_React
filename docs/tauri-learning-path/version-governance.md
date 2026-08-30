# Tauri 版本边界与迁移

**目标**：让学习内容、示例项目和生产项目拥有清晰的版本边界，降低大版本升级和 API 混用风险。

## 1. Tauri 1 与 Tauri 2 的差异

| 能力 | Tauri 1 | Tauri 2 |
|---|---|---|
| 配置 | `tauri.conf.json` 结构单一 | 拆分为 build/app/bundle/plugins 多块 |
| 权限 | ACL 实验性、默认放开 | capabilities 显式声明，默认收敛 |
| 插件 | 部分内置（fs/dialog 等） | 独立插件 crate，按需安装 |
| 前端 API | `@tauri-apps/api` 单一包 | 按插件分包，如 `@tauri-apps/plugin-fs` |
| 移动端 | 不支持 | 支持 iOS 与 Android |
| Rust API | `Window`/`AppHandle` 生命周期管理 | `Manager` 统一，运行时类型参数化 |

## 2. 每个示例必须记录的元数据

```markdown
> 适用范围：Tauri 2 + React/TypeScript + Rust
> 需要的工具：Node.js、npm、Rust 工具链、系统 WebView
> 验证命令：npm run tauri dev、npm run tauri build
> 最后复核日期：YYYY-MM-DD
```

## 3. 迁移要点（1.x -> 2.x）

- 升级 Cargo 依赖到 `tauri` 2.x，更新 `Cargo.toml` 中的 tauri-build。
- 重写 `tauri.conf.json` 为 2.x 结构，拆分 `bundle` 与 `plugins`。
- 把系统能力迁移为对应插件，并在 capabilities 声明权限。
- 前端把 `@tauri-apps/api` 中的内置能力迁移到插件分包。
- 用 `cargo tauri migrate` 辅助迁移，官方提供迁移工具。

## 4. 升级前检查

1. 阅读官方 [升级指南](https://v2.tauri.app/start/migrate/)。
2. 检查配置结构、插件依赖与权限声明。
3. 检查前端 API 导入路径。
4. 检查自定义命令的 Rust API 兼容性。
5. 在分支环境完整验证 dev 与 build。

## 5. 升级后检查

- `tauri dev` 与 `tauri build` 正常。
- 所有插件能力可用且权限声明匹配。
- 事件与命令行为一致。
- 更新机制与签名正常。
- 产物体积与启动行为符合预期。
