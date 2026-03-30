# Go 后端项目结构 (Standard Go Project Layout)

## 通用 Go 项目布局

```text
project/
├── cmd/                 # 应用入口点，主要存放 main.go
├── internal/            # 私有应用和库代码
│   ├── handler/         # HTTP 路由与处理器
│   ├── service/         # 业务逻辑服务层
│   ├── repository/      # 数据访问层
│   ├── model/           # 数据模型定义
│   └── middleware/      # 中间件
├── pkg/                 # 可被外部应用引用的公共库代码
├── api/                 # OpenAPI/Swagger 规范, JSON schema 协议等
├── configs/             # 配置文件模板或默认配置
├── scripts/             # 构建、安装、分析等操作的脚本
├── deployments/         # IaaS, PaaS, 系统和容器编排配置和模板
├── test/                # 额外的外部测试应用和测试数据
├── docs/                # 设计和用户文档
├── go.mod               # 模块依赖文件
├── go.sum               # 模块校验和
├── Dockerfile
├── .gitignore
└── README.md
```

## 框架补充说明

- Gin Web 项目：目录分层、响应规范、中间件顺序等以 `gin-web-rule` 为准
- go-zero 项目：服务结构、分层职责、生成代码约束等以 `go-zero-rule` 为准
- 本文件只保留通用 Go 项目布局，避免与 skill 内容重复
