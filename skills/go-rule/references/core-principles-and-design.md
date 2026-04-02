# Go 核心原则与设计规范

## AI 使用提示

- 当任务涉及接口设计、包边界、context 传递、错误链或依赖注入时，先看本文件。
- 如果不确定某段逻辑该如何分层、抽象或暴露接口，默认以“更小、更显式、更易测”为准。
- 如果某个设计需要长解释才能说清，通常说明抽象可能过重。

> AI Code 遵循规约，编写地道的 Go 代码。

## 1. 设计哲学

```go
// 简单性优于复杂性
func calculateTotal(items []Item) float64 {
    var total float64
    for _, item := range items {
        total += item.Price * float64(item.Quantity)
    }
    return total
}

// 可读性优先
func isValidUser(u *User) bool {
    return u.ID != "" && u.Name != "" && u.Email != ""
}

// 显式优于隐式：始终检查错误
result, err := someFunction()
if err != nil {
    return err
}
```

## 2. 接口设计

```go
// 接口应该小而专注
type Reader interface {
    Read(p []byte) (n int, err error)
}

// 按需组合接口
type ReadWriter interface {
    Reader
    Writer
}

// 在使用方定义接口
type UserService struct {
    repo UserRepository  // 使用者定义需要的接口
}

type UserRepository interface {
    Find(id string) (*User, error)
    Save(user *User) error
}

// 接受接口，返回结构体
func ProcessData(r io.Reader) (*Result, error) { ... }
```

## 3. 错误处理

```go
// 错误是值
func (s *Service) Process(id string) error {
    item, err := s.repo.Find(id)
    if err != nil {
        return err  // 无新增上下文时直接返回
    }
    return s.processor.Process(item)
}

// 需要新增上下文时包装
return fmt.Errorf("查找项目 %s: %w", id, err)

// 使用标准库判断错误链
if errors.Is(err, ErrNotFound) { ... }

// 避免过度使用 panic（仅用于真正的不可恢复情况）
if s.config.DBURL == "" {
    panic("数据库 URL 未配置")
}
```

## 4. 并发原则

```go
// 用 channel 处理任务编排与所有权转移
type Worker struct {
    jobs    <-chan Job
    results chan<- Result
}

// 共享状态优先显式同步
type Counter struct {
    mu    sync.Mutex
    value int
}

// 不要共享正在通信的 channel
func (s *Server) Start(workers int) {
    for i := 0; i < workers; i++ {
        go s.worker(s.jobs)  // jobs 只由 Server 写入
    }
}
```

## 5. 包设计

```go
// 包名：简短、全小写、单单词
package user
package auth

// 避免循环依赖
// main -> handler -> service -> repository

// 包应该内聚
user/
  - user.go       // 核心类型
  - repository.go // 数据访问
  - service.go    // 业务逻辑
```

## 6. 类型设计

```go
// 为行为定义类型
type Validator interface {
    Validate() error
}

func (u *User) Validate() error {
    if u.ID == "" {
        return errors.New("ID 不能为空")
    }
    return nil
}

// 零值应该可用
type Counter struct {
    mu    sync.Mutex
    count int
}
// Counter{} 可以直接使用
```

## 7. 函数设计

```go
// 函数做一件事
func validateInput(input string) error { ... }
func processInput(input string) (string, error) { ... }

// 参数列表不应过长（使用结构体或选项模式）
type Config struct {
    Host    string
    Port    int
    Timeout time.Duration
}
func NewServer(cfg Config) *Server { ... }

// 或使用选项模式
func NewServer(opts ...ServerOption) *Server { ... }
```

## 8. context 传递

```go
// context 作为第一个参数
func (s *Service) ProcessUser(ctx context.Context, userID string) error {
    user, err := s.repo.Find(ctx, userID)
    // ...
}
```

## 9. 接收者类型

```go
// 指针接收者：需要修改或大类型
func (u *User) SetName(name string) {
    u.Name = name
}

// 值接收者：不可变操作或小类型
func (u User) ID() string {
    return u.id
}
```

## 10. 代码组织

```
project/
├── cmd/server/main.go       # 入口点
├── internal/
│   ├── handler/             # HTTP 处理器
│   ├── service/             # 业务逻辑
│   ├── repository/          # 数据访问
│   └── model/               # 领域模型
│   └── types/               # 数据类型
└── pkg/util/                # 可重用工具
```

## 11. defer 使用

```go
// 先检查错误，再使用 defer
file, err := os.Open(path)
if err != nil {
    return err
}
defer file.Close()

// 限制 defer 作用域
result, err := func() (string, error) {
    r, err := acquire()
    if err != nil {
        return "", err
    }
    defer release(r)
    return use(r), nil
}()
```

## 12. 常见陷阱

```go
// 循环中的 range
for _, item := range items {
    go func(i Item) {  // 传递副本
        fmt.Println(i.Name)
    }(item)
}

// 区分 nil 和 empty slice
var items []Item     // nil -> JSON: null
items = []Item{}     // empty -> JSON: []
```

## 13. 设计清单

- [ ] 代码简单明了，避免过度抽象
- [ ] 接口小而专注
- [ ] 在使用方定义接口
- [ ] 错误作为值处理
- [ ] 避免不必要的 panic
- [ ] 包名简短、小写
- [ ] 避免循环依赖
- [ ] 零值可用
- [ ] 函数职责单一
- [ ] context 作为第一个参数
- [ ] 使用依赖注入便于测试
- [ ] 避免全局状态
- [ ] 提供测试辅助构造器

## 14. 可测试性设计

### 依赖注入

