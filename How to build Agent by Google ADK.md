**Google ADK (Agent Development Kit)** is Google's framework for building AI agents that can reason with an LLM, call tools, maintain state, delegate to other agents, use RAG, and eventually deploy to Google Cloud.

The easiest way to understand ADK is:

> **Agent = Gemini model + instructions + tools + state/memory + orchestration**

Google's current ADK supports Python, Java, Go, and Kotlin, with Python being a very good starting point for GenAI engineering. ([Google GitHub][1])

![Image](https://images.openai.com/static-rsc-4/wpJsNrWEKCUO7WqOubtgEwVgsKy3qMZswgDFMSz_TCKd6yMWMIV5t29r75HTUio_XXZ6n5CeJVY9itKjYh0OxjVrIMYK_YFJO5PDvy6suvvOJD4cxksQ3dVfOhqVpma8ackGf7aKY80yZqEjf6G7MxzxK9UL5qZx66l41F6O4GoUX-qcnM75vZ46uPkjum7j?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/_E7Wfz0n0qW6320YT82PuJ1ICGwIeQhZf7tvBuYBQx7IgkLQejeXudKNNtG25IHt63mvoFOacw3_aW7-14dgJCMqgDfkQGJgT5vjYYBYGduFFcInw2nu8xcRcV0j0Mole3GB_BYpctBNjpqvWNd6eHTJK9yS1N15I1LEBI7DUo6BViCpPhe6t99ijVcVhPaw?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/Nuc5DRViGMO4_FbskpEJrJvEa_RXw9Hl-jvN7eIh0hYT5w2QpxUsx0FzQjhaDU8KAoLN1Hg52u6ePgRzFsbJe1HxvyQGLEnDJ6PeiLeaNBCtfHXCbouPRXaMYVP-MMmVBw0i5FQGeMhrpWm6XUu76oRVuBWXzvXOK9XjO9unwD-dvveVaF93Z3ai6G2Mblmb?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/rN8WizG5N6Wate4gwspI1ghY4Fn7Vh9m0jDnUqmWz1Y3WXX9GrKiaaNev3JPsnP2skdUEQdWV9ToGz2jWA10rJF01u4wFLS9PeLws7bwH8TnQIPx2wOZS8fImJqIw4ZpojBjWHw401DYZgyqGGXLs9gae7LH0NpbVgYnMtHWor_WHz42WPXgj_TrvpJRGaI4?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/QfaW76K0otn1k7N6QolUCYyn4sFmiTKSOjGzDRYGbDFppPB_NpOambJqCXEQ8cQjuu222lF4NnoGHOdZTqF9S_eHEgyvyRx2jfCZEOnrXZ8jOlO5eGRWtBdXKHQLb60Z8Ccq-DDbu2fBxYuGGnMhInFuxz13hn975nv2iXbfavxDcXW3PymQ--TDg3LcRel6?purpose=fullsize)

## 1. Install Google ADK

For Python, create a virtual environment and install ADK:

```bash
python -m venv .venv

# Windows
.venv\Scripts\activate

# Linux/Mac
source .venv/bin/activate

pip install google-adk
```

You'll need either Gemini API credentials or Google Cloud authentication, depending on how you want to run the agent. Google's current tooling also supports `uv` and an Agents CLI for scaffolding and deployment. ([Google GitHub][2])

---

# 2. Create your first ADK agent

A simple ADK project can look like:

```text
my_agent/
│
├── agent.py
└── __init__.py
```

`agent.py`:

```python
from google.adk.agents import Agent

root_agent = Agent(
    name="customer_support_agent",
    model="gemini-2.5-flash",
    description="A customer support assistant",
    instruction="""
    You are a helpful customer support agent.

    Answer customer questions clearly.
    If you don't know the answer, say so.
    Do not invent information.
    """
)
```

That's already an **agent**.

The important part is:

```python
root_agent = Agent(
    name="customer_support_agent",
    model="gemini-2.5-flash",
    instruction="..."
)
```

Think of it as:

```text
                ┌─────────────────┐
User ──────────►│      Agent      │
                │                 │
                │ Gemini          │
                │ Instructions    │
                │ Tools           │
                │ State           │
                └────────┬────────┘
                         │
                         ▼
                      Answer
```

---

# 3. Add tools

This is where ADK becomes much more interesting.

An agent can call Python functions as **tools**.

For example:

```python
from google.adk.agents import Agent


def get_customer_balance(customer_id: str) -> float:
    """Get the customer's current account balance."""

    customers = {
        "C001": 1250.50,
        "C002": 850.75,
        "C003": 4300.20
    }

    return customers.get(customer_id, 0.0)


root_agent = Agent(
    name="banking_agent",
    model="gemini-2.5-flash",
    instruction="""
    You are a banking assistant.

    Help users check their account balance.
    Always ask for the customer ID if it is not provided.
    """,
    tools=[
        get_customer_balance
    ]
)
```

Now the user can ask:

```text
What is my balance?
```

The agent can determine:

```text
I need customer ID
        ↓
Ask user
        ↓
Customer gives C001
        ↓
Call get_customer_balance("C001")
        ↓
Receive 1250.50
        ↓
Generate response
```

You don't have to manually write an `if/else` deciding when the function should be called. The LLM determines when the tool is appropriate.

---

# 4. Tool calling is the key concept

For a GenAI engineer, I would think about ADK tools like this:

```text
                    ADK Agent
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
       Gemini        Tools        Memory
                       │
          ┌────────────┼─────────────┐
          ▼            ▼             ▼
        Python       API/DB         Search
        function
```

Your tools could be:

```python
tools=[
    search_database,
    get_customer,
    calculate_risk,
    send_email,
    create_ticket
]
```

For example, a financial agent could have:

```python
def get_financial_metric(
    company: str,
    region: str,
    metric: str
):
    ...
```

The user could ask:

```text
What was Citi's revenue in Europe?
```

The agent determines that it needs:

```python
get_financial_metric(
    company="Citi",
    region="Europe",
    metric="revenue"
)
```

This is very similar to the chatbot architecture you've been working with for financial reports and SQL.

---

# 5. Build an SQL agent

Here's a more realistic example.

```python
from google.adk.agents import Agent


def execute_sql(sql: str) -> str:
    """
    Execute a read-only SQL query against the financial database.
    """

    # Your real implementation would connect to
    # PostgreSQL, BigQuery, SQL Server, etc.

    print("Executing:", sql)

    return """
    Region: Europe
    Revenue: $12.4B
    """


root_agent = Agent(
    name="financial_sql_agent",
    model="gemini-2.5-flash",
    instruction="""
    You are a financial data analyst.

    Convert user questions into SQL queries.

    Rules:
    - Only generate SELECT queries.
    - Never modify the database.
    - Use the available database schema.
    - Return the answer with the relevant financial context.
    """,
    tools=[execute_sql]
)
```

Architecture:

```text
User
 │
 │ "What was revenue in Europe?"
 ▼
ADK Agent
 │
 ▼
Gemini
 │
 │ Generate SQL
 ▼
execute_sql()
 │
 ▼
Database
 │
 ▼
Result
 │
 ▼
Gemini
 │
 ▼
Final Answer
```

This is one of the most useful ADK patterns for enterprise GenAI.

---

# 6. Add RAG

You can also make an ADK agent search your documents.

For example:

```text
User
 │
 ▼
ADK Agent
 │
 ▼
Gemini
 │
 ├────► Document Search
 │          │
 │          ▼
 │      Vector DB
 │
 ▼
Retrieved Documents
 │
 ▼
Gemini
 │
 ▼
Answer + citations
```

Your search function could look conceptually like:

```python
def search_documents(query: str) -> list:
    """
    Search company documents using semantic search.
    """

    results = vector_db.similarity_search(
        query,
        k=5
    )

    return results
```

Then:

```python
root_agent = Agent(
    name="document_agent",
    model="gemini-2.5-flash",
    instruction="""
    Answer questions using the company documents.

    Always search the documents before answering
    questions about company-specific information.

    If the documents don't contain the answer,
    say that the information was not found.
    """,
    tools=[search_documents]
)
```

You could connect this to:

* Vertex AI Vector Search
* BigQuery
* Cloud Storage
* Elasticsearch
* PostgreSQL/pgvector
* other vector databases

---

# 7. ADK supports multi-agent systems

This is one of the biggest reasons to learn ADK.

Instead of building one giant agent:

```text
                    Main Agent
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      Research       SQL          Writer
       Agent         Agent         Agent
```

For example:

```python
from google.adk.agents import Agent


research_agent = Agent(
    name="research_agent",
    model="gemini-2.5-flash",
    instruction="""
    Research financial documents and
    identify relevant information.
    """,
    tools=[search_documents]
)


sql_agent = Agent(
    name="sql_agent",
    model="gemini-2.5-flash",
    instruction="""
    Query the financial database
    and calculate financial metrics.
    """,
    tools=[execute_sql]
)


root_agent = Agent(
    name="financial_analysis_agent",
    model="gemini-2.5-flash",
    instruction="""
    You are the lead financial analyst.

    Delegate document research to research_agent.
    Delegate database questions to sql_agent.

    Combine their results into one answer.
    """,
    sub_agents=[
        research_agent,
        sql_agent
    ]
)
```

Now you have:

```text
                    User
                      │
                      ▼
             Financial Agent
                      │
             ┌────────┴────────┐
             ▼                 ▼
      Research Agent       SQL Agent
             │                 │
             ▼                 ▼
       PDF/DOCX/RAG        SQL Database
             │                 │
             └────────┬────────┘
                      ▼
                 Final Answer
```

ADK also supports more advanced multi-agent/A2A architectures. Google's current tooling describes A2A as built into the ADK template for distributed specialist agents. ([Google GitHub][3])

---

# 8. Sequential workflows

Suppose you're building your **FRD response agent**.

You could make the workflow:

```text
FRD
 │
 ▼
Document Retrieval
 │
 ▼
Draft Response
 │
 ▼
Compliance Review
 │
 ▼
Gap Detection
 │
 ▼
Redraft
 │
 ▼
Final FRD Response
```

Instead of one agent doing everything, you can create specialists:

```text
                  FRD Manager
                       │
       ┌───────────────┼────────────────┐
       ▼               ▼                ▼
  Retrieval         Writer           Reviewer
    Agent            Agent             Agent
       │               │                │
       ▼               ▼                ▼
 Documents          Draft           Compliance
                                       Check
```

This is a very natural ADK architecture for your Wipfli project.

---

# 9. Parallel agents

Another useful pattern is parallel execution.

Imagine a financial research agent:

```text
                   User Question
                         │
                         ▼
                    Coordinator
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       PDF Agent       SQL Agent     Market Agent
          │              │              │
          ▼              ▼              ▼
       Reports        Database       External API
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                    Synthesizer
                         │
                         ▼
                    Final Answer
```

This can be much faster than:

```text
PDF → SQL → Market → Answer
```

because independent research can happen concurrently.

ADK provides orchestration patterns for this kind of multi-agent architecture. ([Google GitHub][4])

---

# 10. State and sessions

Real agents need to remember the current conversation.

For example:

```text
User:
Show revenue for Europe.

Agent:
Revenue was $12.4B.

User:
What about Asia?

Agent:
Asia revenue was $9.8B.
```

The second question depends on the context of the first.

ADK provides session/state mechanisms so agents can maintain conversational context rather than treating every request as completely independent.

Conceptually:

```text
Session
│
├── user information
├── conversation history
├── current task
├── intermediate results
└── agent state
```

This becomes particularly important when building enterprise agents rather than simple chatbots.

---

# 11. Run the agent locally

Google's current Agents CLI can create a prototype and start an ADK playground:

```bash
agents-cli create my-agent --prototype --yes

cd my-agent

agents-cli install

agents-cli playground
```

The playground runs locally, with the current documentation specifying `localhost:8080`. ([Google GitHub][1])

You can then interact with your agent through the browser.

The CLI also supports:

```bash
agents-cli run "What was Citi revenue in Europe?"
```

and development commands such as:

```bash
agents-cli lint

uv run pytest
```

([Google GitHub][5])

---

# 12. A better project structure

For a production-style ADK application, I'd structure it something like:

```text
financial_agent/
│
├── app/
│   ├── agent.py
│   │
│   ├── agents/
│   │   ├── research_agent.py
│   │   ├── sql_agent.py
│   │   └── writer_agent.py
│   │
│   ├── tools/
│   │   ├── search.py
│   │   ├── sql.py
│   │   └── calculator.py
│   │
│   ├── prompts/
│   │   ├── research.txt
│   │   └── writer.txt
│   │
│   └── config.py
│
├── tests/
│   ├── test_agents.py
│   └── test_tools.py
│
├── requirements.txt
└── README.md
```

This separates:

**Agents**

```text
What should the AI do?
```

from:

**Tools**

```text
What can the AI actually do?
```

That distinction is extremely important when designing agentic systems.

---

# 13. ADK + Gemini + Vertex AI

The Google ecosystem looks roughly like this:

```text
                    Google Cloud
                         │
             ┌───────────┴───────────┐
             │                       │
         Vertex AI                BigQuery
             │                       │
        Gemini Models             Data
             │
             ▼
          Google ADK
             │
     ┌───────┼────────┐
     ▼       ▼        ▼
   Tools    RAG    Multi-Agent
     │       │        │
     └───────┼────────┘
             ▼
        Agent Runtime
             │
             ▼
        Production
```

ADK is therefore **not itself an LLM**.

Rather:

```text
Gemini
   +
ADK
   +
Tools
   +
Data
   +
Memory
   +
Orchestration
   =
Production Agent
```

---

# 14. ADK + MCP

ADK can also work with **MCP (Model Context Protocol)**.

That means your agent can consume standardized external tools rather than implementing every integration yourself.

Conceptually:

```text
                  ADK Agent
                      │
                      ▼
                 Gemini
                      │
          ┌───────────┴──────────┐
          ▼                      ▼
      ADK Tools              MCP Tools
          │                      │
          ▼                      ▼
    Python/API/DB       External MCP Server
```

This becomes particularly useful in enterprise environments where you have many existing tools and services.

---

# 15. ADK + code execution

ADK can also give agents a sandboxed code-execution capability. Google's current Agent Runtime code-execution tool supports persistent execution across requests and data-analysis workflows. ([Google GitHub][6])

For example:

```text
User:
Analyze this CSV and find anomalies.

              │
              ▼
          ADK Agent
              │
              ▼
        Code Execution
              │
       ┌──────┴──────┐
       ▼             ▼
     Python        Dataset
       │             │
       └──────┬──────┘
              ▼
          Analysis
              │
              ▼
           Answer
```

This is useful for:

* data analysis
* financial calculations
* statistical analysis
* Python execution
* chart generation
* code debugging

---

# 16. ADK vs LangGraph

Since you're working with agentic AI, this distinction is worth understanding.

|                          | Google ADK          | LangGraph               |
| ------------------------ | ------------------- | ----------------------- |
| Primary ecosystem        | Google              | LangChain ecosystem     |
| LLM focus                | Gemini + others     | Many providers          |
| Agent abstraction        | Strong              | Strong                  |
| Tools                    | Yes                 | Yes                     |
| Multi-agent              | Yes                 | Yes                     |
| RAG                      | Yes                 | Yes                     |
| MCP                      | Yes                 | Yes                     |
| State                    | Yes                 | Yes                     |
| Workflow orchestration   | Yes                 | Excellent               |
| Vertex AI integration    | Excellent           | Good                    |
| Google Cloud deployment  | Excellent           | Good                    |
| Graph-oriented workflows | Good                | Excellent               |
| Best fit                 | Google Cloud agents | Complex agent workflows |

If your target environment is heavily **GCP + Gemini + Vertex AI**, ADK is particularly attractive.

---

# 17. The architecture I'd recommend for you

Given the kinds of GenAI systems you've been building, I would learn ADK through an **enterprise financial RAG + SQL agent** rather than a toy chatbot.

Build this:

```text
                         User
                           │
                           ▼
                 ┌──────────────────┐
                 │ Financial Manager │
                 │      Agent        │
                 └────────┬─────────┘
                          │
             ┌────────────┼─────────────┐
             │            │             │
             ▼            ▼             ▼
        Document       SQL Agent    Calculator
          Agent
             │            │             │
             ▼            ▼             ▼
       PDF/DOCX       SQL DB        Python
       Semantic       Queries      Calculations
       Search
             │            │             │
             └────────────┼─────────────┘
                          ▼
                    Answer Agent
                          │
                          ▼
                 Answer + Citations
```

Then add:

```text
             + MCP
             + Memory
             + Evaluation
             + Observability
             + Guardrails
             + Human approval
             + A2A
             + Vertex AI
             + Agent Runtime
```

That project would teach you almost all of the important ADK concepts at once.

Google's current development tooling explicitly covers scaffolding, agent code, evaluation, deployment, observability and publishing, so ADK is intended to be used as a full agent-development lifecycle rather than merely an SDK for calling Gemini. ([Google GitHub][5])

**Official documentation:** [Google ADK documentation](https://google.github.io/adk-docs/?utm_source=chatgpt.com)

**My recommended learning order:**

```text
1. Basic Agent
      ↓
2. Gemini + Instructions
      ↓
3. Custom Python Tools
      ↓
4. RAG
      ↓
5. SQL/Data Agents
      ↓
6. Sessions & State
      ↓
7. Multi-Agent
      ↓
8. MCP
      ↓
9. Evaluation
      ↓
10. Observability
      ↓
11. Vertex AI / Agent Runtime
      ↓
12. Production Deployment
```

If your goal is **resume-level Google ADK experience**, the most valuable next step would be to build a complete **ADK enterprise RAG + SQL multi-agent chatbot**, with actual Python code, project structure, Gemini/Vertex AI configuration, tools, RAG, SQL generation, agent delegation, evaluation, and deployment.

[1]: https://google.github.io/agents-cli/guide/getting-started/?utm_source=chatgpt.com "Getting Started - agents-cli"
[2]: https://google.github.io/agents-cli/guide/hands-on-tutorial/?utm_source=chatgpt.com "Tutorial: Manual Workflow - agents-cli"
[3]: https://google.github.io/agents-cli/reference/from-agent-starter-pack/?utm_source=chatgpt.com "From Agent Starter Pack - agents-cli"
[4]: https://google.github.io/agents-cli/guide/use-cases/?utm_source=chatgpt.com "Use Cases - agents-cli"
[5]: https://google.github.io/agents-cli/guide/development/?utm_source=chatgpt.com "Development Guide - agents-cli"
[6]: https://google.github.io/adk-docs/tools/google-cloud/code-exec-agent-engine/?utm_source=chatgpt.com "Agent Runtime Code Execution tool for ADK - Agent Development Kit (ADK)"
