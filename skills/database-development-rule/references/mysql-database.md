# 数据库开发规范

**版本**: 1.0  
**适用范围**: 所有数据库表设计、DDL、DML、查询 SQL 与 MyBatis-Plus 映射  
**参考来源**: 阿里巴巴 Java 开发手册（数据库部分），并结合本仓库现有 Java / MyBatis-Plus 使用习惯做了落地约束

## 1. 命名规范

### 1.1 表命名

- 使用 `小写字母 + 下划线`
- 业务表按业务模块命名，如 `sys_user`、`mcp_server`
- 关联表使用 `table1_table2_rel`
- 字典表使用 `dict_xxx`
- 禁止使用复数形式，如 `users`
- 禁止使用拼音、无意义缩写、保留字

### 1.2 字段命名

- 使用 `小写字母 + 下划线`
- 布尔字段使用 `is_` 前缀，如 `is_enabled`、`is_deleted`
- 外键字段统一使用 `{关联表名}_id`
- 禁止使用保留字、大写字母、拼音

### 1.3 索引命名

- 主键索引：`pk_表名`
- 唯一索引：`uk_表名_字段名`
- 普通索引：`idx_表名_字段名`
- 联合索引：`idx_表名_字段1_字段2`

## 2. 通用字段规范

### 2.1 每张表必须包含的 5 个字段

| 字段名 | 类型 | 约束 | 说明 |
|--------|------|------|------|
| `insert_user` | `VARCHAR(64)` | `NOT NULL DEFAULT 'system'` | 创建人 |
| `insert_date` | `DATETIME` | `NOT NULL DEFAULT CURRENT_TIMESTAMP` | 创建时间 |
| `update_user` | `VARCHAR(64)` | `NOT NULL DEFAULT 'system'` | 最后修改人 |
| `update_date` | `DATETIME` | `NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP` | 最后修改时间 |
| `version` | `BIGINT` | `NOT NULL DEFAULT 0` | 乐观锁版本号 |

### 2.2 重要说明

- `update_user` 不要写成 `ON UPDATE insert_user`
- MySQL 不支持对 `VARCHAR` 字段使用这种 `ON UPDATE` 表达式
- `update_user` 由应用层自动维护，推荐通过 MyBatis-Plus 自动填充实现
- `insert_user`、`update_user` 默认值可用 `system` 兜底，但真实业务写入必须覆盖为当前操作人

## 3. 数据类型规范

### 3.1 字符串类型

| 类型 | 使用场景 | 约束 |
|------|----------|------|
| `VARCHAR` | 可变长字符串 | 推荐长度小于等于 5000，预留合理余量 |
| `CHAR` | 固定长度标识 | 仅用于 MD5、UUID 等固定长度值 |
| `TEXT` | 长文本 | 仅用于大文本、JSON、日志等 |

补充要求：

- 手机号使用 `VARCHAR(11)`，若需国际化支持需单独扩展
- 身份证号使用 `VARCHAR(18)`
- URL、路径、编码串按真实上限留余量，不要盲目给 `VARCHAR(255)`
- `VARCHAR` 长度优先按业务实际长度设计，并预留合理余量
- 对标识类、编码类、短文本类字段，推荐优先考虑 `2` 的 `n` 次方长度，如 `32`、`64`、`128`、`256`
- 对名称、URL、描述、备注等字段，不要求长度必须为 `2` 的 `n` 次方，应按真实业务上限设置

### 3.2 数值类型

| 类型 | 使用场景 | 示例 |
|------|----------|------|
| `BIGINT` | 主键、外键、数量、版本号 | `id BIGINT NOT NULL AUTO_INCREMENT` |
| `INT` | 普通整型数值 | `sort INT NOT NULL DEFAULT 0` |
| `DECIMAL` | 金额、精度数值 | `amount DECIMAL(10,2) NOT NULL DEFAULT 0.00` |
| `TINYINT` | 状态、布尔、枚举值 | `is_enabled TINYINT(1) NOT NULL DEFAULT 1` |

补充要求：

- 金额必须使用 `DECIMAL`，禁止使用 `FLOAT` / `DOUBLE`
- 所有字段必须 `NOT NULL`
- 所有字段必须有默认值
- 主键必须使用 `BIGINT`，禁止使用 `INT`
- 禁止使用 `UNSIGNED`

