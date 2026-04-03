# Python 代码风格与可读性规范

## AI 使用提示

- 当任务涉及命名、格式化、类型注解、函数签名、导入组织、异常写法或测试可读性时，先看本文件。
- 默认优先减少噪音和嵌套，不为了“聪明写法”牺牲可读性。
- 如果一段代码需要额外解释才能看懂，优先考虑简化写法而不是补注释。

> AI Code 遵循规约，确保 Python 代码可读、统一、易维护。

## 1. 命名规范

```python
# 模块名：小写加下划线
# user_service.py


# 类名：PascalCase
class UserService:
    pass


# 函数和变量：snake_case
def calculate_total_price() -> int:
    return 0


user_count = 0
is_active = True


# 常量：UPPER_SNAKE_CASE
MAX_RETRIES = 3
DEFAULT_TIMEOUT = 30


# 非公开属性：单下划线
class Cache:
    def __init__(self) -> None:
        self._items: dict[str, object] = {}
```

## 2. 格式化

```bash
black .
ruff check . --fix
```

- 项目已有统一工具链时，优先服从项目配置。
- 没有明确约定时，默认保持 4 空格缩进、单一风格、最小必要换行。

## 3. 类型注解

```python
def greet(name: str) -> str:
    return f"hello, {name}"


def build_user_map(users: list[User]) -> dict[str, User]:
    return {user.id: user for user in users}


def find_user(user_id: str) -> User | None:
    return None
```

- 公共 API、跨模块边界和复杂返回值优先补类型注解。
- 动态元编程、框架魔法或历史代码边界，不做机械补全。
- 类型注解的目标是减少歧义，不是追求表面覆盖率。

## 4. 文档字符串

```python
class UserService:
    """处理用户注册、查询和状态变更。"""


def create_user(username: str, email: str) -> User:
    """创建用户并返回新对象。"""
```

- 公共模块、公共类、公共函数在需要时补简洁 docstring。
- 说明“做什么”和关键约束，不重复参数名翻译。
- 私有小函数如果名字已经足够清晰，可以不写 docstring。

## 5. 函数设计

```python
def create_user(
    *,
    username: str,
    email: str,
    password: str,
    role: str = "user",
) -> User:
    ...


def add_item(item: str, items: list[str] | None = None) -> list[str]:
    if items is None:
        items = []
    items.append(item)
    return items
```

- 参数多到影响阅读时，优先关键字参数、数据类或配置对象。
- 函数优先单一职责；超过 50 行通常要怀疑是否可拆。
- 避免可变默认参数。

## 6. 控制结构

```python
if users:
    process(users)


for index, name in enumerate(names):
    print(index, name)


for name, age in zip(names, ages):
    print(name, age)


result = "active" if is_active else "inactive"
```

- 简单场景用推导式；一旦嵌套或条件过多就改回普通循环。
- 优先早返回，减少深层嵌套。
- `match-case` 只在确实提升清晰度时使用。

## 7. 异常写法

```python
try:
    result = call_external_api()
except NetworkError as exc:
    raise ServiceUnavailableError("外部服务暂不可用") from exc


try:
    process()
except (ValueError, TypeError) as exc:
    logger.warning("处理失败: %s", exc)
```

- 只捕获能处理的异常。
- 新增上下文时再包装或转换异常。
- 禁止裸 `except` 和静默吞错。

## 8. 类与对象

```python
from dataclasses import dataclass, field
from datetime import datetime


@dataclass
class User:
    id: str
    name: str
    email: str
    created_at: datetime = field(default_factory=datetime.utcnow)
```

- 纯数据容器优先 `dataclass`。
- 属性访问逻辑复杂时再使用 `@property`。
- 混入和继承只在确有收益时使用。

## 9. 导入组织

```python
from __future__ import annotations

import os
from pathlib import Path

import requests

from myapp.models import User
from myapp.services import UserService
```

- 导入顺序：标准库 -> 第三方 -> 本地模块。
- 避免通配符导入。
- 循环依赖优先通过重构边界解决，而不是堆局部导入。

## 10. 测试可读性

```python
def test_create_user_valid_input_returns_success() -> None:
    # Arrange
    payload = {"username": "alice", "email": "alice@example.com"}

    # Act
    user = create_user(**payload)

    # Assert
    assert user.email == "alice@example.com"
```

```python
import pytest


@pytest.mark.parametrize(
    ("email", "expected"),
    [
        ("user@example.com", True),
        ("invalid-email", False),
    ],
)
def test_validate_email(email: str, expected: bool) -> None:
    assert validate_email(email) is expected
```

- 测试名应描述行为、场景和预期。
- 断言优先清晰、直接，不要依赖模糊真值。
- 测试逻辑本身应比生产代码更简单。

## 11. 禁止事项

- ❌ 公共边界完全缺失类型信息
- ❌ 可变对象作为默认参数
- ❌ 裸 `except`
- ❌ 过深嵌套（超过 3 层）
- ❌ 超长函数或单文件职责失控
- ❌ 魔法数字和无语义变量名
- ❌ 用复杂推导式压缩本应拆开的逻辑
