# Building a RAG Pipeline with Semantic Kernel: A Step-by-Step Guide

Retrieval-Augmented Generation (RAG) is a powerful pattern that combines the strengths of semantic search with large language models (LLMs). In this blog post, we'll walk through how to build a RAG pipeline using **Microsoft's Semantic Kernel (SK)** — a lightweight SDK that makes integrating LLMs, embeddings, and memory seamless.

## Why Semantic Kernel?
Semantic Kernel provides modular components for skills, memory, planners, and connectors that make RAG implementations more intuitive and reusable.

---

## RAG Pipeline Architecture
1. **User Query Input**
2. **Embedding & Vector Search (Memory Retrieval)**
3. **LLM Prompt Composition with Retrieved Context**
4. **LLM Answer Generation**

---

## Prerequisites
- Python 3.8+
- `semantic-kernel`
- `faiss-cpu` or any vector DB
- `openai` or Azure OpenAI key

```bash
pip install semantic-kernel faiss-cpu openai
```

---

## Step 1: Initialize Semantic Kernel
```python
import semantic_kernel as sk
from semantic_kernel.connectors.ai.open_ai import OpenAITextCompletion, OpenAITextEmbedding

kernel = sk.Kernel()

# Add text completion service
kernel.add_text_completion_service(
    service_id="gpt3",
    service=OpenAITextCompletion(
        model_id="gpt-3.5-turbo",
        api_key="YOUR_OPENAI_KEY"
    )
)

# Add embedding generation service
kernel.add_text_embedding_generation_service(
    service_id="embedding",
    service=OpenAITextEmbedding(
        model_id="text-embedding-ada-002",
        api_key="YOUR_OPENAI_KEY"
    )
)
```

---

## Step 2: Load Documents into Memory (Vector DB)
```python
from semantic_kernel.memory import MemoryStoreBase
from semantic_kernel.memory.memory_record import MemoryRecord
from semantic_kernel.memory.vector_memory_store import FaissMemoryStore

memory_store = FaissMemoryStore()
kernel.register_memory_store(memory_store)

# Sample knowledge base
documents = [
    ("doc1", "Semantic Kernel enables lightweight orchestration of LLMs and skills."),
    ("doc2", "RAG pipelines combine retrieval with generative models for improved responses.")
]

for doc_id, content in documents:
    memory_store.save_information(
        collection="knowledge-base",
        record=MemoryRecord.local_record(id=doc_id, text=content, description="")
    )
```

---

## Step 3: Create a Semantic Function with Context Injection
```python
prompt_template = """
You are a helpful assistant. Answer the following question using the context below.

Context:
{{$retrieved_context}}

Question: {{$input}}
Answer:
"""

rag_function = kernel.create_semantic_function(
    prompt_template,
    description="RAG Answer Generator",
    max_tokens=300
)
```

---

## Step 4: Build the RAG Execution Logic
```python
def rag_pipeline(query: str, top_k: int = 2):
    # Retrieve relevant documents
    results = memory_store.search("knowledge-base", query, top_k=top_k)
    context_snippets = "\n".join([r.text for r in results])

    # Call semantic function with context
    variables = sk.ContextVariables()
    variables["retrieved_context"] = context_snippets
    variables["input"] = query

    return rag_function.invoke(variables)

# Try it out
response = rag_pipeline("What is a RAG pipeline?")
print(response.result)
```

---

## Final Thoughts
With Semantic Kernel, building a RAG pipeline becomes modular and extensible. You can enhance this further by:
- Using chunked documents for fine-grained retrieval
- Integrating planning or chaining tools
- Combining multiple memory collections (e.g., FAQs, product docs)

