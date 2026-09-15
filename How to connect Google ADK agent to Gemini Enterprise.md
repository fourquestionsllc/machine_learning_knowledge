If your goal is:

> **Build an ADK agent locally → use Gemini/Vertex AI → deploy it → connect it to Gemini Enterprise so employees can use it**

the connection is **not a direct connection from your local VS Code to Gemini Enterprise**.

The current Google architecture is:

```text
Your laptop
   │
   │ develop/test
   ▼
Google ADK Agent
   │
   │ deploy
   ▼
Agent Runtime
   │
   │ register
   ▼
Gemini Enterprise
   │
   ▼
Enterprise users
```

Google's documentation specifically says that an ADK agent must be **hosted on Agent Runtime** to function within Gemini Enterprise. ([Google Cloud Documentation][1])

## 1. First: build the agent locally

Your project might be:

```text
financial-agent/
│
├── app/
│   └── financial_agent/
│       ├── __init__.py
│       ├── agent.py
│       └── tools/
│           ├── rag.py
│           └── sql.py
│
├── tests/
├── evals/
├── pyproject.toml
└── README.md
```

Your `agent.py`:

```python
from google.adk.agents import Agent

root_agent = Agent(
    name="financial_agent",
    model="gemini-2.5-flash",
    description="Financial research agent",
    instruction="""
    You are a financial research assistant.

    Answer questions using company documents
    and structured financial data.

    Never invent financial information.
    """
)
```

At this point:

```text
VS Code
   │
   ▼
ADK
   │
   ▼
Gemini on Vertex AI
```

You can test this locally first. Google's ADK quickstart explicitly supports local IDE/terminal development and local testing before deployment. ([Google Cloud Documentation][2])

---

# 2. Deploy the ADK agent to Agent Runtime

This is the **bridge to Gemini Enterprise**.

Google's current Agent Runtime supports ADK agents and provides managed execution, sessions, and other production capabilities. ([Google Cloud Documentation][2])

For Python, the current Agent Platform SDK can create an `AdkApp`:

```python
from google.adk.agents import Agent
from vertexai import agent_engines

agent = Agent(
    name="financial_agent",
    model="gemini-2.5-flash",
    instruction="""
    You are a financial research assistant.
    """,
)

app = agent_engines.AdkApp(
    agent=agent
)
```

Then you deploy it to Agent Runtime.

The current Google quickstart uses the Agent Platform SDK to create the managed `reasoningEngine` resource. ([Google Cloud Documentation][2])

Conceptually:

```text
                    GCP
                     │
              Agent Runtime
                     │
              ┌──────┴──────┐
              │             │
          ADK Agent       Gemini
              │
              ├── RAG
              ├── Tools
              ├── BigQuery
              └── MCP
```

---

# 3. Now connect it to Gemini Enterprise

Once the ADK agent is deployed on Agent Runtime, an **administrator registers that agent in the Gemini Enterprise app**.

Google's current console flow is:

```text
Google Cloud Console
        │
        ▼
Gemini Enterprise
        │
        ▼
Select your app
        │
        ▼
Agents
        │
        ▼
Add agent
        │
        ▼
Custom agent via Agent Runtime
        │
        ▼
Select/register ADK agent
```

This makes your custom agent available to users inside the Gemini Enterprise web application. ([Google Cloud Documentation][1])

So the important relationship is:

```text
                  Gemini Enterprise
                         │
                         │ invokes
                         ▼
                   Agent Runtime
                         │
                         │ runs
                         ▼
                    Your ADK Agent
                         │
                 ┌───────┴────────┐
                 ▼                ▼
              Gemini             RAG
            Vertex AI          Vertex AI
```

---

# 4. What happens when the employee asks a question?

Suppose your employee opens Gemini Enterprise and asks:

> "What was Citi's revenue in Europe in 2025?"

The flow can be:

```text
Employee
   │
   ▼
Gemini Enterprise
   │
   │ selects your Financial Agent
   ▼
Agent Runtime
   │
   ▼
ADK Financial Agent
   │
   ├───────────────┐
   ▼               ▼
Vertex AI RAG    BigQuery
   │               │
   └───────┬───────┘
           ▼
        Gemini
           │
           ▼
    Financial Agent
           │
           ▼
    Gemini Enterprise
           │
           ▼
       Employee
```

That's the architecture you want to understand.

---

# 5. What does Gemini Enterprise actually do?

This is where it's different from ADK.

### ADK

You are the developer:

```text
I define:
   Agent
   Instructions
   Tools
   RAG
   Workflow
   Multi-agent
   MCP
```

### Agent Runtime

Google runs your agent:

```text
Deploy
Run
Sessions
Identity
Scaling
```

### Gemini Enterprise

The enterprise user interacts with and discovers your agent:

```text
Employees
    │
    ▼
Gemini Enterprise
    │
    ├── Google agents
    ├── Your custom agents
    ├── A2A agents
    └── Other enterprise agents
```

Google describes Gemini Enterprise as being able to govern custom organizational agents, including ADK agents hosted on Agent Runtime. ([Google Cloud Documentation][3])

---

# 6. What if your agent is in another GCP project?

This is also supported.

For example:

```text
GCP Project A
────────────────────
Gemini Enterprise
       │
       │
       ▼
GCP Project B
────────────────────
Agent Runtime
       │
       ▼
Financial ADK Agent
```