### 3.3 时间类型

| 类型 | 使用场景 | 格式 |
|------|----------|------|
| `DATETIME` | 日期时间 | `yyyy-MM-dd HH:mm:ss` |
| `DATE` | 日期 | `yyyy-MM-dd` |
| `TIME` | 时间 | `HH:mm:ss` |

补充要求：

- 优先统一为 `DATETIME`
- 禁止使用 `TIMESTAMP`，避免 2038 年问题与时区副作用

### 3.4 布尔类型

布尔值统一使用 `TINYINT(1)`：

- `0` 表示 `false`
- `1` 表示 `true`

示例：

```sql
is_enabled TINYINT(1) NOT NULL DEFAULT 1 COMMENT '是否启用：1=启用，0=禁用'
```

## 4. 字段设计规范

### 4.1 主键

- 字段名统一为 `id`
- 类型为 `BIGINT`
- 约束为 `NOT NULL AUTO_INCREMENT`

```sql
id BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键ID'
```

### 4.2 外键字段

- 命名为 `{关联表名}_id`
- 类型为 `BIGINT`
- 必须 `NOT NULL`

```sql
server_id BIGINT NOT NULL COMMENT '所属服务器ID'
```

### 4.3 状态字段

- 命名为 `status` 或 `{业务}_status`
- 可使用 `VARCHAR(20)` 或 `TINYINT`
- 推荐优先使用字符串枚举，提高 SQL 可读性

```sql
status VARCHAR(20) NOT NULL DEFAULT 'STOPPED' COMMENT '状态：STOPPED/RUNNING/ERROR'
```

### 4.4 逻辑删除字段

- 推荐命名为 `is_deleted`
- 类型使用 `TINYINT(1)`
- 默认值为 `0`

```sql
is_deleted TINYINT(1) NOT NULL DEFAULT 0 COMMENT '逻辑删除：0=未删除，1=已删除'
```

### 4.5 金额字段

```sql
amount DECIMAL(10,2) NOT NULL DEFAULT 0.00 COMMENT '金额'
```

## 5. 约束与注释规范

### 5.1 NOT NULL

- 所有字段必须定义为 `NOT NULL`
- 不允许依赖业务约定隐式处理 `NULL`

### 5.2 默认值

- 字符串默认值使用 `''` 或明确业务默认值
- 数值默认值使用 `0`
- 日期时间默认值使用 `CURRENT_TIMESTAMP`
- 布尔默认值使用 `0` 或 `1`

### 5.3 COMMENT

- 所有表必须定义表注释
- 所有字段必须定义字段注释
- 索引建议通过命名表达语义，必要时在设计文档中补充说明

## 6. 索引规范

### 6.1 索引设计原则

1. 主键索引自动创建，不再重复定义
2. 业务唯一字段必须建立唯一索引
3. `WHERE`、`ORDER BY`、`JOIN` 高频使用字段建立普通索引
4. 联合索引必须遵循最左前缀原则
5. 单表索引数量建议不超过 5 个
6. 先设计查询路径，再反推索引，不为“可能会查”创建索引

### 6.2 索引定义示例

```sql
PRIMARY KEY (id)

UNIQUE KEY uk_mcp_server_name (name)
KEY idx_mcp_server_status (status)
KEY idx_tool_server_id (server_id)
KEY idx_audit_log_server_id_action (server_id, action)
```

## 7. 表设计规范

### 7.1 存储引擎

```sql
ENGINE=InnoDB
```

### 7.2 字符集

```sql
DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
```

### 7.3 建表示例

