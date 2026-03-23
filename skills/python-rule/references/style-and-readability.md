# Python 代码风格与可读性规范

> AI Code 遵循规约，确保代码可读性和风格统一。

---

## 🤖 AI 使用说明

**何时查阅本规范**：当你编写 Python 代码、审查代码、格式化代码、命名变量/函数、编写文档字符串、添加类型注解、处理异常、编写测试时。

**核心规则（必须遵守）**：
1. 公共 API 必须有类型注解
2. 公共函数/类必须有文档字符串
3. 禁止可变默认参数（如 `def func(items=[])`）
4. 禁止裸 except 语句
5. 函数不超过 50 行，嵌套不超过 3 层

**快速决策**：
- 要命名 → 本章第 1 节
- 要格式化 → 本章第 2 节
- 要写文档 → 本章第 3 节
- 要类型注解 → 本章第 4 节
- 要写测试 → 本章第 13 节

---

## 1. 命名规范

```python
# 包名：全小写，简短，单单词（允许下划线）
# 包目录
package_name/
    __init__.py
    module_name.py

# 模块名：全小写，单词间用下划线
# user_service.py
# auth_manager.py

# 类名：大驼峰（PascalCase）
class UserService:
    pass

class HTTPClient:
    pass

# 函数和变量：小写 + 下划线（snake_case）
def calculate_total_price():
    pass

user_count = 0
is_active = True

# 常量：全大写 + 下划线
MAX_RETRIES = 3
DEFAULT_TIMEOUT = 30

# 私有变量：单下划线前缀
class Cache:
    def __init__(self):
        self._cache = {}
        self._hit_count = 0

# 名称修饰（子类不覆盖）：双下划线前缀
class Parent:
    def __init__(self):
        self.__internal = None  # 被修饰为 _Parent__internal

# 特殊方法：双下划线前后
__init__
__str__
__len__
```

## 2. 格式化

```bash
# 使用 black 格式化（行宽 100）
black --line-length 100 .

# 使用 ruff 检查
ruff check . --fix

# .editorconfig 配置
[*.py]
indent_style = space
indent_size = 4
end_of_line = lf
trim_trailing_whitespace = true
insert_final_newline = true
max_line_length = 100
```

## 3. 文档字符串

```python
# 模块文档字符串（文件顶部）
"""用户服务模块。

提供用户注册、登录、信息管理等功能。
"""

# 类文档字符串
class UserService:
    """用户服务类。

    负责处理用户相关的业务逻辑，包括注册、认证和信息管理。

    Attributes:
        db: 数据库连接对象
        cache: 缓存对象
    """

    def __init__(self, db, cache):
        """初始化用户服务。

        Args:
            db: 数据库连接
            cache: 缓存对象
        """
        self.db = db
        self.cache = cache

# 函数文档字符串
def create_user(username: str, email: str, password: str) -> User:
    """创建新用户。

    验证用户信息并创建新用户账户。

    Args:
        username: 用户名，3-32 个字符
        email: 邮箱地址
        password: 密码，最少 8 个字符

    Returns:
        创建的用户对象

    Raises:
        ValidationError: 当输入数据验证失败时
        DuplicateError: 当用户名或邮箱已存在时

    Example:
        >>> user = create_user("john", "john@example.com", "secure123")
        >>> print(user.id)
    """
    pass

# 单行文档字符串
def add(a: int, b: int) -> int:
    """返回两数之和。"""
    return a + b
```

## 4. 类型注解

```python
# 基本类型注解
def greet(name: str) -> str:
    return f"Hello, {name}"

# 容器类型
from typing import list, dict, set, tuple

def process_items(items: list[str]) -> dict[str, int]:
    pass

# Optional（可以为 None）
from typing import Optional

def find_user(user_id: int) -> Optional[User]:
    """查找用户，不存在时返回 None。"""
    pass

# Union（多种类型）
from typing import Union

def parse(value: Union[str, int, float]) -> int:
    pass

# 3.10+ 使用 | 操作符
def parse(value: str | int | float) -> int:
    pass

# Callable（函数类型）
from typing import Callable

def run_callback(
    callback: Callable[[str], bool],
    data: str
) -> bool:
    pass

# 泛型类型
from typing import Generic, TypeVar

T = TypeVar('T')

class Stack(Generic[T]):
    def push(self, item: T) -> None:
        pass

    def pop(self) -> T:
        pass

# 类属性类型注解
class Config:
    name: str
    timeout: int = 30
    debug: bool = False
```

## 5. 控制结构

