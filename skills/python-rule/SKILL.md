---
name: python-rule
description: Python 编码规范指南。Cover Python code style (PEP 8), type hints, security patterns, performance optimization, API design, and testing best practices. Invoke when: writing/reviewing Python code, designing APIs or data models, configuring Python projects (pyproject.toml, requirements.txt), debugging issues, refactoring legacy code, setting up CI/CD pipelines, writing technical documentation, or discussing Python architecture decisions.
---

# Python 编码规范助手

帮助开发者编写符合 Python 语言习惯的代码，确保代码质量、安全性和性能。

## 何时使用

当你进行以下操作时，应该参考本规范：
- 编写新的 Python 代码或修改现有代码
- 设计接口、类、函数和数据模型
- 处理异步编程和并发
- 进行代码审查
- 优化性能或排查性能问题
- 处理安全敏感代码（密码、加密、输入验证）
- 组织项目结构和包设计
- 编写测试代码

## 快速参考

### 代码格式化

```bash
# 格式化代码
black .
ruff check . --fix

# 类型检查
mypy .

# 运行测试
pytest
```

### 常用模式

```python
# 上下文管理器
with open(path) as f:
    data = f.read()

# 类型注解
def greet(name: str) -> str:
    return f"Hello, {name}"

# 异常处理
try:
    process_data()
except ValueError as e:
    logger.error(f"处理失败：{e}")
    raise
```

---

## 详细规范索引

本规范按主题拆分为多个文档，根据你的具体需求参考相应的章节：

| 主题 | 参考文档 | 使用场景 |
|------|---------|----------|
| 代码风格与可读性 | `style-and-readability.md` | 命名、格式化、注释、控制结构、**测试可读性** |
| 安全编码 | `security.md` | 输入验证、密码存储、加密、并发安全、**测试安全** |
| 性能与可靠性 | `performance-and-reliability.md` | 内存优化、并发性能、I/O 优化、**单元测试规范** |
| 核心原则与设计 | `core-principles-and-design.md` | 接口设计、类设计、异常处理、**可测试性设计** |

### 何时查看哪个文档

- **日常编码** → `style-and-readability.md`
- **处理敏感数据** → `security.md`
- **性能优化** → `performance-and-reliability.md`
- **架构设计** → `core-principles-and-design.md`

---

## 核心原则a

### Python 的设计哲学

| 原则 | 说明 |
|------|------|
| 可读性 | 代码首先是写给人看的 |
| 简洁性 | 简单优于复杂，明了优于隐式 |
| 实用性 | 实用优于纯粹 |
| 显式 | 显式优于隐式 |

### 接口设计

```python
# 使用抽象基类定义接口
from abc import ABC, abstractmethod

class DataProcessor(ABC):
    @abstractmethod
    def process(self, data: bytes) -> bytes:
        pass

# 依赖注入
class Service:
    def __init__(self, processor: DataProcessor):
        self._processor = processor
```

### 异常处理

```python
# 只捕获能处理的异常
try:
    result = api_call()
except RequestError as e:
    logger.warning(f"API 调用失败：{e}")
    return fallback_value()

# 使用异常链
def load_config(path: str) -> dict:
    try:
        return json.load(open(path))
    except (IOError, json.JSONDecodeError) as e:
        raise ConfigError(f"加载配置失败：{e}") from e
```

### 并发原则

```python
# 使用 asyncio 处理 I/O 密集型
async def fetch_all(urls: list[str]) -> list:
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(session, url) for url in urls]
        return await asyncio.gather(*tasks)

# 使用 ThreadPoolExecutor 处理 CPU 密集型
from concurrent.futures import ThreadPoolExecutor

with ThreadPoolExecutor() as executor:
    results = executor.map(cpu_intensive_task, data_list)
```

---

## 禁止事项

- ❌ 不使用类型注解（公共 API）
- ❌ 使用无意义的变量名（x, data, temp）
- ❌ 公共方法/类不写文档字符串
- ❌ 忽略异常或使用裸 except
- ❌ 在循环中进行重复的字符串拼接
- ❌ 嵌套过深（超过 3 层）
- ❌ SQL 使用字符串拼接（使用参数化查询）
- ❌ 密码不使用 bcrypt/argon2 哈希
- ❌ 记录敏感信息到日志
- ❌ 无锁访问共享状态
- ❌ 外部调用不设置超时
- ❌ 硬编码密钥（从环境变量读取）
- ❌ 可变类型作为默认参数

---

## 推荐实践

1. **类型注解** - 公共 API 必须有类型注解
2. **遵循 PEP 8** - 所有代码符合 PEP 8 规范
3. **使用 dataclass** - 数据容器优先使用 dataclass
4. **文档字符串** - 公共函数/类写清晰的文档
5. **上下文管理器** - 资源管理使用 with 语句
6. **命名语义化** - 变量名清晰表达意图
7. **组合优于继承** - 优先使用组合模式
8. **惰性求值** - 大数据集使用生成器
9. **设置超时** - 所有外部调用必须有超时
10. **测试驱动** - 新功能先写测试

---

## 代码审查清单

提交代码前，请确认：

- [ ] 代码已通过 `black` 和 `ruff` 格式化
- [ ] 公共 API 有类型注解
- [ ] 公共函数/类有文档字符串
- [ ] 所有异常都经过处理，没有裸 except
- [ ] 使用参数化查询防止 SQL 注入
- [ ] 密码使用 bcrypt/argon2 哈希
- [ ] 敏感信息已脱敏或从环境变量读取
- [ ] 并发访问共享状态使用锁
- [ ] 外部调用（HTTP、数据库）设置了超时
- [ ] 使用上下文管理器确保资源释放
- [ ] 没有可变默认参数
- [ ] 函数职责单一，参数不超过 5 个
- [ ] 测试覆盖率不低于 80%

---

## 参考资料

官方文档和更多详细信息请查看：
- [PEP 8 - Style Guide for Python Code](https://peps.python.org/pep-0008/)
- [PEP 484 - Type Hints](https://peps.python.org/pep-0484/)
- [The Python Tutorial](https://docs.python.org/3/tutorial/)
- [Effective Python](https://effectivepython.com/)

完整的规范文档请查看 `references/` 目录下的各个主题文档。
