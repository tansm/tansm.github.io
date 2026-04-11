# Python 异步编程完全指南

## 第一章：异步编程基础

Python 的异步编程模型基于事件循环（Event Loop），通过 `asyncio` 标准库实现。与多线程相比，异步编程在 I/O 密集型任务中有显著优势，因为它避免了线程切换的开销，并且不需要处理锁和竞态条件等并发问题。

### 1.1 同步 vs 异步

同步代码按顺序执行，每个操作必须等待前一个操作完成：

```python
import time

def fetch_data(url):
    time.sleep(2)  # 模拟网络请求
    return f"Data from {url}"

def main():
    result1 = fetch_data("https://api1.example.com")
    result2 = fetch_data("https://api2.example.com")
    result3 = fetch_data("https://api3.example.com")
    # 总耗时约 6 秒
    return [result1, result2, result3]
```

异步代码可以在等待 I/O 时切换到其他任务：

```python
import asyncio

async def fetch_data(url):
    await asyncio.sleep(2)  # 模拟网络请求
    return f"Data from {url}"

async def main():
    results = await asyncio.gather(
        fetch_data("https://api1.example.com"),
        fetch_data("https://api2.example.com"),
        fetch_data("https://api3.example.com"),
    )
    # 总耗时约 2 秒
    return results
```

### 1.2 核心概念

**协程（Coroutine）**

使用 `async def` 定义的函数就是协程函数，调用它返回一个协程对象。协程对象需要被 `await` 或放入事件循环才能执行。

**事件循环（Event Loop）**

事件循环是异步程序的核心，负责调度和执行协程。它不断地检查是否有可以执行的任务，并在任务等待 I/O 时切换到其他任务。

**Task（任务）**

Task 是对协程的封装，允许协程并发执行。通过 `asyncio.create_task()` 创建。

**Future**

Future 是一个低级别的对象，表示一个异步操作的最终结果。Task 是 Future 的子类。

## 第二章：asyncio 详解

### 2.1 基本用法

```python
import asyncio

async def say_hello(name: str, delay: float):
    await asyncio.sleep(delay)
    print(f"Hello, {name}!")
    return f"Greeted {name}"

async def main():
    # 顺序执行
    await say_hello("Alice", 1)
    await say_hello("Bob", 1)
    # 耗时 2 秒

asyncio.run(main())
```

### 2.2 并发执行

```python
async def main():
    # 并发执行
    task1 = asyncio.create_task(say_hello("Alice", 1))
    task2 = asyncio.create_task(say_hello("Bob", 1))

    result1 = await task1
    result2 = await task2
    # 耗时约 1 秒

# 或者使用 gather
async def main():
    results = await asyncio.gather(
        say_hello("Alice", 1),
        say_hello("Bob", 1),
        say_hello("Charlie", 1),
    )
    print(results)
```

### 2.3 超时控制

```python
async def slow_operation():
    await asyncio.sleep(10)
    return "Done"

async def main():
    try:
        result = await asyncio.wait_for(slow_operation(), timeout=3.0)
    except asyncio.TimeoutError:
        print("Operation timed out!")
```

### 2.4 异步上下文管理器

```python
import aiofiles

async def read_file(path: str):
    async with aiofiles.open(path, 'r') as f:
        content = await f.read()
    return content
```

### 2.5 异步迭代器

```python
class AsyncCounter:
    def __init__(self, start: int, stop: int):
        self.current = start
        self.stop = stop

    def __aiter__(self):
        return self

    async def __anext__(self):
        if self.current >= self.stop:
            raise StopAsyncIteration
        await asyncio.sleep(0.1)
        value = self.current
        self.current += 1
        return value

async def main():
    async for num in AsyncCounter(0, 5):
        print(num)
```

## 第三章：异步 HTTP 请求

### 3.1 使用 aiohttp

```python
import aiohttp
import asyncio
from typing import List

async def fetch_url(session: aiohttp.ClientSession, url: str) -> dict:
    async with session.get(url) as response:
        response.raise_for_status()
        return await response.json()

async def fetch_multiple_urls(urls: List[str]) -> List[dict]:
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url(session, url) for url in urls]
        results = await asyncio.gather(*tasks, return_exceptions=True)
    return results
```

### 3.2 重试机制

