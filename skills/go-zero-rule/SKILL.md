---
name: go-zero-rule
description: Go-Zero 编码规范助手。帮助开发者在使用 go-zero 框架开发时遵循编码规范，包括项目结构、API/Proto 定义、代码生成、分层职责、错误处理、配置管理、日志规范、数据库操作、中间件、测试等最佳实践。当用户创建或修改 go-zero 项目、编写 Handler/Logic/Model 代码、使用 goctl 工具、定义 API/Proto 文件、进行代码审查时触发。
---

# Go-Zero 编码规范助手

帮助开发者在使用 go-zero 框架开发时遵循编码规范和最佳实践。

## 何时使用

当你进行以下操作时，应该参考本规范：
- 创建新的 go-zero API/RPC 服务
- 编写 go-zero 项目的 Handler、Logic、Model 层代码
- 使用 goctl 工具生成代码
- 定义 API (.api) 或 Proto (.proto) 文件
- 配置 go-zero 服务的配置文件
- 编写 go-zero 项目的单元测试
- 进行 go-zero 项目的代码审查

## 快速参考

### 常用命令

```bash
# 安装 goctl
go install github.com/zeromicro/go-zero/tools/goctl@latest

# 创建 API 服务
goctl api go -api user.api -dir ./user-api -style goZero

# 创建 RPC 服务
goctl rpc protoc user.proto --go_out=. --go-grpc_out=. --zrpc_out=. -style goZero

# 生成 Model（带缓存）
goctl model mysql datasource \
    --url "user:password@tcp(127.0.0.1:3306)/dbname" \
    --table "user" \
    --dir ./model \
    --cache \
    --style goZero

# 验证 API 文件
goctl api validate -api user.api

# 格式化 API 文件
goctl api format -api user.api
```

---

## 详细规范索引

本规范按主题拆分为多个文档，根据你的具体需求参考相应的章节：

| 主题 | 参考文档 | 使用场景 |
|------|---------|----------|
| 命名规范 | `naming.md` | 定义包名、文件名、变量名时 |
| goctl 工具 | `goctl.md` | 使用 goctl 生成代码时 |
| API 定义 | `api-definition.md` | 编写 .api 文件时 |
| Proto 定义 | `proto-definition.md` | 编写 .proto 文件时 |
| 项目结构 | `project-structure.md` | 创建新项目或组织目录时 |
| 分层职责 | `layer-responsibility.md` | 编写 Handler/Logic/Model 代码时 |
| 错误处理 | `error-handling.md` | 定义错误码或处理错误时 |
| 配置管理 | `config-management.md` | 编写配置文件时 |
| 日志规范 | `logging.md` | 记录日志时 |
| 数据库操作 | `database.md` | 使用 Model 或事务时 |
| 中间件 | `middleware.md` | 编写中间件时 |
| 测试 | `testing.md` | 编写单元测试时 |
| 代码审查 | `code-review-checklist.md` | 提交代码前检查 |
| 推荐实践 | `best-practices.md` | 查看最佳实践和常用命令 |

### 何时查看哪个文档

- **新建服务** → `project-structure.md` + `goctl.md`
- **定义 API** → `api-definition.md` + `naming.md`
- **定义 Proto** → `proto-definition.md` + `naming.md`
- **编写业务逻辑** → `layer-responsibility.md` + `error-handling.md`
- **记录日志** → `logging.md`
- **数据库操作** → `database.md`
- **编写测试** → `testing.md`
- **代码审查** → `code-review-checklist.md`

---

## 核心原则

### 分层职责

| 层 | 包 | 职责 | 业务逻辑 |
|---|---|---|:---:|
| Handler | `internal/handler` | 解析 HTTP 请求，调用 logic，写响应 | ❌ |
| Server | `internal/server` | gRPC 服务端，解析请求，调用 logic | ❌ |
| Logic | `internal/logic` | 实现业务用例，编排 DB/缓存/RPC 调用 | ✅ |
| Model | `model`/`internal/model` | 数据访问层 | ❌ |

### 禁止事项

- ❌ 在 Handler 中编写业务逻辑
- ❌ 在 Model 层编写业务逻辑或调用 RPC
- ❌ 直接调用数据库或 RPC（应通过 ServiceContext）
- ❌ 硬编码配置（应放入 `etc/*.yaml`）
- ❌ 使用日志不带 Context（应使用 `logx.WithContext(ctx)`）

### 推荐实践

1. Handler 保持精简 - 只做请求解码、调用 logic、编码响应
2. 每个用例一个 logic 文件 - 如 `CreateOrderLogic`、`GetOrderLogic`
3. ServiceContext 是唯一的构造入口 - 所有依赖在启动时初始化
4. 配置优于硬编码 - 超时、开关、下游地址等放入配置文件
5. Model 层自动生成 - 可以随时重新生成，logic 层不会被覆盖
6. 使用 goctl 生成代码 - 保持代码结构标准化
7. 保持服务韧性 - 利用 go-zero 内置的熔断、限流能力
8. 可观测性第一 - 配置好日志、指标和链路追踪

---

## 代码审查清单

提交代码前，请确认：

- [ ] 代码已通过 `gofmt` 和 `goimports` 格式化
- [ ] Handler 层仅做请求/响应处理，无业务逻辑
- [ ] 业务逻辑在 Logic 层实现
- [ ] 错误使用了自定义错误码
- [ ] 所有日志使用 `WithContext` 保持链路追踪
- [ ] 敏感信息已脱敏处理
- [ ] 配置参数已放入 `etc/*.yaml`，无硬编码
- [ ] 单元测试覆盖率不低于 80%
- [ ] 事务操作使用 `TransactCtx` 方法
- [ ] API/Proto 定义文件与实际代码同步
- [ ] 使用 goctl 生成代码，而非手写

---

## 参考资料

官方文档和更多详细信息请查看：
- [go-zero 官方文档](https://go-zero.dev/zh-cn/)
- [go-zero 项目结构](https://go-zero.dev/zh-cn/concepts/project-structure/)
- [goctl 工具文档](https://go-zero.dev/zh-cn/reference/goctl/)

完整的规范文档请查看 `references/` 目录下的各个主题文档。
