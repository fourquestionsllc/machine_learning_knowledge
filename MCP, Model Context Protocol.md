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
