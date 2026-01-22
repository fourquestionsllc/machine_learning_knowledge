## What is MCP in Agentic AI?

**MCP = Model Context Protocol**

It is an **open protocol** that standardizes **how LLM agents connect to tools, data sources, and services** in a *model-agnostic* way.

> Think of MCP as **“USB-C for AI agents”**
> One protocol → many tools → any model

Instead of writing custom LangChain / tool code per model, MCP gives agents a **consistent interface** to:

* Databases
* APIs
* File systems
* Internal services
* Knowledge bases

---

## Why MCP matters for Agentic AI

Without MCP:

* Tight coupling to frameworks (LangChain, Bedrock Agents, OpenAI tools)
* Tool rewrites when models change
* Hard to govern, audit, and sandbox

With MCP:

* 🔁 Swap models freely (Claude, GPT, Llama)
* 🔐 Centralized security & permissions
* 📦 Reusable tools across agents
* 🧠 Cleaner agent reasoning

---

## Core MCP Concepts

### 1️⃣ MCP Server

A **tool provider** that exposes capabilities.

Examples:

* SQL database server
* Vector search server
* Internal REST API server
* File system server

### 2️⃣ MCP Client

The **agent / LLM runtime** that calls MCP tools.

Examples:

* Claude Desktop
* Custom Python agent
* IDE copilots

### 3️⃣ Tools

Each server exposes **tools** with:

* Name
* Description
* JSON schema (inputs/outputs)

LLMs decide **when** to call them.

---

## MCP Architecture

```
Agent (Claude / GPT)
   ↓ MCP Client
Model Context Protocol
   ↓
MCP Server(s)
   ├── SQL
   ├── Vector DB
   ├── REST API
   └── Files
```

---

## Minimal MCP Example (Python)

### Step 1: Install MCP SDK

```bash
pip install mcp
```

---

### Step 2: Create an MCP Server (Tool Provider)

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("patient-data")

@mcp.tool()
def get_patient_record(patient_id: str) -> dict:
    """
    Fetch patient record by ID
    """
    return {
        "patient_id": patient_id,
        "age": 54,
        "conditions": ["hypertension"],
        "medications": ["lisinopril"]
    }

if __name__ == "__main__":
    mcp.run()
```

This exposes:

```
Tool name: get_patient_record
Input: { patient_id: string }
```

---

### Step 3: Use MCP from an Agent (Client Side)

```python
from mcp.client import ClientSession
from mcp.client.stdio import stdio_client

async def run_agent():
    async with stdio_client() as (read, write):
        session = ClientSession(read, write)

        result = await session.call_tool(
            "get_patient_record",
            {"patient_id": "12345"}
        )

        print(result)

```

---

## MCP + Agentic Reasoning (Real Pattern)

### Agent Thought Loop

```
1. User asks question
2. LLM decides tool is needed
3. Calls MCP tool
4. Receives structured data
5. Generates grounded answer
```

---

## MCP with Claude (Typical Setup)

```json
{
  "mcpServers": {
    "clinical": {
      "command": "python",
      "args": ["patient_server.py"]
    }
  }
}
```

Claude automatically discovers:

* Tool names
* Schemas
* Descriptions

No prompt glue needed.

---

## MCP vs LangChain Tools

| Feature             | MCP    | LangChain |
| ------------------- | ------ | --------- |
| Model agnostic      | ✅      | ❌         |
| Standard protocol   | ✅      | ❌         |
| Tool reuse          | High   | Medium    |
| Security boundaries | Strong | Weak      |
| Vendor lock-in      | Low    | High      |

---

## When You Should Use MCP

✅ You have **multiple agents**
✅ You want **tool reuse across teams**
✅ You need **security / auditability**
✅ You plan to **swap models** (Claude ↔ GPT ↔ Llama)
✅ Enterprise / regulated environments (CDC, J&J, GEICO)

---

## MCP + RAG (Very Common)

* MCP server exposes:

  * `search_documents`
  * `get_chunk`
* Agent retrieves via MCP
* Answer is always grounded → fewer hallucinations

---

## Production Best Practices

* One MCP server per **domain**
* Read-only tools for LLMs
* Add rate limits
* Log all tool calls
* Version tool schemas

---

## Mental Model (Remember This)

> **Agents reason, MCP executes.**

LLMs decide *what* to do.
MCP defines *how* it’s done — safely and consistently.



-------

Below is a **concrete, end-to-end example** of using **MCP (Model Context Protocol)** as a **retrieval tool** inside a **LangGraph-based RAG system**.

I’ll show:

1. **Architecture overview** (LangGraph + MCP)
2. **MCP server** exposing a vector search tool
3. **LangGraph agent** calling MCP for retrieval
4. **Why MCP is useful vs normal LangChain tools**
5. **Production notes** (very relevant to your RAG background)

---

## 1️⃣ Architecture: LangGraph + MCP for RAG

```
User Question
   ↓
