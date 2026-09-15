If you're setting up a **local Google ADK + Gemini + Vertex AI project**, I recommend **VS Code + Python + `uv`**. This gives you a normal software-development workflow while Gemini runs through Google Cloud.

## 1. What you need

Install:

* **VS Code**
* **Python 3.10+**
* **Git**
* **Google Cloud CLI (`gcloud`)**
* **uv** — Python project/package manager
* Google ADK

Your local architecture will be:

```text
┌──────────────────────────────┐
│         VS Code              │
│                              │
│  Python + Google ADK         │
│  Agent + Tools + RAG         │
└──────────────┬───────────────┘
               │
               │ Google Cloud auth
               ▼
┌──────────────────────────────┐
│          Vertex AI           │
│                              │
│  Gemini                      │
│  Embeddings                  │
│  RAG / Vector Search         │
│  Agent services              │
└──────────────────────────────┘
```

---

# 2. Create the Google Cloud project

Open Google Cloud Console and create a project, for example:

```text
Project name:
genai-adk-demo

Project ID:
genai-adk-demo-123
```

Then enable Vertex AI.

Using the CLI:

```bash
gcloud auth login
```

Then:

```bash
gcloud config set project YOUR_PROJECT_ID
```

Enable the APIs you need:

```bash
gcloud services enable aiplatform.googleapis.com
```

For later RAG development you may also enable services such as Cloud Storage and BigQuery.

---

# 3. Authenticate your local IDE

For local development, I recommend **Application Default Credentials**.

Run:

```bash
gcloud auth application-default login
```

A browser will open. Authenticate with your Google account.

Then your local Python application can authenticate to Google Cloud without putting a service-account key JSON file inside your project.

You can verify:

```bash
gcloud auth application-default print-access-token
```

If you get an access token, your local environment is authenticated.

---

# 4. Install VS Code

Install VS Code, then install these extensions:

```text
Python
Python Debugger
Pylance
GitHub Pull Requests
```

For AI-assisted development, you can additionally use your preferred coding assistant.

Your IDE should eventually look like:

```text
VS Code
│
├── Explorer
│   └── financial-agent/
│
├── Terminal
│
├── Python
│
└── Debugger
```

---

# 5. Install `uv`

I recommend `uv` instead of manually managing `venv` + `pip`.

On Windows PowerShell:

```powershell
irm https://astral.sh/uv/install.ps1 | iex
```

Verify:

```bash
uv --version
```

---

# 6. Create the project

Create a directory:

```bash
mkdir financial-agent
cd financial-agent
```

Initialize a Python project:

```bash
uv init
```

You'll get something similar to:

```text
financial-agent/
├── pyproject.toml
├── README.md
└── .python-version
```

Now create your virtual environment:

```bash
uv venv
```

Activate it.

Windows:

```powershell
.venv\Scripts\activate
```

Mac/Linux:

```bash
source .venv/bin/activate
```

---

# 7. Install Google ADK

```bash
uv add google-adk
```

Now your `pyproject.toml` will contain the ADK dependency.

You can verify:

```bash
uv run python -c "import google.adk; print('ADK OK')"
```

---

# 8. Create the ADK agent structure

For your first project, I recommend:

```text
financial-agent/
│
├── app/
│   └── financial_agent/
│       ├── __init__.py
│       ├── agent.py
│       │
│       ├── tools/
│       │   ├── __init__.py
│       │   ├── search.py
│       │   └── sql.py
│       │
│       ├── agents/
│       │   ├── __init__.py
│       │   └── research_agent.py
│       │
│       └── prompts/
│           └── financial.txt
│
├── tests/
│   └── test_tools.py
│
├── evals/
│   └── financial_eval.json
│
├── .env
├── .gitignore
├── pyproject.toml
└── README.md
```

For a very first experiment, however, you can start much simpler:

```text
financial-agent/
│
├── agent.py
├── __init__.py
├── .env
├── .gitignore
├── pyproject.toml
└── README.md
```

I recommend starting simple and expanding the structure once the agent works.

---

# 9. Create your first agent

Create:

```text
agent.py
```

Put:

```python
from google.adk.agents import Agent

root_agent = Agent(
    name="financial_agent",
    model="gemini-2.5-flash",
    description="A financial research assistant",
    instruction="""
    You are a financial research assistant.

    Answer financial questions accurately.
    If information is unavailable, say so.
    Do not invent financial information.
    """
)
```

That's your first ADK agent.

---

# 10. Configure Vertex AI

For Google Cloud/Vertex AI development, configure:

```text
GOOGLE_CLOUD_PROJECT=your-project-id
GOOGLE_CLOUD_LOCATION=us-central1
GOOGLE_GENAI_USE_VERTEXAI=TRUE
```

You can put these in `.env` if your local ADK setup expects environment configuration.

For example:

