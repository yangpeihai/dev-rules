# Python 安全编码规范

> AI Code 遵循规约，编写安全可靠的代码。

---

## 🤖 AI 使用说明

**何时查阅本规范**：当你处理用户输入、数据库查询、密码存储、文件操作、外部 API 调用、并发编程、记录日志、序列化/反序列化时。

**核心规则（必须遵守）- 违反将导致严重漏洞**：
1. 所有外部输入必须验证
2. SQL 查询必须参数化（禁止字符串拼接）
3. 密码必须使用 bcrypt/argon2 哈希（禁止 MD5/SHA1）
4. 禁止硬编码密钥（从环境变量读取）
5. 禁止记录敏感信息到日志
6. 禁止反序列化不可信数据（pickle/eval）

**快速决策**：
- 处理用户输入 → 第 1 节（输入验证）
- 数据库查询 → 第 2 节（SQL 注入防护）
- 密码处理 → 第 5 节（密码存储）
- 文件操作 → 第 4 节（路径遍历防护）
- 密钥管理 → 第 6 节（敏感信息处理）
- 加密需求 → 第 7 节（加密算法）
- 并发场景 → 第 10 节（并发安全）

---

## 1. 输入验证

```python
# 验证所有外部输入
from typing import Optional
import re

def create_user(username: str, email: str) -> Optional[dict]:
    """创建用户并验证输入。"""
    # 长度验证
    if not (3 <= len(username) <= 32):
        raise ValidationError("用户名长度必须在 3-32 字符之间")

    # 格式验证
    if not re.match(r'^[a-zA-Z0-9_]+$', username):
        raise ValidationError("用户名只能包含字母、数字和下划线")

    # 邮箱验证
    email_pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    if not re.match(email_pattern, email):
        raise ValidationError("邮箱格式无效")

    return {"username": username, "email": email}

# 使用 Pydantic 进行数据验证（推荐）
from pydantic import BaseModel, EmailStr, Field, validator

class UserCreate(BaseModel):
    username: str = Field(..., min_length=3, max_length=32)
    email: EmailStr
    password: str = Field(..., min_length=8)

    @validator('username')
    def username_alphanumeric(cls, v):
        if not v.isalnum():
            raise ValueError('用户名必须是字母数字')
        return v
```

## 2. SQL 注入防护

```python
# ✅ 使用参数化查询
# SQLite
cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))

# PostgreSQL/MySQL (psycopg2)
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))

# SQLAlchemy ORM
user = session.query(User).filter(User.id == user_id).first()

# SQLAlchemy Core
stmt = select(User).where(User.id == bindparam('user_id'))
result = connection.execute(stmt, {'user_id': user_id})

# ❌ 禁止字符串拼接/格式化
# query = "SELECT * FROM users WHERE id = '%s'" % user_id  # 危险！
# query = f"SELECT * FROM users WHERE id = {user_id}"      # 危险！
```

## 3. 命令注入防护

```python
import subprocess
import shlex

# ✅ 使用列表参数，避免 shell 解释
subprocess.run(['ls', '-la', '/home'], check=True)

# ✅ 如需 shell 功能，使用引号保护参数
user_input = "file.txt; rm -rf /"
safe_input = shlex.quote(user_input)
subprocess.run(f"cat {safe_input}", shell=True, check=True)

# ❌ 禁止直接拼接用户输入到命令
# os.system("cat " + user_input)  # 危险！
# subprocess.call(f"ls {user_input}", shell=True)  # 危险！
```

## 4. 路径遍历防护

```python
from pathlib import Path
import os

def read_file(base_dir: str, filename: str) -> bytes:
    """安全地读取文件。"""
    base = Path(base_dir).resolve()
    target = (base / filename).resolve()

    # 确保目标路径在基目录内
    if not str(target).startswith(str(base)):
        raise SecurityError("非法的文件路径")

    return target.read_bytes()

# 使用安全的路径操作
def save_upload(upload_dir: str, filename: str, content: bytes):
    """安全保存上传文件。"""
    # 清理文件名
    safe_name = os.path.basename(filename)

    # 生成随机文件名防止覆盖
    import uuid
    unique_name = f"{uuid.uuid4().hex}_{safe_name}"

    path = Path(upload_dir) / unique_name
    path.write_bytes(content)
```

