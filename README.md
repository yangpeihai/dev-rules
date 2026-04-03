# dev-rules

AI 编码规则与技能配置仓库，面向 Claude Code、Trae 等 AI 编程助手，提供统一的项目级约束、语言规范、技能入口与代码审查代理配置。

## 项目定位

- 用 `CLAUDE.md` 作为项目主入口，集中声明默认技术栈、技能加载规则、Agent 加载规则和核心约束。
- 用 `rules/` 维护可复用的通用规则与多语言规范。
- 用 `skills/` 提供按场景加载的技能说明，覆盖开发、检查、修复、测试、安全、Git、API 设计等常见任务。
- 用 `agents/` 提供按语言划分的代码审查代理配置。

## 快速开始

### 1. 作为项目规则使用

将仓库放到目标项目中，并让 AI 助手读取根目录的 `CLAUDE.md`。

### 2. 选择对应规则

- 通用规则：优先阅读 `rules/common/`
- Python 项目：补充 `rules/python/`
- Go 项目：补充 `rules/go/`
- Java 项目：补充 `rules/java/`
- 前端项目：补充 `rules/frontend/`

### 3. 按场景加载技能

根据任务类型加载对应 skill，例如：

- Python 后端：`python-rule`
- Go 后端：`go-rule`
- Java 后端：`java-rule`
- Gin Web：`go-rule` + `gin-web-rule`
- Go-Zero：`go-rule` + `go-zero-rule`
- 代码检查：`code-checker-*`
- 代码修复：`code-fixer-*`
- HTTP API 测试：`http-api-tester`
- 代码审查：`code-review`

推荐把语言 skill、代码检查、代码修复和 reviewer agent 视为一条闭环：

- Python：`python-rule` → `code-checker-python` → `code-fixer-python` → `python-reviewer`
- Go：`go-rule` → `code-checker-go` → `code-fixer-go` → `go-reviewer`
- Java：`java-rule` → `code-checker-java` → `code-fixer-java` → `java-reviewer`

## 默认技术栈偏好

| 层级 | 默认选择 |
|------|----------|
| 后端 | Python 3.10+（FastAPI / Flask） |
| 前端 | React 18+（TypeScript + Vite） |
| 数据库 | SQLite |
| 测试 | pytest / Jest |

## 目录结构

以下目录结构仅展示当前已纳入版本控制的主要内容，不包含 `.gitignore` 中忽略的数据与工作目录。

```text
dev-rules/
├── LICENSE
├── CLAUDE.md
├── README.md
├── agents/
│   ├── go-reviewer.md
│   ├── java-reviewer.md
│   └── python-reviewer.md
├── rules/
│   ├── common/
│   │   ├── baseline.md
│   │   ├── collaboration.md
│   │   └── newbie.md
│   ├── frontend/
│   │   ├── coding-standards.md
│   │   └── project-structure.md
│   ├── go/
│   │   ├── coding-standards.md
│   │   └── project-structure.md
│   ├── java/
│   │   ├── coding-standards.md
│   │   └── project-structure.md
│   └── python/
│       ├── coding-standards.md
│       └── project-structure.md
└── skills/
    ├── code-checker-{go,java,python}/
    ├── code-fixer-{go,java,python}/
    ├── code-review/
    ├── frontend-design/
    ├── gin-web-rule/
    ├── git-version-control/
    ├── http-api-tester/
    ├── go-rule/
    ├── go-zero-rule/
    ├── java-rule/
    ├── python-rule/
    ├── restful-api-design/
    ├── secure-development/
    └── theme-factory/
```

## Rules 索引

### 通用规则

- `rules/common/baseline.md`：工程与安全基线
- `rules/common/collaboration.md`：协作规范、Git 约束与检查清单
- `rules/common/newbie.md`：新手引导与默认技术选型

### 语言规则

- Python：`rules/python/coding-standards.md`、`rules/python/project-structure.md`
- Go：`rules/go/coding-standards.md`、`rules/go/project-structure.md`
- Java：`rules/java/coding-standards.md`、`rules/java/project-structure.md`
- Frontend：`rules/frontend/coding-standards.md`、`rules/frontend/project-structure.md`

## Skills 索引

| 场景 | Skill |
|------|-------|
| Python 后端开发 | `python-rule` |
| Python 代码检查 | `code-checker-python` |
| Python 代码修复 | `code-fixer-python` |
| Go 后端开发 | `go-rule` |
| Gin Web 开发 | `go-rule` + `gin-web-rule` |
| Go-Zero 开发 | `go-rule` + `go-zero-rule` |
| Go 代码检查 | `code-checker-go` |
| Go 代码修复 | `code-fixer-go` |
| Java 后端开发 | `java-rule` |
| Java 代码检查 | `code-checker-java` |
| Java 代码修复 | `code-fixer-java` |
| HTTP API 测试 | `http-api-tester` |
| 前端开发 | `frontend-design` |
| 代码审查 | `code-review` |
| Git 操作 | `git-version-control` |
| API 设计 | `restful-api-design` |
| 安全开发 | `secure-development` |
| 主题配色 | `theme-factory` |

## Agents 索引

当需要做深度代码审查时，可按语言选择对应 Agent：

| 场景 | Agent |
|------|-------|
| Python 代码审查 | `python-reviewer` |
| Go 代码审查 | `go-reviewer` |
| Java 代码审查 | `java-reviewer` |

当前推荐的语言闭环：

- Python：`python-rule` → `code-checker-python` → `code-fixer-python` → `python-reviewer`
- Go：`go-rule` → `code-checker-go` → `code-fixer-go` → `go-reviewer`
- Java：`java-rule` → `code-checker-java` → `code-fixer-java` → `java-reviewer`

## 核心约束

### 代码风格

- 缩进：4 空格（Python / Go），2 空格（JS / TS）
- 命名：snake_case（Python / Go），camelCase（JS / TS）
- 公共 API 优先提供清晰类型信息和必要文档字符串
- 禁止魔法值、禁止无意义变量名

### 安全基线

- 所有输入必须验证
- 数据库查询必须参数化
- 禁止硬编码密钥
- 禁止记录敏感信息
- 密码必须使用 bcrypt 或 argon2 哈希

### 测试要求

- 新功能必须有单元测试
- 优先覆盖关键路径、错误分支和回归风险点
- 不机械追求固定覆盖率数字

## 配置优先级

```text
用户显式指令 > 项目级配置 > 本仓库默认配置 > AI 默认行为
```

其中项目级配置主入口为 `CLAUDE.md`。

## 维护建议

- 更新 `CLAUDE.md` 时，同步检查 README 中的规则索引和技能映射。
- 新增 skill 或 agent 后，补充到 README 的目录结构和索引表。
- 规则目录调整后，优先修正 README 中的路径示例，避免失效引用。
- 更新目录结构时，仅统计当前已跟踪文件，忽略 `.gitignore` 覆盖的本地数据与产物目录。

## 许可证

本仓库默认使用 `MIT License`，完整文本见根目录 [LICENSE](LICENSE)。

需要注意：

- 部分子目录包含各自的第三方许可证文件，例如 `skills/frontend-design/LICENSE.txt`、`skills/theme-factory/LICENSE.txt`
- 这些许可证仅适用于对应子目录或其上游来源，不影响仓库根目录采用 MIT 作为默认许可证

---

*最后更新：2026-04-03*
