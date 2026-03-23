---
name: "secure-development"
description: "安全开发规范指南。Cover input validation, SQL injection prevention, XSS/CSRF protection, authentication, authorization, sensitive data protection, secure coding practices, and security best practices. Invoke when: implementing authentication/authorization, handling sensitive data, validating user input, preventing security vulnerabilities, writing security-related code, conducting security reviews, writing technical documentation for security protocols, or discussing secure architecture decisions."
---

# 安全开发规范

## 概述

本规范定义了企业级软件开发中的安全编码标准，涵盖输入验证、SQL注入防护、XSS/CSRF防护、认证授权、敏感数据保护和依赖安全等内容。

## 输入验证（SEC-001~SEC-003）

### 核心原则

**所有外部输入必须验证**，包括用户输入、API请求、文件上传、数据库记录、第三方回调等。

### 白名单验证（SEC-002）

输入验证应采用白名单策略（明确允许什么），而非黑名单策略（禁止什么）。

```javascript
// 白名单验证文件类型
const ALLOWED_FILE_TYPES = ['image/jpeg', 'image/png', 'application/pdf'];

function validateFileType(file) {
    if (!ALLOWED_FILE_TYPES.includes(file.mimetype)) {
        throw new Error('不支持的文件类型');
    }
}
```

### 路径和标识符验证（SEC-003）

```java
@GetMapping("/api/documents/{docId}")
public ResponseEntity<?> getDocument(@PathVariable String docId) {
    if (!docId.matches("^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$")) {
        return ResponseEntity.badRequest().body("无效的文档ID格式");
    }
    if (!documentService.hasAccess(currentUser, docId)) {
        return ResponseEntity.status(403).body("无权访问");
    }
    return ResponseEntity.ok(documentService.get(docId));
}
```

## SQL注入防护（SEC-010~SEC-012）

### 强制使用参数化查询（SEC-010）

```python
# 正确：使用参数化查询
query = text("SELECT * FROM users WHERE username = :username")
result = await db.execute(query, {"username": username})

# 错误：字符串拼接SQL（严重安全漏洞！）
query = f"SELECT * FROM users WHERE username = '{username}'"
```

```java
// JDBC PreparedStatement
String sql = "SELECT * FROM users WHERE username = ?";
PreparedStatement stmt = connection.prepareStatement(sql);
stmt.setString(1, username);
```

## XSS防护（SEC-020~SEC-022）

### 上下文编码（SEC-020）

根据输出位置选择合适的编码方式：

```javascript
function escapeHtml(text) {
    const div = document.createElement('div');
    div.textContent = text;
    return div.innerHTML;
}

// HTML内容上下文
element.innerHTML = escapeHtml(userInput);

// JavaScript上下文
const jsSafe = JSON.stringify(userInput);

// URL上下文
const urlSafe = encodeURIComponent(userInput);
```

### Cookie安全设置（SEC-022）

```python
response.set_cookie(
    'session_id',
    value=generate_session_token(),
    httponly=True,      # 禁止JavaScript访问
    secure=True,        # 仅通过HTTPS传输
    samesite='Strict',  # 防止CSRF
    max_age=3600
)
```

## CSRF防护（SEC-030~SEC-032）

### CSRF Token机制（SEC-030）

所有状态改变操作必须验证CSRF Token：

```python
# Flask-WTF CSRF保护
from flask_wtf.csrf import CSRFProtect
csrf = CSRFProtect(app)

# 表单中嵌入Token
"""
<input type="hidden" name="csrf_token" value="{{ csrf_token() }}">
"""

# AJAX请求携带Token
"""
headers: {'X-CSRFToken': '{{ csrf_token() }}'}
"""
```

### SameSite Cookie属性（SEC-031）

- `Strict`：完全禁止跨站发送Cookie
- `Lax`：允许安全的跨站GET请求
- `None`：允许跨站发送，但必须配合Secure

## 认证与会话管理（SEC-040~SEC-043）

### 强密码哈希（SEC-040）

