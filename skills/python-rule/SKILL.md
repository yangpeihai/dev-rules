---
name: python-rule
description: "Python 编码规范指南。Cover Python code style, typing, exception handling, security, performance, async/concurrency choices, project structure, testing, and API design best practices. Invoke when: writing/reviewing Python code, designing modules or data models, handling async tasks, refactoring legacy code, configuring Python projects, debugging issues, writing tests, or documenting Python architecture decisions."
---

# Python 编码规范助手

帮助开发者编写更符合 Python 语言习惯、真实工程实践和 AI 可执行风格的代码，兼顾可读性、正确性、安全性、可靠性与维护成本。

## AI 触发条件

当任务满足以下任一条件时，应优先使用本规范：
- 编写新的 Python 代码或修改现有代码
- 设计模块、类、函数、数据模型或公共 API
- 处理异步编程、并发或 I/O
- 进行代码审查
- 优化性能或排查可靠性问题
- 处理安全敏感代码（输入验证、密码、文件、序列化、外部调用）
- 组织项目结构、依赖和配置
- 编写测试代码或技术文档

## AI 执行优先级

按以下顺序决策，避免只凭经验自由发挥：

1. 先判断任务类型：实现、重构、调试、优化、审查、文档。
2. 再判断核心问题属于：可读性、设计边界、异常处理、安全、性能、测试。
3. 涉及模块边界、类设计、依赖注入、异常模型时，优先参考 `core-principles-and-design.md`。
4. 涉及命名、格式化、类型注解、文档字符串、函数签名时，优先参考 `style-and-readability.md`。
5. 涉及异步、并发、I/O、资源管理、性能瓶颈、测试策略时，优先参考 `performance-and-reliability.md`。
6. 涉及输入验证、SQL、文件路径、密码、密钥、反序列化、日志脱敏时，优先参考 `security.md`。

## AI 默认执行步骤

1. 识别本次任务主目标是“可读性”“正确性”“安全性”“可靠性”还是“性能”。
2. 只读取当前问题直接相关的 references，不要整套全读。
3. 默认优先选择最简单、最显式、最容易维护的 Python 写法。
4. 公共 API 默认补齐合适的类型注解；对动态边界或历史代码不做机械补全。
5. 异常处理默认只捕获能处理的异常；新增上下文时再包装或转换异常。
6. 并发默认按场景选择：I/O 密集看 `asyncio` 或线程池，CPU 密集看进程池，简单共享状态优先保持清晰和正确。
7. 输出前自检命名、异常、资源释放、超时设置、输入校验和关键路径测试。

## 快速参考

### 常用命令

```bash
# 格式化与静态检查
black .
ruff check . --fix

# 类型检查
mypy .

# 运行测试
pytest
```

### 常用模式

```python
from pathlib import Path


def load_text(path: Path) -> str:
    return path.read_text(encoding="utf-8")


def create_user(*, name: str, email: str) -> dict[str, str]:
    return {"name": name, "email": email}


try:
    result = call_external_api(timeout=10)
except NetworkError as exc:
    raise ServiceUnavailableError("外部服务暂不可用") from exc
```

---

## 文档索引

- 主 skill：用于快速判断任务类型、执行优先级、最小工作流和输出方式。
- `core-principles-and-design.md`：用于模块边界、类设计、依赖组织、异常模型、配置方案。
- `style-and-readability.md`：用于命名、格式化、类型注解、函数签名、导入和测试可读性。
- `performance-and-reliability.md`：用于并发选型、I/O、资源管理、可靠性和测试重点。
- `security.md`：用于输入校验、SQL、路径、密码、密钥、序列化和日志脱敏。

## 详细规范索引

本规范按主题拆分为多个文档，根据你的具体需求参考相应章节：

- 结构化规范文档：更适合做日常开发、设计取舍和代码审查。
- 主题参考文档：更适合按问题类型快速查约定、示例和注意事项。

