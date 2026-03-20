# 错误处理规范

## 错误处理原则

1. **永远不要静默忽略错误**
2. **使用上下文包装错误**：`fmt.Errorf("operation: %w", err)`
3. **在包级别定义哨兵错误用于类型断言**

## 定义自定义错误码

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

## 错误码范围规范

| 范围 | 用途 | 示例 |
|------|------|------|
| 0 | 成功 | `Success = 0` |
| 1-999 | HTTP 标准状态码 | `400`, `401`, `404`, `500` |
| 1000-9999 | 系统级错误 | 数据库错误、缓存错误 |
| 10000-19999 | 用户模块错误 | `10001` 用户已存在 |
| 20000-29999 | 订单模块错误 | `20001` 订单不存在 |
| 30000-39999 | 支付模块错误 | `30001` 支付失败 |

## 错误处理示例

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
