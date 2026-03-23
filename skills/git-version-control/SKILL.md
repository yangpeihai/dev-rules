---
name: "git-version-control"
description: "Git 版本控制规范指南。Cover Git branching strategies, commit message conventions (Conventional Commits), tag management, merge strategies, .gitignore configuration, and sensitive data protection. Invoke when: managing branches, writing commit messages, creating tags, resolving merge conflicts, setting up .gitignore, protecting sensitive files, writing technical documentation for Git workflows, or discussing version control best practices."
---

# Git 版本控制规范

## 概述

本规范定义了企业级软件开发中 Git 版本控制的标准，涵盖分支管理、提交规范、标签管理、合并策略、.gitignore 配置和敏感信息保护等内容。

## 核心规则速查

### 分支管理策略

| 策略类型 | 适用场景 | 特点 |
|---------|---------|------|
| Git Flow | 版本发布型项目 | 分支结构完整，适合有明确版本周期 |
| Trunk-Based | 持续交付型项目 | 主干开发，适合频繁发布的敏捷团队 |

### Git Flow 分支定义

| 分支名称 | 用途 | 生命周期 | 保护级别 |
|---------|------|---------|---------|
| `main` | 生产环境代码 | 永久 | 严格保护 |
| `develop` | 开发集成分支 | 永久 | 保护 |
| `feature/*` | 功能开发分支 | 临时 | 无保护 |
| `release/*` | 版本发布分支 | 临时 | 保护 |
| `hotfix/*` | 紧急修复 | 临时 | 保护 |

## 分支命名规范

格式：`<类型>/<标识>-<描述>`

| 类型前缀 | 用途 | 示例 |
|---------|------|---------|
| `feature/` | 新功能 | `feature/PROJ-1234-user-login` |
| `bugfix/` | Bug 修复 | `bugfix/PROJ-5678-fix-null-pointer` |
| `hotfix/` | 紧急修复 | `hotfix/PROJ-9999-critical-error` |
| `release/` | 版本发布 | `release/v2.3.0` |
| `docs/` | 文档更新 | `docs/api-documentation` |
| `refactor/` | 代码重构 | `refactor/payment-module` |

**规则**：必须使用小写字母，单词间用连字符 `-` 分隔。

## Commit Message 规范

格式：`type(scope): description`

### Type 类型定义

| Type | 用途 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat(auth): 添加 JWT 认证` |
| `fix` | Bug 修复 | `fix(api): 修复空指针异常` |
| `docs` | 文档更新 | `docs(readme): 更新部署说明` |
| `style` | 代码格式 | `style(lint): 修复 ESLint 警告` |
| `refactor` | 重构 | `refactor(db): 优化查询性能` |
| `perf` | 性能优化 | `perf(cache): 减少缓存查询时间` |
| `test` | 测试 | `test(unit): 添加用户服务测试` |
| `build` | 构建 | `build(webpack): 升级至 v5` |
| `ci` | CI/CD | `ci(github): 添加自动化部署` |
| `chore` | 杂项 | `chore(deps): 更新依赖包` |
| `revert` | 回滚 | `revert: 回滚 feat(auth)` |

### Description 规则

- 使用中文或英文，团队内统一
- 简洁明了，不超过 50 个字符
- 使用动词开头，描述做了什么
- 首字母小写（英文），不使用句号结尾

**正确示例**：
```bash
git commit -m "feat(auth): 添加用户登录功能"
git commit -m "fix(api): 修复订单查询超时问题"
git commit -m "refactor(db): 优化数据库连接池"
```

**错误示例**：
```bash
git commit -m "修改代码"                    # 过于笼统
git commit -m "fix"                         # 缺少 scope 和描述
git commit -m "FIX: BUG 修复"                 # type 大写
git commit -m "feat(auth): Added login."    # 英文使用过去式，有句号
```

## Tag 规范

语义化版本格式：`v<MAJOR>.<MINOR>.<PATCH>[-<prerelease>][+<build>]`

| 部分 | 说明 | 递增规则 |
|------|------|---------|
| MAJOR | 主版本号 | 不兼容的 API 变更时递增 |
| MINOR | 次版本号 | 向下兼容的功能新增时递增 |
| PATCH | 修订号 | 向下兼容的问题修复时递增 |

**正确示例**：
```bash
git tag -a v1.0.0 -m "首个正式版本"
git tag -a v1.0.0-alpha.1 -m "内测版本 1"
git tag -a v1.0.0-beta.2 -m "公测版本 2"
```

**错误示例**：
```bash
git tag v1.0           # 缺少 PATCH 版本
git tag version1.0.0   # 缺少 v 前缀
git tag V1.0.0         # 大写 V
```

## 合并策略

| 场景 | 推荐策略 | 说明 |
|------|---------|------|
| feature → develop | Squash and Merge | 保持 develop 历史整洁 |
| release → main | Merge Commit | 保留版本发布历史 |
| hotfix → main | Merge Commit | 保留紧急修复记录 |

**禁止**：使用 Fast-forward 合并到 main 分支。

## .gitignore 规范

### 通用忽略规则

```gitignore
## 操作系统文件
.DS_Store
Thumbs.db
desktop.ini

