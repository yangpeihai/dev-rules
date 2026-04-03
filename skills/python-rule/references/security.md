# Python 安全编码规范

## AI 使用提示

- 当任务涉及输入校验、SQL、路径、密码、密钥、日志、反序列化、HTTP 调用或并发共享状态时，先看本文件。
- 如果代码把外部输入直接拼接进 SQL、路径、命令、模板或日志，默认视为高优先级风险。
- 当安全与便捷冲突时，默认优先安全，再说明权衡。

> AI Code 遵循规约，编写安全可靠的 Python 代码。

## 1. 输入验证

```python
import re


def create_user(username: str, email: str) -> dict[str, str]:
    if not 3 <= len(username) <= 32:
        raise ValidationError("用户名长度必须在 3-32 字符之间")

    if not re.fullmatch(r"[a-zA-Z0-9_]+", username):
        raise ValidationError("用户名只能包含字母、数字和下划线")

    if not re.fullmatch(r"[^@]+@[^@]+\.[^@]+", email):
        raise ValidationError("邮箱格式无效")

    return {"username": username, "email": email}
```

- 所有外部输入默认校验长度、格式、枚举范围和边界。
- 框架已提供验证能力时，优先使用框架的正式入口，而不是重复散写判断。

## 2. SQL 注入防护

```python
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))


user = session.query(User).filter(User.id == user_id).first()


# ❌ 禁止字符串拼接
# query = f"SELECT * FROM users WHERE id = {user_id}"
```

- 数据库访问默认参数化查询或使用 ORM 参数绑定。
- 外部输入不直接进入 SQL 字符串。

## 3. 命令与代码执行风险

```python
import shlex
import subprocess


subprocess.run(["ls", "-la", "/tmp"], check=True)


safe_arg = shlex.quote(user_input)
subprocess.run(f"cat {safe_arg}", shell=True, check=True)


# ❌ 禁止
# eval(user_input)
# exec(user_code)
# os.system("cat " + user_input)
```

- 默认禁止 `eval`、`exec`、不受控的 shell 拼接。
- 如果必须使用 shell，先确认参数边界并做引用保护。

## 4. 路径与文件安全

```python
from pathlib import Path


def read_file(base_dir: str, filename: str) -> bytes:
    base = Path(base_dir).resolve()
    target = (base / filename).resolve()

    if not str(target).startswith(str(base)):
        raise SecurityError("非法的文件路径")

    return target.read_bytes()
```

- 文件读写必须限制在预期目录内。
- 上传文件名、导出路径、压缩包解压路径都要做边界校验。

## 5. 密码与密钥

```python
import bcrypt
import os


def hash_password(password: str) -> str:
    return bcrypt.hashpw(password.encode(), bcrypt.gensalt()).decode()


SECRET_KEY = os.environ["SECRET_KEY"]
DATABASE_URL = os.environ["DATABASE_URL"]
```

- 密码使用成熟 KDF/哈希方案，如 `bcrypt`、`argon2`。
- 密钥、令牌、数据库密码从环境变量或安全配置注入，不硬编码。
- 不要把密码哈希和普通摘要混为一谈。

## 6. 反序列化与模板安全

```python
import ast
import json
import yaml


data = json.loads(payload)
config = yaml.safe_load(yaml_payload)
value = ast.literal_eval("[1, 2, 3]")


# ❌ 禁止对不可信输入使用
# pickle.loads(payload)
# yaml.load(payload, Loader=yaml.Loader)
```

- 不可信输入禁止 `pickle.loads`。
- YAML 使用 `safe_load`。
- 需要解析字面值时优先 `ast.literal_eval`，而不是 `eval`。

## 7. HTTP 与 TLS

```python
import requests


response = requests.get("https://api.example.com", timeout=(3, 10), verify=True)
response.raise_for_status()
```

- HTTPS 默认验证证书。
- 外部 HTTP 调用默认设置超时。
- 不因“调试方便”关闭证书校验并把代码带进正式环境。

## 8. 日志与错误信息

```python
import logging


logger = logging.getLogger(__name__)


logger.info("user login", extra={"user_id": user.id, "email": mask_email(user.email)})
```

```python
def authenticate(username: str, password: str) -> None:
    if not is_valid_login(username, password):
        raise AuthenticationError("用户名或密码错误")
```

- 日志默认脱敏，禁止记录密码、Token、密钥、原始身份证件号等敏感数据。
- 对外错误信息不泄露内部结构、SQL、密钥或攻击面细节。

## 9. 并发与共享状态安全

```python
import threading


class SafeCounter:
    def __init__(self) -> None:
        self._value = 0
        self._lock = threading.Lock()

    def increment(self) -> int:
        with self._lock:
            self._value += 1
            return self._value
```

- 线程或协程共享可变状态时，要有明确同步策略。
- 不要把“有 GIL”误当成所有复合操作都安全。

## 10. 资源与超时控制

```python
from contextlib import contextmanager


@contextmanager
def get_db_connection():
    conn = create_connection()
    try:
        yield conn
        conn.commit()
    except Exception:
        conn.rollback()
        raise
    finally:
        conn.close()
```

```python
import asyncio


async def fetch_with_timeout(url: str) -> str:
    async with asyncio.timeout(5):
        return await fetch(url)
```

- 外部连接、数据库事务、文件句柄要显式释放。
- 外部调用默认设置超时，不无限等待。

## 11. 安全测试

```python
import pytest


def test_sql_injection_protection() -> None:
    malicious = "admin' OR '1'='1"
    users = find_by_username(malicious)
    assert len(users) <= 1
```

```python
def test_path_traversal_protection() -> None:
    with pytest.raises(SecurityError):
        read_file("/safe/base", "../../../etc/passwd")
```

- 测试使用假的密钥、假的数据库配置、隔离的测试环境。
- 对 SQL 注入、路径遍历、权限边界、错误信息泄露等高风险点优先补测试。
- 不在测试代码里硬编码真实生产凭证。

## 12. 安全清单

- [ ] 外部输入已做边界校验
- [ ] 数据库访问使用参数化查询
- [ ] 文件路径限制在预期目录内
- [ ] 未对不可信输入使用 `pickle.loads`、`eval`、`exec`
- [ ] 密码使用成熟哈希方案
- [ ] 密钥和凭证未硬编码
- [ ] 日志与错误信息已脱敏
- [ ] HTTP 调用启用证书校验并设置超时
- [ ] 共享可变状态有明确同步策略
- [ ] 安全关键路径有对应测试

## 13. 禁止事项

- ❌ 字符串拼接 SQL
- ❌ 对不可信输入执行代码或反序列化
- ❌ 硬编码密钥、令牌、数据库密码
- ❌ 日志输出敏感明文
- ❌ 关闭证书校验进入正式代码
- ❌ 共享可变状态却没有同步方案