## 5. 密码存储

```python
# ✅ 使用 bcrypt 或 argon2
import bcrypt

def hash_password(password: str) -> str:
    """哈希密码。"""
    salt = bcrypt.gensalt(rounds=12)
    hashed = bcrypt.hashpw(password.encode(), salt)
    return hashed.decode()

def verify_password(password: str, hashed: str) -> bool:
    """验证密码。"""
    return bcrypt.checkpw(password.encode(), hashed.encode())

# 使用 argon2（更安全的替代方案）
from argon2 import PasswordHasher
from argon2.exceptions import VerifyMismatchError

ph = PasswordHasher()

def hash_password_argon2(password: str) -> str:
    return ph.hash(password)

def verify_password_argon2(password: str, hashed: str) -> bool:
    try:
        ph.verify(hashed, password)
        return True
    except VerifyMismatchError:
        return False

# ❌ 禁止使用弱哈希
# hashlib.md5(password.encode())   # 危险！
# hashlib.sha1(password.encode())  # 危险！
# hashlib.sha256(password.encode())  # 不推荐用于密码
```

## 6. 敏感信息处理

```python
import os
import logging

# ✅ 从环境变量读取密钥
DATABASE_URL = os.getenv('DATABASE_URL')
SECRET_KEY = os.getenv('SECRET_KEY')
API_KEY = os.getenv('API_KEY')

# ❌ 禁止硬编码密钥
# SECRET_KEY = "super-secret-key-123"  # 危险！
# API_KEY = "sk-live-abcdef123456"    # 危险！

# ✅ 日志脱敏
def mask_email(email: str) -> str:
    """脱敏邮箱地址。"""
    if '@' not in email:
        return email
    name, domain = email.split('@', 1)
    masked_name = name[0] + '*' * (len(name) - 2) + name[-1] if len(name) > 2 else '*'
    return f"{masked_name}@{domain}"

logger = logging.getLogger(__name__)

# ✅ 记录脱敏后的信息
logger.info(f"用户登录：{mask_email(user.email)}, ID: {user.id}")

# ❌ 禁止记录敏感信息
# logger.info(f"用户登录：{user.email}, 密码：{password}")  # 危险！
```

## 7. 加密算法

```python
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
import os

# ✅ 使用 AES-GCM（认证加密）
def encrypt_aes_gcm(key: bytes, plaintext: bytes, associated_data: bytes = b"") -> tuple:
    """AES-GCM 加密。"""
    aesgcm = AESGCM(key)
    nonce = os.urandom(12)  # 96-bit nonce
    ciphertext = aesgcm.encrypt(nonce, plaintext, associated_data)
    return nonce, ciphertext

def decrypt_aes_gcm(key: bytes, nonce: bytes, ciphertext: bytes, associated_data: bytes = b"") -> bytes:
    """AES-GCM 解密。"""
    aesgcm = AESGCM(key)
    plaintext = aesgcm.decrypt(nonce, ciphertext, associated_data)
    return plaintext

# ✅ 生成安全随机数
import secrets

# 生成安全令牌
token = secrets.token_hex(32)

# 生成安全随机数
random_int = secrets.randbelow(100)

# ❌ 禁止使用弱算法
# from Crypto.Cipher import DES    # 危险！
# import random; random.randint()  # 不是加密安全的

# ❌ 禁止使用 math/random 模块用于安全场景
```

## 8. HTTPS/TLS

