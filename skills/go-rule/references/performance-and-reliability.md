# Go 性能与可靠性规范

> AI Code 遵循规约，编写高性能可靠的代码。

## 1. 内存管理

```go
// 预分配切片容量
result := make([]string, 0, len(items))

// 预分配 map
m := make(map[string]int, 100)

// 使用 sync.Pool 复用对象
var bufferPool = sync.Pool{
    New: func() interface{} { return new(bytes.Buffer) },
}

// 使用 strings.Builder 构建字符串
var builder strings.Builder
builder.Grow(1000)
```

## 2. 并发性能

```go
// 使用 worker pool 限制并发数
func processItems(items []Item) {
    const numWorkers = 10
    itemChan := make(chan Item, len(items))

    for i := 0; i < numWorkers; i++ {
        go worker(itemChan)
    }

    for _, item := range items {
        itemChan <- item
    }
    close(itemChan)
}

// 使用分片锁减少竞争
type ShardedMap struct {
    shards []*sync.RWMutex
    maps   []map[string]interface{}
}

// 使用原子操作
atomic.AddInt64(&counter.value, 1)
```

## 3. I/O 性能

```go
// 使用缓冲 I/O
bw := bufio.NewWriter(w)
defer bw.Flush()

// 批量处理数据库操作
const batchSize = 100
for i := 0; i < len(items); i += batchSize {
    // 批量插入
}

// 流式处理大文件
scanner := bufio.NewScanner(file)
for scanner.Scan() {
    processLine(scanner.Text())
}
```

## 4. 错误处理可靠性

```go
// 始终检查错误
file, err := os.Open(path)
if err != nil {
    return err
}
defer file.Close()

// 使用 defer 确保清理
resource, err := acquireResource()
if err != nil {
    return err
}
defer releaseResource(resource)

// 错误包装保留上下文
return fmt.Errorf("处理用户 %s 失败: %w", userID, err)
```

## 5. 超时控制

```go
// 所有外部调用必须设置超时
func (c *Client) FetchData(ctx context.Context, id string) error {
    ctx, cancel := context.WithTimeout(ctx, 10*time.Second)
    defer cancel()

    req, _ := http.NewRequestWithContext(ctx, "GET", url, nil)
    // ...
}

// 数据库操作超时
rows, err := db.QueryContext(ctx, query)
```

## 6. 资源限制

```go
// 连接池配置
db.SetMaxOpenConns(25)
db.SetMaxIdleConns(5)
db.SetConnMaxLifetime(5 * time.Minute)

// 限制读取大小
limited := io.LimitReader(r, maxSize)
```

## 7. 可观察性

```go
// 结构化日志
logger.Info("处理用户",
    zap.String("user_id", userID),
    zap.Duration("duration", time.Since(start)),
)

// 添加指标
requestDuration.WithLabelValues(method, endpoint, status).Observe(duration)

// 分布式追踪
ctx, span := otel.Tracer("service").Start(ctx, "ProcessOrder")
defer span.End()
```

## 8. 优雅关闭

```go
func (s *Server) Shutdown(ctx context.Context) error {
    // 1. 停止接受新请求
    s.httpServer.Shutdown(ctx)

    // 2. 关闭数据库连接
    s.db.Close()

    return nil
}

// 处理信号
sigChan := make(chan os.Signal, 1)
signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
<-sigChan
server.Shutdown(context.Background())
```

## 9. 测试

### 覆盖率要求

```bash
# 单元测试覆盖率必须达到 85%
go test -cover ./...

# 生成覆盖率报告
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out -o coverage.html

# 查看每个包的覆盖率
go test -cover ./internal/...

# 按函数查看覆盖率
go tool cover -func=coverage.out
```

### 测试组织

```go
// 表驱动测试
func TestParseEmail(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        wantErr bool
    }{
        {"有效邮箱", "user@example.com", false},
        {"无效邮箱", "userexample.com", true},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // 测试逻辑
        })
    }
}

// 使用 t.Cleanup 清理资源
func TestWithResource(t *testing.T) {
    resource := acquireTestResource()
    t.Cleanup(func() { releaseTestResource(resource) })
}

// 使用 t.Setup 进行初始化
func TestWithSetup(t *testing.T) {
    var db *sql.DB

    t.Run("setup", func(t *testing.T) {
        var err error
        db, err = sql.Open("postgres", os.Getenv("TEST_DB_URL"))
        if err != nil {
            t.Fatal(err)
        }
    })

    t.Run("test case 1", func(t *testing.T) {
        // 使用 db
    })
}
```

### HTTP Handler 测试（httptest）