| 主题 | 参考文档 | 使用场景 |
|------|---------|----------|
| 代码风格与可读性 | `style-and-readability.md` | 命名、格式化、注释、类型注解、函数设计、测试可读性 |
| 安全编码 | `security.md` | 输入验证、SQL、密码、文件、序列化、日志脱敏 |
| 性能与可靠性 | `performance-and-reliability.md` | 并发选型、I/O、资源管理、热点优化、测试策略 |
| 核心原则与设计 | `core-principles-and-design.md` | 模块设计、类设计、依赖注入、异常模型、配置组织 |

### 何时查看哪个文档

- **日常编码** → `style-and-readability.md`
- **处理敏感数据** → `security.md`
- **性能优化或可靠性治理** → `performance-and-reliability.md`
- **架构设计与模块边界** → `core-principles-and-design.md`
- **设计异常与配置策略** → `core-principles-and-design.md` + `style-and-readability.md`
- **整理并发、I/O 与测试策略** → `performance-and-reliability.md` + `security.md`

### 文档使用建议

- 需要做设计取舍时，优先看 `core-principles-and-design.md`。
- 需要查编码细节、命名和函数写法时，优先看 `style-and-readability.md`。
- 需要判断可靠性或安全风险时，结合 `performance-and-reliability.md` 和 `security.md` 一起看。

## AI 执行入口

- 遇到 Python 实现任务：先判断是设计、风格、可靠性还是安全问题，再只读最相关的 1-2 份 reference。
- 遇到 Python 审查任务：先从异常处理、资源释放、安全缺口和测试风险切入，再补充一致性问题。
- 遇到 Python 文档任务：优先写成可执行、少歧义、适合多轮会话重复使用的规则。
- 如果项目已有明确框架或团队约定：优先保持项目一致性，不强行套通用理想写法。

## 当前推荐的下一步

- 如果要继续完善 `python-rule`，下一步适合补充更明确的框架例外说明，例如 FastAPI、Django、脚本型项目的差异化入口。
- 如果要继续做 AI 工具化，下一步适合为关键 reference 补充更短的“实现/审查/文档”局部最小工作流。

## AI 默认输出要求

- 实现任务：优先直接给出代码修改，再简短说明采用的 Python 惯用法。
- 审查任务：先报问题，再给总结；优先异常处理、资源泄漏、并发风险、安全缺口、测试空洞。
- 优化任务：先判断是否有真实瓶颈证据，再决定是否引入更复杂方案。
- 文档任务：优先输出可执行、少歧义、适合 AI 连续使用的规则表述。

## AI 例外条件

- 项目已有稳定风格且与通用 Python 惯用法略有差异时，优先保持项目一致性。
- 动态元编程、框架魔法或历史代码场景下，不机械追求完整类型注解。
- 未有 profiling、压测或明确瓶颈证据时，不主动引入复杂缓存、并发或微优化。
- 内部脚本、一次性工具可适当放宽公共库级别的文档和兼容性要求，但不能牺牲基本可读性与安全性。

## AI 最小工作流

### 实现任务

1. 先判断任务重点是设计、可读性、安全、可靠性还是性能。
2. 读取最相关的 1-2 份 reference，不整套通读。
3. 按 Python 惯用法实现，优先简单、显式、易测。
4. 自检异常处理、资源管理、输入校验、超时和命名。
5. 输出修改结果，并简短说明关键取舍。

### 审查任务

1. 先看异常处理、资源泄漏、安全风险和接口设计。
2. 再看可读性、测试缺口和不必要复杂度。
3. 按严重程度列出问题，优先真实行为风险而非风格噪音。
4. 最后给简短总结和残余风险。

### 文档任务

1. 先判断文档面向 AI 执行还是人类阅读。
2. 优先写可判定、可执行、少歧义的规则。
3. 用短句和清单组织内容，减少背景铺垫。
4. 保持示例与当前规则一致。

---

## 核心原则

### Python 的设计哲学

