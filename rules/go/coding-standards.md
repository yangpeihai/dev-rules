# Go 编码标准

> 配合 `go-rule` 和 `go-zero-rule` skill 使用

## 基础规范

| 项 | 规则 |
|----|------|
| 缩进 | 4 空格 |
| 变量/函数命名 | snake_case |
| 类名 | PascalCase |
| 常量 | UPPER_SNAKE_CASE |

## 工具链

- 格式化：`gofmt` + `goimports`
- Lint：`golangci-lint`

## 具体规则

- 导出标识符必须有注释
- 错误必须立即检查，使用 `%w` 包装
- 接口在使用方定义（小而专注）
- 通过 channel 共享内存
- `context` 作为第一个参数
