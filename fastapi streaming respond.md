# Streaming LLM Responses with FastAPI (Python)

This guide shows how to **take user input and return model output in a streaming fashion** (token-by-token) using **:contentReference[oaicite:0]{index=0}**. This is the standard approach for ChatGPT-style UX in LLM apps.

---

## Core Idea

Instead of waiting for the entire response:

```python
return {"answer": full_text}
````

You stream chunks as they’re generated:

```python
return StreamingResponse(generator())
```

Where `generator()` **yields tokens or text chunks incrementally**.

---

## Pattern 1 — Basic StreamingResponse (Minimal Example)

### Server

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
import asyncio

app = FastAPI()

async def fake_llm_stream():
    text = "Hello this is streamed token by token"
    for word in text.split():
        yield word + " "
        await asyncio.sleep(0.2)

@app.get("/stream")
async def stream():
    return StreamingResponse(fake_llm_stream(), media_type="text/plain")
```

### Client (Python)

```python
import requests

r = requests.get("http://localhost:8000/stream", stream=True)

for chunk in r.iter_content(chunk_size=None):
    print(chunk.decode(), end="")
```

---

## Pattern 2 — Streaming with OpenAI-style APIs

If you use **OpenAI** streaming:

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
from openai import OpenAI

app = FastAPI()
client = OpenAI()

async def stream_openai(prompt: str):
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}],
        stream=True,
    )

    for chunk in response:
        token = chunk.choices[0].delta.content
        if token:
            yield token

@app.get("/chat")
async def chat(prompt: str):
    return StreamingResponse(stream_openai(prompt), media_type="text/plain")
```

---

## Pattern 3 — Streaming Local Hugging Face Models

Using **Hugging Face** Transformers with PyTorch:

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
from transformers import TextIteratorStreamer
from threading import Thread

app = FastAPI()

def hf_stream(prompt: str):
    streamer = TextIteratorStreamer(tokenizer)

    generation_kwargs = dict(
        inputs=tokenizer(prompt, return_tensors="pt"),
        streamer=streamer,
        max_new_tokens=200,
    )

    thread = Thread(target=model.generate, kwargs=generation_kwargs)
    thread.start()

    for token in streamer:
        yield token

@app.get("/local")
async def local(prompt: str):
    return StreamingResponse(hf_stream(prompt), media_type="text/plain")
```

---

## Pattern 4 — Server-Sent Events (SSE) ⭐ Recommended for Browsers

SSE works especially well with React / Next.js frontends.

### Server

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse

app = FastAPI()

async def event_stream(prompt: str):
    async for token in llm_stream(prompt):
        yield f"data: {token}\n\n"

@app.get("/chat")
async def chat(prompt: str):
    return StreamingResponse(
        event_stream(prompt),
        media_type="text/event-stream",
    )
```

### Frontend (JavaScript)

```javascript
const evtSource = new EventSource("/chat?prompt=hello");

evtSource.onmessage = (e) => {
  console.log(e.data);
};
```

---

## When to Use Each Approach

| Use Case            | Recommended            |
| ------------------- | ---------------------- |
| Python-only client  | StreamingResponse      |
| Browser UI          | SSE ⭐                  |
| Bi-directional chat | WebSockets             |
| OpenAI APIs         | `stream=True`          |
| Local HF models     | `TextIteratorStreamer` |

---

## Production Notes

* Always use **async generators** (`async def` + `yield`)
* Set correct `media_type`
* Handle client disconnects
* Flush tokens early and often
* Prefer SSE over WebSockets unless you need bidirectional control

---

## TL;DR

For modern LLM APIs:

> **FastAPI + StreamingResponse + SSE = Best default**

This pattern scales well for:

* RAG systems
* Agent frameworks
* LangChain / LangGraph services
* Chat-style UIs



```
```
