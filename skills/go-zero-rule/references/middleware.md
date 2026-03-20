# 中间件规范

## 中间件定义

```go
// internal/middleware/auth.go
package middleware

import "net/http"

type AuthMiddleware struct {
    secret string
}

func NewAuthMiddleware(secret string) *AuthMiddleware {
    return &AuthMiddleware{secret: secret}
}

func (m *AuthMiddleware) Handle(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        // 1. 获取 Token
        token := r.Header.Get("Authorization")
        if token == "" {
            httpx.ErrorCtx(r.Context(), w, errorx.Unauthorized)
            return
        }

        // 2. 验证 Token
        claims, err := m.verifyToken(token)
        if err != nil {
            httpx.ErrorCtx(r.Context(), w, errorx.Unauthorized)
            return
        }

        // 3. 将用户信息存入 Context
        ctx := context.WithValue(r.Context(), "userId", claims.UserId)
        r = r.WithContext(ctx)

        // 4. 继续处理请求
        next(w, r)
    }
}
```

## 在 API 中使用中间件

```go
@server(
    group: user
    prefix: /user
    middleware: AuthMiddleware, LogMiddleware  // 多个中间件用逗号分隔
)
service user-api {
    @handler getUserInfo
    get /info (GetUserInfoReq) returns (GetUserInfoResp)
}
```
