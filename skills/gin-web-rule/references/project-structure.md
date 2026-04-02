# 项目结构规范

## 推荐目录

默认使用 `logic/` 作为业务编排层目录命名，与 go-zero 项目结构保持一致；如团队历史项目仍使用 `service/`，迁移时应保持职责一致而不是混用两套语义。

```text
cmd/server/main.go
configs/
internal/bootstrap/
internal/config/
internal/router/
internal/handler/
internal/logic/
internal/repository/
internal/model/
internal/middleware/
internal/transport/response/
internal/pkg/
pkg/
```

## 分层职责

| 目录 | 职责 | 允许依赖 |
|------|------|----------|
| `cmd/` | 程序入口、启动 HTTP 服务 | `bootstrap` |
| `bootstrap/` | 初始化配置、日志、数据库、路由 | `config`、`router`、基础设施组件 |
| `router/` | 路由分组与中间件挂载 | `handler`、`middleware` |
| `handler/` | 解析请求、调用 logic、输出响应 | `logic`、`response` |
| `logic/` | 业务用例编排 | `repository`、领域对象、外部客户端 |
| `repository/` | 数据库访问 | `model`、`gorm` |
| `model/` | 持久化实体与表映射 | 无业务逻辑 |
| `middleware/` | recovery、request id、鉴权、访问日志等横切逻辑 | `response`、基础设施组件 |

## 强制规则

- `main.go` 只负责启动，不堆积初始化细节。
- `handler` 不直接依赖 `*gorm.DB`、配置解析器、JWT 库底层实现。
- `logic` 不接收 `*gin.Context`，统一接收 `context.Context`。
- `repository` 不返回 HTTP 语义错误，只返回数据层错误或领域错误。
- 公共响应封装、错误码、日志字段常量应集中管理，避免散落在各层。

## 推荐实践

- 目录按职责拆分，不按“controllers/utils/common”这类模糊名称堆叠。
- 单个文件只承载一个主要职责，例如一个 handler 文件聚焦一个资源或一个用例。
- 对外暴露给多个服务复用的工具放在 `pkg/`，仅服务内部使用的辅助能力放在 `internal/pkg/`。
- 初始化顺序保持稳定：配置 -> 日志 -> 数据库/外部依赖 -> 中间件 -> 路由 -> Server。

## 禁止事项

- 禁止：在 `main.go` 中直接拼装所有业务依赖。
- 禁止：在 `handler` 中访问数据库。
- 禁止：将 `gin.Context` 下传到 `repository`。
- 禁止：把 DTO、Entity、VO 混用成一个结构体。
