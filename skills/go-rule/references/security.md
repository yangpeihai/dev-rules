# Go 安全编码规范

## AI 使用提示

- 当任务涉及输入校验、密码、密钥、路径、TLS、日志脱敏或测试凭证时，先看本文件。
- 如果代码把外部输入直接拼接进 SQL、路径、命令或日志，默认视为高优先级风险。
- 当安全与便捷冲突时，默认优先安全，再说明权衡。

> AI Code 遵循规约，编写安全可靠的代码。

## 1. 输入验证

```go
// 验证所有外部输入
func CreateUser(username, email string) error {
    if len(username) < 3 || len(username) > 32 {
        return errors.New("用户名长度必须在3-32字符之间")
    }
    if !isValidEmail(email) {
        return errors.New("邮箱格式无效")
    }
    return nil
}
```

## 2. SQL 注入防护

```go
// 使用参数化查询
db.QueryRow("SELECT * FROM users WHERE id = $1", id)

// ❌ 禁止字符串拼接
// query := fmt.Sprintf("SELECT * FROM users WHERE id = '%s'", userID)
```

## 3. 路径遍历防护

```go
import (
    "path/filepath"
    "strings"
)

func ReadFile(baseDir, filename string) ([]byte, error) {
    baseDir = filepath.Clean(baseDir)
    cleanPath := filepath.Clean(filepath.Join(baseDir, filename))

    rel, err := filepath.Rel(baseDir, cleanPath)
    if err != nil {
        return nil, err
    }
    if rel == ".." || strings.HasPrefix(rel, ".."+string(filepath.Separator)) {
        return nil, errors.New("非法的文件路径")
    }
    return os.ReadFile(cleanPath)
}
```

## 4. 密码存储

```go
import "golang.org/x/crypto/bcrypt"

// 默认使用 bcrypt；如项目已有安全基线，也可使用 argon2id 等成熟方案
func HashPassword(password string) (string, error) {
    hashedPassword, err := bcrypt.GenerateFromPassword(
        []byte(password),
        bcrypt.DefaultCost,
    )
    return string(hashedPassword), err
}
```

## 5. 敏感信息处理

```go
// ❌ 禁止记录敏感信息
log.Infof("用户登录: %+v", user)  // 可能包含密码

// ✅ 脱敏处理
log.WithFields(log.Fields{
    "user_id": user.ID,
    "email": maskEmail(user.Email),
}).Info("用户登录")

// 密钥从环境变量读取
url := os.Getenv("DATABASE_URL")
```

## 6. 加密算法

```go
// ✅ 使用 AES-GCM
import "crypto/cipher"

// ❌ 禁止使用弱算法
// DES、RC4、MD5、SHA1

// ✅ 使用加密安全的随机数
import "crypto/rand"

// ❌ 禁止使用 math/rand 用于安全场景
```

## 7. HTTPS/TLS

```go
// 生产环境必须验证证书
&tls.Config{
    MinVersion: tls.VersionTLS12,
    InsecureSkipVerify: false,  // 必须
}
```

## 8. 服务端 HTTP 安全头（仅 Web/HTTP 场景）

```go
// 设置安全响应头
w.Header().Set("X-Frame-Options", "DENY")
w.Header().Set("X-Content-Type-Options", "nosniff")
w.Header().Set("Strict-Transport-Security", "max-age=31536000")
w.Header().Set("Content-Security-Policy", "default-src 'self'")
```

## 9. 并发安全

```go
// 共享状态必须使用 mutex
type Counter struct {
    mu    sync.Mutex
    value int
}

func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.value++
}

// ❌ 禁止无保护的并发访问
```

## 10. 错误信息安全

```go
// ❌ 错误信息不泄露敏感数据
return fmt.Errorf("数据库错误: %v, 用户: %s", err, username)

// ✅ 使用通用错误消息
return errors.New("认证失败")
```

## 11. 资源管理

```go
// 使用 defer 确保资源释放
file, err := os.Open(path)
if err != nil {
    return err
}
defer file.Close()
```

## 12. 超时控制

```go
// 所有外部调用必须设置超时
ctx, cancel := context.WithTimeout(ctx, 10*time.Second)
defer cancel()

req, _ := http.NewRequestWithContext(ctx, "GET", url, nil)
```

## 13. 安全清单

- [ ] 所有外部输入都经过验证
- [ ] 使用参数化查询
- [ ] 路径验证防止遍历攻击
- [ ] 密码使用成熟的 KDF/哈希方案（默认 `bcrypt`，允许项目约定的 `argon2id` 等）
- [ ] 敏感数据脱敏后再记录日志
- [ ] 密钥从环境变量读取
- [ ] 使用 HTTPS 和 TLS
- [ ] Web/HTTP 服务已按需设置安全头
- [ ] 并发访问使用 mutex
- [ ] 外部调用设置超时
- [ ] 定期更新依赖

## 14. 测试安全

### 测试中的敏感数据处理

```go
// ❌ 禁止在测试中使用真实密钥或生产数据
const TestAPIKey = "sk-live-1234567890abcdef"
const TestDBPassword = "ProdPass123!"
const TestJWTSecret = "real-production-secret-key"

// ✅ 使用测试专用假值
const TestAPIKey = "sk-test-0000000000000000"
const TestDBPassword = "TestPassword123!"
const TestJWTSecret = "test-secret-key-for-testing-only"
```

