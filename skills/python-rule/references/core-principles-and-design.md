# Python 核心原则与设计规范

> AI Code 遵循规约，设计优雅、可维护的系统。

---

## 🤖 AI 使用说明

**何时查阅本规范**：当你设计新模块、创建类结构、选择设计模式、组织项目结构、处理依赖关系、定义接口时。

**核心原则（指导所有设计决策）**：
1. 组合优于继承
2. 依赖抽象而非具体实现
3. 数据类优先（dataclass）
4. 纯函数优于副作用
5. 显式优于隐式

**设计决策树**：
- 要存储数据 → 使用 dataclass（第 4 节）
- 要定义接口 → 使用 ABC 或 Protocol（第 7 节）
- 要复用代码 → 优先组合而非继承（第 3 节）
- 要便于测试 → 依赖注入（第 5 节）
- 要组织模块 → 参考项目结构（第 8 节）
- 要管理配置 → 使用配置对象（第 10 节）

---

## 1. Python 之禅

```python
# import this 查看 Python 之禅
# 核心原则：
# - 优美优于丑陋
# - 明了优于隐式
# - 简单优于复杂
# - 扁平优于嵌套
# - 可读性很重要
```

## 2. 设计原则

### SOLID 原则

```python
# 单一职责原则 (SRP)
# 一个类应该只有一个改变的理由

# ❌ 违反 SRP
class UserService:
    def create_user(self, data): ...
    def send_email(self, user): ...      # 通信职责
    def log_activity(self, action): ...  # 日志职责

# ✅ 遵循 SRP
class UserService:
    def create_user(self, data): ...

class EmailService:
    def send_welcome_email(self, user): ...

class ActivityLogger:
    def log(self, action): ...

# 开闭原则 (OCP)
# 对扩展开放，对修改关闭

from abc import ABC, abstractmethod

class PaymentProcessor(ABC):
    @abstractmethod
    def process(self, amount: float) -> bool:
        pass

class CreditCardProcessor(PaymentProcessor):
    def process(self, amount: float) -> bool:
        # 信用卡处理
        pass

class PayPalProcessor(PaymentProcessor):
    def process(self, amount: float) -> bool:
        # PayPal 处理
        pass

# 使用时不需要修改现有代码
def checkout(processor: PaymentProcessor, amount: float):
    processor.process(amount)

# 里氏替换原则 (LSP)
# 子类应该能够替换父类

class Bird(ABC):
    @abstractmethod
    def move(self):
        pass

class Sparrow(Bird):
    def move(self):
        return "Flying"

class Penguin(Bird):
    def move(self):
        return "Swimming"  # 合理的实现

# 接口隔离原则 (ISP)
# 客户端不应依赖不需要的接口

# ❌ 臃肿的接口
class Worker(ABC):
    @abstractmethod
    def work(self): pass

    @abstractmethod
    def eat(self): pass  # Robot 不需要

# ✅ 分离的接口
class Workable(ABC):
    @abstractmethod
    def work(self): pass

class Eatable(ABC):
    @abstractmethod
    def eat(self): pass

class Human(Workable, Eatable):
    def work(self): ...
    def eat(self): ...

class Robot(Workable):
    def work(self): ...

# 依赖倒置原则 (DIP)
# 依赖抽象而非具体实现

class OrderService:
    def __init__(self, repository: OrderRepository):  # 依赖抽象
        self._repository = repository

    def create_order(self, data):
        return self._repository.save(data)
```

## 3. 组合优于继承

```python
# ❌ 过度使用继承
class Animal: pass
class Mammal(Animal): pass
class Bird(Animal): pass
class FlyingMammal(Mammal): pass  # 复杂且难以维护

# ✅ 使用组合
class FlyBehavior:
    def fly(self):
        return "Flying"

class SwimBehavior:
    def swim(self):
        return "Swimming"

class Animal:
    def __init__(self, behaviors=None):
        self._behaviors = behaviors or []

    def perform(self, action):
        for behavior in self._behaviors:
            if hasattr(behavior, action):
                return getattr(behavior, action)()

# 灵活组合
duck = Animal([FlyBehavior(), SwimBehavior()])
```

## 4. 数据类优先

