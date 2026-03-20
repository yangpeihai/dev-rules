# 分层职责规范

## 各层职责定义

| 层 | 包 | 职责 | 是否包含业务逻辑 |
|---|---|---|:---:|
| **Handler** | `internal/handler` | 解析并校验 HTTP 请求，调用 logic，写响应 | 否 |
| **Server** | `internal/server` | gRPC 服务端，解析请求，调用 logic | 否 |
| **Logic** | `internal/logic` | 实现业务用例，编排 DB/缓存/RPC 调用 | **是** |
| **ServiceContext** | `internal/svc` | 启动时初始化并持有共享依赖 | 否 |
| **Config** | `internal/config` | 将 YAML 字段映射为强类型 Go 结构体 | 否 |
| **Model** | `model`/`internal/model` | 数据访问层 | 否 |

## Handler 层规范

### 推荐示例

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

### 禁止事项

- ❌ 在 Handler 中编写业务逻辑
- ❌ 直接调用数据库或 RPC
- ❌ 复杂的参数校验（应在 Logic 层）

## Logic 层规范

### 推荐示例

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

## Model 层规范

### 使用 goctl 生成

Model 层应使用 goctl 工具自动生成，**禁止**：
- ❌ 在 Model 层编写业务逻辑
- ❌ 在 Model 层调用 RPC 或其他服务
- ❌ 在 Model 层记录业务日志
