---
name: go-rule
description: "Go 语言编码规范指南。Cover Go code style (Effective Go), concurrency patterns, security, performance optimization, and package design best practices. Invoke when: writing/reviewing Go code, designing interfaces or packages, configuring Go modules (go.mod), handling concurrency (goroutines/channels), debugging issues, refactoring legacy code, writing technical documentation, or discussing Go architecture decisions."
---

# Go 语言编码规范助手

帮助开发者编写符合 Go 语言习惯的代码，确保代码质量、安全性和性能。

## AI 触发条件

当任务满足以下任一条件时，应优先使用本规范：
- 编写新的 Go 代码或修改现有代码
- 设计接口、结构体和函数
- 处理并发和 goroutine
- 进行代码审查
- 优化性能或排查性能问题
- 处理安全敏感代码（密码、加密、输入验证）
- 组织项目结构和包设计
- 编写测试代码

## AI 执行优先级

按以下顺序决策，避免只凭印象写 Go：

1. 先判断任务类型：实现、重构、调试、优化、审查、文档。
2. 再判断核心问题属于：代码风格、设计边界、错误处理、安全、性能、测试。
3. 涉及接口、包边界、context、错误链、依赖注入时，优先参考 `core-principles-and-design.md`。
4. 涉及命名、注释、函数签名、代码组织时，优先参考 `style-and-readability.md`。
5. 涉及超时、并发、基准、热点优化、测试策略时，优先参考 `performance-and-reliability.md`。
6. 涉及输入验证、密码、路径、密钥、TLS、敏感信息时，优先参考 `security.md`。

## AI 默认执行步骤

1. 识别本次任务的主目标是“可读性”“正确性”“安全性”“可靠性”还是“性能”。
2. 只读取当前问题直接相关的 references，不要整套文档全读。
3. 默认优先选择最简单、最惯用、最容易维护的 Go 写法。
4. 错误处理默认保留错误链；新增上下文时包装，仅透传时直接返回。
5. 并发默认先保证正确性和可取消性，再考虑优化。
6. 输出前自检：命名、错误、context、资源释放、测试覆盖关键路径。

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

- 结构化规范文档：更适合做日常开发、设计取舍和代码审查。
- 主题参考文档：更适合按问题类型快速查约定、示例和注意事项。

| 主题 | 参考文档 | 使用场景 |
|------|---------|----------|
| 代码风格与可读性 | `style-and-readability.md` | 命名、格式化、注释、控制结构、测试可读性 |
| 安全编码 | `security.md` | 输入验证、密码存储、加密、并发安全、测试安全 |
| 性能与可靠性 | `performance-and-reliability.md` | 内存优化、并发性能、I/O 优化、测试策略 |
| 核心原则与设计 | `core-principles-and-design.md` | 接口设计、包设计、错误处理、可测试性设计 |

### 何时查看哪个文档

- **日常编码** → `style-and-readability.md`
- **处理敏感数据** → `security.md`
- **性能优化** → `performance-and-reliability.md`
- **架构设计** → `core-principles-and-design.md`
- **设计错误处理策略** → `core-principles-and-design.md` + `style-and-readability.md`
- **整理测试与可靠性策略** → `performance-and-reliability.md` + `core-principles-and-design.md`

### 文档使用建议

- 需要做设计取舍时，优先看 `core-principles-and-design.md`。
- 需要查编码细节和常见示例时，优先看 `style-and-readability.md`。
- 需要评估安全与可靠性风险时，结合 `security.md` 和 `performance-and-reliability.md` 一起看。

## AI 默认输出要求

- 实现任务：优先直接给出代码修改，再简短说明采用的 Go 惯用法。
- 审查任务：先报问题，再给总结；优先错误处理、并发风险、API 设计、资源泄漏、测试缺口。
- 优化任务：先说明是否有证据表明存在瓶颈，再决定是否引入复杂优化。

## AI 例外条件

- 项目已有稳定风格且与通用 Go 惯用法略有差异时，优先保持项目一致性。
- 性能优化未有 profiling 或明确瓶颈证据时，不主动引入 `sync.Pool` 等复杂优化。
- 内部包、内部服务可适当放宽公共库级别的注释和兼容性要求，但不能牺牲可维护性。

## AI 最小工作流

### 实现任务

1. 先判断任务重点是设计、可读性、安全、可靠性还是性能。
2. 读取最相关的 1-2 份 reference，不整套通读。
3. 按 Go 惯用法实现，优先简单、显式、易测试。
4. 自检错误处理、`context`、资源释放和命名。
5. 输出修改结果，并简短说明关键取舍。

### 审查任务

1. 先看错误处理、并发风险、资源泄漏和 API 设计。
2. 再看可读性、测试缺口和不必要复杂度。
3. 按严重程度列出问题，优先报真实风险而非风格噪音。
4. 最后给简短总结和残余风险。

### 文档任务

1. 先判断文档面向 AI 执行还是人类阅读。
2. 优先写可判定、可执行、少歧义的规则。
3. 用短句和清单组织内容，减少背景铺垫。
4. 保持示例与当前规则一致。

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

// 需要新增上下文时包装
return fmt.Errorf("查找项目 %s: %w", id, err)

// 使用 errors.Is / errors.As 判断错误
if errors.Is(err, ErrNotFound) { ... }

// 避免过度使用 panic（仅用于真正的不可恢复情况）
```

### 并发原则

```go
// 用 channel 编排任务或转移数据所有权
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
- ❌ 使用明文、可逆加密或弱哈希存储密码
- ❌ 记录敏感信息到日志
- ❌ 无保护的并发访问共享状态
- ❌ 外部调用不设置超时
- ❌ 硬编码密钥（从环境变量读取）

---

## 推荐实践

1. **格式化优先** - 所有代码必须通过 `gofmt` 和 `goimports`
2. **命名清晰** - 使用驼峰式、语义明确的名称
3. **接口小而专注** - 单方法接口用 `-er` 后缀
4. **错误是值** - 立即检查；有新增上下文时用 `%w` 包装，否则直接返回；优先标准库 `errors`
5. **并发原语按场景选择** - 编排与所有权转移用 channel，共享状态保护用 mutex/atomic
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
- [ ] 密码使用成熟的 KDF/哈希方案（默认 `bcrypt`，允许项目约定的 `argon2id` 等）
- [ ] 敏感信息已脱敏或从环境变量读取
- [ ] 并发访问共享状态使用 mutex
- [ ] 外部调用（HTTP、数据库）设置了超时
- [ ] 使用 defer 确保资源释放
- [ ] 包名全小写、简短
- [ ] 接口在使用方定义
- [ ] 函数职责单一；当参数列表影响可读性时，使用结构体或选项模式

---

## 参考资料

官方文档和更多详细信息请查看：
- [Effective Go](https://go.dev/doc/effective_go)
- [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
- [Go Blog](https://go.dev/blog/)

完整的规范文档请查看 `references/` 目录下的各个主题文档。