```python
# if 语句：使用真值测试
# ✅ 推荐
if users:  # 列表非空为真
    process(users)

if name:  # 字符串非空为真
    greet(name)

# ❌ 避免
if len(users) > 0:
    process(users)

if name != "":
    greet(name)

# for 循环：优先使用枚举和压缩
names = ['Alice', 'Bob', 'Charlie']

# ✅ 推荐：使用 enumerate
for i, name in enumerate(names):
    print(f"{i}: {name}")

# ✅ 推荐：使用 zip
ages = [25, 30, 35]
for name, age in zip(names, ages):
    print(f"{name}: {age}")

# 列表推导式（简单场景）
squares = [x**2 for x in range(10)]
even_squares = [x**2 for x in range(10) if x % 2 == 0]

# 生成器表达式（大数据集）
sum_squares = sum(x**2 for x in range(1000000))

# 字典推导式
user_map = {user.id: user.name for user in users}

# 三元表达式
status = "active" if is_active else "inactive"

# match-case（Python 3.10+）
def handle_status(status: str) -> str:
    match status:
        case "active":
            return "处理活跃状态"
        case "pending" | "waiting":
            return "处理等待状态"
        case s if s.startswith("error_"):
            return f"处理错误：{s}"
        case _:
            return "未知状态"
```

## 6. 函数设计

```python
# 参数顺序：位置参数 -> *args -> 关键字参数 -> **kwargs
def create_request(
    method: str,
    url: str,
    *args,
    timeout: int = 30,
    retries: int = 3,
    **kwargs
) -> Response:
    pass

# 只接收关键字参数（强制可读性）
def create_user(
    *,
    username: str,
    email: str,
    password: str,
    role: str = "user"
) -> User:
    pass

# 调用时必须使用关键字
create_user(
    username="john",
    email="john@example.com",
    password="secure123"
)

# 可变参数：注意默认参数陷阱
# ❌ 错误：可变默认参数
def add_item(item, items=[]):  # 危险！列表在所有调用间共享
    items.append(item)
    return items

# ✅ 正确：使用 None 作为默认值
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items

# 参数过多时使用数据类或配置对象
# ❌ 参数过多
def create_order(user_id, product_id, quantity, price, discount, tax, shipping_address, billing_address):
    pass

# ✅ 使用配置对象
from dataclasses import dataclass

@dataclass
class OrderConfig:
    quantity: int
    discount: float = 0.0
    tax: float = 0.1
    shipping_address: str = ""
    billing_address: str = ""

def create_order(user_id: str, product_id: str, config: OrderConfig):
    pass
```

## 7. 类和对象

```python
# 使用 dataclass（Python 3.7+）
from dataclasses import dataclass, field
from datetime import datetime

@dataclass
class User:
    """用户数据类。"""
    id: str
    name: str
    email: str
    created_at: datetime = field(default_factory=datetime.now)
    tags: list[str] = field(default_factory=list)

    def to_dict(self) -> dict:
        """转换为字典。"""
        return {
            'id': self.id,
            'name': self.name,
            'email': self.email,
        }

# 类继承结构
class Base:
    """基类。"""
    pass

class MixinA:
    """混入类 A。"""
    pass

class MyClass(MixinA, Base):
    """组合类。"""
    pass

# 属性装饰器
class Product:
    def __init__(self, price: float):
        self._price = price

    @property
    def price(self) -> float:
        """获取价格（只读）。"""
        return self._price

    @price.setter
    def price(self, value: float):
        """设置价格（带验证）。"""
        if value < 0:
            raise ValueError("价格不能为负数")
        self._price = value
```

## 8. 导入组织

```python
# 导入顺序：标准库 -> 第三方库 -> 本地应用
# 1. 标准库
from __future__ import annotations
import os
import sys
from typing import Optional

# 2. 第三方库
import requests
from flask import Flask

# 3. 本地应用/项目内模块
from .utils import helper
from ..models import User
from myapp.services import UserService

# 导入最佳实践
# ✅ 推荐
import os
from pathlib import Path

# ❌ 避免：通配符导入
from module import *

# ❌ 避免：循环导入（使用类型注解字符串）
# file_a.py
def process(node: 'Node') -> None:  # 使用字符串避免循环导入
    pass

# 相对导入（包内）
from . import sibling_module
from .submodule import function
from .. import parent_module
```

## 9. 异常处理

```python
# 只捕获能处理的异常
try:
    result = api_call()
except NetworkError:
    return fallback_value()

# ❌ 避免：裸 except
try:
    process()
except:  # 会捕获 SystemExit、KeyboardInterrupt
    pass

# 捕获多个异常
try:
    process()
except (IOError, ValueError) as e:
    logger.error(f"处理失败：{e}")

# 异常链（保留原始异常）
def load_config(path: str) -> dict:
    try:
        with open(path) as f:
            return json.load(f)
    except (IOError, json.JSONDecodeError) as e:
        raise ConfigError(f"加载配置失败") from e

# else 子句（无异常时执行）
try:
    result = risky_operation()
except ValueError:
    handle_error()
else:
    # 只有在没有异常时执行
    process_result(result)
finally:
    # 总是会执行（清理资源）
    cleanup()
```

## 10. 上下文管理器

```python
# 使用现有的上下文管理器
with open('file.txt') as f:
    data = f.read()

# 多个上下文管理器
with open('input.txt') as fin, open('output.txt', 'w') as fout:
    fout.write(fin.read())

# 使用 contextlib
from contextlib import contextmanager

@contextmanager
def timer(name: str):
    """计时上下文管理器。"""
    import time
    start = time.time()
    try:
        yield
    finally:
        elapsed = time.time() - start
        print(f"{name}: {elapsed:.2f}s")

# 使用
with timer("processing"):
    process_data()

# 类实现的上下文管理器
class ManagedResource:
    def __enter__(self):
        self.resource = acquire_resource()
        return self.resource

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.resource.close()
```