## 编辑器/IDE
.idea/
.vscode/
*.swp
*.swo
*~
*.iml

## 日志文件
*.log
logs/
log/

## 临时文件
tmp/
temp/
*.tmp
*.temp

## 编译输出
dist/
build/
target/
out/

## 依赖目录
node_modules/
vendor/
```

### 敏感文件保护

```gitignore
## 配置文件（包含敏感信息）
.env
.env.local
.env.*.local
config.local.yaml
config.prod.yaml
secrets.yaml

## 密钥文件
*.pem
*.key
*.crt
*.p12
*.pfx
id_rsa
id_rsa.pub
.ssh/

## 数据库文件
*.db
*.sqlite
*.sqlite3
```

## 敏感信息保护

### 禁止提交的内容

| 类型 | 示例 |
|------|------|
| 密码和密钥 | 数据库密码、API 密钥、私钥、Token |
| 配置文件 | 生产环境配置、内部网络配置 |
| 个人信息 | 员工信息、客户数据、身份证号 |
| 商业机密 | 算法实现、业务逻辑、财务数据 |
| 凭证文件 | .env 文件、证书文件、SSH 密钥 |

### 发现敏感信息已提交的处理

1. 使用 BFG Repo-Cleaner 或 git-filter-repo 清除
2. 立即轮换所有泄露的密码和密钥
3. 使用 git-secrets 或 pre-commit 钩子预防

## Rebase vs Merge 策略

| 场景 | 推荐策略 | 说明 |
|------|---------|------|
| 本地 feature 分支更新 | Rebase | 保持提交历史线性整洁 |
| 多人协作的 feature 分支 | Merge | 保留协作历史 |
| 合并到 develop/main | Squash Merge | 保持主干历史整洁 |
| 已推送的公共分支 | 禁止 Rebase | 避免重写公共历史 |

## Git LFS 使用规范

超过 100MB 的文件必须使用 Git LFS 管理：

```bash
# 安装
git lfs install

# 追踪大文件
git lfs track "*.psd"
git lfs track "*.mp4"
git lfs track "*.zip"

# 提交.gitattributes
git add .gitattributes
git commit -m "chore(git): 配置 Git LFS 追踪大文件"
```

**必须纳入 LFS 的文件类型**：多媒体文件 (.psd, .ai, .mp4, .mov)、大型二进制、数据集等。

## 快速检查清单

### 提交前检查
- [ ] 分支命名符合规范
- [ ] Commit Message 符合 Conventional Commits
- [ ] 无敏感信息提交
- [ ] 大文件已使用 LFS
- [ ] .gitignore 配置正确

### 合并前检查
- [ ] 已通过 Code Review（至少 2 人）
- [ ] CI/CD 检查通过
- [ ] 无冲突或冲突已正确解决
- [ ] 目标分支正确

### 发布前检查
- [ ] 版本标签符合 SemVer
- [ ] 发布说明完整
- [ ] 已合并到 main 分支
- [ ] 标签已推送
