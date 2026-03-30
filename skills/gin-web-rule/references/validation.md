# 参数校验规范

## 绑定规则

- JSON 请求体使用 `ShouldBindJSON`。
- Query 参数使用 `ShouldBindQuery`。
- 表单参数使用 `ShouldBind` 或 `ShouldBindWith`，明确类型。
- Path 参数使用 `Param` 读取后显式转换与校验。

## validator 规则

- 使用结构体 tag 描述约束，例如 `required`、`min`、`max`、`oneof`、`email`。
- 自定义校验器统一在启动阶段注册，不在 handler 中临时注册。
- 校验失败统一转换为标准错误响应，必要时对字段名做友好映射。

## 推荐示例

```go
type CreateUserRequest struct {
    Name  string `json:"name" binding:"required,min=2,max=50"`
    Email string `json:"email" binding:"required,email"`
    Role  string `json:"role" binding:"omitempty,oneof=admin member guest"`
}
```

## 强制规则

- 进入业务层前必须完成格式与边界校验。
- 校验错误码必须稳定，不能依赖底层库报错文案。
- 对分页参数、排序字段、过滤条件设置上限和白名单。

## 推荐实践

- 将复杂校验拆成“基础格式校验 + 业务约束校验”，前者在 handler，后者在 service。
- 为常用校验规则建立公共 DTO 或自定义 tag，避免复制粘贴。

## 禁止事项

- 禁止：跳过校验直接进入业务逻辑。
- 禁止：在多个 Handler 重复手写相同的校验流程。
- 禁止：将 validator 的原始英文错误直接返回前端。