```text
GOOGLE_CLOUD_PROJECT=my-genai-project
GOOGLE_CLOUD_LOCATION=us-central1
GOOGLE_GENAI_USE_VERTEXAI=TRUE
```

Don't commit `.env`.

Add this to `.gitignore`:

```text
.env
.venv/
__pycache__/
*.pyc
```

---

# 11. Run your agent locally

ADK provides a local development experience.

From the directory containing your ADK application, you can use:

```bash
adk web
```

Then open the local development UI, typically:

```text
http://localhost:8000
```

Depending on your installed ADK version, the CLI commands can differ slightly, so you can check:

```bash
adk --help
```

You can also use Google's current Agents CLI for scaffolding and development:

```bash
uvx google-agents-cli
```

The important concept is:

```text
                 Your Laptop

VS Code
   │
   ▼
Python
   │
   ▼
Google ADK
   │
   ▼
Local Agent Server
   │
   ▼
Vertex AI
   │
   ▼
Gemini
```

---

# 12. Add your first tool

Now let's turn the toy agent into something useful.

Create:

```text
tools/search.py
```

For example:

```python
def search_company_documents(query: str) -> str:
    """
    Search company financial documents.

    Args:
        query: Search question.

    Returns:
        Relevant document content.
    """

    # Later replace this with
    # Vertex AI RAG / Vector Search.

    return f"Search results for: {query}"
```

Then:

```python
from google.adk.agents import Agent
from .tools.search import search_company_documents

root_agent = Agent(
    name="financial_agent",
    model="gemini-2.5-flash",
    description="Financial research assistant",
    instruction="""
    Answer financial questions using
    company documents.

    Always search the documents when
    the user asks about company-specific
    information.
    """,
    tools=[
        search_company_documents
    ],
)
```

Now the architecture becomes:

```text
User
 │
 ▼
ADK Agent
 │
 ▼
Gemini
 │
 ├──── search_company_documents()
 │
 ▼
Search results
 │
 ▼
Gemini
 │
 ▼
Answer
```

---

# 13. Then replace your fake search with Vertex AI RAG

Once the basic agent works, don't immediately build everything.

Do it in stages:

### Stage 1

```text
ADK
 +
Gemini
```

### Stage 2

```text
ADK
 +
Gemini
 +
Python tools
```

### Stage 3

```text
ADK
 +
Gemini
 +
RAG
```

### Stage 4

```text
ADK
 +
Gemini
 +
Vertex AI RAG
 +
Vector Search
```

### Stage 5

```text
ADK
 +
Gemini
 +
RAG
 +
BigQuery
 +
MCP
```

### Stage 6

```text
Multi-Agent
     │
     ├── RAG Agent
     ├── SQL Agent
     ├── Research Agent
     └── Review Agent
```

That's a much better learning path than trying to build the entire Google Agent Platform on day one.

---

# 14. Your eventual project

For your background, I would make your first serious project a **Financial Research Agent**:

```text
                       User
                        │
                        ▼
                Financial Manager
                    ADK Agent
                        │
          ┌─────────────┼──────────────┐
          ▼             ▼              ▼
      RAG Agent      SQL Agent      Calculator
          │             │              │
          ▼             ▼              ▼
     Vertex RAG      BigQuery       Python
          │             │
          └─────────────┼──────────────┘
                        ▼
                      Gemini
                        │
                        ▼
               Answer + Citations
```

Your local repo could ultimately become:

```text
financial-agent/
│
├── app/
│   └── financial_agent/
│       │
│       ├── agent.py
│       │
│       ├── agents/
│       │   ├── manager.py
│       │   ├── rag_agent.py
│       │   └── sql_agent.py
│       │
│       ├── tools/
│       │   ├── rag.py
│       │   ├── sql.py
│       │   └── calculator.py
│       │
│       └── prompts/
│           ├── manager.txt
│           └── analyst.txt
│
├── tests/
│   ├── test_rag.py
│   ├── test_sql.py
│   └── test_agent.py
│
├── evals/
│   └── financial_questions.json
│
├── .env
├── .gitignore
├── pyproject.toml
└── README.md
```

Then your **developer lifecycle** is:

```text
                 VS Code
                    │
                    ▼
              Write ADK code
                    │
                    ▼
             Run locally
                    │
                    ▼
             Debug agent
                    │
                    ▼
              Unit tests
                    │
                    ▼
             Agent evaluation
                    │
                    ▼
                  Git
                    │
                    ▼
                  CI/CD
                    │
                    ▼
           Vertex AI / Agent Runtime
                    │
                    ▼
              Production
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
      Monitoring          Evaluation
          │                   │
          └─────────┬─────────┘
                    ▼
                 Improve
```

**The key setup is therefore:** **VS Code → Python/uv → ADK → local agent → Vertex AI → Gemini**, and then add **Vertex AI RAG/Vector Search, BigQuery, evaluation, and managed deployment** as the project grows.
