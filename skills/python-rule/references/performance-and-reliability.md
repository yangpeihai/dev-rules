# Python 性能与可靠性规范

> AI Code 遵循规约，编写高效可靠的代码。

---

## 🤖 AI 使用提示

**何时优先查阅**：当你处理并发选型、I/O、资源管理、性能热点、可靠性治理、超时重试和测试策略时。

**阅读顺序建议**：
1. 先判断问题是“真实性能瓶颈”还是“可靠性风险”。
2. 没有 profiling、压测或明确证据时，先选简单稳妥方案，不急于复杂优化。
3. 只读取与当前问题直接相关的章节，例如并发、I/O、资源管理或测试。

**默认决策倾向**：
- I/O 密集型：优先考虑 `asyncio` 或线程池
- CPU 密集型：优先考虑进程池
- 大数据读取：优先惰性迭代、流式处理
- 外部调用：默认设置超时；是否重试取决于幂等性和失败模式
- 测试：优先覆盖关键路径、错误分支和回归风险点，不机械追求固定覆盖率

---

## 1. 内存优化

```python
# ✅ 使用生成器处理大数据集
def read_large_file(path: str):
    """逐行读取大文件。"""
    with open(path, 'r') as f:
        for line in f:
            yield line.strip()

# 内存友好：一次只加载一行
for line in read_large_file('large.txt'):
    process(line)

# ❌ 避免一次性加载所有内容
# lines = open('large.txt').readlines()  # 内存占用大

# ✅ 使用生成器表达式
sum_squares = sum(x**2 for x in range(1000000))  # 惰性求值

# ❌ 列表推导式占用更多内存
# sum_squares = sum([x**2 for x in range(1000000)])

# ✅ 使用 __slots__ 减少实例内存
class Point:
    __slots__ = ['x', 'y']  # 节省约 40-50% 内存

    def __init__(self, x, y):
        self.x = x
        self.y = y

# ✅ 使用 array 模块处理数值数组
import array
nums = array.array('i', [1, 2, 3, 4, 5])  # 比 list 更紧凑

# ✅ 使用 numpy 处理大规模数值计算
import numpy as np
arr = np.array([1, 2, 3, 4, 5])
```

## 2. 字符串操作优化

```python
# ✅ 使用 join 拼接字符串
parts = ['hello', 'world', 'python']
result = '-'.join(parts)

# ❌ 避免在循环中使用 + 拼接
# result = ''
# for part in parts:
#     result += part  # O(n²) 时间复杂度

# ✅ 使用 f-string（Python 3.6+）
name = "Alice"
age = 25
message = f"{name} is {age} years old"

# ✅ 使用 str.maketrans 进行批量字符替换
trans = str.maketrans({'a': '1', 'b': '2', 'c': '3'})
result = "abc".translate(trans)

# ✅ 使用 startswith/endswith 替代正则
if filename.startswith(('test_', 'spec_')):
    process(filename)
```

## 3. 数据结构选择

```python
# ✅ 使用 set 进行成员检查（O(1) vs O(n)）
valid_users = {'alice', 'bob', 'charlie'}
if username in valid_users:  # 快速查找
    process(username)

# ❌ 列表查找慢
# valid_users = ['alice', 'bob', 'charlie']
# if username in valid_users:  # O(n)

# ✅ 使用 dict 进行快速查找
user_map = {user.id: user for user in users}
user = user_map.get(user_id)

# ✅ 使用 deque 处理队列（双端队列）
from collections import deque

queue = deque(maxlen=1000)
queue.append(item)      # O(1)
queue.appendleft(item)  # O(1)
queue.pop()             # O(1)

# ✅ 使用 defaultdict 简化代码
from collections import defaultdict

counts = defaultdict(int)
for item in items:
    counts[item] += 1  # 无需检查键是否存在

# ✅ 使用 Counter 计数
from collections import Counter

top_items = Counter(items).most_common(10)

# ✅ 使用 lru_cache 缓存函数结果
from functools import lru_cache

@lru_cache(maxsize=128)
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
```

## 4. I/O 优化

```python
# ✅ 使用缓冲读取
with open('large.txt', 'r', buffering=8192) as f:
    for line in f:
        process(line)

# ✅ 批量写入
with open('output.txt', 'w') as f:
    f.writelines(lines)  # 比逐行写入快

# ✅ 异步 I/O（高并发场景）
import aiohttp
import asyncio

async def fetch_all(urls: list[str]) -> list:
    async with aiohttp.ClientSession() as session:
        tasks = [session.get(url) for url in urls]
        responses = await asyncio.gather(*tasks)
        return [await r.text() for r in responses]

# ✅ 使用连接池
from sqlalchemy import create_engine

# 创建连接池
engine = create_engine(
    'postgresql://user:pass@localhost/db',
    pool_size=10,        # 连接池大小
    max_overflow=20,     # 最大溢出连接数
    pool_timeout=30,     # 获取连接超时
    pool_recycle=3600,   # 连接回收时间
)

# ✅ 使用上下文管理器处理数据库会话
from contextlib import contextmanager

@contextmanager
def get_session():
    session = Session()
    try:
        yield session
        session.commit()
    except Exception:
        session.rollback()
        raise
    finally:
        session.close()
```

