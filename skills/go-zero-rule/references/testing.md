# 测试规范

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

## 测试覆盖率

### 运行测试并生成覆盖率报告

```bash
# 运行测试并生成覆盖率报告
go test -race -coverprofile=coverage.out ./...
go tool cover -html=coverage.out -o coverage.html

# 查看覆盖率
go tool cover -func=coverage.out | grep total
```

### 最低覆盖率目标

- **新代码**: 80%
- **核心业务逻辑**: 90%
- **工具函数**: 100%
