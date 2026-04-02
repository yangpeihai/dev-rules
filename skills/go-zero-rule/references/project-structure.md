# 项目结构规范

## 核心原则

- 优先沿用 go-zero 常见目录布局，降低团队协作和生成代码接入成本。
- 目录职责应清晰稳定，不要为了“绝对标准”牺牲历史项目可维护性。
- 生成代码、手写业务代码、公共模块尽量分层放置，避免混杂。

## 默认规则

- API 服务默认使用 `handler / logic / svc / types` 结构。
- RPC 服务默认使用 `server / logic / svc / pb` 结构。
- 多服务仓库按业务域拆分，再按 `api / rpc` 组织子服务。

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
│   │   ├── routes.go              # 路由注册（常见为脚手架生成）
│   │   └── loginhandler.go        # HTTP 处理器（常见为脚手架生成）
│   ├── logic/
│   │   └── loginlogic.go          # 业务逻辑（手动实现）
│   ├── middleware/
│   │   └── authmiddleware.go      # 中间件
│   ├── svc/
│   │   └── servicecontext.go      # 服务上下文
│   └── types/
│       └── types.go               # 请求/响应类型（常见为脚手架生成）
├── model/                         # 数据模型（可选，可能为 goctl 生成）
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

## 推荐实践

- 公共能力下沉到 `common/` 或独立模块时，优先抽稳定能力，不要把半成品业务代码抽成“公共库”。
- 历史项目可以逐步向标准目录靠拢，不必为了目录名一次性大搬家。
- 对生成代码目录加清晰注释或约定，避免误把脚手架产物当作长期手写入口。

## 禁止事项

- ❌ 在 `handler` / `server` 目录堆积业务编排代码
- ❌ 把生成代码和手写扩展混在同一文件反复改动
- ❌ 目录按个人习惯随意漂移，导致服务间结构完全不一致

## 审查清单

- [ ] API / RPC 服务目录职责清晰
- [ ] 生成代码与手写代码边界清楚
- [ ] 多服务仓库按业务域组织合理
- [ ] 历史项目结构调整具备渐进迁移路径
