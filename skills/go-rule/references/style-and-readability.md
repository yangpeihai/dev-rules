# Go 代码风格与可读性规范

## AI 使用提示

- 当任务涉及命名、注释、函数签名、导入组织、测试命名或代码可读性时，先看本文件。
- 默认优先减少噪音和嵌套，不为了“聪明”牺牲可读性。
- 如果一段 Go 代码看起来像需要解释而不是能直接读懂，优先考虑简化写法。

> AI Code 遵循规约，确保代码可读性和风格统一。

## 1. 命名规范

```go
// 包名：全小写，简短，单单词
package user
package auth
package httpserver

// 变量：驼峰式，语义清晰
var userCount int
var isActive bool
var httpClient *http.Client

// 常量：驼峰式，导出常量大写
const (
    MaxRetries = 3
    DefaultTimeout = 30 * time.Second
)

// 枚举：使用 iota + 类型前缀
const (
    StatusPending Status = iota
    StatusActive
    StatusInactive
)

// 接口：单方法接口用 -er 后缀，多方法接口用描述性名称
type Reader interface { Read(p []byte) (n int, err error) }
type FileSystem interface { ... }  // 多方法
```

## 2. 格式化

```bash
# 必须通过 gofmt 和 goimports
gofmt -w .
goimports -w .
```

## 3. 注释

```go
// 包注释：描述包的功能
// Package crypto 提供加密和解密功能。
package crypto

// 对外暴露的公共 API、跨包复用组件、复杂约束的导出标识符应注释
// NewClient 创建并返回一个新的 HTTP 客户端。
// timeout 指定请求超时时间，0 表示使用默认超时。
func NewClient(timeout time.Duration) *Client { ... }

// TODO/FIXME 格式
// TODO(username): 这里的错误处理需要改进
// FIXME: 在高并发场景下可能出现竞态条件
```

## 4. 控制结构

```go
// if: 错误处理优先，变量作用域最小化
if err != nil {
    return err
}

// 变量定义在 if 中，限制作用域
if result, err := someFunc(); err != nil {
    return err
} else {
    useResult(result)
}

// for: 优先使用 range
for _, item := range items { ... }

// switch: 不需要显式 break
switch status {
case StatusActive:
    handleActive()
default:
    handleUnknown()
}
```

## 5. 函数设计

```go
// 当参数列表影响可读性时，使用结构体或选项模式
func ProcessUser(ctx context.Context, userID string, opts Options) (*User, error)

// 返回值：结果 + 错误
func GetData() (data []byte, err error) { ... }

// 多返回值顺序：(result, error)
func Divide(a, b float64) (result float64, err error) { ... }
```

## 6. 错误处理

```go
// 立即检查错误
result, err := someFunc()
if err != nil {
    return fmt.Errorf("操作失败: %w", err)
}

// 错误包装：在新增上下文时使用 %w 保留原始错误
return fmt.Errorf("连接数据库失败: %w", err)

// 错误判断
if errors.Is(err, ErrNotFound) { ... }

// 仅透传时可以直接返回，避免无意义包装
if err != nil {
    return err
}
```

## 7. 结构体组织

```go
// 字段顺序：导出字段 -> 非导出字段 -> 内嵌字段
type User struct {
    // 导出字段
    ID    string
    Name  string
    Email string

    // 非导出字段
    mu    sync.RWMutex

    // 内嵌字段
    log *logrus.Logger
}

// 初始化：命名参数
user := &User{
    ID:    "123",
    Name:  "张三",
}
```

## 8. 导入组织

```go
import (
    // 1. 标准库
    "context"
    "time"

    // 2. 项目内部包
    "myproject/internal/auth"

    // 3. 第三方库
    "github.com/gin-gonic/gin"
)
```

## 9. 并发可读性

```go
// channel 命名：描述用途
type Service struct {
    jobQueue   chan Job
    resultChan chan Result
    shutdownCh chan struct{}
}
```

## 10. 代码简化

```go
// 避免过度抽象，直接明了
func calculatePrice(quantity int, unitPrice float64) float64 {
    return float64(quantity) * unitPrice
}

// 避免深层嵌套，使用早期返回
func process(data []byte) error {
    if len(data) == 0 {
        return errors.New("数据为空")
    }
    // 主要逻辑
    return doProcess(data)
}
```

## 11. 禁止事项

