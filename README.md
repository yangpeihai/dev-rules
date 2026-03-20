# dev-rules

AI 驱动开发规则配置仓库，为 Claude Code、Trae 等 AI 编程助手提供统一的编码规范和技能配置。

## 快速开始

### 使用方式

1. **Claude Code 用户**：将此仓库复制到项目根目录，或配置为全局规则
2. **其他 AI 工具**：参考 `rules/` 和 `skills/` 目录结构配置类似规则

### 技术栈偏好

| 层级 | 选择 |
|------|------|
| **后端** | Python 3.10+ (FastAPI/Flask) |
| **前端** | React 18+ (TypeScript + Vite) |
| **数据库** | SQLite |
| **测试** | pytest / Jest |

## 目录结构

```
dev-rules/
├── CLAUDE.md           # 全局配置主入口
├── README.md           # 项目说明文档
├── rules/              # 规则文档
│   ├── coding-standards.md      # 编码标准详情
│   ├── project-structure.md     # 项目结构规范
│   ├── security-baseline.md     # 安全开发基线
│   ├── checklists.md            # 提交/发布检查清单
│   ├── cheatsheets.md           # 工具链/Git 速查表
│   └── newbie.md                # 新手用户约束配置
└── skills/             # AI 技能配置
    ├── python-rule/           # Python 开发技能
    ├── go-rule/               # Go 开发技能
    ├── go-zero-rule/          # go-zero 框架技能
    ├── java-rule/             # Java 开发技能
    ├── frontend-design/       # 前端设计技能
    ├── code-review/           # 代码审查技能
    ├── git-version-control/   # Git 操作技能
    ├── restful-api-design/    # RESTful API 设计技能
    ├── secure-development/    # 安全开发技能
    └── theme-factory/         # 主题配色技能
```

## 核心规则

### 代码风格

- **缩进**：4 空格 (Python/Go) / 2 空格 (JS/TS)
- **命名**：snake_case (Python/Go) / camelCase (JS/TS)
- **类型注解**：必须使用（Python/TS/Go）
- **文档字符串**：公共 API 必须有文档

### 安全基线

- 所有输入必须验证
- 数据库查询必须参数化
- 禁止硬编码密钥
- 密码使用 bcrypt/argon2 哈希

### 测试要求

- 新功能必须有单元测试
- 覆盖率目标：≥80%
- 遵循 AAA 模式（Arrange-Act-Assert）

## Skills 使用

在 Claude Code 中，使用 `/skill-name` 命令加载对应技能：

| 场景 | 命令 |
|------|------|
| Python 后端 | `/python-rule` |
| Go 后端 | `/go-rule` + `/go-zero-rule` |
| Java 后端 | `/java-rule` |
| 前端开发 | `/frontend-design` |
| 代码审查 | `/code-review` |
| Git 操作 | `/git-version-control` |
| API 设计 | `/restful-api-design` |
| 安全开发 | `/secure-development` |
| 主题配色 | `/theme-factory` |

## 配置覆盖规则

```
用户显式指令 > 项目级配置 (CLAUDE.md) > 本全局配置 > AI 默认行为
```

## 项目规范详情

详见 [rules/](rules/) 目录：

- [coding-standards.md](rules/coding-standards.md) - 各语言编码标准
- [project-structure.md](rules/project-structure.md) - 项目目录结构规范
- [security-baseline.md](rules/security-baseline.md) - 安全开发要求
- [checklists.md](rules/checklists.md) - 开发流程检查清单
- [cheatsheets.md](rules/cheatsheets.md) - 常用工具和 Git 速查表

## 许可证

MIT License

---

*最后更新：2026-03-20*