```python
import bcrypt

def hash_password(password: str) -> str:
    salt = bcrypt.gensalt(rounds=12)
    hashed = bcrypt.hashpw(password.encode('utf-8'), salt)
    return hashed.decode('utf-8')

def verify_password(password: str, hashed: str) -> bool:
    return bcrypt.checkpw(password.encode('utf-8'), hashed.encode('utf-8'))
```

**禁止使用**：MD5、SHA1等弱哈希算法。

### 会话管理（SEC-041）

```python
class SessionManager:
    SESSION_TIMEOUT = 1800  # 30分钟
    ABSOLUTE_TIMEOUT = 86400  # 24小时
    
    def create_session(self, user_id: str) -> dict:
        return {
            'session_id': secrets.token_urlsafe(32),
            'user_id': user_id,
            'created_at': time.time(),
            'last_activity': time.time(),
        }
```

### JWT安全使用（SEC-042）

```python
JWT_SECRET = secrets.token_urlsafe(64)  # 512位密钥
JWT_ALGORITHM = 'HS256'
ACCESS_TOKEN_EXPIRE = 15  # 15分钟

payload = {
    'sub': user_id,
    'exp': datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE),
    'jti': secrets.token_urlsafe(16)
}
```

**禁止**：
- 使用弱签名算法
- Token永不过期
- 在Token中存储敏感信息

## 授权控制（SEC-050~SEC-053）

### RBAC权限模型（SEC-050）

```python
class Permission(Enum):
    USER_READ = "user:read"
    USER_WRITE = "user:write"
    ADMIN_ACCESS = "admin:access"

ROLE_PERMISSIONS = {
    "guest": [Permission.USER_READ],
    "user": [Permission.USER_READ, Permission.USER_WRITE],
    "admin": list(Permission)
}

def require_permission(permission: Permission):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            if permission not in user_permissions:
                raise HTTPException(403, "权限不足")
            return func(*args, **kwargs)
        return wrapper
    return decorator
```

### 最小权限原则（SEC-051）

```python
# 数据库连接使用最小权限
DB_READ_USER = {'permissions': ['SELECT']}
DB_WRITE_USER = {'permissions': ['SELECT', 'INSERT', 'UPDATE', 'DELETE']}
```

### 资源级别鉴权（SEC-053）

```python
@app.get("/api/orders/{order_id}")
async def get_order(order_id: str, user: User = Depends(get_current_user)):
    order = await Order.query.filter(
        Order.id == order_id,
        Order.user_id == user.id  # 确保只能访问自己的订单
    ).first()
    if not order:
        raise HTTPException(404, "订单不存在或无权访问")
    return order
```

## 敏感数据保护（SEC-060~SEC-063）

### 加密存储（SEC-060）

```python
from cryptography.fernet import Fernet

def encrypt_sensitive_data(data: str) -> str:
    encrypted = cipher_suite.encrypt(data.encode('utf-8'))
    return encrypted.decode('utf-8')

def decrypt_sensitive_data(encrypted_data: str) -> str:
    decrypted = cipher_suite.decrypt(encrypted_data.encode('utf-8'))
    return decrypted.decode('utf-8')
```

### 日志脱敏（SEC-062）

```python
class SensitiveDataFilter(logging.Filter):
    SENSITIVE_FIELDS = ['password', 'token', 'api_key', 'credit_card', 'phone', 'email']
    
    PATTERNS = {
        'phone': (r'\b1[3-9]\d{9}\b', '1****'),
        'email': (r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b', '***@***.com'),
    }
    
    def filter(self, record):
        record.msg = self._sanitize(str(record.msg))
        return True
```

## 依赖安全（SEC-070~SEC-072）

### 漏洞扫描（SEC-070）

```bash
# Node.js
npm audit
npm audit fix

# Python
pip install safety
safety check
```

### 版本锁定（SEC-071）

```json
{
  "dependencies": {
    "express": "4.18.2",
    "lodash": "^4.17.21"
  }
}
```

## 错误处理（SEC-080~SEC-082）