```python
import requests
import ssl

# ✅ 验证 SSL 证书
response = requests.get('https://api.example.com', verify=True)

# ✅ 自定义证书验证
response = requests.get(
    'https://api.example.com',
    verify='/path/to/cert.pem'
)

# ✅ 安全的 SSL 配置
import ssl
context = ssl.create_default_context()
context.check_hostname = True
context.verify_mode = ssl.CERT_REQUIRED

# ❌ 禁止跳过证书验证
# requests.get('https://...', verify=False)  # 危险！
# context.verify_mode = ssl.CERT_NONE       # 危险！
```

## 9. HTTP 安全头

```python
from flask import Flask, make_response

app = Flask(__name__)

@app.after_request
def add_security_headers(response):
    """添加安全响应头。"""
    response.headers['X-Frame-Options'] = 'DENY'
    response.headers['X-Content-Type-Options'] = 'nosniff'
    response.headers['X-XSS-Protection'] = '1; mode=block'
    response.headers['Strict-Transport-Security'] = 'max-age=31536000; includeSubDomains'
    response.headers['Content-Security-Policy'] = "default-src 'self'"
    response.headers['Referrer-Policy'] = 'strict-origin-when-cross-origin'
    return response

# 使用 Flask-Talisman（推荐）
# pip install flask-talisman
from flask_talisman import Talisman

talisman = Talisman(app,
    force_https=True,
    strict_transport_security=True,
    strict_transport_security_preload=True,
    strict_transport_security_max_age=31536000,
    content_security_policy={
        'default-src': "'self'",
        'script-src': "'self'",
    }
)
```

## 10. 并发安全

```python
import threading
from concurrent.futures import ThreadPoolExecutor

# ✅ 共享状态使用锁
class ThreadSafeCounter:
    def __init__(self):
        self._value = 0
        self._lock = threading.Lock()

    def increment(self) -> int:
        with self._lock:
            self._value += 1
            return self._value

    @property
    def value(self) -> int:
        with self._lock:
            return self._value

# ✅ 线程安全队列
from queue import Queue

job_queue = Queue()
job_queue.put(task)
task = job_queue.get()

# ❌ 无锁访问共享状态
# counter.value += 1  # 竞态条件！
```

## 11. 错误信息安全

```python
# ✅ 使用通用错误消息
def authenticate(username: str, password: str) -> bool:
    try:
        user = get_user(username)
        if not user or not verify_password(password, user.hashed_password):
            # 不泄露是用户名错误还是密码错误
            raise AuthenticationError("用户名或密码错误")
        return True
    except AuthenticationError:
        raise
    except Exception as e:
        logger.error(f"认证系统异常：{e}")
        raise AuthenticationError("认证失败，请稍后重试")

# ❌ 避免泄露敏感信息
# raise Exception(f"数据库错误：{e}, 用户：{username}, SQL: {sql}")
```

## 12. 资源管理

```python
# ✅ 使用上下文管理器确保资源释放
def process_file(path: str):
    with open(path, 'r') as f:
        data = f.read()
    # 文件自动关闭

# ✅ 数据库连接管理
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

# 使用
with get_db_connection() as conn:
    execute_query(conn)
```

## 13. 超时控制

```python
import requests
from requests.adapters import HTTPAdapter
from urllib3.util import Timeout

# ✅ 设置超时
# 连接超时 + 读取超时
response = requests.get('https://api.example.com', timeout=(5, 30))

# 使用会话配置默认超时
session = requests.Session()
adapter = HTTPAdapter(max_retries=3)
session.mount('https://', adapter)

response = session.get('https://api.example.com', timeout=10)

# asyncio 超时
import asyncio

async def fetch_with_timeout(url: str, timeout: float = 10.0):
    try:
        async with asyncio.timeout(timeout):
            return await fetch(url)
    except asyncio.TimeoutError:
        raise FetchError(f"请求超时：{url}")
```

## 14. 反序列化安全

```python
import json
import yaml

# ✅ 使用安全的序列化格式
data = json.loads(json_string)  # 安全

# 使用 YAML SafeLoader
data = yaml.safe_load(yaml_string)  # 安全

# ❌ 禁止反序列化不可信数据
# import pickle
# data = pickle.loads(untrusted_data)  # 危险！任意代码执行

# ❌ 禁止使用 eval/exec
# result = eval(user_input)    # 危险！
# exec(user_code)              # 危险！

# ✅ 如果必须使用 eval，使用 ast.literal_eval
import ast
result = ast.literal_eval(safe_string)  # 只能解析字面值
```