## 11. 代码组织

```python
# 模块内组织顺序
"""模块文档字符串。"""

# 1. 导入
from __future__ import annotations
import os
from typing import Optional

# 2. 模块级常量
MAX_SIZE = 100
DEFAULT_TIMEOUT = 30

# 3. 模块级变量（谨慎使用）
_cache: dict = {}

# 4. 异常类（如果有）
class ServiceError(Exception):
    """服务异常基类。"""
    pass

# 5. 类和函数
class Service:
    pass

def process() -> None:
    pass

# 函数内部组织
def complex_function():
    # 1. 文档字符串
    # 2. 输入验证
    # 3. 局部变量定义
    # 4. 主要逻辑
    # 5. 返回结果
    pass
```

## 12. 禁止事项

- ❌ 不使用类型注解（公共 API）
- ❌ 不使用文档字符串（公共函数/类）
- ❌ 可变类型作为默认参数 `def func(items=[])`
- ❌ 裸 except 语句
- ❌ 在 finally 中使用 return
- ❌ 在循环中执行不必要的重复计算
- ❌ 过深的嵌套（超过 3 层）
- ❌ 过长的函数（超过 50 行）
- ❌ 单个文件超过 500 行
- ❌ 使用魔法数字（使用具名常量）

## 13. 测试可读性

### 测试文件命名

```python
# 测试文件：test_<module>.py 或 <module>_test.py
# test_user_service.py 或 user_service_test.py
# 推荐：test_<module>.py（pytest 默认识别）
```

### 测试函数命名

```python
# 格式：test_<function>_<scenario>_<expected>
def test_create_user_valid_input_returns_success():
    """测试创建用户 - 有效输入返回成功。"""
    pass

def test_create_user_duplicate_email_returns_error():
    """测试创建用户 - 重复邮箱返回错误。"""
    pass

def test_parse_email_empty_string_raises_error():
    """测试解析邮箱 - 空字符串抛出异常。"""
    pass
```

### 测试结构（AAA 模式）

```python
def test_calculate_total_price():
    # Arrange（准备）
    cart = ShoppingCart()
    cart.add_item(Item(price=100))
    cart.add_item(Item(price=200))

    # Act（执行）
    total = cart.calculate_total()

    # Assert（断言）
    assert total == 300
```

### 参数化测试

```python
import pytest

@pytest.mark.parametrize("email,expected", [
    ("user@example.com", True),
    ("invalid-email", False),
    ("@example.com", False),
    ("user@", False),
])
def test_validate_email(email, expected):
    assert validate_email(email) == expected
```

### 测试夹具（Fixtures）

```python
# conftest.py 或测试文件中
import pytest

@pytest.fixture
def sample_user():
    """创建测试用户夹具。"""
    return User(
        id="test-123",
        name="测试用户",
        email="test@example.com"
    )

@pytest.fixture
def mock_db():
    """模拟数据库夹具。"""
    db = MagicMock()
    db.query.return_value = []
    return db

# 使用夹具
def test_user_service_create(sample_user, mock_db):
    service = UserService(mock_db)
    result = service.create(sample_user)
    assert result.id == sample_user.id
```

### 异常测试

```python
import pytest

def test_divide_by_zero_raises_error():
    with pytest.raises(ZeroDivisionError):
        divide(1, 0)

def test_invalid_input_raises_type_error():
    with pytest.raises(TypeError, match="must be numbers"):
        add("1", 2)
```

### 测试中的模拟

```python
from unittest.mock import Mock, patch, MagicMock

# 使用 Mock
def test_with_mock():
    mock_api = Mock()
    mock_api.get.return_value = {"status": "ok"}

    result = call_api(mock_api)

    mock_api.get.assert_called_once_with("/endpoint")
    assert result == {"status": "ok"}

# 使用 patch 装饰器
@patch('module.requests.get')
def test_with_patch_decorator(mock_get):
    mock_get.return_value.json.return_value = {"data": "test"}

    result = fetch_data()

    assert result == {"data": "test"}

# 使用 patch 上下文
def test_with_patch_context():
    with patch('module.requests.get') as mock_get:
        mock_get.return_value.json.return_value = {"data": "test"}
        result = fetch_data()
        assert result == {"data": "test"}
```

### 测试清晰度

```python
# ✅ 清晰的断言
assert user.id is not None
assert user.status == UserStatus.ACTIVE
assert len(errors) == 0

# ❌ 模糊的断言
assert user
assert user.status
assert not errors

# ✅ 带有清晰消息的断言
assert total == 300, f"期望总价 300，实际{total}"
```

### 禁止事项（测试）

- ❌ 测试函数命名不清晰（`test_1`, `test_something`）
- ❌ 断言消息模糊不清
- ❌ 测试用例缺少文档字符串
- ❌ 测试逻辑过于复杂（测试本身应简单）
- ❌ 测试之间有依赖关系
- ❌ 使用真实的网络/数据库（应使用 mock）
