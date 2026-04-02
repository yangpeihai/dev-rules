# 路由与 Handler 规范

## 路由设计

- API 路径默认使用 `/api/v1` 作为版本前缀。
- 资源路径使用复数名词，如 `/users`、`/orders`。
- 查询使用 `GET`，创建使用 `POST`，完整更新使用 `PUT`，部分更新使用 `PATCH`，删除使用 `DELETE`。
- 路由分组按业务域组织，不按“公开/私有”随意堆叠；鉴权通过中间件挂载。

## Handler 职责

- 从 `*gin.Context` 中读取路径参数、查询参数、请求体和请求上下文。
- 调用 `logic` 执行业务用例。
- 将 `logic` 返回值转换为统一响应。
- 记录必要的审计字段，不拼装复杂业务流程。

## 推荐写法

```go
func (h *UserHandler) Create(c *gin.Context) {
    var req CreateUserRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        response.ValidationError(c, err)
        return
    }

    user, err := h.userLogic.Create(c.Request.Context(), req)
    if err != nil {
        response.FromError(c, err)
        return
    }

    response.Success(c, user)
}
```

## 强制规则

- Handler 只允许出现“绑定参数 -> 调用 logic -> 输出响应”的主线。
- 统一从 `c.Request.Context()` 派生标准上下文传入下层。
- 对幂等接口显式约定幂等键或重复提交策略，不将该逻辑散落在多个 handler。
- 文件上传、下载、流式响应要单独抽象，避免污染通用 CRUD handler。

## 推荐实践

- 每个资源聚合到一个 handler 结构体中，例如 `UserHandler`、`OrderHandler`。
- 路由注册集中在 `router/`，不要在 handler 文件中隐式注册。
- 公共响应通过 `response.Success`、`response.Page`、`response.FromError` 统一输出。

## 禁止事项

- 禁止：在 Handler 中直接操作 GORM。
- 禁止：在 Handler 中开启事务。
- 禁止：在 Handler 中写权限判定细节以外的业务规则。
- 禁止：在 Handler 中手工拼接不同格式的成功/失败 JSON。