```python
from dataclasses import dataclass, field
from typing import Optional
from datetime import datetime

# ✅ 使用 dataclass 存储数据
@dataclass
class User:
    id: int
    name: str
    email: str
    created_at: datetime = field(default_factory=datetime.now)
    is_active: bool = True

# ✅ 使用 frozen 创建不可变对象
@dataclass(frozen=True)
class Point:
    x: float
    y: float

# ✅ 使用 __post_init__ 进行验证
@dataclass
class Product:
    name: str
    price: float
    quantity: int = 0

    def __post_init__(self):
        if self.price < 0:
            raise ValueError("价格不能为负")
        if self.quantity < 0:
            raise ValueError("数量不能为负")

# ✅ 添加方法
@dataclass
class ShoppingCart:
    items: list = field(default_factory=list)

    def add(self, item):
        self.items.append(item)

    def total(self) -> float:
        return sum(item.price * item.quantity for item in self.items)
```

## 5. 可测试性设计

```python
# ✅ 依赖注入便于测试
class OrderService:
    def __init__(self, db, email_service, payment_gateway):
        self._db = db
        self._email_service = email_service
        self._payment_gateway = payment_gateway

    def create_order(self, user_id, items):
        # 可以轻松 mock 各个依赖
        pass

# ✅ 纯函数易于测试
def calculate_total(items: list, tax_rate: float) -> float:
    """纯函数：相同输入总是返回相同输出。"""
    subtotal = sum(item.price * item.quantity for item in items)
    return subtotal * (1 + tax_rate)

# ✅ 避免全局状态
# ❌ 难以测试
config = {}

def get_setting(key):
    return config.get(key)

# ✅ 使用配置对象
class Config:
    def __init__(self, settings: dict):
        self._settings = settings

    def get(self, key, default=None):
        return self._settings.get(key, default)
```

## 6. 错误处理设计

```python
# ✅ 定义领域异常
class AppError(Exception):
    """应用异常基类。"""
    pass

class ValidationError(AppError):
    """数据验证失败。"""
    pass

class NotFoundError(AppError):
    """资源未找到。"""
    pass

class DuplicateError(AppError):
    """重复资源。"""
    pass

# ✅ 使用异常链
def load_user(user_id: int) -> User:
    try:
        return db.query(User).filter_by(id=user_id).one()
    except NoResultFound as e:
        raise NotFoundError(f"用户 {user_id} 未找到") from e

# ✅ 上下文管理器处理事务
from contextlib import contextmanager

@contextmanager
def transaction():
    """事务上下文管理器。"""
    try:
        yield
        db.commit()
    except Exception:
        db.rollback()
        raise
    finally:
        db.close()

# 使用
with transaction():
    create_user(data)
    send_welcome_email(data)
```

## 7. 接口设计

```python
# ✅ 使用抽象基类定义接口
from abc import ABC, abstractmethod

class Repository(ABC):
    """仓储接口。"""

    @abstractmethod
    def find_by_id(self, id: int) -> Optional[object]:
        pass

    @abstractmethod
    def save(self, entity: object) -> object:
        pass

    @abstractmethod
    def delete(self, id: int) -> bool:
        pass

# ✅ 实现接口
class UserRepository:
    def __init__(self, session):
        self._session = session

    def find_by_id(self, id: int) -> Optional[User]:
        return self._session.query(User).filter_by(id=id).first()

    def save(self, user: User) -> User:
        self._session.add(user)
        self._session.commit()
        return user

    def delete(self, id: int) -> bool:
        user = self.find_by_id(id)
        if user:
            self._session.delete(user)
            self._session.commit()
            return True
        return False

# ✅ 使用 Protocol（结构子类型，Python 3.8+）
from typing import Protocol

class SupportsClose(Protocol):
    def close(self) -> None: ...

def cleanup(resource: SupportsClose):
    resource.close()
```

## 8. 模块组织

