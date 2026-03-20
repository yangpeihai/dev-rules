# 项目结构规范

## 后端项目结构（Python/Go 风格）

```
project/
├── api/                 # API 定义
├── cmd/                 # 应用入口
├── internal/            # 私有应用代码
│   ├── handler/         # HTTP 处理器
│   ├── service/         # 业务逻辑
│   ├── repository/      # 数据访问层
│   ├── model/           # 数据模型
│   └── middleware/      # 中间件
├── pkg/                 # 公共库（可被外部引用）
├── config/              # 配置文件
├── scripts/             # 脚本
├── test/                # 测试
├── docs/                # 文档
├── .env.example         # 环境变量示例
├── .gitignore           # Git 忽略文件
├── Dockerfile           # Docker 镜像定义
├── README.md            # 项目说明
└── requirements.txt / go.mod / pom.xml
```

---

## 前端项目结构（React 风格）

```
project/
├── src/
│   ├── assets/          # 静态资源
│   ├── components/      # 公共组件
│   ├── pages/           # 页面组件
│   ├── hooks/           # 自定义 Hooks
│   ├── services/        # API 服务
│   ├── store/           # 状态管理
│   ├── utils/           # 工具函数
│   ├── types/           # TypeScript 类型
│   ├── App.tsx          # 根组件
│   └── main.tsx         # 入口文件
├── public/              # 公共资源
├── package.json
├── tsconfig.json
├── vite.config.ts
├── .eslintrc.json
├── .prettierrc
└── README.md
```
