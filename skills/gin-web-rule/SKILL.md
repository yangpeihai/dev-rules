---
name: gin-web-rule
description: "Gin Web 项目规范指南。Cover company-style Gin project structure, routing and handler rules, request/response contracts, validation, middleware, unified error handling, GORM usage, configuration bootstrap, JWT security, logging, testing, and code review practices. Invoke when: creating or modifying Gin Web services, designing project scaffolds, writing handler/logic/repository code, defining unified API responses and error codes, configuring middleware chains, integrating GORM/Zap/Viper/JWT, writing tests, conducting code reviews, or writing technical documentation for Gin Web projects."
---

# Gin Web 项目规范助手

帮助开发者在使用 Gin 构建 Web 服务时遵循统一的团队工程规范，确保项目结构清晰、接口风格一致、错误处理统一、日志可观测、配置可治理、数据库访问可维护。

## AI 触发条件

当任务满足以下任一条件时，应优先使用本规范：
- 创建或修改 Gin Web 服务
- 新增/修改 Handler、Logic、Repository
- 设计统一返回体、错误码、参数校验、中间件链
- 编写 GORM 相关数据访问或事务逻辑
- 进行 Gin Web 代码审查或文档整理

## AI 执行优先级

按以下顺序决策，避免只看单点代码：

1. 先判断任务类型：新增接口、修改接口、改事务/查询、改中间件、代码审查。
2. 再判断影响层次：Handler、Logic、Repository、Middleware、Bootstrap。
3. 涉及返回体、错误码、兼容性时，优先参考 `request-and-response.md` 与 `error-handling.md`。
4. 涉及事务、分页、大表查询、排序时，优先参考 `database-with-gorm.md`。
5. 涉及目录命名和分层争议时，优先参考 `project-structure.md`。
6. 若项目已有稳定历史结构，优先兼容现状，再考虑渐进迁移。

## AI 默认执行步骤

1. 识别任务属于“协议层”“业务编排层”“数据访问层”还是“横切能力”。
2. 只读取本次任务直接相关的 references，不要整套全读。
3. 默认遵循 `handler -> logic -> repository` 分层，不把业务逻辑下沉到 Handler/Repository。
4. 需要事务时默认由 Logic 发起；Repository 接收事务句柄或 DB 抽象。
5. 输出前自检返回体一致性、错误处理、参数校验、事务边界和上下文透传。

## 快速参考

### 常见技术栈

- Web 框架：`gin`
- ORM：`gorm`
- 参数校验：`go-playground/validator`
- 日志：`zap`
- 配置：`viper`
- 鉴权：`jwt`

## 核心原则

1. `Handler` 只负责请求解析、参数校验、调用 Logic、输出响应。
2. `Logic` 负责业务用例编排，不直接依赖 Gin 细节；默认命名对齐 go-zero 的 `logic` 层。
3. `Repository` 只负责数据访问，不承载业务规则。
4. DTO、数据库实体、响应对象默认分离；简单内部接口如合并需明确边界与原因。
5. 错误码、错误响应、成功响应应保持服务内一致。
6. 配置、日志、数据库、鉴权组件应在启动阶段集中初始化，避免运行期随意拼装依赖。
7. 所有外部调用与数据库访问应透传 `context.Context`。
8. 默认遵循 RESTful 风格与结构化日志规范；如受历史接口约束，可兼容演进。

## 禁止事项

- 禁止：在 Handler 中编写业务逻辑。
- 禁止：在 Repository 中编写业务规则。
- 禁止：直接返回 GORM Model 给前端。
- 禁止：硬编码配置、密钥、JWT Secret。
- 禁止：每个接口自定义错误 JSON 结构。
- 禁止：忽略参数校验失败。
- 禁止：在事务中执行长耗时外部请求。
- 禁止：无分页或无条件扫描大表。

## 详细规范索引

本规范按主题拆分为多个文档，根据你的具体需求参考相应的章节：

