# AI Code 全局编码约束配置

> 适用于 Claude Code、Trae 等 AI 编程助手
>
> 详细规则：[rules/](rules/) 文件夹
>
> **新手用户**：请阅读 [rules/newbie.md](rules/newbie.md) 了解默认技术栈和选型规则

---

## 技术栈偏好

| 层级 | 选择 |
|------|------|
| **后端** | Python 3.10+ (FastAPI/Flask) |
| **前端** | React 18+ (TypeScript + Vite) |
| **数据库** | SQLite |
| **测试** | pytest / Jest |

---

## Skills 加载规则

| 场景 | Skills |
|------|--------|
| Python 后端 | `python-rule` |
| Go 后端 | `go-rule` + `go-zero-rule` |
| Java 后端 | `java-rule` |
| 前端开发 | `frontend-design` |
| 代码审查 | `code-review` |
| Git 操作 | `git-version-control` |
| API 设计 | `restful-api-design` |
| 安全开发 | `secure-development` |
| 主题配色 | `theme-factory` |

---

## 核心约束

**代码风格**：
- 缩进：4 空格 (Python/Go) / 2 空格 (JS/TS)
- 命名：snake_case (Python/Go) / camelCase (JS/TS)
- 必须使用类型注解和文档字符串
- 禁止魔法值、无意义变量名

**安全基线**：
- 所有输入必须验证，数据库查询必须参数化
- 禁止硬编码密钥、禁止记录敏感信息
- 密码使用 bcrypt/argon2 哈希

**测试要求**：
- 新功能必须有单元测试
- 覆盖率目标：≥80%

---

## 配置覆盖规则

```
用户显式指令 > 项目级配置 > 本全局配置 > AI 默认行为
```

---

## Rules 文件索引

| 文件 | 内容 |
|------|------|
| [coding-standards.md](rules/coding-standards.md) | 各语言编码标准详情 |
| [project-structure.md](rules/project-structure.md) | 项目结构规范 |
| [security-baseline.md](rules/security-baseline.md) | 安全开发基线 |
| [checklists.md](rules/checklists.md) | 提交/发布检查清单 |
| [cheatsheets.md](rules/cheatsheets.md) | 工具链/Git 速查表 |
| [newbie.md](rules/newbie.md) | 新手用户约束配置（技术栈选型/数据存储优先级） |

---

*最后更新：2026-03-19*