## 5. 并发性能

```python
# ✅ I/O 密集型：使用 asyncio 或 ThreadPoolExecutor
import asyncio
import aiohttp

async def fetch(session, url):
    async with session.get(url) as response:
        return await response.text()

async def fetch_all(urls):
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(session, url) for url in urls]
        return await asyncio.gather(*tasks)

# 或使用线程池
from concurrent.futures import ThreadPoolExecutor

def fetch_urls(urls: list[str]) -> list:
    with ThreadPoolExecutor(max_workers=10) as executor:
        return list(executor.map(fetch_single, urls))

# ✅ CPU 密集型：使用 ProcessPoolExecutor
from concurrent.futures import ProcessPoolExecutor

def process_large_data(data_chunks):
    with ProcessPoolExecutor() as executor:
        results = list(executor.map(process_chunk, data_chunks))
    return results

# ✅ 使用 GIL 友好的数据结构
# 列表和字典在 GIL 下是线程安全的（原子操作）
# 但复合操作需要锁
```

## 6. 算法优化

```python
# ✅ 使用内置函数（C 实现，更快）
total = sum(numbers)  # 比循环快

# ✅ 使用列表推导式（比 for 循环快）
squares = [x**2 for x in range(1000)]

# ✅ 使用 bisect 进行二分查找
import bisect

sorted_list = [1, 3, 5, 7, 9]
index = bisect.bisect_left(sorted_list, 5)  # O(log n)

# ✅ 使用 heapq 处理前 N 个元素
import heapq

top_10 = heapq.nlargest(10, large_list)
bottom_10 = heapq.nsmallest(10, large_list)

# ✅ 使用 itertools 优化迭代
from itertools import chain, islice, cycle

# 链式迭代
for item in chain(list1, list2, list3):
    process(item)

# 切片迭代（不创建新列表）
for item in islice(large_iterable, 100, 200):
    process(item)
```

## 7. 性能分析

```python
# ✅ 使用 cProfile 分析性能
import cProfile
import pstats

def main():
    # 需要优化的代码
    pass

# 运行分析
profiler = cProfile.Profile()
profiler.enable()
main()
profiler.disable()

# 输出结果
stats = pstats.Stats(profiler)
stats.sort_stats('cumulative').print_stats(10)

# ✅ 使用 timeit 基准测试
import timeit

time = timeit.timeit(
    'sum(range(1000))',
    number=10000
)

# ✅ 使用 memory_profiler 分析内存
# pip install memory_profiler
from memory_profiler import profile

@profile
def my_func():
    a = [1] * (10**6)
    b = [2] * (10**7)
    del b
    return a
```

## 8. 可靠性设计

```python
# ✅ 使用重试机制
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=4, max=10)
)
def fetch_with_retry(url):
    response = requests.get(url, timeout=10)
    response.raise_for_status()
    return response.json()

# ✅ 使用熔断器
from circuitbreaker import circuit

@circuit(failure_threshold=5, recovery_timeout=30)
def external_api_call():
    return requests.get('https://api.example.com').json()

# ✅ 实现优雅降级
def get_user_data(user_id):
    try:
        return fetch_from_cache(user_id) or fetch_from_db(user_id)
    except DatabaseError:
        # 数据库不可用时返回缓存数据
        return fetch_from_cache(user_id, allow_stale=True)
    except Exception as e:
        logger.error(f"获取用户数据失败：{e}")
        return None
```

## 9. 资源管理

```python
# ✅ 使用 contextlib 管理资源
from contextlib import contextmanager, ExitStack

@contextmanager
def managed_resources():
    with ExitStack() as stack:
        file1 = stack.enter_context(open('file1.txt'))
        file2 = stack.enter_context(open('file2.txt'))
        db = stack.enter_context(get_db_connection())
        yield file1, file2, db

# ✅ 限制资源使用
import resource

# 限制 CPU 时间
resource.setrlimit(resource.RLIMIT_CPU, (10, 10))

# 限制内存
resource.setrlimit(resource.RLIMIT_AS, (1024*1024*1024, 1024*1024*1024))

# ✅ 使用信号处理
import signal

def timeout_handler(signum, frame):
    raise TimeoutError("操作超时")

signal.signal(signal.SIGALRM, timeout_handler)
signal.alarm(30)  # 30 秒超时

try:
    risky_operation()
finally:
    signal.alarm(0)  # 取消闹钟
```

