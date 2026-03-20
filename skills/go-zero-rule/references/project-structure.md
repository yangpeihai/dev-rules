# 项目结构规范

## API 服务目录结构

```
user-api/
├── etc/
│   └── user-api.yaml              # 运行时配置
├── api/
│   └── user.api                   # API 定义文件
├── internal/
│   ├── config/
│   │   └── config.go              # 配置结构体
│   ├── handler/
│   │   ├── routes.go              # 路由注册（自动生成）
│   │   └── loginhandler.go        # HTTP 处理器（自动生成）
│   ├── logic/
│   │   └── loginlogic.go          # 业务逻辑（手动实现）
│   ├── middleware/
│   │   └── authmiddleware.go      # 中间件
│   ├── svc/
│   │   └── servicecontext.go      # 服务上下文
│   └── types/
│       └── types.go               # 请求/响应类型（自动生成）
├── model/                         # 数据模型（可选）
│   └── usermodel.go
├── pkg/                           # 公共包
│   ├── errx/                      # 错误定义
│   └── util/                      # 工具函数
├── deploy/                        # 部署配置
│   ├── sql/                       # 数据库脚本
│   └── docker/                    # Docker 配置
├── Dockerfile
├── Makefile
├── user-api.go                    # 主入口
└── go.mod
```

## RPC 服务目录结构

```
user-rpc/
├── etc/
│   └── user-rpc.yaml              # RPC 配置
├── proto/
│   └── user.proto                 # Proto 定义文件
├── internal/
│   ├── config/
│   │   └── config.go
│   ├── logic/
│   │   └── getuserlogic.go        # 业务逻辑
│   ├── server/
│   │   └── userserver.go          # gRPC 服务端
│   └── svc/
│       └── servicecontext.go
├── pb/                            # Protobuf 生成代码
│   └── user/
│       ├── user.pb.go
│       └── user_grpc.pb.go
├── userclient/                    # RPC 客户端封装
│   └── user.go
├── model/                         # 数据模型
├── user.go                        # 主入口
└── go.mod
```

## 多服务项目布局

```
project-root/
├── service/
│   ├── user/
│   │   ├── api/                   # user-api 服务
│   │   └── rpc/                   # user-rpc 服务
│   ├── order/
│   │   ├── api/
│   │   └── rpc/
│   └── payment/
│       └── rpc/
├── common/                        # 公共模块
│   ├── errx/                      # 错误码定义
│   ├── middleware/                # 公共中间件
│   ├── utils/                     # 工具函数
│   └── types/                     # 公共类型
├── proto/                         # 共享 Proto 文件
│   ├── user/
│   └── order/
├── deploy/                        # 部署配置
│   ├── docker-compose.yaml
│   └── k8s/
├── scripts/                       # 构建脚本
├── Makefile
└── README.md
```
