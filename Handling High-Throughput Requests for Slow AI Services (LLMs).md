# Handling High-Throughput Requests for Slow AI Services (LLMs)

Large Language Models (LLMs) are powerful—but slow and expensive. A single request can take seconds, sometimes tens of seconds, which creates a mismatch when your system needs to handle **high concurrency**.

If you naïvely expose an LLM behind a synchronous API, your system will quickly hit bottlenecks:

* Thread exhaustion
* Timeouts
* Poor user experience
* Spiraling infrastructure cost

This post walks through **production-grade patterns** to handle high-throughput traffic for slow AI services:

## Core Strategies

1. **Async + Streaming Responses**
2. **Queue-based Decoupling**
3. **Concurrency Control & Rate Limiting**
4. **Caching & Deduplication**
5. **Batching Requests**
6. **Fallbacks & Graceful Degradation**

---

# 1. Async + Streaming Responses

## Why it matters

Instead of blocking until the full LLM response is ready, you:

* Return partial output immediately
* Free up server threads
* Improve perceived latency dramatically

## Architecture

```
Client → API (async) → LLM → stream tokens → Client
```

## Example: FastAPI Streaming with OpenAI-like API

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
import asyncio

app = FastAPI()

async def fake_llm_stream(prompt: str):
    # simulate token streaming
    words = prompt.split()
    for word in words:
        await asyncio.sleep(0.2)  # simulate latency
        yield word + " "

@app.get("/generate")
async def generate(prompt: str):
    async def stream():
        async for chunk in fake_llm_stream(prompt):
            yield chunk

    return StreamingResponse(stream(), media_type="text/plain")
```

## Client-side (Python)

```python
import requests

with requests.get("http://localhost:8000/generate?prompt=hello world", stream=True) as r:
    for chunk in r.iter_content(chunk_size=None):
        if chunk:
            print(chunk.decode(), end="")
```

## Key Benefits

* Reduces perceived latency from ~5s → ~500ms
* Prevents request timeouts
* Enables real-time UX (chat apps, copilots)

---

# 2. Queue-Based Architecture (Decoupling)

## Why it matters

LLMs are slow. Your API shouldn’t wait for them.

Instead:

* Accept request
* Push to queue
* Process asynchronously
* Return result later (polling or webhook)

## Architecture

```
Client → API → Queue → Worker → LLM → Storage → Client fetch
```

## Example using Redis Queue (RQ)

### Install

```bash
pip install rq redis
```

### Producer (API)

```python
from fastapi import FastAPI
from redis import Redis
from rq import Queue
import uuid

app = FastAPI()
redis_conn = Redis()
q = Queue(connection=redis_conn)

def llm_task(prompt):
    import time
    time.sleep(5)  # simulate slow LLM
    return f"Processed: {prompt}"

@app.post("/submit")
def submit(prompt: str):
    job = q.enqueue(llm_task, prompt)
    return {"job_id": job.id}
```

### Worker

```bash
rq worker
```

### Check Result

```python
from fastapi import FastAPI
from redis import Redis
from rq.job import Job

app = FastAPI()
redis_conn = Redis()

@app.get("/result/{job_id}")
def result(job_id: str):
    job = Job.fetch(job_id, connection=redis_conn)
    return {
        "status": job.get_status(),
        "result": job.result
    }
```

## Enhancements

* Replace Redis Queue with:

  * Kafka (high throughput)
  * RabbitMQ (reliable delivery)
  * AWS SQS (managed)

---

# 3. Concurrency Control & Rate Limiting

## Problem

Without limits, LLM APIs will:

* Hit provider rate limits
* Crash your workers
* Blow up costs

## Solution: Semaphore-based control

```python
import asyncio

semaphore = asyncio.Semaphore(5)  # max 5 concurrent LLM calls

async def call_llm(prompt):
    async with semaphore:
        await asyncio.sleep(2)
        return f"Result: {prompt}"
```

## Rate limiting (FastAPI example)

```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@app.get("/generate")
@limiter.limit("10/minute")
async def generate(prompt: str):
    return {"result": await call_llm(prompt)}
```

---

# 4. Caching & Deduplication

## Why it matters

LLM calls are expensive and often repetitive.

## Strategy

* Cache identical prompts
* Use embedding similarity for near-duplicates

## Simple Redis Cache

```python
import hashlib
from redis import Redis

redis = Redis()

def get_cache_key(prompt):
    return hashlib.sha256(prompt.encode()).hexdigest()

def cached_llm(prompt):
    key = get_cache_key(prompt)
    
    if redis.exists(key):
        return redis.get(key).decode()
    
    result = slow_llm_call(prompt)
    redis.set(key, result, ex=3600)
    
    return result
```

---

# 5. Request Batching

## Why it matters

Many LLM providers support batching → lower cost + higher throughput.

## Example

```python
async def batch_llm(prompts):
    await asyncio.sleep(2)
    return [f"Result: {p}" for p in prompts]

# batching queue
batch = []

async def process_batch():
    global batch
    while True:
        if batch:
            current = batch[:]
            batch = []
            results = await batch_llm(current)
            print(results)
        await asyncio.sleep(1)
```

---

# 6. Fallbacks & Graceful Degradation

## Techniques

* Timeout fallback
* Smaller model fallback
* Cached/stale response fallback

## Example

```python
async def robust_llm(prompt):
    try:
        return await asyncio.wait_for(call_llm(prompt), timeout=3)
    except asyncio.TimeoutError:
        return "Fallback response (timeout)"
```

---

# Putting It All Together (Reference Architecture)

```
                ┌──────────────┐
                │   Client     │
                └──────┬───────┘
                       │
                ┌──────▼───────┐
                │  API Layer   │
                │ Async + Rate │
                └──────┬───────┘
                       │
        ┌──────────────┼──────────────┐
        │                              │
 ┌──────▼──────┐               ┌──────▼──────┐
 │  Cache      │               │  Queue       │
 └─────────────┘               └──────┬──────┘
                                      │
                               ┌──────▼──────┐
                               │  Workers     │
                               │ (LLM calls)  │
                               └──────┬──────┘
                                      │
                               ┌──────▼──────┐
                               │  Storage     │
                               └─────────────┘
```

---

# Key Takeaways

* **Streaming > Blocking** for UX and scalability
* **Queues decouple** slow LLMs from fast APIs
* **Concurrency control prevents collapse**
* **Caching saves cost dramatically**
* **Batching increases throughput**
* **Fallbacks make systems resilient**
