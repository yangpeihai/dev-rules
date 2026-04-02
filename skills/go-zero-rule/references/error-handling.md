# 错误处理规范

## AI 使用提示

- 当任务涉及错误码、错误映射、`errors.Is/As`、对外响应语义时，先看本文件。
- 默认优先保留错误链并在 Logic 层做业务错误映射。
- 如果代码直接返回底层 `err.Error()` 给客户端，默认视为需要修正。

## 核心原则

- 永远不要静默忽略错误。
- 需要补充上下文时使用 `%w` 包装，便于 `errors.Is` / `errors.As` 判断。
- 业务错误与底层错误分层处理，不要把底层细节直接透出到 API / RPC 响应。
- 错误码体系以稳定、可演进为目标，不要为每个小分支滥造错误码。

## 默认规则

- 对外返回稳定的业务错误码或业务错误语义。
- Logic 层负责把底层错误转换为可识别业务错误。
- 未知错误保留错误链并记录上下文日志，对外统一降级。

## 自定义错误码示例

```go
// common/errx/error.go
package errx

import "fmt"

// 通用错误码
const (
    Success      = 0
    ServerError  = 500
    InvalidParam = 400
    NotFound     = 404
    Unauthorized = 401
)

// 业务错误码（10000-99999）
const (
    UserAlreadyExists = 10001
    UserNotFound      = 10002
    InvalidPassword   = 10003
)

type Errorx struct {
    Code int64
    Msg  string
}

func NewErrCode(code int64, msg string) *Errorx {
    return &Errorx{
        Code: code,
        Msg:  msg,
    }
}

func NewErrCodeMsg(code int64, format string, args ...interface{}) *Errorx {
    return &Errorx{
        Code: code,
        Msg:  fmt.Sprintf(format, args...),
    }
}

func (e *Errorx) Error() string {
    return fmt.Sprintf("[%d] %s", e.Code, e.Msg)
}
```

## 错误码范围建议

| 范围 | 用途 | 示例 |
|------|------|------|
| 0 | 成功 | `Success = 0` |
| 1-999 | HTTP 标准状态码 | `400`, `401`, `404`, `500` |
| 1000-9999 | 系统级错误 | 数据库错误、缓存错误 |
| 10000-19999 | 用户模块错误 | `10001` 用户已存在 |
| 20000-29999 | 订单模块错误 | `20001` 订单不存在 |
| 30000-39999 | 支付模块错误 | `30001` 支付失败 |

## Logic 层错误处理示例

```go
// Logic 层错误处理
func (l *GetUserLogic) GetUser(req *types.GetUserReq) (*types.GetUserResp, error) {
    // 使用自定义错误码
    user, err := l.svcCtx.UserModel.FindOne(l.ctx, req.UserId)
    if err != nil {
        if errors.Is(err, model.ErrNotFound) {
            return nil, errorx.NewErrCode(errorx.UserNotFound)
        }
        return nil, err
    }

    return &types.GetUserResp{
        UserId:   user.Id,
        Username: user.Username,
    }, nil
}
```

## 推荐实践

- 为常见跨层错误建立稳定映射，如参数错误、资源不存在、权限不足、冲突、系统错误。
- 保持错误码文档化，避免不同模块对同一类错误给出不同语义。
- 优先用 `errors.Is` / `errors.As` 判断底层错误，而不是比较错误字符串。

## 禁止事项

- ❌ 静默吞掉错误
- ❌ 直接把底层 `err.Error()` 返回给客户端
- ❌ 用字符串匹配代替稳定错误类型或错误码
- ❌ 为每个细枝末节都定义新的错误码，导致体系失控

## 审查清单

- [ ] 业务错误与底层错误边界清晰
- [ ] 错误链可通过 `%w`、`errors.Is`、`errors.As` 追踪
- [ ] 对外错误码稳定且语义清楚
- [ ] 未知错误已记录上下文并统一降级
