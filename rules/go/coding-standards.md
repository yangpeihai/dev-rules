# Go 编码标准

> 配合 `go-rule` 使用
>
> 使用 Gin 构建 Web 服务时，额外加载 `gin-web-rule`
>
> 使用 go-zero 构建服务时，额外加载 `go-zero-rule`

## 基础规范

| 项 | 规则 |
|----|------|
| 缩进 | Tab 或 gofmt 默认格式 |
| 包名 | 全小写、简短、避免下划线 |
| 变量/函数命名 | camelCase / MixedCaps |
| 导出标识符 | PascalCase |
| 常量 | MixedCaps；仅跨语言共享常量或约定值考虑全大写 |

## 工具链

- 格式化：`gofmt` + `goimports`
- Lint：`golangci-lint`

## 具体规则

- 导出标识符必须有注释
- 错误必须立即检查；有新增上下文时使用 `%w` 包装，否则直接返回；优先使用标准库 `errors`
- 接口在使用方定义（小而专注）
- 通过 channel 共享内存
- `context` 作为第一个参数
- 避免无意义缩写和冗余包名前缀

## Web 项目补充

- Gin Web 项目：以 `gin-web-rule` 为准
- go-zero 项目：以 `go-zero-rule` 为准
- `rules` 只保留通用 Go 约束，框架细节不要重复写在这里
