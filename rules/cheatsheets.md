# 速查表

## 工具链对比

| 规范 | Python | Go | Java |
|------|--------|-----|------|
| **格式化** | black, ruff | gofmt, goimports | Checkstyle, Spotless |
| **Lint** | ruff, pylint | golangci-lint | SonarQube, PMD, SpotBugs |
| **类型检查** | mypy | go vet | javac, SpotBugs |
| **测试框架** | pytest | testing | JUnit 5 |
| **密码哈希** | bcrypt, argon2 | bcrypt | BCryptPasswordEncoder |
| **日志框架** | logging | log/slog | SLF4J + Logback |
| **依赖管理** | pip, poetry | go mod | Maven, Gradle |
| **Web 框架** | FastAPI, Flask | gin, go-zero | Spring Boot |

---

## Git 常用命令

```bash
# 查看状态
git status

# 添加文件
git add .

# 提交
git commit -m "feat(auth): 添加用户登录功能"

# 推送
git push origin feature/PROJ-1234-user-login

# 拉取最新代码
git pull origin main

# 创建分支
git checkout -b feature/PROJ-1234-user-login

# 合并分支
git merge feature/PROJ-1234-user-login

# 创建标签
git tag -a v1.0.0 -m "首个正式版本"

# 推送标签
git push origin v1.0.0

# Rebase
git rebase main

# 查看日志
git log --oneline --graph
```

---

## 分支命名规范

格式：`<类型>/<标识>-<描述>`

示例：
- `feature/PROJ-1234-user-login`
- `bugfix/ISSUE-567-fix-null-pointer`
- `hotfix/security-patch`
- `release/v1.2.0`

---

## Commit Message 规范

格式：`<type>(<scope>): <description>`

**Type 类型**：
- `feat` - 新功能
- `fix` - Bug 修复
- `docs` - 文档更新
- `style` - 代码格式（不影响代码运行）
- `refactor` - 重构
- `test` - 测试相关
- `chore` - 构建/工具/配置

示例：
- `feat(auth): 添加用户登录功能`
- `fix(api): 修复空指针异常`
- `docs(readme): 更新安装说明`

---

## 参考资料

**官方文档**：
- Python: [PEP 8](https://peps.python.org/pep-0008/), [PEP 484](https://peps.python.org/pep-0484/)
- Go: [Effective Go](https://go.dev/doc/effective_go)
- Java: [阿里巴巴 Java 开发手册](https://github.com/alibaba/p3c)
- RESTful API: [REST API Design](https://restfulapi.net/)

**安全资源**：
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [CWE](https://cwe.mitre.org/)