```go
import (
    "net/http"
    "net/http/httptest"
    "encoding/json"
)

func TestHandler_GetUser(t *testing.T) {
    // 创建 mock service
    mockService := &MockUserService{
        users: map[string]*User{
            "123": {ID: "123", Name: "张三", Email: "zhangsan@example.com"},
        },
    }

    handler := NewHandler(mockService)

    // 创建测试请求
    req := httptest.NewRequest("GET", "/users/123", nil)
    w := httptest.NewRecorder()

    // 执行请求
    handler.ServeHTTP(w, req)

    // 验证状态码
    if w.Code != http.StatusOK {
        t.Errorf("期望状态码 %d，得到 %d", http.StatusOK, w.Code)
    }

    // 验证响应体
    var got User
    if err := json.NewDecoder(w.Body).Decode(&got); err != nil {
        t.Fatal(err)
    }

    if got.ID != "123" {
        t.Errorf("期望 ID 123，得到 %s", got.ID)
    }
}

// 测试 POST 请求
func TestHandler_CreateUser(t *testing.T) {
    handler := NewHandler(&MockUserService{})

    body := `{"name": "李四", "email": "lisi@example.com"}`
    req := httptest.NewRequest("POST", "/users", strings.NewReader(body))
    req.Header.Set("Content-Type", "application/json")
    w := httptest.NewRecorder()

    handler.ServeHTTP(w, req)

    if w.Code != http.StatusCreated {
        t.Errorf("期望状态码 %d，得到 %d", http.StatusCreated, w.Code)
    }
}
```

### 接口 Mock（gomock）

```go
//go:generate go run go.uber.org/mock/mockgen -source=service.go -destination=mock_service.go

type MockUserRepository struct {
    ctrl *gomock.Controller
}

func TestUserService_GetUser(t *testing.T) {
    ctrl := gomock.NewController(t)
    defer ctrl.Finish()

    mockRepo := NewMockUserRepository(ctrl)

    // 设置 mock 期望
    mockRepo.EXPECT().
        Find(gomock.Any(), "123").
        Return(&User{ID: "123", Name: "张三"}, nil)

    mockRepo.EXPECT().
        Find(gomock.Any(), "999").
        Return(nil, ErrNotFound)

    service := NewUserService(mockRepo)

    // 测试成功场景
    user, err := service.GetUser(context.Background(), "123")
    if err != nil {
        t.Fatalf("意外错误: %v", err)
    }
    if user.ID != "123" {
        t.Errorf("期望 ID 123，得到 %s", user.ID)
    }

    // 测试失败场景
    _, err = service.GetUser(context.Background(), "999")
    if err != ErrNotFound {
        t.Errorf("期望 ErrNotFound，得到 %v", err)
    }
}

// 使用 gomock.Matcher 进行参数匹配
func TestUserService_UpdateUser(t *testing.T) {
    ctrl := gomock.NewController(t)
    defer ctrl.Finish()

    mockRepo := NewMockUserRepository(ctrl)

    // 匹配任意 context
    mockRepo.EXPECT().
        Save(gomock.Any(), gomock.Any()).
        Return(nil)

    // 匹配特定的 User 对象
    mockRepo.EXPECT().
        Save(gomock.Any(), gomock.Cond(func(u *User) bool {
            return u.Email == "new@example.com"
        })).
        Return(nil)

    service := NewUserService(mockRepo)
    // ...
}
```

### 并发测试（按需）

```go
// 对于涉及并发的代码，使用 race detector
// 运行：go test -race ./...

func TestCache_ConcurrentAccess(t *testing.T) {
    cache := NewCache()
    done := make(chan bool)

    // 并发写入
    for i := 0; i < 100; i++ {
        go func(id int) {
            cache.Set(id, fmt.Sprintf("value%d", id))
            done <- true
        }(i)
    }

    // 并发读取
    for i := 0; i < 100; i++ {
        go func(id int) {
            cache.Get(id)
            done <- true
        }(i)
    }

    // 等待所有 goroutine 完成
    for i := 0; i < 200; i++ {
        <-done
    }
}
```

### 测试辅助函数

```go
// 创建测试用的 HTTP 服务器
func newTestServer(t *testing.T, handler http.Handler) *httptest.Server {
    t.Helper()
    server := httptest.NewServer(handler)
    t.Cleanup(func() { server.Close() })
    return server
}

// 创建测试用的 context
func newTestContext(t *testing.T) context.Context {
    t.Helper()
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    t.Cleanup(cancel)
    return ctx
}
```

### 测试命令

```bash
# 运行所有测试
go test ./...

# 运行测试并显示覆盖率
go test -cover ./...

# 运行指定包的测试
go test ./internal/service/...

# 运行特定测试
go test -run TestUserService_CreateUser ./internal/service/...

# 并发安全检查
go test -race ./...

# 生成覆盖率报告
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out -o coverage.html

# 查看函数级别的覆盖率
go tool cover -func=coverage.out | grep -v "100.0%"
```

## 10. 性能清单

- [ ] 预分配切片/map 容量
- [ ] 使用 sync.Pool 复用对象
- [ ] 使用 buffer 进行 I/O
- [ ] 批量处理数据库操作
- [ ] 限制并发 goroutine 数量
- [ ] 所有外部调用设置超时
- [ ] 使用连接池
- [ ] 添加关键路径指标
- [ ] 实现优雅关闭
- [ ] 使用 pprof 进行性能分析
- [ ] 单元测试覆盖率达到 85%
- [ ] 并发代码通过 race detector 检查
- [ ] HTTP handler 使用 httptest 测试
- [ ] 接口依赖使用 gomock mock
