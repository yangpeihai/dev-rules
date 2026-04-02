# 分层职责规范

## AI 使用提示

- 当任务涉及“这段逻辑该放哪一层”时，先看本文件。
- 如果发现 Handler / Server 直连数据库、RPC 或事务，默认视为高优先级问题。
- 若项目是历史结构，优先保持职责边界正确，再决定是否迁移目录命名。

## 核心原则

- Handler / Server 负责协议层适配，不承载业务编排。
- Logic 负责业务用例编排，是默认的事务、缓存、下游调用组织位置。
- Model 负责数据访问，不承载业务规则。
- `ServiceContext` 负责共享依赖初始化与聚合，不承载业务逻辑。

## 各层职责定义

| 层 | 包 | 职责 | 是否包含业务逻辑 |
|---|---|---|:---:|
| **Handler** | `internal/handler` | 解析并校验 HTTP 请求，调用 logic，写响应 | 否 |
| **Server** | `internal/server` | gRPC 服务端，解析请求，调用 logic | 否 |
| **Logic** | `internal/logic` | 实现业务用例，编排 DB/缓存/RPC 调用 | **是** |
| **ServiceContext** | `internal/svc` | 启动时初始化并持有共享依赖 | 否 |
| **Config** | `internal/config` | 将 YAML 字段映射为强类型 Go 结构体 | 否 |
| **Model** | `model`/`internal/model` | 数据访问层 | 否 |

## 默认规则

- Handler 保持“解析请求 -> 调用 logic -> 返回响应”的主线。
- 复杂业务校验、事务边界、下游编排放在 Logic 层。
- 共享依赖通过 `ServiceContext` 注入，不在业务代码里临时创建。
- Model 扩展应与 goctl 生成代码分层管理，避免再生成时相互覆盖。

## Handler 层示例

### 推荐写法

```go
// 推荐 - 精简的 Handler
func (h *CreateUserHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    var req types.CreateUserReq
    if err := httpx.Parse(r, &req); err != nil {
        httpx.ErrorCtx(r.Context(), w, err)
        return
    }

    l := logic.NewCreateUserLogic(r.Context(), h.svcCtx)
    resp, err := l.CreateUser(&req)
    if err != nil {
        httpx.ErrorCtx(r.Context(), w, err)
        return
    }

    httpx.OkJsonCtx(r.Context(), w, resp)
}
```

## Logic 层示例

### 推荐写法

```go
// 推荐 - 业务逻辑在 Logic 层
func (l *CreateUserLogic) CreateUser(req *types.CreateUserReq) (*types.CreateUserResp, error) {
    // 1. 参数校验
    if len(req.Username) < 3 {
        return nil, errorx.NewErrCodeMsg(errorx.InvalidParam, "用户名至少3个字符")
    }

    // 2. 检查用户是否存在
    exist, err := l.svcCtx.UserModel.FindByUsername(l.ctx, req.Username)
    if err != nil && err != model.ErrNotFound {
        return nil, err
    }
    if exist != nil {
        return nil, errorx.NewErrCode(errorx.UserAlreadyExists)
    }

    // 3. 创建用户
    userId, err := l.svcCtx.UserModel.Insert(l.ctx, &model.User{
        Username: req.Username,
        Password: crypto.HashPassword(req.Password),
    })
    if err != nil {
        return nil, err
    }

    return &types.CreateUserResp{UserId: userId}, nil
}
```

## Model 层说明

### 生成与扩展

Model 层默认使用 goctl 工具生成；如果历史项目或定制场景需要手写扩展，应与生成代码分层管理。**禁止**：
- ❌ 在 Model 层编写业务逻辑
- ❌ 在 Model 层调用 RPC 或其他服务
- ❌ 在 Model 层记录业务日志

## 推荐实践

- 一个 logic 文件聚焦一个用例，避免“大 logic”承载多个业务流程。
- 通过接口或扩展文件隔离手写 Model 能力，减少对生成代码的侵入。
- 在 Handler 和 Logic 之间保持 DTO/VO 边界，避免把底层模型直接透传到上层。

## 禁止事项

- ❌ 在 Handler 中编写业务逻辑
- ❌ 在 Handler/Server 中直接调用数据库或 RPC
- ❌ 在 Model 层编排业务流程
- ❌ 复杂参数校验全部堆到 Handler

## 审查清单

- [ ] Handler / Server 仅承担协议层职责
- [ ] 事务、缓存、下游编排位于 Logic 层
- [ ] Model 仅承担数据访问职责
- [ ] `ServiceContext` 只负责依赖聚合，不含业务逻辑