Google supports cross-project ADK agents. The Gemini Enterprise service agent in the Gemini Enterprise project needs appropriate permissions in the project hosting the ADK agent. ([Google Cloud Documentation][4])

This is useful in a large organization because you might have:

```text
Central AI Platform Project
        │
        ├── Enterprise Gemini
        │
        └── Agent Registry
                 │
     ┌───────────┼────────────┐
     ▼           ▼            ▼
Finance       HR           Legal
Project       Project      Project
   │             │            │
 ADK            ADK          ADK
 Agent          Agent        Agent
```

---

# 7. And this is where Agent Registry becomes important

If your company has 100 agents, you don't want Gemini Enterprise to have a bunch of manually maintained URLs.

Google's Agent Platform includes an **Agent Registry** for centralized discovery of agents and MCP servers. It can contain metadata such as:

```text
Agent name
Version
Framework
Capabilities
Endpoint
MCP tools
```

([Google Cloud Documentation][5])

So your enterprise architecture becomes:

```text
                       Gemini Enterprise
                              │
                              ▼
                       Agent Registry
                              │
            ┌─────────────────┼─────────────────┐
            ▼                 ▼                 ▼
       Financial           HR Agent         Legal Agent
          Agent
            │                 │                 │
            ▼                 ▼                 ▼
       Agent Runtime     Agent Runtime     Agent Runtime
```

That's much more scalable.

---

# 8. There are actually two different connections

This distinction is important.

### Connection A — Your ADK agent → Gemini

Your agent uses Gemini as its model:

```text
ADK
 │
 ▼
Vertex AI
 │
 ▼
Gemini
```

For example:

```python
Agent(
    model="gemini-2.5-flash"
)
```

### Connection B — Your ADK agent → Gemini Enterprise

This is:

```text
ADK Agent
    │
    ▼
Agent Runtime
    │
    ▼
Gemini Enterprise
```

You **deploy first**, then **register the deployed agent**.

You don't normally do:

```text
Local Python
     │
     └──────► Gemini Enterprise
```

---

# 9. What about MCP and A2A?

These give you another layer.

For example:

```text
                     Gemini Enterprise
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
         ADK Agent      A2A Agent       MCP Tools
             │              │              │
             ▼              ▼              ▼
        Agent Runtime    External       Enterprise
                          Agent          Systems
```

A2A allows agents to communicate with other agents, while MCP standardizes access to tools/context.

Google's current Agent Registry can expose A2A endpoints, and ADK can resolve registered remote A2A agents as sub-agents. ([Google Cloud Documentation][6])

---

# 10. The complete developer lifecycle

For the workflow you've been asking about, memorize this:

```text
             DEVELOPER
                 │
                 ▼
          VS Code / IDE
                 │
                 ▼
             Google ADK
                 │
       ┌─────────┼──────────┐
       ▼         ▼          ▼
     Gemini     RAG        Tools
       │         │          │
       └─────────┼──────────┘
                 ▼
           Local Testing
                 │
                 ▼
          Agent Evaluation
                 │
                 ▼
               Git
                 │
                 ▼
              CI/CD
                 │
                 ▼
          Agent Runtime
                 │
                 ▼
          Agent Registry
                 │
                 ▼
        Gemini Enterprise
                 │
                 ▼
             Employees
```

And production operations:

```text
Gemini Enterprise
        │
        ▼
   Agent Runtime
        │
 ┌──────┼────────┐
 ▼      ▼        ▼
Logs  Traces   Metrics
 │      │        │
 └──────┼────────┘
        ▼
   Evaluation
        │
        ▼
   New Version
        │
        └──────► Deploy
```

So if you're learning this for **GCP/GenAI engineering**, the practical sequence I recommend is:

**Local VS Code → ADK → Gemini on Vertex AI → RAG → tools → local testing → evaluation → Agent Runtime → Agent Registry → Gemini Enterprise.**

That is the clean end-to-end story. ([Google Cloud Documentation][2])

[1]: https://docs.cloud.google.com/gemini/enterprise/docs/register-and-manage-an-adk-agent?authuser=2&utm_source=chatgpt.com "Register and manage ADK agents hosted on Agent Runtime  |  Gemini Enterprise  |  Google Cloud Documentation"
[2]: https://docs.cloud.google.com/gemini-enterprise-agent-platform/build/runtime/quickstart-adk?utm_source=chatgpt.com "Quickstart: Develop and deploy agents on Agent Runtime with Agent Development Kit  |  Gemini Enterprise Agent Platform  |  Google Cloud Documentation"
[3]: https://docs.cloud.google.com/gemini/enterprise/docs/agents-overview?authuser=50&utm_source=chatgpt.com "Agents overview  |  Gemini Enterprise  |  Google Cloud Documentation"
[4]: https://docs.cloud.google.com/gemini/enterprise/docs/configure-cross-project-adk-agents?authuser=610&utm_source=chatgpt.com "Configure cross-project ADK agent access  |  Gemini Enterprise  |  Google Cloud Documentation"
[5]: https://docs.cloud.google.com/gemini-enterprise-agent-platform/agents?utm_source=chatgpt.com "Agents overview  |  Gemini Enterprise Agent Platform  |  Google Cloud Documentation"
[6]: https://docs.cloud.google.com/gemini/enterprise/docs/invoke-agent-a2a?authuser=2&utm_source=chatgpt.com "Call an agent using its registry A2A endpoint  |  Gemini Enterprise  |  Google Cloud Documentation"
