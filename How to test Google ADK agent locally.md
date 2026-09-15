For **local development**, you normally test the ADK agent **on your laptop before deploying it to Agent Runtime or connecting it to Gemini Enterprise**.

The flow is:

```text
VS Code
   │
   ▼
ADK Agent
   │
   ├── Tools
   ├── RAG
   ├── SQL
   └── Sub-agents
   │
   ▼
Vertex AI
   │
   ▼
Gemini
```

Google's current ADK/Agent Runtime documentation explicitly supports running an `AdkApp` locally with in-memory sessions. ([Google Cloud Documentation][1])

## 1. Set up authentication

From your terminal:

```bash
gcloud auth login

gcloud auth application-default login

gcloud config set project YOUR_PROJECT_ID
```

Set your environment variables:

```bash
export GOOGLE_CLOUD_PROJECT="YOUR_PROJECT_ID"
export GOOGLE_CLOUD_LOCATION="us-central1"
export GOOGLE_GENAI_USE_VERTEXAI="TRUE"
```

On Windows PowerShell:

```powershell
$env:GOOGLE_CLOUD_PROJECT="YOUR_PROJECT_ID"
$env:GOOGLE_CLOUD_LOCATION="us-central1"
$env:GOOGLE_GENAI_USE_VERTEXAI="TRUE"
```

Install ADK:

```bash
pip install google-adk
```

If you're preparing for Agent Runtime as well:

```bash
pip install --upgrade "google-cloud-aiplatform[agent_engines,adk]>=1.112"
```

Google's current quickstart uses this Agent Platform SDK + ADK package combination. ([Google Cloud Documentation][1])

---

# 2. Create your local agent

For example:

```text
my-agent/
│
├── agent.py
├── requirements.txt
└── test_agent.py
```

`agent.py`:

```python
from google.adk.agents import Agent
from vertexai import agent_engines


def get_customer_info(customer_id: str):
    """Get customer information."""
    customers = {
        "1001": {
            "name": "John Smith",
            "segment": "Enterprise",
            "revenue": 2500000
        },
        "1002": {
            "name": "Jane Doe",
            "segment": "SMB",
            "revenue": 850000
        }
    }

    return customers.get(
        customer_id,
        {"error": "Customer not found"}
    )


agent = Agent(
    model="gemini-3.5-flash",
    name="customer_agent",
    instruction="""
    You are a customer information assistant.

    Use the customer tool when the user asks
    about a specific customer.
    """,
    tools=[
        get_customer_info
    ],
)

app = agent_engines.AdkApp(
    agent=agent
)
```

The important pieces are:

```python
agent = Agent(...)
```

and:

```python
app = agent_engines.AdkApp(agent=agent)
```

`AdkApp` is the wrapper that lets you test the ADK agent locally and later deploy the same application to Agent Runtime. ([Google Cloud Documentation][1])

---

# 3. Test it directly from Python

Create `test_agent.py`:

```python
import asyncio

from agent import app


async def main():

    async for event in app.async_stream_query(
        user_id="developer-001",
        message="Tell me about customer 1001"
    ):
        print(event)


if __name__ == "__main__":
    asyncio.run(main())
```

Run:

```bash
python test_agent.py
```

You should see the ADK events and eventually the Gemini response.

This is the **simplest developer test**.

Google specifically documents `async_stream_query()` as the local testing mechanism for an `AdkApp`. ([Google Cloud Documentation][1])

---

# 4. Better: use the ADK web playground

For day-to-day development, you probably don't want to run a Python test file every time.

You can use the ADK web UI/playground.

The current Google Agents CLI provides:

```bash
agents-cli playground
```

which launches a local playground at:

```text
http://localhost:8080
```

and supports hot reload while you modify your agent. ([Google GitHub][2])

Then you can interact with your agent like:

```text
You:
What is customer 1001?

Agent:
Customer 1001 is John Smith,
an Enterprise customer with revenue of $2.5M.
```

Change your Python code → save → test again.

That's much more convenient than repeatedly executing Python scripts.

---

# 5. Test your tools

Suppose your agent has:

```python
def search_customer_database(customer_id: str):
    ...
```

You should test the tool separately.

For example:

```python
def test_customer_database():

    result = get_customer_info("1001")

    assert result["name"] == "John Smith"
    assert result["segment"] == "Enterprise"
```

Then:

```bash
pytest
```

This separates:

```text
Tool testing
     ↓
Agent testing
     ↓
End-to-end testing
```

That's important for a production ADK project.

---

# 6. Test the actual agent behavior

You also want to test whether Gemini correctly chooses the tool.

For example:

```text
Question:
What is the revenue of customer 1001?
```

Expected behavior:

```text
Gemini
  ↓
understands question
  ↓
calls get_customer_info("1001")
  ↓
receives tool result
  ↓
generates answer
```

You can test questions such as:

```text
What is customer 1001's revenue?

What segment is customer 1001?

Tell me about customer 1002.

What customers do you know?

What is the revenue of customer 9999?
```

The last one tests error handling.

---

# 7. Test sessions

ADK also supports sessions locally.

For example:

```python
import asyncio

from agent import app


async def main():

    session = await app.async_create_session(
        user_id="developer-001"
    )

    print("Session:", session)

    async for event in app.async_stream_query(
        user_id="developer-001",
        session_id=session.id,
        message="My preferred region is North America."
    ):
        print(event)

    async for event in app.async_stream_query(
        user_id="developer-001",
        session_id=session.id,
        message="What region did I tell you?"
    ):
        print(event)


asyncio.run(main())
```

This lets you test conversational state.

Local ADK testing uses **in-memory sessions** by default. ([Google Cloud Documentation][3])

---

# 8. Test your RAG agent locally

For your type of project, imagine:

```text
Financial Research Agent
        │
        ├── RAG Agent
        │      └── Vertex AI RAG / Vector Search
        │
        ├── SQL Agent
        │      └── BigQuery
        │
        └── Gemini
```

Your local test could be:

```text
User:
What was Citi's revenue in Asia in 2025?
```

The agent might execute:

```text
User question
     ↓
Gemini
     ↓
Determine required source
     ↓
RAG search ───────► Financial reports
     ↓
SQL query ────────► Database
     ↓
Combine results
     ↓
Gemini
     ↓
Answer + citations
```

You can test all of that locally **before deploying the agent**.

---

# 9. Test multi-agent ADK locally

Suppose you have:

```text
root_agent
│
├── financial_rag_agent
│
├── sql_agent
│
└── calculator_agent
```

You can run the entire hierarchy locally.

For example:

```text
User
 │
 ▼
Root Agent
 │
 ├── "This requires documents"
 │       ↓
 │   RAG Agent
 │
 ├── "This requires database"
 │       ↓
 │   SQL Agent
 │
 └── "This requires calculation"
         ↓
     Calculator
```

You don't need Gemini Enterprise for this development phase.

That's a very important distinction:

### Local development

```text
VS Code
   ↓
ADK
   ↓
Vertex AI
   ↓
Gemini
```

### Production

```text
Gemini Enterprise
        ↓
Agent Runtime
        ↓
ADK Agent
        ↓
Vertex AI / Gemini
        ↓
RAG / SQL / APIs / MCP
```

---

# 10. Then test the deployed agent

Once local testing passes:

```text
LOCAL
  ↓
Unit tests
  ↓
Agent tests
  ↓
RAG tests
  ↓
Integration tests
  ↓
Evaluation
  ↓
DEPLOY
```

Deploy the same `AdkApp`:

```python
from vertexai import agent_engines
from vertexai import types

remote_agent = client.agent_engines.create(
    agent=app,
    config={
        "requirements": [
            "google-cloud-aiplatform[agent_engines,adk]"
        ],
        "staging_bucket": "gs://YOUR_BUCKET",
        "identity_type": types.IdentityType.AGENT_IDENTITY,
    },
)
```

Then test the **cloud version**:

```python
async for event in remote_agent.async_stream_query(
    user_id="developer-001",
    message="Tell me about customer 1001"
):
    print(event)
```

Google documents this as the next step after local testing. ([Google Cloud Documentation][1])

---

## The developer workflow I recommend

For your enterprise agents, think of local testing as this:

```text
                 YOUR LAPTOP
┌─────────────────────────────────────────┐
│                                         │
│  VS Code                                │
│    │                                    │
│    ▼                                    │
│  ADK Agent                              │
│    │                                    │
│    ├── Tools                            │
│    ├── RAG                              │
│    ├── SQL                              │
│    ├── MCP                              │
│    └── Sub-agents                       │
│         │                               │
│         ▼                               │
│      Gemini                             │
│      Vertex AI                          │
│                                         │
│  ┌───────────────────────────────┐      │
│  │ Local ADK Playground          │      │
│  │ localhost:8080                │      │
│  └───────────────────────────────┘      │
│                                         │
│  Unit Tests → Agent Tests → Eval        │
└─────────────────────────────────────────┘
                    │
                    │ deploy
                    ▼
             Agent Runtime
                    │
                    ▼
            Gemini Enterprise
                    │
                    ▼
               Employees
```

So **you do not need Gemini Enterprise to test your ADK agent locally**. Gemini Enterprise becomes important after your agent is deployed/hosted in Agent Runtime and you want enterprise users to discover and invoke it. ([Google Cloud Documentation][1])

For your next step, the most useful thing would be to build a **real local ADK project from scratch**—`VS Code → uv → ADK → Gemini on Vertex AI → RAG → SQL tool → local playground → pytest → deploy to Agent Runtime → Gemini Enterprise`—rather than just the small example above.

[1]: https://docs.cloud.google.com/gemini-enterprise-agent-platform/build/runtime/quickstart-adk?utm_source=chatgpt.com "Quickstart: Develop and deploy agents on Agent Runtime with Agent Development Kit  |  Gemini Enterprise Agent Platform  |  Google Cloud Documentation"
[2]: https://google.github.io/agents-cli/guide/hands-on-tutorial/?utm_source=chatgpt.com "Tutorial: Manual Workflow - agents-cli"
[3]: https://docs.cloud.google.com/gemini-enterprise-agent-platform/build/runtime/create-an-adk-agent?utm_source=chatgpt.com "Develop an Agent Development Kit agent  |  Gemini Enterprise Agent Platform  |  Google Cloud Documentation"