### 测试环境配置

```go
// 使用 TestMain 初始化测试环境
func TestMain(m *testing.M) {
    // 从测试环境变量或使用默认测试值
    if os.Getenv("TEST_DB_URL") == "" {
        os.Setenv("TEST_DB_URL", "postgres://localhost/test_db?sslmode=disable")
    }
    if os.Getenv("TEST_JWT_SECRET") == "" {
        os.Setenv("TEST_JWT_SECRET", "test-secret-key")
    }

    // 运行测试
    code := m.Run()

    // 清理
    os.Exit(code)
}

// 测试中使用环境变量
func TestAuthService_Login(t *testing.T) {
    secret := os.Getenv("TEST_JWT_SECRET")
    service := NewAuthService(secret)
    // ...
}
```

### 并发安全测试

```go
// 当代码涉及并发时，必须使用 race detector 运行
// go test -race ./...

// 示例：测试 Counter 的并发安全性
func TestCounter_ConcurrentIncrement(t *testing.T) {
    counter := &Counter{}
    const goroutines = 100
    const incrementsPerGoroutine = 1000

    var wg sync.WaitGroup
    wg.Add(goroutines)

    for i := 0; i < goroutines; i++ {
        go func() {
            defer wg.Done()
            for j := 0; j < incrementsPerGoroutine; j++ {
                counter.Increment()
            }
        }()
    }
    wg.Wait()

    expected := goroutines * incrementsPerGoroutine
    if got := counter.Value(); got != expected {
        t.Errorf("并发递增结果：得到 %d，期望 %d", got, expected)
    }
}

// 测试 map 的并发安全
func TestSafeMap_ConcurrentAccess(t *testing.T) {
    m := NewSafeMap()
    const goroutines = 50

    var wg sync.WaitGroup
    wg.Add(goroutines * 2)

    // 并发写入
    for i := 0; i < goroutines; i++ {
        go func(n int) {
            defer wg.Done()
            m.Set(fmt.Sprintf("key%d", n), n)
        }(i)
    }

    // 并发读取
    for i := 0; i < goroutines; i++ {
        go func(n int) {
            defer wg.Done()
            m.Get(fmt.Sprintf("key%d", n))
        }(i)
    }

    wg.Wait()
}
```

### 测试环境隔离

```go
// 使用独立的测试数据库，避免污染生产数据
func setupTestDB(t *testing.T) *sql.DB {
    t.Helper()

    // 从环境变量读取测试数据库配置
    testDBURL := os.Getenv("TEST_DB_URL")
    if testDBURL == "" {
        testDBURL = "postgres://localhost/test_db?sslmode=disable"
    }

    db, err := sql.Open("postgres", testDBURL)
    if err != nil {
        t.Fatalf("连接测试数据库失败: %v", err)
    }

    // 验证连接
    if err := db.Ping(); err != nil {
        t.Fatalf("测试数据库连接失败: %v", err)
    }

    // 清理测试数据
    t.Cleanup(func() {
        db.Exec("DELETE FROM users WHERE email LIKE 'test-%'")
        db.Exec("DELETE FROM sessions WHERE user_id IN (SELECT id FROM users WHERE email LIKE 'test-%')")
        db.Close()
    })

    return db
}

// 使用测试数据库
func TestUserRepository_Create(t *testing.T) {
    db := setupTestDB(t)
    repo := NewUserRepository(db)

    user := &User{
        ID:    "test-123",
        Name:  "测试用户",
        Email: "test-001@example.com",
    }

    err := repo.Create(context.Background(), user)
    if err != nil {
        t.Fatalf("创建用户失败: %v", err)
    }

    // 验证用户已创建
    got, err := repo.Find(context.Background(), "test-123")
    if err != nil {
        t.Fatalf("查找用户失败: %v", err)
    }
    if got.Email != "test-001@example.com" {
        t.Errorf("用户邮箱 = %s, 期望 %s", got.Email, "test-001@example.com")
    }
}
```

### 测试命令

```bash
# 运行测试并检查数据竞争
go test -race ./...

# 只运行涉及并发的测试时使用 race detector
go test -race -run TestConcurrent ./...

# 检查测试覆盖率
go test -cover ./...
```

### 禁止事项（测试安全）

- ❌ 在测试中使用真实生产密钥
- ❌ 在测试中连接生产数据库
- ❌ 测试代码中硬编码真实凭证
- ❌ 测试失败时泄露敏感信息到错误消息
- ❌ 并发代码不通过 race detector 检查

## 15. 安全清单（含测试）

- [ ] 所有外部输入都经过验证
- [ ] 使用参数化查询
- [ ] 路径验证防止遍历攻击
- [ ] 密码使用成熟的 KDF/哈希方案（默认 `bcrypt`，允许项目约定的 `argon2id` 等）
- [ ] 敏感数据脱敏后再记录日志
- [ ] 密钥从环境变量读取
- [ ] 使用 HTTPS 和 TLS
- [ ] Web/HTTP 服务已按需设置安全头
- [ ] 并发访问使用 mutex
- [ ] 外部调用设置超时
- [ ] 定期更新依赖
- [ ] 测试使用独立的假值
- [ ] 测试环境与生产环境隔离
- [ ] 并发代码通过 race detector 检查
