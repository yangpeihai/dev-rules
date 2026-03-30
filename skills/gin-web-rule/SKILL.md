---
name: gin-web-rule
description: "Gin Web 项目规范指南。Cover company-style Gin project structure, routing and handler rules, request/response contracts, validation, middleware, unified error handling, GORM usage, configuration bootstrap, JWT security, logging, testing, and code review practices. Invoke when: creating or modifying Gin Web services, designing project scaffolds, writing handler/service/repository code, defining unified API responses and error codes, configuring middleware chains, integrating GORM/Zap/Viper/JWT, writing tests, conducting code reviews, or writing technical documentation for Gin Web projects."
---

# Gin Web 项目规范助手

帮助开发者在使用 Gin 构建 Web 服务时遵循统一的团队工程规范，确保项目结构清晰、接口风格一致、错误处理统一、日志可观测、配置可治理、数据库访问可维护。

## 快速参考

### 默认技术栈

- Web 框架：`gin`
- ORM：`gorm`
- 参数校验：`go-playground/validator`
- 日志：`zap`
- 配置：`viper`
- 鉴权：`jwt`

### 何时查看哪个文档

- **创建新项目** -> `project-structure.md` + `config-and-bootstrap.md`
- **编写路由和 Handler** -> `routing-and-handler.md` + `request-and-response.md`
- **处理参数校验** -> `validation.md`
- **编写中间件** -> `middleware.md` + `auth-and-security.md`
- **统一错误输出** -> `error-handling.md`
- **数据库访问** -> `database-with-gorm.md`
- **日志、测试、代码审查** -> `logging-testing-and-review.md`

## 核心原则

1. `Handler` 只负责请求解析、参数校验、调用 Service、输出响应。
2. `Service` 负责业务用例编排，不直接依赖 Gin 细节。
3. `Repository` 只负责数据访问，不承载业务规则。
4. DTO、数据库实体、响应对象必须分离。
5. 错误码、错误响应、成功响应必须统一。
6. 配置、日志、数据库、鉴权组件必须在启动阶段集中初始化。
7. 所有外部调用与数据库访问必须透传 `context.Context`。
8. 默认遵循 RESTful 风格与结构化日志规范。

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

| 主题 | 参考文档 | 使用场景 |
|------|---------|----------|
| 项目结构 | `project-structure.md` | 初始化目录和分层时 |
| 路由与 Handler | `routing-and-handler.md` | 编写路由、分组、Handler 时 |
| 请求与响应 | `request-and-response.md` | 定义 DTO、VO、统一返回体时 |
| 参数校验 | `validation.md` | 使用绑定和 validator 时 |
| 中间件 | `middleware.md` | 设计恢复、日志、鉴权链路时 |
| 错误处理 | `error-handling.md` | 统一错误码和响应时 |
| GORM 数据库 | `database-with-gorm.md` | 编写 Model、Repository、事务时 |
| 配置与启动 | `config-and-bootstrap.md` | 初始化配置和依赖时 |
| 认证与安全 | `auth-and-security.md` | JWT、密码、敏感信息处理时 |
| 日志、测试、审查 | `logging-testing-and-review.md` | 测试、日志、提交流程时 |

## 参考资料

- Gin 官方仓库：<https://github.com/gin-gonic/gin>
- Gin 官方文档：<https://gin-gonic.com/en/docs/>
- GORM 官方文档：<https://gorm.io/>

完整规范请查看 `references/` 目录下的专题文档。
