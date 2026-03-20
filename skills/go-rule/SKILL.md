---
name: go-rule
description: Go 语言编码规范助手。帮助开发者编写地道、安全、高性能的 Go 代码，涵盖代码风格、安全编码、性能优化、核心设计原则等最佳实践。当用户编写 Go 代码、进行代码审查、优化性能、处理安全问题、设计接口和包结构时触发。
---

# Go 语言编码规范助手

帮助开发者编写符合 Go 语言习惯的代码，确保代码质量、安全性和性能。

## 何时使用

当你进行以下操作时，应该参考本规范：
- 编写新的 Go 代码或修改现有代码
- 设计接口、结构体和函数
- 处理并发和 goroutine
- 进行代码审查
- 优化性能或排查性能问题
- 处理安全敏感代码（密码、加密、输入验证）
- 组织项目结构和包设计
- 编写测试代码

## 快速参考

### 代码格式化

```bash
# 格式化代码
gofmt -w .
goimports -w .

# 运行 linter
golangci-lint run
```

### 常用模式

```go
// 错误处理
if err != nil {
    return fmt.Errorf("操作失败: %w", err)
}

// defer 资源清理
file, err := os.Open(path)
if err != nil {
    return err
}
defer file.Close()

// context 超时
ctx, cancel := context.WithTimeout(ctx, 10*time.Second)
defer cancel()
```

---

## 详细规范索引

本规范按主题拆分为多个文档，根据你的具体需求参考相应的章节：

| 主题 | 参考文档 | 使用场景 |
|------|---------|----------|
| 代码风格与可读性 | `style-and-readability.md` | 命名、格式化、注释、控制结构、**测试可读性** |
| 安全编码 | `security.md` | 输入验证、密码存储、加密、并发安全、**测试安全** |
| 性能与可靠性 | `performance-and-reliability.md` | 内存优化、并发性能、I/O 优化、**单元测试规范** |
| 核心原则与设计 | `core-principles-and-design.md` | 接口设计、包设计、错误处理、**可测试性设计** |

### 何时查看哪个文档

- **日常编码** → `style-and-readability.md`
- **处理敏感数据** → `security.md`
- **性能优化** → `performance-and-reliability.md`
- **架构设计** → `core-principles-and-design.md`

---

## 核心原则

### Go 的设计哲学

| 原则 | 说明 |
|------|------|
| 简单性 | 避免过度抽象，代码直接明了 |
| 可读性 | 代码即文档，优先考虑可读性 |
| 显式优于隐式 | 显式处理错误，不忽略返回值 |

### 接口设计

```go
// 接口应该小而专注
type Reader interface {
    Read(p []byte) (n int, err error)
}

// 在使用方定义接口
type UserService struct {
    repo UserRepository  // 使用者定义
}

// 接受接口，返回结构体
func ProcessData(r io.Reader) (*Result, error) { ... }
```

### 错误处理

```go
// 错误是值
if err != nil {
    return err  // 直接返回
}

// 需要上下文时包装
return fmt.Errorf("查找项目 %s: %w", id, err)

// 避免过度使用 panic（仅用于真正的不可恢复情况）
```

### 并发原则

```go
// 通过通信共享内存
type Worker struct {
    jobs    <-chan Job
    results chan<- Result
}

// 共享状态使用 mutex
type Counter struct {
    mu    sync.Mutex
    value int
}
```

---

## 禁止事项

- ❌ 不使用 `gofmt` 格式化
- ❌ 使用无意义的变量名（u, data, flag1）
- ❌ 导出标识符不写注释
- ❌ 忽略错误检查
- ❌ 使用位置依赖初始化结构体
- ❌ 过度嵌套（超过 3 层）
- ❌ SQL 使用字符串拼接（参数化查询）
- ❌ 密码不使用 bcrypt 哈希
- ❌ 记录敏感信息到日志
- ❌ 无保护的并发访问共享状态
- ❌ 外部调用不设置超时
- ❌ 硬编码密钥（从环境变量读取）

---

## 推荐实践

1. **格式化优先** - 所有代码必须通过 `gofmt` 和 `goimports`
2. **命名清晰** - 使用驼峰式、语义明确的名称
3. **接口小而专注** - 单方法接口用 `-er` 后缀
4. **错误是值** - 立即检查，使用 `%w` 包装
5. **通过通信共享内存** - 优先使用 channel 而非 mutex
6. **包名简短** - 全小写、单单词
7. **context 传递** - 作为第一个参数
8. **零值可用** - 类型设计时考虑零值语义
9. **预分配容量** - 切片和 map 预估大小
10. **设置超时** - 所有外部调用必须有超时

---

## 代码审查清单

提交代码前，请确认：

- [ ] 代码已通过 `gofmt` 和 `goimports` 格式化
- [ ] 导出标识符都有注释
- [ ] 所有错误都经过检查
- [ ] 没有忽略错误（使用 `_` 丢弃）
- [ ] 使用参数化查询防止 SQL 注入
- [ ] 密码使用 bcrypt 哈希
- [ ] 敏感信息已脱敏或从环境变量读取
- [ ] 并发访问共享状态使用 mutex
- [ ] 外部调用（HTTP、数据库）设置了超时
- [ ] 使用 defer 确保资源释放
- [ ] 包名全小写、简短
- [ ] 接口在使用方定义
- [ ] 函数职责单一，参数不超过 4 个

---

## 参考资料

官方文档和更多详细信息请查看：
- [Effective Go](https://go.dev/doc/effective_go)
- [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
- [Go Blog](https://go.dev/blog/)

完整的规范文档请查看 `references/` 目录下的各个主题文档。