| 原则 | 说明 |
|------|------|
| 可读性 | 代码首先是写给人看的 |
| 显式 | 显式优于隐式，边界和异常要清楚 |
| 简洁性 | 简单优于复杂，但不是机械压缩 |
| 实用性 | 以真实工程收益为准，不为规则而规则 |

### 接口与设计

```python
from dataclasses import dataclass
from typing import Protocol


class Sender(Protocol):
    def send(self, payload: bytes) -> None: ...


@dataclass
class Message:
    topic: str
    body: bytes


class Publisher:
    def __init__(self, sender: Sender) -> None:
        self._sender = sender
```

### 异常处理

```python
try:
    config = load_config(path)
except FileNotFoundError:
    return default_config()
except OSError as exc:
    raise ConfigError(f"读取配置失败: {path}") from exc
```

### 并发原则

```python
# I/O 密集型优先考虑 asyncio
async def fetch_all(urls: list[str]) -> list[str]:
    ...


# 简单阻塞型 I/O 也可以用线程池
from concurrent.futures import ThreadPoolExecutor


with ThreadPoolExecutor(max_workers=8) as executor:
    results = list(executor.map(read_one, paths))
```

---

## 禁止事项

- ❌ 使用裸 `except`
- ❌ 忽略外部调用超时
- ❌ SQL 使用字符串拼接
- ❌ 密码使用明文、可逆加密或弱哈希
- ❌ 记录敏感信息到日志
- ❌ 无保护地共享可变状态
- ❌ 对不可信输入使用 `pickle.loads`、`eval`、`exec`
- ❌ 使用可变对象作为默认参数
- ❌ 在没有证据的前提下做复杂优化

---

## 推荐实践

1. **公共 API 适度类型化** - 对外接口优先提供清晰类型信息。
2. **选择合适的数据模型** - 数据容器优先考虑 `dataclass`、`TypedDict`、Pydantic 等合适方案。
3. **异常只捕获能处理的部分** - 需要新增上下文时再转换或包装异常。
4. **组合优于继承** - 优先做清晰协作，而不是构建深继承树。
5. **并发按场景选择** - `asyncio`、线程池、进程池按任务类型决策。
6. **资源显式管理** - 文件、连接、会话优先用上下文管理器。
7. **超时与重试分开考虑** - 先设置超时，再决定是否值得重试。
8. **安全默认前置** - 输入验证、参数化查询、路径约束、日志脱敏。
9. **测试聚焦关键路径** - 优先覆盖错误分支、边界条件和回归风险点。
10. **保持项目一致性** - 在不牺牲质量的前提下遵循现有工程约定。

---

## 代码审查清单

提交代码前，请确认：

- [ ] 代码已通过 `black` 和 `ruff` 基本整理
- [ ] 公共 API 在需要时提供了清晰类型注解
- [ ] 异常处理明确，没有裸 `except`
- [ ] 外部调用（HTTP、数据库、RPC）设置了合理超时
- [ ] 使用参数化查询防止 SQL 注入
- [ ] 密码使用成熟 KDF/哈希方案（如 `bcrypt`、`argon2`）
- [ ] 敏感信息已脱敏或从环境变量/配置安全读取
- [ ] 文件、连接、锁等资源能正确释放
- [ ] 并发访问共享状态时有明确同步策略
- [ ] 没有可变默认参数
- [ ] 函数职责清晰；当参数列表影响可读性时，使用配置对象、数据类或关键字参数
- [ ] 关键路径、错误分支和回归风险点已有测试覆盖

---

## 参考资料

- [PEP 8 - Style Guide for Python Code](https://peps.python.org/pep-0008/)
- [PEP 484 - Type Hints](https://peps.python.org/pep-0484/)
- [The Python Tutorial](https://docs.python.org/3/tutorial/)
- [Python Standard Library](https://docs.python.org/3/library/)

完整规范文档请查看 `references/` 目录下的各个主题文档。