## 10. 日志和监控

```python
import logging
import time
from functools import wraps

# ✅ 配置结构化日志
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

logger = logging.getLogger(__name__)

# ✅ 性能监控装饰器
def timing_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        try:
            return func(*args, **kwargs)
        finally:
            elapsed = time.perf_counter() - start
            logger.info(f"{func.__name__} 执行时间：{elapsed:.4f}s")
    return wrapper

@timing_decorator
def slow_function():
    time.sleep(1)

# ✅ 上下文性能日志
from contextlib import contextmanager

@contextmanager
def timing_context(operation: str):
    start = time.perf_counter()
    try:
        yield
    finally:
        elapsed = time.perf_counter() - start
        logger.info(f"{operation} 耗时：{elapsed:.4f}s")

with timing_context("数据处理"):
    process_data()
```

## 11. 单元测试规范

### 测试结构

```python
# test_user_service.py
import pytest
from unittest.mock import Mock, patch

class TestUserService:
    """UserService 测试类。"""

    @pytest.fixture
    def mock_db(self):
        """数据库 mock 夹具。"""
        return Mock()

    @pytest.fixture
    def user_service(self, mock_db):
        """创建测试服务。"""
        from user_service import UserService
        return UserService(mock_db)

    def test_create_user_success(self, user_service, mock_db):
        """测试创建用户成功。"""
        # Arrange
        user_data = {
            'username': 'testuser',
            'email': 'test@example.com',
            'password': 'SecurePass123'
        }

        # Act
        result = user_service.create(user_data)

        # Assert
        assert result is not None
        assert result.username == 'testuser'
        mock_db.insert.assert_called_once()

    def test_create_user_duplicate_email(self, user_service, mock_db):
        """测试创建用户 - 重复邮箱。"""
        # Arrange
        mock_db.find_by_email.return_value = {'id': 1}

        # Act & Assert
        with pytest.raises(DuplicateError):
            user_service.create({'email': 'existing@example.com'})
```

### 测试覆盖率

```python
# pytest.ini 或 pyproject.toml 配置
# [tool.pytest.ini_options]
# addopts = "--cov=src --cov-report=html --cov-report=term"

# 运行测试并生成覆盖率报告
# pytest --cov=src --cov-report=html

# 最低覆盖率要求
# pytest --cov=src --cov-fail-under=80
```

### 参数化测试

```python
@pytest.mark.parametrize("input,expected", [
    (0, 1),
    (1, 1),
    (5, 120),
    (10, 3628800),
])
def test_factorial(input, expected):
    assert factorial(input) == expected
```

### Mock 最佳实践

```python
from unittest.mock import Mock, patch, MagicMock

# 测试外部 API 调用
@patch('requests.get')
def test_fetch_data(mock_get):
    mock_get.return_value.json.return_value = {'status': 'ok'}

    result = fetch_data('https://api.example.com')

    assert result == {'status': 'ok'}
    mock_get.assert_called_once_with('https://api.example.com', timeout=10)

# 测试异常处理
@patch('module.risky_operation')
def test_risky_operation_failure(mock_op):
    mock_op.side_effect = ConnectionError("连接失败")

    with pytest.raises(ServiceUnavailableError):
        call_service()
```

### 测试隔离

```python
# 每个测试使用独立的数据
@pytest.fixture
def fresh_database():
    """创建独立的测试数据库。"""
    db = create_test_database()
    yield db
    drop_test_database(db)

@pytest.fixture(autouse=True)
def reset_cache():
    """每个测试前重置缓存。"""
    cache.clear()
    yield
    cache.clear()
```

## 12. 性能清单

- [ ] 大数据集使用生成器而非列表
- [ ] 使用合适的数据结构（set 用于查找，deque 用于队列）
- [ ] 字符串拼接使用 join
- [ ] I/O 操作使用缓冲和批处理
- [ ] 高并发场景使用 asyncio 或线程池
- [ ] CPU 密集型任务使用进程池
- [ ] 使用缓存（lru_cache）避免重复计算
- [ ] 外部调用设置超时和重试
- [ ] 实现熔断器防止级联故障
- [ ] 添加性能监控和日志
- [ ] 测试覆盖率达到 80% 以上
- [ ] 使用性能分析工具识别瓶颈

## 13. 禁止事项

- ❌ 在循环中进行字符串拼接
- ❌ 使用列表进行频繁的成员检查
- ❌ 一次性加载大文件到内存
- ❌ 无限制地创建线程/进程
- ❌ 外部调用不设置超时
- ❌ 不使用连接池频繁创建数据库连接
- ❌ 忽略异常继续重试（应使用熔断器）
- ❌ 在生产环境使用 print 调试
- ❌ 测试覆盖率低于 80%