LangGraph Router Node
   ↓
Retriever Node (MCP Client → MCP Server)
   ↓
LLM Answer Node
   ↓
Final Answer
```

**Key idea**

* LangGraph controls **workflow + state**
* MCP provides **standardized retrieval / tools** (vector DB, SQL, APIs)
* LLM calls MCP tools *without knowing backend details*

---

## 2️⃣ MCP Server (Vector Search Tool)

This MCP server exposes a **semantic search tool** over a vector DB
(FAISS / Pinecone / Chroma — abstracted away)

### `mcp_server.py`

```python
from mcp.server.fastmcp import FastMCP
from typing import List

# Create MCP server
mcp = FastMCP("rag-retriever")

# Mock vector store
DOCUMENTS = [
    {"id": 1, "text": "LangGraph is a stateful orchestration framework for LLM agents"},
    {"id": 2, "text": "MCP standardizes tool and data access for LLMs"},
    {"id": 3, "text": "RAG combines retrieval with generation for better grounding"},
]

@mcp.tool()
def semantic_search(query: str, k: int = 2) -> List[str]:
    """
    Perform semantic search over documents.
    """
    # 🔹 Replace with real vector DB logic
    matches = [
        doc["text"]
        for doc in DOCUMENTS
        if query.lower() in doc["text"].lower()
    ]
    return matches[:k]

if __name__ == "__main__":
    mcp.run()
```

🔹 In production, this would wrap:

* Pinecone
* Azure AI Search
* Weaviate
* PGVector

---

## 3️⃣ LangGraph RAG Using MCP Client

### Install

```bash
pip install langgraph langchain mcp
```

---

### MCP Client Tool Wrapper

```python
from langchain.tools import tool
from mcp.client import ClientSession, StdioServerParameters
import asyncio

server_params = StdioServerParameters(
    command="python",
    args=["mcp_server.py"],
)

async def call_mcp_search(query: str):
    async with ClientSession(server_params) as session:
        await session.initialize()
        result = await session.call_tool(
            "semantic_search",
            {"query": query, "k": 3}
        )
        return result.content

@tool
def retrieve_docs(query: str) -> str:
    """Retrieve documents using MCP semantic search"""
    return asyncio.run(call_mcp_search(query))
```

---

## 4️⃣ LangGraph RAG Workflow

### Graph State

```python
from typing import TypedDict, List

class RAGState(TypedDict):
    question: str
    documents: List[str]
    answer: str
```

---

### Nodes

#### Retriever Node (via MCP)

```python
def retriever_node(state: RAGState):
    docs = retrieve_docs.run(state["question"])
    return {"documents": docs}
```

---

#### Answer Generation Node

```python
from langchain.chat_models import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o-mini")

def generator_node(state: RAGState):
    context = "\n".join(state["documents"])
    prompt = f"""
Answer the question using only the context below.

Context:
{context}

Question:
{state['question']}
"""
    answer = llm.invoke(prompt).content
    return {"answer": answer}
```

---

## 5️⃣ LangGraph Assembly

```python
from langgraph.graph import StateGraph, END

graph = StateGraph(RAGState)

graph.add_node("retrieve", retriever_node)
graph.add_node("generate", generator_node)

graph.set_entry_point("retrieve")
graph.add_edge("retrieve", "generate")
graph.add_edge("generate", END)

rag_app = graph.compile()
```

---

## 6️⃣ Run the RAG App

```python
result = rag_app.invoke({
    "question": "What is MCP and how does it help RAG?"
})

print(result["answer"])
```

---

## 7️⃣ Why MCP Is Powerful for RAG (Important)

### Compared to standard LangChain tools:

| Feature               | LangChain Tool | MCP |
| --------------------- | -------------- | --- |
| Standardized protocol | ❌              | ✅   |
| Language-agnostic     | ❌              | ✅   |
| Secure sandboxing     | ❌              | ✅   |
| Tool discovery        | ❌              | ✅   |
| Enterprise-ready      | ⚠️             | ✅   |

**For your use cases (enterprise RAG, multi-agent, Azure, CI/CD):**

* MCP decouples **LLM logic from data infra**
* You can version, audit, and secure tools
* Works perfectly with **LangGraph orchestration**

---

## 8️⃣ Production-Grade Pattern (Recommended)

Given your background in:

* multi-agent RAG
* validation
* throughput
* enterprise deployment

👉 **Best practice setup**

```
LangGraph
 ├─ Router Agent
 ├─ Retriever Agent (MCP → Search)
 ├─ SQL Agent (MCP → DB)
 ├─ Tool Agent (MCP → APIs)
 └─ Answer Synthesizer
```

Each MCP server:

* Runs independently
* Has its own auth, logging, scaling
* Can be reused across teams

---