```go
// ✅ 通过接口注入依赖，便于测试
type UserService struct {
    repo UserRepository
    log  Logger
}

func NewUserService(repo UserRepository, log Logger) *UserService {
    return &UserService{
        repo: repo,
        log:  log,
    }
}

// 测试时可以注入 mock
func TestUserService_GetUser(t *testing.T) {
    mockRepo := &MockUserRepository{}
    mockLog := &MockLogger{}
    service := NewUserService(mockRepo, mockLog)
    // ...
}

// ❌ 硬编码依赖，难以测试
type UserService struct {
    repo *UserRepository  // 具体类型
    log  *logrus.Logger   // 具体类型
}

func NewUserService() *UserService {
    return &UserService{
        repo: NewUserRepository(),  // 硬编码
        log:  logrus.New(),         // 硬编码
    }
}
```

### 小接口设计

```go
// ✅ 定义小接口，易于 mock
type Logger interface {
    Info(msg string, fields ...Field)
    Error(msg string, fields ...Field)
}

type UserRepository interface {
    Find(ctx context.Context, id string) (*User, error)
    Save(ctx context.Context, user *User) error
}

// 测试时只需实现少数方法
type MockLogger struct {
    logs []string
}

func (m *MockLogger) Info(msg string, fields ...Field) {
    m.logs = append(m.logs, msg)
}

func (m *MockLogger) Error(msg string, fields ...Field) {
    m.logs = append(m.logs, "ERROR: "+msg)
}

// ❌ 大接口难以 mock
type GiantInterface interface {
    Find(...) error
    Save(...) error
    Delete(...) error
    Update(...) error
    List(...) error
    // ... 20 个方法
}

// 测试时需要实现所有方法
type MockGiantInterface struct {
    // 需要为所有 20 个方法提供实现
}
```

### 测试辅助构造器

```go
// 提供便捷的测试构造函数
func NewTestUser(opts ...TestUserOption) *User {
    user := &User{
        ID:       "test-id",
        Name:     "测试用户",
        Email:    "test@example.com",
        Status:   StatusActive,
        CreateAt: time.Now(),
    }
    for _, opt := range opts {
        opt(user)
    }
    return user
}

// 测试选项模式
type TestUserOption func(*User)

func WithTestID(id string) TestUserOption {
    return func(u *User) { u.ID = id }
}

func WithTestEmail(email string) TestUserOption {
    return func(u *User) { u.Email = email }
}

func WithTestStatus(status Status) TestUserOption {
    return func(u *User) { u.Status = status }
}

// 使用示例
func TestUserProcessor_Process(t *testing.T) {
    // 使用默认测试用户
    user1 := NewTestUser()

    // 自定义部分字段
    user2 := NewTestUser(
        WithTestID("custom-id"),
        WithTestEmail("custom@test.com"),
        WithTestStatus(StatusInactive),
    )

    // 测试逻辑...
}
```

### 避免全局状态

```go
// ✅ 使用依赖传递状态
type Handler struct {
    db    *Database
    cache *Cache
}

func NewHandler(db *Database, cache *Cache) *Handler {
    return &Handler{db: db, cache: cache}
}

// 测试时可以注入测试依赖
func TestHandler_HandleRequest(t *testing.T) {
    testDB := setupTestDB(t)
    testCache := setupTestCache(t)
    handler := NewHandler(testDB, testCache)
    // ...
}

// ❌ 全局变量导致测试隔离困难
var globalDB *Database
var globalCache *Cache

type Handler struct{}

func NewHandler() *Handler {
    return &Handler{}
}

func (h *Handler) HandleRequest() error {
    // 使用全局变量，测试难以隔离
    return globalDB.Query(...)
}
```

### context 传递

```go
// ✅ 通过 context 传递请求级数据
func (s *Service) Process(ctx context.Context, id string) error {
    // 可以在测试中控制 context 超时、取消等
    user, err := s.repo.Find(ctx, id)
    if err != nil {
        return err
    }
    return s.processor.Process(ctx, user)
}

// 测试超时行为
func TestService_Process_Timeout(t *testing.T) {
    ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
    defer cancel()

    service := NewService(&SlowRepository{})
    err := service.Process(ctx, "123")

    if err == nil {
        t.Error("期望超时错误，得到 nil")
    }
}

// 测试取消行为
func TestService_Process_Cancel(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())

    go func() {
        time.Sleep(50 * time.Millisecond)
        cancel() // 取消 context
    }()

    service := NewService(&SlowRepository{})
    err := service.Process(ctx, "123")

    if err == nil || !errors.Is(err, context.Canceled) {
        t.Errorf("期望 context.Canceled，得到 %v", err)
    }
}
```

### 接口在设计时考虑测试

```go
// ✅ 接口设计考虑测试场景
type FileStorage interface {
    // 返回 error 而非 panic，便于测试错误处理
    Read(path string) ([]byte, error)
    Write(path string, data []byte) error
    Delete(path string) error
}

// 测试错误处理
func TestFileProcessor_Read_NotFound(t *testing.T) {
    mockStorage := &MockFileStorage{
        files: map[string][]byte{},
    }
    processor := NewFileProcessor(mockStorage)

    _, err := processor.Read("nonexistent.txt")
    if err == nil {
        t.Error("期望错误，得到 nil")
    }
}

// ❌ 返回 bool 或 panic 的设计难以测试
type FileStorageBad interface {
    Read(path string) ([]byte, bool)  // 用 bool 表示错误
    Write(path string, data []byte)   // 失败时 panic
}
```

### 禁止事项（可测试性）

- ❌ 硬编码依赖（直接 new 具体类型）
- ❌ 使用全局变量或单例
- ❌ 定义大而全的接口
- ❌ 函数内部直接调用外部服务
- ❌ 不使用 context 传递请求级控制