```python
from functools import wraps

def async_retry(max_attempts: int = 3, delay: float = 1.0):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            last_exception = None
            for attempt in range(max_attempts):
                try:
                    return await func(*args, **kwargs)
                except Exception as e:
                    last_exception = e
                    if attempt < max_attempts - 1:
                        wait_time = delay * (2 ** attempt)  # 指数退避
                        await asyncio.sleep(wait_time)
            raise last_exception
        return wrapper
    return decorator

@async_retry(max_attempts=3, delay=1.0)
async def fetch_with_retry(url: str) -> dict:
    async with aiohttp.ClientSession() as session:
        async with session.get(url, timeout=aiohttp.ClientTimeout(total=5)) as response:
            response.raise_for_status()
            return await response.json()
```

## 第四章：异步数据库操作

### 4.1 使用 asyncpg（PostgreSQL）

```python
import asyncpg

async def create_pool():
    return await asyncpg.create_pool(
        host="localhost", port=5432,
        database="mydb", user="postgres", password="password",
        min_size=5, max_size=20
    )

async def get_users(pool: asyncpg.Pool, limit: int = 10):
    async with pool.acquire() as conn:
        rows = await conn.fetch("SELECT id, name, email FROM users LIMIT $1", limit)
        return [dict(row) for row in rows]
```

### 4.2 使用 SQLAlchemy 异步

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker, DeclarativeBase
from sqlalchemy import Column, Integer, String, select

engine = create_async_engine("postgresql+asyncpg://user:password@localhost/mydb")
AsyncSessionLocal = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)

async def get_user_by_id(user_id: int):
    async with AsyncSessionLocal() as session:
        result = await session.execute(select(User).where(User.id == user_id))
        return result.scalar_one_or_none()
```

## 第五章：异步队列与生产者消费者模式

```python
import asyncio
import random

async def producer(queue: asyncio.Queue, name: str, count: int):
    for i in range(count):
        item = f"{name}-item-{i}"
        await queue.put(item)
        await asyncio.sleep(random.uniform(0.1, 0.5))

async def consumer(queue: asyncio.Queue, name: str):
    while True:
        try:
            item = await asyncio.wait_for(queue.get(), timeout=2.0)
            await asyncio.sleep(random.uniform(0.2, 0.8))
            queue.task_done()
        except asyncio.TimeoutError:
            break

async def main():
    queue = asyncio.Queue(maxsize=10)
    producers = [asyncio.create_task(producer(queue, f"P{i}", 5)) for i in range(3)]
    consumers = [asyncio.create_task(consumer(queue, f"C{i}")) for i in range(2)]
    await asyncio.gather(*producers)
    await queue.join()
    for c in consumers:
        c.cancel()
    await asyncio.gather(*consumers, return_exceptions=True)

asyncio.run(main())
```

## 第六章：异步编程最佳实践

### 6.1 避免阻塞事件循环

永远不要在协程中调用阻塞的同步代码：

```python
# 错误做法
async def bad_example():
    import time
    time.sleep(5)  # 阻塞事件循环！

# 正确做法
async def good_example():
    await asyncio.sleep(5)  # 非阻塞
```

如果必须调用阻塞代码，使用 `run_in_executor`：

```python
from concurrent.futures import ThreadPoolExecutor

async def run_blocking_code():
    loop = asyncio.get_event_loop()
    with ThreadPoolExecutor() as pool:
        result = await loop.run_in_executor(pool, blocking_function, arg1, arg2)
    return result
```

### 6.2 信号量控制并发数

```python
async def fetch_with_semaphore(session, url, semaphore):
    async with semaphore:
        async with session.get(url) as response:
            return await response.json()

async def fetch_all_limited(urls: list, max_concurrent: int = 10):
    semaphore = asyncio.Semaphore(max_concurrent)
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_with_semaphore(session, url, semaphore) for url in urls]
        return await asyncio.gather(*tasks)
```

### 6.3 取消任务

```python
async def cancellable_task():
    try:
        while True:
            await asyncio.sleep(1)
    except asyncio.CancelledError:
        # 执行清理操作
        raise  # 重新抛出

async def main():
    task = asyncio.create_task(cancellable_task())
    await asyncio.sleep(3)
    task.cancel()
    try:
        await task
    except asyncio.CancelledError:
        print("Task cancelled successfully")
```

## 第七章：总结

Python 异步编程是处理 I/O 密集型任务的强大工具。核心要点：

1. 使用 `async/await` 语法定义和调用协程
2. 用 `asyncio.gather()` 并发执行多个协程
3. 用 `asyncio.create_task()` 创建后台任务
4. 永远不要在协程中调用阻塞代码
5. 用信号量控制并发数量，避免资源耗尽
6. 正确处理 `CancelledError` 和超时
7. 选择支持异步的库（aiohttp、asyncpg、aiofiles 等）

异步编程不是银弹，对于 CPU 密集型任务，应该使用多进程（`multiprocessing`）而不是异步。