- ❌ 不使用 `gofmt` 格式化
- ❌ 使用无意义的变量名（u, data, flag1）
- ❌ 公共 API 或复杂导出标识符缺少必要注释
- ❌ 忽略错误检查
- ❌ 使用位置依赖初始化结构体
- ❌ 过度嵌套（超过 3 层）
- ❌ 文件职责混乱、过大且难以维护（不要只以行数机械判断）

## 12. 测试可读性

### 测试文件命名

```go
// 测试文件：xxx_test.go，与被测文件同包
// user_service_test.go 测试 user_service.go
// handler_test.go 测试 handler.go
```

### 测试函数命名

```go
// 格式：TestFunctionName_Scenario_ExpectedResult
// 优点：清晰描述被测函数、场景、预期结果

func TestUserService_CreateUser_ValidInput_Success(t *testing.T) { }
func TestUserService_CreateUser_DuplicateEmail_ReturnError(t *testing.T) { }
func TestUserService_DeleteUser_NotFound_ReturnError(t *testing.T) { }
func TestParseEmail_EmptyString_ReturnError(t *testing.T) { }
func TestParseEmail_InvalidFormat_ReturnError(t *testing.T) { }
```

### 表驱动测试

```go
// 使用表驱动测试多个场景
func TestParseEmail(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        wantErr bool
        errMsg  string
    }{
        {"有效邮箱", "user@example.com", false, ""},
        {"缺少@", "userexample.com", true, "无效的邮箱格式"},
        {"缺少域名", "user@", true, "无效的邮箱格式"},
        {"多个@", "user@@example.com", true, "无效的邮箱格式"},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := ParseEmail(tt.input)
            if (err != nil) != tt.wantErr {
                t.Errorf("ParseEmail() error = %v, wantErr %v", err, tt.wantErr)
                return
            }
            if tt.wantErr && err.Error() != tt.errMsg {
                t.Errorf("ParseEmail() error = %v, want %v", err, tt.errMsg)
            }
            if !tt.wantErr && got != tt.input {
                t.Errorf("ParseEmail() = %v, want %v", got, tt.input)
            }
        })
    }
}
```

### 测试辅助函数

```go
// 使用 t.Helper() 标记辅助函数
// 错误信息会指向调用位置而非辅助函数内部

func setupTestUser(t *testing.T) *User {
    t.Helper()
    user := &User{
        ID:    "test-123",
        Name:  "测试用户",
        Email: "test@example.com",
    }
    if err := user.Validate(); err != nil {
        t.Fatal(err)
    }
    return user
}

func setupTestDB(t *testing.T) *sql.DB {
    t.Helper()
    db, err := sql.Open("postgres", os.Getenv("TEST_DB_URL"))
    if err != nil {
        t.Fatalf("连接测试数据库失败: %v", err)
    }
    return db
}
```

### 子测试组织

```go
// 使用 t.Run() 组织相关测试
func TestUserService(t *testing.T) {
    service := setupTestService(t)

    t.Run("CreateUser", func(t *testing.T) {
        t.Run("有效输入_创建成功", func(t *testing.T) {
            // ...
        })
        t.Run("重复邮箱_返回错误", func(t *testing.T) {
            // ...
        })
    })

    t.Run("DeleteUser", func(t *testing.T) {
        t.Run("用户不存在_返回错误", func(t *testing.T) {
            // ...
        })
    })
}
```

### 测试注释

```go
// 测试用例应该清晰描述意图
// 注释应说明：当 [条件] 时，[操作] 应该 [结果]

// 当用户已存在时，创建用户应该返回重复错误
func TestCreateUser_DuplicateEmail_ReturnsDuplicateError(t *testing.T) {
    // 测试代码...
}

// 当邮箱格式无效时，应该返回验证错误
func TestCreateUser_InvalidEmail_ReturnsValidationError(t *testing.T) {
    // 测试代码...
}
```

### 断言清晰性

```go
// ✅ 清晰的断言消息
if user.ID == "" {
    t.Error("用户 ID 不应为空")
}
if got, want := user.Status, StatusActive; got != want {
    t.Errorf("用户状态 = %v, 期望 %v", got, want)
}

// ❌ 模糊的断言消息
if user.ID == "" {
    t.Error("失败")
}
if user.Status != StatusActive {
    t.Error("状态错误")
}
```

### 禁止事项（测试）

- ❌ 测试函数命名不清晰（`Test1`、`TestSomething`）
- ❌ 不使用 `t.Helper()` 标记辅助函数
- ❌ 断言消息模糊不清
- ❌ 测试用例缺少描述性名称
- ❌ 过度复杂的测试逻辑（测试本身应该简单）
