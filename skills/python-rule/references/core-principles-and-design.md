# Python 核心原则与设计规范

## AI 使用提示

- 当任务涉及模块边界、类设计、依赖组织、异常模型、配置对象或接口抽象时，先看本文件。
- 如果不确定该抽象成类、函数、协议还是数据模型，默认以“更小、更显式、更易测”为准。
- 如果某个设计需要很长解释才能说清，通常说明抽象偏重或边界不清。

> AI Code 遵循规约，设计优雅、可维护的 Python 系统。

## 1. 设计哲学

```python
# 可读性优先
def is_valid_user(user: User) -> bool:
    return bool(user.id and user.name and user.email)


# 显式优于隐式
def load_user(repo: UserRepository, user_id: str) -> User:
    user = repo.find_by_id(user_id)
    if user is None:
        raise NotFoundError(f"用户不存在: {user_id}")
    return user


# 简单优于复杂
def calculate_total(items: list[Item]) -> int:
    return sum(item.price * item.quantity for item in items)
```

## 2. 数据与行为边界

```python
from dataclasses import dataclass, field
from datetime import datetime


# 数据容器优先使用 dataclass
@dataclass
class User:
    id: str
    name: str
    email: str
    created_at: datetime = field(default_factory=datetime.utcnow)


# 需要行为时再提供方法
@dataclass
class Order:
    items: list[Item]

    def total_amount(self) -> int:
        return sum(item.price * item.quantity for item in self.items)
```

## 3. 组合优于继承

```python
# ✅ 使用组合组织协作
class EmailSender:
    def send(self, to: str, body: str) -> None: ...


class UserNotifier:
    def __init__(self, sender: EmailSender) -> None:
        self._sender = sender

    def send_welcome(self, email: str) -> None:
        self._sender.send(email, "welcome")


# ❌ 不要为了复用少量逻辑构建深继承树
class BaseNotifier: ...
class EmailNotifier(BaseNotifier): ...
class WelcomeEmailNotifier(EmailNotifier): ...
```

## 4. 接口设计

```python
from typing import Protocol


# 协议优先表达能力边界
class UserRepository(Protocol):
    def find_by_id(self, user_id: str) -> User | None: ...
    def save(self, user: User) -> User: ...


class AuditLogger(Protocol):
    def info(self, message: str, **fields: object) -> None: ...


class UserService:
    def __init__(self, repo: UserRepository, logger: AuditLogger) -> None:
        self._repo = repo
        self._logger = logger
```

## 5. 函数设计

```python
# 函数只做一件事
def validate_email(email: str) -> None: ...


def create_user(repo: UserRepository, payload: UserCreate) -> User: ...


# 参数过多时使用关键字参数或配置对象
@dataclass
class RetryConfig:
    timeout: float = 3.0
    retries: int = 2
    backoff: float = 0.5


def call_remote_api(url: str, *, config: RetryConfig) -> dict[str, object]:
    ...
```

## 6. 异常模型

```python
class AppError(Exception):
    """应用异常基类。"""


class ValidationError(AppError):
    """输入验证失败。"""


class NotFoundError(AppError):
    """资源不存在。"""


class ConflictError(AppError):
    """资源冲突。"""


def load_config(path: str) -> dict[str, object]:
    try:
        return read_config_file(path)
    except FileNotFoundError as exc:
        raise NotFoundError(f"配置文件不存在: {path}") from exc
    except OSError as exc:
        raise AppError(f"读取配置失败: {path}") from exc
```

## 7. 可测试性设计

```python
# 依赖注入便于替换与测试
class PaymentService:
    def __init__(self, gateway: PaymentGateway, clock: Clock) -> None:
        self._gateway = gateway
        self._clock = clock


# 纯函数边界便于验证
def calculate_discount(total: int, rate: float) -> int:
    return int(total * rate)


# 避免隐式全局状态
class AppConfig:
    def __init__(self, settings: dict[str, str]) -> None:
        self._settings = settings

    def get(self, key: str, default: str | None = None) -> str | None:
        return self._settings.get(key, default)
```

## 8. 模块组织

```text
project/
├── pyproject.toml
├── README.md
├── tests/
├── src/
│   └── myapp/
│       ├── api/
│       ├── domain/
│       ├── services/
│       ├── infrastructure/
│       └── shared/
└── scripts/
```

- 按职责拆分文件，而不是机械按技术层切碎。
- 会一起修改的代码尽量放近。
- 单个文件如果已经大到难以整体理解，优先考虑按职责拆分。

## 9. 配置与资源管理

```python
from dataclasses import dataclass
import os


@dataclass
class DatabaseConfig:
    url: str
    pool_size: int = 10


def load_database_config() -> DatabaseConfig:
    return DatabaseConfig(
        url=os.environ["DATABASE_URL"],
        pool_size=int(os.getenv("DB_POOL_SIZE", "10")),
    )


class SessionManager:
    def __enter__(self) -> Session:
        self._session = create_session()
        return self._session

    def __exit__(self, exc_type, exc, tb) -> None:
        self._session.close()
```

## 10. 日志设计

```python
import logging


logger = logging.getLogger(__name__)


def log_user_login(user_id: str, email: str) -> None:
    logger.info("user login", extra={"user_id": user_id, "email": mask_email(email)})
```

- 日志记录行为和关键上下文，不记录明文敏感数据。
- 优先结构化字段，避免只堆字符串。

## 11. 设计清单

- [ ] 模块职责清晰，没有上帝类/上帝函数
- [ ] 数据模型和服务对象边界明确
- [ ] 优先组合，而不是为了复用少量逻辑引入深继承
- [ ] 依赖通过构造参数或显式接口注入
- [ ] 异常类型能表达领域语义
- [ ] 配置和资源生命周期显式管理
- [ ] 避免全局状态，保留可测试边界
- [ ] 文件大小和职责仍然可控

## 12. 禁止事项

- ❌ 为简单问题设计过重抽象
- ❌ 使用继承替代清晰组合
- ❌ 一个模块承担过多职责
- ❌ 业务代码直接依赖全局配置和全局状态
- ❌ 捕获异常后静默吞掉
- ❌ 返回结构随意漂移，缺少清晰契约
