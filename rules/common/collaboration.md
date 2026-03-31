# 协作与规范速查 (Cheatsheet & Checklists)

> 提供 Git 协作、Code Review 和发布的快速参考标准。

## 1. Git 协作规范

### 分支管理
- **格式**：`<类型>/<标识>-<简短描述>`
- **示例**：
  - `feature/PROJ-123-user-login` (新功能)
  - `bugfix/ISSUE-456-fix-npe` (普通修复)
  - `hotfix/security-patch` (紧急线上修复)

### 提交信息 (Commit Message)
- **格式**：`<type>(<scope>): <description>`
- **Type 列表**：
  - `feat`: 新功能
  - `fix`: Bug 修复
  - `docs`: 文档修改
  - `style`: 格式化/空格 (不影响逻辑)
  - `refactor`: 重构 (无新功能且无 bug 修复)
  - `test`: 增删测试代码
  - `chore`: 构建/工具/配置变动

---

## 2. 检查清单 (Checklists)

### 🚀 提交/推送前自检 (Pre-commit)
- [ ] 代码在本地编译/构建无报错
- [ ] 单元测试在本地执行全部通过
- [ ] Linter/格式化工具未报告警告
- [ ] 已移除所有硬编码的敏感信息和测试用的 `print/console.log`
- [ ] 提交信息符合 Conventional Commits 规范

### 🔍 Code Review 核心关注点
Reviewer 需从以下维度检查：
1. **正确性**：是否满足需求？边界条件是否处理？
2. **安全性**：是否存在注入、越权、并发竞态等风险？
3. **可维护性**：命名是否清晰？是否有重复代码？函数是否过长？
4. **性能**：是否存在慢查询、循环内调用外部接口、内存泄漏？

*注：评论请带上前缀明确意图，如 `[MUST]`(必须改), `[SHOULD]`(建议改), `[NICE]`(优化项), `[Q]`(疑问)*

### 📦 发布前检查 (Pre-release)
- [ ] CI/CD 流水线 (包含安全扫描) 绿色通过
- [ ] 代码合并至少包含 2 个 Approve，无未解决的 `[MUST]` 评论
- [ ] 相关的配置文件 (如 `.env.example`, 字典表) 已同步更新
- [ ] 数据库变更脚本 (Migration) 已就绪且通过验证
- [ ] 降级或回滚方案已确认