```sql
CREATE TABLE tool (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键ID',
    server_id BIGINT NOT NULL COMMENT '所属服务器ID',
    name VARCHAR(255) NOT NULL DEFAULT '' COMMENT '工具名称',
    endpoint_url VARCHAR(2048) NOT NULL DEFAULT '' COMMENT '执行端点URL',
    http_method VARCHAR(10) NOT NULL DEFAULT 'POST' COMMENT 'HTTP方法',
    status VARCHAR(20) NOT NULL DEFAULT 'STOPPED' COMMENT '状态：STOPPED/RUNNING/ERROR',
    is_enabled TINYINT(1) NOT NULL DEFAULT 1 COMMENT '是否启用：1=启用，0=禁用',
    is_deleted TINYINT(1) NOT NULL DEFAULT 0 COMMENT '逻辑删除：0=未删除，1=已删除',
    insert_user VARCHAR(64) NOT NULL DEFAULT 'system' COMMENT '创建人',
    insert_date DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    update_user VARCHAR(64) NOT NULL DEFAULT 'system' COMMENT '最后修改人',
    update_date DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后修改时间',
    version BIGINT NOT NULL DEFAULT 0 COMMENT '乐观锁版本号',
    PRIMARY KEY (id),
    UNIQUE KEY uk_tool_name_server_id (name, server_id),
    KEY idx_tool_server_id (server_id),
    KEY idx_tool_status (status),
    KEY idx_tool_is_enabled (is_enabled)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='工具配置表';
```

## 8. SQL 编写规范

### 8.1 关键字大写

```sql
SELECT id, name
FROM mcp_server
WHERE status = 'RUNNING';
```

### 8.2 多行对齐

```sql
SELECT
    id,
    name,
    port,
    status
FROM mcp_server
WHERE status = 'RUNNING'
  AND port > 3000
ORDER BY id DESC;
```

### 8.3 显式别名

```sql
SELECT
    ms.id AS server_id,
    ms.name AS server_name,
    t.name AS tool_name
FROM mcp_server AS ms
LEFT JOIN tool AS t ON t.server_id = ms.id;
```

### 8.4 查询约束

- 禁止无条件 `DELETE`、`UPDATE`
- 禁止线上使用 `SELECT *`
- 分页查询必须显式排序
- 大批量操作必须评估锁、索引与事务范围

## 9. MyBatis-Plus 映射规范

### 9.1 实体映射示例

```java
@TableName("mcp_server")
public class McpServer {

    @TableId(type = IdType.AUTO)
    private Long id;

    private String name;

    @NotNull
    @Min(1)
    @Max(65535)
    private Integer port;

    @TableField(fill = FieldFill.INSERT)
    private String insertUser;

    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime insertDate;

    @TableField(fill = FieldFill.INSERT_UPDATE)
    private String updateUser;

    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateDate;

    @Version
    private Long version;
}
```

### 9.2 自动填充示例

```java
@Component
public class MetaObjectHandlerConfig implements MetaObjectHandler {

    @Override
    public void insertFill(MetaObject metaObject) {
        String currentUser = getCurrentUser();
        LocalDateTime now = LocalDateTime.now();

        this.strictInsertFill(metaObject, "insertUser", String.class, currentUser);
        this.strictInsertFill(metaObject, "insertDate", LocalDateTime.class, now);
        this.strictInsertFill(metaObject, "updateUser", String.class, currentUser);
        this.strictInsertFill(metaObject, "updateDate", LocalDateTime.class, now);
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        this.strictUpdateFill(metaObject, "updateUser", String.class, getCurrentUser());
        this.strictUpdateFill(metaObject, "updateDate", LocalDateTime.class, LocalDateTime.now());
    }

    private String getCurrentUser() {
        return SecurityContextHolder.getContext().getAuthentication().getName();
    }
}
```

## 10. 审查与交付清单

提交前至少确认以下事项：

- [ ] 表名、字段名、索引名均符合统一命名规范
- [ ] 主键为 `BIGINT NOT NULL AUTO_INCREMENT`
- [ ] 每张表包含 5 个通用审计字段
- [ ] 所有字段均为 `NOT NULL`
- [ ] 所有字段均有默认值
- [ ] 所有字段均有 `COMMENT`
- [ ] 金额字段使用 `DECIMAL`
- [ ] 布尔字段使用 `TINYINT(1)`
- [ ] 时间字段使用 `DATETIME`，未使用 `TIMESTAMP`
- [ ] 字符集为 `utf8mb4`
- [ ] 索引数量与查询模式匹配，且不超过建议上限
- [ ] SQL 关键字已大写，复杂 SQL 已按块对齐
- [ ] MyBatis-Plus 自动填充与乐观锁配置完整