## 15. 安全清单

- [ ] 所有外部输入都经过验证
- [ ] 使用参数化查询防止 SQL 注入
- [ ] 使用安全的文件路径操作
- [ ] 密码使用 bcrypt/argon2 哈希
- [ ] 敏感数据脱敏后再记录日志
- [ ] 密钥从环境变量读取，不硬编码
- [ ] 使用 HTTPS 并验证证书
- [ ] 设置 HTTP 安全响应头
- [ ] 并发访问共享状态使用锁
- [ ] 外部调用设置超时
- [ ] 不使用 pickle 反序列化不可信数据
- [ ] 不使用 eval/exec 执行用户输入
- [ ] 定期更新依赖，修复安全漏洞

## 16. 测试安全

### 测试中的敏感数据处理

```python
# ❌ 禁止在测试中使用真实密钥
TEST_API_KEY = "sk-live-1234567890abcdef"
TEST_DB_PASSWORD = "ProdPass123!"

# ✅ 使用测试专用假值
TEST_API_KEY = "sk-test-0000000000000000"
TEST_DB_PASSWORD = "TestPassword123!"
TEST_JWT_SECRET = "test-secret-key-for-testing-only"

# ✅ 使用 pytest fixtures
import pytest

@pytest.fixture
def test_config():
    return {
        'api_key': 'sk-test-fake-key',
        'db_url': 'postgresql://localhost/test_db',
        'secret': 'test-secret',
    }
```

### 测试环境配置

```python
import os
import pytest

@pytest.fixture(autouse=True)
def setup_test_env(monkeypatch):
    """设置测试环境变量。"""
    monkeypatch.setenv('DATABASE_URL', 'postgresql://localhost/test_db')
    monkeypatch.setenv('SECRET_KEY', 'test-secret-key')
    monkeypatch.setenv('DEBUG', 'False')
    yield

# 测试中使用
def test_service(setup_test_env):
    db_url = os.getenv('DATABASE_URL')
    assert 'test_db' in db_url
```

### 安全相关测试

```python
import pytest

def test_password_hashing():
    """测试密码哈希。"""
    from auth import hash_password, verify_password

    password = "SecurePassword123!"
    hashed = hash_password(password)

    # 验证哈希不同（因为有 salt）
    assert hashed != password
    assert len(hashed) > len(password)

    # 验证密码正确
    assert verify_password(password, hashed)

    # 验证错误密码失败
    assert not verify_password("WrongPassword", hashed)

def test_sql_injection_protection():
    """测试 SQL 注入防护。"""
    from user_repo import find_by_username

    # 尝试 SQL 注入
    malicious_input = "admin' OR '1'='1"

    # 应该返回空或特定用户，而不是所有用户
    users = find_by_username(malicious_input)
    assert len(users) <= 1

def test_path_traversal_protection():
    """测试路径遍历防护。"""
    from file_service import read_file

    # 尝试路径遍历
    with pytest.raises(SecurityError):
        read_file('/safe/base', '../../../etc/passwd')
```

### 依赖安全检查

```bash
# 使用 pip-audit 检查漏洞
pip install pip-audit
pip-audit

# 使用 safety 检查
pip install safety
safety check

# 使用 pipreqs 生成 requirements
pip install pipreqs
pipreqs /path/to/project

# 在 CI 中运行
# .github/workflows/security.yml
- name: Check dependencies
  run: |
    pip install pip-audit
    pip-audit
```

### 禁止事项（测试安全）

- ❌ 在测试中使用真实生产密钥
- ❌ 在测试中连接生产数据库
- ❌ 测试代码中硬编码真实凭证
- ❌ 测试失败时泄露敏感信息
- ❌ 并发测试不使用线程安全措施
