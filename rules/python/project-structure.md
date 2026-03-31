# Python 后端项目结构 (FastAPI/Flask 风格)

```
project/
├── api/                 # API 路由与控制器定义
├── core/                # 核心配置 (依赖注入, 异常处理等)
├── models/              # 数据模型 (ORM/Pydantic)
├── schemas/             # 请求响应格式 (Pydantic models)
├── services/            # 业务逻辑服务层
├── repositories/        # 数据库交互封装层
├── utils/               # 工具函数
├── tests/               # 单元与集成测试
├── config/              # 配置文件与环境变量加载
├── scripts/             # 开发/部署相关脚本
├── docs/                # API 及项目文档
├── main.py              # 应用入口
├── requirements.txt     # 或 pyproject.toml
├── Dockerfile
├── .env.example
└── README.md
```