- 结构化规范文档：更适合做日常开发、分层设计、代码审查和工程决策。
- 参考约定型文档：更适合查接口结构、示例和具体落地方式。

| 主题 | 参考文档 | 使用场景 |
|------|---------|----------|
| 项目结构 | `project-structure.md` | 初始化目录、整理分层和迁移命名时 |
| 路由与 Handler | `routing-and-handler.md` | 编写路由、分组、Handler 时 |
| 请求与响应 | `request-and-response.md` | 定义 DTO、VO、统一返回体时 |
| 参数校验 | `validation.md` | 使用绑定和 validator 时 |
| 中间件 | `middleware.md` | 设计恢复、日志、鉴权链路时 |
| 错误处理 | `error-handling.md` | 统一错误码、错误映射和响应时 |
| GORM 数据库 | `database-with-gorm.md` | 设计 Repository、事务、分页和查询边界时 |
| 配置与启动 | `config-and-bootstrap.md` | 初始化配置和依赖时 |
| 认证与安全 | `auth-and-security.md` | JWT、密码、权限与敏感信息处理时 |
| 日志、测试、审查 | `logging-testing-and-review.md` | 测试、日志、提交流程和 review 时 |

### 何时查看哪个文档

- **创建新项目** → `project-structure.md` + `config-and-bootstrap.md`
- **编写路由和 Handler** → `routing-and-handler.md` + `request-and-response.md`
- **设计返回体和错误码** → `request-and-response.md` + `error-handling.md`
- **处理参数校验** → `validation.md`
- **设计事务或查询边界** → `database-with-gorm.md` + `project-structure.md`
- **编写中间件** → `middleware.md` + `auth-and-security.md`
- **日志、测试、代码审查** → `logging-testing-and-review.md`

### 文档使用建议

- 需要做工程决策时，优先看结构化规范文档，如 `project-structure.md`、`database-with-gorm.md`、`error-handling.md`、`logging-testing-and-review.md`。
- 需要核对接口对象、返回体和 handler 写法时，优先看参考约定型文档，如 `request-and-response.md`、`routing-and-handler.md`、`validation.md`。

## AI 默认输出要求

- 实现任务：优先直接给出修改，再简短说明为什么这样分层。
- 审查任务：先报问题，再给总结；优先看分层越界、错误响应、事务边界、兼容性风险。
- 历史项目：明确说明“保持兼容”还是“建议渐进迁移”。

## AI 例外条件

- 历史项目仍使用 `service/` 时，不强制一次性改为 `logic/`，但新代码不应混用两套语义。
- 纯内部接口可在评估调用方成本后简化返回体，但同一模块不应长期并存多套风格。
- 用户明确要求偏离 RESTful 或统一返回体时，应指出影响后再执行。

## AI 最小工作流

### 实现任务

1. 先判断任务落在 Handler、Logic、Repository 还是 Middleware。
2. 读取最相关的 1-2 份 reference，默认联动 `project-structure.md` 校验分层。
3. 按 `handler -> logic -> repository` 落实现，避免越层。
4. 自检返回体一致性、参数校验、错误处理、事务边界和 `context` 透传。
5. 输出修改结果，并简短说明分层与兼容性取舍。

### 审查任务

1. 先看分层越界、错误响应、事务边界和大表查询风险。
2. 再看中间件职责、配置初始化和兼容性问题。
3. 按严重程度列问题，优先行为风险和架构边界。
4. 最后给简短总结和残余风险。

### 文档任务

1. 先判断文档是工程决策型还是接口约定型。
2. 优先写“默认怎么做”和“何时例外”。
3. 减少理想化硬约束，保留历史项目兼容空间。
4. 保持命名、分层和示例与当前规范一致。

## 参考资料

- Gin 官方仓库：<https://github.com/gin-gonic/gin>
- Gin 官方文档：<https://gin-gonic.com/en/docs/>
- GORM 官方文档：<https://gorm.io/>

完整规范请查看 `references/` 目录下的专题文档。
