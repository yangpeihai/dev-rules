---
name: go-zero-rule
description: "Go-Zero 编码规范指南。Cover project structure, API/Proto definitions, code generation with goctl, layer responsibilities, error handling, config management, logging, database operations, and middleware best practices. Invoke when: creating/modifying go-zero API/RPC services, writing Handler/Logic/Model code, using goctl tool, defining API/Proto files, configuring services, writing unit tests, conducting code reviews, or writing technical documentation for go-zero projects."
---

# Go-Zero 编码规范助手

帮助开发者在使用 go-zero 框架开发时遵循编码规范和最佳实践。

## AI 触发条件

当任务满足以下任一条件时，应优先使用本规范：
- 创建新的 go-zero API/RPC 服务
- 编写 go-zero 项目的 Handler、Logic、Model 层代码
- 使用 goctl 工具生成或维护代码骨架
- 定义 API (.api) 或 Proto (.proto) 文件
- 配置 go-zero 服务的配置文件
- 编写 go-zero 项目的单元测试
- 进行 go-zero 项目的代码审查

## AI 执行优先级

按以下顺序决策，避免自由发挥：

1. 先判断任务类型：新建、修改、调试、审查、文档。
2. 再判断影响层次：Handler/Server、Logic、Model、ServiceContext、API/Proto。
3. 涉及事务、缓存、下游调用时，优先参考 `database.md` 和 `layer-responsibility.md`。
4. 涉及错误码、错误映射、对外返回时，优先参考 `error-handling.md`。
5. 涉及生成代码时，优先保持生成代码与手写扩展边界，不急于直接修改脚手架产物。
6. 若项目已有稳定历史结构，优先兼容现状，再考虑向默认结构渐进靠拢。

## AI 默认执行步骤

1. 识别任务是“新增功能”“修改现有逻辑”“代码审查”还是“接口/脚手架调整”。
2. 只读取本次任务直接相关的 references，不要无差别全读。
3. 默认遵循 `handler/server -> logic -> model` 分层，不把业务逻辑放到协议层或数据层。
4. 需要事务时默认放在 Logic 层组织；Model 负责数据访问。
5. 需要生成或维护骨架时优先使用 `goctl`，但手写逻辑放在清晰扩展点。
6. 输出前按 `code-review-checklist.md` 做一次最小自检。

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

- 结构化规范文档：更适合做日常开发、审查和落地决策，通常包含“核心原则 / 默认规则 / 推荐实践 / 禁止事项 / 审查清单”。
- 参考手册型文档：更适合查语法、看示例、核对命名和生成命令。

| 主题 | 参考文档 | 使用场景 |
|------|---------|----------|
| 命名规范 | `naming.md` | 定义包名、文件名、变量名时 |
| goctl 工具 | `goctl.md` | 使用 goctl 生成或维护代码骨架时 |
| API 定义 | `api-definition.md` | 编写 `.api` 文件、核对注解和路由约定时 |
| Proto 定义 | `proto-definition.md` | 编写 `.proto` 文件、规划包路径和兼容性时 |
| 项目结构 | `project-structure.md` | 创建新项目或整理目录结构时 |
| 分层职责 | `layer-responsibility.md` | 编写 Handler/Logic/Model 代码时 |
| 错误处理 | `error-handling.md` | 设计错误码和错误映射时 |
| 配置管理 | `config-management.md` | 编写配置文件时 |
| 日志规范 | `logging.md` | 记录日志时 |
| 数据库操作 | `database.md` | 设计事务、缓存和 Model 使用方式时 |
| 中间件 | `middleware.md` | 编写中间件时 |
| 测试 | `testing.md` | 编写单元测试和评估覆盖重点时 |
| 代码审查 | `code-review-checklist.md` | 提交前自查或代码审查时 |
| 推荐实践 | `best-practices.md` | 快速把握团队常见落地方式时 |

### 何时查看哪个文档

- **新建服务** → `project-structure.md` + `goctl.md`
- **定义 API** → `api-definition.md` + `naming.md`
- **定义 Proto** → `proto-definition.md` + `naming.md`
- **编写业务逻辑** → `layer-responsibility.md` + `error-handling.md`
- **设计事务或缓存** → `database.md` + `layer-responsibility.md`
- **整理测试策略** → `testing.md` + `code-review-checklist.md`
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
3. ServiceContext 是默认的依赖聚合入口 - 所有共享依赖在启动时初始化
4. 配置优于硬编码 - 超时、开关、下游地址等放入配置文件
5. Model 层通常由 goctl 生成 - 生成代码与手写扩展应分离管理
6. 优先使用 goctl 生成代码骨架 - 保持代码结构标准化，但允许在历史项目中渐进接入
7. 保持服务韧性 - 利用 go-zero 内置的熔断、限流能力
8. 可观测性第一 - 配置好日志、指标和链路追踪

### 文档使用建议

- 需要做工程决策时，优先看结构化规范文档，如 `layer-responsibility.md`、`database.md`、`testing.md`、`code-review-checklist.md`。
- 需要查语法、命令、注解和生成方式时，优先看参考手册型文档，如 `goctl.md`、`api-definition.md`、`proto-definition.md`。

## AI 默认输出要求

- 实现任务：优先直接落代码或文档修改，再简要说明理由。
- 审查任务：先报问题，再给总结；优先分层越界、错误处理、事务边界、兼容性风险。
- 文档任务：用短句和清晰列表，避免重复背景说明。
- 涉及历史项目：明确说明“保持兼容”还是“建议渐进迁移”。

## AI 例外条件

- 历史项目未使用标准目录时，不强制一次性迁移到标准布局。
- 项目已有统一事务封装但不是 `TransactCtx` 时，遵循项目约定，不强行替换。
- 项目未使用 `goctl` 或使用定制模板时，只要分层清晰，可按现状增量维护。
- 用户明确要求偏离默认结构时，遵从用户要求，但应指出后果或权衡。

## AI 最小工作流

### 实现任务

1. 先判断任务落在 Handler/Server、Logic、Model、ServiceContext、API/Proto 哪一层。
2. 读取最相关的 1-2 份 reference；涉及事务或缓存时联动 `database.md`。
3. 默认保持 `handler/server -> logic -> model` 分层和生成代码边界。
4. 自检错误码、事务封装、日志上下文和生成代码改动范围。
5. 输出修改结果，并简短说明是否保持兼容或建议渐进迁移。

### 审查任务

1. 先看分层越界、错误处理、事务边界和生成代码误改。
2. 再看测试覆盖重点、日志上下文和配置硬编码。
3. 按严重程度列出问题，优先真实行为风险。
4. 最后给简短总结和残余风险。

### 文档任务

1. 先判断文档是结构化规范还是参考手册。
2. 优先补充触发条件、默认步骤和例外条件。
3. 避免把 goctl、目录结构或事务写法写成唯一标准。
4. 保持与现有 references 的术语和分层一致。

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
- [ ] 关键业务路径、错误分支和回归风险点已覆盖
- [ ] 事务操作使用项目当前统一的事务封装（如 `TransactCtx`）
- [ ] API/Proto 定义文件与实际代码同步
- [ ] goctl 生成代码与手写业务代码边界清晰

---

## 参考资料

官方文档和更多详细信息请查看：
- [go-zero 官方文档](https://go-zero.dev/zh-cn/)
- [go-zero 项目结构](https://go-zero.dev/zh-cn/concepts/project-structure/)
- [goctl 工具文档](https://go-zero.dev/zh-cn/reference/goctl/)

完整的规范文档请查看 `references/` 目录下的各个主题文档。