```python
# 推荐的项目结构
project/
├── pyproject.toml          # 项目配置和依赖
├── README.md
├── tests/
│   ├── __init__.py
│   ├── test_user.py
│   └── test_order.py
├── src/
│   └── mypackage/
│       ├── __init__.py
│       ├── api/            # 公共 API
│       │   ├── __init__.py
│       │   └── endpoints.py
│       ├── core/           # 核心业务逻辑
│       │   ├── __init__.py
│       │   ├── user.py
│       │   └── order.py
│       ├── infrastructure/ # 基础设施
│       │   ├── __init__.py
│       │   ├── database.py
│       │   └── cache.py
│       └── utils/          # 工具函数
│           ├── __init__.py
│           └── helpers.py
└── scripts/                # 脚本工具
    ├── setup_db.py
    └── migrate.py
```

## 9. 包设计

```python
# __init__.py - 定义公共 API
from .core.user import UserService, User
from .core.order import OrderService, Order
from .infrastructure.database import Database

__all__ = [
    'UserService',
    'User',
    'OrderService',
    'Order',
    'Database',
]

__version__ = '1.0.0'

# 懒加载（减少启动时间）
def __getattr__(name):
    if name == 'LegacyService':
        from .legacy import LegacyService
        return LegacyService
    raise AttributeError(f"module {__name__!r} has no attribute {name!r}")
```

## 10. 配置管理

```python
# ✅ 使用数据类管理配置
from dataclasses import dataclass, field
from typing import Optional
import os

@dataclass
class DatabaseConfig:
    host: str = "localhost"
    port: int = 5432
    name: str = "mydb"
    user: str = field(default_factory=lambda: os.getenv("DB_USER", "postgres"))
    password: str = field(default_factory=lambda: os.getenv("DB_PASSWORD", ""))

    @property
    def url(self) -> str:
        return f"postgresql://{self.user}:{self.password}@{self.host}:{self.port}/{self.name}"

@dataclass
class AppConfig:
    debug: bool = False
    database: DatabaseConfig = field(default_factory=DatabaseConfig)

    @classmethod
    def from_env(cls) -> 'AppConfig':
        return cls(
            debug=os.getenv("DEBUG", "false").lower() == "true",
            database=DatabaseConfig(
                host=os.getenv("DB_HOST", "localhost"),
                port=int(os.getenv("DB_PORT", "5432")),
                name=os.getenv("DB_NAME", "mydb"),
            )
        )
```

## 11. 日志设计

```python
import logging
from logging.handlers import RotatingFileHandler

# ✅ 配置日志
def setup_logger(name: str, level=logging.INFO) -> logging.Logger:
    logger = logging.getLogger(name)
    logger.setLevel(level)

    # 控制台处理器
    console = logging.StreamHandler()
    console.setFormatter(logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    ))
    logger.addHandler(console)

    # 文件处理器（轮转）
    file_handler = RotatingFileHandler(
        f'logs/{name}.log',
        maxBytes=10*1024*1024,  # 10MB
        backupCount=5
    )
    file_handler.setFormatter(logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    ))
    logger.addHandler(file_handler)

    return logger

# ✅ 结构化日志
import json

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            'timestamp': self.formatTime(record),
            'level': record.levelname,
            'logger': record.name,
            'message': record.getMessage(),
        }
        if record.exc_info:
            log_data['exception'] = self.formatException(record.exc_info)
        return json.dumps(log_data)
```

## 12. 代码审查清单

### 设计方面
- [ ] 是否遵循单一职责原则
- [ ] 是否优先使用组合而非继承
- [ ] 是否依赖抽象而非具体实现
- [ ] 是否有合适的异常处理
- [ ] 模块职责是否清晰

### 可测试性
- [ ] 是否使用依赖注入
- [ ] 是否有足够的纯函数
- [ ] 是否避免了全局状态
- [ ] 是否有清晰的测试接口

### 可维护性
- [ ] 命名是否清晰表达意图
- [ ] 函数是否足够小（< 50 行）
- [ ] 类是否足够专注（< 500 行）
- [ ] 是否有适当的文档字符串

## 13. 禁止事项

- ❌ 使用继承当可以用组合时
- ❌ 创建上帝类/上帝函数
- ❌ 硬编码配置值
- ❌ 使用全局变量存储状态
- ❌ 捕获异常后不处理（吞掉异常）
- ❌ 使用异常控制流程
- ❌ 返回裸字典（使用数据类或命名元组）
- ❌ 模块承担过多职责
