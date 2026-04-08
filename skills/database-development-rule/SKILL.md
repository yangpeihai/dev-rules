---
name: database-development-rule
description: Use when designing or reviewing database tables, MySQL DDL, SQL statements, indexes, audit fields, or MyBatis-Plus entity mappings, especially when Codex needs to enforce team database naming, type, constraint, charset, indexing, and optimistic-lock conventions.
---

# 数据库开发规范

## 概述

该 skill 用于统一数据库表设计、SQL 编写和 MyBatis-Plus 映射规则。处理建表、字段设计、索引设计、SQL 审查或 ORM 映射时，先按本 skill 检查，再给出建议或修改方案。

## 使用方式

1. 先判断当前任务属于哪一类：
   - 新建表或修改表结构
   - 审查 DDL / SQL
   - 审查 MyBatis-Plus 实体或自动填充配置
2. 读取 [references/mysql-database.md](references/mysql-database.md) 中对应章节。
3. 输出结论时优先覆盖以下高频问题：
   - 表名、字段名、索引名是否符合统一命名
   - 是否包含 5 个通用审计字段
   - 主键、外键、状态、布尔、金额、时间字段类型是否正确
   - 是否错误使用 `TIMESTAMP`、`FLOAT`、`DOUBLE`、`NULL`
   - 索引是否与查询路径匹配，是否过多或缺失
   - SQL 是否存在 `SELECT *`、无条件更新删除、分页无排序等问题
   - MyBatis-Plus 是否正确配置自动填充与 `@Version`

## 快速判定

- 出现 `CREATE TABLE`、`ALTER TABLE`、索引设计、字段设计问题时，重点检查命名、类型、约束、字符集和审计字段。
- 出现查询优化、联表查询、分页查询问题时，重点检查 SQL 格式、别名、排序、索引命中和禁止项。
- 出现 Java Entity、`@TableName`、`@TableField(fill=...)`、`@Version` 时，重点检查 MyBatis-Plus 映射规范。

## 输出要求

- 审查时明确指出违反项、影响和推荐修复方式。
- 写 DDL / SQL 时直接产出符合规范的结果，不要只复述规则。
- 若用户给出的原始规范与 MySQL 事实冲突，优先输出可执行、可落地的版本，并说明调整原因。

## 重点约束

- 表名禁止复数，推荐单数业务名，如 `mcp_server`、`tool`
- 所有字段必须 `NOT NULL` 且必须有默认值
- 所有表必须包含：`insert_user`、`insert_date`、`update_user`、`update_date`、`version`
- 主键必须为 `BIGINT NOT NULL AUTO_INCREMENT`
- 金额必须为 `DECIMAL`
- 布尔必须为 `TINYINT(1)`
- 时间优先使用 `DATETIME`，禁止 `TIMESTAMP`
- 字符集必须为 `utf8mb4`
- 单表索引建议不超过 5 个

## 参考资料

- 详细规范见 [references/mysql-database.md](references/mysql-database.md)
- 需要审查 Java / Spring / MyBatis 代码时，可联动使用现有的 `java-rule`
