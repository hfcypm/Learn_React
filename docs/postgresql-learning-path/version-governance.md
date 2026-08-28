# PostgreSQL 版本边界与迁移

**目标**：让学习内容、示例项目和生产项目拥有清晰的版本边界，降低大版本升级和功能混用风险。

## 1. 版本节奏

PostgreSQL 每年发布一个大版本（如 16、17、18）。大版本决定默认参数、新特性与弃用行为。生产库应跟随受支持版本（5 年维护窗口）。

## 2. 当前主线版本

学习主线为 PostgreSQL 17，注意以下版本相关行为：

- `EXPLAIN (ANALYZE, BUFFERS)` 等选项长期稳定。
- 部分函数与操作符在新版本增强，文档以官方当前版本为准。
- 默认 `max_wal_senders`、`wal_level` 等参数随版本调整，生产配置以安装版本为准。

## 3. 每个示例必须记录的元数据

```markdown
> 适用范围：PostgreSQL 17
> 需要的工具：psql、pg_dump
> 验证命令：EXPLAIN ANALYZE、备份恢复演练
> 最后复核日期：YYYY-MM-DD
```

## 4. 大版本升级

升级 PostgreSQL 大版本使用 `pg_upgrade` 或逻辑备份恢复：

```bash
# 逻辑方式（小数据量）
pg_dumpall > all.sql
# 新版本实例
psql -f all.sql

# 物理方式（大数据量）
pg_upgrade --old-datadir /var/lib/pg16 --new-datadir /var/lib/pg17
```

## 5. 升级前检查

1. 阅读目标版本 release notes。
2. 确认被移除或行为变化的特性。
3. 在测试环境执行完整升级演练。
4. 备份并验证可恢复。
5. 检查第三方扩展（如 PostGIS）的兼容版本。
6. 确认应用连接串与客户端库版本兼容。

## 6. 升级后检查

- 基础查询与报表结果一致。
- 所有索引有效（可 `REINDEX`）。
- 统计信息已重新分析。
- 复制、备份、监控恢复正常。
- 慢查询对比升级前后。
