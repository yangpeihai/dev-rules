# Java 后端项目结构 (Spring Boot 风格)

```
project/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com.example.project/
│   │   │       ├── controller/  # REST 控制器层
│   │   │       ├── service/     # 业务逻辑服务层 (接口)
│   │   │       │   └── impl/    # 服务层实现
│   │   │       ├── repository/  # 数据库访问层 (JPA/MyBatis Mapper)
│   │   │       ├── model/       # 数据模型层 (Entity, DTO, VO)
│   │   │       ├── config/      # 配置类 (Security, Swagger 等)
│   │   │       ├── exception/   # 全局异常处理及自定义异常
│   │   │       ├── util/        # 工具类
│   │   │       └── Application.java # Spring Boot 启动类
│   │   └── resources/
│   │       ├── application.yml  # 主配置文件
│   │       ├── application-dev.yml
│   │       ├── application-prod.yml
│   │       ├── mapper/          # MyBatis XML 映射文件
│   │       └── static/          # 静态资源 (通常不在此处放前端代码)
│   └── test/                    # 单元与集成测试
├── pom.xml                      # Maven 依赖管理 (或 build.gradle)
├── Dockerfile
├── .gitignore
└── README.md
```