### 安全错误处理（SEC-080）

```python
@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    error_id = generate_error_id()
    
    logger.error(f"Unhandled exception [ID: {error_id}]: {str(exc)}", exc_info=True)
    
    return JSONResponse(
        status_code=500,
        content={
            "error": {
                "code": "ERR_INTERNAL",
                "message": "服务器内部错误，请稍后重试",
                "error_id": error_id
            }
        }
    )
```

**禁止**：向用户暴露堆栈跟踪、SQL语句等敏感信息。

## 文件上传安全（SEC-090~SEC-092）

### 多层次文件验证（SEC-090）

```python
import magic

def validate_file(file):
    mime = magic.from_buffer(file.read(2048), mime=True)
    file.seek(0)
    
    if mime not in ALLOWED_FILE_TYPES:
        raise HTTPException(400, "不支持的文件类型")
    
    # 验证扩展名与MIME类型匹配
    file_ext = Path(file.filename).suffix.lower()
    if file_ext not in ALLOWED_EXTENSIONS[mime]:
        raise HTTPException(400, "文件扩展名与类型不匹配")
```

### 安全存储（SEC-091）

```python
def store_file(file, original_filename: str) -> dict:
    file_id = str(uuid.uuid4())
    safe_ext = Path(original_filename).suffix.lower()
    stored_filename = f"{file_id}{safe_ext}"
    
    # 按日期分目录存储
    storage_dir = os.path.join(UPLOAD_ROOT, datetime.now().strftime("%Y/%m/%d"))
    os.makedirs(storage_dir, exist_ok=True)
    
    file_path = os.path.join(storage_dir, stored_filename)
    os.chmod(file_path, 0o644)  # 设置安全权限
```

## API安全（SEC-100~SEC-102）

### 速率限制（SEC-100）

```python
from flask_limiter import Limiter

limiter = Limiter(
    key_func=get_remote_address,
    default_limits=["200 per day", "50 per hour"]
)

@app.route("/login")
@limiter.limit("5 per minute")  # 登录接口严格限制
def login():
    pass
```

### 请求签名（SEC-101）

```python
def generate_signature(secret: str, method: str, path: str, 
                       timestamp: str, body: str = '') -> str:
    message = f"{method}\n{path}\n{timestamp}\n{body}"
    signature = hmac.new(
        secret.encode('utf-8'),
        message.encode('utf-8'),
        hashlib.sha256
    ).digest()
    return base64.b64encode(signature).decode('utf-8')
```

## 密钥管理（SEC-110~SEC-113）

### 禁止硬编码（SEC-110）

```python
# 正确：从环境变量读取
DATABASE_URL = os.environ['DATABASE_URL']
SECRET_KEY = os.environ['SECRET_KEY']

# 错误：硬编码密钥（绝对禁止！）
DATABASE_PASSWORD = "MyP@ssw0rd123"
API_KEY = "sk_live_1234567890abcdef"
```

### 配置分离（SEC-112）

```
project/
├── config/
│   ├── base.py           # 基础配置
│   ├── development.py    # 开发环境
│   └── production.py      # 生产环境（不提交到版本控制）
├── .env.example          # 环境变量示例（提交）
└── .env.production       # 生产环境变量（不提交）
```

## 安全检查清单

- [ ] 所有外部输入都经过验证
- [ ] 数据库查询使用参数化查询
- [ ] 用户输出都进行了适当的编码
- [ ] 敏感Cookie设置了HttpOnly、Secure和SameSite
- [ ] 状态改变操作有CSRF保护
- [ ] 密码使用bcrypt/Argon2等强哈希算法
- [ ] 会话有合理的超时机制
- [ ] API端点有适当的权限检查
- [ ] 敏感数据已加密存储
- [ ] 日志中没有记录敏感信息
- [ ] 错误信息不暴露系统内部细节
- [ ] 文件上传有类型和大小限制
- [ ] API实施了速率限制
- [ ] 没有硬编码的密钥或密码
- [ ] 依赖库已进行漏洞扫描
