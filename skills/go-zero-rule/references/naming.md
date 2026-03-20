# 命名规范

## 包命名

- **简短、小写、无下划线**: `logx`、`httpx`、`core`
- **禁止**: 使用复数形式、大写字母、无意义的缩写

```go
// 推荐
package user
package orderhandler

// 禁止
package users
package UserHandler
```

## 接口命名

- **描述性名词或 `-er` 后缀**: `UserModel`、`Breaker`、`CacheStore`
- **单一职责原则**: 接口应该专注且简洁

## 构造函数命名

- **`NewXxx`**: 普通构造函数，返回 error
- **`MustNewXxx`**: 必须成功的构造函数，失败则 panic

## 错误变量命名

- **`ErrXxx` 格式**: `ErrNotFound`、`ErrTimeout`、`ErrInvalidParam`

```go
// 在包级别定义哨兵错误
var (
    ErrUserNotFound = errors.New("user not found")
    ErrInvalidToken = errors.New("invalid token")
    ErrTimeout      = errors.New("request timeout")
)
```

## 文件命名

| 类型 | 命名规则 | 示例 |
|------|---------|------|
| Handler | `{operation}handler.go` | `loginhandler.go` |
| Logic | `{operation}logic.go` | `loginlogic.go` |
| Server | `{service}server.go` | `userserver.go` |
| Repository | `{entity}.go` | `user.go` |
| Middleware | `{name}middleware.go` | `authmiddleware.go` |

## 变量命名

| 类型 | 约定 | 示例 |
|------|------|------|
| 常量 | 大写+下划线 | `MAX_RETRY_COUNT` |
| 局部变量 | 驼峰命名 | `userId`, `userName` |
| 结构体字段 | 驼峰命名，导出首字母大写 | `UserId`, `CreateTime` |
| 接口方法 | 驼峰命名 | `GetUser`, `CreateOrder` |
