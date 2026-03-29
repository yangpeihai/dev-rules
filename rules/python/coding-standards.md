# Python 编码标准

> 配合 `python-rule` skill 使用

## 基础规范

| 项 | 规则 |
|----|------|
| 缩进 | 4 空格 |
| 变量/函数命名 | snake_case |
| 类名 | PascalCase |
| 常量 | UPPER_SNAKE_CASE |

## 工具链

- 格式化：`black` + `ruff`
- 类型检查：`mypy`

## 具体规则

- 优先使用 `dataclass` 作为数据容器
- 使用上下文管理器管理资源 (`with` 语句)
- 禁止使用可变类型作为默认参数
- 禁止使用无意义变量名（x, data, temp）
- 公共 API 必须有类型注解
