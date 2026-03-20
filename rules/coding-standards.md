# 编码标准详情

> 各语言具体编码规范，配合对应 skill 使用

## 通用约束

| 项 | 规则 |
|----|------|
| 缩进 | 4 空格（Python/Go）/ 2 空格（JS/TS） |
| 变量命名 | snake_case（Python/Go）/ camelCase（JS/TS） |
| 类名 | PascalCase |
| 常量 | UPPER_SNAKE_CASE |
| 最大函数长度 | 50 行 |
| 最大嵌套深度 | 3 层 |
| 类型注解 | 必须（Python/TS/Go） |
| 文档字符串 | 必须（公共 API） |

---

## Python 规范

**工具链**：
- 格式化：`black` + `ruff`
- 类型检查：`mypy`

**规则**：
- 优先使用 `dataclass` 作为数据容器
- 使用上下文管理器管理资源 (`with` 语句)
- 禁止使用可变类型作为默认参数
- 禁止使用无意义变量名（x, data, temp）
- 公共 API 必须有类型注解

---

## Go 规范

**工具链**：
- 格式化：`gofmt` + `goimports`
- Lint：`golangci-lint`

**规则**：
- 导出标识符必须有注释
- 错误必须立即检查，使用 `%w` 包装
- 接口在使用方定义（小而专注）
- 通过 channel 共享内存
- `context` 作为第一个参数

---

## Java 规范

**工具链**：
- 遵循阿里巴巴 Java 开发手册
- Lint：SonarQube / PMD / SpotBugs

**规则**：
- 类名：UpperCamelCase
- 方法/变量：lowerCamelCase
- 使用 SLF4J 记录日志（禁止 System.out.println）
- 使用 ThreadPoolExecutor 创建线程池
- 避免 NPE（空指针异常）

---

## 架构原则

- 分层架构：API → Service → Repository
- 单一职责原则
- 依赖注入
- 组合优于继承
- 开闭原则
- 禁止循环依赖

---

## 测试要求

- 新功能必须有单元测试
- 覆盖率目标：≥80%
- 分支覆盖：≥70%
- 遵循 AAA 模式（Arrange-Act-Assert）
- 测试独立，不依赖顺序
- 使用 Mock 隔离外部依赖
