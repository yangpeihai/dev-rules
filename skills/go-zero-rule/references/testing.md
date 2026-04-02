# 测试规范

## 核心原则

- 优先覆盖关键业务路径、错误分支和回归风险点，而不是只追求覆盖率数字。
- Logic 层测试优先级高于样板代码和薄封装代码。
- 测试应服务于可维护性，优先简单、稳定、可读的替身和断言方式。

## 默认规则

- 测试文件使用 `xxx_test.go`，通常与源文件同包。
- 测试函数以 `Test` 开头，接收 `*testing.T`。
- 多场景测试优先表驱动。
- 涉及并发、共享状态或竞态风险的代码按需使用 `-race`。

## 单元测试

### 基本要求

- **测试文件**: `xxx_test.go`，与源文件同包
- **测试函数**: 以 `Test` 开头，接收 `*testing.T` 参数
- **表驱动测试**: 优先使用

### 测试示例

```go
// internal/logic/loginlogic_test.go
package logic

import (
    "context"
    "testing"

    "github.com/stretchr/testify/assert"
)

func TestLoginLogic_Login(t *testing.T) {
    tests := []struct {
        name    string
        req     *types.LoginReq
        wantErr bool
        errMsg  string
    }{
        {
            name: "正常登录",
            req: &types.LoginReq{
                Username: "admin",
                Password: "123456",
            },
            wantErr: false,
        },
        {
            name: "密码错误",
            req: &types.LoginReq{
                Username: "admin",
                Password: "wrong",
            },
            wantErr: true,
            errMsg:  "密码错误",
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // 初始化
            svcCtx := &svc.ServiceContext{...}
            logic := NewLoginLogic(context.Background(), svcCtx)

            // 执行
            resp, err := logic.Login(tt.req)

            // 断言
            if tt.wantErr {
                assert.Error(t, err)
                assert.Contains(t, err.Error(), tt.errMsg)
            } else {
                assert.NoError(t, err)
                assert.NotEmpty(t, resp.AccessToken)
            }
        })
    }
}
```

## 覆盖与运行

### 运行测试并生成覆盖率报告

```bash
# 运行测试并生成覆盖率报告
go test -race -coverprofile=coverage.out ./...
go tool cover -html=coverage.out -o coverage.html

# 查看覆盖率
go tool cover -func=coverage.out | grep total
```

### 覆盖重点

- 新代码优先覆盖关键路径、错误分支和回归风险点
- 核心业务逻辑比工具函数更值得优先投入测试
- 覆盖率数字可作为观察指标，不宜脱离风险单独考核

## 推荐实践

- 为 Logic 层优先准备可替换的 `ServiceContext` 依赖或 fake。
- 对错误码映射、事务回滚、缓存一致性等高风险场景补测试。
- 断言优先检查行为和语义，不依赖脆弱的日志文本或实现细节。

## 禁止事项

- ❌ 只测 happy path
- ❌ 用覆盖率数字代替真实风险分析
- ❌ 断言过度依赖内部实现，导致重构即大面积误报

## 审查清单

- [ ] 关键业务路径、错误分支和回归风险点已覆盖
- [ ] Logic 层测试优先于样板代码测试
- [ ] 并发或共享状态代码已按需使用 `-race`
- [ ] 断言聚焦业务行为而非实现噪音
