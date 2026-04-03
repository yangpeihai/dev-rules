# Python 性能与可靠性规范

## AI 使用提示

- 当任务涉及并发选型、I/O、资源管理、超时重试、关键路径测试或性能优化时，先看本文件。
- 默认先保证正确性、超时控制和可观测性，再考虑微优化。
- 没有 profiling、压测或明确瓶颈时，不主动引入复杂性能技巧。

> AI Code 遵循规约，编写高效、稳定、可验证的 Python 代码。

## 1. 内存管理

```python
def read_large_file(path: str):
    with open(path, "r", encoding="utf-8") as file:
        for line in file:
            yield line.rstrip("\n")


total = sum(x * x for x in range(1_000_000))
```

- 大数据集优先生成器、迭代器、流式处理。
- 没有必要时，不把整个文件或整个结果集一次性读入内存。
- `__slots__`、特殊容器或第三方优化结构只在证明确有收益时使用。

## 2. 字符串与数据结构

```python
parts = ["hello", "world"]
result = "-".join(parts)


valid_users = {"alice", "bob", "charlie"}
if username in valid_users:
    process(username)


from collections import deque

queue = deque(maxlen=1000)
queue.append(task)
```

- 循环中频繁拼接字符串优先 `''.join(...)`。
- 查找优先 `set` / `dict`，队列优先 `deque`。
- 先选择合适数据结构，再讨论优化技巧。

## 3. I/O 与资源释放

```python
with open("large.txt", "r", buffering=8192, encoding="utf-8") as file:
    for line in file:
        process(line)


with Session() as session:
    rows = session.execute(statement)
```

- 文件、连接、会话、锁优先用上下文管理器。
- 批量写入、连接池和缓冲 I/O 优先于手写微优化。
- 外部资源使用后必须清理，不把资源释放寄托在 GC 上。

## 4. 并发选型

```python
import asyncio


async def fetch_all(urls: list[str]) -> list[str]:
    ...
```

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor


def run_io_tasks(items: list[str]) -> list[str]:
    with ThreadPoolExecutor(max_workers=8) as executor:
        return list(executor.map(read_one, items))


def run_cpu_tasks(items: list[int]) -> list[int]:
    with ProcessPoolExecutor() as executor:
        return list(executor.map(expensive_compute, items))
```

- I/O 密集型优先 `asyncio` 或线程池。
- CPU 密集型优先进程池。
- 没有明确收益时，不要混合同步、异步、线程、进程多种模型。
- 并发代码先保证取消、超时、资源回收和共享状态安全。

## 5. 超时、重试与降级

```python
import requests


response = requests.get("https://api.example.com", timeout=(3, 10))
response.raise_for_status()
```

```python
import asyncio


async def fetch_with_timeout(url: str) -> str:
    async with asyncio.timeout(5):
        return await fetch(url)
```

- 所有外部调用默认设置超时。
- 是否重试取决于幂等性、失败模式和成本，不机械重试所有错误。
- 降级逻辑应明确，不要用吞错伪装“稳定”。

## 6. 错误处理与可靠性

```python
try:
    return repo.load(user_id)
except RepositoryTimeoutError as exc:
    raise ServiceUnavailableError("下游服务超时") from exc
```

```python
def get_user_data(user_id: str) -> User | None:
    cached = cache.get(user_id)
    if cached is not None:
        return cached
    return repo.find(user_id)
```

- 可靠性首先来自清晰错误边界、超时控制和合理降级。
- 不要在证据不足时引入缓存、熔断器、批处理框架等复杂机制。
- 先修正真实故障路径，再做性能优化。

## 7. 可观测性

```python
import logging
import time


logger = logging.getLogger(__name__)


def process_order(order_id: str) -> None:
    start = time.perf_counter()
    try:
        handle_order(order_id)
    finally:
        elapsed = time.perf_counter() - start
        logger.info("process_order done", extra={"order_id": order_id, "elapsed": elapsed})
```

- 关键路径至少要有日志、耗时或指标中的一种。
- 性能优化前尽量先有基本观测手段。
- 不在生产代码里长期保留 `print` 调试。

## 8. 性能分析

```python
import cProfile
import pstats


profiler = cProfile.Profile()
profiler.enable()
run_main_flow()
profiler.disable()

stats = pstats.Stats(profiler)
stats.sort_stats("cumulative").print_stats(20)
```

- 优化前先定位瓶颈。
- `timeit` 适合小范围基准，`cProfile` 适合整体热点定位。
- 没有证据就不要为了“可能更快”改复杂设计。

## 9. 测试重点

```python
import pytest


def test_create_user_duplicate_email_raises_error() -> None:
    with pytest.raises(DuplicateError):
        service.create_user("existing@example.com")
```

```python
@pytest.mark.parametrize("value,expected", [(0, 1), (5, 120)])
def test_factorial(value: int, expected: int) -> None:
    assert factorial(value) == expected
```

- 覆盖关键业务路径、错误分支和回归风险点。
- 并发或异步代码优先验证取消、超时、资源释放和共享状态安全。
- 不机械追求固定覆盖率数字；覆盖率是信号，不是目标本身。

## 10. 性能与可靠性清单

- [ ] 大数据集使用流式或惰性处理
- [ ] 已选择合适的数据结构
- [ ] 文件、连接、会话等资源能正确释放
- [ ] 外部调用设置了超时
- [ ] 重试策略符合幂等性和失败模式
- [ ] 并发模型和任务类型匹配
- [ ] 关键路径具备基本可观测性
- [ ] 优化前已有 profiling、压测或明确瓶颈证据
- [ ] 关键路径、错误分支和回归风险点已有测试

## 11. 禁止事项

- ❌ 在没有证据时做复杂优化
- ❌ 一次性把大文件或大结果集全部读入内存
- ❌ 外部调用不设置超时
- ❌ 混用多种并发模型而无清晰边界
- ❌ 依赖吞错来“提高稳定性”
- ❌ 用固定覆盖率目标替代真实测试设计
