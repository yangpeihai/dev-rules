# 推荐实践

## 核心原则

1. **Handler 保持精简** — handler 只做请求解码、调用 logic、编码响应，不堆业务代码
2. **每个用例一个 logic 文件** — `CreateOrderLogic`、`GetOrderLogic`、`CancelOrderLogic` 各自独立
3. **ServiceContext 是默认依赖入口** — 共享依赖优先集中在 `svc.NewServiceContext` 初始化，避免在业务代码中四处分散创建
4. **配置优于硬编码** — 超时时间、功能开关、下游地址等所有可调参数都放入 `etc/*.yaml`
5. **Model 层通常由 goctl 生成，Logic 层归你所有** — 生成代码与手写扩展分开管理，便于后续再生成或升级
6. **优先使用 goctl 生成代码骨架** — 使用 `goctl api`、`goctl rpc` 和 `goctl model` 快速建立标准结构，历史项目可渐进接入
7. **保持服务韧性** — 利用 go-zero 内置的熔断、限流、降载能力
8. **可观测性第一** — 从第一天起就配置好日志、指标和链路追踪

## 默认规则

- 目录结构优先遵循 go-zero 常见布局，再按项目规模做小幅调整。
- 业务逻辑集中在 Logic 层，生成代码与手写扩展分层管理。
- 共享依赖集中在 `ServiceContext`，避免四处分散初始化。

## 常用命令速查

### 项目初始化

```bash
# 创建 API 服务
goctl api go -api user.api -dir ./user-api -style goZero

# 创建 RPC 服务
goctl rpc protoc user.proto --go_out=. --go-grpc_out=. --zrpc_out=. -style goZero

# 生成 Model（带缓存）
goctl model mysql datasource \
    --url "user:password@tcp(127.0.0.1:3306/db" \
    --table "user" \
    --dir ./model \
    --cache \
    --style goZero

# 生成 Dockerfile
goctl docker -go user-api.go

# 生成 K8s 部署文件
goctl kube deploy -name user-api -namespace default -image user-api:latest -o ./user-api.yaml -port 8888
```

### 环境检查

```bash
# 检查 goctl 环境
goctl env check --install

# 验证 API 文件
goctl api validate -api user.api

# 格式化 API 文件
goctl api format -api user.api
```

### 代码生成

```bash
# 安装 goctl
go install github.com/zeromicro/go-zero/tools/goctl@latest

# 安装 protoc (macOS)
brew install protobuf

# 安装 protoc (Linux)
apt install protobuf-compiler

# 安装 Go protobuf 插件
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

### 初始化模板

```bash
# 初始化所有模板到本地
goctl template init

# 初始化特定类别模板
goctl template init --category api
goctl template init --category rpc
goctl template init --category model
```

## 推荐实践

- 先用脚手架建立骨架，再把精力放在错误处理、事务、测试和领域边界上。
- 对历史项目优先渐进整理目录与依赖，不为“完全标准化”一次性大改稳定模块。
- 在团队内固定 `goctl` 使用方式、模板来源、目录命名和错误码策略。

## 禁止事项

- ❌ 把“用了 goctl”当成代码质量的替代品
- ❌ 在生成代码上直接堆积大量业务改动
- ❌ 忽视历史项目的兼容成本，强行一次性重构所有目录

## 审查清单

- [ ] 分层、目录和依赖组织符合团队约定
- [ ] 生成代码与手写扩展边界清晰
- [ ] 历史项目演进方案是渐进的
- [ ] 可观测性与韧性配置没有被忽略

## 参考资料

- [go-zero 官方文档](https://go-zero.dev/zh-cn/)
- [go-zero 设计原则](https://go-zero.dev/zh-cn/concepts/design-principles/)
- [go-zero 项目结构](https://go-zero.dev/zh-cn/concepts/project-structure/)
- [go-zero 代码规范](https://go-zero.dev/zh-cn/community/code-style/)
- [goctl API 代码生成](https://go-zero.dev/zh-cn/reference/goctl/api/)
- [goctl RPC 代码生成](https://go-zero.dev/zh-cn/reference/goctl/rpc/)
- [goctl Model 代码生成](https://go-zero.dev/zh-cn/reference/goctl/model/)
- [构建 API 服务](https://go-zero.dev/zh-cn/guides/quickstart/api-service/)
