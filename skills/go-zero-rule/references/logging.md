# 日志规范

## 日志配置

```go
// main.go
func main() {
    // 配置日志
    logx.MustSetup(logx.LogConf{
        ServiceName: c.Name,
        Mode:        c.Log.Mode,
        Path:        c.Log.Path,
        Level:       c.Log.Level,
        Compress:    c.Log.Compress,
        KeepDays:    c.Log.KeepDays,
    })
}
```

## 日志使用规范

### 推荐用法

```go
// 推荐 - 使用 WithContext 保持链路追踪
logx.WithContext(ctx).Infow("user created",
    logx.Field("userId", userId),
    logx.Field("username", username),
)

// 错误日志
logx.WithContext(ctx).Errorw("failed to create user",
    logx.Field("error", err),
    logx.Field("username", username),
)
```

### 禁止用法

```go
// 禁止 - 不使用 Context
logx.Info("user created")  // 失去链路追踪能力
```

## 日志级别使用

| 级别 | 使用场景 | 示例 |
|------|----------|------|
| **Info** | 正常业务流程 | 用户创建、订单完成 |
| **Error** | 需要关注的错误 | RPC 调用失败、DB 操作失败 |
| **Slow** | 慢查询监控 | 执行时间超过阈值的操作 |
| **Stat** | 统计信息 | 请求耗时、QPS 统计 |

## 日志脱敏

```go
// 配置日志脱敏规则
logx.WithContext(ctx).Infow("user login",
    logx.Field("username", username),
    logx.Field("phone", desensitize.Phone(phone)),  // 脱敏手机号
    logx.Field("idcard", desensitize.IdCard(idCard)), // 脱敏身份证
)
```
