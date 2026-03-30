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
| Python 代码检查 | `code-checker-python` |
| Python 代码修复 | `code-fixer-python` |
| Go 后端 | `go-rule` + `go-zero-rule` |
| Gin Web 后端 | `go-rule` + `gin-web-rule` |
| Go 代码检查 | `code-checker-go` |
| Go 代码修复 | `code-fixer-go` |
| Java 后端 | `java-rule` |
| Java 代码检查 | `code-checker-java` |
| Java 代码修复 | `code-fixer-java` |
| 前端开发 | `frontend-design` |
| 代码审查 | `code-review` |
| Git 操作 | `git-version-control` |
| API 设计 | `restful-api-design` |
| 安全开发 | `secure-development` |
| 主题配色 | `theme-factory` |

---

## Agents 加载规则 (代码审查)

当需要进行深度代码审查 (Code Review) 时，调用对应语言的 Agent：

| 场景 | Agent |
|------|--------|
| Python 代码审查 | `python-reviewer` |
| Go 代码审查 | `go-reviewer` |
| Java 代码审查 | `java-reviewer` |

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

## 核心规则路径索引

- **通用规则** (始终加载): 
  - [工程与安全基线](rules/common/baseline.md) (包含架构原则、测试要求与安全规范)
  - [协作与规范速查](rules/common/collaboration.md) (包含 Git 规范与 Checklists)
  - [新手引导](rules/common/newbie.md)
  
- **特定语言规则** (根据当前项目语言动态加载):
  - **Python**: [编码规范](rules/python/coding-standards.md) | [项目结构](rules/python/project-structure.md)
  - **Go**: [编码规范](rules/go/coding-standards.md) | [项目结构](rules/go/project-structure.md)
    - Gin Web 项目额外遵循 `gin-web-rule`
  - **Java**: [编码规范](rules/java/coding-standards.md) | [项目结构](rules/java/project-structure.md)
  - **前端**: [编码规范](rules/frontend/coding-standards.md) | [项目结构](rules/frontend/project-structure.md)

---

*最后更新：2026-03-30*
