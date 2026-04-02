# goctl 工具使用

## 核心原则

- `goctl` 是 go-zero 的默认脚手架工具，适合快速建立标准骨架。
- 生成代码是起点，不是最终架构边界；项目可以在保持职责清晰的前提下渐进演化。
- 手写扩展应尽量避开直接改生成代码，降低再生成和升级成本。

## 安装 goctl（需要使用脚手架或代码生成时）

```bash
# 安装 goctl
go install github.com/zeromicro/go-zero/tools/goctl@latest

# 验证安装
goctl --version
```

## 默认使用场景

- 新建 API / RPC / Model 骨架
- 校验和格式化 `.api` 文件
- 生成 Docker / K8s 模板等辅助文件
- 定制模板或批量生成重复结构

## goctl API 命令

### 基本用法

```bash
# 从 .api 文件生成 HTTP 服务
goctl api go -api user.api -dir ./user-api -style goZero
```

### 参数说明

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `-api` | API 定义文件路径 | - |
| `-dir` | 输出目录 | `.` |
| `-style` | 命名风格：`goZero`/`go_zero`/`GoZero` | `goZero` |

### API 验证

```bash
# 验证 .api 文件语法
goctl api validate -api user.api
```

### API 格式化

```bash
# 格式化 .api 文件
goctl api format -api user.api
```

## goctl RPC 命令

### 基本用法

```bash
# 从 .proto 文件生成 gRPC 服务
goctl rpc protoc user.proto --go_out=. --go-grpc_out=. --zrpc_out=. -style goZero
```

### 参数说明

| 参数 | 说明 |
|------|------|
| `--go_out` | protobuf Go 代码输出目录 |
| `--go-grpc_out` | gRPC Go 代码输出目录 |
| `--zrpc_out` | go-zero zrpc 代码输出目录 |
| `-m` | 生成多服务模式 |
| `-style` | 命名风格 |

## goctl Model 命令

### 从 MySQL DDL 生成

```bash
# 从 SQL 文件生成 Model
goctl model mysql ddl \
    --src ./deploy/sql/user.sql \
    --dir ./internal/model \
    --cache \
    --style goZero
```

### 从数据库连接生成

```bash
# MySQL 数据源
goctl model mysql datasource \
    --url "user:password@tcp(localhost:3306)/dbname" \
    --table "user,order" \
    --dir ./internal/model \
    --cache \
    --style goZero

# PostgreSQL 数据源
goctl model pg datasource \
    --url "postgres://user:pass@localhost:5432/dbname?sslmode=disable" \
    --table "public.users" \
    --dir ./internal/model \
    --cache \
    --style goZero
```

### MongoDB 生成

```bash
# 生成 MongoDB Model
goctl model mongo \
    --type User \
    --dir ./internal/model \
    --easy
```

### 参数说明

| 参数 | 说明 |
|------|------|
| `--src` | SQL DDL 文件路径 |
| `--url` | 数据库连接字符串 |
| `--table` | 表名，逗号分隔或通配符 `*` |
| `--dir` | 输出目录 |
| `--cache` | 启用缓存层（内存 LRU + Redis） |
| `--strict` | 将 null 列映射为 Go 指针类型 |
| `--easy` | 生成简化 CRUD 方法（MongoDB） |

## goctl 其他命令

### Docker 生成（按需）

```bash
# 生成 Dockerfile
goctl docker -go user-api.go
```

### Kubernetes 生成（按需）

```bash
# 生成 K8s 部署文件
goctl kube deploy \
    --name user-api \
    --namespace default \
    --image user-api:latest \
    --o ./user-api.yaml \
    --port 8888
```

### 模板管理（按需）

```bash
# 初始化模板到本地
goctl template init

# 导出特定类别模板
goctl template init --category api
goctl template init --category rpc
goctl template init --category model
```

### Swagger 生成

```bash
# 从 .api 文件生成 Swagger 文档
goctl api swagger -api user.api -o ./doc/swagger.json
```

## 推荐实践

- 生成代码后尽快把手写逻辑放入独立文件或明确扩展点。
- 在团队内统一 `-style`、目录布局和模板来源，减少项目间风格漂移。
- 对历史项目优先渐进接入 `goctl`，不要为“纯脚手架化”大规模重构稳定模块。

## 禁止事项

- ❌ 把 `goctl` 当成质量保证本身，忽略分层、错误处理和测试
- ❌ 频繁直接改动生成代码核心段落，导致后续难以再生成
- ❌ 因为工具限制强行扭曲业务目录结构

## 审查清单

- [ ] 生成代码与手写扩展边界清晰
- [ ] 团队内脚手架风格和目录风格一致
- [ ] 历史项目接入方式是渐进的，不是破坏式重构
